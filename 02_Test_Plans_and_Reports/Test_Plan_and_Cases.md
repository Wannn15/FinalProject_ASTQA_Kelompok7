# Test Plan & Test Cases
## Sistem Informasi Sekolah dan Absensi

**Document Version**: 1.0  
**Date**: August 10, 2026  
**Author**: [QA Team]  
**Status**: Draft

---

## 1. TEST PLAN OVERVIEW

### 1.1 Test Scope

**In Scope**:
- Unit testing for core business logic
- Integration testing for module interactions
- System testing for end-to-end workflows
- User acceptance testing (UAT)
- Performance & load testing
- Security testing (OWASP Top 10)

**Out of Scope**:
- Third-party library testing
- Operating system testing
- Hardware compatibility testing

### 1.2 Testing Levels

| Level | Type | Tool | Coverage | Schedule |
|-------|------|------|----------|----------|
| Unit | White-box | PHPUnit | 70%+ | Week 1 |
| Integration | Gray-box | PHPUnit + Custom | 60%+ | Week 2 |
| System | Black-box | Postman, Manual | 80%+ | Week 2-3 |
| UAT | Black-box | Manual | 100% | Week 3-4 |
| Performance | Load | JMeter | All critical paths | Week 4 |

---

## 2. UNIT TESTING

### 2.1 Unit Test Cases - Authentication

#### UT-AUTH-001: Valid Login
**Function**: `validateLogin($username, $password)`
**Input**:
```
username = "admin"
password = "admin123"
```
**Expected Output**:
```
[
  'success' => true,
  'user_id' => 1,
  'role' => 'Admin'
]
```
**Test Condition**: Credentials match database and password hash verified

---

#### UT-AUTH-002: Invalid Password
**Function**: `validateLogin($username, $password)`
**Input**:
```
username = "admin"
password = "wrongpassword"
```
**Expected Output**:
```
[
  'success' => false,
  'error' => 'Invalid credentials'
]
```
**Test Condition**: Password does not match hash

---

#### UT-AUTH-003: Non-existent User
**Function**: `validateLogin($username, $password)`
**Input**:
```
username = "nonexistent"
password = "anypassword"
```
**Expected Output**:
```
[
  'success' => false,
  'error' => 'User not found'
]
```
**Test Condition**: Username not in database

---

#### UT-AUTH-004: Password Hashing
**Function**: `hashPassword($password)`
**Input**:
```
password = "TestPass123"
```
**Expected Output**:
```
Hash string starting with $2y$ (bcrypt)
Length >= 60 characters
```
**Test Condition**: Uses bcrypt with salt, different each time

---

### 2.2 Unit Test Cases - Student Management

#### UT-STUDENT-001: Add Valid Student
**Function**: `addStudent($data)`
**Input**:
```php
$data = [
  'name' => 'Budi Santoso',
  'nis' => '2024001',
  'nisn' => '123456789',
  'date_of_birth' => '2010-05-15',
  'class_id' => 1
]
```
**Expected Output**:
```
[
  'success' => true,
  'student_id' => 1,
  'nis' => '2024001'
]
```
**Test Condition**: All required fields valid, NIS unique

---

#### UT-STUDENT-002: Duplicate NIS
**Function**: `addStudent($data)`
**Input**:
```php
$data = [
  'name' => 'Another Student',
  'nis' => '2024001', // Duplicate
  'class_id' => 1
]
```
**Expected Output**:
```
[
  'success' => false,
  'error' => 'NIS already exists'
]
```
**Test Condition**: NIS already in database

---

#### UT-STUDENT-003: Missing Required Field
**Function**: `addStudent($data)`
**Input**:
```php
$data = [
  'name' => 'Test Student',
  // 'nis' is missing
  'class_id' => 1
]
```
**Expected Output**:
```
[
  'success' => false,
  'error' => 'NIS is required'
]
```
**Test Condition**: Required field validation

---

#### UT-STUDENT-004: Invalid Date Format
**Function**: `addStudent($data)`
**Input**:
```php
$data = [
  'name' => 'Test Student',
  'nis' => '2024002',
  'date_of_birth' => 'invalid-date',
  'class_id' => 1
]
```
**Expected Output**:
```
[
  'success' => false,
  'error' => 'Invalid date format'
]
```
**Test Condition**: Date validation

