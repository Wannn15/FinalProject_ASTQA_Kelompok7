# DEFECT REPORT TEMPLATE
## Sistem Informasi Sekolah dan Absensi

---

## DEFECT REPORT #[ID]

**Report Date**: [Date]  
**Reported By**: [Name]  
**Priority**: [Critical/High/Medium/Low]  
**Status**: [Open/In Progress/Fixed/Verified]

---

## 1. DEFECT INFORMATION

### Basic Details

| Field | Value |
|-------|-------|
| **Defect ID** | [DEF-XXX-YYYY] |
| **Title** | [Brief description of issue] |
| **Component** | [Module affected: Login, Students, Attendance, Reports, etc.] |
| **Found In** | [Version/Build number] |
| **Severity** | [Critical/High/Medium/Low] |
| **Status** | [Open/In Progress/Fixed/Verified/Closed] |
| **Assigned To** | [Developer name] |
| **Date Reported** | [Date] |
| **Date Fixed** | [Date] |
| **Date Verified** | [Date] |

### Environment

| Item | Details |
|------|---------|
| **Operating System** | Windows 10 / Ubuntu 20.04 / macOS |
| **Browser** | Chrome 96, Firefox 95, Safari 15 |
| **PHP Version** | 8.1.x |
| **MySQL Version** | 8.0.35 |
| **Server** | Apache 2.4 / PHP Built-in |
| **Database** | school_attendance |

---

## 2. DEFECT DESCRIPTION

### Summary

[One-line summary of the issue]

Example: "CSV export shows corrupted data with special characters"

### Detailed Description

[Provide complete details about what the issue is]

**Background**:
- What module or feature is affected?
- What was the user trying to do?
- What happened instead of the expected behavior?

**Step-by-step description of issue**:
1. [Step 1]
2. [Step 2]
3. [Step 3]
4. [Result of issue]

---

## 3. STEPS TO REPRODUCE

### Prerequisite Conditions

- [ ] User logged in as [Role]
- [ ] Test data: [Specific students/records available]
- [ ] System in specific state: [Describe]

### Exact Steps

1. Navigate to [URL or menu path]
2. Click on [Element]
3. Enter [Data]
4. Select [Option]
5. Submit [Form/Request]
6. [Issue occurs]

### Expected Result

[What should have happened according to requirements]

### Actual Result

[What actually happened - include error message if any]

---

## 4. SEVERITY & IMPACT ASSESSMENT

### Severity Level

- **Critical**: System down, data loss, security breach
- **High**: Major feature broken, workaround difficult
- **Medium**: Feature partially broken, minor workaround available
- **Low**: Minor cosmetic issue, no functional impact

**Selected Level**: [Critical/High/Medium/Low]

### Business Impact

| Aspect | Impact |
|--------|--------|
| **Functional Impact** | [Describes which functions are affected] |
| **User Impact** | [How many users affected, role affected] |
| **Data Impact** | [Any data loss or corruption risk?] |
| **Security Impact** | [Any security risk?] |

**Business Risk**: [Low/Medium/High]

---

## 5. EVIDENCE & ATTACHMENTS

### Screenshots

**Screenshot 1**: Showing the error/issue

[Attach screenshot file]
- **Description**: Error message on attendance input form
- **File Name**: defect_screenshot_001.png

**Screenshot 2**: Showing expected behavior

[Attach screenshot file]
- **Description**: How form should look before the issue
- **File Name**: defect_screenshot_002.png

### Log Files

**Error Log Excerpt**:

```
[2024-08-22 14:35:12] ERROR: mysqli_prepare failed
File: admin/students/tambah.php Line: 45
Message: SQL syntax error in INSERT statement
Context: Adding new student with special characters
```

**Browser Console Output**:

```
Uncaught TypeError: Cannot read property 'status' of undefined
  at validateAttendance (script.js:125)
  at HTMLButtonElement.onclick (attendance.html:85)
```

### Test Data Used

- **Student Name**: Joko Setyawan
- **NIS**: 12345 (or specify test data used)
- **Attendance Date**: 2024-08-22
- **Test Data File**: `test_data_sample.sql`

