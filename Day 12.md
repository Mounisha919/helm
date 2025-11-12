Great! Let's move on to **Day 12 — Helm Chart Testing**. Testing is an essential part of ensuring your Helm charts are reliable, reproducible, and well-structured. Today, we’ll learn how to **test Helm charts** and make sure everything works correctly before deploying.

---

# 🎓 **Day 12 — Helm Chart Testing**

Testing Helm charts involves checking that:

* The **structure** of the chart is correct.
* The **templates** generate valid Kubernetes resources.
* The **values** used in the templates are valid.
* The chart follows **best practices** for Helm.

We'll go through a few techniques and tools for testing Helm charts, including using `helm lint`, `helm test`, and unit tests.

---

## 🧩 Step 1: Using `helm lint` to Check Chart Structure

The **`helm lint`** command is used to **lint** a chart to check for errors in the structure and formatting.

### Run `helm lint`:

```bash
helm lint ./mychart
```

This will analyze the chart and provide feedback on:

* Invalid files.
* Misused keys.
* Deprecated features.

Example output:

```
==> Linting ./mychart
[INFO] Chart.yaml: icon is recommended
[WARNING] templates/deployment.yaml: duplicate label "app" found
[INFO] templates/_helpers.tpl: use of unused template "mychart.fullname"
```

* `helm lint` doesn’t validate whether the chart works correctly in Kubernetes; it’s mainly focused on structure and syntax.

---

## 🧩 Step 2: Running Helm Tests with `helm test`

Helm allows you to run tests on your **installed charts** by defining a `test` hook.

### 1️⃣ **Define a Test Hook**

In your chart's `templates/`, you can define Kubernetes resources that should be used for testing. This is done using **test hooks** in your chart.

Example of a test hook:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
  annotations:
    helm.sh/hook: test-success
spec:
  containers:
    - name: test-container
      image: busybox
      command: ["/bin/sh", "-c", "echo Test completed"]
```

This will create a `Pod` to run a test and print `Test completed`.

* **`helm.sh/hook: test-success`** tells Helm to run this resource only after the chart is successfully installed.

### 2️⃣ **Run the Helm Test**

Once the chart is deployed, you can run the test:

```bash
helm test <release-name>
```

This will execute the test resources you defined and show the output of your test.

### Example:

```bash
helm install myapp ./mychart
helm test myapp
```

This will run the `test-pod` defined in your templates, and Helm will provide feedback about whether the test passed or failed.

---

## 🧩 Step 3: Unit Testing Helm Charts with `helm-unittest`

For **automated testing**, you can use a tool like **helm-unittest**. It allows you to write unit tests for your Helm templates.

### 1️⃣ **Install helm-unittest**

First, you need to install `helm-unittest`:

```bash
helm plugin install https://github.com/quintush/helm-unittest
```

### 2️⃣ **Write Unit Tests for Templates**

Create a `tests/` directory inside your chart and create test files in it.

Example: `tests/test_deployment.yaml`

```yaml
suite: "myapp"
test:
  - it: should create a deployment
    set:
      replicaCount: 3
    template:
      metadata:
        name: {{ .Release.Name }}-deployment
      spec:
        replicas: {{ .Values.replicaCount }}
    assert:
      - equal:
          path: spec.replicas
          value: 3
