# Daftar Endpoint, Request, dan Response Berdasarkan Service dan Konsumennya

Berikut daftar endpoint sistem **AI Readiness Analyzer**, disusun berdasarkan service yang membuatnya dan siapa konsumennya (frontend, internal, antar-service). Setiap API disertai contoh request dan response.

---

## 1. Auth Service

**Tujuan:** autentikasi dan manajemen pengguna.
**Konsumen:** Frontend (melalui API Gateway)

### `POST /auth/register`

**Untuk:** User mendaftar akun baru.

```json
// Request
{
  "email": "siswa@example.com",
  "password": "rahasia123",
  "full_name": "Budi Santoso",
  "birth_date": "2006-05-21",
  "school_origin": "SMA Negeri 1"
}

// Response
{
  "id": "uuid-user-xxxx",
  "email": "siswa@example.com",
  "full_name": "Budi Santoso",
  "birth_date": "2006-05-21",
  "school_origin": "SMA Negeri 1",
  "created_at": "2025-10-17T08:30:00Z"
}
```

### `POST /auth/login`

**Untuk:** User login untuk mendapatkan JWT.

```json
// Request
{
  "email": "siswa@example.com",
  "password": "rahasia123"
}

// Response
{
  "access_token": "jwt.token.here",
  "token_type": "bearer",
  "expires_in": 3600,
  "user": {
    "id": "uuid-user-xxxx",
    "email": "siswa@example.com",
    "full_name": "Budi Santoso"
  }
}
```

### `GET /users/me`

**Untuk:** Frontend mengambil profil user dari token JWT.

```json
// Response
{
  "id": "uuid-user-xxxx",
  "email": "siswa@example.com",
  "full_name": "Budi Santoso",
  "birth_date": "2006-05-21",
  "school_origin": "SMA Negeri 1",
  "created_at": "2025-10-17T08:30:00Z"
}
```

### `PATCH /users/me`

**Untuk:** User memperbarui profilnya.

```json
// Request
{
  "full_name": "Budi S.",
  "school_origin": "SMA Negeri 2"
}

// Response
{ "ok": true }
```

---

## 2. Session Service

**Tujuan:** mengelola sesi penilaian.
**Konsumen:** Frontend dan Chat Service (via API Gateway)

### `POST /sessions`

**Untuk:** User memulai sesi penilaian baru.

```json
// Request
{
  "model_version": "gpt-xyz-v1",
  "metadata": { "task": "esai", "topic": "sejarah" }
}

// Response
{
  "id": "uuid-session-aaaa",
  "user_id": "uuid-user-xxxx",
  "started_at": "2025-10-17T09:00:00Z",
  "status": "in_progress",
  "model_version": "gpt-xyz-v1",
  "metadata": { "task": "esai", "topic": "sejarah" }
}
```

### `GET /sessions/{session_id}`

**Untuk:** Mendapat detail sesi.

```json
{
  "id": "uuid-session-aaaa",
  "user_id": "uuid-user-xxxx",
  "started_at": "2025-10-17T09:00:00Z",
  "finished_at": null,
  "status": "in_progress",
  "model_version": "gpt-xyz-v1",
  "metadata": { "task": "esai", "topic": "sejarah" }
}
```

### `PATCH /sessions/{session_id}`

**Untuk:** Update status sesi.

```json
// Request
{ "status": "cancelled" }

// Response
{ "ok": true }
```

---

## 3. Chat / Grading Service

**Tujuan:** menangani percakapan antara user dan AI.
**Konsumen:** Frontend, LLM, dan Result Service.

### `POST /chat/start`

**Untuk:** Inisialisasi sesi chat dan membuat pesan asisten awal.

```json
// Request
{
  "session_id": "uuid-session-aaaa",
  "initial_prompt": "Silakan kirim esai Anda. Fokus pada struktur dan argumen."
}

// Response
{
  "message_id": "uuid-message-init",
  "role": "assistant",
  "type": "system",
  "content": "Silakan kirim esai Anda. Fokus pada struktur dan argumen.",
  "created_at": "2025-10-17T09:00:01Z"
}
```

### `POST /chat/message`

**Untuk:** User mengirim pesan/esai.

```json
// Request
{
  "session_id": "uuid-session-aaaa",
  "role": "user",
  "type": "essay",
  "content": "Ini esai saya tentang kemerdekaan.",
  "metadata": {}
}

// Response
{
  "assistant_message": {
    "id": "uuid-message-1111",
    "role": "assistant",
    "type": "follow_up",
    "content": "Bisa jelaskan lebih detail bagian kronologi?",
    "tokens_used": 120,
    "confidence": 0.85,
    "created_at": "2025-10-17T09:01:00Z"
  },
  "session_status": "in_progress"
}
```

