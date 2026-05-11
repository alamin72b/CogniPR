# PR Cognitive Load Analyzer — Full Text Requirements Document

## 1. Product Overview

### 1.1 Product Name
PR Cognitive Load Analyzer

### 1.2 Product Summary
PR Cognitive Load Analyzer is a web-based developer tool that connects to GitHub repositories and automatically analyzes open Pull Requests to estimate how difficult they are to review. It measures code complexity, scope of changes, and structural impact — then produces a human-readable "Review Difficulty Score" and posts it as an automated comment directly on the GitHub Pull Request. The goal is to help engineering teams prioritize PR reviews, reduce reviewer burnout, and maintain code quality standards.

### 1.3 Problem Statement
Code reviewers often open a Pull Request without knowing upfront how mentally demanding it will be. A PR with 50 files changed, deeply nested logic, and high cyclomatic complexity can take hours to review properly. Without a way to estimate this upfront, teams suffer from:

*   Reviewer burnout from unexpectedly complex PRs
*   Bottlenecks where hard PRs sit unreviewed for days
*   Inconsistent review quality because reviewers rush through hard PRs
*   No data on which areas of the codebase are consistently complex

This tool solves that by scoring every PR automatically and surfacing the difficulty before a reviewer even opens the diff.

### 1.4 Target Users
*   **Software Engineers** who submit and review Pull Requests
*   **Engineering Managers** who track team review load and codebase health
*   **Tech Leads** who want to enforce complexity standards across a repository
*   **DevOps / Platform Engineers** who manage CI/CD pipelines and developer workflows

---

## 2. Goals and Objectives
*   Automatically analyze every Pull Request when it is opened or updated
*   Produce a cognitive load score based on objective code metrics
*   Post the score as a comment on the PR within minutes of it being opened
*   Give teams a dashboard to view historical analysis and trends
*   Allow teams to configure scoring thresholds to match their standards
*   Require no changes to existing development workflows to adopt

---

## 3. Scope

### 3.1 In Scope
*   GitHub repository integration via GitHub App
*   Automatic PR analysis triggered by GitHub webhook events
*   Manual PR analysis triggered from the dashboard
*   Cyclomatic complexity calculation per file and per function
*   Scoring based on files changed, lines changed, and complexity metrics
*   Automated GitHub comment with the score and breakdown
*   Web dashboard to view all analyzed PRs, scores, and trends
*   User authentication via GitHub OAuth
*   Support for JavaScript and TypeScript files in complexity analysis
*   Repository management (add, remove, list tracked repositories)
*   Historical data storage and trend visualization

### 3.2 Out of Scope
*   Support for GitLab, Bitbucket, or other Git providers (future consideration)
*   Analysis of non-JavaScript/TypeScript files for cyclomatic complexity (e.g., Python, Go, Java)
*   Automated PR rejection or merge blocking based on score
*   Real-time pair review assistance
*   AI-generated review suggestions or code fix recommendations
*   Mobile application

---

## 4. Functional Requirements

### 4.1 User Authentication
*   **FR-AUTH-01:** Users must be able to log in using their GitHub account via OAuth.
*   **FR-AUTH-02:** On first login, the system must create a user profile associated with the GitHub account.
*   **FR-AUTH-03:** Users must remain logged in across browser sessions using a secure session token.
*   **FR-AUTH-04:** Users must be able to log out, which invalidates their session.
*   **FR-AUTH-05:** Unauthenticated users must be redirected to the login page when trying to access any protected page.
*   **FR-AUTH-06:** The system must only allow users to view and manage repositories they have access to on GitHub.

### 4.2 Repository Management
*   **FR-REPO-01:** Authenticated users must be able to add a GitHub repository to the system for tracking by providing the repository owner and name.
*   **FR-REPO-02:** The system must verify that the user has access to the repository before adding it.
*   **FR-REPO-03:** Users must be able to view a list of all repositories they have added to the system.
*   **FR-REPO-04:** Users must be able to remove a repository from tracking, which stops future analysis and hides historical data from the dashboard.
*   **FR-REPO-05:** When a repository is added, the system must prompt the user to install the GitHub App on that repository to enable webhook events.
*   **FR-REPO-06:** The system must display the installation status of the GitHub App for each repository (installed or not installed).

