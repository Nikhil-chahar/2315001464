# Campus Notification Platform - System Design

## Stage 1: REST API Design & Architecture

### Overview
The notification platform is designed to deliver real-time updates to students regarding three core categories: **Placements**, **Events**, and **Results**. The system uses a pull-based REST API architecture combined with WebSocket support for real-time notifications.

### Core Actions
The notification system supports the following core actions:

1. **Fetch Notifications** - Retrieve list of notifications for the authenticated user
2. **Get Notification Details** - Retrieve full details of a specific notification
3. **Mark Notification as Read** - Update read status for individual notifications
4. **Mark All Notifications as Read** - Bulk update read status
5. **Delete Notification** - Remove a notification from the user's list
6. **Get Notification Statistics** - Fetch count of unread, read, and total notifications
7. **Set Notification Preferences** - Configure notification categories and frequency
8. **Get Notification Preferences** - Retrieve user's notification settings
9. **Subscribe to Real-time Notifications** - WebSocket connection for push updates

---

## REST API Endpoints

### 1. Get All Notifications
**Endpoint:** `GET /api/v1/notifications`

**Purpose:** Retrieve paginated list of notifications for the authenticated user

**Request Headers:**
```
Content-Type: application/json
Authorization: Bearer <token>
Accept: application/json
```

**Request Parameters (Query):**
```
{
  "page": 1,
  "limit": 20,
  "category": "all|placements|events|results",
  "status": "all|read|unread",
  "sortBy": "date|importance",
  "order": "asc|desc"
}
```

**Response Headers:**
```
Content-Type: application/json
X-Total-Count: <total_count>
X-Page: <current_page>
X-Limit: <items_per_page>
X-Unread-Count: <unread_count>
Cache-Control: no-cache
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "message": "Notifications retrieved successfully",
  "data": {
    "notifications": [
      {
        "id": "notif_123456",
        "userId": "user_789",
        "category": "placements",
        "type": "offer",
        "title": "Congratulations! New Job Offer",
        "description": "TCS has sent you an offer for the Software Engineer position",
        "content": "You have received an internship offer from TCS for Rs. 6 LPA. Please respond by 5 PM today.",
        "priority": "high",
        "isRead": false,
        "createdAt": "2026-06-10T10:30:00Z",
        "updatedAt": "2026-06-10T10:30:00Z",
        "expiresAt": "2026-06-17T10:30:00Z",
        "metadata": {
          "company": "TCS",
          "position": "Software Engineer",
          "salary": "6 LPA"
        },
        "actionUrl": "/placements/offer/123456",
        "readAt": null
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 45,
      "totalPages": 3
    },
    "statistics": {
      "unreadCount": 5,
      "readCount": 40,
      "totalCount": 45
    }
  },
  "timestamp": "2026-06-10T10:35:00Z"
}
```

**Error Response (400 Bad Request):**
```json
{
  "success": false,
  "code": 400,
  "message": "Invalid pagination parameters",
  "error": "Limit must be between 1 and 100"
}
```

**Error Response (401 Unauthorized):**
```json
{
  "success": false,
  "code": 401,
  "message": "Unauthorized access",
  "error": "Valid authorization token required"
}
```

---

### 2. Get Notification by ID
**Endpoint:** `GET /api/v1/notifications/{notificationId}`

**Purpose:** Retrieve detailed information about a specific notification

**Request Headers:**
```
Content-Type: application/json
Authorization: Bearer <token>
Accept: application/json
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "message": "Notification retrieved successfully",
  "data": {
    "id": "notif_123456",
    "userId": "user_789",
    "category": "placements",
    "type": "offer",
    "title": "Congratulations! New Job Offer",
    "description": "TCS has sent you an offer for the Software Engineer position",
    "content": "You have received an internship offer from TCS for Rs. 6 LPA. Please respond by 5 PM today.",
    "priority": "high",
    "isRead": false,
    "createdAt": "2026-06-10T10:30:00Z",
    "updatedAt": "2026-06-10T10:30:00Z",
    "expiresAt": "2026-06-17T10:30:00Z",
    "metadata": {
      "company": "TCS",
      "position": "Software Engineer",
      "salary": "6 LPA",
      "response_deadline": "2026-06-10T17:00:00Z"
    },
    "actionUrl": "/placements/offer/123456",
    "readAt": null,
    "attachments": [
      {
        "id": "att_001",
        "name": "offer_letter.pdf",
        "url": "/attachments/att_001",
        "type": "pdf",
        "size": 256000
      }
    ]
  },
  "timestamp": "2026-06-10T10:35:00Z"
}
```

