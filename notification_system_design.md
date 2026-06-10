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

# Stage 3: SQL Query Optimization & Performance Analysis

## Problem Analysis: Slow Query

### Original Query
```sql
SELECT * FROM notifications
WHERE studentID = 1042 AND isRead = false
ORDER BY createdAt ASC;
```

### Query Accuracy Assessment

**Issues with Query Accuracy:**

1. **Semantic Issue with ORDER BY ASC**
   - Current: `ORDER BY createdAt ASC` (oldest first)
   - Expected: `ORDER BY createdAt DESC` (newest first)
   - **Accuracy: INCORRECT** - Users typically want most recent notifications first
   - Fetching oldest first defeats the purpose of real-time notifications

2. **Inefficient Column Selection**
   - Using `SELECT *` fetches unnecessary columns
   - For UI display, likely only need: `id`, `title`, `description`, `category`, `priority`, `createdAt`
   - Adds unnecessary I/O and network transfer overhead

3. **Missing Pagination**
   - Query returns ALL unread notifications without LIMIT
   - For 50,000 students with ~100 unread notifications each, this could return 100+ rows
   - No OFFSET for pagination support

---

## Why Is This Query Slow?

### Root Causes:

#### 1. **Full Table Scan (Most Critical)**
   - With 5,000,000 notifications in table
   - Without indexes on `studentID` and `isRead`, database performs **full table scan**
   - Examines all 5M rows to find matching records
   - Complexity: **O(n)** = ~5 million comparisons per query

#### 2. **No Composite Index**
   - Query filters on TWO columns: `studentID` AND `isRead`
   - Without composite index (studentID, isRead), optimizer cannot efficiently jump to relevant records
   - Even if individual indexes exist, they may not be used optimally

#### 3. **High Cardinality + High Selectivity**
   - `studentID` = 1042: High cardinality (many distinct values), highly selective
   - `isRead` = false: Low cardinality (only 2 values), moderate selectivity
   - Combined: Very selective query, but no index to leverage this

#### 4. **Sort Operation on Unsorted Data**
   - ORDER BY createdAt requires sorting all matched records
   - Without index on createdAt in result set order, database must:
     - Fetch all matching rows
     - Load into memory
     - Perform sort operation
   - For large result sets, may trigger disk-based sorting (very slow)

#### 5. **Concurrent Load**
   - 50,000 students × page loads throughout day = high query volume
   - Each full table scan locks resources
   - Query queue builds up, causing cascading slowdown

#### 6. **SELECT * Overhead**
   - Fetches unnecessary columns (e.g., metadata, attachments URLs)
   - Increases memory usage and I/O bandwidth
   - Network transfer time increases for remote clients

---

## Optimized Query & Computational Cost Analysis

### Optimized Query (Corrected)

```sql
SELECT 
  id,
  studentID,
  title,
  description,
  category,
  notificationType,
  priority,
  createdAt,
  metadata
FROM notifications
WHERE studentID = 1042 
  AND isRead = false
  AND deleted = false
ORDER BY createdAt DESC
LIMIT 20
OFFSET 0;
```

### Required Index

```sql
-- Create composite index for optimal performance
CREATE INDEX idx_studentID_isRead_createdAt 
ON notifications(studentID, isRead, createdAt DESC);

-- Optional: Add covering index (includes all SELECT columns)
CREATE INDEX idx_studentID_isRead_covering
ON notifications(studentID, isRead, createdAt DESC)
INCLUDE (title, description, category, notificationType, priority, metadata);
```

### Performance Improvements

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| **Query Time** | ~2000-5000ms | ~5-50ms | **99.7% faster** |
| **Rows Examined** | 5,000,000 | ~100-500 | **99.99% reduction** |
| **Disk I/O Operations** | 50,000+ random seeks | 5-10 sequential seeks | **99.98% reduction** |
| **Memory Usage** | 500MB+ (sorting large result) | 5-10MB (20 rows) | **98% reduction** |
| **CPU Usage** | 40-60% (full scan + sort) | <5% (index lookup + sort 20 rows) | **92% reduction** |
| **Lock Time** | 500-2000ms | <1ms | **99% reduction** |

### Computational Cost Analysis