### 4.3 Automatic PR Analysis via Webhooks
*   **FR-WEBHOOK-01:** The system must expose a public webhook endpoint that GitHub can send events to.
*   **FR-WEBHOOK-02:** The system must receive and process pull_request webhook events for the following actions: opened, synchronize (new commits pushed), and reopened.
*   **FR-WEBHOOK-03:** When a qualifying webhook event is received, the system must validate the webhook signature to confirm it was sent by GitHub.
*   **FR-WEBHOOK-04:** Upon receiving a valid webhook event, the system must queue analysis job for the affected Pull Request within 5 seconds.
*   **FR-WEBHOOK-05:** The system must process the queued analysis job and complete the full analysis within 2 minutes of the job being queued under normal conditions.
*   **FR-WEBHOOK-06:** If a PR is updated with new commits (synchronize event), the system must re-analyze the PR and update the existing GitHub comment with the new score rather than posting a duplicate comment.
*   **FR-WEBHOOK-07:** The system must handle webhook delivery failures gracefully by logging the error and not crashing.

### 4.4 Manual PR Analysis
*   **FR-MANUAL-01:** Authenticated users must be able to trigger a manual analysis for any open PR in a tracked repository directly from the dashboard.
*   **FR-MANUAL-02:** The dashboard must show the current status of a manual analysis job (queued, in progress, completed, failed).
*   **FR-MANUAL-03:** If a PR has already been analyzed, triggering a manual re-analysis must overwrite the previous result and update the GitHub comment.

### 4.5 Code Metrics Collection
*   **FR-METRIC-01:** The system must fetch the full list of changed files for a Pull Request using the GitHub REST API.
*   **FR-METRIC-02:** The system must record the total number of files changed in the PR.
*   **FR-METRIC-03:** The system must record the total number of lines added in the PR.
*   **FR-METRIC-04:** The system must record the total number of lines deleted in the PR.
*   **FR-METRIC-05:** The system must calculate the net lines changed (additions + deletions combined) as a single metric.
*   **FR-METRIC-06:** For each changed file that contains JavaScript or TypeScript code, the system must fetch the file's new content from the GitHub API.
*   **FR-METRIC-07:** The system must parse the fetched file content and calculate the cyclomatic complexity for each function or method in the file.
*   **FR-METRIC-08:** The system must record the average cyclomatic complexity across all analyzed functions in the PR.
*   **FR-METRIC-09:** The system must record the maximum cyclomatic complexity of any single function in the PR.
*   **FR-METRIC-10:** The system must record the count of functions whose cyclomatic complexity exceeds 10, which is the standard threshold for "complex" functions.
*   **FR-METRIC-11:** The system must handle files that cannot be parsed (e.g., minified files, syntax errors) gracefully by skipping them and logging a warning rather than failing the entire analysis.

### 4.6 Cognitive Load Scoring
*   **FR-SCORE-01:** The system must calculate a single composite Cognitive Load Score between 0 and 100 for each PR based on the collected metrics.
*   **FR-SCORE-02:** The scoring formula must incorporate the following weighted factors:
    *   Lines changed (normalized against a configurable maximum)
    *   Files changed (normalized against a configurable maximum)
    *   Average cyclomatic complexity (normalized against a configurable maximum)
    *   Maximum cyclomatic complexity of any single function
    *   Count of functions exceeding the complexity threshold
*   **FR-SCORE-03:** The system must assign a Difficulty Level to each PR based on the Cognitive Load Score using the following thresholds:
    *   **EASY** — Score 0 to 30 — routine change, quick review expected
    *   **MEDIUM** — Score 31 to 55 — moderate effort, some focused review needed
    *   **HARD** — Score 56 to 75 — significant effort, allocate dedicated review time
    *   **CRITICAL** — Score 76 to 100 — very high complexity, consider breaking the PR up
