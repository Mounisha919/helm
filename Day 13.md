Perfect 😎 Mounisha — welcome to **🎯 Day 10 of your Helm course!**
Today we’re learning one of Helm’s most powerful and slightly advanced features — **Helm Hooks & Lifecycle Management**.
Let’s go step-by-step nice and slow 👇

---

## 🌟 **Day 10 — Helm Hooks & Lifecycle**

---

### 🧠 **1️⃣ What Are Helm Hooks?**

Helm hooks are *special instructions* that tell Helm to run certain Kubernetes resources **before or after** a release event such as:

* `helm install`
* `helm upgrade`
* `helm delete`
* `helm test`
* `helm rollback`

They’re very useful when you want to **run specific jobs** at certain points during your chart’s lifecycle.

---

### ⚙️ **2️⃣ Why Are Hooks Useful?**

Let’s imagine:

* You want to run a **database migration job** before deploying your app.
* You want to run a **cleanup script** before uninstalling your app.
* You want to run **tests** after install or upgrade.

Helm hooks make all this possible ✅.

---

### 🧩 **3️⃣ Helm Lifecycle Events**

Here are the main hook types Helm supports:

| Hook Name       | When It Runs                       |
| --------------- | ---------------------------------- |
| `pre-install`   | Before any resources are installed |
| `post-install`  | After all resources are installed  |
| `pre-delete`    | Before Helm deletes resources      |
| `post-delete`   | After resources are deleted        |
| `pre-upgrade`   | Before upgrade starts              |
| `post-upgrade`  | After upgrade finishes             |
| `pre-rollback`  | Before rollback starts             |
| `post-rollback` | After rollback finishes            |
| `test`          | When you run `helm test`           |

---

### 🧾 **4️⃣ Syntax — How to Define a Hook**

You define hooks using **annotations** inside your Kubernetes manifest file.

Example:
Let’s say you want to run a Kubernetes Job **before installing** your chart.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pre-install-job
  annotations:
    "helm.sh/hook": pre-install
spec:
  template:
    spec:
      containers:
        - name: pre-install-container
          image: busybox
          command: ['sh', '-c', 'echo Pre-install job running! && sleep 5']
      restartPolicy: Never
```

👉 This tells Helm:

> “Run this Job before installing the rest of the release.”

---

### 💡 **5️⃣ Multiple Hooks**

You can even assign multiple hooks to the same resource:

```yaml
annotations:
  "helm.sh/hook": pre-install,pre-upgrade
```

This means the same job will run both **before install** and **before upgrade**.

---

### 🧹 **6️⃣ Deletion Policies**

By default, Helm will delete hook resources after they complete.
But you can control this behavior using:

```yaml
annotations:
  "helm.sh/hook-delete-policy": hook-succeeded
```

Available values:

| Policy                       | Meaning                                 |
| ---------------------------- | --------------------------------------- |
| `before-hook-creation`       | Delete old hook before creating new one |
| `hook-succeeded`             | Delete hook after successful completion |
| `hook-failed`                | Delete hook if it fails                 |
| `hook-succeeded,hook-failed` | Delete hook after success or failure    |

---

### 🧪 **7️⃣ Testing Example**

You can define a **test hook** to run tests after installation:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-connection
  annotations:
    "helm.sh/hook": test
spec:
  containers:
  - name: curl
    image: curlimages/curl
    command: ["curl", "http://myapp:8080/health"]
  restartPolicy: Never
```

You run this test with:

```bash
helm test my-release
```

---

### 🎯 **8️⃣ Real Use Cases**

| Scenario                                       | Hook Type                |
| ---------------------------------------------- | ------------------------ |
| Run DB migration before install                | `pre-install`            |
| Run cleanup before uninstall                   | `pre-delete`             |
| Run smoke tests after install                  | `post-install` or `test` |
| Delete temporary resources after job completes | `hook-delete-policy`     |

---

### 🧩 **9️⃣ Summary**

| Concept       | Description                                                       |
| ------------- | ----------------------------------------------------------------- |
| Hook          | A special annotation that triggers resource during Helm lifecycle |
| Common hooks  | pre-install, post-install, pre-delete, post-upgrade, test         |
| Purpose       | Automate DB migrations, cleanup, testing, etc.                    |
| Syntax        | `"helm.sh/hook": <hook-type>`                                     |
| Delete policy | Controls whether hook resources are deleted                       |

---

### 💬 **Example Output**

When you install the chart with `helm install myapp .`, you’ll see logs like:

```
HOOKS:
---
# pre-install
NAME: pre-install-job
Last Started: ...
State: succeeded
---
# main resources
...
Release "myapp" has been successfully deployed!
```

---
Awesome 👏 Mounisha! Let’s do a **real practical example** to help you *see exactly how Helm hooks work in action.*

We’ll go very slow and simple — no rush 🐢

---

## ⚙️ **Day 10 – Practical Example: Pre-install Hook**

### 🎯 Goal

We’ll create a small Helm chart that:

1. Deploys a **pre-install Job** (runs before installation).
2. Then deploys a **simple NGINX app**.

---

### 🪄 **Step 1: Create a new chart**

```bash
helm create hook-demo
```

Now go inside it:

```bash
cd hook-demo
```

---

### 🧾 **Step 2: Delete unnecessary files**

To make it simple, delete these:

```bash
rm -rf templates/tests
rm templates/ingress.yaml
```

---

### 🧱 **Step 3: Create a pre-install job**

Create a new file:

```bash
nano templates/pre-install-job.yaml
```

Paste this YAML 👇

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ .Release.Name }}-preinstall-job"
  annotations:
    "helm.sh/hook": pre-install
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    spec:
      containers:
        - name: pre-install
          image: busybox
          command: ['sh', '-c', 'echo "🚀 Pre-install job running before deployment!" && sleep 5']
      restartPolicy: Never
