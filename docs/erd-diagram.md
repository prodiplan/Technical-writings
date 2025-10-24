# ERD (Entity Relationship Diagram) - AI Essay Preparedness Grader

## PlantUML ERD Diagram

```plantuml
@startuml ERD_AI_Essay_Preparedness_Grader

!define RECTANGLE class

' Entity Definitions
entity users {
  * **id**: uuid PK
  --
  * email: varchar(255) unique
  * full_name: varchar(255)
  * birth_date: date
  * school_origin: varchar(255)
  * dream_major: varchar(255)
  * firebase_uid: varchar(255) unique
  * created_at: timestamp
  * updated_at: timestamp
  * is_active: boolean
}

entity grading_sessions {
  * **id**: uuid PK
  --
  * user_id: uuid FK
  * target_major: varchar(255)
  * status: enum('active', 'completed', 'expired')
  * current_score: integer default 0
  * threshold_score: integer default 70
  * question_count: integer default 0
  * started_at: timestamp
  * completed_at: timestamp nullable
  * expires_at: timestamp
  * created_at: timestamp
  * updated_at: timestamp
}

entity messages {
  * **id**: uuid PK
  --
  * session_id: uuid FK
  * message_type: enum('question', 'answer')
  * content: text
  * score: integer nullable
  * is_analyzed: boolean default false
  * created_at: timestamp
  * updated_at: timestamp
}

entity grading_results {
  * **id**: uuid PK
  --
  * session_id: uuid FK unique
  * final_score: integer
  * readiness_level: enum('not_ready', 'somewhat_ready', 'ready', 'very_ready')
  * analysis_report: JSONB
  * created_at: timestamp
  * updated_at: timestamp
}

' Relationships
users ||--o{ grading_sessions : "has"
grading_sessions ||--o{ messages : "contains"
grading_sessions ||--|| grading_results : "produces"

' Notes
note top of users
  User profile information including
  personal data and dream major
end note

note top of grading_sessions
  Active grading session with
  scoring threshold and timeout
end note

note top of messages
  Chat messages between AI and user
  with individual scoring
end note

note top of grading_results
  Final analysis report with
  comprehensive readiness assessment
end note

@enduml
```

## Entity Descriptions

### Users Table
Menyimpan data profil pengguna yang terdaftar dalam sistem.

**Fields:**
- `id`: Primary key unik untuk setiap user
- `email`: Email unik user untuk login
- `full_name`: Nama lengkap user
- `birth_date`: Tanggal lahir user
- `school_origin`: Asal sekolah user
- `dream_major`: Jurusan impian user (dari profile)
- `firebase_uid`: UID dari Firebase Authentication
- `created_at`: Timestamp pembuatan record
- `updated_at`: Timestamp update terakhir
- `is_active`: Status aktif user

### Grading Sessions Table
Menyimpan sesi grading aktif untuk setiap user.

**Fields:**
- `id`: Primary key unik untuk setiap sesi
- `user_id`: Foreign key ke tabel users
- `target_major`: Jurusan yang dituju untuk sesi ini (bisa berbeda dengan dream_major)
- `status`: Status sesi (active, completed, expired)
- `current_score`: Skor akumulasi saat ini
- `threshold_score`: Threshold skor untuk menyelesaikan sesi
- `question_count`: Jumlah pertanyaan yang telah diajukan
- `started_at`: Timestamp mulai sesi
- `completed_at`: Timestamp selesai sesi (nullable)
- `expires_at`: Timestamp kedaluwarsa sesi
- `created_at`: Timestamp pembuatan record
- `updated_at`: Timestamp update terakhir

### Messages Table
Menyimpan pesan-pesan dalam sesi grading.

**Fields:**
- `id`: Primary key unik untuk setiap pesan
- `session_id`: Foreign key ke tabel grading_sessions
- `message_type`: Tipe pesan (question/answer)
- `content`: Isi pesan
- `score`: Skor untuk jawaban user (nullable)
- `is_analyzed`: Status apakah pesan sudah dianalisis AI
- `created_at`: Timestamp pembuatan record
- `updated_at`: Timestamp update terakhir

### Grading Results Table
Menyimpan hasil akhir analisis grading session.

**Fields:**
- `id`: Primary key unik untuk setiap hasil
- `session_id`: Foreign key ke tabel grading_sessions (unique)
- `final_score`: Skor akhir dari sesi
- `readiness_level`: Level kesiapan user
- `analysis_report`: Laporan analisis komprehensif dalam format JSONB (fleksibel untuk kebutuhan klien yang berubah)
- `created_at`: Timestamp pembuatan record
- `updated_at`: Timestamp update terakhir

## Relationship Details

### Users to Grading Sessions (One-to-Many)
- Satu user dapat memiliki multiple grading sessions
- Setiap grading session dimiliki oleh satu user
- Relationship: `users.id` → `grading_sessions.user_id`

### Grading Sessions to Messages (One-to-Many)
- Satu grading session dapat memiliki multiple messages
- Setiap message terkait dengan satu grading session
- Relationship: `grading_sessions.id` → `messages.session_id`

### Grading Sessions to Grading Results (One-to-One)
- Satu grading session menghasilkan satu grading result
- Setiap grading result terkait dengan satu grading session
- Relationship: `grading_sessions.id` → `grading_results.session_id`

## Indexing Strategy

### Primary Keys
- Semua tabel menggunakan UUID sebagai primary key untuk distribusi yang lebih baik

### Foreign Keys
- `grading_sessions.user_id` → index untuk query cepat berdasarkan user
- `messages.session_id` → index untuk retrieval pesan per sesi
- `grading_results.session_id` → unique index untuk one-to-one relationship

### Additional Indexes
- `users.email` → unique index untuk login
- `users.firebase_uid` → unique index untuk Firebase integration
- `grading_sessions.status` → index untuk filtering active sessions
- `grading_sessions.expires_at` → index untuk cleanup expired sessions
- `messages.created_at` → index untuk chronological ordering
- `grading_results.created_at` → index untuk result history

## Data Validation Rules

### Users
- Email harus valid dan unique
- Firebase UID harus unique
- Birth date tidak boleh di masa depan

### Grading Sessions
- Target major tidak boleh kosong
- Status harus valid enum value
- Current score tidak boleh negatif
- Expires_at harus setelah started_at

### Messages
- Content tidak boleh kosong
- Message_type harus valid enum value
- Score hanya untuk message_type 'answer'

### Grading Results
- Final score harus antara 0-100
- Readiness_level harus valid enum value
- Analysis_report tidak boleh kosong