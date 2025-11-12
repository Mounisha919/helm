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