---

### 2.3 Unit Test Cases - Attendance Management

#### UT-ATTEND-001: Valid Attendance Input
**Function**: `recordAttendance($schedule_id, $student_id, $status, $date)`
**Input**:
```php
schedule_id = 5
student_id = 1
status = 'Hadir'
date = '2024-08-10'
```
**Expected Output**:
```
[
  'success' => true,
  'attendance_id' => 1,
  'status' => 'Hadir'
]
```
**Test Condition**: All parameters valid, no duplicate record

---

#### UT-ATTEND-002: Invalid Status
**Function**: `recordAttendance($schedule_id, $student_id, $status, $date)`
**Input**:
```php
status = 'Invalid' // Not in enum
```
**Expected Output**:
```
[
  'success' => false,
  'error' => 'Status must be one of: Hadir, Sakit, Izin, Alfa'
]
```
**Test Condition**: Status validation against enum

---

#### UT-ATTEND-003: Duplicate Attendance Record
**Function**: `recordAttendance($schedule_id, $student_id, $status, $date)`
**Input**:
```php
// Record already exists for same schedule, student, date
```
**Expected Output**:
```
[
  'success' => false,
  'error' => 'Attendance record already exists'
]
```
**Test Condition**: Unique constraint check

---

#### UT-ATTEND-004: Calculate Attendance Percentage
**Function**: `calculateAttendancePercentage($student_id, $start_date, $end_date)`
**Input**:
```php
student_id = 1
start_date = '2024-01-01'
end_date = '2024-08-10'
// Student has: 18 Hadir, 2 Sakit/Izin, 0 Alfa out of 20 sessions
```
**Expected Output**:
```
[
  'percentage' => 90,
  'total_sessions' => 20,
  'hadir' => 18,
  'absent' => 0
]
```
**Test Condition**: Percentage calculation accuracy

---

### 2.4 Unit Test Cases - Validation & Security

#### UT-SEC-001: SQL Injection Prevention
**Function**: `getStudentByNIS($nis)`
**Input**:
```php
$nis = "2024001' OR '1'='1"
```
**Expected Output**:
```
[
  'success' => false,
  'error' => 'No student found'
]
```
**Test Condition**: Parameterized query prevents injection

---

#### UT-SEC-002: XSS Prevention
**Function**: `displayStudentName($name)`
**Input**:
```php
$name = "<script>alert('XSS')</script>"
```
**Expected Output**:
```
HTML-escaped output:
&lt;script&gt;alert('XSS')&lt;/script&gt;
```
**Test Condition**: HTML special chars escaped

---

#### UT-SEC-003: Password Strength Validation
**Function**: `validatePasswordStrength($password)`
**Input Cases**:
```
"short" → false (min 8 chars)
"12345678" → false (requires letters and numbers)
"Password123" → true (letters, numbers, 8+ chars)
```
**Expected Output**: Boolean validation result

---

### 2.5 Code Coverage Target

**Target Coverage**: 70% minimum

**Coverage Breakdown**:
- Authentication: 80%
- User Management: 75%
- Student Management: 70%
- Teacher Management: 70%
- Attendance: 80%
- Validation: 85%

---

## 3. INTEGRATION TESTING

### 3.1 Integration Test Cases

#### IT-001: User Login → Dashboard Load
**Modules Involved**: Authentication → Session → Dashboard Query → Database

**Test Steps**:
1. Send login request with valid credentials
2. Verify session created
3. Fetch user dashboard data
4. Verify data returned correctly

**Expected Results**:
- ✓ Login successful
- ✓ Session stored in database
- ✓ Dashboard displays correct user data
- ✓ User statistics calculated correctly

---

#### IT-002: Add Student → Database Save → List Display
**Modules Involved**: Student Input Form → Validation → Database → List Query

**Test Steps**:
1. Submit form with valid student data
2. Verify data saved in database
3. Query student list
4. Verify new student appears in list

**Expected Results**:
- ✓ Record inserted in `students` table
- ✓ Foreign key constraint satisfied (class_id valid)
- ✓ List query returns new record
- ✓ Pagination correctly updated

---

#### IT-003: Teacher Input Attendance → Database Save → Report Generate
**Modules Involved**: Attendance Input → Validation → Database → Report Query

