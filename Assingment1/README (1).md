# Assignment 1 — Jenkins Git Operations

## Overview

This assignment demonstrates how Jenkins can automate common Git branch operations using a **Freestyle Project**.

The Jenkins job accepts the required Git operation and branch names as parameters and performs the selected operation on the configured GitHub repository.

### Git operations implemented

- Create a branch
- List branches
- Merge one branch into another
- Rebase a branch onto another branch
- Delete a branch

The job is also configured to send **Slack and Email notifications when a build fails**.

---

# 1. Jenkins Global Configuration

Before creating the Jenkins job, the following global configurations were completed.

## 1.1 Slack Notification

Slack notification was configured from:

```text
Manage Jenkins → System → Slack Notifications
```

The Slack integration is used to notify the configured Slack channel when the Jenkins build fails.

> Slack was configured globally, so no additional Slack server configuration is required inside the job.
<img width="1791" height="443" alt="image" src="https://github.com/user-attachments/assets/6f2da12b-2924-4efc-a5c9-32c97715ffca" />


## 1.2 Gmail / Email Notification

Email notification was configured from:

```text
Manage Jenkins → System → E-mail Notification
```

Gmail SMTP was configured so Jenkins can send build notifications through the configured Gmail account.

The email notification is used to notify the configured recipient when the Jenkins build fails.
<img width="1685" height="621" alt="image" src="https://github.com/user-attachments/assets/77cf08f3-e37c-4de2-a3bf-f22127e4fa27" />


## 1.3 GitHub Credentials

GitHub credentials were configured from:

```text
Manage Jenkins
→ Credentials
→ System
→ Global credentials
```

A GitHub credential was created and later selected in the Jenkins job's Git SCM configuration.

The same credential was also bound to environment variables for use by the shell build step:

```text
GIT_USERNAME
GIT_PASSWORD
```

The actual credential values are not included in this README.
<img width="1595" height="247" alt="image" src="https://github.com/user-attachments/assets/7f2235bb-05fc-4fd4-b03c-c97005d30331" />

---

# 2. Jenkins Freestyle Project

A Freestyle Project was created with the name:

```text
Assignment-1Part1
```

The job is responsible for executing all five Git operations.

---

# 3. Parameterized Build

The project was configured as a parameterized Jenkins job.

Enable:

```text
This project is parameterized
```

Three parameters were created.

## 3.1 OPERATION

Type:

```text
Choice Parameter
```

Name:

```text
OPERATION
```

Choices:

```text
create
list
merge
rebase
delete
```

This parameter determines which Git operation Jenkins executes.
<img width="1261" height="460" alt="image" src="https://github.com/user-attachments/assets/ef7654a5-d036-45e2-b2c9-30b0726f99d2" />

## 3.2 SOURCE_BRANCH

Type:

```text
String Parameter
```

Name:

```text
SOURCE_BRANCH
```
<img width="1257" height="458" alt="image" src="https://github.com/user-attachments/assets/dcc53f18-ec9d-4b16-960d-e678778f8b28" />

This parameter represents the source branch used by:

- Create
- Merge
- Rebase
- Delete

## 3.3 TARGET_BRANCH

Type:

```text
String Parameter
```

Name:

```text
TARGET_BRANCH
```
<img width="1250" height="437" alt="image" src="https://github.com/user-attachments/assets/b4049ced-bec5-4699-9a44-26f8dab8890e" />

This parameter represents the target branch used by:

- Merge
- Rebase

It is not required for:

- Create
- List
- Delete

For example:

```text
OPERATION      = merge
SOURCE_BRANCH  = feature-1
TARGET_BRANCH  = main
```

means:

```text
feature-1 → merge → main
```

---

# 4. Source Code Management

The job uses Git under:

```text
Source Code Management → Git
```

## Repository

```text
https://github.com/Garvit-singhal-04/git-repo.git
```

## Credentials

The previously configured GitHub credential is selected.

## Branch to Build

The Jenkins job builds from:

```text
*/main
```

Jenkins therefore checks out the `main` branch before executing the shell script.

---
<img width="1316" height="681" alt="image" src="https://github.com/user-attachments/assets/8f749565-347e-4667-af29-ab95c205bf85" />

# 5. Credential Binding

Under:

```text
Build Environment
```

the following option was enabled:

```text
Use secret text(s) or file(s)
```

A:

```text
Username and password (separated)
```

binding was configured.

### Variables

