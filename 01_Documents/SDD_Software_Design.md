# Software Design Document (SDD)
## Sistem Informasi Sekolah dan Absensi

**Document Version**: 1.0  
**Date**: August 10, 2026  
**Author**: [Group Name]  
**Status**: Draft

---

## 1. DESIGN OVERVIEW

### 1.1 Architecture Pattern
The system uses a **Layered Architecture Pattern** with the following layers:
- Presentation Layer (UI - HTML/CSS/JavaScript)
- Application/Business Logic Layer (PHP Controllers)
- Data Access Layer (Database Abstraction)
- Database Layer (MySQL)

### 1.2 High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                     CLIENT TIER (Presentation)                   │
│  ┌───────────────┐  ┌──────────────┐  ┌─────────────────────┐  │
│  │  Admin        │  │  Teacher     │  │  Login/Auth Pages   │  │
│  │  Dashboard    │  │  Dashboard   │  │                     │  │
│  └───────────────┘  └──────────────┘  └─────────────────────┘  │
└──────────────────────────────┬──────────────────────────────────┘
                               │
                    HTTP/HTTPS Requests
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│              APPLICATION TIER (Business Logic)                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              Router/Entry Point (index.php)              │  │
│  └────────────────────────┬─────────────────────────────────┘  │
│                           │                                     │
│  ┌────────────────────────▼─────────────────────────────────┐  │
│  │         Authentication & Session Management               │  │
│  │         (auth/login.php, auth/logout.php)               │  │
│  └──────────────────────────────────────────────────────────┘  │
│                           │                                     │
│  ┌────────────────────────▼─────────────────────────────────┐  │
│  │            Business Logic Modules                        │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │  │
│  │  │   User       │  │   Student    │  │  Attendance  │   │  │
│  │  │ Management   │  │ Management   │  │  Management  │   │  │
│  │  └──────────────┘  └──────────────┘  └──────────────┘   │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │  │
│  │  │   Teacher    │  │   Class      │  │  Subject &   │   │  │
│  │  │ Management   │  │ Management   │  │  Schedule    │   │  │
│  │  └──────────────┘  └──────────────┘  └──────────────┘   │  │
│  └──────────────────────────────────────────────────────────┘  │
│                           │                                     │
│  ┌────────────────────────▼─────────────────────────────────┐  │
│  │       Data Access Layer (Query Building & Execution)    │  │
│  │        - Parameterized Queries (SQL Injection Prevention)│  │
│  │        - Database Connection Management                  │  │
│  │        - Transaction Handling                            │  │
│  └──────────────────────────────────────────────────────────┘  │
└──────────────────────────────┬──────────────────────────────────┘
                               │
                  SQL Queries & Transactions
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                    DATABASE TIER (MySQL)                        │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  ┌─────────┐  ┌────────┐  ┌──────────┐  ┌──────────┐   │  │
│  │  │  users  │  │ students│  │ teachers │  │ classes  │   │  │
│  │  └─────────┘  └────────┘  └──────────┘  └──────────┘   │  │
│  │  ┌────────────┐  ┌─────────┐  ┌──────────────────┐     │  │
│  │  │  subjects  │  │schedules│  │  attendance      │     │  │
│  │  └────────────┘  └─────────┘  └──────────────────┘     │  │
│  │  ┌────────────────────────────────────────────────────┐ │  │
│  │  │           Indexes & Constraints                  │ │  │
│  │  │  - Primary Keys  - Foreign Keys                  │ │  │
│  │  │  - Unique Constraints  - Check Constraints      │ │  │
│  │  └────────────────────────────────────────────────────┘ │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### 1.3 Scalability Architecture (Future Enhancement)

```
┌─────────────────────────────────────────────────────────────────┐
│                       LOAD BALANCER (Nginx)                     │
│                      (Round-robin routing)                      │
└────────┬──────────────────────┬──────────────────────┬──────────┘
         │                      │                      │
    ┌────▼─────┐           ┌────▼─────┐          ┌────▼─────┐
    │  Web     │           │  Web     │          │  Web     │
    │ Server 1 │           │ Server 2 │          │ Server N │
    │(Apache)  │           │(Apache)  │          │(Apache)  │
    └────┬─────┘           └────┬─────┘          └────┬─────┘
         │                      │                      │
         └──────────┬───────────┴──────────┬───────────┘
                    │                      │
              ┌─────▼──────┐         ┌─────▼──────┐
              │ Cache      │         │ Database   │
              │ Layer      │         │ Primary    │
              │(Redis)     │         │(MySQL)     │
              └────────────┘         └─────┬──────┘
                                           │
                                      ┌────▼──────┐
                                      │ Database  │
                                      │ Replica   │
                                      │(MySQL)    │
                                      └───────────┘
```