#### **Before Optimization:**
- **Time Complexity:** O(n log n) where n = 5,000,000
  - Full table scan: O(n) = 5,000,000 comparisons
  - Sort operation: O(n log n) = ~130 million operations
  - Total: ~135 million operations

- **Space Complexity:** O(n) for sorting
  - All 5M rows loaded into memory/temp disk
  - Estimated: 500MB+ RAM or disk space

- **I/O Cost:** ~50,000 disk pages read (assuming 100-byte row size)
  - Approximately 50,000-100,000ms I/O time at 1000 IOPS

#### **After Optimization:**
- **Time Complexity:** O(k log k) where k = LIMIT (20 rows)
  - Index range scan: O(log n) = ~23 comparisons
  - Retrieve 20 rows: O(k) = 20 operations
  - Sort 20 rows: O(k log k) = ~86 operations
  - Total: ~129 operations

- **Space Complexity:** O(k) = O(20)
  - Only result set + sort buffer
  - Estimated: <1MB RAM

- **I/O Cost:** ~5-10 disk pages
  - Approximately 5-50ms I/O time

**Cost Reduction Factor:** ~100-1000x faster

---

## Index Strategy Analysis

### Question: "Adding indexes on every column to be safe"

**Answer: NO - This is counterproductive**

### Why Adding Indexes to Every Column is Ineffective:

#### **1. Maintenance Overhead**
```
For each INSERT or UPDATE operation:
- Write to data row
- Update 10+ indexes (if 10+ columns indexed)
- Sync log entries
- Example: 1 INSERT becomes 11 write operations
Result: Write throughput drops 10x
```

- For high-volume systems (50K students, 5M notifications growing daily):
  - Writing 100 new notifications → 1,100 index updates instead of 100
  - Heavy write amplification

#### **2. Storage Bloat**
- Indexes consume disk space (often 50-70% of data size)
- 10 indexes on 5M rows:
  - Data size: ~500MB
  - Index size: ~5GB (10 × 500MB)
  - Storage cost multiplied 10x

#### **3. Query Planner Confusion**
```sql
-- With 10+ indexes, optimizer must evaluate all possible paths:
-- - Index on studentID alone?
-- - Index on isRead alone?
-- - Index on studentID + isRead?
-- - Index on studentID + isRead + createdAt?
-- Decision tree explodes, planning time increases
```

- More indexes = slower query planning
- Suboptimal plan selection more likely
- Can result in worse performance than before

#### **4. Memory Pressure**
- Index cache (buffer pool) needs to fit working set
- 5GB indexes compete with data cache
- More cache misses = more disk I/O
- Performance degrades

#### **5. Poor Index Selectivity**
```sql
-- Index on low-cardinality column (bad)
CREATE INDEX idx_isRead ON notifications(isRead);
-- Only 2 values: true/false
-- Index says: "500K true entries, 4.5M false entries"
-- Still need to scan 4.5M rows!
-- Index doesn't help when selectivity is low
```

### Effective Index Strategy Instead:

#### **1. Create Composite Indexes (Selective Columns Only)**
```sql
-- For this specific query need
CREATE INDEX idx_studentID_isRead_createdAt 
ON notifications(studentID, isRead, createdAt DESC);

-- For other common queries
CREATE INDEX idx_category_createdAt 
ON notifications(category, createdAt DESC);

-- For analytics
CREATE INDEX idx_studentID_category 
ON notifications(studentID, category, isRead);

-- Total: 3 indexes instead of 20+
```

#### **2. Use Covering Indexes**
```sql
-- Includes all columns needed (no table lookup required)
CREATE INDEX idx_notifications_covering
ON notifications(studentID, isRead, createdAt DESC)
INCLUDE (title, description, category, priority, notificationType);
```

#### **3. Partial Indexes (if DB supports)**
```sql
-- Only index unread notifications (saves 50% space)
CREATE INDEX idx_unread_notifications 
ON notifications(studentID, createdAt DESC)
WHERE isRead = false;
```

#### **4. Regular Index Maintenance**
```sql
-- Identify unused indexes (query at least once per month)
SELECT * FROM sys.dm_db_missing_indexes;

-- Rebuild fragmented indexes
ALTER INDEX idx_studentID_isRead_createdAt 
ON notifications REBUILD;

-- Monitor index statistics
DBCC SHOW_STATISTICS(notifications, idx_studentID_isRead_createdAt);
```

