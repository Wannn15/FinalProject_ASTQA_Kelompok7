# Software Requirements Specification (SRS)
## Sistem Informasi Sekolah dan Absensi (School Information System & Attendance)

**Document Version**: 1.0  
**Date**: August 10, 2026  
**Author**: [Group Name]  
**Status**: Draft

---

## 1. EXECUTIVE SUMMARY

Sistem Informasi Sekolah dan Absensi adalah aplikasi web berbasis PHP Native yang dirancang untuk mengelola data siswa, guru, kelas, mata pelajaran, jadwal mengajar, dan sistem absensi di lingkungan sekolah. Sistem ini menyediakan dashboard interaktif untuk administrator dan guru dengan fitur pelaporan komprehensif.

---

## 2. FUNCTIONAL REQUIREMENTS (FR)

### FR-1: User Authentication & Authorization
**Description**: Sistem harus menyediakan mekanisme login yang aman dengan role-based access control (RBAC).

**Actors**: Admin, Guru (Teacher)

**Detailed Requirements**:
- FR-1.1: User harus login dengan username dan password
- FR-1.2: Password harus dienkripsi menggunakan bcrypt
- FR-1.3: Sistem harus membedakan akses berdasarkan role (Admin/Guru)
- FR-1.4: Session management dengan timeout 30 menit
- FR-1.5: User dapat logout dan session akan terminated
- FR-1.6: Failed login attempts harus dicatat dalam audit log

**Acceptance Criteria**:
- ✓ Login berhasil dengan kredensial yang valid
- ✓ Login gagal dengan kredensial yang tidak valid
- ✓ Session expired setelah 30 menit inaktif
- ✓ Logout menghapus session dengan benar

---

### FR-2: User Management (Admin Only)
**Description**: Administrator dapat mengelola user (CRUD) dengan role assignment.

**Actors**: Admin

**Detailed Requirements**:
- FR-2.1: Admin dapat menambah user baru dengan username, password, email, dan role
- FR-2.2: Admin dapat melihat daftar semua user dengan status aktif/nonaktif
- FR-2.3: Admin dapat mengedit data user (email, status, role)
- FR-2.4: Admin dapat menghapus user (soft delete)
- FR-2.5: Admin dapat reset password user ke password default
- FR-2.6: Sistem harus validasi username unique dan password strength (min 8 karakter, kombinasi huruf-angka)

**Acceptance Criteria**:
- ✓ User baru dapat ditambahkan dengan validasi lengkap
- ✓ User dapat di-edit tanpa mengubah username
- ✓ User dapat di-hapus (tidak menampilkan di list aktif)
- ✓ Password reset berhasil dan user dapat login dengan password baru

---

### FR-3: Data Master Management - Siswa (Students)
**Description**: Mengelola data siswa dengan informasi lengkap dan export functionality.

**Actors**: Admin

**Detailed Requirements**:
- FR-3.1: Admin dapat menambah siswa dengan data: nama, NIS/NISN, kelas, tanggal lahir, alamat, no. telepon orang tua
- FR-3.2: Admin dapat melihat list siswa dengan fitur search dan filter berdasarkan kelas
- FR-3.3: Admin dapat mengedit data siswa yang sudah ada
- FR-3.4: Admin dapat menghapus siswa (soft delete)
- FR-3.5: Admin dapat export data siswa ke format CSV
- FR-3.6: Sistem harus validasi NIS/NISN unique dan format valid
- FR-3.7: Sistem harus menampilkan jumlah siswa per kelas di dashboard

**Acceptance Criteria**:
- ✓ Siswa baru dapat ditambahkan dengan data lengkap
- ✓ Data siswa dapat dicari berdasarkan nama atau NIS
- ✓ Filter berdasarkan kelas berfungsi dengan baik
- ✓ Export CSV menghasilkan file dengan format yang benar

---

### FR-4: Data Master Management - Guru (Teachers)
**Description**: Mengelola data guru dengan informasi profesional.

**Actors**: Admin

**Detailed Requirements**:
- FR-4.1: Admin dapat menambah guru dengan data: nama, NIP, mata pelajaran, email, no. telepon
- FR-4.2: Admin dapat melihat list guru dengan status aktif/nonaktif
- FR-4.3: Admin dapat mengedit data guru
- FR-4.4: Admin dapat menghapus guru (soft delete)
- FR-4.5: Sistem harus validasi NIP unique dan format valid
- FR-4.6: Guru harus dapat dihubungkan dengan user account untuk login

**Acceptance Criteria**:
- ✓ Guru baru dapat ditambahkan dengan data lengkap
- ✓ Guru dapat diubah statusnya (aktif/nonaktif)
- ✓ NIP tidak dapat diduplikasi di sistem

---

### FR-5: Attendance System - Input & Tracking
**Description**: Guru dapat input dan track kehadiran siswa untuk setiap jadwal mengajar.