**Scalability Features**:
- Load Balancer: Distribute traffic across multiple web servers
- Session Storage: Redis untuk session management (independent dari server)
- Caching: Redis untuk query result caching
- Database Replication: Master-Slave untuk read scalability
- Database Sharding: Attendance records dapat di-shard by semester/year

---

## 2. DATABASE DESIGN

### 2.1 Entity-Relationship Diagram (ERD)

```
┌──────────────────┐
│     users        │
├──────────────────┤
│ PK user_id       │◄────┐
│    username      │     │
│    password_hash │     │
│    email         │     │
│    role          │     │
│    is_active     │     │ 1:1
│    created_at    │     │
│    updated_at    │     │
└──────────────────┘     │
         │               │
         │           ┌───┴──────────────┐
         │           │                  │
    ┌────▼──────────┐ │ ┌──────────────┐│
    │  teachers     │ │ │   students   ││
    ├───────────────┤ │ ├──────────────┤│
    │PK teacher_id  │ │ │PK student_id ││
    │  name         │ │ │  name        ││
    │  nip          │ │ │  nis/nisn    ││
    │FK user_id    ─┘ │ │  date_birth  ││
    │  subject_id  ┐  │ │  address     ││
    │  is_active   │  │ │FK class_id   ││
    └──────────────┘  │ │  phone       ││
         │            │ │  created_at  ││
         │            │ └──────────────┘│
         │            │         │       │
         └────────────┘         │       │
                                │       │
                     ┌──────────▼───────┘
                     │ N:1
                     │
                 ┌───┴──────────┐
                 │   classes    │
                 ├──────────────┤
                 │PK class_id   │
                 │  name        │
                 │  level       │◄────────┐
                 │FK teacher_id │         │
                 │  year        │ 1:N     │
                 └──────────────┘         │
                     ▲                    │
                     │ N:1                │
                     │                    │
              ┌──────┴──────┐      ┌──────┴────────┐
              │  subjects   │      │  schedules    │
              ├─────────────┤      ├───────────────┤
              │PK subject_id│      │PK schedule_id │
              │  name       │◄─────│  name         │
              │  code       │ N:1  │FK teacher_id ─┤
              │  credit     │      │FK subject_id ─┤
              └─────────────┘      │FK class_id    │
                                   │  day          │
                                   │  start_time   │
                                   │  end_time     │
                                   │  room         │
                                   └───────────────┘
                                        │
                                   N:1  │
                                        │
                                   ┌────▼─────────────┐
                                   │  attendance      │
                                   ├──────────────────┤
                                   │PK attendance_id  │
                                   │FK schedule_id    │
                                   │FK student_id     │
                                   │  date            │
                                   │  status          │
                                   │  (hadir,sakit,   │
                                   │   izin, alfa)    │
                                   │  input_by        │
                                   │  input_time      │
                                   │  last_updated    │
                                   └──────────────────┘
```

### 2.2 Database Schema Details

#### Table: users
```sql
CREATE TABLE users (
  user_id INT PRIMARY KEY AUTO_INCREMENT,
  username VARCHAR(50) NOT NULL UNIQUE,
  password_hash VARCHAR(255) NOT NULL,
  email VARCHAR(100) NOT NULL,
  role ENUM('Admin', 'Guru') NOT NULL,
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  created_by INT,
  updated_by INT,
  FOREIGN KEY (created_by) REFERENCES users(user_id),
  FOREIGN KEY (updated_by) REFERENCES users(user_id),
  INDEX idx_username (username),
  INDEX idx_role (role),
  INDEX idx_is_active (is_active)
);
```

#### Table: students
```sql
CREATE TABLE students (
  student_id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100) NOT NULL,
  nis VARCHAR(20) NOT NULL UNIQUE,
  nisn VARCHAR(20) UNIQUE,
  date_of_birth DATE,
  gender ENUM('M', 'F'),
  address TEXT,
  phone_parent VARCHAR(20),
  class_id INT NOT NULL,
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (class_id) REFERENCES classes(class_id),
  INDEX idx_nis (nis),
  INDEX idx_class_id (class_id),
  INDEX idx_name (name),
  INDEX idx_is_active (is_active)
);
```