### Index Comparison Table

| Strategy | Indexes | Read Speed | Write Speed | Storage | Maintenance |
|----------|---------|-----------|-----------|---------|------------|
| **No Index** | 0 | ⭐ Very Slow (5000ms) | ⭐⭐⭐⭐⭐ Fast | ⭐⭐⭐⭐⭐ Minimal | ⭐⭐⭐⭐⭐ Low |
| **Every Column** | 15+ | ⭐⭐ Moderate (could backfire) | ⭐ Very Slow | ⭐ Bloated | ⭐ Nightmare |
| **Smart Composite** | 3-5 | ⭐⭐⭐⭐⭐ Very Fast (10-50ms) | ⭐⭐⭐⭐ Good | ⭐⭐⭐ Reasonable | ⭐⭐⭐⭐ Manageable |

---

## Query 2: Find Students with Placement Notifications in Last 7 Days

### Schema Context
```sql
-- notifications table has:
-- - studentID (INT)
-- - notificationType (ENUM: 'Event', 'Result', 'Placement')
-- - createdAt (TIMESTAMP)
-- - isRead (BOOLEAN)
-- - deleted (BOOLEAN)
```

### Optimized Query

```sql
-- Query 1: Basic - Get student IDs with placement notifications
SELECT DISTINCT 
  studentID
FROM notifications
WHERE notificationType = 'Placement'
  AND createdAt >= DATE_SUB(NOW(), INTERVAL 7 DAY)
  AND deleted = false
ORDER BY studentID ASC;
```

### Alternative: With Additional Details

```sql
-- Query 2: With notification details
SELECT 
  studentID,
  COUNT(*) as notificationCount,
  MAX(createdAt) as latestNotificationTime,
  GROUP_CONCAT(DISTINCT id ORDER BY createdAt DESC) as notificationIDs
FROM notifications
WHERE notificationType = 'Placement'
  AND createdAt >= DATE_SUB(NOW(), INTERVAL 7 DAY)
  AND deleted = false
GROUP BY studentID
ORDER BY notificationCount DESC, latestNotificationTime DESC;
```

### Query 3: With Student Count and Status

```sql
-- Get placement notification statistics
SELECT 
  'Total Students with Placements' as metric,
  COUNT(DISTINCT studentID) as value
FROM notifications
WHERE notificationType = 'Placement'
  AND createdAt >= DATE_SUB(NOW(), INTERVAL 7 DAY)
  AND deleted = false

UNION ALL

SELECT 
  'Unread Placement Notifications',
  COUNT(DISTINCT studentID)
FROM notifications
WHERE notificationType = 'Placement'
  AND isRead = false
  AND createdAt >= DATE_SUB(NOW(), INTERVAL 7 DAY)
  AND deleted = false

UNION ALL

SELECT 
  'Total Placement Notifications',
  COUNT(*) as value
FROM notifications
WHERE notificationType = 'Placement'
  AND createdAt >= DATE_SUB(NOW(), INTERVAL 7 DAY)
  AND deleted = false;
```

### Required Indexes for These Queries

```sql
-- Primary index for all placement queries
CREATE INDEX idx_notificationType_createdAt 
ON notifications(notificationType, createdAt DESC, deleted);

-- Composite index for faster aggregation
CREATE INDEX idx_notificationType_studentID_createdAt 
ON notifications(notificationType, studentID, createdAt DESC)
WHERE deleted = false;

-- Covering index to avoid table lookup
CREATE INDEX idx_placement_notifications_covering
ON notifications(notificationType, createdAt DESC, deleted)
INCLUDE (studentID, id, isRead);
```

### Performance Metrics

| Metric | Without Index | With Index | Improvement |
|--------|--------------|-----------|------------|
| **Query Time** | 1500-3000ms | 30-100ms | **99% faster** |
| **Rows Scanned** | 5,000,000 | ~5,000-50,000 | **99.5% reduction** |
| **Memory** | 200MB | 5MB | **97.5% reduction** |

---

## Summary: Best Practices for SQL Query Optimization

