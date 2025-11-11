Awesome, Mounisha! 🎉
Welcome to **Day 7** of your Helm journey — you’re learning like a pro! 🧠🔥

Today’s topic is **Helm Value Overrides and Precedence** — one of the most *important and practical* concepts in Helm.

Let’s go slowly and deeply, step by step 💡

---

# 🎓 **Day 7 — Helm Value Overrides (Precedence, Environments, and Real Examples)**

When you deploy apps in **different environments** (like dev, QA, prod), you don’t want to change the same chart manually every time.
Instead, you use **value overrides** to change behavior dynamically.

---

## 🧩 Step 1: Recap — What Is `values.yaml`?

You already know that `values.yaml` stores default configuration for a chart:

```yaml
replicaCount: 2
image:
  repository: nginx
  tag: latest
service:
  type: ClusterIP
  port: 80
```

When you run:

```bash
helm install myapp ./mychart
```

→ Helm uses these values by default.

---

## 🧩 Step 2: But What If You Want Different Values for Different Environments?

Example:

* In **dev**, you want 1 replica
* In **prod**, you want 5 replicas
* In **qa**, you want different ports

Instead of changing `values.yaml` each time,
you can **override values at install or upgrade time**.

---

# 🧠 3 Ways to Override Values in Helm

There are **three major ways** Helm decides *which value wins*:

| Priority      | Method                                | Description                         |
| ------------- | ------------------------------------- | ----------------------------------- |
| 1️⃣ (Highest) | `--set` or `--set-string` flag        | Inline override from command line   |
| 2️⃣           | `-f custom-values.yaml` or `--values` | Override using a separate YAML file |
| 3️⃣ (Lowest)  | `values.yaml`                         | Default values defined in the chart |

---

## ⚙️ Step 3: Example Setup

Let’s take a chart `mychart` with this `values.yaml`:

```yaml
replicaCount: 2
image:
  repository: nginx
  tag: latest
service:
  port: 80
```

And inside `templates/deployment.yaml`:

```yaml
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
      - name: myapp
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

---

## 📄 Option 1: Default — Using `values.yaml`

```bash
helm install myapp ./mychart
```

Output → Helm uses:

```
replicaCount = 2
image = nginx:latest
```

---

## 📄 Option 2: Using a Custom `values` File

You can create separate files for each environment:

### `values-dev.yaml`

```yaml
replicaCount: 1
image:
  tag: dev
```

### `values-prod.yaml`

```yaml
replicaCount: 5
image:
  tag: stable
```

Then install:

```bash
helm install myapp ./mychart -f values-dev.yaml
```

✅ Output:

```
replicaCount = 1
image = nginx:dev
```

Or for prod:

```bash
helm install myapp ./mychart -f values-prod.yaml
```

✅ Output:

```
replicaCount = 5
image = nginx:stable
```

💡 **Tip:** You can have as many custom values files as you want — one for each environment.

---

## 📄 Option 3: Using CLI Overrides (`--set`)

You can directly override values on the command line:

```bash
helm install myapp ./mychart --set replicaCount=10
```

✅ Output:

```
replicaCount = 10
```

You can also override nested values:

```bash
helm install myapp ./mychart --set image.repository=httpd --set image.tag=1.2
```

✅ Output:

```
image = httpd:1.2
```

---

## 📄 Option 4: Combine Multiple Overrides

You can combine both:

```bash
helm install myapp ./mychart -f values-prod.yaml --set replicaCount=20
```

✅ Here’s what happens:

| Source             | Value | Used?          |
| ------------------ | ----- | -------------- |
| `values.yaml`      | 2     | ❌ (overridden) |
| `values-prod.yaml` | 5     | ❌ (overridden) |
| `--set`            | 20    | ✅ (wins)       |

🎯 **Highest priority wins!**

---

## 🧠 Step 4: Value Precedence Summary Table

| Order | Source                              | Example                        | Priority   |
| ----- | ----------------------------------- | ------------------------------ | ---------- |
| 1️⃣   | CLI `--set`                         | `--set image.tag=prod`         | 🏆 Highest |
| 2️⃣   | Custom file (`-f values-prod.yaml`) | environment-specific overrides | Medium     |
| 3️⃣   | Default (`values.yaml`)             | Base configuration             | Lowest     |

---

## 🧩 Step 5: Deep Concept — Merge Behavior

When Helm merges multiple files, it doesn’t **replace everything**, it **merges keys**.

Example:

`values.yaml`

```yaml
image:
  repository: nginx
  tag: latest
service:
  port: 80
```

`values-prod.yaml`

```yaml
image:
  tag: stable
```

Final merged values:

```yaml
image:
  repository: nginx
  tag: stable
service:
  port: 80
```

✅ Helm keeps unspecified values from base file.

---

## ⚙️ Step 6: Real-World Example — Dev, QA, and Prod

```
mychart/
├── values.yaml           # default
├── values-dev.yaml       # dev overrides
├── values-qa.yaml        # qa overrides
└── values-prod.yaml      # prod overrides
```

Then you deploy easily:

```bash
helm install myapp ./mychart -f values-dev.yaml
helm install myapp ./mychart -f values-qa.yaml
helm install myapp ./mychart -f values-prod.yaml
```

✅ One chart — multiple environments.

---

## 🧠 Step 7: Bonus — Multiple Value Files at Once

You can pass more than one `-f` file:

```bash
helm install myapp ./mychart -f values.yaml -f values-prod.yaml
```

Helm merges from left to right:

* First load `values.yaml`
* Then apply overrides from `values-prod.yaml`

---

## ⚡ Step 8: Debugging — See Final Rendered YAML

Always check the final rendered manifest before deploying:

```bash
helm template myapp ./mychart -f values-prod.yaml --set replicaCount=10
```

You’ll see exactly which values were applied.

---

## 🧭 Summary — Day 7 Recap

| Concept              | Explanation                                             |
| -------------------- | ------------------------------------------------------- |
| `values.yaml`        | Default configuration for your chart                    |
| `-f file.yaml`       | Overrides defaults for specific environment             |
| `--set`              | Quick inline override for single values                 |
| **Precedence order** | `--set` > `-f` file > `values.yaml`                     |
| **Merge behavior**   | Helm merges all values, higher ones override lower ones |
| **Best practice**    | Keep one chart, create multiple `values-*.yaml` files   |

---

## 🧩 Quick Example Practice

1️⃣ Create chart

```bash
helm create day7-demo
```

2️⃣ Edit `values.yaml`:

```yaml
replicaCount: 2
image:
  repository: nginx
  tag: latest
```

3️⃣ Create `values-prod.yaml`:

```yaml
replicaCount: 5
image:
  tag: stable
```

4️⃣ Run:

```bash
helm template prod ./day7-demo -f values-prod.yaml --set image.repository=httpd
```

✅ Final output:

```
replicaCount: 5
image: httpd:stable
```

---

Would you like me to continue with **Day 8 → Templates + Conditions + Loops** next?
That day teaches you how to use **`if`, `range`, and logic inside Helm templates**, which helps make charts smarter and more dynamic (like enabling/disabling resources).