---

## 6. ROOT CAUSE ANALYSIS

### Initial Investigation

**Hypothesis**: [Initial suspected cause]

**Findings**:
1. [Finding 1]
2. [Finding 2]
3. [Finding 3]

### Root Cause

**Confirmed Root Cause**: [What is the actual underlying problem?]

**Code Location**:
- **File**: `admin/students/tambah.php`
- **Line**: 45-48
- **Function**: `validateNIS()`

**Code Snippet**:

```php
// BEFORE (Incorrect)
$query = "INSERT INTO students VALUES (null, '$name', '$nis')";

// Issue: $name contains special characters (é, ç, etc.)
// Not properly escaped/encoded to UTF-8
```

**Why It Happens**:

[Explain the technical reason for the issue. What coding mistake or design flaw caused this?]

---

## 7. PROPOSED FIX

### Solution

**Fix Type**: Code change / Configuration change / Design change

**Proposed Fix**:

```php
// AFTER (Correct)
$name = htmlspecialchars($name, ENT_QUOTES, 'UTF-8');
$stmt = $connection->prepare("INSERT INTO students (name, nis) VALUES (?, ?)");
$stmt->bind_param("ss", $name, $nis);
$stmt->execute();

// Now properly handles special characters
```

### Fix Details

**Change Summary**: [What will be changed?]

**Files Affected**:
- [ ] admin/students/tambah.php (Line 45-48)
- [ ] config/database.php (If config change)
- [ ] tests/Unit/StudentManagementTest.php (Test update needed)

**Impact Assessment**:
- [ ] No breaking changes
- [ ] Minimal impact on other modules
- [ ] Requires database migration: No/Yes
- [ ] Requires cache clear: No/Yes

---

## 8. FIX VERIFICATION

### Fix Verification Steps

1. Apply fix to code
2. Run affected unit tests

```bash
./vendor/bin/phpunit tests/Unit/StudentManagementTest.php
```

3. Execute manual testing

```
Test Case: Add student with special characters
Steps:
1. Login as admin
2. Go to Add Student
3. Enter name: "José María García"
4. Enter NIS: "12345"
5. Submit form
Expected: Student saved correctly, no encoding issues
```

4. Run regression tests

```bash
# Ensure other student tests still pass
./vendor/bin/phpunit tests/Unit/StudentManagementTest.php
./vendor/bin/phpunit tests/Integration/
```

### Verification Results

**Fix Applied On**: [Date]

- [x] Code change deployed
- [x] Unit tests passing
- [x] Integration tests passing
- [x] Manual testing completed
- [x] No regression issues found

**Verified By**: [Name]  
**Date Verified**: [Date]

**Status**: ✅ **FIXED & VERIFIED**

---

## 9. DEFECT METRICS

### Timeline

| Event | Date | Time | User |
|-------|------|------|------|
| Reported | 2024-08-22 | 14:30 | John Doe |
| Assigned | 2024-08-22 | 14:45 | Project Manager |
| In Progress | 2024-08-22 | 15:00 | Developer |
| Fixed | 2024-08-22 | 16:30 | Developer |
| Verified | 2024-08-23 | 09:00 | QA Team |
| Closed | 2024-08-23 | 10:00 | Project Manager |

### Resolution Summary

| Metric | Value |
|--------|-------|
| **Time to Fix** | 1.5 hours |
| **Time to Verify** | 18 hours |
| **Total Resolution Time** | 19.5 hours |
| **Status** | RESOLVED |

---

## 10. COMMUNICATION LOG

### Comments & Discussion

**Comment 1** - Aug 22, 14:35 by John Doe (Reporter)
> "Issue occurs when adding students with Indonesian names containing special characters. Tested with 'Joko Setyawan', 'Bambang Wijaya' - no issue. Then tried 'José' - CSV export showed 'Jos?' instead."

**Comment 2** - Aug 22, 15:00 by Developer
> "I've identified the issue. The problem is in the CSV export header - missing UTF-8 encoding declaration. Working on fix."

