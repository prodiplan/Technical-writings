# API Specification - AI Essay Preparedness Grader

## Overview

API documentation untuk AI Essay Preparedness Grader system menggunakan OpenAPI 3.0 specification. API ini mengikuti RESTful design principles dengan JWT-based authentication.

## Base URL

```
Development: TO BE DETERMINED
Production: TO BE DETERMINED
```

## Authentication

Semua API endpoints (kecuali auth endpoints) memerlukan JWT token di header:

```
Authorization: Bearer <jwt_token>
```

## API Endpoints

### 1. Authentication Service

#### 1.1 User Registration

```http
POST /auth/register
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "securePassword123",
  "full_name": "John Doe",
  "birth_date": "2000-01-15",
  "school_origin": "SMAN 1 Jakarta",
  "dream_major": "Computer Science"
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "uuid",
      "email": "user@example.com",
      "full_name": "John Doe",
      "birth_date": "2000-01-15",
      "school_origin": "SMAN 1 Jakarta",
      "dream_major": "Computer Science",
      "created_at": "2024-01-15T10:30:00Z"
    },
    "token": "jwt_token_here",
    "refresh_token": "refresh_token_here"
  },
  "message": "User registered successfully"
}
```

#### 1.2 User Login

```http
POST /auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "securePassword123"
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "uuid",
      "email": "user@example.com",
      "full_name": "John Doe",
      "last_login_at": "2024-01-15T10:30:00Z"
    },
    "token": "jwt_token_here",
    "refresh_token": "refresh_token_here"
  },
  "message": "Login successful"
}
```

#### 1.3 Refresh Token

```http
POST /auth/refresh
Content-Type: application/json

{
  "refresh_token": "refresh_token_here"
}
```

#### 1.4 Logout

```http
POST /auth/logout
Authorization: Bearer <jwt_token>
```

#### 1.5 Get Current User Profile

```http
GET /auth/me
Authorization: Bearer <jwt_token>
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "email": "user@example.com",
    "full_name": "John Doe",
    "birth_date": "2000-01-15",
    "school_origin": "SMAN 1 Jakarta",
    "dream_major": "Computer Science",
    "avatar_url": "https://example.com/avatar.jpg",
    "phone_number": "+62812345678",
    "email_verified": true,
    "created_at": "2024-01-15T10:30:00Z",
    "updated_at": "2024-01-15T10:30:00Z"
  }
}
```

### 2. Grading Session Service

#### 2.1 Create Grading Session

```http
POST /grading-sessions
Authorization: Bearer <jwt_token>
Content-Type: application/json

{
  "target_major": "Computer Science",
  "max_questions": 10,
  "session_duration_minutes": 60
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "user_id": "uuid",
    "target_major": "Computer Science",
    "status": "active",
    "current_score": 0,
    "threshold_score": 70,
    "question_count": 0,
    "max_questions": 10,
    "session_duration_minutes": 60,
    "started_at": "2024-01-15T10:30:00Z",
    "expires_at": "2024-01-15T11:30:00Z",
    "created_at": "2024-01-15T10:30:00Z"
  },
  "message": "Grading session created successfully"
}
```

#### 2.2 Get Grading Session by ID

```http
GET /grading-sessions/{session_id}
Authorization: Bearer <jwt_token>
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "user_id": "uuid",
    "target_major": "Computer Science",
    "status": "active",
    "current_score": 45,
    "threshold_score": 70,
    "question_count": 3,
    "max_questions": 10,
    "started_at": "2024-01-15T10:30:00Z",
    "expires_at": "2024-01-15T11:30:00Z",
    "last_activity_at": "2024-01-15T10:45:00Z",
    "created_at": "2024-01-15T10:30:00Z"
  }
}
```

#### 2.3 Get User's Grading Sessions

```http
GET /grading-sessions?status=active&limit=10&offset=0
Authorization: Bearer <jwt_token>
```

**Query Parameters:**
- `status` (optional): Filter by status (active, completed, expired)
- `limit` (optional): Number of results per page (default: 10)
- `offset` (optional): Number of results to skip (default: 0)

