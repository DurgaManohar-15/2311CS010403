# Notification System Design

# Stage 1

## Core Actions

### 1. Send Notification
This API is used to send placement, result, or event notifications to selected students.

### 2. Fetch Notifications
Students can use this API to view all their notifications after logging in.

### 3. Mark Notification as Read
Once a student opens a notification, it will be marked as read.

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

**Request**

```json
{
  "title": "New Placement Drive",
  "message": "CSX Corporation is hiring! Apply before deadline.",
  "type": "Placement",
  "target_student_ids": [1042, 1043]
}
```

**Response**

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

**Response**

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

**Response**

```json
{
  "status": "success",
  "message": "Notification marked as read"
}
```

---

## Real-Time Notifications

Whenever a new placement, result, or event is announced, the backend creates a notification and stores it in the database. If the student is online, they receive it instantly using WebSockets. If they are offline, they can see it the next time they log in.

---

# Stage 2

## Database Choice

I would choose **MySQL** because it is simple, reliable, and works well for storing structured data like notifications. It also supports transactions and indexing, which helps improve performance.

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

## Problems When Data Increases

If the number of notifications becomes very large, then:

- Searching takes more time.
- Sorting becomes slower.
- Database size increases.
- Fetching unread notifications also becomes slower.

---

## Solution

To improve performance:

- Create indexes on important columns.
- Archive old notifications if they are no longer needed.
- Use table partitioning for very large data.
- Use pagination to load only a few notifications at a time.

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

---

# Stage 3

## Query Analysis

### Current Query

```sql
SELECT *
FROM notifications
WHERE studentID = 1042
AND isRead = FALSE
ORDER BY createdAt DESC;
```

### Is the Query Correct?

Yes. This query correctly gets all unread notifications of a student and shows the latest notifications first.

### Why Can It Become Slow?

If the table has millions of records:

- Searching takes more time.
- Sorting by `createdAt` becomes expensive.
- CPU and disk usage increase.

---

## Should We Add Indexes on Every Column?

No.

Adding indexes to every column is not a good idea because:

- It uses more storage.
- INSERT, UPDATE, and DELETE operations become slower.
- The database has to update every index whenever data changes.

---

## Better Solution

Create one composite index that matches the query.

```sql
CREATE INDEX idx_student_unread_created
ON notifications(studentID, isRead, createdAt DESC);
```

This helps the database quickly find unread notifications for a student and return them in the correct order.

---

## Fetch Placement Notifications from the Last 7 Days

```sql
SELECT *
FROM notifications
WHERE notificationType = 'Placement'
AND createdAt >= NOW() - INTERVAL 7 DAY;
```

This query returns all placement notifications created during the last seven days.