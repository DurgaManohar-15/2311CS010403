# Notification System Design

# Stage 1

## Core Actions

1. **Send Notification**
   - Send placement, result, or event notifications to selected students.

2. **Fetch Notifications**
   - Allow logged-in students to view all their notifications.

3. **Mark Notification as Read**
   - Update the notification status after the student reads it.

---

## API Endpoints

### 1. Send Notification

**Endpoint**

```
POST /api/v1/notifications
```

**Headers**

```
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body**

```json
{
  "title": "New Placement Drive",
  "message": "CSX Corporation is hiring! Apply before deadline.",
  "type": "Placement",
  "target_student_ids": [1042, 1043]
}
```

**Response (201 Created)**

```json
{
  "status": "success",
  "message": "Notifications queued successfully."
}
```

---

### 2. Fetch Notifications

**Endpoint**

```
GET /api/v1/notifications
```

**Headers**

```
Authorization: Bearer <token>
```

**Query Parameters**

```
page=1
limit=10
```

**Response (200 OK)**

```json
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
```

---

### 3. Mark Notification as Read

**Endpoint**

```
PATCH /api/v1/notifications/:id/read
```

**Headers**

```
Authorization: Bearer <token>
```

**Response (200 OK)**

```json
{
  "status": "success",
  "message": "Notification marked as read"
}
```

---

## Real-Time Notification Mechanism

- Backend creates a notification whenever a placement, event, or result is published.
- Notification is stored in the database.
- Connected users receive the notification instantly using WebSockets (or Server-Sent Events).
- If a user is offline, the notification is stored and displayed when they log in again.

---

# Stage 2

## Database Choice

I would use **MySQL** because it is reliable, easy to maintain, supports transactions, and works well for structured notification data.

---

## Database Schema

### Students Table

```sql
CREATE TABLE students (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL
);
```

### Notifications Table

```sql
CREATE TABLE notifications (
    id VARCHAR(36) PRIMARY KEY,
    studentID INT NOT NULL,
    notificationType ENUM('Placement','Result','Event') NOT NULL,
    message TEXT NOT NULL,
    isRead BOOLEAN DEFAULT FALSE,
    createdAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY(studentID)
    REFERENCES students(id)
    ON DELETE CASCADE
);
```

---

## Problems When Data Grows

As the number of notifications increases:

- Searching becomes slower.
- Database size increases.
- Fetching unread notifications takes more time.
- Sorting by latest notifications becomes slower.

---

## Solution

To improve performance:

- Create indexes on `studentID`, `isRead`, and `createdAt`.
- Archive old notifications after a few months.
- Partition the notification table by date if the data becomes very large.
- Fetch notifications using pagination (`page` and `limit`).

---

## SQL Queries

### Fetch Unread Notifications

```sql
SELECT id,
       notificationType,
       message,
       createdAt
FROM notifications
WHERE studentID = 1042
AND isRead = FALSE
ORDER BY createdAt DESC;
```

---

### Mark Notification as Read

```sql
UPDATE notifications
SET isRead = TRUE
WHERE id = 'b283218f-ea5a-4b7c-93a9-1f2f240d64b0';
```

---

### Fetch Placement Notifications

```sql
SELECT *
FROM notifications
WHERE notificationType = 'Placement';
```