**Actors**: Guru (Teacher)

**Detailed Requirements**:
- FR-5.1: Guru dapat melihat jadwal mengajar mereka di dashboard
- FR-5.2: Guru dapat membuka form input absensi untuk kelas yang sedang diajar
- FR-5.3: Form absensi menampilkan list siswa dengan status: Hadir, Sakit, Izin, Alfa
- FR-5.4: Guru dapat menyimpan absensi untuk setiap pertemuan
- FR-5.5: Guru dapat mengubah absensi yang sudah diinput (dalam periode tertentu)
- FR-5.6: Sistem harus mencatat waktu input dan last update untuk setiap absensi
- FR-5.7: Guru dapat melihat history absensi kelas mereka
- FR-5.8: Sistem harus validate bahwa semua siswa di kelas memiliki status absensi

**Acceptance Criteria**:
- ✓ Form absensi menampilkan semua siswa dalam kelas
- ✓ Absensi dapat disimpan dengan status valid
- ✓ History absensi dapat ditampilkan dengan filter tanggal
- ✓ User dapat mengubah absensi dalam periode yang diizinkan

---

### FR-6: Reporting & Analytics
**Description**: Admin dapat membuat laporan dan analisis absensi dengan berbagai filter dan export format.

**Actors**: Admin

**Detailed Requirements**:
- FR-6.1: Admin dapat melihat laporan absensi dengan filter: tanggal, kelas, siswa
- FR-6.2: Admin dapat melihat statistik absensi: total hadir, sakit, izin, alfa per kelas
- FR-6.3: Admin dapat melihat trend absensi siswa (bulanan/tahunan)
- FR-6.4: Admin dapat export laporan absensi ke format CSV/PDF
- FR-6.5: Sistem harus menampilkan alert jika absensi siswa melampaui threshold (misal: 10 alfa)
- FR-6.6: Admin dapat melihat detail historis perubahan absensi (audit trail)

**Acceptance Criteria**:
- ✓ Laporan absensi dapat difilter berdasarkan kriteria yang dipilih
- ✓ Export laporan menghasilkan file dengan format yang benar dan lengkap
- ✓ Statistik ditampilkan dengan visualisasi yang jelas (chart/table)
- ✓ Alert ditampilkan untuk siswa dengan absensi tinggi

---

## 3. NON-FUNCTIONAL REQUIREMENTS (NFR)

### NFR-1: Performance
**Requirement**: Sistem harus responsif dan cepat dalam memproses data.

**Metrics**:
- NFR-1.1: Response time untuk halaman list (< 500ms) untuk 1000 records
- NFR-1.2: Response time untuk report generation (< 2 detik) untuk data 1 tahun
- NFR-1.3: Export CSV untuk 5000 records (< 3 detik)
- NFR-1.4: Database query harus menggunakan index yang tepat
- NFR-1.5: Implementasi pagination untuk list yang menampilkan lebih dari 100 items

**Testing Approach**:
- Load testing menggunakan JMeter dengan 100 concurrent users
- Response time monitoring menggunakan browser DevTools

---

### NFR-2: Scalability
**Requirement**: Sistem harus dapat menangani pertumbuhan data dan user.

**Metrics**:
- NFR-2.1: Database schema harus mendukung pertumbuhan hingga 10,000 siswa, 500 guru, 1,000,000 absensi records
- NFR-2.2: Sistem harus dapat menangani 500 concurrent users tanpa degradasi signifikan
- NFR-2.3: Implementasi database indexing untuk query optimization
- NFR-2.4: Implementasi connection pooling untuk database connections

**Architecture Considerations**:
- Horizontal scaling: Load balancer untuk mendistribusikan traffic
- Vertical scaling: Database query optimization dan caching
- Cache layer: Redis untuk menyimpan session dan query results
- Database sharding untuk attendance records (by semester/year)
- Microservices separation untuk reporting module

---

### NFR-3: Security
**Requirement**: Sistem harus melindungi data sensitif dan mencegah serangan umum.

**Metrics**:
- NFR-3.1: Password harus dienkripsi menggunakan bcrypt dengan salt minimum 10 rounds
- NFR-3.2: All input harus di-sanitasi untuk mencegah SQL injection dan XSS attacks
- NFR-3.3: Implementasi CSRF protection menggunakan token
- NFR-3.4: Session harus secure (HttpOnly, Secure flag untuk HTTPS)
- NFR-3.5: Audit logging untuk semua operasi CRUD dan login attempts
- NFR-3.6: Rate limiting untuk login attempts (max 5 attempts per 15 menit)
- NFR-3.7: Data sensitive (password, NIS) tidak boleh di-log dalam plain text

**Testing Approach**:
- Security vulnerability scanning menggunakan OWASP ZAP
- Penetration testing untuk common vulnerabilities (SQL injection, XSS, CSRF)
- Code review untuk security best practices

