# Frontend API Documentation

Base URL: `https://api.prodiplan.my.id`

This document outlines all public endpoints available for the Frontend Application (User-facing).

## 1. Authentication
Managed by `auth-service`. The application uses Firebase Auth integration with a custom backend handling PostgreSQL data.

### 1.1 Register
**POST** `/v1/auth/register`
Create a new user account.

**Body:**
```json
{
  "email": "user@example.com",
  "password": "securePassword123",
  "full_name": "John Doe",
  "birth_date": "2000-01-01",
  "school_origin": "High School A",
  "dream_major": "Computer Science"
}
```
**Response:** `201 Created` with User object and Tokens.

### 1.2 Login
**POST** `/v1/auth/login`
Authenticate an existing user.

**Body:**
```json
{
  "email": "user@example.com",
  "password": "securePassword123"
}
```
**Response:** `200 OK` with User object and Tokens.

### 1.3 Get Current User
**GET** `/v1/auth/me`
Retrieve profile of the currently logged-in user.
**Headers:** `Authorization: Bearer <access_token>`

### 1.4 Refresh Token
**POST** `/v1/auth/refresh`
Refresh the access token when it expires.

**Body:**
```json
{
  "refresh_token": "..."
}
```

### 1.5 Update Profile
**PATCH** `/v1/auth/profile`
Update user profile fields.

**Body (All fields optional):**
```json
{
  "full_name": "New Name",
  "phone_number": "+1234567890",
  "avatar_url": "https://example.com/avatar.jpg",
  "dream_major": "Physics"
}
```

### 1.6 Delete Account
**DELETE** `/v1/auth/user`
Permanently delete the user account from both database and auth provider.

**Body:**
```json
{
  "password": "currentPasswordConfirmation"
}
```

### 1.7 Logout
**POST** `/v1/auth/logout`
Log out and invalidate refresh tokens.

**Body:**
```json
{
  "refresh_token": "..." // Optional
}
```

### 1.8 Password Management
#### Forgot Password
**POST** `/v1/auth/forgot-password`
Triggers a password reset email to the user.

**Body:**
```json
{
  "email": "user@example.com"
}
```

#### Reset Password
**POST** `/v1/auth/reset-password`
Completes the password reset process using the code from the email.

**Body:**
```json
{
  "oobCode": "code_from_email_link",
  "newPassword": "newSecurePassword123"
}
```

---

## 2. Grading Sessions
Managed by `session-service`. All endpoints require `Authorization` header.

### 2.1 Create Session (Start Application)
**POST** `/v1/grading-sessions`
Initialize a new grading session. This automatically generates the first question via AI.

**Body:**
```json
{
  "target_major": "Computer Science",
  "max_questions": 10,             // Optional, default: 10
  "session_duration_minutes": 60   // Optional, default: 60
}
```

**Response:**
Returns the session ID and the `first_question`.

### 2.2 List Sessions
**GET** `/v1/grading-sessions`
Get a paginated list of the user's sessions.

**Query Parameters:**
- `status`: Filter by status (`active`, `completed`, `expired`).
- `limit`: Items per page (default: 10).
- `offset`: Pagination offset (default: 0).

### 2.3 Get Session Details
**GET** `/v1/grading-sessions/:session_id`
Get full details of a specific session, including current score and status.

### 2.4 Send Message (Answer & Next Question)
**POST** `/v1/grading-sessions/:session_id/messages`
Submit an answer to the current question. The AI will process this asynchronously.
You should listen for WebSocket updates or poll for the next question.

**Body:**
```json
{
  "message_type": "answer",
  "content": "My answer to the question..."
}
```

### 2.5 Get Chat History
**GET** `/v1/grading-sessions/:session_id/messages`
Retrieve the full conversation history (questions and answers).

**Query Parameters:**
- `limit`: defaults to 50.
- `offset`: defaults to 0.

### 2.6 Complete Session
**POST** `/v1/grading-sessions/:session_id/complete`
Manually finish the session. This triggers final result generation.
*Note: The result might be `pending` verification immediately after completion.*

### 2.7 Delete Session
**DELETE** `/v1/grading-sessions/:session_id`
Delete a grading session and all its associated data (messages, results).

**Response:**
`200 OK`


---

## 3. Grading Results
Managed by `result-service`.

### 3.1 Get Result for Session
**GET** `/v1/grading-results/:session_id`
Fetch the final grading result for a session.
*Returns 404 if result is not yet available.*

### 3.2 List Results
**GET** `/v1/grading-results`
Get a history of grading results with pagination.

**Query Parameters:**
- `readiness_level`: Filter by level (e.g., `ready`, `not_ready`).
- `limit`: defaults to 10.
- `offset`: defaults to 0.

### 3.3 Get Statistics
**GET** `/v1/grading-results/statistics`
Get aggregated statistics for the user:
- Total sessions taken.
- Average score.
- Distribution of readiness levels.
- Latest result details.