| Practice | Benefit | Effort |
|----------|---------|--------|
| **Create composite indexes on frequently filtered columns** | 100-1000x speedup | Medium |
| **Use DESC in indexes for ORDER BY DESC queries** | Eliminates sort operation | Low |
| **Select specific columns instead of SELECT *** | 50-80% I/O reduction | Low |
| **Add LIMIT and OFFSET for pagination** | Constant response time | Low |
| **Use covering indexes** | No table lookup needed | Medium |
| **Monitor and remove unused indexes** | Reduced write overhead | Low |
| **Implement partial indexes for filtered data** | 50% storage savings | Medium |
| **Update statistics regularly** | Better query plans | Low |
| **Avoid indexes on low-cardinality columns** | Faster inserts | Low |
| **Use EXPLAIN ANALYZE to validate plans** | Identify bottlenecks | Low |

---

# Stage 4: Caching Strategy & Performance Optimization

## Problem Statement

**Issue:** Notifications fetched on every page load for all 50,000 students → Database overwhelmed → Poor user experience

**Current Situation:**
- 50,000 students
- Estimated 20-40 page loads per student per day
- 1-2 million notification queries per day
- Database connection pool exhausted
- Average response time: 2-5 seconds
- Server timeout errors occurring

---

## Solution 1: In-Memory Caching (Redis/Memcached)

### Implementation Strategy

```
Student Page Load Flow:
1. Student loads page
2. App checks Redis cache for notifications
3. If cache HIT: Return cached data (1-5ms)
4. If cache MISS: Query database + cache result
```

### Cache Architecture

```javascript
// Cache key structure
const cacheKey = `notifications:student:${studentID}:unread`;
const cacheValue = {
  notifications: [...],
  statistics: {...},
  lastUpdated: timestamp
};

// TTL: 2 minutes (balance between freshness and cache hits)
const ttl = 120;

// Cache hit rate target: 80-90%
```

### Implementation Example (Node.js + Redis)

```javascript
// With Logging Middleware
async function getNotifications(studentID, limit = 20) {
  logger.info('Fetching notifications', { 
    studentID, 
    limit,
    timestamp: new Date().toISOString()
  });

  try {
    // Step 1: Check cache
    const cacheKey = `notifications:${studentID}:unread`;
    const cachedData = await redis.get(cacheKey);
    
    if (cachedData) {
      logger.info('Cache HIT', { 
        studentID, 
        source: 'redis',
        cacheAge: 'recent'
      });
      return JSON.parse(cachedData);
    }

    logger.warn('Cache MISS', { 
      studentID, 
      database_query: 'executing'
    });

    // Step 2: Query database (with optimized query from Stage 3)
    const notifications = await db.query(`
      SELECT id, title, description, category, priority, createdAt
      FROM notifications
      WHERE studentID = ?
        AND isRead = false
        AND deleted = false
      ORDER BY createdAt DESC
      LIMIT ?
    `, [studentID, limit]);

    // Step 3: Cache result
    await redis.setex(cacheKey, 120, JSON.stringify(notifications));
    
    logger.info('Cached notifications', { 
      studentID,
      count: notifications.length,
      ttl: 120
    });

    return notifications;

  } catch (error) {
    logger.error('Error fetching notifications', {
      studentID,
      error: error.message,
      stack: error.stack
    });
    throw error;
  }
}
```

### Advantages
- **Response Time:** 1-5ms (vs 2-5s database query)
- **Database Load:** Reduced by 80-90% (with 80-90% hit rate)
- **Scalability:** Horizontal scaling with Redis cluster
- **User Experience:** Instant page load

### Disadvantages
- **Stale Data:** 2-minute delay before fresh updates
- **Operational Complexity:** Need to manage Redis cluster
- **Memory Cost:** ~100MB for 50K students × ~2KB avg per student
- **Cache Invalidation:** Need event-driven updates
- **Consistency:** Eventually consistent, not immediately consistent

### Tradeoffs
- **Freshness vs Performance:** 2-minute cache TTL is a compromise
  - Could reduce to 30 seconds for more freshness (more DB hits)
  - Could increase to 5 minutes for better performance (more stale data)

---

## Solution 2: Cache Invalidation Strategy (Event-Driven)

### Push Update Architecture

When new notification created:
```
1. New notification written to database
2. Event published to message queue (Kafka/RabbitMQ)
3. Notification service consumes event
4. Cache invalidated for affected student(s)
5. WebSocket pushes real-time update to connected clients
```

### Implementation