*   **FR-SCORE-04:** Authorized users (repository owners/admins) must be able to customize the scoring thresholds and weights per repository through the dashboard settings.
*   **FR-SCORE-05:** The system must store every computed score along with a timestamp so historical comparisons can be made.

### 4.7 Automated GitHub Comment
*   **FR-COMMENT-01:** After analysis is complete, the system must post a comment on the Pull Request in GitHub using the GitHub App bot account.
*   **FR-COMMENT-02:** The comment must include the Cognitive Load Score (numeric value out of 100).
*   **FR-COMMENT-03:** The comment must include the Difficulty Level label (EASY / MEDIUM / HARD / CRITICAL) with a corresponding color indicator or emoji.
*   **FR-COMMENT-04:** The comment must include a breakdown table showing each individual metric and its contribution to the score (files changed, lines changed, average complexity, max complexity, complex function count).
*   **FR-COMMENT-05:** The comment must include a plain-language summary interpreting the score, for example: "This PR touches 12 files and contains 3 functions with high cyclomatic complexity. Plan for a focused review session."
*   **FR-COMMENT-06:** The comment must include a timestamp indicating when the analysis was performed.
*   **FR-COMMENT-07:** If the PR was previously analyzed, the system must edit the existing comment rather than posting a new one.
*   **FR-COMMENT-08:** The comment must include a link to the full detailed analysis page on the dashboard.
*   **FR-COMMENT-09:** The comment must visually distinguish the bot's output from human review comments (e.g., using a branded header and footer).

### 4.8 Dashboard — Overview Page
*   **FR-DASH-01:** The dashboard must display a summary of all tracked repositories with the count of analyzed PRs and average difficulty scores.
*   **FR-DASH-02:** The dashboard must display a feed of the most recently analyzed PRs across all tracked repositories, ordered by analysis time.
*   **FR-DASH-03:** Each item in the recent feed must show the PR title, repository name, Difficulty Level badge, score, and a link to the PR on GitHub.
*   **FR-DASH-04:** The dashboard must display aggregate statistics including total PRs analyzed, average score across all repos, and distribution of difficulty levels (how many EASY, MEDIUM, HARD, CRITICAL).
*   **FR-DASH-05:** The overview page must update to reflect newly completed analyses without requiring a full page reload.

### 4.9 Dashboard — Repository Detail Page
*   **FR-REPO-DETAIL-01:** Each tracked repository must have a dedicated page listing all analyzed PRs for that repository.
*   **FR-REPO-DETAIL-02:** The PR list must be sortable by score, difficulty level, date analyzed, and PR number.
*   **FR-REPO-DETAIL-03:** The PR list must be filterable by difficulty level (show only HARD and CRITICAL PRs, for example).
*   **FR-REPO-DETAIL-04:** The page must display a trend chart showing how average PR complexity has changed over time for that repository.
*   **FR-REPO-DETAIL-05:** The page must display a bar chart showing the distribution of difficulty levels (how many PRs fall into each category) across all time or a configurable date range.
*   **FR-REPO-DETAIL-06:** The page must show the repository's current scoring configuration (weights and thresholds).

### 4.10 Dashboard — PR Analysis Detail Page
*   **FR-PR-DETAIL-01:** Each analyzed PR must have a dedicated detail page on the dashboard.
*   **FR-PR-DETAIL-02:** The detail page must show the PR title, author, link to GitHub, and open/closed status.
*   **FR-PR-DETAIL-03:** The detail page must display the full Cognitive Load Score with the Difficulty Level badge prominently.
*   **FR-PR-DETAIL-04:** The detail page must display a per-file breakdown showing each changed file, its individual complexity score, lines changed, and number of complex functions.
*   **FR-PR-DETAIL-05:** The detail page must display a chart visualizing the complexity distribution across all changed files in the PR.
*   **FR-PR-DETAIL-06:** The detail page must list all functions that exceed the complexity threshold, showing function name, file name, and complexity value.
*   **FR-PR-DETAIL-07:** The detail page must show the analysis history for the PR if it was analyzed multiple times (e.g., after new commits were pushed), allowing comparison of scores before and after.
*   **FR-PR-DETAIL-08:** The detail page must have a button to manually trigger a re-analysis.