**Error Response (404 Not Found):**
```json
{
  "success": false,
  "code": 404,
  "message": "Notification not found",
  "error": "Notification with ID notif_123456 does not exist"
}
```

---

### 3. Mark Notification as Read
**Endpoint:** `PATCH /api/v1/notifications/{notificationId}/read`

**Purpose:** Mark a single notification as read

**Request Headers:**
```
Content-Type: application/json
Authorization: Bearer <token>
Accept: application/json
```

**Request Body:**
```json
{
  "readAt": "2026-06-10T10:35:00Z"
}
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "message": "Notification marked as read",
  "data": {
    "id": "notif_123456",
    "isRead": true,
    "readAt": "2026-06-10T10:35:00Z"
  },
  "timestamp": "2026-06-10T10:35:00Z"
}
```

---

### 4. Mark All Notifications as Read
**Endpoint:** `PATCH /api/v1/notifications/read-all`

**Purpose:** Mark all or filtered notifications as read in bulk

**Request Headers:**
```
Content-Type: application/json
Authorization: Bearer <token>
Accept: application/json
```

**Request Body:**
```json
{
  "category": "placements|events|results|all",
  "readAt": "2026-06-10T10:35:00Z"
}
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "message": "Notifications marked as read",
  "data": {
    "markedCount": 12,
    "category": "placements",
    "readAt": "2026-06-10T10:35:00Z"
  },
  "timestamp": "2026-06-10T10:35:00Z"
}
```

---

### 5. Delete Notification
**Endpoint:** `DELETE /api/v1/notifications/{notificationId}`

**Purpose:** Delete a notification from user's list

**Request Headers:**
```
Content-Type: application/json
Authorization: Bearer <token>
Accept: application/json
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "message": "Notification deleted successfully",
  "data": {
    "id": "notif_123456",
    "deleted": true
  },
  "timestamp": "2026-06-10T10:35:00Z"
}
```

---

### 6. Get Notification Statistics
**Endpoint:** `GET /api/v1/notifications/stats`

**Purpose:** Get aggregated notification statistics for the user

**Request Headers:**
```
Content-Type: application/json
Authorization: Bearer <token>
Accept: application/json
```

**Request Parameters (Query):**
```
{
  "category": "all|placements|events|results",
  "days": 7
}
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "message": "Statistics retrieved successfully",
  "data": {
    "summary": {
      "totalNotifications": 150,
      "unreadCount": 8,
      "readCount": 142,
      "expiredCount": 5
    },
    "byCategory": {
      "placements": {
        "total": 60,
        "unread": 3,
        "read": 57
      },
      "events": {
        "total": 50,
        "unread": 3,
        "read": 47
      },
      "results": {
        "total": 40,
        "unread": 2,
        "read": 38
      }
    },
    "byPriority": {
      "high": 15,
      "medium": 85,
      "low": 50
    },
    "trendLast7Days": [
      {
        "date": "2026-06-04",
        "count": 12
      },
      {
        "date": "2026-06-05",
        "count": 15
      },
      {
        "date": "2026-06-10",
        "count": 8
      }
    ]
  },
  "timestamp": "2026-06-10T10:35:00Z"
}
```

---

### 7. Get Notification Preferences
**Endpoint:** `GET /api/v1/notifications/preferences`

**Purpose:** Retrieve user's notification settings and preferences

**Request Headers:**
```
Content-Type: application/json
Authorization: Bearer <token>
Accept: application/json
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "message": "Preferences retrieved successfully",
  "data": {
    "userId": "user_789",
    "preferences": {
      "categories": {
        "placements": {
          "enabled": true,
          "types": ["offer", "rejection", "shortlist"]
        },
        "events": {
          "enabled": true,
          "types": ["tech_talks", "workshops", "competitions"]
        },
        "results": {
          "enabled": true,
          "types": ["exam_results", "gpa_update"]
        }
      },
      "channels": {
        "inApp": true,
        "email": true,
        "push": true,
        "sms": false
      },
      "frequency": "immediate",
      "quietHours": {
        "enabled": true,
        "startTime": "22:00",
        "endTime": "08:00"
      },
      "doNotDisturb": {
        "enabled": false,
        "until": null
      }
    },
    "createdAt": "2026-01-15T10:00:00Z",
    "updatedAt": "2026-06-10T08:00:00Z"
  },
  "timestamp": "2026-06-10T10:35:00Z"
}
```

---

### 8. Update Notification Preferences
**Endpoint:** `PUT /api/v1/notifications/preferences`

**Purpose:** Update user's notification settings

**Request Headers:**
```
Content-Type: application/json
Authorization: Bearer <token>
Accept: application/json
```