```javascript
// When creating new notification
async function createNotification(studentID, notificationData) {
  logger.info('Creating notification', { studentID, notificationData });

  try {
    // Insert into database
    const notification = await db.notifications.insert(notificationData);
    
    logger.info('Notification created', { 
      notificationID: notification.id,
      studentID 
    });

    // Publish event for cache invalidation
    await messageQueue.publish('notifications.created', {
      notificationID: notification.id,
      studentID,
      timestamp: new Date().toISOString()
    });

    logger.info('Cache invalidation event published', { 
      studentID,
      event: 'notifications.created'
    });

    // Invalidate cache immediately
    const cacheKey = `notifications:${studentID}:unread`;
    await redis.del(cacheKey);
    
    logger.info('Cache invalidated', { 
      studentID,
      cacheKey 
    });

    // Push to connected WebSocket clients
    await websocketBroadcaster.sendToUser(studentID, {
      type: 'notification_new',
      data: notification
    });

    logger.info('WebSocket notification pushed', { 
      studentID,
      notificationID: notification.id 
    });

  } catch (error) {
    logger.error('Error creating notification', {
      studentID,
      error: error.message,
      stack: error.stack
    });
    throw error;
  }
}
```

### Advantages
- **Immediate Updates:** Cache invalidated on event
- **Consistency:** Fresh data for real-time notifications
- **Reduced Stale Data:** Only 5-100ms stale during processing

### Disadvantages
- **Operational Complexity:** Message queue setup required
- **Consistency Guarantees:** At-least-once delivery possible duplicates
- **Failure Scenarios:** Events could be lost if not properly handled

---

## Solution 3: Lazy Loading & Pagination

### Strategy

Only load notifications user requests (not all 100+ unread notifications at once)

```javascript
// Initial page load: Fetch only first 20
GET /api/v1/notifications?page=1&limit=20

// Response: Include pagination info
{
  "notifications": [...20 items...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 237,
    "hasMore": true
  }
}

// When user scrolls: Load next page
GET /api/v1/notifications?page=2&limit=20
```

### Implementation

```javascript
async function getNotificationsPaginated(studentID, page = 1, limit = 20) {
  logger.info('Fetching paginated notifications', { 
    studentID, 
    page, 
    limit 
  });

  try {
    const offset = (page - 1) * limit;

    // Query with LIMIT and OFFSET
    const notifications = await db.query(`
      SELECT id, title, description, category, priority, createdAt
      FROM notifications
      WHERE studentID = ?
        AND isRead = false
        AND deleted = false
      ORDER BY createdAt DESC
      LIMIT ? OFFSET ?
    `, [studentID, limit, offset]);

    // Get total count (cached separately)
    const countCacheKey = `notifications:${studentID}:count`;
    let total = await redis.get(countCacheKey);
    
    if (!total) {
      const result = await db.query(`
        SELECT COUNT(*) as total
        FROM notifications
        WHERE studentID = ?
          AND isRead = false
          AND deleted = false
      `, [studentID]);
      total = result[0].total;
      await redis.setex(countCacheKey, 300, total);
    }

    logger.info('Paginated results retrieved', { 
      studentID,
      page,
      limit,
      total,
      itemsReturned: notifications.length
    });

    return {
      notifications,
      pagination: {
        page,
        limit,
        total: parseInt(total),
        hasMore: offset + limit < total,
        totalPages: Math.ceil(total / limit)
      }
    };

  } catch (error) {
    logger.error('Error fetching paginated notifications', {
      studentID,
      page,
      error: error.message
    });
    throw error;
  }
}
```

### Advantages
- **Database Load:** Only fetch needed data
- **Memory Usage:** Constant per request (20 items)
- **Network Bandwidth:** Smaller payloads
- **Scalability:** Can handle any number of notifications

### Disadvantages
- **User Experience:** Requires clicking "Load More"
- **Implementation:** Requires frontend changes
- **Data Inconsistency:** Position 21-40 might change if new notifications arrive

---

## Solution 4: Read Replicas for Distributed Load

### Architecture

```
            Load Balancer
                 |
        ┌────────┼────────┐
        |        |        |
    Primary   Replica1  Replica2
    (Write)   (Read)    (Read)

Writes: → Primary only
Reads:  → Distributed across Replica1, Replica2
```

### Implementation

