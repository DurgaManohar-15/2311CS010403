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




# Stage 2

Persistent Storage Choice
I'd recommend MySQL (using the default InnoDB storage engine). MySQL handles relational mapping between students and notifications efficiently, scales well when structured with composite indexes, and provides row-level locking for safe data modification.

DB Schema
students Table
SQL
CREATE TABLE students (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL
) ENGINE=InnoDB;
notifications Table
SQL
CREATE TABLE notifications (
    id VARCHAR(36) PRIMARY KEY, -- Stores UUIDs as strings
    studentID INT NOT NULL,
    notificationType ENUM('Placement', 'Result', 'Event') NOT NULL,
    message TEXT NOT NULL,
    isRead BOOLEAN DEFAULT FALSE,
    createdAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (studentID) REFERENCES students(id) ON DELETE CASCADE
) ENGINE=InnoDB;
Potential Scale Issues & Solutions
Data Growth: 5M+ notifications per month will bloat the table, hurting sequential scan times and indexing trees.

Fix: Implement MySQL Native Table Partitioning by range on the createdAt column (e.g., partitioning tables by month). Set up automated background batch cron jobs to clean out or archive rows older than 6 months.

Sample SQL Queries
Fetch unread for a student:

SQL
SELECT id, notificationType, message, createdAt 
FROM notifications 
WHERE studentID = 1042 AND isRead = FALSE 
ORDER BY createdAt DESC;
Mark as read:

SQL
UPDATE notifications 
SET isRead = TRUE 
WHERE id = 'b283218f-ea5a-4b7c-93a9-1f2f240d64b0';