```

This test checks that the `replicaCount` is correctly set to 3 in the deployment.

### 3️⃣ **Run the Unit Test**

Now, you can run the unit tests using:

```bash
helm unittest ./mychart
```

This will execute the tests and give you feedback on whether the chart behaves as expected.

Example output:

```
PASS  (1.23s)
```

---

## 🧩 Step 4: Integration Testing with `helm test`

While **unit testing** checks individual templates and logic, **integration testing** ensures that the chart works correctly when installed into a Kubernetes cluster. Helm itself provides the `helm test` command for this, but you can also manually run integration tests like checking whether the services, deployments, and ingress are up and running after installation.

---

## 🧩 Step 5: Helm Testing Best Practices

### 1️⃣ **Use `helm lint` Regularly**

Always run `helm lint` before installing or deploying a chart. This will help you catch structural issues early.

### 2️⃣ **Define Test Hooks for Critical Resources**

For charts that deploy services or jobs, always define **test hooks** to ensure your app works after installation.

### 3️⃣ **Use Unit Tests for Template Logic**

With `helm-unittest`, you can verify if specific values are correctly rendered and if the logic works as expected (for example, ensuring that your `replicaCount` is applied properly).

### 4️⃣ **Integration Testing in Kubernetes**

Once your chart passes unit tests, deploy it in a test Kubernetes cluster and run tests to ensure all components work together (e.g., services can communicate, ingress is accessible, etc.).

---

## 🧠 Summary of Helm Chart Testing

| Test Type             | Command                    | Description                                                                  |
| --------------------- | -------------------------- | ---------------------------------------------------------------------------- |
| **Lint**              | `helm lint ./mychart`      | Check the structure and syntax of the chart                                  |
| **Run Tests**         | `helm test <release-name>` | Run test hooks defined in the chart (Post-installation tests)                |
| **Unit Tests**        | `helm unittest ./mychart`  | Write and execute unit tests for Helm templates                              |
| **Integration Tests** | Manual                     | Test the actual deployment in Kubernetes to ensure everything works together |

---

## 🎯 Practice Task

1️⃣ Create a chart:

```bash
helm create myapp
```

2️⃣ Add a test hook to your `templates/` (like a Pod that prints a message).

3️⃣ Run `helm lint` to check for structure errors:

```bash
helm lint ./myapp
```

4️⃣ Install the chart:

```bash
helm install myapp ./myapp
```

5️⃣ Run the test:

```bash
helm test myapp
```

---

Would you like to dive deeper into **advanced chart testing** or explore a **real-world example** of testing a complex chart with multiple dependencies? Let me know!

Perfect 💪 Mounisha! Let’s start **✨ Day 12 — Helm Testing & Debugging (Advanced Helm for Professionals)**

This is an **important DevOps-level** topic — it helps you **verify**, **debug**, and **validate** your Helm deployments confidently before pushing to production.

---

## 🧩 **Day 12 — Helm Testing & Debugging**

### 🎯 Learning Goals

By the end of today’s class, you’ll understand:

1. What Helm tests are
2. How to create Helm test pods
3. How to debug templates before installing
4. How to view rendered manifests and detect YAML issues
5. How to simulate an install without applying it

---

## 🧠 **1. What is Helm Test?**

Helm allows you to define **tests** that run after a chart is installed.
These tests help confirm that your app is working as expected (for example, checking if the web service is reachable).

🧩 Example:
You can create a Pod that runs a command like:

```bash
curl myapp:8080/health
```

If it succeeds (exit code 0), Helm marks the test as **PASSED** ✅.
If it fails (exit code ≠ 0), Helm marks it as **FAILED** ❌.

---

## 🧾 **2. Defining a Helm Test**

You define a test Pod inside your chart — usually in:

```
templates/tests/test-connection.yaml
```

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "mychart.fullname" . }}-test-connection"
  labels:
    app.kubernetes.io/name: {{ include "mychart.name" . }}
    helm.sh/chart: {{ include "mychart.chart" . }}
    app.kubernetes.io/instance: {{ .Release.Name }}
    app.kubernetes.io/managed-by: {{ .Release.Service }}
  annotations:
    "helm.sh/hook": test
spec:
  containers:
    - name: curl
      image: curlimages/curl
      command: ['curl']
      args: ['{{ include "mychart.fullname" . }}:{{ .Values.service.port }}']
  restartPolicy: Never
```

👉 Here:

* The annotation `"helm.sh/hook": test` tells Helm that this is a **test hook**.
* The Pod runs a simple **curl** command to test connectivity.

---

## 🧪 **3. Running Helm Tests**

After you install your release:

```bash
helm install myapp ./mychart
```

You can run your tests using:

```bash
helm test myapp
```

📊 Output:

```
RUNNING: myapp-test-connection
PASSED: myapp-test-connection
```

If your pod fails or times out:

```
FAILED: myapp-test-connection
Error: 1 test(s) failed
```

---

## 🔍 **4. Debugging Helm Templates**

Sometimes, you want to see what Helm will deploy **before actually deploying** it.

### ✅ Show rendered templates

```bash
helm template myapp ./mychart
```

This command **renders** all templates and prints them as YAML to your terminal — without sending anything to Kubernetes.

---

### ✅ Simulate installation

```bash
helm install myapp ./mychart --dry-run --debug
```

It:

* Doesn’t install the chart
* Prints rendered templates
* Shows debug output and variable values

Very useful to check **syntax errors**, **missing values**, or **YAML indentation issues**.

---

### ✅ Verify values being passed

```bash
helm get values myapp
```

This shows which values were used in a deployed release.

---

## 🧰 **5. Common Debugging Commands**

| Command                          | Description                              |
| -------------------------------- | ---------------------------------------- |
| `helm lint ./mychart`            | Checks chart for YAML or template errors |
| `helm template ./mychart`        | Renders templates locally                |
| `helm install --dry-run --debug` | Simulates install with debug info        |
| `helm get manifest myapp`        | Shows manifests of installed release     |
| `helm test myapp`                | Runs defined test pods                   |

---

## 🧠 **6. Real Example**

Let’s say your app deploys successfully, but you’re not sure it’s reachable.

You can create a test Pod that:

* Waits for your service
* Executes `curl myapp:8080`
* Returns 0 if successful

Then simply run:

```bash
helm test myapp
```

If the pod succeeds → ✅ Test passed
If the pod fails → ❌ You debug with:

```bash
kubectl logs myapp-test-connection
```

---

## 🎓 **Summary**

| Feature                | Command / Concept   | Purpose                              |
| ---------------------- | ------------------- | ------------------------------------ |
| **Helm Test**          | `helm test release` | Runs test hooks (like health checks) |
| **Dry Run**            | `--dry-run`         | Simulate install without applying    |
| **Debug Mode**         | `--debug`           | Shows detailed logs and rendering    |
| **Template Rendering** | `helm template`     | Print rendered YAML                  |
| **Linting**            | `helm lint`         | Validate chart syntax                |
| **Manifest Retrieval** | `helm get manifest` | View applied manifests               |

---