```text
Username Variable: GIT_USERNAME
Password Variable: GIT_PASSWORD
Credentials: GitHub credential
```

These variables allow the shell script to authenticate when pushing changes to GitHub.
<img width="1325" height="361" alt="image" src="https://github.com/user-attachments/assets/af90718a-c5a4-4f1f-b351-4b6562379a09" />

---

# 6. Build Step

Under:

```text
Build Steps → Execute shell
```

the following script was configured:

```bash
case "$OPERATION" in
    create)
        git checkout -b "$SOURCE_BRANCH"
        git push "https://$GIT_USERNAME:$GIT_PASSWORD@github.com/Garvit-singhal-04/git-repo.git" "$SOURCE_BRANCH"
        ;;

    list)
        git branch -a
        ;;

    merge)
        git checkout "$TARGET_BRANCH"
        git merge "$SOURCE_BRANCH"
        git push "https://$GIT_USERNAME:$GIT_PASSWORD@github.com/Garvit-singhal-04/git-repo.git" "$TARGET_BRANCH"
        ;;

    rebase)
        git fetch origin
        git checkout -B "$SOURCE_BRANCH" "origin/$SOURCE_BRANCH"
        git rebase "origin/$TARGET_BRANCH"
        git fetch origin "$SOURCE_BRANCH"
        git push --force-with-lease "https://$GIT_USERNAME:$GIT_PASSWORD@github.com/Garvit-singhal-04/git-repo.git" "$SOURCE_BRANCH"
        ;;

    delete)
        git push "https://$GIT_USERNAME:$GIT_PASSWORD@github.com/Garvit-singhal-04/git-repo.git" --delete "$SOURCE_BRANCH"
        ;;

    *)
        echo "Invalid operation"
        exit 1
        ;;
esac
```

---

# 7. Script Explanation

## Create

```bash
git checkout -b "$SOURCE_BRANCH"
git push "https://..." "$SOURCE_BRANCH"
```

Creates a new local branch and pushes it to the GitHub repository.

## List

```bash
git branch -a
```

Displays local and remote branches available in the Jenkins workspace.

## Merge

```bash
git checkout "$TARGET_BRANCH"
git merge "$SOURCE_BRANCH"
git push "https://..." "$TARGET_BRANCH"
```

Switches to the target branch, merges the source branch into it, and pushes the updated target branch to GitHub.

## Rebase

```bash
git checkout "$SOURCE_BRANCH"
git fetch origin
git rebase "origin/$TARGET_BRANCH"
git push --force-with-lease "https://..." "$SOURCE_BRANCH"
```

Switches to the source branch, fetches the latest remote information, rebases the source branch onto the target branch, and pushes the rewritten history.

`--force-with-lease` is used because a rebase can rewrite commit history.

## Delete

```bash
git push "https://..." --delete "$SOURCE_BRANCH"
```

Deletes the specified branch from the GitHub remote repository.

## Invalid Operation

```bash
echo "Invalid operation"
exit 1
```

If an unsupported operation is supplied, the shell exits with status `1`. Jenkins therefore marks the build as failed, which triggers the configured Slack and Email notifications.

---

# 8. Testing

Each operation should be tested separately using **Build with Parameters**.

The tests below demonstrate the expected behavior of the Jenkins job.

---

## Test 1 — List Branches

### Parameters

```text
OPERATION      = list
SOURCE_BRANCH  = leave empty
TARGET_BRANCH  = main
```

### Expected behavior

Jenkins executes:

```bash
git branch -a
```

The console output should display local and remote branches.

Expected final status:

```text
Finished: SUCCESS
```

### Screenshot

<img width="580" height="305" alt="image" src="https://github.com/user-attachments/assets/787cbb26-b46e-43f1-af79-7fa38ea4ca4e" />

---

## Test 2 — Create Branch

### Parameters

```text
OPERATION      = create
SOURCE_BRANCH  = jenkins-create-test12
TARGET_BRANCH  = main
```

`TARGET_BRANCH` is ignored by the `create` operation.

### Expected behavior

Jenkins creates:

```text
jenkins-create-test
```

and pushes it to GitHub.

Expected console output includes:

```text
Switched to a new branch 'jenkins-create-test'
```

and a successful push.

Expected final status:

```text
Finished: SUCCESS
```

### Verification

Check the GitHub repository and confirm that `jenkins-create-test` exists.

### Screenshot