**Test Steps**:
1. Teacher submits attendance for multiple students
2. Verify all records saved in attendance table
3. Generate attendance report for same date
4. Verify report includes all submitted records

**Expected Results**:
- ✓ All attendance records inserted
- ✓ Unique constraint prevents duplicates
- ✓ Report shows correct counts (Hadir, Sakit, Izin, Alfa)
- ✓ Audit log captures all changes

---

#### IT-004: User Permission Check → Module Access
**Modules Involved**: Authentication → Authorization → Module Controller

**Test Steps**:
1. Login as Teacher
2. Attempt to access Admin-only page (User Management)
3. Verify access denied
4. Login as Admin
5. Attempt same access
6. Verify access granted

**Expected Results**:
- ✓ Teacher gets 403 Forbidden
- ✓ Admin gets 200 OK with data
- ✓ Audit log records access attempts

---

#### IT-005: Class Schedule → Attendance Input → Validation
**Modules Involved**: Schedule Query → Attendance Form → Student List → Validation

**Test Steps**:
1. Query schedule for specific date
2. Load attendance form for that schedule
3. Verify all students in class listed
4. Submit attendance with missing student status
5. Verify validation error

**Expected Results**:
- ✓ Form shows all students from class
- ✓ Validation prevents incomplete submissions
- ✓ Error message clear and actionable

---

### 3.2 Data Flow Integration Tests

#### IT-006: Cross-Module Data Integrity
**Test**: Deleting a class should handle student records properly

**Steps**:
1. Create class with 10 students
2. Create schedules for that class
3. Input attendance records for those schedules
4. Attempt to delete the class
5. Verify cascading delete or soft delete behavior

**Expected Results**:
- ✓ Students remain active but class_id handled
- ✓ Attendance records preserved for history
- ✓ Integrity constraints maintained

---

## 4. SYSTEM TESTING

### 4.1 System Test Scenarios

#### ST-001: Complete User Workflow - Admin User Management
**Objective**: Test entire user management workflow from creation to deletion

**Test Steps**:
1. Admin logs in
2. Navigate to User Management
3. Add new teacher user (guru001)
4. Search for newly created user
5. Edit user email
6. Toggle active/inactive status
7. Delete user
8. Verify user no longer in active list

**Expected Results**:
- ✓ All CRUD operations successful
- ✓ Search functionality works
- ✓ Status changes reflected immediately
- ✓ Soft delete removes from active list
- ✓ Audit trail recorded

---

#### ST-002: Complete Workflow - Teacher Attendance Input
**Objective**: Test complete attendance workflow

**Test Steps**:
1. Teacher logs in
2. View today's schedule
3. Select class to input attendance
4. Mark attendance for all students
5. Submit attendance
6. Verify saved in database
7. View history of attendance
8. Edit previous attendance record
9. Verify changes saved

**Expected Results**:
- ✓ Schedule displayed correctly
- ✓ Attendance form shows all students
- ✓ Can mark with different statuses
- ✓ Data persisted in database
- ✓ Edit functionality works
- ✓ Audit log shows changes

---

#### ST-003: Complete Workflow - Attendance Reporting
**Objective**: Test reporting workflow

**Test Steps**:
1. Admin logs in
2. Navigate to Reports
3. Generate attendance report with filters:
   - Date range: 1-31 August 2024
   - Class: 7A
4. View summary statistics
5. View detailed student records
6. Export to CSV
7. Verify exported file format and content

**Expected Results**:
- ✓ Filters applied correctly
- ✓ Statistics calculated accurately
- ✓ Detailed view shows all records
- ✓ CSV exports with proper format
- ✓ File downloadable and valid

---

### 4.2 End-to-End Test Scenarios

#### ST-004: Multi-Day Attendance Tracking
**Duration**: 5-day simulation

**Scenario**:
- Day 1: Input attendance for Class 7A (20 students) × 4 subjects = 80 records
- Day 2: Input attendance for Class 7A + 7B = 160 records
- Day 3-5: Continue pattern
- Generate weekly report

**Validations**:
- ✓ All 400 records stored correctly
- ✓ Report calculations accurate
- ✓ No data loss or duplication
- ✓ System performance acceptable
- ✓ Audit trail complete