---

### NFR-4: Reliability & Availability
**Requirement**: Sistem harus reliable dan tersedia untuk diakses.

**Metrics**:
- NFR-4.1: System uptime minimum 99% per bulan (max 7.2 jam downtime)
- NFR-4.2: Recovery time objective (RTO) < 2 jam
- NFR-4.3: Recovery point objective (RPO) < 1 jam (backup frequency)
- NFR-4.4: Database harus memiliki redundancy dan replication
- NFR-4.5: Error handling yang proper dengan user-friendly error messages

**Disaster Recovery**:
- Automated database backup setiap jam
- Backup replication ke secondary server
- Rollback capability untuk failed updates

---

### NFR-5: Usability
**Requirement**: Interface harus user-friendly dan mudah digunakan.

**Metrics**:
- NFR-5.1: Interface harus responsive dan mobile-friendly
- NFR-5.2: Navigation harus intuitif dengan breadcrumb
- NFR-5.3: Form harus memiliki validation feedback yang jelas
- NFR-5.4: Aksesibilitas WCAG 2.1 Level AA compliance
- NFR-5.5: Page load time < 3 detik di network 4G

**Testing Approach**:
- Usability testing dengan end-users
- Accessibility testing menggunakan Axe atau WAVE
- Responsive design testing di berbagai devices

---

### NFR-6: Maintainability
**Requirement**: Code harus mudah dirawat dan di-maintain.

**Metrics**:
- NFR-6.1: Code harus follow coding standards (PSR-12 untuk PHP)
- NFR-6.2: Minimum code coverage 70% untuk unit tests
- NFR-6.3: Documentation lengkap untuk API dan database schema
- NFR-6.4: Version control menggunakan Git dengan meaningful commit messages
- NFR-6.5: Automated testing (unit, integration) dalam CI/CD pipeline

---

## 4. ASSUMPTIONS & DEPENDENCIES

### Assumptions
1. Sistem diakses melalui web browser modern (Chrome, Firefox, Safari, Edge)
2. Server menggunakan PHP 7.4+ dan MySQL 5.7+
3. Database connection stable dan reliable
4. User memiliki basic understanding dalam menggunakan web aplikasi
5. Network bandwidth mencukup untuk transfer data

### Dependencies
1. MySQLi extension untuk PHP
2. mbstring extension untuk character encoding
3. Date extension untuk date/time handling
4. PDO atau MySQLi untuk database abstraction
5. Session management extension

---

## 5. CONSTRAINTS

1. **Technology Stack**: Menggunakan PHP Native (no framework restrictions mentioned)
2. **Browser Compatibility**: Harus support IE 11+ dan modern browsers
3. **Database**: MySQL 5.7+
4. **Data Storage**: Max file size untuk upload 10MB
5. **Compliance**: Harus comply dengan data protection regulations (GDPR-like)

---

## 6. USE CASES

### UC-1: Login
**Actors**: User (Admin/Guru)
**Flow**: User enters credentials → System validates → Access granted/denied

### UC-2: Add Student
**Actors**: Admin
**Flow**: Click Add → Fill form → Validate → Save → Display confirmation

### UC-3: Input Attendance
**Actors**: Guru
**Flow**: View schedule → Click Input → Select status for each student → Save → Confirmation

### UC-4: View Attendance Report
**Actors**: Admin
**Flow**: Navigate to Reports → Select filters → Generate report → View/Export

---

## 7. PRIORITY MATRIX

| Requirement | Priority | Complexity |
|-------------|----------|-----------|
| FR-1: Authentication | HIGH | Medium |
| FR-2: User Management | HIGH | Medium |
| FR-3: Student Management | HIGH | Low |
| FR-4: Teacher Management | HIGH | Low |
| FR-5: Attendance System | CRITICAL | High |
| FR-6: Reporting & Analytics | HIGH | High |
| NFR-1: Performance | HIGH | High |
| NFR-2: Scalability | MEDIUM | High |
| NFR-3: Security | CRITICAL | High |

---

## 8. GLOSSARY

| Term | Definition |
|------|-----------|
| NIS | Nomor Induk Siswa (Student ID) |
| NISN | Nomor Induk Siswa Nasional (National Student ID) |
| NIP | Nomor Induk Pegawai (Teacher ID) |
| RBAC | Role-Based Access Control |
| CSRF | Cross-Site Request Forgery |
| XSS | Cross-Site Scripting |
| RTO | Recovery Time Objective |
| RPO | Recovery Point Objective |

---

## SIGN-OFF

| Role | Name | Date | Signature |
|------|------|------|-----------|
| Business Analyst | | | |
| Project Manager | | | |
| Quality Assurance Lead | | | |
| Development Lead | | | |

---

**Document Control**:
- Version 1.0 - Initial Draft - 2026-08-10
