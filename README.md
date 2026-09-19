# CI/CD Assignment 5 – Scripted Pipeline for Spring3Hibernate (Java)

Submitted by Jeetendra Singh

Same requirements as Assignment 4 (checkout, parallel stability/quality/coverage checks, Quality Gate, report, manual approval before publish, Slack + Email notifications), but this time written as a **Scripted pipeline** (imperative Groovy inside a `node {}` block) instead of a Declarative one - and reusing the SonarQube server, GitHub, Slack, and Gmail setup already configured in Jenkins from Assignment 4.

## Created the pipeline job

```
Jenkins dashboard → New Item → Name: spring3hibernate-scripted-ci → Type: Pipeline
```

<img width="1440" height="900" alt="Screenshot 2026-09-18 at 11 32 07 PM" src="https://github.com/user-attachments/assets/cfd5b1d4-3918-4a80-b9e8-b4c81c407445" />


## Wrote the pipeline as a Scripted script

Went with the "Pipeline script" definition instead of pulling from SCM, and wrote it in scripted syntax - `properties([parameters([...])])` up top to define the skip-stage options, then a single `node { }` block containing the stage logic (as opposed to Declarative's `pipeline { stages { stage(...) { steps { } } } }` structure from Assignment 4).

The four boolean parameters give the user the option to skip any scan independently:

* `SKIP_UNIT_TESTS` → skip the Code Stability stage
* `SKIP_SONAR` → skip Code Quality Analysis (SonarQube)
* `SKIP_COVERAGE` → skip Code Coverage Analysis
* `SKIP_PUBLISH` → skip the Publish Artifacts stage entirely

Also defined `SONARQUBE_ENV` and `SLACK_CHANNEL` as script variables at the top, pointing at the same `MySonarQube` server and `#jenkins-ci-alerts` channel used in Assignment 4.


<img width="1440" height="900" alt="Screenshot 2026-09-19 at 9 21 17 AM" src="https://github.com/user-attachments/assets/7d6c648f-58cf-4e59-9f26-e9163010b831" />


## First run - success on build #1

Full stage view confirms every required stage ran in the scripted pipeline: **Tool Install → Code Checkout → Build → Build & Analysis (the parallel stability/quality/coverage block) → Quality Gate → Generate Report → Approval for Publish → Publish Artifacts → Notifications**. SonarQube Quality Gate for `Spring3HibernateApp` came back **Passed**, and the built WAR (`Spring3HibernateApp.war`, 20.59 MiB) was archived as the last successful artifact - meaning the manual approval step was accepted and the publish stage actually executed.


<img width="1440" height="900" alt="Screenshot 2026-09-19 at 9 23 45 AM" src="https://github.com/user-attachments/assets/049db96c-9513-4222-b29f-0090a4fc4e3d" />


## Notifications

### Slack

```
Job spring3hibernate-scripted-ci #1 -> SUCCESS
Publish decision: Approve
```

Same notification format as the declarative pipeline in Assignment 4 - confirms the scripted version also reports the approval outcome, not just the build status.

### Email

Matching email with the build status and publish decision in the subject line, `build.log` attached for reference.

## Scripted vs Declarative (Assignment 4)

Both pipelines satisfy the same requirements, but structured differently:

| Declarative (Assignment 4) | Scripted (this assignment)                                 |                                                                                          |
| -------------------------- | ---------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Syntax                     | `pipeline { agent {} stages { stage('X') { steps {} } } }` | `node { ... }` with plain Groovy control flow                                            |
| Skip logic                 | `when` conditions per stage                                | `if (!params.SKIP_X) { ... }` around each stage's logic                                  |
| Parameters                 | `parameters {}` block inside `pipeline {}`                 | `properties([parameters([...])])` called before `node {}`                                |
| Flexibility                | Structured, easier to read, built-in validation            | More flexible - full Groovy scripting available, but requires more manual error handling |

Functionally, both pipelines satisfy the same requirements: parallel checks, a SonarQube Quality Gate, a manual approval gate before publishing, and Slack/Email notifications that include the approval decision.

