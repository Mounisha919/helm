Great! Let’s dive into **Day 10 — Helm Chart Dependencies and Subcharts**.
This is an exciting topic because, when you’re dealing with multiple services or microservices, you often want to manage dependencies between different charts (for example, one chart may need another one to run, like a database or a caching service).

---

# 🎓 **Day 10 — Helm Chart Dependencies and Subcharts**

Helm makes it **super easy to manage dependencies** between charts. Instead of duplicating logic, you can create reusable **subcharts** that another chart can depend on. You’ll also learn about **`requirements.yaml`** (Helm 2) and **`Chart.yaml`** (Helm 3) dependencies.

Let’s go step-by-step! 👇

---

## 🧩 Step 1: What Are Helm Chart Dependencies?

### A dependency is simply **one Helm chart that another chart depends on**.

For example, your app might need a Redis or MySQL chart to be installed alongside it, and you don’t want to manually install them every time. Instead, you can define a dependency inside your app’s chart.

### **Example Use Cases:**

* A web app chart might depend on a database chart (like MySQL or PostgreSQL).
* A backend service chart might need a cache service (like Redis or Memcached).
* A chart might need monitoring tools like Prometheus or Grafana.

---

## 🧩 Step 2: Adding Dependencies to Your Chart

### In **Helm 3**, dependencies are defined in the `Chart.yaml` file.

Here’s how you can add a Redis dependency to your `Chart.yaml`.

### 🧾 Example: `Chart.yaml`

```yaml
apiVersion: v2
name: myapp
description: A simple app chart
version: 1.0.0
dependencies:
  - name: redis
    version: 14.6.0
    repository: "https://charts.bitnami.com/bitnami"
```

* **name**: The name of the dependency (e.g., Redis).
* **version**: The specific version of the dependency.
* **repository**: The Helm repository where the chart is located (e.g., Bitnami charts).

---

## 🧩 Step 3: Helm Dependency Management

After you add the dependency in `Chart.yaml`, you need to **download** and **manage** it.
Helm provides a command to handle this:

### Download Dependencies:

```bash
helm dependency update
```

This will download the Redis chart (and its dependencies) into the `charts/` directory in your app’s chart.

### 🧩 Directory Structure After Download:

```
myapp/
├── Chart.yaml
├── charts/
│   ├── redis-14.6.0.tgz
├── templates/
└── values.yaml
```

---

## 🧩 Step 4: How to Use Dependencies in Your Templates

Once a dependency (e.g., Redis) is added, it’s available to use within your own chart’s templates. You can **reference the dependency’s values** in your own chart’s resources.

For example, if Redis has a `password` value, you can use it like this:

### 🧾 Example: `templates/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 1
  template:
    spec:
      containers:
        - name: myapp
          image: nginx
          env:
            - name: REDIS_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: {{ .Release.Name }}-redis
                  key: password
```

This example uses the Redis password from the Redis subchart's `Secret` object.

---

## 🧩 Step 5: Subchart Values

Each dependency comes with its own `values.yaml`. If you want to **override the values** of the Redis chart, you can do that in your app’s `values.yaml`.

### 🧾 Example: `values.yaml`

```yaml
redis:
  password: my-redis-password
  global:
    redis:
      password: "my-global-password"
```

In this example, you're **overriding the Redis password** used in the subchart.

---

## 🧩 Step 6: Using a Local Chart as a Dependency

If you want to use a local chart as a dependency (maybe because you have an internal chart you don’t want to share externally), you can add it as a **local dependency**.

### 🧾 Example: `Chart.yaml`

```yaml
apiVersion: v2
name: myapp
dependencies:
  - name: redis
    version: 14.6.0
    repository: "file://../redis"  # local directory path
```

Then run:

```bash
helm dependency update
```

This will use the local chart from the specified directory.

---

## 🧩 Step 7: Managing Dependency Versions

Helm also supports **version constraints** for dependencies, so you can specify a range of acceptable versions.

For example, you might want to use Redis version `^14.0.0` (any version that is `14.x.x`):

### 🧾 Example: `Chart.yaml`

```yaml
dependencies:
  - name: redis
    version: "^14.0.0"
    repository: "https://charts.bitnami.com/bitnami"
```

This means Helm will install any Redis chart with a version starting from `14.0.0` but below `15.0.0`.

---

## 🧩 Step 8: Subcharts and Global Values

Subcharts can use **global values** from the parent chart. If you define a global value in the parent chart, it’s available to all subcharts.

For example, you might define a **global database password** in your parent chart:

### 🧾 Example: `values.yaml` (parent chart)

```yaml
global:
  redis:
    password: global-redis-password
```

The Redis subchart can access this value by using the `global.redis.password` key.

---

## 🧩 Step 9: Helm Dependencies Summary

| Concept                   | Description                                                      |
| ------------------------- | ---------------------------------------------------------------- |
| **Dependencies**          | One chart that depends on another                                |
| **`Chart.yaml`**          | File where dependencies are defined in Helm 3                    |
| **Download Dependencies** | `helm dependency update` to download dependencies                |
| **Local Dependencies**    | Use a local chart with `repository: "file://path/to/chart"`      |
| **Global Values**         | Define global values in parent chart to be accessed by subcharts |
| **Versioning**            | Specify version constraints for dependencies (e.g., `^14.0.0`)   |

---

## 🎯 Practice Exercise

1️⃣ Create a new chart:

```bash
helm create myapp
```

2️⃣ In `Chart.yaml`, add a dependency to Redis:

```yaml
dependencies:
  - name: redis
    version: "^14.0.0"
    repository: "https://charts.bitnami.com/bitnami"
```

3️⃣ Run:

```bash
helm dependency update
```

4️⃣ Now, modify your `values.yaml` to customize Redis:

```yaml
redis:
  password: "supersecretpassword"
```

5️⃣ Deploy your app to see the final rendered Kubernetes YAML and check that Redis is included.

---

### 🧠 Recap:

* **Dependencies** in Helm allow you to manage linked charts (like Redis or MySQL).
* The **Chart.yaml** file defines your chart’s dependencies in Helm 3.
* Use **subchart values** and **global values** to control and customize dependencies.
* Helm makes it easy to manage **local charts** and **version constraints** for dependencies.

---

Would you like to continue with **Day 11 — Helm Chart Repositories** (covering how to manage and push your custom charts to repositories)? This is essential for **sharing your charts** with others or using charts in production.
