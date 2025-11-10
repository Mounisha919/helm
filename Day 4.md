Perfect 😎 Welcome to **Day 4 of your Helm journey** — you’re doing awesome so far!
We’ve covered:
✅ Day 1 – What is Helm & Why it’s used
✅ Day 2 – Helm Chart structure
✅ Day 3 – Helm templating

Now let’s move to **Day 4: Helm Values and Overrides (real-world multi-environment setup)**

---

## 🧭 **Day 4 – Helm Values and Overrides (dev, qa, prod)**

This is one of the most **practical** and **real-world** concepts — every company uses this for managing different environments like **dev, QA, stage, and production**.

---

### 🧩 1️⃣ Quick Recap – What is `values.yaml`?

`values.yaml` contains **default configurations** used by templates.
Example:

```yaml
replicaCount: 2

image:
  repository: nginx
  tag: latest

service:
  type: ClusterIP
  port: 80
```

If you just run:

```bash
helm install myapp ./mychart
```

Helm uses these default values.

But what if your **production** app needs:

* 5 replicas
* A different image tag
* A LoadBalancer service type

You don’t want to change the main file every time, right?

That’s where **override files** come in.

---

### ⚙️ 2️⃣ Multiple Values Files (dev, qa, prod)

You can create **separate values files** for each environment:

```
mychart/
├── values.yaml
├── values-dev.yaml
├── values-qa.yaml
└── values-prod.yaml
```

---

#### Example: `values-dev.yaml`

```yaml
replicaCount: 1
image:
  tag: dev
service:
  type: ClusterIP
```

#### Example: `values-qa.yaml`

```yaml
replicaCount: 2
image:
  tag: qa
service:
  type: NodePort
```

#### Example: `values-prod.yaml`

```yaml
replicaCount: 5
image:
  tag: stable
service:
  type: LoadBalancer
```

---

### 🧪 3️⃣ Using Different Values Files

Now install your chart for each environment:

```bash
# For Dev
helm install myapp-dev ./mychart -f values-dev.yaml

# For QA
helm install myapp-qa ./mychart -f values-qa.yaml

# For Prod
helm install myapp-prod ./mychart -f values-prod.yaml
```

👉 `-f` = file override
Helm merges your environment file with the default `values.yaml`.
If the same key exists in both — the environment file wins.

---

### 🧠 4️⃣ How Helm Merges Values

Helm uses a simple rule:

> “Environment-specific values override default values.”

Example:

**values.yaml**

```yaml
replicaCount: 2
```

**values-prod.yaml**

```yaml
replicaCount: 5
```

When installing with:

```bash
helm install app ./mychart -f values-prod.yaml
```

✅ Final value used → `replicaCount = 5`

---

### 🧩 5️⃣ Another Override Option – `--set`

You can override single values directly from CLI (useful for quick changes).

Example:

```bash
helm install myapp ./mychart --set replicaCount=3
```

You can override nested values using dot notation:

```bash
helm install myapp ./mychart --set image.repository=nginx,image.tag=v2
```

✅ Pro tip: `--set` is for quick one-time changes.
For long-term environment settings, always use separate YAML files (`values-dev.yaml`, etc.).

---

### ⚙️ 6️⃣ Combining Multiple Overrides

You can also combine them:

```bash
helm install myapp ./mychart -f values.yaml -f values-prod.yaml --set replicaCount=10
```

Order matters!
👉 The last one overrides the previous ones.

So here:

1. Helm loads `values.yaml`
2. Merges with `values-prod.yaml`
3. Then applies `--set` overrides

✅ Final value = 10 replicas

---

### 📦 7️⃣ Check What Values Were Used

Helm allows you to check the final rendered configuration:

```bash
helm get values myapp
```

or check before installing:

```bash
helm install myapp ./mychart -f values-prod.yaml --dry-run --debug
```

---

### 🧠 8️⃣ Real Company Example

Let’s say you work at a company that deploys a web app to 3 environments:

| Environment | File             | Replicas | Image      | Service Type |
| ----------- | ---------------- | -------- | ---------- | ------------ |
| Dev         | values-dev.yaml  | 1        | app:dev    | ClusterIP    |
| QA          | values-qa.yaml   | 2        | app:qa     | NodePort     |
| Prod        | values-prod.yaml | 5        | app:stable | LoadBalancer |

Commands:

```bash
helm install web-dev ./webchart -f values-dev.yaml
helm install web-qa ./webchart -f values-qa.yaml
helm install web-prod ./webchart -f values-prod.yaml
```

💡 All use the **same Helm chart**, but with different configurations — that’s the real power of Helm.

---

### 🧹 9️⃣ Updating a Release

If you want to modify existing Helm releases:

```bash
helm upgrade myapp ./mychart -f values-prod.yaml
```

If you just change one setting:

```bash
helm upgrade myapp ./mychart --set image.tag=v3
```

To rollback if something breaks:

```bash
helm rollback myapp 1
```

(`1` = previous revision number)

---

### ⚠️ 10️⃣ Common Beginner Errors

| Error                            | Reason                         | Solution                           |
| -------------------------------- | ------------------------------ | ---------------------------------- |
| `Error: cannot read values file` | Wrong path or file name        | Check `-f` file path               |
| `unknown field`                  | Wrong indentation or key name  | Compare with default `values.yaml` |
| `merge conflict`                 | Duplicate nested keys          | Keep consistent structure          |
| `no deployed release`            | Using `upgrade` before install | Do `helm install` first            |

---

### 🧪 Hands-on Task for Today

1. Create 3 values files → dev, qa, prod
2. Change replicas, image tags, and service types in each
3. Install 3 separate releases:

   ```bash
   helm install myapp-dev ./mychart -f values-dev.yaml
   helm install myapp-qa ./mychart -f values-qa.yaml
   helm install myapp-prod ./mychart -f values-prod.yaml
   ```
4. Verify:

   ```bash
   helm list
   helm get values myapp-prod
   ```
5. Try upgrading one release and rolling back.

---

### 🏁 Summary

| Concept           | Description                    |
| ----------------- | ------------------------------ |
| `values.yaml`     | Default values                 |
| `values-dev.yaml` | Environment-specific overrides |
| `-f`              | Load custom value file         |
| `--set`           | Override inline from CLI       |
| `helm upgrade`    | Change existing deployment     |
| `helm rollback`   | Undo previous deployment       |

---

🎯 **Tomorrow (Day 5)** →
We’ll cover **Helm Chart Dependencies & Requirements.yaml**
That means — how to connect your chart to another one (like your app + Redis + MySQL).
You’ll learn how one chart can depend on another — just like microservices in production.

---

Would you like me to include a **small real example on Day 5** (like app + redis dependency chart)?