```javascript
async function getNotificationsWithReadReplica(studentID, limit = 20) {
  logger.info('Fetching from read replica', { studentID, limit });

  try {
    // Use read replica connection
    const notifications = await dbReadReplica.query(`
      SELECT id, title, description, category, priority, createdAt
      FROM notifications
      WHERE studentID = ?
        AND isRead = false
        AND deleted = false
      ORDER BY createdAt DESC
      LIMIT ?
    `, [studentID, limit]);

    logger.info('Results from read replica', { 
      studentID,
      itemsReturned: notifications.length,
      connection: 'replica'
    });

    return notifications;

  } catch (error) {
    logger.error('Read replica error, falling back to primary', {
      studentID,
      error: error.message
    });
    
    // Fallback to primary on replica failure
    return await dbPrimary.query(
      'SELECT ... FROM notifications WHERE studentID = ?',
      [studentID]
    );
  }
}
```

### Advantages
- **Load Distribution:** 3x throughput (1 primary + 2 replicas)
- **Availability:** Failover if replica fails
- **Consistency:** Eventually consistent (okay for read operations)
- **Scalability:** Add more replicas as needed

### Disadvantages
- **Replication Lag:** Replica may be 5-100ms behind
- **Cost:** 3x database instance cost
- **Operational Complexity:** Monitoring replication lag
- **Data Inconsistency:** Read different version during replication

---

## Solution 5: Notification Summary Endpoint

### Strategy

Instead of fetching all notifications, fetch only count + high-priority notifications

```
Old Approach:
GET /api/v1/notifications → Returns 100+ items → 500ms

New Approach:
GET /api/v1/notifications/summary → Returns count + 5 high-priority → 50ms
```

### Implementation

```javascript
async function getNotificationsSummary(studentID) {
  logger.info('Fetching notification summary', { studentID });

  try {
    // Step 1: Get counts (cached)
    const countCacheKey = `notifications:${studentID}:summary`;
    let summary = await redis.get(countCacheKey);
    
    if (!summary) {
      const stats = await db.query(`
        SELECT 
          COUNT(*) as total,
          SUM(CASE WHEN isRead = false THEN 1 ELSE 0 END) as unread,
          SUM(CASE WHEN priority = 'high' THEN 1 ELSE 0 END) as highPriority
        FROM notifications
        WHERE studentID = ?
          AND deleted = false
      `, [studentID]);

      summary = stats[0];
      await redis.setex(countCacheKey, 60, JSON.stringify(summary));
    }

    logger.info('Summary counts retrieved', { 
      studentID,
      ...summary 
    });

    // Step 2: Get high-priority notifications
    const highPriorityNotifications = await db.query(`
      SELECT id, title, category, priority, createdAt
      FROM notifications
      WHERE studentID = ?
        AND deleted = false
        AND priority = 'high'
      ORDER BY createdAt DESC
      LIMIT 5
    `, [studentID]);

    logger.info('High-priority notifications retrieved', { 
      studentID,
      count: highPriorityNotifications.length 
    });

    return {
      summary: summary,
      highPriority: highPriorityNotifications
    };

  } catch (error) {
    logger.error('Error fetching summary', {
      studentID,
      error: error.message
    });
    throw error;
  }
}
```

### Advantages
- **Performance:** 50-100ms response time
- **Database Load:** Minimal queries
- **User Experience:** See count immediately, full list on demand
- **Scalability:** Constant time regardless of notification count

### Disadvantages
- **Limited Data:** Only summary + high priority
- **User Experience:** Need to click to see full list
- **Implementation:** Requires API changes

---

## Solution 6: Notification Batching & Pre-computation

### Strategy

Pre-compute and batch notifications instead of computing per request

```
Traditional:
Page Load → DB Query → Compute → Response (500ms per student)

Batched:
Batch Process (every 5 mins) → Pre-compute → Cache
Page Load → Read Cache (5ms)
```

### Implementation