### 4.11 Notifications and Alerts
*   **FR-NOTIFY-01:** Users must be able to configure email notifications to alert them when a PR in a tracked repository receives a CRITICAL difficulty score.
*   **FR-NOTIFY-02:** Users must be able to set a score threshold per repository above which they want to receive notifications.
*   **FR-NOTIFY-03:** Notifications must include the PR title, score, difficulty level, and a direct link to the GitHub PR.

---

## 5. Non-Functional Requirements

### 5.1 Performance
*   **NFR-PERF-01:** PR analysis jobs must complete within 2 minutes of being triggered under normal load.
*   **NFR-PERF-02:** Dashboard pages must load and display data within 2 seconds for repositories with up to 500 analyzed PRs.
*   **NFR-PERF-03:** The webhook endpoint must respond with an HTTP 200 acknowledgment within 500 milliseconds of receiving an event, before the analysis job is processed.
*   **NFR-PERF-04:** The system must be capable of processing up to 50 concurrent PR analysis jobs without degradation.
*   **NFR-PERF-05:** API responses for dashboard data must be cached where appropriate to reduce database load on repeated requests.

### 5.2 Reliability
*   **NFR-REL-01:** Failed analysis jobs must be automatically retried up to 3 times with exponential backoff before being marked as permanently failed.
*   **NFR-REL-02:** The system must log all failed analysis jobs with enough detail to diagnose and manually re-trigger them.
*   **NFR-REL-03:** GitHub API rate limiting must be handled gracefully — if the rate limit is hit, jobs must be paused and resumed when the limit resets rather than failing.
*   **NFR-REL-04:** The system must maintain 99.5% uptime for the webhook endpoint, as missed events will result in unanalyzed PRs.

### 5.3 Security
*   **NFR-SEC-01:** All webhook payloads must be verified using the GitHub-signed HMAC signature before processing.
*   **NFR-SEC-02:** GitHub access tokens and the GitHub App private key must be stored encrypted at rest, never in plaintext.
*   **NFR-SEC-03:** All API endpoints must be protected and require a valid authenticated session unless explicitly public (e.g., the webhook endpoint).
*   **NFR-SEC-04:** Users must only be able to access data for repositories they are authorized to view on GitHub — the system must verify this at the API level, not just the UI level.
*   **NFR-SEC-05:** All communication between the frontend, backend, and GitHub must occur over HTTPS/TLS.
*   **NFR-SEC-06:** The system must not store the raw content of analyzed source files — only the computed metrics derived from them.

### 5.4 Scalability
*   **NFR-SCALE-01:** The job queue architecture must allow additional worker instances to be added horizontally to handle increased analysis volume without code changes.
*   **NFR-SCALE-02:** The database schema must support efficient querying of PR analysis history as the dataset grows to millions of records.
*   **NFR-SCALE-03:** The system must support multiple organizations and repositories tracked by different users simultaneously without data leakage between accounts.

### 5.5 Usability
*   **NFR-USE-01:** A new user must be able to connect a repository and see their first PR analyzed within 10 minutes of signing up.
*   **NFR-USE-02:** The difficulty score and label on the GitHub comment must be immediately understandable to a developer with no prior knowledge of the tool.
*   **NFR-USE-03:** The dashboard must be fully functional on modern desktop browsers (Chrome, Firefox, Safari, Edge — latest 2 versions).
*   **NFR-USE-04:** The dashboard must be responsive and usable on tablet screen sizes.

### 5.6 Maintainability
*   **NFR-MAINT-01:** The backend and frontend must be maintained as separate deployable applications.
*   **NFR-MAINT-02:** All environment-specific configuration must be managed via environment variables, not hardcoded values.
*   **NFR-MAINT-03:** The scoring weights and thresholds must be configurable without a code deployment (stored in the database and editable via the dashboard).
*   **NFR-MAINT-04:** The system must expose structured application logs for all major operations (analysis start, analysis complete, comment posted, job failed).

---

## 6. User Stories