**Response (200):**
```json
{
  "success": true,
  "data": {
    "sessions": [
      {
        "id": "uuid",
        "target_major": "Computer Science",
        "status": "active",
        "current_score": 45,
        "question_count": 3,
        "started_at": "2024-01-15T10:30:00Z",
        "expires_at": "2024-01-15T11:30:00Z"
      }
    ],
    "pagination": {
      "total": 5,
      "limit": 10,
      "offset": 0,
      "has_next": false,
      "has_prev": false
    }
  }
}
```

#### 2.4 Complete Grading Session

```http
POST /grading-sessions/{session_id}/complete
Authorization: Bearer <jwt_token>
Content-Type: application/json

{
  "final_score": 75,
  "readiness_level": "ready"
}
```

### 3. Messages Service

#### 3.1 Get Session Messages

```http
GET /grading-sessions/{session_id}/messages?limit=50&offset=0
Authorization: Bearer <jwt_token>
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "messages": [
      {
        "id": "uuid",
        "session_id": "uuid",
        "message_type": "question",
        "content": "Mengapa Anda tertarik dengan jurusan Computer Science?",
        "score": null,
        "is_analyzed": false,
        "created_at": "2024-01-15T10:30:00Z"
      },
      {
        "id": "uuid",
        "session_id": "uuid",
        "message_type": "answer",
        "content": "Saya tertarik karena saya suka programming dan ingin membuat aplikasi yang bermanfaat.",
        "score": 15,
        "is_analyzed": true,
        "created_at": "2024-01-15T10:32:00Z"
      }
    ],
    "pagination": {
      "total": 6,
      "limit": 50,
      "offset": 0,
      "has_next": false,
      "has_prev": false
    }
  }
}
```

#### 3.2 Send Message

```http
POST /grading-sessions/{session_id}/messages
Authorization: Bearer <jwt_token>
Content-Type: application/json

{
  "message_type": "answer",
  "content": "Saya tertarik karena saya suka programming dan ingin membuat aplikasi yang bermanfaat."
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "session_id": "uuid",
    "message_type": "answer",
    "content": "Saya tertarik karena saya suka programming dan ingin membuat aplikasi yang bermanfaat.",
    "score": null,
    "is_analyzed": false,
    "created_at": "2024-01-15T10:32:00Z"
  },
  "message": "Message sent successfully"
}
```

### 4. Grading Results Service

#### 4.1 Get Grading Result

```http
GET /grading-results/{session_id}
Authorization: Bearer <jwt_token>
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "session_id": "uuid",
    "final_score": 75,
    "readiness_level": "ready",
    "analysis_report": {
      "summary": "Berdasarkan analisis jawaban Anda, menunjukkan kesiapan yang baik untuk jurusan Computer Science...",
      "recommendations": "Disarankan untuk memperdalam pemahaman tentang algoritma dan struktur data...",
      "strengths": "Motivasi yang kuat, pemahaman dasar programming yang baik",
      "weaknesses": "Kurang pemahaman tentang matematika diskrit",
      "key_insights": {
        "motivation_score": 85,
        "technical_understanding": 70,
        "career_alignment": 80
      },
      "personality_traits": {
        "analytical_thinking": "high",
        "problem_solving": "medium",
        "creativity": "high"
      },
      "career_suggestions": [
        "Software Engineer",
        "Data Scientist",
        "Product Manager"
      ]
    },
    "created_at": "2024-01-15T11:00:00Z"
  }
}
```

#### 4.2 Get User's Grading Results

```http
GET /grading-results?readiness_level=ready&limit=10&offset=0
Authorization: Bearer <jwt_token>
```

**Query Parameters:**
- `readiness_level` (optional): Filter by readiness level
- `limit` (optional): Number of results per page (default: 10)
- `offset` (optional): Number of results to skip (default: 0)

## WebSocket Events

### Connection

```javascript
const socket = io('wss://api.prodiplan.id/v1', {
  auth: {
    token: 'jwt_token_here'
  }
});
```

### Events

#### 1. Join Session

```javascript
socket.emit('join_session', {
  session_id: 'uuid'
});
```

#### 2. Receive Question