#### Table: teachers
```sql
CREATE TABLE teachers (
  teacher_id INT PRIMARY KEY AUTO_INCREMENT,
  user_id INT UNIQUE,
  name VARCHAR(100) NOT NULL,
  nip VARCHAR(20) NOT NULL UNIQUE,
  subject_id INT,
  email VARCHAR(100),
  phone VARCHAR(20),
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(user_id),
  FOREIGN KEY (subject_id) REFERENCES subjects(subject_id),
  INDEX idx_nip (nip),
  INDEX idx_user_id (user_id),
  INDEX idx_is_active (is_active)
);
```

#### Table: classes
```sql
CREATE TABLE classes (
  class_id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(50) NOT NULL,
  level INT NOT NULL COMMENT '7, 8, 9 untuk SMP',
  teacher_id INT COMMENT 'Wali Kelas',
  academic_year VARCHAR(9) NOT NULL COMMENT '2023/2024',
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (teacher_id) REFERENCES teachers(teacher_id),
  UNIQUE KEY unique_class_year (name, academic_year),
  INDEX idx_level (level),
  INDEX idx_is_active (is_active)
);
```

#### Table: subjects
```sql
CREATE TABLE subjects (
  subject_id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100) NOT NULL,
  code VARCHAR(10) NOT NULL UNIQUE,
  credit INT,
  description TEXT,
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  INDEX idx_code (code),
  INDEX idx_is_active (is_active)
);
```

#### Table: schedules
```sql
CREATE TABLE schedules (
  schedule_id INT PRIMARY KEY AUTO_INCREMENT,
  teacher_id INT NOT NULL,
  subject_id INT NOT NULL,
  class_id INT NOT NULL,
  day_of_week INT NOT NULL COMMENT '0=Sunday, 6=Saturday',
  start_time TIME NOT NULL,
  end_time TIME NOT NULL,
  room VARCHAR(20),
  academic_year VARCHAR(9) NOT NULL,
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (teacher_id) REFERENCES teachers(teacher_id),
  FOREIGN KEY (subject_id) REFERENCES subjects(subject_id),
  FOREIGN KEY (class_id) REFERENCES classes(class_id),
  INDEX idx_teacher_id (teacher_id),
  INDEX idx_class_id (class_id),
  INDEX idx_day (day_of_week),
  INDEX idx_academic_year (academic_year)
);
```

#### Table: attendance
```sql
CREATE TABLE attendance (
  attendance_id INT PRIMARY KEY AUTO_INCREMENT,
  schedule_id INT NOT NULL,
  student_id INT NOT NULL,
  attendance_date DATE NOT NULL,
  status ENUM('Hadir', 'Sakit', 'Izin', 'Alfa') NOT NULL DEFAULT 'Alfa',
  notes TEXT,
  input_by INT NOT NULL,
  input_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  last_updated_by INT,
  last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (schedule_id) REFERENCES schedules(schedule_id),
  FOREIGN KEY (student_id) REFERENCES students(student_id),
  FOREIGN KEY (input_by) REFERENCES users(user_id),
  FOREIGN KEY (last_updated_by) REFERENCES users(user_id),
  UNIQUE KEY unique_attendance (schedule_id, student_id, attendance_date),
  INDEX idx_student_id (student_id),
  INDEX idx_attendance_date (attendance_date),
  INDEX idx_status (status),
  INDEX idx_schedule_date (schedule_id, attendance_date)
);
```

---

## 3. API CONTRACT SPECIFICATIONS

### 3.1 Authentication API

#### POST /auth/login
**Purpose**: Authenticate user and create session

**Request**:
```json
{
  "username": "admin",
  "password": "admin123"
}
```

**Response (200 OK)**:
```json
{
  "success": true,
  "message": "Login successful",
  "user": {
    "user_id": 1,
    "username": "admin",
    "role": "Admin",
    "email": "admin@school.com"
  },
  "session_token": "session_hash_value"
}
```

**Response (401 Unauthorized)**:
```json
{
  "success": false,
  "message": "Invalid username or password",
  "error_code": "AUTH_INVALID_CREDENTIALS"
}
```

**Validation Rules**:
- Username: Required, max 50 characters
- Password: Required, min 8 characters

---

#### POST /auth/logout
**Purpose**: End user session

**Response (200 OK)**:
```json
{
  "success": true,
  "message": "Logout successful"
}
```

---

### 3.2 User Management API

#### GET /api/users
**Purpose**: Retrieve all users (Admin only)

**Query Parameters**:
```
- page: INT (default: 1)
- limit: INT (default: 10, max: 100)
- role: ENUM('Admin', 'Guru') (optional)
- is_active: BOOLEAN (optional)
- search: STRING (search by username or email)
```