**Request Body:**
```json
{
  "preferences": {
    "categories": {
      "placements": {
        "enabled": true,
        "types": ["offer", "rejection", "shortlist"]
      },
      "events": {
        "enabled": false,
        "types": []
      },
      "results": {
        "enabled": true,
        "types": ["exam_results"]
      }
    },
    "channels": {
      "inApp": true,
      "email": true,
      "push": false,
      "sms": false
    },
    "frequency": "daily",
    "quietHours": {
      "enabled": true,
      "startTime": "23:00",
      "endTime": "08:00"
    }
  }
}
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "message": "Preferences updated successfully",
  "data": {
    "userId": "user_789",
    "preferences": {
      "categories": {
        "placements": {
          "enabled": true,
          "types": ["offer", "rejection", "shortlist"]
        },
        "events": {
          "enabled": false,
          "types": []
        },
        "results": {
          "enabled": true,
          "types": ["exam_results"]
        }
      },
      "channels": {
        "inApp": true,
        "email": true,
        "push": false,
        "sms": false
      },
      "frequency": "daily",
      "quietHours": {
        "enabled": true,
        "startTime": "23:00",
        "endTime": "08:00"
      }
    },
    "updatedAt": "2026-06-10T10:35:00Z"
  },
  "timestamp": "2026-06-10T10:35:00Z"
}
```

---

## Real-Time Notification Mechanism

### WebSocket Connection
**Endpoint:** `WS /api/v1/notifications/stream`

**Purpose:** Establish WebSocket connection for real-time push notifications

**Connection Request:**
```
GET /api/v1/notifications/stream?token=<auth_token>
Connection: Upgrade
Upgrade: websocket
Sec-WebSocket-Version: 13
Sec-WebSocket-Key: <random_key>
```

**Connection Response (101 Switching Protocols):**
```
HTTP/1.1 101 Switching Protocols
Connection: Upgrade
Upgrade: websocket
Sec-WebSocket-Accept: <response_key>
```

### WebSocket Message Format

**Real-time Notification Message:**
```json
{
  "type": "notification",
  "event": "new_notification",
  "timestamp": "2026-06-10T10:35:00Z",
  "data": {
    "id": "notif_123456",
    "userId": "user_789",
    "category": "placements",
    "type": "offer",
    "title": "Congratulations! New Job Offer",
    "description": "TCS has sent you an offer for the Software Engineer position",
    "priority": "high",
    "metadata": {
      "company": "TCS",
      "position": "Software Engineer",
      "salary": "6 LPA"
    }
  }
}
```

**Heartbeat Message:**
```json
{
  "type": "heartbeat",
  "timestamp": "2026-06-10T10:35:00Z"
}
```

**Connection Status Message:**
```json
{
  "type": "connection_status",
  "status": "connected",
  "userId": "user_789",
  "timestamp": "2026-06-10T10:35:00Z",
  "message": "WebSocket connection established"
}
```

---

## JSON Schema Definitions

### Notification Schema
```json
{
  "type": "object",
  "required": ["id", "userId", "category", "type", "title", "priority", "createdAt"],
  "properties": {
    "id": {
      "type": "string",
      "pattern": "^notif_[a-zA-Z0-9]{20}$",
      "description": "Unique notification identifier"
    },
    "userId": {
      "type": "string",
      "pattern": "^user_[a-zA-Z0-9]{10}$",
      "description": "User ID receiving the notification"
    },
    "category": {
      "type": "string",
      "enum": ["placements", "events", "results"],
      "description": "Notification category"
    },
    "type": {
      "type": "string",
      "description": "Specific notification type within category"
    },
    "title": {
      "type": "string",
      "maxLength": 200,
      "description": "Notification title"
    },
    "description": {
      "type": "string",
      "maxLength": 500,
      "description": "Short description/preview"
    },
    "content": {
      "type": "string",
      "description": "Full notification content"
    },
    "priority": {
      "type": "string",
      "enum": ["low", "medium", "high"],
      "description": "Notification priority level"
    },
    "isRead": {
      "type": "boolean",
      "description": "Read status of notification"
    },
    "readAt": {
      "type": ["string", "null"],
      "format": "date-time",
      "description": "Timestamp when notification was read"
    },
    "createdAt": {
      "type": "string",
      "format": "date-time",
      "description": "Notification creation timestamp"
    },
    "updatedAt": {
      "type": "string",
      "format": "date-time",
      "description": "Last update timestamp"
    },
    "expiresAt": {
      "type": ["string", "null"],
      "format": "date-time",
      "description": "Expiration timestamp"
    },
    "metadata": {
      "type": "object",
      "description": "Category-specific metadata"
    },
    "actionUrl": {
      "type": ["string", "null"],
      "format": "uri",
      "description": "URL for action related to notification"
    },
    "attachments": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "id": {"type": "string"},
          "name": {"type": "string"},
          "url": {"type": "string", "format": "uri"},
          "type": {"type": "string"},
          "size": {"type": "integer"}
        }
      }
    }
  }
}
```