---

### 4.3 Negative Test Scenarios

#### ST-005: Invalid Data Submission
**Test**: Submit attendance with invalid data

**Test Cases**:
1. Date in future → Reject
2. Non-existent student_id → Reject
3. Invalid status → Reject
4. Missing required fields → Reject
5. Duplicate record → Reject

**Expected**: All cases rejected with proper error messages

---

#### ST-006: Concurrent Operations
**Test**: Two users simultaneously accessing same data

**Scenario**:
1. Teacher A opens attendance form for Class 7A
2. Teacher B (of same class) opens attendance form
3. Teacher A submits attendance
4. Teacher B submits attendance (should detect conflict)

**Expected**: Proper locking/error handling, no data corruption

---

---

## 5. ACCEPTANCE TESTING (UAT)

### 5.1 UAT Sign-Off Sheet Template

| # | Test Case | Requirement | Expected Result | Actual Result | Pass/Fail | Notes | Tester | Date |
|----|-----------|-------------|-----------------|----------------|-----------|-------|--------|------|
| 1 | User can login with valid credentials | FR-1.1 | Login successful, redirected to dashboard | | [ ] | | | |
| 2 | Invalid login shows error message | FR-1.1 | Error message displayed | | [ ] | | | |
| 3 | Admin can add new student | FR-3.1 | Student added to system | | [ ] | | | |
| 4 | Student list searchable by name | FR-3.2 | Results filtered correctly | | [ ] | | | |
| 5 | Teacher can input attendance | FR-5.2 | Attendance saved successfully | | [ ] | | | |
| 6 | Attendance report generates | FR-6.1 | Report displays correct data | | [ ] | | | |
| 7 | Report exportable to CSV | FR-6.4 | CSV file valid and complete | | [ ] | | | |
| 8 | User permissions enforced | FR-1.3 | Teacher cannot access admin functions | | [ ] | | | |
| 9 | Session timeout works | FR-1.4 | User logged out after 30 min inactivity | | [ ] | | | |
| 10 | System responsive on slow connection | NFR-1.2 | Page loads acceptably | | [ ] | | | |

### 5.2 UAT Checklist

**Business Requirements**:
- [ ] All 6 functional requirements implemented
- [ ] All non-functional requirements met (performance, security, scalability)
- [ ] User documentation provided
- [ ] User training completed

**System Stability**:
- [ ] No critical bugs remaining
- [ ] No data loss observed
- [ ] System handles errors gracefully
- [ ] Audit logging working

**Performance**:
- [ ] Page load times acceptable
- [ ] Report generation fast enough
- [ ] Export operations complete timely
- [ ] No timeout issues

**Security**:
- [ ] Login secure and validated
- [ ] Passwords properly encrypted
- [ ] Session management working
- [ ] SQL injection/XSS prevented

**Usability**:
- [ ] Interface intuitive
- [ ] Error messages clear
- [ ] Navigation logical
- [ ] Forms easy to complete

---

## 6. PERFORMANCE & LOAD TESTING

### 6.1 Performance Test Scenarios (JMeter)

#### PT-001: Dashboard Load - 100 Concurrent Users
**Objective**: Test dashboard performance under load

**Parameters**:
- Concurrent Users: 100
- Ramp-up Time: 10 seconds
- Loop Count: 5
- Think Time: 2000ms

**Expected Results**:
- Response time < 500ms (average)
- 95th percentile < 1000ms
- Error rate < 1%
- Throughput > 10 requests/sec

---

#### PT-002: Attendance Report Generation - Load Test
**Objective**: Test report generation with concurrent users

**Parameters**:
- Concurrent Users: 50
- Report Date Range: 1 month (1000+ records)
- Ramp-up: 5 seconds
- Loop: 3

**Expected Results**:
- Response time < 2000ms
- Database connections stable
- No memory leaks
- Error rate < 1%

---

### 6.2 Stress Testing

#### ST-STRESS-001: Maximum Concurrent Users
**Objective**: Find breaking point of system

**Test Method**:
1. Start with 100 concurrent users
2. Increase by 50 every minute
3. Monitor response time and error rate
4. Stop when error rate > 5%

**Expected Behavior**:
- System stable up to 500 concurrent users
- Graceful degradation beyond capacity
- No data corruption under stress

