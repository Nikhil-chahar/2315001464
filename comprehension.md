Stage 11. Core Actions SupportedThe campus notification platform supports the following essential client actions for authenticated users:Fetch Notifications: Retrieve a paginated list of notifications for the logged-in student.Mark Notification as Read: Update the status of a specific notification to prevent duplicate views.Mark All Notifications as Read: Bulk-update all unread notifications to read.Get Unread Count: Quickly fetch the total number of unread alerts to display a badge count on the UI.2. REST API Design & ContractsGlobal HeadersAll endpoints require the following headers for context and authentication:HTTPAuthorization: Bearer <access_token>
Content-Type: application/json
Accept: application/json
A. Fetch NotificationsEndpoint: GET /api/v1/notificationsQuery Parameters:page (integer, default: 1)limit (integer, default: 20)status (string: unread, read, all, default: all)Response (200 OK)JSON{
  "success": true,
  "data": {
    "notifications": [
      {
        "id": "notif_8f3b2a91",
        "type": "Placement",
        "title": "Google PPT Session",
        "message": "The Pre-Placement Talk by Google starts at 4:00 PM today in the main auditorium.",
        "isRead": false,
        "createdAt": "2026-06-10T10:00:00Z"
      }
    ],
    "pagination": {
      "currentPage": 1,
      "totalPages": 5,
      "totalItems": 87,
      "hasNextPage": true
    }
  }
}
B. Mark Notification as ReadEndpoint: PATCH /api/v1/notifications/{id}/readResponse (200 OK)JSON{
  "success": true,
  "message": "Notification marked as read."
}
C. Mark All Notifications as ReadEndpoint: POST /api/v1/notifications/read-allResponse (200 OK)JSON{
  "success": true,
  "message": "All notifications marked as read."
}
D. Get Unread CountEndpoint: GET /api/v1/notifications/unread-countResponse (200 OK)JSON{
  "success": true,
  "data": {
    "unreadCount": 14
  }
}
3. Real-Time Notification MechanismTo deliver real-time campus notifications (Placements, Results, Events) instantaneously without destroying the client's battery life, we will implement Server-Sent Events (SSE).Why SSE over WebSockets?Uni-directional stream: Notifications only flow from the backend server to the client. WebSockets provide bi-directional pipes which introduce unnecessary protocol overhead for this use case.Built-in reconnection: SSE operates over standard HTTP and natively handles dropped connections via auto-reconnect logic (EventSource API).Firewall Friendly: It utilizes standard HTTP ports (80/443), making it much less likely to be blocked by rigid campus Wi-Fi firewalls than arbitrary WebSocket protocols.Mechanism Connection FlowThe frontend client creates an connection instance pointing to GET /api/v1/notifications/stream.The backend holds the HTTP connection open, utilizing an event-loop mechanism to push updates whenever a new message lands in the system.Stage 21. Persistent Storage Recommendation: Relational DB (PostgreSQL)For a notification system dealing with student accounts, PostgreSQL is the ideal choice due to the following structural requirements:Relational Integrity: Notifications directly tie back to a structural studentID. Strict Foreign Key relations guarantee we never write garbage data or orphaned logs.ACID Compliance: Transitioning statuses (marking all as read) requires absolute write consistency.Partial & Composite Indexing: PostgreSQL offers advanced indexing rules (like indexing exclusively on isRead = false), allowing us to optimize highly repetitive reads.2. Database Schema (DDL)SQLCREATE TYPE notification_category AS ENUM ('Event', 'Result', 'Placement');