### Notification Preferences Schema
```json
{
  "type": "object",
  "required": ["userId", "preferences"],
  "properties": {
    "userId": {
      "type": "string",
      "description": "User ID"
    },
    "preferences": {
      "type": "object",
      "properties": {
        "categories": {
          "type": "object",
          "properties": {
            "placements": {
              "type": "object",
              "properties": {
                "enabled": {"type": "boolean"},
                "types": {"type": "array", "items": {"type": "string"}}
              }
            },
            "events": {
              "type": "object",
              "properties": {
                "enabled": {"type": "boolean"},
                "types": {"type": "array", "items": {"type": "string"}}
              }
            },
            "results": {
              "type": "object",
              "properties": {
                "enabled": {"type": "boolean"},
                "types": {"type": "array", "items": {"type": "string"}}
              }
            }
          }
        },
        "channels": {
          "type": "object",
          "properties": {
            "inApp": {"type": "boolean"},
            "email": {"type": "boolean"},
            "push": {"type": "boolean"},
            "sms": {"type": "boolean"}
          }
        },
        "frequency": {
          "type": "string",
          "enum": ["immediate", "hourly", "daily", "weekly"]
        },
        "quietHours": {
          "type": "object",
          "properties": {
            "enabled": {"type": "boolean"},
            "startTime": {"type": "string", "pattern": "^([0-1]?[0-9]|2[0-3]):[0-5][0-9]$"},
            "endTime": {"type": "string", "pattern": "^([0-1]?[0-9]|2[0-3]):[0-5][0-9]$"}
          }
        }
      }
    },
    "createdAt": {
      "type": "string",
      "format": "date-time"
    },
    "updatedAt": {
      "type": "string",
      "format": "date-time"
    }
  }
}
```

---

## Error Response Schema
```json
{
  "type": "object",
  "required": ["success", "code", "message"],
  "properties": {
    "success": {
      "type": "boolean",
      "enum": [false]
    },
    "code": {
      "type": "integer",
      "enum": [400, 401, 403, 404, 409, 500, 503]
    },
    "message": {
      "type": "string",
      "description": "Error message"
    },
    "error": {
      "type": ["string", "object"],
      "description": "Detailed error information"
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    }
  }
}
```

---

## HTTP Status Codes
- **200 OK** - Request successful
- **201 Created** - Resource created successfully
- **204 No Content** - Successful but no content to return
- **400 Bad Request** - Invalid request parameters
- **401 Unauthorized** - Missing or invalid authorization token
- **403 Forbidden** - User lacks permissions
- **404 Not Found** - Resource not found
- **409 Conflict** - Resource conflict (e.g., duplicate)
- **422 Unprocessable Entity** - Validation error
- **429 Too Many Requests** - Rate limit exceeded
- **500 Internal Server Error** - Server error
- **503 Service Unavailable** - Service temporarily unavailable

---

# Stage 2: Database Design & Scalability

## Database Choice: MongoDB (NoSQL)

### Rationale for Choosing MongoDB

**Why MongoDB:**
1. **Flexible Schema** - Notification metadata varies significantly by category (placements, events, results). MongoDB's schema-less nature allows storing diverse data structures without predefined rigid schemas.

2. **Scalability** - Built-in horizontal scaling through sharding allows efficient distribution of data as user base grows.

3. **Real-time Performance** - Optimized for rapid write operations needed in notification systems where thousands of notifications are generated simultaneously.

4. **Indexing Capabilities** - Supports compound indexes on multiple fields (userId, category, isRead, createdAt) for efficient querying of large datasets.

5. **TTL (Time-To-Live) Indexes** - Native support for automatic expiration of old notifications reduces manual cleanup overhead.

6. **Replication & High Availability** - Replica sets ensure data redundancy and automatic failover.

7. **Aggregation Pipeline** - Powerful aggregation framework for computing statistics and analytics queries efficiently.

8. **Horizontal Scaling** - Sharding support allows distributing data across multiple servers as data volume increases.

### MongoDB Atlas Benefits
- Automated backups and disaster recovery
- Built-in monitoring and alerting
- Multi-region deployment for geo-redundancy
- Automatic scaling based on performance metrics

