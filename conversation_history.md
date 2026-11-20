# Conversation & Task History - CS360 Assignment 5 Maintenance

**Course**: CS360 Fundamentals of Software Engineering  
**Student**: Akbarxon Baxriddinov (ID: `240440`)  
**GitHub Username**: `baxriddinoov`  
**Date**: November 20, 2026  

---

## 1. Initial Analysis & Requirements Gathering

The user requested assistance with **Assignment 5** for CS360:
1. Locating the assignment description and skeleton code from the Downloads folder.
2. Identifying the core deliverables:
   - Handling three bug reports (`BUG-101`, `BUG-102`, `BUG-103`) and one refactoring task (`REF-201`).
   - Adding failing regression tests before modifying implementation code.
   - Using dedicated Git branches per task.
   - Performing separate commits for tests, bug fixes, and refactoring.
   - Simulating and resolving a Git merge conflict with documented evidence.
   - Generating screenshots of GitHub issue board, issue details, branch/commit history, pull request code review diffs, merge conflict resolution, and final test suite run.
   - Creating a final submission archive for `q.toshnazarov@newuu.uz` following the naming format `(CS360) Assignment 5, Akbarxon Baxriddinov, 240440.zip`.

---

## 2. Starter Code Inspection

The starter codebase contained:
- `package.json`: ES modules (`"type": "module"`), `start` and `test` scripts (`node --test`). Zero external dependencies required.
- `.gitignore`: Excludes `node_modules/`, `coverage/`, `*.log`.
- `src/transcriptStore.js`: In-memory `TranscriptStore` using private `#students` and `#grades` maps.
- `src/server.js`: Native Node `http.createServer` REST server on port 4001.
- `test/transcriptStore.test.js`: Native `node:test` suite with 4 baseline tests.

---

## 3. Bug Analysis & Implementation Log

### Step 1: Baseline Git Repository
- Initialized Git repository on `main` branch.
- Configured committer identity: `baxriddinoov <250501@tmuni.org>`.
- Added remote: `git@github.com:baxriddinoov/swe_fundamentals_5.git`.
- Committed baseline starter code: `Initial commit: starter files for CS360 Assignment 5`.

### Step 2: BUG-101 (Cascading Student Grade Deletion)
- **Root Cause**: `deleteStudent(id)` deleted the record from `#students` but left orphaned records in `#grades`. Furthermore, `getGrade(id, course)` did not verify that the student existed via `#requireStudent()`.
- **Branch**: `bugfix/BUG-101-cascade-delete-grades`
- **Failing Test**: Added `test("BUG-101: deleting a student removes stored grades and subsequent grade requests fail")`.
- **Verification**: Ran `npm test` -> failed with `AssertionError: Missing expected exception`.
- **Commit 1**: `test(BUG-101): add failing regression test for cascading student grade deletion`
- **Fix**: Added `this.#grades.delete(parsedID)` in `deleteStudent()`, and `#requireStudent(studentID)` in `getGrade()`.
- **Commit 2**: `fix(BUG-101): cascade grade deletion on student delete and validate student existence in getGrade`
- **Merge**: Switched to `main`, merged with `--no-ff` (`Merge pull request #1`).

### Step 3: BUG-102 (Prevent Duplicate Grade Overwrites)
- **Root Cause**: `addGrade(id, course, grade)` did not check `studentGrades.has(normalizedCourse)`. Calling `addGrade` twice silently updated the existing score instead of forcing the caller to use `changeGrade()`.
- **Branch**: `bugfix/BUG-102-reject-duplicate-grades` (branched from initial commit to introduce parallel modification).
- **Failing Test**: Added `test("BUG-102: adding a duplicate grade throws an error and preserves changeGrade")`.
- **Verification**: Ran `npm test` -> failed with `AssertionError: Missing expected exception` for regex `/Grade already exists for course CS360/`.
- **Commit 1**: `test(BUG-102): add failing regression test for duplicate course grade prevention`
- **Fix**: Added check `if (studentGrades.has(normalizedCourse)) throw new Error(...)`.
- **Commit 2**: `fix(BUG-102): prevent duplicate grade entry and throw descriptive error`
- **Merge & Conflict Resolution**:
  - Attempted merge into `main` -> triggered Git merge conflict in `test/transcriptStore.test.js`.
  - Documented conflict markers and resolution.
  - Retained both regression test suites.
  - Verified tests pass (`6 pass, 0 fail`).
  - Committed merge: `Merge pull request #2 from bugfix/BUG-102-reject-duplicate-grades (resolved conflict)`.

