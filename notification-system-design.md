# Notification System Design


# Stage 1

### Core Actions
1. **Send Notification**: Triggered by backend events (Placements, Results, Events) to target students.
2. **Fetch Notifications**: Allows logged-in students to view their notification feed.
3. **Mark as Read**: Updates notification status once viewed.

### API Endpoints & Contracts

#### 1. POST /api/v1/notifications (Send Notification)
* **Headers**: `Authorization: Bearer <token>`, `Content-Type: application/json`
* **Request Body**:
```json
{
  "title": "New Placement Drive",
  "message": "CSX Corporation is hiring! Apply before deadline.",
  "type": "Placement",
  "target_student_ids": [1042, 1043]
}
Response (201 Created):

JSON
{
  "status": "success",
  "message": "Notifications queued successfully."
}
2. GET /api/v1/notifications (Fetch Notifications)
Headers: Authorization: Bearer <token>

Query Params: page=1&limit=10

Response (200 OK):

JSON
{
  "notifications": [
    {
      "id": "b283218f-ea5a-4b7c-93a9-1f2f240d64b0",
      "type": "Placement",
      "message": "CSX Corporation hiring",
      "timestamp": "2026-04-22 17:51:18",
      "isRead": false
    }
  ]
}
3. PATCH /api/v1/notifications/:id/read (Mark as Read)
Headers: Authorization: Bearer <token>

Response (200 OK):

JSON
{
  "status": "success",
  "message": "Notification marked as read"
}