---

## Database Schema

### Collections Overview

#### 1. notifications Collection
Primary collection storing all notifications

```javascript
db.createCollection("notifications", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["userId", "category", "type", "title", "priority", "createdAt"],
      properties: {
        _id: { bsonType: "objectId" },
        id: { 
          bsonType: "string",
          pattern: "^notif_[a-zA-Z0-9]{20}$"
        },
        userId: { 
          bsonType: "string",
          pattern: "^user_[a-zA-Z0-9]{10}$"
        },
        category: { 
          enum: ["placements", "events", "results"]
        },
        type: { bsonType: "string" },
        title: { bsonType: "string" },
        description: { bsonType: "string" },
        content: { bsonType: "string" },
        priority: { 
          enum: ["low", "medium", "high"]
        },
        isRead: { bsonType: "bool" },
        readAt: { 
          bsonType: ["date", "null"]
        },
        createdAt: { bsonType: "date" },
        updatedAt: { bsonType: "date" },
        expiresAt: { 
          bsonType: ["date", "null"]
        },
        metadata: { bsonType: "object" },
        actionUrl: { 
          bsonType: ["string", "null"]
        },
        attachments: {
          bsonType: "array",
          items: {
            bsonType: "object",
            properties: {
              id: { bsonType: "string" },
              name: { bsonType: "string" },
              url: { bsonType: "string" },
              type: { bsonType: "string" },
              size: { bsonType: "int" }
            }
          }
        },
        deleted: { bsonType: "bool" }
      }
    }
  }
});
```

#### 2. notification_preferences Collection
User notification settings and preferences

```javascript
db.createCollection("notification_preferences", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["userId", "preferences"],
      properties: {
        _id: { bsonType: "objectId" },
        userId: { bsonType: "string" },
        preferences: {
          bsonType: "object",
          properties: {
            categories: {
              bsonType: "object",
              properties: {
                placements: {
                  bsonType: "object",
                  properties: {
                    enabled: { bsonType: "bool" },
                    types: { 
                      bsonType: "array",
                      items: { bsonType: "string" }
                    }
                  }
                },
                events: {
                  bsonType: "object",
                  properties: {
                    enabled: { bsonType: "bool" },
                    types: { 
                      bsonType: "array",
                      items: { bsonType: "string" }
                    }
                  }
                },
                results: {
                  bsonType: "object",
                  properties: {
                    enabled: { bsonType: "bool" },
                    types: { 
                      bsonType: "array",
                      items: { bsonType: "string" }
                    }
                  }
                }
              }
            },
            channels: {
              bsonType: "object",
              properties: {
                inApp: { bsonType: "bool" },
                email: { bsonType: "bool" },
                push: { bsonType: "bool" },
                sms: { bsonType: "bool" }
              }
            },
            frequency: { enum: ["immediate", "hourly", "daily", "weekly"] },
            quietHours: {
              bsonType: "object",
              properties: {
                enabled: { bsonType: "bool" },
                startTime: { bsonType: "string" },
                endTime: { bsonType: "string" }
              }
            }
          }
        },
        createdAt: { bsonType: "date" },
        updatedAt: { bsonType: "date" }
      }
    }
  }
});
```

#### 3. notification_read_status Collection (Optional - for audit trail)
Maintains read event history for analytics

```javascript
db.createCollection("notification_read_status", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["notificationId", "userId", "readAt"],
      properties: {
        _id: { bsonType: "objectId" },
        notificationId: { bsonType: "string" },
        userId: { bsonType: "string" },
        readAt: { bsonType: "date" },
        deviceInfo: {
          bsonType: "object",
          properties: {
            userAgent: { bsonType: "string" },
            ipAddress: { bsonType: "string" }
          }
        }
      }
    }
  }
});
```

#### 4. notification_analytics Collection
Aggregated statistics and metrics

```javascript
db.createCollection("notification_analytics", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["date", "metrics"],
      properties: {
        _id: { bsonType: "objectId" },
        date: { bsonType: "date" },
        metrics: {
          bsonType: "object",
          properties: {
            totalNotificationsSent: { bsonType: "int" },
            totalNotificationsRead: { bsonType: "int" },
            readRate: { bsonType: "double" },
            byCategory: { bsonType: "object" },
            byPriority: { bsonType: "object" },
            avgResponseTime: { bsonType: "double" }
          }
        }
      }
    }
  }
});
```

---

## Indexes for Performance

### Critical Indexes for notifications Collection