```javascript
socket.on('question', (data) => {
  console.log('New question:', data);
  // data format:
  // {
  //   id: "uuid",
  //   session_id: "uuid",
  //   content: "Question content here",
  //   created_at: "2024-01-15T10:30:00Z"
  // }
});
```

#### 3. Send Answer

```javascript
socket.emit('answer', {
  session_id: 'uuid',
  content: 'Answer content here'
});
```

#### 4. Receive Score Update

```javascript
socket.on('score_update', (data) => {
  console.log('Score updated:', data);
  // data format:
  // {
  //   session_id: "uuid",
  //   current_score: 45,
  //   question_score: 15,
  //   question_count: 3
  // }
});
```

#### 5. Session Completed

```javascript
socket.on('session_completed', (data) => {
  console.log('Session completed:', data);
  // data format:
  // {
  //   session_id: "uuid",
  //   final_score: 75,
  //   readiness_level: "ready",
  //   result_id: "uuid"
  // }
});
```

#### 6. Error Events

```javascript
socket.on('error', (data) => {
  console.error('Socket error:', data);
  // data format:
  // {
  //   code: "SESSION_EXPIRED",
  //   message: "Session has expired",
  //   session_id: "uuid"
  // }
});
```

## Error Responses

### Standard Error Format

```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human readable error message",
    "details": {
      "field": "Additional error details"
    }
  }
}
```

### Common Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `UNAUTHORIZED` | 401 | Invalid or missing authentication token |
| `FORBIDDEN` | 403 | User does not have permission to access resource |
| `NOT_FOUND` | 404 | Resource not found |
| `VALIDATION_ERROR` | 400 | Request validation failed |
| `SESSION_EXPIRED` | 410 | Grading session has expired |
| `SESSION_COMPLETED` | 410 | Grading session already completed |
| `RATE_LIMIT_EXCEEDED` | 429 | Too many requests |
| `INTERNAL_ERROR` | 500 | Internal server error |
| `SERVICE_UNAVAILABLE` | 503 | External service unavailable |

### Validation Error Example

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": {
      "email": "Invalid email format",
      "birth_date": "Birth date cannot be in the future"
    }
  }
}
```

## Rate Limiting

### Rate Limits by Endpoint

| Endpoint | Rate Limit | Window |
|----------|------------|--------|
| `/auth/login` | 5 requests | 15 minutes |
| `/auth/register` | 3 requests | 1 hour |
| `/grading-sessions` | 10 requests | 1 hour |
| `/grading-sessions/{id}/messages` | 60 requests | 1 minute |
| Admin endpoints | 100 requests | 1 minute |

### Rate Limit Headers

```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1642248000
```

## Pagination

### Standard Pagination Response

```json
{
  "success": true,
  "data": {
    "items": [...],
    "pagination": {
      "total": 150,
      "limit": 20,
      "offset": 0,
      "has_next": true,
      "has_prev": false,
      "total_pages": 8,
      "current_page": 1
    }
  }
}
```

### Query Parameters

- `limit`: Number of items per page (max: 100)
- `offset`: Number of items to skip
- `page`: Alternative to offset (1-based)

## Caching

### Cache Headers

```http
Cache-Control: public, max-age=300
ETag: "abc123"
Last-Modified: Wed, 15 Jan 2024 10:30:00 GMT
```

### Cacheable Endpoints

- `GET /grading-results/{session_id}`: 5 minutes
- `GET /admin/dashboard`: 1 minute
- `GET /admin/health`: 30 seconds

## API Versioning

### Version Strategy

- URL path versioning: `/v1/`, `/v2/`
- Backward compatibility maintained for at least 6 months
- Deprecation warnings in response headers

```http
API-Version: v1
Deprecation: true
Sunset: Sat, 15 Jul 2024 00:00:00 GMT
```

## Testing

### Postman Collection

API documentation includes Postman collection for easy testing with:

1. Authentication flows
2. Grading session management
3. Real-time messaging
4. Admin operations
5. Error scenarios

### Environment Variables

```json
{
  "base_url": "https://api-dev.prodiplan.id/v1",
  "jwt_token": "{{auth_token}}",
  "session_id": "{{session_id}}",
  "admin_token": "{{admin_token}}"
}