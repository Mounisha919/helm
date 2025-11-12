Perfect 🙌 Mounisha!
Let’s jump straight into **✨ Day 13 — Helm Upgrade, Rollback & Version Management**

This is a **real DevOps-level** topic — it teaches how to safely update your Helm releases, track changes, and roll back instantly when something goes wrong.
Let’s go step-by-step 👇

---

## 🎯 **Day 13 — Helm Upgrade, Rollback & Version Management**

---

### 🧩 1️⃣ What is a Helm Upgrade?

When you make changes to your Helm chart (like updating values or changing templates), you can apply them using:

```bash
helm upgrade <release-name> <chart-path>
```

🧠 Example:

```bash
helm upgrade myapp ./mychart
```

This tells Helm:

> “Compare the current release (myapp) with the new chart, find what changed, and update only the differences.”

---

### ⚙️ 2️⃣ Helm Upgrade with New Values

You can update your release by passing a modified `values.yaml` file:

```bash
helm upgrade myapp ./mychart -f new-values.yaml
```

Or by overriding a specific value directly:

```bash
helm upgrade myapp ./mychart --set replicaCount=5
```

💡 **Tip:**
Always use `--dry-run --debug` before applying — it simulates the upgrade safely.

```bash
helm upgrade myapp ./mychart --dry-run --debug
```

---

### 🧱 3️⃣ Viewing Helm Release History

Every time you install or upgrade a Helm release, Helm stores a **revision** in Kubernetes.

You can list all versions (revisions) using:

```bash
helm history myapp
```

📊 Example output:

```
REVISION    UPDATED                  STATUS      CHART          APP VERSION  DESCRIPTION
1            Tue Nov 12 14:03:42     deployed    mychart-0.1.0   1.16.0      Install complete
2            Wed Nov 13 10:17:12     deployed    mychart-0.2.0   1.17.0      Upgrade complete
3            Wed Nov 13 10:40:05     failed      mychart-0.2.0   1.17.0      Upgrade failed
```

You can now rollback to any **revision** easily!

---

### 🔄 4️⃣ Helm Rollback (Undo Upgrade)

If an upgrade goes wrong, rollback to a previous version:

```bash
helm rollback myapp 2
```

Here:

* `myapp` → release name
* `2` → revision number from `helm history`

Helm will:
✅ Revert all changes
✅ Redeploy the last stable configuration

---

### 🧾 5️⃣ Helm Rollback Example

Let’s say:

* You upgraded `replicaCount` from 2 → 5
* But your app crashed after upgrade

To fix:

```bash
helm history myapp   # Find stable revision number
helm rollback myapp 1
```

Output:

```
Rollback was a success! Happy Helming!
```

Then verify:

```bash
kubectl get pods
```

---

### 📦 6️⃣ Chart Version vs App Version

In `Chart.yaml`, you’ll find:

```yaml
version: 0.2.0
appVersion: "2.1.3"
```

| Field          | Purpose                                                                   |
| -------------- | ------------------------------------------------------------------------- |
| **version**    | Version of the Helm chart itself (used for Helm upgrade logic)            |
| **appVersion** | Version of the application being deployed (used for your image tag, etc.) |

💡 **Example:**
If you upgrade your app image from `1.0` → `2.0`, you might:

* Keep chart version: `0.1.1`
* Change appVersion: `"2.0"`

If you change chart structure or templates → increment `chart version`.

---

### 🔍 7️⃣ Checking Differences Before Upgrading

Before running `helm upgrade`, you can compare current vs new manifests.

If using Helm 3.12+:

```bash
helm diff upgrade myapp ./mychart
```

(You might need to install the `helm-diff` plugin.)

It shows exactly **what YAML lines will change**, so you can avoid risky deployments.

---

### 🚀 8️⃣ Best Practices for Upgrades & Rollbacks

✅ Always test with `--dry-run` before actual upgrade
✅ Tag your Docker images properly (avoid “latest”)
✅ Keep `values.yaml` in Git (version control)
✅ Use `helm history` before rollback
✅ Test the rollback too (don’t assume it always works)

---

### 🧠 9️⃣ Example Scenario

#### Step 1 — Install v1

```bash
helm install myapp ./mychart --set image.tag=1.0
```

#### Step 2 — Upgrade to v2

```bash
helm upgrade myapp ./mychart --set image.tag=2.0
```

#### Step 3 — Something fails

Pods crash or errors occur…

#### Step 4 — Rollback

```bash
helm rollback myapp 1
```

✅ App restored to v1 instantly.

---

### 📋 Summary

| Command                              | Purpose                          |
| ------------------------------------ | -------------------------------- |
| `helm upgrade <release> <chart>`     | Upgrade an existing release      |
| `helm upgrade --dry-run --debug`     | Simulate upgrade                 |
| `helm history <release>`             | Show revision history            |
| `helm rollback <release> <revision>` | Rollback to previous revision    |
| `helm diff upgrade`                  | Compare manifests before upgrade |
| `Chart.yaml → version`               | Helm chart version               |
| `Chart.yaml → appVersion`            | Application version              |

---

Would you like me to now continue with **Day 14 — Helm Chart Security & Best Practices (Image scanning, secrets, RBAC)** next?
It’s a super practical session used in real DevOps pipelines.