```javascript
// Index 1: Query by userId and createdAt (most common query)
db.notifications.createIndex(
  { userId: 1, createdAt: -1 },
  { name: "idx_userId_createdAt", background: true }
);

// Index 2: Query by userId, isRead, and category
db.notifications.createIndex(
  { userId: 1, isRead: 1, category: 1 },
  { name: "idx_userId_isRead_category", background: true }
);

// Index 3: Query by category and createdAt (for bulk operations)
db.notifications.createIndex(
  { category: 1, createdAt: -1 },
  { name: "idx_category_createdAt", background: true }
);

// Index 4: Unique index on id field
db.notifications.createIndex(
  { id: 1 },
  { name: "idx_id_unique", unique: true }
);

// Index 5: TTL Index for automatic expiration (auto-delete expired notifications after 2592000 seconds = 30 days)
db.notifications.createIndex(
  { expiresAt: 1 },
  { name: "idx_expiresAt_ttl", expireAfterSeconds: 0 }
);

// Index 6: Partial index for unread notifications (optimize unread query)
db.notifications.createIndex(
  { userId: 1, createdAt: -1 },
  { 
    name: "idx_unread_notifications",
    partialFilterExpression: { isRead: false }
  }
);

// Index 7: Text search index for title and description
db.notifications.createIndex(
  { title: "text", description: "text", content: "text" },
  { name: "idx_text_search" }
);
```

### Indexes for notification_preferences Collection

```javascript
// Index for userId (unique user preferences)
db.notification_preferences.createIndex(
  { userId: 1 },
  { name: "idx_userId_unique", unique: true }
);
```

### Indexes for notification_read_status Collection

```javascript
// Index for query by notificationId and userId
db.notification_read_status.createIndex(
  { notificationId: 1, userId: 1 },
  { name: "idx_notif_user" }
);

// Index for query by userId and readAt
db.notification_read_status.createIndex(
  { userId: 1, readAt: -1 },
  { name: "idx_userId_readAt" }
);
```

---

## Scalability Challenges & Solutions

### Challenge 1: Data Volume Growth
**Problem:** As user base grows, number of notifications increases exponentially (millions per day)

**Solutions:**
1. **Sharding Strategy**
   - Shard key: `userId` (ensures even distribution and localizes user queries)
   - Alternative: Composite shard key `{userId: 1, createdAt: 1}` for balanced distribution

2. **Archive Strategy**
   - Move notifications older than 90 days to archive collection
   - Keep recent data in hot storage
   - Use time-series collections for aggregated metrics

3. **Compression**
   - Compress old notification attachments
   - Store metadata separately from content

### Challenge 2: Query Performance Degradation
**Problem:** Complex queries on large datasets become slow

**Solutions:**
1. **Denormalization**
   - Store read count and unread count in user profile document
   - Update counts asynchronously

2. **Caching Layer**
   - Implement Redis cache for:
     - User notification stats (10-minute TTL)
     - Preferences (24-hour TTL)
     - Recent notifications (1-minute TTL)

3. **Read Replicas**
   - Use MongoDB replica sets
   - Route read queries to secondary replicas
   - Keep writes on primary node

### Challenge 3: Write Throughput Limitations
**Problem:** High volume of concurrent notification writes

**Solutions:**
1. **Write Scaling**
   - Use sharded cluster with multiple shards
   - Batch write operations where possible
   - Implement connection pooling

2. **Queue System**
   - Use message queue (RabbitMQ/Kafka) for notification generation
   - Workers process queue and write to MongoDB

3. **Bulk Operations**
   - Use `insertMany()` with ordered: false for parallel inserts
   - Configure appropriate write concern (balancing consistency vs. speed)

### Challenge 4: Real-time Notification Delivery
**Problem:** Broadcasting notifications to thousands of concurrent users

**Solutions:**
1. **WebSocket Scaling**
   - Use load balancer (HAProxy/NGINX) with sticky sessions
   - Deploy multiple WebSocket servers
   - Use Redis Pub/Sub for cross-server communication

2. **Event Bus Architecture**
   - Publish notification events to Kafka/RabbitMQ
   - Multiple subscribers handle delivery to different channels
   - Allows decoupling of notification generation from delivery

3. **Batch Notifications**
   - For low-priority notifications, batch and deliver at intervals
   - Respect user's quiet hours and preferences

### Challenge 5: Storage Cost & Optimization
**Problem:** Rapidly growing storage costs

**Solutions:**
1. **Tiered Storage**
   - Hot tier: Last 30 days (frequently accessed)
   - Warm tier: 30-90 days (occasionally accessed)
   - Cold tier: >90 days (archived, rarely accessed)

2. **Compression**
   - Enable MongoDB compression on connection
   - Archive old collections with compression

