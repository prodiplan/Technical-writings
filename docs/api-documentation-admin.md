# Admin API Documentation

Base URL: `https://api.prodiplan.my.id`

This document details all endpoints available for the Admin Dashboard, covering User Management, Analytics, Grading Verification, and System Monitoring.

## 1. Authentication (Admin)
Currently, the admin dashboard uses a dedicated login mechanism (or reuses the main auth with admin roles).
*Note: Ensure you include the Admin Token in the Authorization header.*

### 1.1 Admin Login
**POST** `/v1/admin/login`
(Development/Mock Endpoint)
**Body:**
```json
{
  "email": "admin@admin.com",
  "password": "any"
}
```

### 1.2 Get Admin Profile
**GET** `/v1/admin/me`
Returns current admin user details.

---

## 2. Dashboard & Analytics
Managed by `admin-service` -> `dashboard` & `analytics`.

### 2.1 Dashboard Stats
**GET** `/v1/admin/dashboard/stats`
High-level overview metrics.
**Response:**
```json
{
  "total_users": 150,
  "active_sessions": 5,
  "completed_sessions": 120,
  "average_score": "75.5"
}
```

### 2.2 Recent Assessments
**GET** `/v1/admin/dashboard/recent-assessments`
Returns the 5 most recent grading sessions.

### 2.3 Analytics Overview
**GET** `/v1/admin/analytics/overview`
Detailed overview for the analytics page.

### 2.4 Trends (Charts)
**GET** `/v1/admin/analytics/trends`
Returns time-series data for:
- User registrations (Last 30 days).
- Completed assessments (Last 30 days).

### 2.5 Score Distribution
**GET** `/v1/admin/analytics/score-distribution`
Returns histograms data: 0-49, 50-69, 70-84, 85-100.

### 2.6 Major Distribution
**GET** `/v1/admin/analytics/major-distribution`
Top 10 target majors selected by users.

---

## 3. User Management
Managed by `admin-service` -> `users`.

### 3.1 List Users
**GET** `/v1/admin/users`
Paginated list of users.

**Query Parameters:**
- `page`: default 1.
- `limit`: default 10.
- `search`: Search by email or full name.

### 3.2 User Stats
**GET** `/v1/admin/users/stats`
Counts of active/inactive users and new users in the last 7 days.

### 3.3 Get User Details
**GET** `/v1/admin/users/:id`
Full profile details of a specific user.

### 3.4 Update User
**PUT** `/v1/admin/users/:id`
Modify user details.

**Body:**
```json
{
  "full_name": "Updated Name",
  "is_active": false  // Ban/Unban user
}
```

### 3.5 Delete User
**DELETE** `/v1/admin/users/:id`
Permanently delete a user from Firebase and PostgreSQL.

---

## 4. Assessment Management
Managed by `admin-service` -> `assessments`.

### 4.1 List Assessments
**GET** `/v1/admin/assessments`
Paginated list of all grading sessions.

**Query Parameters:**
- `page`: default 1.
- `limit`: default 10.

### 4.2 Assessment Stats
**GET** `/v1/admin/assessments/stats`
General stats on session statuses (completed/active/expired).

### 4.3 Get Assessment Details
**GET** `/v1/admin/assessments/:id`
Combined view of Session + User + Result.

### 4.4 Update Status
**PUT** `/v1/admin/assessments/:id/status`
Manually change session status.
**Body:** `{ "status": "completed" }`

---

## 5. Grading Verification (Core Feature)
Managed by `admin-service` -> `sessions`.
This is the workflow for Human-in-the-Loop verification of AI grading.

### 5.1 List Pending Verifications
**GET** `/v1/admin/sessions`
Use filters to find items needing review.

**Query Parameters:**
- `verification_status`: `pending` (Crucial for the review inbox).
- `status`: `completed`.
- `page`, `limit`.

### 5.2 Review: Chat History
**GET** `/v1/admin/sessions/:session_id/messages`
Read the conversation to validate AI's scoring.

### 5.3 Review: Result Content
**GET** `/v1/admin/sessions/:session_id/result`
See the generated score and report.

### 5.4 Submit Decision
**POST** `/v1/admin/sessions/:session_id/verify`
Finalize the result.

**Body (Approve):**
```json
{
  "action": "approve",
  "feedback": "Approved"
}
```

**Body (Reject):**
```json
{
  "action": "reject",
  "feedback": "AI hallucinated the score."
}
```

### 5.5 Regenerate Result
**POST** `/v1/admin/sessions/:session_id/regenerate`
Trigger AI re-evaluation. Resets status to `pending`.
**Body:** `{}` (empty)