<img width="791" height="268" alt="image" src="https://github.com/user-attachments/assets/c293d06d-c143-42c5-adb1-9bf4ddaf629a" />
<img width="1275" height="112" alt="image" src="https://github.com/user-attachments/assets/2c89594a-f514-4ed5-b581-ce4a824584d4" />

---

## Test 3 — Merge Branch

First create a separate test branch and add at least one commit to it.

Example branch:

```text
jenkins-git-test
```

Then run the Jenkins job.

### Parameters

```text
OPERATION      = merge
SOURCE_BRANCH  = git-merge-test
TARGET_BRANCH  = main
```


### Verification

Check the `main` branch on GitHub and confirm that the changes from the source branch are present.

### Screenshot


<img width="1197" height="230" alt="image" src="https://github.com/user-attachments/assets/43be28f8-bec4-4222-9574-64c58d8eee76" />
<img width="707" height="412" alt="image" src="https://github.com/user-attachments/assets/322d2dcb-769a-464f-bd70-25a210fc663c" />


---

## Test 4 — Rebase Branch

Create a separate test branch:

```text
jenkins-rebase-test
```

Add a commit to it.

For a meaningful rebase test, `main` should also contain a commit that is not already present in the test branch.

### Parameters

```text
OPERATION      = rebase
SOURCE_BRANCH  = jenkins-rebase-test
TARGET_BRANCH  = main
```

### Screenshot


<img width="751" height="195" alt="image" src="https://github.com/user-attachments/assets/ed65d16e-996d-4285-a92f-6f486ceb1005" />



---

## Test 5 — Delete Branch

Use a test branch created specifically for deletion.

### Parameters

```text
OPERATION      = delete
SOURCE_BRANCH  = jenkins-create-test
TARGET_BRANCH  = main
```

Expected final status:

```text
Finished: SUCCESS
```

### Screenshot


<img width="820" height="190" alt="image" src="https://github.com/user-attachments/assets/206055b5-c410-42f9-bd96-af9b3d26b047" />



---

## Test 7 — Slack Failure Notification

The failed build from Test 6 should trigger the configured Slack notification.

### Expected behavior

A Jenkins failure notification should appear in the configured Slack channel.

### Screenshot

<img width="822" height="132" alt="image" src="https://github.com/user-attachments/assets/1a4e5642-0f39-4c4d-86e3-810bce37348a" />



---

## Test 8 — Email Failure Notification

The failed build from Test 6 should also trigger the configured email notification.

### Expected behavior

The configured Gmail recipient should receive a Jenkins build failure email.

### Screenshot

```markdown
<img width="850" height="757" alt="image" src="https://github.com/user-attachments/assets/10fc9827-a590-40c7-94e8-576d1e6517a9" />

```

---

# 9. Test Summary

| Test | Operation | Expected Result |
|---|---|---|
| 1 | `list` | Branches displayed, build succeeds |
| 2 | `create` | New branch created on GitHub |
| 3 | `merge` | Source branch merged into target branch |
| 4 | `rebase` | Source branch rebased onto target branch |
| 5 | `delete` | Remote branch deleted |
| 6 | Invalid operation | Jenkins build fails |
| 7 | Slack notification | Failure notification received in Slack |
| 8 | Email notification | Failure notification received by email |

---

# 10. Jenkins Job Flow

```text
                  Jenkins
                     |
                     v
             Build with Parameters
                     |
                     v
               Select OPERATION
                     |
        +------------+------------+
        |            |            |
      create        list        merge
        |            |            |
      rebase       delete      target branch
        |            |            |
        +------------+------------+
                     |
                     v
               Execute Shell
                     |
                     v
              GitHub Repository
                     |
              +------+------+
              |             |
           SUCCESS        FAILURE
                            |
                       +----+----+
                       |         |
                     Slack     Email
```

---

# 11. Important Notes

- `TARGET_BRANCH` is only meaningful for `merge` and `rebase`.
- `SOURCE_BRANCH` is required for `create`, `merge`, `rebase`, and `delete`.
- `list` does not require either branch parameter.
- An authenticated push is required when an operation needs to modify the remote GitHub repository.
- `git rebase` can rewrite commit history, which is why the script uses `--force-with-lease`.
- Jenkins automatically marks the build as failed when the shell exits with a non-zero status.
- Slack and Email notifications are configured as post-build actions.
- GitHub credentials must not be committed to the repository or written directly into the README.

---

# 12. Result

The Jenkins Freestyle project provides a parameterized interface for automating:

```text
Create → List → Merge → Rebase → Delete
```

and provides failure notifications through:

```text
Slack + Email
```