3. **Cleanup Policies**
   - Automatically delete read notifications older than 60 days
   - Keep unread notifications indefinitely or with longer TTL
   - Use TTL indexes for automatic cleanup

### Challenge 6: Consistency & Durability
**Problem:** Ensuring no notification loss

**Solutions:**
1. **Write Concern**
   ```javascript
   // Use majority write concern for important operations
   db.notifications.insertOne(
     { /* notification data */ },
     { writeConcern: { w: "majority", j: true } }
   );
   ```

2. **Replica Sets**
   - Configure 3-node replica set minimum
   - Enables automatic failover
   - Provides read redundancy

3. **Backup Strategy**
   - Daily automated backups
   - Point-in-time recovery capability
   - Store backups in geographically distributed locations

---

## Query Examples

### Query 1: Get All Unread Notifications for a User
```javascript
db.notifications.find(
  {
    userId: "user_123456789",
    isRead: false,
    deleted: false
  },
  {
    projection: {
      id: 1,
      title: 1,
      category: 1,
      priority: 1,
      createdAt: 1,
      metadata: 1
    }
  }
)
.sort({ createdAt: -1 })
.limit(20)
.skip(0);
```

### Query 2: Get Notifications by Category and Status
```javascript
db.notifications.find(
  {
    userId: "user_123456789",
    category: "placements",
    isRead: false,
    deleted: false
  }
)
.sort({ priority: -1, createdAt: -1 })
.limit(20);
```

### Query 3: Mark Multiple Notifications as Read
```javascript
db.notifications.updateMany(
  {
    userId: "user_123456789",
    category: "placements",
    isRead: false,
    deleted: false
  },
  {
    $set: {
      isRead: true,
      readAt: new Date()
    }
  }
);
```

### Query 4: Get Notification Statistics
```javascript
db.notifications.aggregate([
  {
    $match: {
      userId: "user_123456789",
      deleted: false,
      createdAt: {
        $gte: new Date(new Date().setDate(new Date().getDate() - 7))
      }
    }
  },
  {
    $group: {
      _id: "$category",
      total: { $sum: 1 },
      unread: {
        $sum: { $cond: ["$isRead", 0, 1] }
      },
      read: {
        $sum: { $cond: ["$isRead", 1, 0] }
      },
      highPriority: {
        $sum: { $cond: [{ $eq: ["$priority", "high"] }, 1, 0] }
      }
    }
  },
  {
    $sort: { _id: 1 }
  }
]);
```

### Query 5: Get User Notification Preferences
```javascript
db.notification_preferences.findOne(
  { userId: "user_123456789" },
  {
    projection: {
      userId: 1,
      preferences: 1,
      updatedAt: 1
    }
  }
);
```

### Query 6: Update Notification Preferences
```javascript
db.notification_preferences.updateOne(
  { userId: "user_123456789" },
  {
    $set: {
      "preferences.channels.email": false,
      "preferences.channels.push": true,
      "preferences.frequency": "daily",
      updatedAt: new Date()
    }
  },
  { upsert: true }
);
```

### Query 7: Get Read Statistics by Category
```javascript
db.notifications.aggregate([
  {
    $match: {
      userId: "user_123456789",
      deleted: false
    }
  },
  {
    $group: {
      _id: "$category",
      total: { $sum: 1 },
      read: {
        $sum: { $cond: ["$isRead", 1, 0] }
      },
      unread: {
        $sum: { $cond: ["$isRead", 0, 1] }
      }
    }
  },
  {
    $project: {
      _id: 1,
      total: 1,
      read: 1,
      unread: 1,
      readPercentage: {
        $multiply: [
          { $divide: ["$read", "$total"] },
          100
        ]
      }
    }
  }
]);
```

### Query 8: Get Notifications by Priority and Priority Count
```javascript
db.notifications.aggregate([
  {
    $match: {
      userId: "user_123456789",
      isRead: false,
      deleted: false
    }
  },
  {
    $group: {
      _id: "$priority",
      count: { $sum: 1 },
      notifications: {
        $push: {
          id: "$id",
          title: "$title",
          category: "$category",
          createdAt: "$createdAt"
        }
      }
    }
  },
  {
    $sort: {
      _id: -1
    }
  }
]);
```

### Query 9: Delete Old Notifications (Archive Strategy)
```javascript
db.notifications.deleteMany(
  {
    userId: "user_123456789",
    isRead: true,
    createdAt: {
      $lt: new Date(new Date().setDate(new Date().getDate() - 60))
    }
  }
);
```