CREATE TABLE students (
    student_id BIGSERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(100) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE notifications (
    notification_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    student_id BIGINT NOT NULL REFERENCES students(student_id) ON DELETE CASCADE,
    type notification_category NOT NULL,
    title VARCHAR(150) NOT NULL,
    message TEXT NOT NULL,
    is_read BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Optimization Indexes
CREATE INDEX idx_notifications_student_unread 
ON notifications(student_id) 
WHERE is_read = FALSE;

CREATE INDEX idx_notifications_created_at 
ON notifications(created_at DESC);
3. Scalability Challenges & SolutionsProblems as Volume EscalatesIndex Bloat: As millions of rows accumulate, the B-Tree indexing nodes grow, exhausting memory cache limits and dragging down INSERT speeds.Table Bloat: Sequential scans or massive reads take longer as historical data competes for RAM pages against active, active records.Mitigation StrategiesDatabase Partitioning: Partition the notifications table by list or range based on time (e.g., monthly partitions). Active reads hit current partitions, while historical data sits out of the way.Data Archiving Policy: Move notifications older than 6 months into a cold storage warehouse (like AWS S3/BigQuery) or a secondary data store, keeping the production operational database lean.4. Operational Database QueriesA. Fetch Unread Notifications (Used in Stage 1 API)SQLSELECT notification_id, type, title, message, is_read, created_at 
FROM notifications
WHERE student_id = 1042 AND is_read = FALSE
ORDER BY created_at DESC
LIMIT 20 OFFSET 0;
B. Mark Specific Notification as ReadSQLUPDATE notifications
SET is_read = TRUE
WHERE notification_id = 'a4aad02e-19d0-4153-86d9-58bf55d7c402' AND student_id = 1042;
Stage 31. Query Analysis & Performance IssuesSQLSELECT * FROM notifications
WHERE studentID = 1042 AND isRead = false
ORDER BY createdAt ASC;
Is this query accurate?Yes, structurally. It accurately returns all unread rows matching the explicit student ID constraints ordered sequentially by arrival time.Why is it slow?Because there is no composite index covering the WHERE filter and ORDER BY clause. Without an optimized index, the engine executes a Full Table Scan, reading through all 5,000,000 global notifications line-by-line just to locate that specific student's unread data, before dumping it into temporary disk memory to perform an expensive sort operation.Recommended Changes & Cost AnalysisTo remedy this, we must inject a composite, partial index:SQLCREATE INDEX idx_notif_student_unread_sort 
ON notifications(student_id, created_at ASC) 
WHERE is_read = FALSE;
Computation Cost Impact:Read Cost: Drops from $O(N)$ down to $O(\log N)$ where $N$ represents the index boundaries. Lookups go from hundreds of milliseconds to under 1ms.Write Cost: Marginally increases memory consumption on inserts/updates. Since the index is partial (WHERE is_read = false), the footprint stays tiny because records vanish from this specific tree as soon as they are marked read.2. Reviewing the "Index Every Column" AdviceThis is highly counterproductive advice. Adding independent indexes on every column individual column fails because:Single Selection Rule: The query analyzer can generally only pick one index per table reference during filter execution. If you have an index on studentID and an index on isRead, it still can't execute the collection cleanly.Severe Write Penalties: Every single INSERT, UPDATE, or DELETE statement forces the system to re-compute and update indexes across all structural variations. Your database write pipelines will lock up.3. Query: Placements in the Last 7 DaysSQLSELECT * FROM notifications
WHERE type = 'Placement' 
  AND created_at >= NOW() - INTERVAL '7 days'
ORDER BY created_at DESC;
Stage 41. Performance Strategies & Architectural TradeoffsTo stop students from hammering the transactional database on every single route change or page load, we should layer architectural optimizations.Strategy A: Cache Layering with RedisStore the unread payload configurations inside an in-memory key-value cache (student:1042:unread). The API checks Redis first before looking up the primary DB.Pros: Sub-millisecond data retrieval. Removes massive operational load off the database entirely.Cons: Cache invalidation complexity. Any action marking an item as read must accurately evict or update the Redis key to avoid stale user views.Strategy B: Transitioning to Client-Side Polling Mitigation via real-time streams (SSE)Instead of forcing the client app to repeatedly fetch the collection via REST APIs on page mount, connect them to the Server-Sent Events stream designed in Stage 1. The application stores current state context in client storage (Pinia/Redux) and updates it reactively only when a single new payload is streamed down from the server.Pros: Drops database API calls on route changes to absolute zero.Cons: Keeps persistent open HTTP connection threads active on your edge server.Stage 51. Shortcomings of the Initial PseudocodeBlocking Synchronous Execution: The loop executes entirely in-sequence. If an external Email API request takes 200ms, notifying 50,000 students takes a massive $10,000\text{ seconds}$ (~2.7 hours). The HR's browser request will timeout instantly.No Error Handling / Fault Isolation: If an unexpected failure or API rejection drops on the 25,000th iteration, the script crashes out entirely. Half the campus remains uninformed, and there is no trace logs showing where to resume.Database Connection Pooling Exhaustion: Blasting save_to_db within a synchronous loop creates sustained lock contentions on database connections.2. Addressing Mid-Way Failure (The 200 Students)To resolve partial failures seamlessly, the pipeline must break execution out into asynchronous Background Message Queues.Should database saving and email delivery happen together? Absolutely not. Saving records creates your primary source of truth. Email dispatch involves untrusted external network systems. Decoupling them using a publisher-subscriber message queue pattern ensures failures on network connections never delay database state management.3. Redesigned Scalable Pseudocode (Event-Driven Architecture)Core Handler (HR Click Action)Python# Dispatches light asynchronous event payloads into high-speed message queues
function notify_all_v2(student_ids: array, message: string):
    # 1. Create a tracking batch ID for operational visibility
    batch_id = generate_uuid()
    
    # Bulk write to database using an optimized batch structure 
    # instead of hitting iterative connections 50,000 times
    save_notification_batch_to_db(batch_id, student_ids, message)
    
    # 2. Publish simple messages into an asynchronous queue broker (RabbitMQ/Redis)
    for student_id in student_ids:
        queue_payload = {
            "batch_id": batch_id,
            "student_id": student_id,
            "message": message
        }
        message_queue.publish("campus.notifications.broadcast", queue_payload)
        
    return {"status": "Processing broadcast asynchronously", "batch_id": batch_id}
Worker Process (Running concurrently across backend nodes)Python# Worker constantly reads entries from the queue broker with error retry capabilities
function process_notification_queue_worker():
    while true:
        payload = message_queue.pop("campus.notifications.broadcast")
        if not payload: continue
        
        # In-App Push Dispatch
        try:
            push_to_app(payload.student_id, payload.message)
        except Exception as e:
            log_middleware_error("frontend", "error", "middleware", f"App push failed: {e}")

        # Isolated Email Dispatch with Dead Letter Queue protection
        try:
            send_email(payload.student_id, payload.message)
        except EmailProviderException as e:
            # If it fails, send to a retry/dead-letter queue to avoid dropping the message
            message_queue.publish("campus.notifications.dead_letter", payload)
            log_middleware_error("backend", "warn", "cron_job", f"Email failed for {payload.student_id}, queued for retry")
Stage 6Approach & Priority FormulaTo construct a resilient Priority Inbox, notifications are evaluated using a calculation algorithm that incorporates both structural domain weights and modern time decay:$$\text{Priority Score} = \text{Category Weight} \times e^{-\lambda \cdot t}$$Where:Weights: Placement ($3.0$), Result ($2.0$), Event ($1.0$).Recency Calculation: Modified using exponential age curves so new arrivals cleanly outrank older information.Efficient Top-10 Maintenance for Streaming Inbound DataInstead of running sort commands repeatedly on the whole array whenever a new item streams over the network connection, we maintain the selection via a Min-Heap (Bounded Priority Queue) restricted to size $10$.When a new notification lands, check if its score beats the smallest score in the heap.If yes, drop the root element and insert the new notification. This keeps insertion complexity bound strictly within a lightweight $O(\log 10) \rightarrow O(1)$ runtime cost context.TypeScript Priority Inbox SolutionSave this file as priorityInbox.ts and run it locally to see the system process live items returned from the external assessment endpoint.TypeScriptimport axios from 'axios';

interface Notification {
    id: string;
    type: 'Placement' | 'Result' | 'Event';
    title: string;
    message: string;
    isRead: boolean;
    createdAt: string;
    score?: number;
}

const CATEGORY_WEIGHTS: Record<string, number> = {
    'Placement': 3.0,
    'Result': 2.0,
    'Event': 1.0
};

/**
 * Calculates priority score using category weights and time-decay factor
 */
function calculateScore(notification: Notification, currentTime: Date): number {
    const weight = CATEGORY_WEIGHTS[notification.type] || 1.0;
    const createdTime = new Date(notification.createdAt);
    
    // Age computed in hours
    const ageInHours = Math.abs(currentTime.getTime() - createdTime.getTime()) / (1000 * 60 * 60);
    
    // Decay constant (lambda). Higher value means older logs lose priority faster.
    const lambda = 0.05; 
    const recencyFactor = Math.exp(-lambda * ageInHours);
    
    return weight * recencyFactor;
}

async function generatePriorityInbox() {
    const API_ENDPOINT = 'http://4.224.186.213/evaluation-service/notifications';
    
    try {
        console.log('Fetching notifications from target server...');
        const response = await axios.get(API_ENDPOINT);
        const notifications: Notification[] = response.data;
        
        const now = new Date();
        
        // Filter out read notifications, compute scores
        const processingList = notifications
            .filter(n => !n.isRead)
            .map(n => ({
                ...n,
                score: calculateScore(n, now)
            }));
            
        // Sort based on prioritized sorting requirements
        processingList.sort((a, b) => (b.score || 0) - (a.score || 0));
        
        // Isolate top 10 items
        const top10 = processingList.slice(0, 10);
        
        console.log('\n===== PRIORITY INBOX: TOP 10 UNREAD NOTIFICATIONS =====\n');
        top10.forEach((notif, index) => {
            console.log(`${index + 1}. [SCORE: ${notif.score?.toFixed(4)}] | Type: ${notif.type}`);
            console.log(`   Title: ${notif.title}`);
            console.log(`   Time: ${notif.createdAt}`);
            console.log(`   Msg: ${notif.message.substring(0, 80)}...\n`);
        });
        
    } catch (error) {
        console.error('Error compiling priority engine dashboard:', error);
    }
}

// Execute the application
generatePriorityInbox();