# Jenkins Day 1 — Lab Output Explanation
## Understanding Your First Pipeline Execution Output

---

## The Most Important Thing to Understand First

You have **two separate machines** in this setup:

```
Your ThinkPad (Ubuntu)          Jenkins Container (Docker)
─────────────────────           ──────────────────────────
Your name: muskan               Username inside: jenkins
Your files: /home/muskan/...    Files: /var/jenkins_home/...
```

When you run a Jenkins pipeline, **Jenkins runs inside a Docker container** — not directly on your Ubuntu machine.

Think of it like this:

> You live in a house (Ubuntu). Inside your house, you have a **separate locked room** (Docker container).
> Jenkins lives inside that locked room.
> When Jenkins runs commands, it runs them **inside the room** — not in your house.

That is why `whoami` printed `jenkins` and not `muskan`.

---

## Your Actual Console Output — Every Line Explained

```
Started by user Jenkins Admin
```
→ You clicked **Build Now** in the Jenkins UI. Jenkins Admin = your Jenkins login user.

---

```
Obtained Jenkinsfile from git https://github.com/muskan7860/Devops-Practice-lab.git
```
→ Jenkins went to GitHub, found your `Jenkinsfile`, and read it.
**Before running a single stage**, Jenkins reads the entire file first.

---

```
Running on Jenkins in /var/jenkins_home/jobs/Day1_Declarative_Pipeline/workspace
```
→ Jenkins picked an agent to run on. Here, the agent **is the Jenkins master itself**
(because you have no separate agent set up yet).
The workspace folder it created is `/var/jenkins_home/jobs/Day1_Declarative_Pipeline/workspace`.
This is the folder where all your code will live during this build.

---

```
[Pipeline] stage
[Pipeline] { (Declarative: Checkout SCM)
[Pipeline] checkout
```
→ This stage you did **NOT write** in your Jenkinsfile. Jenkins added it **automatically**.
Whenever you use `agent any`, Jenkins runs an implicit checkout first.
It cloned your GitHub repo into the workspace folder.

---

```
Cloning repository https://github.com/muskan7860/Devops-Practice-lab.git
git init /var/jenkins_home/jobs/Day1_Declarative_Pipeline/workspace
git fetch ...
git checkout -f 1a6de47c8e6ce76ef50febdeae7d53ed5b332bd0
Commit message: "Clean up Jenkinsfile formatting"
```
→ Jenkins ran `git clone` behind the scenes.
It fetched your repo and checked out the exact commit `1a6de47` — the latest commit on `main`.
This is **your code now sitting inside the Jenkins container's workspace**.

---

```
[Pipeline] stage
[Pipeline] { (Hello)
[Pipeline] echo
stage 1: Hello from Jenkins!
```
→ Your first stage ran. `echo` simply printed the text. Nothing complex here.

---

```
[Pipeline] stage
[Pipeline] { (Who Am I)
[Pipeline] sh
+ whoami
jenkins
```
→ Here is the key moment.
`sh 'whoami'` ran a shell command **inside the Jenkins Docker container**.
The user running Jenkins inside that container is named `jenkins` — not `muskan`.
The `+` before `whoami` means Jenkins is showing you **the command it ran**.
The line below the `+` is the **output** of that command.

---

```
[Pipeline] sh
+ pwd
/var/jenkins_home/jobs/Day1_Declarative_Pipeline/workspace
```
→ `pwd` = print working directory.
Jenkins is currently sitting inside this workspace folder — which is inside the **Docker container's filesystem**.
Not your Ubuntu `/home/muskan/`.

---

```
[Pipeline] sh
+ ls -la
total 16
drwxr-xr-x 3 jenkins jenkins 4096 Jun 27 15:35 .
drwxr-xr-x 5 jenkins jenkins 4096 Jun 27 15:35 ..
drwxr-xr-x 8 jenkins jenkins 4096 Jun 27 15:35 .git
-rw-r--r-- 1 jenkins jenkins  857 Jun 27 15:35 Jenkinsfile
```
→ Only two things exist in the workspace:
- `.git` folder — created by the git clone
- `Jenkinsfile` — your pipeline file

That is correct — your repo only has those files.
The owner of both is `jenkins` — the container user.

---

```
[Pipeline] stage
[Pipeline] { (Environment Check)
[Pipeline] echo
Build Number: 2
Job Name: Day1_Declarative_Pipeline
Workspace: /var/jenkins_home/jobs/Day1_Declarative_Pipeline/workspace
Git Branch: origin/main
```
→ Jenkins automatically sets these environment variables for every build.
You do **not** set them — Jenkins sets them for you.
Build Number is `2` because you ran this job twice (first run = build 1, this run = build 2).

