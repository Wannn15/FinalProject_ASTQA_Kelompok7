# Sistem Informasi Sekolah dan Absensi

Sistem manajemen informasi sekolah dan absensi berbasis PHP Native dengan MySQLi.

## Fitur

### 1. Manajemen Pengguna (Admin)
- Tambah, edit, hapus pengguna
- Tipe pengguna: Admin dan Guru (Guru)
- Password enkripsi bcrypt
- Status aktif/nonaktif

### 2. Data Master (Admin)
- **Siswa**: Manajemen data siswa dengan NIS/NISN
- **Guru**: Manajemen data guru dengan NIP
- **Kelas**: Manajemen kelas dengan tingkat (7-9) dan wali kelas
- **Mata Pelajaran**: Manajemen mata pelajaran
- **Jadwal**: Manajemen jadwal mengajar guru

### 3. Sistem Absensi
- **Guru**: Input absensi untuk setiap jadwal mengajar
- **Admin**: Laporan absensi dengan filter tanggal dan kelas
- Status absensi: Hadir, Sakit, Izin, Alfa
- Fitur cetak dan export CSV

### 4. Dashboard
- **Admin**: Statistik pengguna, siswa, guru, kelas, mata pelajaran, absensi hari ini
- **Guru**: Jadwal mengajar dan absensi terbaru

## Instalasi

### Prerequisites
- PHP 7.4+
- MySQL 5.7+
- Web Server (Apache, Nginx, dll)

### Setup

1. **Clone atau copy project ke folder web server**
   ```
   cp -r sistem_absensi /var/www/html/
   cd /var/www/html/sistem_absensi
   ```

2. **Import database**
   - Buka MySQL client
   - Execute file `database/database.sql`:
   ```sql
   mysql -u username -p < database/database.sql
   ```
   atau melalui phpMyAdmin

3. **Konfigurasi database** (jika diperlukan)
   - Edit `config/database.php`
   - Sesuaikan: HOST, USERNAME, PASSWORD, DATABASE

4. **Akses aplikasi**
   - Buka browser: `http://localhost/sistem_absensi`
   - Redirect otomatis ke login page

## Default Users

Database sudah dilengkapi dengan 2 user default:

### Admin
- **Username**: admin
- **Password**: admin123
- **Role**: Admin

### Guru
- **Username**: guru
- **Password**: guru123
- **Role**: Guru

> **⚠️ PENTING**: Ubah password setelah login pertama kali!

## Struktur Folder

```
sistem_absensi/
├── index.php                 # Landing page / redirect
├── auth/
│   ├── login.php            # Login form
│   ├── process_login.php    # Process login
│   └── logout.php           # Logout
├── admin/
│   ├── dashboard.php        # Admin dashboard
│   ├── users/               # User CRUD
│   ├── students/            # Student CRUD + Export CSV
│   ├── teachers/            # Teacher CRUD
│   ├── classes/             # Class CRUD
│   ├── subjects/            # Subject CRUD
│   ├── schedules/           # Schedule CRUD
│   └── reports/             # Attendance reports
├── teacher/
│   ├── dashboard.php        # Teacher dashboard
│   └── attendance/
│       ├── index.php        # List schedules
│       ├── input.php        # Input attendance
│       └── history.php      # Attendance history
├── config/
│   ├── database.php         # MySQLi connection & helpers
│   └── auth.php             # Auth functions
├── templates/
│   ├── header.php           # HTML head + CSS
│   ├── navbar.php           # Top navigation
│   ├── sidebar_admin.php    # Admin sidebar menu
│   ├── sidebar_teacher.php  # Teacher sidebar menu
│   └── footer.php           # Footer + JS
├── assets/
│   ├── css/
│   │   └── style.css        # Custom CSS
│   └── js/
│       └── script.js        # Custom JS
└── database/
    └── database.sql         # Database schema
```

## URL Routes

### Admin Routes
- `/admin/dashboard.php` - Dashboard
- `/admin/users/` - User management
- `/admin/students/` - Student management
- `/admin/teachers/` - Teacher management
- `/admin/classes/` - Class management
- `/admin/subjects/` - Subject management
- `/admin/schedules/` - Schedule management
- `/admin/reports/` - Attendance reports

### Teacher Routes
- `/teacher/dashboard.php` - Dashboard
- `/teacher/attendance/index.php` - My schedules
- `/teacher/attendance/input.php` - Input attendance
- `/teacher/attendance/history.php` - Attendance history

### Auth Routes
- `/auth/login.php` - Login page
- `/auth/logout.php` - Logout

## Fitur Keamanan

1. **Authentication**: Session-based login dengan password bcrypt
2. **Authorization**: Role-based access control (RBAC)
   - Admin: Akses ke semua halaman admin
   - Guru: Akses ke dashboard guru dan input absensi
3. **Input Sanitization**: htmlspecialchars() + prepared statements
4. **SQL Injection Prevention**: Prepared statements (MySQLi)
5. **CSRF Protection**: Session token validation

## Database Schema

### Tabel Utama

**users**
- Autentikasi pengguna
- Relasi: 1-to-many dengan teachers (guru)

**teachers**
- Data guru
- Relasi: many-to-many dengan subjects via schedules
- Relasi: one-to-many ke classes (wali_kelas)

**students**
- Data siswa
- Relasi: many-to-one ke classes

**classes**
- Data kelas
- Relasi: one-to-many ke students
- Relasi: many-to-many ke teachers via schedules

**subjects**
- Data mata pelajaran
- Relasi: many-to-many ke teachers via schedules

**schedules**
- Jadwal mengajar
- Relasi: many-to-one ke teachers, subjects, classes

**attendance_sessions**
- Session absensi per jadwal
- Unique: (schedule_id, tanggal)
- Relasi: one-to-many ke attendance_details

**attendance_details**
- Detail absensi siswa
- Unique: (session_id, student_id)
- Relasi: many-to-one ke students

## Panduan Penggunaan

### Untuk Admin

1. **Login** dengan akun admin
2. **Buat master data** terlebih dahulu:
   - Tambah Guru
   - Tambah Kelas
   - Tambah Mata Pelajaran
   - Tambah Siswa ke setiap kelas
   - Buat Jadwal mengajar
3. **Manajemen User**: Tambah akun guru dengan link ke data guru
4. **Lihat Laporan Absensi**: Dari menu Absensi → Laporan Absensi

### Untuk Guru

1. **Login** dengan akun guru
2. **Lihat Dashboard**: Statistik jadwal dan absensi
3. **Input Absensi**:
   - Pilih jadwal dari "Jadwal Saya"
   - Pilih tanggal
   - Input status setiap siswa
   - Simpan
4. **Lihat Riwayat**: Daftar absensi yang sudah diinput

## Troubleshooting

### Login gagal
- Pastikan database sudah diimport
- Periksa config/database.php sudah sesuai
- Reset password melalui database:
  ```sql
  UPDATE users SET password = PASSWORD('newpassword') WHERE username = 'admin';
  ```

### Halaman white screen
- Cek error di `error_log` web server
- Pastikan PHP error_reporting aktif di config/database.php

### Data tidak muncul
- Pastikan database.sql sudah diimport dengan benar
- Periksa koneksi database di phpMyAdmin

## Support

Untuk bantuan atau pertanyaan, hubungi tim development.

---

**Versi**: 1.0  
**Last Updated**: 2024  
**Built with**: PHP Native + MySQLi + Bootstrap 5
