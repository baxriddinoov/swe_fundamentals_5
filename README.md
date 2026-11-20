# CS360 Assignment 5 - Software Maintenance & Bug Handling

**Course**: CS360 - Fundamentals of Software Engineering (Fall 2026)  
**Institution**: New Uzbekistan University  
**Instructor**: Dr. Qobiljon Toshnazarov (`q.toshnazarov@newuu.uz`)  
**Student**: Akbarxon Baxriddinov  
**Student ID**: `240440`  
**Email**: `baxriddinovakbarxon@gmail.com`  
**GitHub Username**: `baxriddinoov`  
**Repository**: `https://github.com/baxriddinoov/swe_fundamentals_5`  

---

## 1. Executive Summary

This project implements the required maintenance lifecycle for the **Transcript Manager** application in accordance with the software maintenance principles covered in CS360 (handling bug reports, regression testing, feature branching, code review diff inspection, merge conflict resolution, and refactoring).

All three reported bugs (`BUG-101`, `BUG-102`, `BUG-103`) and the refactoring request (`REF-201`) have been diagnosed, addressed via test-driven maintenance (adding failing regression tests before modifying code), resolved on isolated Git branches, code-reviewed, and merged back into the `main` branch with documented merge conflict resolution.

---

## 2. Tracked Maintenance Tickets

### BUG-101: Cascading Deletion of Student Grades
* **Ticket ID**: `BUG-101`
* **Affected Files**: `src/transcriptStore.js`, `test/transcriptStore.test.js`
* **Branch**: `bugfix/BUG-101-cascade-delete-grades`
* **Problem / Actual Behavior**: Calling `deleteStudent(studentID)` removed the student entry from `#students` but left lingering grade records in `#grades`. Subsequent queries via `getGrade(studentID, course)` or HTTP `GET /transcripts/:id/:course` returned the orphaned grade with status `200 OK` rather than failing.
* **Reproduction Steps**:
  1. Register student: `const id = store.addStudent("Ada");`
  2. Add course grade: `store.addGrade(id, "CS360", 95);`
  3. Delete student: `store.deleteStudent(id);`
  4. Query grade: `store.getGrade(id, "CS360");` (returned `95` in buggy code).
* **Expected Behavior**: Deleting a student must cascade-delete all their stored grades from `#grades`. Any subsequent request for a deleted student's grades must fail with `Student not found: <studentID>`, returning HTTP `404 Not Found`.
* **Fix**: Added `this.#grades.delete(parsedID)` inside `deleteStudent()`, and required student existence validation in `getGrade()` via `this.#requireStudent(studentID)`.
* **Verification**: Added regression test in `test/transcriptStore.test.js`.

---

### BUG-102: Duplicate Grade Registration Prevention
* **Ticket ID**: `BUG-102`
* **Affected Files**: `src/transcriptStore.js`, `test/transcriptStore.test.js`
* **Branch**: `bugfix/BUG-102-reject-duplicate-grades`
* **Problem / Actual Behavior**: Invoking `addGrade(studentID, course, grade)` multiple times for the same student and course silently overwrote existing marks without throwing an error, bypassing `changeGrade()`.
* **Reproduction Steps**:
  1. Register student: `const id = store.addStudent("Lin");`
  2. Add grade: `store.addGrade(id, "cs360", 85);`
  3. Attempt duplicate entry: `store.addGrade(id, "CS360", 90);` (overwrote to 90 without error).
* **Expected Behavior**: A second grade for the same student and course must throw an error: `"Grade already exists for course <COURSE>"` (producing HTTP `409 Conflict`). Corrections must be performed exclusively via `changeGrade()`.
* **Fix**: Added duplicate check `if (studentGrades.has(normalizedCourse)) throw new Error(...)` before writing to `studentGrades`.
* **Verification**: Added regression test in `test/transcriptStore.test.js`.

---

### BUG-103: Whitespace-Insensitive and Case-Insensitive Name Search
* **Ticket ID**: `BUG-103`
* **Affected Files**: `src/transcriptStore.js`, `test/transcriptStore.test.js`
* **Branch**: `bugfix/BUG-103-case-insensitive-trimmed-name-search`
* **Problem / Actual Behavior**: `getStudentIDs(name)` performed strict equality comparison `student.studentName === String(name)`. Any surrounding whitespace (`"  Grace Lee  "`) or casing difference (`"grace lee"`) caused search misses (`[]`).
* **Reproduction Steps**:
  1. Register student: `const id = store.addStudent("Grace Lee");`
  2. Search with whitespace: `store.getStudentIDs("  Grace Lee  ");` -> returned `[]`
  3. Search with lowercase: `store.getStudentIDs("grace lee");` -> returned `[]`
* **Expected Behavior**: Name search must trim surrounding whitespace and ignore casing when filtering students. Stored display names (`student.studentName`) must remain unchanged in their original casing.
* **Fix**: Normalized search query using `String(name).trim().toLowerCase()` and compared against `student.studentName.trim().toLowerCase()`.
* **Verification**: Added regression test in `test/transcriptStore.test.js`.

---