---

## 7. SECURITY TESTING

### 7.1 OWASP Top 10 Test Cases

#### SEC-001: SQL Injection
**Method**: Test input fields with SQL injection payload
```
Payload: " OR '1'='1"
```
**Expected**: Query parameterized, injection prevented

#### SEC-002: Cross-Site Scripting (XSS)
**Method**: Test input with JavaScript
```
Payload: <script>alert('XSS')</script>
```
**Expected**: Script escaped, not executed

#### SEC-003: CSRF Token Validation
**Method**: Submit form without CSRF token
**Expected**: Request rejected with 403 error

#### SEC-004: Authentication Bypass
**Method**: Attempt to access pages without login
**Expected**: Redirect to login page

#### SEC-005: Broken Access Control
**Method**: Teacher tries to access Admin functions
**Expected**: 403 Forbidden error

#### SEC-006: Sensitive Data Exposure
**Method**: Check if passwords logged in plain text
**Expected**: Never logged in plain text, only hashes

---

## 8. AUTOMATION TESTING

### 8.1 Automated Test Tools

| Testing Type | Tool | Format | Coverage |
|--------------|------|--------|----------|
| Unit Testing | PHPUnit | .php test files | 70%+ |
| Integration | PHPUnit + Custom | .php test files | 60%+ |
| API Testing | Postman | .json collection | 80%+ |
| Load Testing | JMeter | .jmx test plans | Critical paths |
| UI/E2E Testing | Cypress/Selenium | .js/.java | Main workflows |

### 8.2 CI/CD Integration

**Automated Tests in Pipeline**:
1. Unit Tests (PHPUnit) - 5 min
2. Integration Tests - 10 min
3. Code Quality (SonarQube) - 5 min
4. Security Scan (SonarQube) - 3 min
5. Smoke Tests - 5 min
6. Performance Baseline - 10 min

**Total CI/CD Time**: ~40 minutes

---

## 9. TEST EXECUTION SCHEDULE

| Phase | Start Date | End Date | Duration | Resources |
|-------|-----------|----------|----------|-----------|
| Unit Testing | Week 1, Day 1 | Week 1, Day 3 | 3 days | 2 QA |
| Integration Testing | Week 1, Day 4 | Week 2, Day 2 | 4 days | 3 QA |
| System Testing | Week 2, Day 3 | Week 3, Day 2 | 5 days | 4 QA |
| UAT | Week 3, Day 3 | Week 4, Day 2 | 5 days | 2 QA + Users |
| Performance Test | Week 4, Day 1 | Week 4, Day 3 | 3 days | 1 QA |

---

## 10. DEFECT TRACKING & REPORTING

### 10.1 Severity Levels

| Severity | Definition | Resolution Time |
|----------|-----------|-----------------|
| Critical | System down / Data loss | 2 hours |
| High | Major feature broken | 8 hours |
| Medium | Feature partially working | 24 hours |
| Low | Minor issue / Enhancement | 72 hours |

### 10.2 Test Summary Report

**Test Results Dashboard**:
```
Total Test Cases: 50
Passed: 48 (96%)
Failed: 2 (4%)
Blocked: 0 (0%)

By Category:
- Functional: 18/20 passed
- Performance: 10/10 passed
- Security: 8/10 passed
- Usability: 12/12 passed

Code Coverage: 72%
Critical Issues: 0
High Issues: 2
Medium Issues: 5
Low Issues: 8
```

---

## 11. ENTRY & EXIT CRITERIA

### 11.1 Entry Criteria
- [ ] Test plan approved
- [ ] Test environment ready
- [ ] Build deployed to test environment
- [ ] Test data prepared
- [ ] Test team trained

### 11.2 Exit Criteria
- [ ] 95%+ test cases passed
- [ ] 0 critical/high severity issues remaining
- [ ] Code coverage >= 70%
- [ ] Performance SLA met
- [ ] Security test passed
- [ ] UAT sign-off obtained

---

**Document Control**:
- Version 1.0 - Initial Draft - 2026-08-10

**Sign-off**:
| Role | Name | Date | Signature |
|------|------|------|-----------|
| QA Lead | | | |
| Project Manager | | | |
| Business Owner | | | |
