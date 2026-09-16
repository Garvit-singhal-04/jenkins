# Jenkins Assignment 1 — Part 2

## Overview

This assignment demonstrates Jenkins automation using parameterized builds, artifact archiving, artifact transfer, job chaining, Nginx web-server publishing, Slack notifications, and Email notifications.

Part 2 consists of two Jenkins Freestyle projects:

1. **Job 1:** Accepts a `NINJA_NAME` parameter and creates a file containing `<Ninja Name> from DevOps Ninja`.
2. **Job 2:** Retrieves the file created by Job 1 and publishes it through an Nginx web server.
3. Job 2 is automatically triggered only when Job 1 completes successfully.
4. Slack and Email notifications are configured for successful and failed builds.

## Assignment Requirement

### Job 1

Create a Jenkins job that:
- Takes a string parameter `<Ninja Name>`.
- Creates a file.
- Adds `<Ninja Name> from DevOps Ninja` to the file.

### Job 2

Create another Jenkins job that:
- Retrieves the file created by Job 1.
- Publishes the file using a web server.
- Is automatically triggered after Job 1 completes successfully.
- Sends Slack and Email notifications when builds succeed or fail.

## Architecture

```text
User
  |
  | NINJA_NAME
  v
Assignment1-Part2-Job1
  |
  | Create ninja.txt
  | Archive Artifact
  v
ninja.txt
  |
  | Job 1 SUCCESS
  v
Assignment1-Part2-Job2
  |
  | Copy Artifact
  | Publish using Nginx
  v
/var/www/html/ninja.txt
  |
  v
Nginx
  |
  v
http://localhost/ninja.txt
```

## Technologies Used

- Jenkins
- Jenkins Freestyle Projects
- Jenkins String Parameters
- Jenkins Artifact Archiving
- Jenkins Copy Artifact Plugin
- Jenkins Build Triggers
- Shell
- Nginx
- Slack Notifications
- Email Notifications

# Job 1 — File Creation

## Job Name

```text
Assignment1-Part2-Job1
```

Job 1 accepts the Ninja Name and creates `ninja.txt`.

## 1. Create the Jenkins Job

From the Jenkins dashboard:

```text
New Item
```

Enter:

```text
Assignment1-Part2-Job1
```

Select:

```text
Freestyle project
```

Click **OK**.

## 2. Configure the Parameter

Enable:

```text
This project is parameterized
```

Add:

```text
String Parameter
```

Configure:

```text
Name:
NINJA_NAME

Default Value:
Garvit

Description:
Enter the Ninja Name
```

The parameter is available to the shell as:

```bash
$NINJA_NAME
```
<img width="1292" height="447" alt="image" src="https://github.com/user-attachments/assets/c4a225ae-9851-4ac1-a835-dba08e72dd4a" />

## 3. Build Step

Under:

```text
Build Steps → Execute shell
```

configure:

```bash
echo "$NINJA_NAME from DevOps Ninja" > ninja.txt
```

For example, if:

```text
NINJA_NAME = Garvit
```

the generated file will contain:

```text
Garvit from DevOps Ninja
```
<img width="1353" height="402" alt="image" src="https://github.com/user-attachments/assets/56ceb3bd-cf83-4559-8d54-9a96a6e7c43b" />

## 4. Archive the File

Under:

```text
Post-build Actions
```

add:

```text
Archive the artifacts
```

Set:

```text
Files to archive:
ninja.txt
```
<img width="1320" height="310" alt="image" src="https://github.com/user-attachments/assets/0032a1b9-2e08-447a-a0f0-8604f9d93e10" />

This allows Job 2 to retrieve the file from Job 1.

# Job 2 — Publish the File

## Job Name

```text
Assignment1-Part2-Job2
```

Job 2 retrieves `ninja.txt` from Job 1 and publishes it through Nginx.

## 5. Configure Artifact Copying

Under:

```text
Build Steps
```

add:

```text
Copy artifacts from another project
```

Configure:

```text
Project name:
Assignment1-Part2-Job1
```

Configure:

```text
Which build:
Latest successful build
```

Set:

```text
Artifacts to copy:
ninja.txt
```
<img width="1376" height="407" alt="image" src="https://github.com/user-attachments/assets/cb3da349-0db9-47ad-bf2d-59573d1d25b1" />


## 6. Install Nginx

On Ubuntu, if Nginx is not already installed:

```bash
sudo apt update
sudo apt install nginx -y
```

Start Nginx:

```bash
sudo systemctl start nginx
```

Check its status:

```bash
sudo systemctl status nginx
```

Expected state:

```text
active (running)
```

## 7. Nginx Web Root

The default Nginx web root is:

```text
/var/www/html
```

A file placed there, such as:

```text
/var/www/html/ninja.txt
```

will be available at:

```text
http://localhost/ninja.txt
```

## 8. Configure Jenkins Permissions

Jenkins runs under the `jenkins` user. The Jenkins user needs permission to copy the artifact into the Nginx web root.

Configure:

```bash
sudo chown -R jenkins:jenkins /var/www/html
sudo chmod -R 755 /var/www/html
```

Verify:

```bash
ls -ld /var/www/html
```
<img width="1225" height="95" alt="image" src="https://github.com/user-attachments/assets/3f73b8b2-8288-4f49-94bc-1164deeefdb1" />

A permission test can be performed with:

```bash
sudo -u jenkins touch /var/www/html/test.txt
```

Remove the test file afterward:

```bash
sudo rm /var/www/html/test.txt
```

## 9. Job 2 Build Step

After the Copy Artifact step, add:

```text
Build Steps → Execute shell
```

Use:

```bash
cp ninja.txt /var/www/html/ninja.txt
```

This copies the artifact into the Nginx web root.
<img width="1286" height="310" alt="image" src="https://github.com/user-attachments/assets/814fa08a-9d05-453e-a56b-2f3edd33a626" />

# Automatic Job Trigger

## 10. Configure Job 2 Trigger

Go to:

```text
Assignment1-Part2-Job2
→ Configure
```

Under:

```text
Build Triggers
```

enable:

```text
Build after other projects are built
```

Set:

```text
Projects to watch:
Assignment1-Part2-Job1
```

<img width="1112" height="466" alt="image" src="https://github.com/user-attachments/assets/6400e2ec-71d9-4e3e-ad98-03671d085a4d" />
Configure the trigger so that Job 2 runs only when Job 1 completes successfully/stably.

The resulting behavior is:

```text
Job 1 SUCCESS
      |
      v
Job 2 automatically starts
```

If Job 1 fails:

```text
Job 1 FAILURE
      |
      v
Job 2 does not start
```

# Notifications

Slack and Email notifications were already configured in Jenkins and are configured for these jobs.

## Slack

Successful build:

```text
Build SUCCESS
     |
     v
Slack Notification
```

Failed build:

```text
Build FAILURE
     |
     v
Slack Notification
```

## Email

Successful build:

```text
Build SUCCESS
     |
     v
Email Notification
```

Failed build:

```text
Build FAILURE
     |
     v
Email Notification
```
<img width="1233" height="610" alt="image" src="https://github.com/user-attachments/assets/9fdb6541-3152-45ef-bc52-bc076136d266" />

# Complete Workflow

```text
              Build with Parameters
                       |
                       v
                NINJA_NAME=Garvit
                       |
                       v
            Assignment1-Part2-Job1
                       |
                       v
       echo "$NINJA_NAME from DevOps Ninja"
                       |
                       v
                  ninja.txt
                       |
                       v
               Archive Artifact
                       |
                       v
                    SUCCESS
                       |
               Automatic Trigger
                       |
                       v
            Assignment1-Part2-Job2
                       |
                       v
                 Copy Artifact
                       |
                       v
      cp ninja.txt /var/www/html/ninja.txt
                       |
                       v
                     Nginx
                       |
                       v
          http://localhost/ninja.txt
                       |
                       v
             Garvit from DevOps Ninja
```

# Failure Handling

## Job 1 Failure

If Job 1 fails:

```text
Job 1
  |
  v
FAILURE
  |
  +---- Slack
  |
  +---- Email
```

Job 2 is not triggered because it is configured to run only after a successful/stable Job 1 build.

## Job 2 Failure

If Job 1 succeeds but Job 2 fails:

```text
Job 1
  |
  v
SUCCESS
  |
  v
Job 2
  |
  v
FAILURE
  |
  +---- Slack
  |
  +---- Email
```

# Testing

## Test 1 — File Creation

Run Job 1 with:

```text
NINJA_NAME = Garvit
```

Expected:

```text
ninja.txt
```

Content:

```text
Garvit from DevOps Ninja
```

## Test 2 — Different Parameter

Run Job 1 with:

```text
NINJA_NAME = Ninja01
```

Expected:

```text
Ninja01 from DevOps Ninja
```

This verifies that the file content is generated dynamically from the Jenkins parameter.

## Test 3 — Artifact Archiving

After a successful Job 1 build, verify:

```text
Build → Artifacts
```

contains:

```text
ninja.txt
```

## Test 4 — Artifact Transfer

Run Job 2 and verify that:

```text
ninja.txt
```

is available in the Job 2 workspace.

## Test 5 — Web Server Publishing

Run:

```bash
cat /var/www/html/ninja.txt
```

Expected:

```text
Garvit from DevOps Ninja
```

Then:

```bash
curl http://localhost/ninja.txt
```

Expected:

```text
Garvit from DevOps Ninja
```

## Test 6 — Automatic Trigger

Run Job 1 using:

```text
NINJA_NAME = Garvit
```

Do not manually start Job 2.

Expected:

```text
Job 1
  |
  v
SUCCESS
  |
  v
Job 2 automatically triggered
  |
  v
SUCCESS
```

## Test 7 — Job 1 Failure

Temporarily add:

```bash
exit 1
```

to Job 1.

Expected:

```text
Job 1 FAILURE
      |
      +---- Slack notification
      |
      +---- Email notification
```

Job 2 should not be triggered.

Remove the failing command after testing.

## Test 8 — Job 2 Failure

Temporarily change the Job 2 copy command to:

```bash
cp ninja.txt /invalid/path/ninja.txt
```

Run Job 1 successfully.

Expected:

```text
Job 1 SUCCESS
      |
      v
Job 2 automatically triggered
      |
      v
Job 2 FAILURE
      |
      +---- Slack notification
      |
      +---- Email notification
```

Restore the correct command afterward:

```bash
cp ninja.txt /var/www/html/ninja.txt
```

# Final Configuration

## Assignment1-Part2-Job1

```text
Type:
Freestyle Project

Parameterized:
Yes

Parameter:
NINJA_NAME

Build Step:
echo "$NINJA_NAME from DevOps Ninja" > ninja.txt

Post-build Action:
Archive ninja.txt

Notifications:
Slack
Email
```

## Assignment1-Part2-Job2

```text
Type:
Freestyle Project

Parameterized:
No

Build Trigger:
After Assignment1-Part2-Job1 succeeds

Build Step 1:
Copy ninja.txt from Assignment1-Part2-Job1

Build Step 2:
cp ninja.txt /var/www/html/ninja.txt

Web Server:
Nginx

Notifications:
Slack
Email
```

# Screenshot Checklist

The following screenshots can be included as evidence:

1. Job 1 parameter configuration
2. Job 1 Execute Shell configuration
3. Job 1 artifact archive configuration
4. Successful Job 1 build
5. `ninja.txt` listed under Job 1 artifacts
6. Job 2 Copy Artifact configuration
7. Job 2 automatic build trigger configuration
8. Job 2 Execute Shell configuration
9. Job 1 automatically triggering Job 2
10. Successful Job 2 console output
11. Nginx/browser showing `ninja.txt`
12. Slack success notification
13. Email success notification
14. Job 1 failure notification
15. Job 2 failure notification

# Result

The assignment successfully demonstrates:

- Jenkins parameterized builds
- Shell scripting
- Dynamic file creation
- Artifact archiving
- Artifact transfer between Jenkins jobs
- Upstream/downstream Jenkins job triggering
- Conditional job execution
- Nginx web-server publishing
- Slack notifications
- Email notifications
- Success and failure handling
- End-to-end Jenkins automation

The generated file is published through:

```text
http://localhost/ninja.txt
```

with content in the format:

```text
<Ninja Name> from DevOps Ninja
```