```javascript
// Background batch job (runs every 5 minutes)
async function batchPrecomputeNotifications() {
  logger.info('Starting batch precomputation', { 
    timestamp: new Date().toISOString()
  });

  const batchStartTime = Date.now();
  let processed = 0;

  try {
    // Get all active students
    const students = await db.query(`
      SELECT DISTINCT studentID 
      FROM notifications 
      WHERE createdAt > DATE_SUB(NOW(), INTERVAL 30 DAY)
        AND deleted = false
      GROUP BY studentID
    `);

    logger.info('Processing students for batch', { 
      totalStudents: students.length 
    });

    // Process students in batches of 1000
    for (let i = 0; i < students.length; i += 1000) {
      const batch = students.slice(i, i + 1000);
      
      await Promise.all(batch.map(async (student) => {
        try {
          // Fetch notifications for this student
          const notifications = await db.query(`
            SELECT id, title, description, category, priority, createdAt
            FROM notifications
            WHERE studentID = ?
              AND isRead = false
              AND deleted = false
            ORDER BY createdAt DESC
            LIMIT 20
          `, [student.studentID]);

          // Cache result
          const cacheKey = `notifications:${student.studentID}:unread`;
          await redis.setex(
            cacheKey, 
            300,  // 5 minute TTL
            JSON.stringify(notifications)
          );

          logger.info('Student notifications cached', { 
            studentID: student.studentID,
            notificationCount: notifications.length 
          });

          processed++;
        } catch (error) {
          logger.error('Error processing student in batch', {
            studentID: student.studentID,
            error: error.message
          });
        }
      }));
    }

    const batchDuration = Date.now() - batchStartTime;
    logger.info('Batch precomputation completed', { 
      processed,
      durationMs: batchDuration,
      avgTimePerStudent: batchDuration / processed
    });

  } catch (error) {
    logger.error('Batch precomputation failed', {
      error: error.message,
      stack: error.stack
    });
  }
}

// Schedule to run every 5 minutes
schedule.scheduleJob('*/5 * * * *', batchPrecomputeNotifications);
```

### Advantages
- **Performance:** 5ms page load time (cached data)
- **Database Load:** Single batch job instead of per-request
- **Consistent Load:** Predictable resource usage
- **Scalability:** 1 batch job scales to any number of users

### Disadvantages
- **Stale Data:** Up to 5 minutes old
- **Memory Usage:** Cache for all 50K students
- **Operational Complexity:** Need background job infrastructure
- **Complexity:** More moving parts to maintain

---

## Solution 7: GraphQL with DataLoader (Batch Query Optimization)

### Strategy

Batch multiple requests into single query to reduce N+1 problem

```
REST API (N+1 problem):
1. GET /notifications/student/1 → DB Query
2. GET /notifications/student/2 → DB Query
3. GET /notifications/student/3 → DB Query
= 3 separate queries

GraphQL with DataLoader:
1. Query students with notifications
= Batched into 1-2 queries
```

### Implementation

```javascript
const DataLoader = require('dataloader');
const logger = require('./logging-middleware');

// DataLoader to batch fetch notifications
const notificationLoader = new DataLoader(
  async (studentIDs) => {
    logger.info('DataLoader batch fetch', { 
      studentIDs, 
      batchSize: studentIDs.length 
    });

    try {
      // Fetch all notifications in one query
      const query = `
        SELECT id, studentID, title, description, category, priority, createdAt
        FROM notifications
        WHERE studentID IN (${studentIDs.map(() => '?').join(',')})
          AND isRead = false
          AND deleted = false
        ORDER BY studentID, createdAt DESC
      `;

      const notifications = await db.query(query, studentIDs);

      // Group by studentID
      const grouped = {};
      studentIDs.forEach(id => grouped[id] = []);
      
      notifications.forEach(notif => {
        grouped[notif.studentID].push(notif);
      });

      logger.info('DataLoader batch completed', { 
        batchSize: studentIDs.length,
        totalNotifications: notifications.length
      });

      return studentIDs.map(id => grouped[id] || []);
    } catch (error) {
      logger.error('DataLoader batch error', {
        studentIDs,
        error: error.message
      });
      throw error;
    }
  },
  { 
    batch: true, 
    cache: true, 
    batchScheduleFn: (cb) => process.nextTick(cb)
  }
);

// GraphQL Resolver
const resolvers = {
  Query: {
    student: async (parent, args) => {
      logger.info('Fetching student', { studentID: args.id });
      return await db.students.findById(args.id);
    }
  },
  Student: {
    notifications: async (parent) => {
      logger.info('Resolving student notifications', { 
        studentID: parent.id 
      });
      return notificationLoader.load(parent.id);
    }
  }
};
```