**Comment 3** - Aug 22, 16:35 by Developer
> "Fix applied. Added header('Content-Type: text/csv; charset=utf-8'); to export function. Ready for testing."

**Comment 4** - Aug 23, 09:15 by QA
> "Fix verified. Re-tested with same test data - special characters now display correctly in CSV file. Issue is RESOLVED."

**Comment 5** - Aug 23, 10:00 by Project Manager
> "Defect officially closed. Fix deployed to production. Monitoring for any new occurrences."

---

## 11. RELATED INFORMATION

### Related Defects

- Defect ID: [DEF-XXX-0001]
  - Title: "Email special characters not saved"
  - Status: OPEN - Similar root cause?

### Related Test Cases

- Test Case: TC-STU-005 (Export student list to CSV)
- Test Case: TC-STU-001 (Add student with special characters)

### Related Code Sections

- `admin/students/tambah.php` (Student entry)
- `admin/students/export.php` (CSV export)
- `config/database.php` (Database connection)

---

## 12. CLOSURE CHECKLIST

- [x] Root cause identified and documented
- [x] Fix implemented and reviewed
- [x] Code changes committed to repository
- [x] Unit tests updated and passing
- [x] Integration tests passing
- [x] Manual testing completed
- [x] No regression issues found
- [x] End-user notified of fix
- [x] Documentation updated (if needed)
- [x] Defect marked as CLOSED

**Closed By**: [Name]  
**Closure Date**: [Date]  
**Final Status**: ✅ **RESOLVED & VERIFIED**

---

## APPENDIX A: DEFECT SEVERITY GUIDE

### Critical
- System is completely down
- Data is lost or corrupted
- Security vulnerability (e.g., SQL injection, unauthorized access)
- All users affected

**Example**: "Login system broken - no one can access application"

### High
- Major feature is broken or non-functional
- Workaround exists but is difficult
- Multiple features affected
- Many users impacted

**Example**: "Attendance input fails for 50% of students due to validation error"

### Medium
- Feature works but with issues
- Workaround is available
- Some users affected or some records affected
- Data integrity might be at risk

**Example**: "CSV export shows corrupted data with special characters (but manual input works)"

### Low
- Minor cosmetic issue
- No functional impact
- Workaround not needed
- Single user or very limited impact

**Example**: "Page heading font is slightly misaligned"

---

## APPENDIX B: SAMPLE DEFECT REPORTS

### Sample 1: SQL Injection Vulnerability (Critical)

```
Defect ID: DEF-2024-0001
Title: SQL Injection in Student Search
Severity: CRITICAL
Component: Student Management
Status: FIXED & VERIFIED

Summary: Search student by name is vulnerable to SQL injection attack

Steps to Reproduce:
1. Go to Student Management > Search
2. In search field, enter: ' OR '1'='1
3. Click Search
4. Result: All students displayed (security breach)

Root Cause: User input not properly sanitized before SQL query
File: admin/students/index.php Line 32
Vulnerable Code: $query = "SELECT * FROM students WHERE name LIKE '%$search%'";

Fix: Use prepared statements
$stmt = $connection->prepare("SELECT * FROM students WHERE name LIKE ?");
$param = "%$search%";
$stmt->bind_param("s", $param);
$stmt->execute();

Status: FIXED & VERIFIED by QA
```

### Sample 2: Session Timeout Not Working (High)

```
Defect ID: DEF-2024-0002
Title: Session timeout not functioning
Severity: HIGH
Component: Authentication
Status: FIXED & VERIFIED

Summary: Users remain logged in indefinitely, even if inactive

Steps to Reproduce:
1. Login to system
2. Don't perform any action for 30 minutes
3. Expected: Auto-logout and redirect to login
4. Actual: Still logged in, can access pages

Root Cause: Session timeout checking not implemented
File: config/auth.php
Missing: Last activity time tracking in session

Fix: Implemented session timeout check
Added: $_SESSION['last_activity'] tracking
Modified: session_check() function

Status: FIXED & VERIFIED
```

---

**Document Version**: 1.0  
**Template Created**: August 10, 2026

For defect reporting questions, contact the QA Lead or Project Manager.

---

**END OF TEMPLATE**