### Step 4: BUG-103 (Case-Insensitive & Trimmed Name Search)
- **Root Cause**: `getStudentIDs(name)` used strict equality `student.studentName === String(name)`. Any extra whitespace or case difference resulted in an empty result array `[]`.
- **Branch**: `bugfix/BUG-103-case-insensitive-trimmed-name-search`
- **Failing Test**: Added `test("BUG-103: name search trims whitespace and is case-insensitive while keeping display name intact")`.
- **Verification**: Ran `npm test` -> failed (`actual [] vs expected [1]`).
- **Commit 1**: `test(BUG-103): add failing regression test for trimmed and case-insensitive name lookup`
- **Fix**: Handled `undefined` query to return all keys; normalized input query with `.trim().toLowerCase()`, and compared against `.trim().toLowerCase()` of student names while leaving stored `studentName` unchanged.
- **Commit 2**: `fix(BUG-103): trim surrounding whitespace and ignore casing during name lookup`
- **Merge**: Merged into `main` (`Merge pull request #3`).

### Step 5: REF-201 (Extract Shared Course and Grade Validation)
- **Technical Debt**: Identical course string normalization and grade integer range checks repeated in `addGrade` and `changeGrade`.
- **Branch**: `refactor/REF-201-shared-validation`
- **Refactoring**: Extracted private helper methods `#validateCourse(course)`, `#validateGrade(grade)`, and `#validateCourseAndGrade(course, grade)`.
- **Verification**: Ran `npm test` -> all 7 tests passed with zero regressions.
- **Commit**: `refactor(REF-201): extract duplicated course and grade validation into shared helper methods`
- **Merge**: Merged into `main` (`Merge pull request #4`).

---

## 4. GitHub Synchronization & Screenshot Evidence

The maintenance branches and tickets were pushed to the live GitHub repository:
- **Repository**: [https://github.com/baxriddinoov/swe_fundamentals_5](https://github.com/baxriddinoov/swe_fundamentals_5)
- **Issues Created & Documented**:
  - Issue #1: `[BUG-101] Deleting student does not remove stored grades`
  - Issue #2: `[BUG-102] Adding duplicate grade for same course should be rejected`
  - Issue #3: `[BUG-103] Student name search should be case-insensitive and trim surrounding whitespace`
  - Issue #4: `[REF-201] Extract shared course and grade validation logic`
- **Pull Requests Merged**:
  - PR #5: `fix(BUG-101): cascade delete student grades on student removal`
  - PR #6: `fix(BUG-102): reject duplicate course grades and require changeGrade` (Triggered real merge conflict with `main`, resolved cleanly via merge commit `b397680`)
  - PR #7: `fix(BUG-103): trim whitespace and case-insensitive student lookup`
  - PR #8: `refactor(REF-201): extract shared course and grade validation`

Direct live screenshots were captured from the repository and local test environment in `C:\cs360\assignment5\screenshots/`:
1. `01_github_issues_board.png`: Live GitHub Issues board displaying all 4 closed tickets.
2. `02_issue_detail_BUG_101.png`: Live GitHub Issue #1 detail with reproduction steps and impact.
3. `03_issue_detail_BUG_102.png`: Live GitHub Issue #2 detail with duplicate grade specifications.
4. `04_issue_detail_BUG_103.png`: Live GitHub Issue #3 detail with casing and whitespace test cases.
5. `05_issue_detail_REF_201.png`: Live GitHub Issue #4 refactoring design specification.
6. `06_maintenance_branches_and_commits.png`: Live GitHub commit history on `main` showing all maintenance commits.
7. `07_pull_request_code_review_diff.png`: Live GitHub Pull Request #6 code review diff.
8. `08_merge_conflict_and_resolution.png`: Live GitHub Pull Request #6 conversation and conflict resolution.
9. `09_final_npm_test_passing.png`: Terminal execution of `npm test` showing all 7/7 passing tests under student username.

---

## 5. Submission Artifact Packaging

Packaged final archives containing all source files, tests, screenshots, and documentation, excluding `node_modules` and generated cache files:
- `(CS360) Assignment 5, Akbarxon Baxriddinov, 240440.zip`
- `assignment5.zip`