### GraphQL Query

```graphql
{
  students(limit: 10) {
    id
    name
    notifications(first: 5) {
      id
      title
      category
      createdAt
    }
  }
}
```

### Advantages
- **Batching:** Reduces N+1 queries to 1-2 queries
- **Performance:** 200ms → 20ms for 10 students
- **Flexibility:** Client specifies exact fields needed
- **Optimization:** Caching at DataLoader level

### Disadvantages
- **Learning Curve:** GraphQL complexity
- **Implementation:** Requires rewrite of API
- **Debugging:** Harder to debug than REST
- **Over-fetching:** Still possible with bad queries

---

## Strategy Comparison Matrix

| Strategy | Response Time | DB Load Reduction | Implementation | Freshness | Cost |
|----------|---------------|------------------|---------------|-----------|----|
| **Redis Caching** | ⭐⭐⭐⭐⭐ (5ms) | 80-90% | Medium | ⭐⭐⭐ (2 min) | 💰💰 |
| **Event-Driven Invalidation** | ⭐⭐⭐⭐⭐ (5ms) | 80-90% | High | ⭐⭐⭐⭐⭐ (real-time) | 💰💰💰 |
| **Lazy Loading** | ⭐⭐⭐⭐ (50ms) | 70-80% | Low | ⭐⭐⭐⭐ (few sec) | 💰 |
| **Read Replicas** | ⭐⭐⭐⭐ (100ms) | 65-75% | High | ⭐⭐⭐⭐ (100ms) | 💰💰💰 |
| **Summary Endpoint** | ⭐⭐⭐⭐⭐ (10ms) | 95%+ | Low | ⭐⭐ (1 min) | 💰 |
| **Batch Precomputation** | ⭐⭐⭐⭐⭐ (5ms) | 99% | High | ⭐⭐ (5 min) | 💰💰 |
| **GraphQL + DataLoader** | ⭐⭐⭐⭐ (20ms) | 85-90% | Very High | ⭐⭐⭐⭐ (few sec) | 💰💰💰 |

---

## Recommended Hybrid Approach

### Phase 1: Immediate (2 weeks)
1. **Redis Cache + Pagination**
   - Deploy Redis cluster
   - Add 2-minute cache with pagination
   - Add indexes from Stage 3
   - **Expected:** 100-1000x improvement

2. **Logging Integration**
   - All cache hits/misses logged
   - Query performance metrics
   - Error tracking

### Phase 2: Short-term (4-6 weeks)
3. **Event-Driven Invalidation**
   - Publish events on notification creation
   - Invalidate cache immediately
   - Push WebSocket updates

4. **Summary Endpoint**
   - New endpoint for notification count + high priority
   - Reduces full list queries by 70%

### Phase 3: Medium-term (2-3 months)
5. **Read Replicas**
   - Deploy 2 read replicas
   - Route reads to replicas
   - Distribute load 3x

6. **Batch Precomputation**
   - Background job for caching
   - Further reduce peak load

### Performance Projection

| Phase | Response Time | Queries/sec | DB CPU | Improvement |
|-------|---------------|------------|--------|------------|
| **Current** | 2-5s | 300+ | 90%+ | Baseline |
| **Phase 1** | 50-100ms | 50-70 | 20-30% | **50-100x** |
| **Phase 2** | 20-50ms | 30-40 | 10-15% | **100-250x** |
| **Phase 3** | 5-20ms | 10-15 | <5% | **250-500x** |

---

## Logging Implementation for Monitoring

All cache and query operations must be logged:

```javascript
logger.info('Cache strategy metrics', {
  operation: 'get_notifications',
  cacheHit: true,
  responseTime: 8,
  studentCount: 50000,
  queryVolume: 1500000,
  dbLoad: '22%'
});

logger.warn('Cache miss threshold exceeded', {
  studentID: 12345,
  missRate: 0.35,
  expectedHitRate: 0.80
});

logger.error('Database overload detected', {
  queryQueue: 500,
  avgResponseTime: 3500,
  cpuUsage: 95,
  action: 'activating_emergency_cache'
});
```

---

This comprehensive strategy ensures the notification system scales to support 50,000 students with minimal database load while providing excellent user experience through intelligent caching and progressive loading strategies.