**Response (200 OK)**:
```json
{
  "success": true,
  "data": [
    {
      "user_id": 1,
      "username": "admin",
      "email": "admin@school.com",
      "role": "Admin",
      "is_active": true,
      "created_at": "2024-01-01T10:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 50,
    "total_pages": 5
  }
}
```

---

#### POST /api/users
**Purpose**: Create new user (Admin only)

**Request**:
```json
{
  "username": "guru001",
  "password": "SecurePass123",
  "email": "guru@school.com",
  "role": "Guru"
}
```

**Response (201 Created)**:
```json
{
  "success": true,
  "message": "User created successfully",
  "data": {
    "user_id": 2,
    "username": "guru001",
    "email": "guru@school.com",
    "role": "Guru",
    "is_active": true,
    "created_at": "2024-08-10T15:30:00Z"
  }
}
```

**Response (400 Bad Request)**:
```json
{
  "success": false,
  "message": "Validation error",
  "errors": {
    "username": "Username already exists",
    "password": "Password must be at least 8 characters"
  }
}
```

---

#### PUT /api/users/{user_id}
**Purpose**: Update user (Admin only)

**Request**:
```json
{
  "email": "newemail@school.com",
  "is_active": false
}
```

**Response (200 OK)**:
```json
{
  "success": true,
  "message": "User updated successfully",
  "data": {
    "user_id": 2,
    "username": "guru001",
    "email": "newemail@school.com",
    "is_active": false,
    "updated_at": "2024-08-10T16:00:00Z"
  }
}
```

---

#### DELETE /api/users/{user_id}
**Purpose**: Delete user (soft delete) (Admin only)

**Response (200 OK)**:
```json
{
  "success": true,
  "message": "User deleted successfully"
}
```

---

### 3.3 Students API

#### GET /api/students
**Purpose**: Retrieve all students with filtering

**Query Parameters**:
```
- page: INT (default: 1)
- limit: INT (default: 10)
- class_id: INT (optional)
- search: STRING (search by name or NIS)
```

**Response Format**:
```json
{
  "success": true,
  "data": [
    {
      "student_id": 1,
      "name": "Budi Santoso",
      "nis": "2024001",
      "nisn": "123456789",
      "class": {
        "class_id": 1,
        "name": "7A"
      },
      "is_active": true,
      "created_at": "2024-06-01T08:00:00Z"
    }
  ],
  "pagination": {...}
}
```

---

#### POST /api/students
**Purpose**: Create new student

**Request**:
```json
{
  "name": "Budi Santoso",
  "nis": "2024001",
  "nisn": "123456789",
  "date_of_birth": "2010-05-15",
  "gender": "M",
  "address": "Jl. Merdeka No. 123",
  "phone_parent": "081234567890",
  "class_id": 1
}
```

**Response (201 Created)**:
```json
{
  "success": true,
  "message": "Student created successfully",
  "data": {
    "student_id": 1,
    "name": "Budi Santoso",
    "nis": "2024001",
    ...
  }
}
```

---

### 3.4 Attendance API

#### POST /api/attendance/input
**Purpose**: Input attendance for a class

**Request**:
```json
{
  "schedule_id": 5,
  "attendance_date": "2024-08-10",
  "attendance_records": [
    {
      "student_id": 1,
      "status": "Hadir"
    },
    {
      "student_id": 2,
      "status": "Sakit",
      "notes": "Demam tinggi"
    },
    {
      "student_id": 3,
      "status": "Izin",
      "notes": "Acara keluarga"
    },
    {
      "student_id": 4,
      "status": "Alfa"
    }
  ]
}
```

**Response (201 Created)**:
```json
{
  "success": true,
  "message": "Attendance recorded successfully",
  "data": {
    "schedule_id": 5,
    "attendance_date": "2024-08-10",
    "total_records": 4,
    "hadir": 1,
    "sakit": 1,
    "izin": 1,
    "alfa": 1
  }
}
```

**Response (400 Bad Request)**:
```json
{
  "success": false,
  "message": "Validation error",
  "errors": [
    "All students must have attendance status recorded"
  ]
}
```

---

#### GET /api/attendance/report
**Purpose**: Get attendance report with filters