### REF-201: Shared Course and Grade Validation Extraction
* **Ticket ID**: `REF-201`
* **Affected Files**: `src/transcriptStore.js`
* **Branch**: `refactor/REF-201-shared-validation`
* **Description**: `addGrade` and `changeGrade` contained duplicate course string normalization and grade integer range validation.
* **Refactoring**: Extracted private helper methods `#validateCourse(course)`, `#validateGrade(grade)`, and `#validateCourseAndGrade(course, grade)`.
* **Outcome**: Adheres to DRY (Don't Repeat Yourself) principle while preserving 100% of public API contracts, error messages, and behavior. Committed separately on a dedicated refactoring branch.

---

## 3. Git Workflow, Branching & Conflict Resolution

The maintenance process followed professional Git branching standards:
1. **Initial Commit**: Baseline starter files committed to `main`.
2. **Feature Branches**: Dedicated branches created for each bug report and refactor task:
   * `bugfix/BUG-101-cascade-delete-grades`
   * `bugfix/BUG-102-reject-duplicate-grades`
   * `bugfix/BUG-103-case-insensitive-trimmed-name-search`
   * `refactor/REF-201-shared-validation`
3. **Test-First Commits**: For each bug fix, a failing regression test was written and committed first (`test(...)`), verified to fail, followed by the bugfix commit (`fix(...)`).
4. **Merge Conflict & Resolution**:
   * Branches `bugfix/BUG-101` and `bugfix/BUG-102` branched concurrently and both modified the end of `test/transcriptStore.test.js`.
   * Merging `bugfix/BUG-102` into `main` after `BUG-101` produced a real Git merge conflict:
     ```text
     CONFLICT (content): Merge conflict in test/transcriptStore.test.js
     Automatic merge failed; fix conflicts and then commit the result.
     ```
   * Resolved the conflict cleanly by retaining regression test suites for both tickets, verified with `npm test`, and committed the merge.
5. **Clean Pull Request Merges**: Merged via `--no-ff` pull request merges (`PR #1`, `PR #2`, `PR #3`, `PR #4`).

### Git Commit Graph

```text
*   ea14cc9 (HEAD -> main) Merge pull request #4 from refactor/REF-201-shared-validation
|\  
| * 7c5205a refactor(REF-201): extract duplicated course and grade validation into shared helper methods
|/  
*   8b49e9d Merge pull request #3 from bugfix/BUG-103-case-insensitive-trimmed-name-search
|\  
| * bcb3e55 fix(BUG-103): trim surrounding whitespace and ignore casing during name lookup
| * 42ff9bc test(BUG-103): add failing regression test for trimmed and case-insensitive name lookup
|/  
*   8e589c5 Merge pull request #2 from bugfix/BUG-102-reject-duplicate-grades (resolved conflict in test/transcriptStore.test.js)
|\  
| * 5df362c fix(BUG-102): prevent duplicate grade entry and throw descriptive error
| * a968fef test(BUG-102): add failing regression test for duplicate course grade prevention
* |   0ae26b9 Merge pull request #1 from bugfix/BUG-101-cascade-delete-grades
|\ \  
| |/  
|/|   
| * 759c4bf fix(BUG-101): cascade grade deletion on student delete and validate student existence in getGrade
| * 5e2ce37 test(BUG-101): add failing regression test for cascading student grade deletion
|/  
* b6b9180 Initial commit: starter files for CS360 Assignment 5
```

---

## 4. Screenshot Evidence Index

All required visual evidence has been captured and archived in the `screenshots/` directory:

| Screenshot | Description | Requirement Covered |
| :--- | :--- | :--- |
| `01_github_issues_board.png` | GitHub Issues Board showing BUG-101, BUG-102, BUG-103, and REF-201 closed | Issue board logging |
| `02_issue_detail_BUG_101.png` | Issue #1 BUG-101 description, steps, actual/expected behavior, impact | Reproduction & impact docs |
| `03_issue_detail_BUG_102.png` | Issue #2 BUG-102 description, steps, actual/expected behavior, impact | Reproduction & impact docs |
| `04_issue_detail_BUG_103.png` | Issue #3 BUG-103 description, steps, actual/expected behavior, impact | Reproduction & impact docs |
| `05_issue_detail_REF_201.png` | Issue #4 REF-201 technical debt analysis and acceptance criteria | Refactoring documentation |
| `06_maintenance_branches_and_commits.png` | Git graph, branch list (`git branch -a`), and commit history | Maintenance branch & commit history |
| `07_pull_request_code_review_diff.png` | GitHub Pull Request #6 diff review and automated check approval | Code review & diff inspection |
| `08_merge_conflict_and_resolution.png` | PR #6 conflict markers, merge commit, and conflict resolution | Merge conflict resolution evidence |
| `09_final_npm_test_passing.png` | Terminal execution of `npm test` showing all 7 tests passing (0 failures) | Final test suite verification |

---

## 5. Quality Check & How to Run

### Requirements
* Node.js v18+ (tested on Node.js v26.8.2)
* Built-in Node test runner (`node --test`), zero external dependencies needed.

### Running Tests
```bash
npm test
```

Expected output:
```text
▶ TranscriptStore
  ✔ adds students and returns their transcripts (3.5636ms)
  ✔ adds and changes a course grade (0.4575ms)
  ✔ rejects invalid student, course, and grade inputs (0.6734ms)
  ✔ supports exact-name lookup and deletion (0.3748ms)
  ✔ BUG-101: deleting a student removes stored grades and subsequent grade requests fail (0.4441ms)
  ✔ BUG-102: adding a duplicate grade throws an error and preserves changeGrade (0.3422ms)
  ✔ BUG-103: name search trims whitespace and is case-insensitive while keeping display name intact (0.2701ms)
✔ TranscriptStore (8.0804ms)
ℹ tests 7
ℹ suites 1
ℹ pass 7
ℹ fail 0
```

### Running the Server
```bash
npm start
```
Starts the Transcript Manager HTTP server on `http://localhost:4001`.