---

```
[Pipeline] { (Declarative: Post Actions)
[Pipeline] echo
All stages passed! Great job
```
→ All stages passed, so the `success` block inside `post` ran.
Jenkins calls it "Declarative: Post Actions" in the log.

---

```
Finished: SUCCESS
```
→ Pipeline completed with no failures. Every stage was green.

---

## Visual Flow of What Happened

```
You clicked Build Now
        ↓
Jenkins read Jenkinsfile from GitHub
        ↓
Jenkins created workspace folder inside Docker container
        ↓
Jenkins cloned your GitHub repo into that workspace
        ↓
Stage: Hello              → printed echo text
        ↓
Stage: Who Am I           → ran shell commands INSIDE the container
                             whoami = jenkins  (container user, NOT muskan)
                             pwd    = /var/jenkins_home/...  (container path)
                             ls     = showed only your repo files
        ↓
Stage: Environment Check  → printed Jenkins auto-set variables
        ↓
post { success }          → printed success message
        ↓
Finished: SUCCESS
```

---

## Why `jenkins` and Not `muskan`? — The Core Concept

| | Your Ubuntu Machine | Jenkins Docker Container |
|--|--|--|
| User | muskan | jenkins |
| Home directory | /home/muskan | /var/jenkins_home |
| Shell commands run here? | ❌ No | ✅ Yes |

When you run `sh 'whoami'` in Jenkins, it runs **inside the container** — so it shows `jenkins`.

---

## What is the `+` Symbol in the Logs?

Every time Jenkins runs a `sh` command, it prints two things:

```
+ whoami          ← the command Jenkins ran (prefixed with +)
jenkins           ← the output of that command
```

The `+` is Jenkins showing you **what it executed**, so you can debug easily.
If a command fails, you see exactly which command caused the failure.

---

## Jenkins Auto-Provided Environment Variables

These are set by Jenkins automatically — you never need to define them:

| Variable | What it Contains | Your Output |
|----------|-----------------|-------------|
| `env.BUILD_NUMBER` | Build run number | `2` |
| `env.JOB_NAME` | Name of the Jenkins job | `Day1_Declarative_Pipeline` |
| `env.WORKSPACE` | Full path of workspace folder | `/var/jenkins_home/jobs/...` |
| `env.GIT_BRANCH` | Git branch that triggered the build | `origin/main` |
| `env.BUILD_URL` | Full URL of this build in Jenkins UI | (shown in Blue Ocean) |

---

## What Happens to the Workspace After Build Finishes?

The workspace folder `/var/jenkins_home/jobs/Day1_Declarative_Pipeline/workspace` **stays on disk** after the build.

Next time you run the same job:
- Jenkins does **not** clone from scratch (by default)
- It does a `git fetch` + `git checkout` instead (faster)
- That is why the second build is faster than the first

To force a clean workspace every time, add this to your `post` block:
```groovy
post {
    always {
        cleanWs()   // deletes the workspace after every build
    }
}
```

---

## How to Connect Jenkins to Your Ubuntu Machine as an Agent (Day 4 Preview)

Right now, `sh 'whoami'` shows `jenkins` because Jenkins runs inside its own container.

On Day 4 (Agents & Nodes), you will learn to add your Ubuntu machine as a **Jenkins agent**.
Once connected:
- Jenkins master stays in the container
- Build jobs run **on your Ubuntu machine**
- `whoami` would show `muskan`
- `pwd` would show `/home/muskan/...`

That is the **Master-Agent architecture** — the Jenkins master schedules, agents execute.

---

## Summary — Key Takeaways from This Lab

| What You Saw | What It Means |
|---|---|
| `whoami` → `jenkins` | Commands run inside the Jenkins container, not Ubuntu |
| `pwd` → `/var/jenkins_home/...` | Jenkins workspace is inside the container filesystem |
| `ls` showed only `.git` and `Jenkinsfile` | Jenkins cloned your repo — only those files exist |
| `Build Number: 2` | This was your second run of this job |
| `Declarative: Checkout SCM` stage appeared automatically | Jenkins adds an implicit checkout stage when `agent any` is used |
| `+ whoami` prefix in logs | Jenkins shows the command before showing its output |
| `Finished: SUCCESS` | All stages passed, `post { success }` block ran |