**Authentication**
*   As a developer, I want to log in with my GitHub account so I don't have to create a new account.
*   As a user, I want my session to persist so I don't have to log in every time I open the dashboard.

**Repository Management**
*   As a developer, I want to add my team's repository to the analyzer so all new PRs are scored automatically.
*   As an engineering manager, I want to remove a repository that is no longer active so the dashboard stays clean.

**Automatic Analysis**
*   As a reviewer, I want to see a difficulty score posted on every new PR automatically so I can prioritize my review queue.
*   As a PR author, I want to know the complexity score of my own PR so I can consider breaking it up before requesting review.
*   As a reviewer, I want the score to update when the PR author pushes new commits so I always have a current estimate.

**Dashboard**
*   As an engineering manager, I want to see which PRs in my team's repo are CRITICAL so I can assign senior reviewers to them.
*   As a tech lead, I want to see a trend of our repo's average PR complexity over the past month so I can tell if code quality is improving.
*   As a developer, I want to click through to the detailed analysis of a PR to understand exactly which files and functions are driving the complexity score.

**Configuration**
*   As a tech lead, I want to adjust the complexity thresholds for my repository because our team has a higher tolerance for large PRs than the defaults assume.

**Notifications**
*   As an engineering manager, I want to receive an email when a CRITICAL PR is opened so I can intervene before it sits in the review queue for days.

---

## 7. Integration Requirements

### 7.1 GitHub App
*   The system requires a GitHub App to be created and installed on target repositories.
*   The GitHub App must request read access to repository contents and pull requests, and write access to post pull request comments.
*   The GitHub App must be configured to receive webhook events for pull request activity.
*   The GitHub App's private key must be securely stored in the backend environment.

### 7.2 GitHub REST API
*   The system uses the GitHub REST API to fetch PR metadata (title, author, status, file list).
*   The system uses the GitHub REST API to fetch the content of changed files for complexity analysis.
*   The system uses the GitHub REST API to create and update comments on Pull Requests.
*   The system must respect GitHub's API rate limits (5,000 requests per hour for authenticated requests) and queue jobs accordingly.

### 7.3 GitHub OAuth
*   The system uses GitHub OAuth to authenticate human users logging into the web dashboard.
*   The OAuth flow must request the minimum necessary scopes: read user profile and read repository access.

---

## 8. Constraints and Assumptions
*   The system assumes all repositories being analyzed are hosted on GitHub.com (not GitHub Enterprise on-premise).
*   The complexity analysis in the initial version supports JavaScript and TypeScript files only.
*   The system requires Redis to be available for the job queue; this is a hard infrastructure dependency.
*   The system requires PostgreSQL for persistent storage; SQLite or other lightweight databases are not supported at this scale.
*   GitHub App installation requires a user with admin access to the target repository.
*   The system does not modify or merge code — it is read-only with the exception of posting comments.
*   Analysis accuracy depends on the quality and completeness of the GitHub diff API response.

---

## 9. Acceptance Criteria Summary

| Feature | Acceptance Criteria |
| :--- | :--- |
| **Login** | User can sign in with GitHub and is redirected to dashboard |
| **Add Repo** | User can add a repo and it appears in the tracked list |
| **Auto Analysis** | Within 2 minutes of opening a PR, a score comment appears on the GitHub PR |
| **Re-analysis** | Pushing new commits updates the existing comment, not a new one |
| **Score Accuracy** | Score reflects files changed, lines changed, and cyclomatic complexity |
| **Difficulty Label** | Every analyzed PR is assigned one of EASY / MEDIUM / HARD / CRITICAL |
| **Dashboard** | User can view all analyzed PRs with scores in the dashboard |
| **Detail Page** | User can see per-file complexity breakdown for any analyzed PR |
| **Trend Chart** | Repository page shows average score trend over time |
| **Threshold Config** | User can change scoring thresholds and the formula updates future analyses |
| **Notification** | User receives an email when a CRITICAL PR is detected |

> This document defines the complete product requirements for the PR Cognitive Load Analyzer. It can be used directly as the basis for technical specification, sprint planning, and QA test case creation.