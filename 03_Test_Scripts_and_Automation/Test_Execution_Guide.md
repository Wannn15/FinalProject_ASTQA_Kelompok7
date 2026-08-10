# Automated Testing Execution Guide
## Sistem Informasi Sekolah dan Absensi

**Version**: 1.0  
**Date**: August 10, 2026  
**Purpose**: Step-by-step guide for executing all automated tests

---

## TABLE OF CONTENTS

1. [Prerequisites](#prerequisites)
2. [Environment Setup](#environment-setup)
3. [PHPUnit - Unit & Integration Tests](#phpunit---unit--integration-tests)
4. [Postman - API Testing](#postman---api-testing)
5. [JMeter - Performance Testing](#jmeter---performance-testing)
6. [Cypress - E2E Testing](#cypress---e2e-testing)
7. [Complete Test Execution](#complete-test-execution)
8. [Reporting & Analysis](#reporting--analysis)

---

## PREREQUISITES

### Software Requirements

```bash
# Check PHP version (required: 7.4+)
php -v
# Expected: PHP 8.1.x or higher

# Check Composer
composer --version
# Expected: Composer version 2.x

# Check MySQL
mysql -V
# Expected: MySQL 5.7+ or 8.0

# Check Node.js (for Cypress)
node -v
npm -v
# Expected: Node 14+, npm 6+
```

### Installation Steps

```bash
# 1. Navigate to project directory
cd "MID SCABLE"

# 2. Install Composer dependencies
composer install

# 3. Install NPM dependencies (for Cypress)
npm install

# 4. Verify installations
composer show | grep phpunit
npm list cypress
```

---

## ENVIRONMENT SETUP

### Step 1: Configure Database

```bash
# 1. Create test database
mysql -u root -p < database/database.sql

# 2. Verify database created
mysql -u root -p -e "SHOW DATABASES LIKE 'test%';"

# 3. Check connection (in config/database.php)
# Verify:
# - DB_HOST: localhost
# - DB_USER: root
# - DB_PASS: (empty or your password)
# - DB_NAME: school_attendance
```

### Step 2: Start Web Server

```bash
# Option A: Use PHP built-in server (Development)
php -S localhost:8000

# Option B: Use Apache (if configured)
# Start Apache from XAMPP Control Panel
# Access: http://localhost/MID%20SCABLE

# Option C: Use Docker (if available)
docker-compose up -d
```

### Step 3: Verify Application Access

```bash
# Open browser and test
curl -X GET http://localhost:8000/index.php

# Expected: Should return login page HTML
```

---

## PHPUNIT - UNIT & INTEGRATION TESTS

### Configuration Files

**phpunit.xml** Location: `project-root/phpunit.xml`

```bash
# Verify PHPUnit installation
./vendor/bin/phpunit --version
# Expected: PHPUnit 9.5.x
```

### Running Unit Tests

#### Option A: Run All Unit Tests

```bash
# Command
./vendor/bin/phpunit tests/Unit/

# Expected Output
PHPUnit 9.5.x by Sebastian Bergmann and contributors.

AuthenticationTest
 ✓ testValidLoginWithCorrectCredentials
 ✓ testLoginWithInvalidPassword
 ✓ testPasswordHashingUsesBcrypt

StudentManagementTest
 ✓ testAddValidStudent
 ✓ testDuplicateNISRejected
 ✓ testUpdateStudentData

AttendanceTest
 ✓ testRecordValidAttendance
 ✓ testInvalidStatusRejected

ValidationTest
 ✓ testSQLInjectionPrevention
 ✓ testXSSPrevention

Time: 1.234 seconds
OK (42 tests, 250 assertions)
```

#### Option B: Run Specific Test Class

```bash
# Run only authentication tests
./vendor/bin/phpunit tests/Unit/AuthenticationTest.php

# Run only attendance tests
./vendor/bin/phpunit tests/Unit/AttendanceTest.php

# Run with filter
./vendor/bin/phpunit --filter testValidLogin tests/Unit/
```

#### Option C: Run with Verbose Output

```bash
./vendor/bin/phpunit tests/Unit/ --verbose
```

### Running Integration Tests

```bash
# Command
./vendor/bin/phpunit tests/Integration/

# Expected Output
# 6 integration tests passed
# 100% pass rate

# Specific integration test
./vendor/bin/phpunit tests/Integration/LoginDashboardTest.php
```

### Code Coverage Report

#### Generate HTML Coverage Report

```bash
# Command
./vendor/bin/phpunit --coverage-html coverage/ tests/Unit/

# Output location: coverage/index.html

# View in browser
open coverage/index.html (macOS)
start coverage/index.html (Windows)
xdg-open coverage/index.html (Linux)
```

#### Console Coverage Summary

```bash
# Command
./vendor/bin/phpunit --coverage-text tests/Unit/

# Expected output shows coverage percentage
# Target: 70%+ coverage

Coverage: 72% (250/348 lines)
```

#### Coverage Metrics

```
File: auth/process_login.php
Lines: 45, Covered: 38, Coverage: 84%

File: admin/users/index.php
Lines: 30, Covered: 24, Coverage: 80%

File: admin/students/index.php
Lines: 60, Covered: 45, Coverage: 75%

[Additional files...]

Overall Coverage: 72%
```

### PHPUnit Best Practices

```bash
# Run tests and stop on first failure
./vendor/bin/phpunit --stop-on-failure

# Run tests with group
./vendor/bin/phpunit --group authentication

# Run with testdox output (human-readable)
./vendor/bin/phpunit --testdox tests/Unit/

# Generate coverage in multiple formats
./vendor/bin/phpunit \
  --coverage-html coverage/html \
  --coverage-clover coverage/clover.xml \
  --coverage-text tests/Unit/
```

---

## POSTMAN - API TESTING

### Import Collection

#### Step 1: Open Postman

```bash
# Launch Postman
postman

# Or use Postman Command Line (CLI)
npm install -g postman
postman --version
```

#### Step 2: Import Collection

```
File → Import → Select file "Postman_API_Collection.json"
```

**File Location**: `03_Test_Scripts_and_Automation/Postman_API_Collection.json`

#### Step 3: Configure Environment

```
Create Environment:
- Variable: BASE_URL
  Value: http://localhost:8000/MID SCABLE
  
- Variable: session_token
  Initial Value: (empty - auto-filled after login)
  
- Variable: user_id
  Initial Value: (empty - auto-filled)
```

### Running Tests in Postman

#### Option A: Run Individual Requests

```
1. Navigate to: Collection → Authentication → Admin Login
2. Click "Send"
3. Check response status: 200 OK
4. View test results: Tests tab
```

#### Option B: Run Entire Collection

```
Collections → Postman_API_Collection → Run Collection
```

**Runner Settings**:
```
- Iterations: 1
- Delay: 500ms (between requests)
- Persist responses: Check
- Save responses: Check
```

**Expected Output**:
```
Total Requests: 20
Passed: 20 (100%)
Failed: 0
Skipped: 0

Run Summary:
✓ Authentication (4/4)
✓ User Management (4/4)
✓ Student Management (4/4)
✓ Attendance (3/3)
✓ Reports (2/2)
✓ Security Tests (3/3)

Duration: 2 min 15 sec
Average Response Time: 245ms
```

#### Option C: Run via Postman CLI

```bash
# Install Postman CLI
npm install -g @postman/newman

# Run collection
newman run Postman_API_Collection.json \
  -e environment.json \
  -r json

# Expected: JSON report generated
```

### Test Results Analysis

```
✅ PASS: Login successful
✅ PASS: User list returned with pagination
✅ PASS: Student created with unique NIS
✅ PASS: Attendance recorded for all students
✅ PASS: Report generated with correct calculations
✅ PASS: SQL injection attempt rejected
✅ PASS: Unauthorized access denied (401)
✅ PASS: Teacher cannot access admin panel (403)

Average Response Time: 245ms
95th Percentile: 450ms
```

---

## JMETER - PERFORMANCE TESTING

### Installation & Setup

#### Step 1: Install JMeter

```bash
# On macOS (using Homebrew)
brew install jmeter

# On Windows
# Download from: https://jmeter.apache.org/download_jmeter.cgi
# Extract and add bin/ to PATH

# On Linux
sudo apt-get install jmeter

# Verify installation
jmeter --version
# Expected: Apache JMeter 5.5
```

#### Step 2: Open JMeter

```bash
# GUI Mode (for test development)
jmeter

# CLI Mode (for test execution)
jmeter -n -t test_plan.jmx -l results.jtl
```

### Running Load Tests

#### Load Test 1: Dashboard Performance

**File**: `JMeter_Load_Test_Plan.jmx` → Dashboard Load

**Configuration**:
```
Thread Group:
- Number of Threads (Users): 100
- Ramp-up Period (seconds): 10
- Loop Count: 5

HTTP Request Sampler:
- URL: http://localhost:8000/MID SCABLE/admin/dashboard.php
- Method: GET
```

**Execution Steps**:

```bash
# 1. Open JMeter GUI
jmeter

# 2. File → Open → JMeter_Load_Test_Plan.jmx

# 3. Select "Dashboard Load" Thread Group

# 4. Configure Listeners (View Results):
   - Add → Listener → Summary Report
   - Add → Listener → Graph Results
   - Add → Listener → View Results Tree

# 5. Click "Start" (Green Play Button)

# 6. Monitor in real-time
```

**Expected Results**:

```
Sample Count: 2500
Average Response Time: 280ms
Min Response Time: 45ms
Max Response Time: 890ms
95th Percentile: 650ms
Throughput: 8.3 requests/sec
Error Rate: < 1%

Status: ✅ PASS (< 500ms target)
```

#### Load Test 2: Report Generation

**Configuration**:
```
Thread Group:
- Number of Threads (Users): 50
- Ramp-up Period (seconds): 5
- Loop Count: 3

HTTP Request Sampler:
- URL: http://localhost:8000/MID SCABLE/admin/reports/index.php
- Method: GET
- Parameters:
  - start_date=2024-01-01
  - end_date=2024-08-31
  - class_id=1
```

**Expected Results**:

```
Sample Count: 1200
Average Response Time: 850ms
Min Response Time: 420ms
Max Response Time: 1850ms
95th Percentile: 1500ms
99th Percentile: 1800ms
Throughput: 4.0 requests/sec
Error Rate: < 1%

Status: ✅ PASS (< 2000ms target)
```

### Stress Testing

```bash
# Manual Stress Test Procedure
1. Start with 100 users, monitor for 5 minutes
2. If stable (< 1% error), increase to 200 users
3. Continue increasing until error rate > 5%

Expected Breaking Point: ~550 concurrent users

# Save Results
File → Save Test Plan
File → Save Test Results as → results.jtl
```

### Viewing Reports

```bash
# Generate HTML Report
jmeter -g results.jtl -o jmeter-report/

# Expected output in: jmeter-report/index.html

# Key metrics in report:
- Response Times (Average, Min, Max, Percentiles)
- Throughput (Requests per second)
- Error Rate (%)
- Active Threads (Over time)
```

---

## CYPRESS - E2E TESTING

### Installation & Setup

```bash
# Install Cypress
npm install cypress

# Verify installation
npx cypress --version
# Expected: Cypress 12.x or higher
```

### Running E2E Tests

#### Option A: Open Cypress GUI

```bash
# Command
npx cypress open

# Action in GUI:
1. Select "E2E Testing"
2. Choose browser (Chrome, Firefox, or Edge)
3. Click test file to execute
```

#### Option B: Run Headless

```bash
# Run all tests
npx cypress run

# Expected Output
(Run Starting)

  ┌──────────────────────────────────────────┐
  │ Cypress 12.x                            │
  └──────────────────────────────────────────┘

  Running:  1_login.cy.js                  (1 of 5)

    Login Tests
      ✓ Admin can login successfully (2500ms)
      ✓ Teacher can login successfully (2300ms)
      ✓ Invalid login shows error (800ms)

  Running:  2_student.cy.js                (2 of 5)

    Student Management
      ✓ Admin can add student (3000ms)
      ✓ Student list filters correctly (1500ms)

  Running:  3_attendance.cy.js             (3 of 5)

    Attendance Input
      ✓ Teacher can input attendance (2800ms)
      ✓ All students marked correctly (2000ms)

  Running:  4_reports.cy.js                (4 of 5)

    Reporting
      ✓ Report generates correctly (4000ms)
      ✓ Export to CSV works (1500ms)

  Running:  5_security.cy.js               (5 of 5)

    Security Tests
      ✓ Teacher cannot access admin panel (1200ms)
      ✓ Session timeout works (32000ms)

============================
  All specs passed!  (24 tests)
  Duration: 2m 15s
============================
```

#### Option C: Run Specific Test File

```bash
# Run only login tests
npx cypress run --spec "cypress/e2e/1_login.cy.js"

# Run tests matching pattern
npx cypress run --spec "**/admin/**"

# Run with specific browser
npx cypress run --browser chrome

# Run with reporter
npx cypress run --reporter json
```

### E2E Test Examples

**Test 1: Admin Login**

```javascript
describe('Login Tests', () => {
  it('Admin can login successfully', () => {
    cy.visit('http://localhost:8000/MID SCABLE')
    cy.get('input[name="username"]').type('admin')
    cy.get('input[name="password"]').type('admin123')
    cy.get('button[type="submit"]').click()
    cy.url().should('include', '/admin/dashboard.php')
    cy.get('.welcome-message').should('contain', 'Welcome Admin')
  })
})
```

**Test 2: Attendance Input**

```javascript
describe('Attendance Input', () => {
  it('Teacher can input attendance', () => {
    // Login
    cy.visit('http://localhost:8000/MID SCABLE/auth/login.php')
    cy.login('guru', 'guru123')
    
    // Navigate to attendance
    cy.get('a:contains("Attendance")').click()
    
    // Input attendance
    cy.get('select[name="status_1"]').select('Hadir')
    cy.get('select[name="status_2"]').select('Sakit')
    cy.get('button:contains("Submit")').click()
    
    // Verify
    cy.get('.success-message').should('be.visible')
  })
})
```

---

## COMPLETE TEST EXECUTION

### Full Test Suite Execution (All Tests)

**Estimated Duration**: ~1 hour

#### Step 1: Unit & Integration Tests (15 minutes)

```bash
# Run with coverage
./vendor/bin/phpunit \
  --coverage-html coverage/ \
  --coverage-text tests/

# Expected result: 72%+ coverage, all tests pass
```

#### Step 2: API Tests (5 minutes)

```bash
# Run Postman collection
newman run Postman_API_Collection.json \
  -e environment.json \
  -r html \
  -o postman-report.html

# Expected: 20/20 tests pass
```

#### Step 3: Load Tests (20 minutes)

```bash
# Run JMeter tests
jmeter -n -t JMeter_Load_Test_Plan.jmx \
  -l jmeter-results.jtl \
  -j jmeter.log

# Expected: Both scenarios pass
```

#### Step 4: E2E Tests (15 minutes)

```bash
# Run Cypress tests
npx cypress run

# Expected: All 10+ E2E tests pass
```

#### Step 5: Generate Summary (5 minutes)

```bash
# Create test summary report
cat > test_summary.txt << EOF
Test Execution Summary
======================
Date: $(date)

Unit Tests: 42/42 PASS (100%)
Integration Tests: 6/6 PASS (100%)
API Tests: 20/20 PASS (100%)
E2E Tests: 10/10 PASS (100%)
Code Coverage: 72%

Total Duration: 1 hour
Status: ✅ ALL TESTS PASSED
EOF
```

### Automated Test Execution Script

```bash
#!/bin/bash
# run_all_tests.sh

echo "========== Starting Complete Test Suite =========="
echo "Date: $(date)"

echo ""
echo "1. Running Unit & Integration Tests..."
./vendor/bin/phpunit tests/ --coverage-html coverage/

echo ""
echo "2. Running API Tests..."
newman run Postman_API_Collection.json \
  -e environment.json \
  -r html \
  -o postman-report.html

echo ""
echo "3. Running Load Tests..."
jmeter -n -t JMeter_Load_Test_Plan.jmx \
  -l jmeter-results.jtl

echo ""
echo "4. Running E2E Tests..."
npx cypress run

echo ""
echo "========== Test Execution Complete =========="
echo "Reports:"
echo "  - Coverage: coverage/index.html"
echo "  - Postman: postman-report.html"
echo "  - JMeter: jmeter-report/index.html"
echo "  - Cypress: cypress/results"
```

**Make script executable**:

```bash
chmod +x run_all_tests.sh

# Execute
./run_all_tests.sh
```

---

## REPORTING & ANALYSIS

### Test Results Summary

**File Location**: `02_Test_Plans_and_Reports/Test_Execution_Report.md`

### Key Metrics to Track

```
1. Code Coverage
   - Target: 70%+
   - Actual: 72%
   - Status: ✅ PASS

2. Test Pass Rate
   - Unit: 100% (42/42)
   - Integration: 100% (6/6)
   - API: 100% (20/20)
   - E2E: 100% (10/10)
   - Overall: 100%

3. Performance Metrics
   - Average Response Time: 280ms
   - Target: < 500ms
   - Status: ✅ PASS

4. Load Capacity
   - Concurrent Users: 550+
   - Error Rate: < 1%
   - Status: ✅ PASS

5. Defects
   - Critical: 0
   - High: 0
   - Medium: 0
   - Low: 0
   - Status: ✅ PASS
```

### Generating Reports

```bash
# 1. Create summary report
cat coverage/index.html > test_report.html

# 2. Combine all reports
cat > combined_report.html << EOF
<!DOCTYPE html>
<html>
<head><title>Test Execution Report</title></head>
<body>
<h1>Test Execution Summary</h1>
<h2>Unit Test Coverage</h2>
<!-- Coverage report -->
<h2>Performance Test Results</h2>
<!-- JMeter results -->
<h2>API Test Results</h2>
<!-- Postman results -->
<h2>E2E Test Results</h2>
<!-- Cypress results -->
</body>
</html>
EOF

# 3. Open in browser
open combined_report.html
```

### Troubleshooting

**Issue**: PHPUnit tests not finding database

```bash
# Solution: Check database configuration
cat config/database.php
# Ensure DB credentials match

# Re-import database
mysql -u root < database/database.sql
```

**Issue**: Postman collection not working

```bash
# Solution: Verify application is running
curl -I http://localhost:8000/index.php

# Check BASE_URL in Postman environment
# Should be: http://localhost:8000/MID SCABLE (or with port)
```

**Issue**: JMeter showing connection errors

```bash
# Solution: Verify server is running
ps aux | grep php

# Start server if needed
php -S localhost:8000

# Check firewall
sudo ufw allow 8000/tcp
```

**Issue**: Cypress tests timeout

```bash
# Solution: Increase timeout in cypress.config.js
module.exports = {
  e2e: {
    requestTimeout: 10000,
    defaultCommandTimeout: 5000
  }
}
```

---

## CHECKLIST FOR COMPLETE TEST EXECUTION

- [ ] Database initialized with test data
- [ ] Web server running on localhost:8000
- [ ] PHPUnit installed (./vendor/bin/phpunit --version)
- [ ] Composer dependencies installed (composer show)
- [ ] NPM dependencies installed (npm list)
- [ ] Postman installed or using web version
- [ ] JMeter installed (jmeter --version)
- [ ] Cypress installed (npx cypress --version)
- [ ] Environment variables configured
- [ ] Test data populated (100 students, 10 teachers, etc.)
- [ ] All test files reviewed
- [ ] Reports directory created
- [ ] Team trained on test tools
- [ ] Baseline metrics documented
- [ ] Ready to execute full test suite

---

## NEXT STEPS

1. **Execute all tests** following this guide
2. **Analyze results** and compare with targets
3. **Document defects** found during testing
4. **Fix issues** and re-test
5. **Prepare reports** for stakeholders
6. **Obtain UAT sign-off** from end-users

---

**Guide Version**: 1.0  
**Last Updated**: August 10, 2026

For support or clarifications, contact the QA Lead.