### `GET /sessions/{session_id}/messages`

**Untuk:** Mengambil semua pesan dalam satu sesi.

```json
[
  {
    "id": "uuid-message-init",
    "role": "assistant",
    "content": "Silakan kirim esai Anda...",
    "created_at": "2025-10-17T09:00:01Z"
  },
  {
    "id": "uuid-message-1111",
    "role": "assistant",
    "content": "Bisa jelaskan lebih detail bagian kronologi?",
    "created_at": "2025-10-17T09:01:00Z"
  }
]
```

### `PATCH /messages/{message_id}/metadata`

**Untuk:** Internal service menandai pesan sebagai evidence hasil grading.

```json
// Request
{ "metadata": { "evidence_for": "uuid-result-zzz" } }

// Response
{ "ok": true }
```

---

## 4. Result Service

**Tujuan:** menyimpan dan mengambil hasil penilaian.
**Konsumen:** Frontend dan Chat Service.

### `GET /results/{result_id}`

**Untuk:** Mengambil hasil akhir grading.

```json
{
  "id": "uuid-result-zzz",
  "session_id": "uuid-session-aaaa",
  "score": 85.5,
  "rubric": {
    "structure": 20,
    "argument": 30,
    "language": 25,
    "originality": 10.5
  },
  "conclusion": "Baik, namun perlu pendalaman argumen.",
  "explanation": "Esai terstruktur dengan baik, argumen masih dangkal.",
  "created_at": "2025-10-17T09:05:00Z"
}
```

### `GET /sessions/{session_id}/results`

**Untuk:** Mendapatkan semua hasil dari satu sesi.

```json
[
  {
    "id": "uuid-result-zzz",
    "score": 85.5,
    "conclusion": "Baik, namun perlu pendalaman argumen.",
    "created_at": "2025-10-17T09:05:00Z"
  }
]
```

### `POST /results/{result_id}/export`

**Untuk:** Menghasilkan laporan PDF hasil penilaian.

```json
// Request
{ "format": "pdf" }

// Response
{ "download_url": "/downloads/result-uuid-result-zzz.pdf" }
```

---

## 5. WebSocket / Realtime Service

**Tujuan:** mengirim notifikasi realtime ke frontend.
**Konsumen:** Frontend.

### `WS /ws`

**Untuk:** Push event saat grading selesai atau asisten mengirim balasan.

```json
// Event: grading_complete
{
  "event": "grading_complete",
  "data": {
    "session_id": "uuid-session-aaaa",
    "result_id": "uuid-result-zzz",
    "score": 85.5
  }
}

// Event: assistant_message
{
  "event": "assistant_message",
  "data": {
    "message": {
      "id": "uuid-message-2222",
      "content": "Terima kasih, penilaian selesai.",
      "role": "assistant"
    }
  }
}
```

---

## 6. Admin / Monitoring Service

**Tujuan:** audit dan pemantauan sistem backend.
**Konsumen:** Admin panel.

### `GET /admin/sessions`

**Untuk:** Melihat daftar sesi aktif dan status.

```json
[
  { "id": "uuid-session-aaaa", "status": "in_progress", "user_id": "uuid-user-xxxx" },
  { "id": "uuid-session-bbbb", "status": "completed", "user_id": "uuid-user-yyyy" }
]
```

### `GET /admin/llm/usage`

**Untuk:** Melihat statistik penggunaan token dan performa model.

```json
{
  "total_tokens": 125000,
  "requests": 350,
  "average_confidence": 0.88,
  "period": "2025-10-01 to 2025-10-17"
}
```

---

## Ringkasan Service

| Service                | Endpoint Prefix      | Konsumen            | Fungsi Utama                 |
| ---------------------- | -------------------- | ------------------- | ---------------------------- |
| **Auth Service**       | `/auth`, `/users`    | Frontend            | Registrasi, login, user info |
| **Session Service**    | `/sessions`          | Frontend            | Manajemen sesi penilaian     |
| **Chat Service**       | `/chat`, `/messages` | Frontend & internal | Percakapan & interaksi LLM   |
| **Result Service**     | `/results`           | Frontend            | Hasil & laporan penilaian    |
| **WebSocket Service**  | `/ws`                | Frontend            | Notifikasi realtime          |
| **Admin / Monitoring** | `/admin`             | Admin panel         | Audit & pemantauan sistem    |

---

Setiap service memiliki tanggung jawab tunggal dan diakses melalui API Gateway. Frontend hanya berinteraksi dengan Auth, Session, Chat, Result, dan WebSocket Service — sedangkan admin dan komunikasi antar-service tetap internal.