```

🧠 This means:

* Helm will **run this Job first**.
* It will then **delete the Job** after it completes successfully.

---

### 🧱 **Step 4: Edit your main Deployment**

Open:

```bash
nano templates/deployment.yaml
```

Make sure it looks simple, like this 👇

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
      - name: {{ .Release.Name }}
        image: nginx
        ports:
        - containerPort: 80
```

---

### 🧪 **Step 5: Install the chart**

Run:

```bash
helm install myapp .
```

You will see:

```
HOOKS:
---
# pre-install
NAME: myapp-preinstall-job
LAST STARTED: ...
STATE: succeeded
---
# main resources
NAME: myapp
NAMESPACE: default
STATUS: deployed
```

✅ Meaning:

* Helm **executed the pre-install hook job** first.
* Then it **installed your main application**.

---

### 🧹 **Step 6: Check what happened**

Check the job logs:

```bash
kubectl logs job/myapp-preinstall-job
```

You’ll see:

```
🚀 Pre-install job running before deployment!
```

Now check your deployment:

```bash
kubectl get pods
```

You’ll find your NGINX pod running normally.

---

### 🧼 **Step 7: Verify the job is deleted automatically**

If you wait a few seconds and run:

```bash
kubectl get jobs
```

The job will be **gone**, because of this line:

```yaml
"helm.sh/hook-delete-policy": hook-succeeded
```

That means Helm cleaned it up ✅

---

### 🎉 **Summary**

| Step | Action                           | Result                       |
| ---- | -------------------------------- | ---------------------------- |
| 1    | Created a Job with `pre-install` | Job ran before install       |
| 2    | Added delete policy              | Job removed automatically    |
| 3    | Installed chart                  | Hook executed + app deployed |
| 4    | Verified cleanup                 | Job deleted after success    |

---

Would you like me to show you **another real example** — like a **post-install hook** that runs a success message *after* the deployment finishes?
Perfect 💪 Mounisha! You’re doing great — now that you’ve understood **pre-install hooks**, let’s go one step ahead and learn **post-install hooks** practically.

---

## 🌈 **Day 10 – Practical Example 2: Post-install Hook**

We’ll make a small Helm chart that:

1. Deploys a **simple NGINX app**.
2. After the app finishes deploying, runs a **post-install Job** that prints a success message like
   👉 “App successfully deployed!”

---

### 🪄 **Step 1: Use the same chart**

We’ll continue with your `hook-demo` chart from earlier.

If you closed it:

```bash
cd hook-demo
```

---

### 🧾 **Step 2: Create a post-install job**

Create a new file:

```bash
nano templates/post-install-job.yaml
```

Now paste this YAML 👇

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ .Release.Name }}-postinstall-job"
  annotations:
    "helm.sh/hook": post-install
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    spec:
      containers:
        - name: post-install
          image: busybox
          command: ['sh', '-c', 'echo "✅ Post-install job running after deployment!" && sleep 5']
      restartPolicy: Never
```

🧠 Explanation:

* `"helm.sh/hook": post-install` → Run this Job *after* the Helm installation finishes.
* `"helm.sh/hook-delete-policy": hook-succeeded` → Delete it after it finishes successfully.

---

### 🧱 **Step 3: Verify your main app still exists**

Your `templates/deployment.yaml` should still be like:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
      - name: {{ .Release.Name }}
        image: nginx
        ports:
        - containerPort: 80
```

---

### 🧪 **Step 4: Upgrade or re-install the chart**

If you already installed the chart before, upgrade it:

```bash
helm upgrade myapp .
```

If not, you can reinstall fresh:

```bash
helm install myapp .
```

---

### 📜 **Step 5: Observe Helm’s output**

You’ll see logs like:

```
HOOKS:
---
# post-install
NAME: myapp-postinstall-job
LAST STARTED: ...
STATE: succeeded
---
NAME: myapp
NAMESPACE: default
STATUS: deployed
```

✅ This means:

* Helm first deployed your NGINX app.
* Then it ran the post-install job after everything was ready.

---

### 🧾 **Step 6: Check logs of post-install job**

Run:

```bash
kubectl logs job/myapp-postinstall-job
```

You’ll see:

```
✅ Post-install job running after deployment!
```

---

### 🧹 **Step 7: Verify deletion**

Wait a few seconds and check:

```bash
kubectl get jobs
```

The post-install job should be **gone**, because Helm deleted it after success.

---

### 🧠 **Understanding Difference Between Pre-install & Post-install**

| Feature         | pre-install                              | post-install                              |
| --------------- | ---------------------------------------- | ----------------------------------------- |
| Runs when?      | Before installing resources              | After all resources are installed         |
| Common use      | DB migration, setup                      | Testing, notifications, cleanup           |
| Example job     | Initialize DB schema                     | Send success message                      |
| Execution order | 1️⃣ Pre-install job → 2️⃣ App deployment | 1️⃣ App deployment → 2️⃣ Post-install job |

---

### 🎉 **Practical Recap**

✅ **pre-install** → Job runs *before* app deploys
✅ **post-install** → Job runs *after* app deploys
✅ **hook-delete-policy** cleans them automatically
✅ You can view all hook events in Helm logs under “HOOKS” section

---

Next, in **Day 11**, we’ll cover:

> 🧩 “Helm Dependencies” — how charts depend on other charts (like using a PostgreSQL subchart with your app).

---

Would you like me to start **Day 11: Helm Dependencies** now, or do you want a quick **recap quiz** for Hooks first (to strengthen your understanding)?

Would you like me to now show a **real practical example (step-by-step)** of running a `pre-install` hook in your own chart — so you can see exactly how it works in action?