**Query Parameters**:
```
- start_date: DATE (required)
- end_date: DATE (required)
- class_id: INT (optional)
- student_id: INT (optional)
- status: ENUM (optional)
```

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "summary": {
      "total_sessions": 20,
      "hadir": 18,
      "sakit": 1,
      "izin": 1,
      "alfa": 0
    },
    "details": [
      {
        "student_id": 1,
        "student_name": "Budi Santoso",
        "hadir": 19,
        "sakit": 0,
        "izin": 1,
        "alfa": 0,
        "attendance_percentage": 95
      }
    ]
  }
}
```

---

### 3.5 Error Handling

**Standard Error Response Format**:
```json
{
  "success": false,
  "message": "User-friendly error message",
  "error_code": "ERROR_CODE_IDENTIFIER",
  "http_status": 400,
  "timestamp": "2024-08-10T15:30:00Z"
}
```

**Common Error Codes**:
- `AUTH_REQUIRED`: User must be authenticated
- `AUTH_INVALID_CREDENTIALS`: Login credentials invalid
- `FORBIDDEN`: User doesn't have permission for action
- `RESOURCE_NOT_FOUND`: Requested resource doesn't exist
- `VALIDATION_ERROR`: Input validation failed
- `DATABASE_ERROR`: Database operation failed
- `INTERNAL_SERVER_ERROR`: Unexpected server error

---

### 3.6 Authentication & Authorization

**Headers Required**:
```
Authorization: Bearer {session_token}
Content-Type: application/json
```

**Role-Based Access Control**:
```
- Admin role: Can access all endpoints
- Guru role: Can only access teacher-specific data and attendance input
```

---

## 4. MODULE ARCHITECTURE

### 4.1 Admin Module
**Components**:
- Dashboard: Statistics and overview
- User Management: CRUD operations
- Student Management: CRUD + Export
- Teacher Management: CRUD
- Class Management: CRUD
- Subject Management: CRUD
- Schedule Management: CRUD
- Reports: Attendance analytics and export

### 4.2 Teacher Module
**Components**:
- Dashboard: Personal schedules and stats
- Attendance Input: Input attendance for owned classes
- Attendance History: View past attendance records

### 4.3 Authentication Module
**Components**:
- Login: User authentication
- Session Management: Session validation
- Logout: Session termination

---

## 5. SECURITY ARCHITECTURE

### 5.1 Authentication Flow
```
User Input → Validation → Hash Verification → Session Creation → Authorization Check
```

### 5.2 Password Security
- Algorithm: bcrypt with salt (min 10 rounds)
- Storage: password_hash field in users table
- Never store plain text passwords

### 5.3 Session Security
- Session timeout: 30 minutes inactivity
- Secure cookies: HttpOnly flag set
- CSRF tokens for state-changing operations
- Session invalidation on logout

### 5.4 Input Validation & Sanitization
- Client-side: JavaScript validation
- Server-side: Strict input validation
- Parameterized queries: Prevent SQL injection
- HTML escaping: Prevent XSS attacks

### 5.5 Audit Logging
Fields tracked for audit trail:
- created_by, created_at
- last_updated_by, last_updated
- All CRUD operations logged with user_id and timestamp

---

## 6. DEPLOYMENT ARCHITECTURE

### 6.1 Development Environment
```
Local Machine → PHP Server (localhost:8000) → MySQL (localhost:3306)
```

### 6.2 Production Environment (Future)
```
Client → Load Balancer → Web Servers (1-N) → Cache (Redis) → Database Cluster
         ↓                                        ↓
       SSL/TLS                          Master-Slave Replication
```

---

## 7. TECHNOLOGY STACK

| Layer | Technology |
|-------|------------|
| Frontend | HTML5, CSS3, JavaScript |
| Backend | PHP 7.4+ (Native, no framework) |
| Database | MySQL 5.7+ |
| Web Server | Apache with mod_rewrite |
| Version Control | Git |
| Testing Tools | PHPUnit, JMeter, Postman, Cypress |
| Cache | Redis (future enhancement) |
| Load Balancer | Nginx (future enhancement) |

---

## 8. DESIGN PATTERNS USED

1. **MVC (Model-View-Controller)**
   - Separation of concerns
   - Models: Database entities
   - Views: HTML templates
   - Controllers: Business logic handlers

2. **Singleton Pattern**
   - Database connection management
   - Logger instance

3. **Factory Pattern**
   - Object creation for different entity types

4. **Repository Pattern**
   - Data access abstraction layer

5. **Strategy Pattern**
   - Different attendance status handling strategies

---

## DIAGRAMS SUMMARY

### High-Level Flow Diagram (Login Process)
```
User → Browser → Server → Auth Check → Database → Session → Dashboard
```

### Data Flow (Attendance Input)
```
Teacher Input → Validation → Database Save → Audit Log → Confirmation Message
```

---

**Document Control**:
- Version 1.0 - Initial Draft - 2026-08-10