### Query 10: Get Trending Notifications (Last 7 Days)
```javascript
db.notifications.aggregate([
  {
    $match: {
      deleted: false,
      createdAt: {
        $gte: new Date(new Date().setDate(new Date().getDate() - 7))
      }
    }
  },
  {
    $group: {
      _id: {
        category: "$category",
        type: "$type"
      },
      count: { $sum: 1 },
      readCount: {
        $sum: { $cond: ["$isRead", 1, 0] }
      }
    }
  },
  {
    $sort: { count: -1 }
  },
  {
    $limit: 10
  }
]);
```

### Query 11: Get Notification Analytics by Date
```javascript
db.notifications.aggregate([
  {
    $match: {
      deleted: false,
      createdAt: {
        $gte: new Date(new Date().setDate(new Date().getDate() - 30))
      }
    }
  },
  {
    $group: {
      _id: {
        $dateToString: {
          format: "%Y-%m-%d",
          date: "$createdAt"
        }
      },
      totalSent: { $sum: 1 },
      totalRead: {
        $sum: { $cond: ["$isRead", 1, 0] }
      },
      highPriority: {
        $sum: { $cond: [{ $eq: ["$priority", "high"] }, 1, 0] }
      }
    }
  },
  {
    $project: {
      _id: 1,
      totalSent: 1,
      totalRead: 1,
      readRate: {
        $multiply: [
          { $divide: ["$totalRead", "$totalSent"] },
          100
        ]
      },
      highPriority: 1
    }
  },
  {
    $sort: { _id: -1 }
  }
]);
```

### Query 12: Get User-Specific Analytics
```javascript
db.notifications.aggregate([
  {
    $match: {
      userId: "user_123456789",
      deleted: false
    }
  },
  {
    $facet: {
      byCategory: [
        {
          $group: {
            _id: "$category",
            count: { $sum: 1 },
            unread: {
              $sum: { $cond: ["$isRead", 0, 1] }
            }
          }
        }
      ],
      byPriority: [
        {
          $group: {
            _id: "$priority",
            count: { $sum: 1 }
          }
        }
      ],
      avgResponseTime: [
        {
          $match: { readAt: { $exists: true } }
        },
        {
          $project: {
            responseTime: {
              $subtract: ["$readAt", "$createdAt"]
            }
          }
        },
        {
          $group: {
            _id: null,
            avgMs: { $avg: "$responseTime" }
          }
        }
      ]
    }
  }
]);
```

---

## Performance Optimization Strategies

### 1. Connection Pooling
```javascript
const client = new MongoClient(uri, {
  maxPoolSize: 50,
  minPoolSize: 10,
  maxIdleTimeMS: 45000,
  waitQueueTimeoutMS: 10000
});
```

### 2. Read Preference
```javascript
// Route read queries to secondary replicas
db.notifications.find(
  { userId: "user_123456789" }
).readPreference('secondary');
```

### 3. Write Concern Configuration
```javascript
// Use majority write concern for critical operations
const opts = { writeConcern: { w: "majority", j: true, wtimeout: 5000 } };
db.notifications.insertOne(doc, opts);
```

---

## Monitoring & Alerts

### Key Metrics to Monitor:
- Query execution time (p95, p99)
- Write throughput (operations/sec)
- Storage growth rate
- Replica set replication lag
- Connection pool utilization
- CPU and Memory usage
- Unread notification count per user
- Notification delivery latency

---

## Disaster Recovery & Backup Strategy

1. **Automated Daily Backups**
   - Full backup: Daily at 2 AM UTC
   - Incremental backups: Every 6 hours
   - Retention: 30 days

2. **Point-in-Time Recovery**
   - Enable oplog retention: 7 days
   - Allows recovery to any point within 7 days

3. **Geo-Redundant Backups**
   - Store backups in multiple regions
   - Ensure data availability during regional outages

4. **Recovery Testing**
   - Monthly recovery drills
   - Test recovery time objectives (RTO)
   - Document recovery procedures

---

## Cost Optimization

1. **Storage Tiering**
   - Use MongoDB Atlas auto-scaling
   - Archive old data to cheaper storage
   - Compress historical data

2. **Index Optimization**
   - Remove unused indexes
   - Monitor index usage statistics
   - Use partial indexes for filtered queries

3. **Connection Management**
   - Reuse connections via pooling
   - Close idle connections
   - Limit maximum connections per application

4. **Capacity Planning**
   - Monitor growth trends
   - Adjust resource allocation proactively
   - Use auto-scaling features where available

---

This comprehensive design provides a scalable, efficient, and reliable notification system capable of handling millions of notifications daily while maintaining real-time delivery and optimal query performance.
