Perfect, Mounisha 👍
Let’s go **slowly and in detail** — step by step — so you understand everything about `_helpers.tpl` and how it’s used in Helm.
This topic is **very important** because it’s the *backbone of clean Helm charts* used in real DevOps work.

---

# 🎓 **Day 6 — Deep Dive into `_helpers.tpl`**

## 🧠 Step 1: What is `_helpers.tpl`?

When you create a Helm chart:

```bash
helm create mychart
```

You get this structure:

```
mychart/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── _helpers.tpl
    ├── deployment.yaml
    ├── service.yaml
```

So, `_helpers.tpl` is a **template helper file**.
It is used to **define reusable small code blocks** that can be called in other YAML files (like functions).

### 💬 Analogy

If your YAMLs are like long essays, `_helpers.tpl` is your “shortcut notebook” where you write small, reusable phrases or formulas.

---

## 🧩 Step 2: Why We Need `_helpers.tpl`

Imagine this:

In `deployment.yaml`, `service.yaml`, and `ingress.yaml` —
you keep writing the **app name** or **labels** again and again:

```yaml
metadata:
  name: myapp
  labels:
    app: myapp
```

If your app name changes, you’d have to edit **every file**! 😣

Instead, we can define it **once** in `_helpers.tpl` and reuse it everywhere.
So `_helpers.tpl` helps us:

* avoid repetition
* make charts **cleaner** and **standardized**

---

## 🧱 Step 3: Example of `_helpers.tpl`

Let’s open `templates/_helpers.tpl` file.

```yaml
{{/*
This helper returns the chart name
*/}}
{{- define "mychart.name" -}}
{{ .Chart.Name }}
{{- end -}}
```

Here’s what’s happening:

| Part                            | Meaning                                 |
| ------------------------------- | --------------------------------------- |
| `{{- define "mychart.name" -}}` | Creates a helper named **mychart.name** |
| `.Chart.Name`                   | Gets the chart name from `Chart.yaml`   |
| `{{- end -}}`                   | Ends the helper definition              |

---

## 🧩 Step 4: How to Use It in Deployment.yaml

Now, open `templates/deployment.yaml`
and instead of writing the name manually like this:

```yaml
metadata:
  name: myapp
```

You can **call your helper** like this:

```yaml
metadata:
  name: {{ include "mychart.name" . }}
```

Here,

* `include` = call the helper function
* `"mychart.name"` = helper name (same as defined in `_helpers.tpl`)
* `.` = passes the current chart context (all values, chart data, etc.)

---

## 🧠 Step 5: Another Example — fullname Helper

You can make another helper in `_helpers.tpl` to generate a **full name** like:
`release-name` + `chart-name` (e.g. `coinapp-mychart`)

```yaml
{{/*
Generate a full release name
*/}}
{{- define "mychart.fullname" -}}
{{ .Release.Name }}-{{ .Chart.Name }}
{{- end -}}
```

Then in your deployment file:

```yaml
metadata:
  name: {{ include "mychart.fullname" . }}
```

If you install the chart with:

```bash
helm install coinapp ./mychart
```

Helm will render this as:

```yaml
metadata:
  name: coinapp-mychart
```

🎯 Automatically generated — no need to hardcode names!

---

## 🧩 Step 6: Using `_helpers.tpl` for Labels

You can also make a helper to generate **common labels** used in all resources.

### Add this in `_helpers.tpl`:

```yaml
{{/*
Generate common labels
*/}}
{{- define "mychart.labels" -}}
app.kubernetes.io/name: {{ include "mychart.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end -}}
```

Now in your `deployment.yaml`:

```yaml
metadata:
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
```

👉 The `| nindent 4` part means:

> "Indent this block by 4 spaces" (so YAML formatting doesn’t break)

It will render as:

```yaml
metadata:
  labels:
    app.kubernetes.io/name: mychart
    app.kubernetes.io/instance: coinapp
    app.kubernetes.io/managed-by: Helm
```

---

## 🧩 Step 7: How Helm Renders It

Run:

```bash
helm template mychart/
```

You’ll see the final YAMLs.
Helm replaces all `{{ include "..." . }}` helpers with the actual values.

---

## 🔍 Step-by-Step Real Example

Let’s do it practically with your app `coinapp`.

### 🧪 Step 1: Create a chart

```bash
helm create coinapp
```

### 🧪 Step 2: Open `_helpers.tpl` and add:

```yaml
{{/*
Chart simple name
*/}}
{{- define "coinapp.name" -}}
{{ .Chart.Name }}
{{- end -}}

{{/*
Full name with release
*/}}
{{- define "coinapp.fullname" -}}
{{ .Release.Name }}-{{ .Chart.Name }}
{{- end -}}

{{/*
Common labels
*/}}
{{- define "coinapp.labels" -}}
app.kubernetes.io/name: {{ include "coinapp.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
helm.sh/chart: {{ .Chart.Name }}
{{- end -}}
```

---

### 🧪 Step 3: Use these in `templates/deployment.yaml`

Replace your metadata like this:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "coinapp.fullname" . }}
  labels:
    {{- include "coinapp.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app.kubernetes.io/name: {{ include "coinapp.name" . }}
  template:
    metadata:
      labels:
        {{- include "coinapp.labels" . | nindent 8 }}
    spec:
      containers:
        - name: {{ include "coinapp.name" . }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: {{ .Values.service.port }}
```

---

### 🧪 Step 4: Render it

Run:

```bash
helm template coinapp/
```

You will see a **clean, dynamic YAML** with all names and labels automatically filled in.

---

## 🧩 Step 8: Why This Is So Important

In real DevOps projects:

* Hundreds of YAML files exist.
* Names, labels, and release identifiers must be consistent.
* `_helpers.tpl` avoids mistakes, repetition, and hardcoding.

Every professional Helm chart (like Bitnami, Grafana, Prometheus) uses `_helpers.tpl`.

---

## 🧩 Summary

| Concept         | Meaning                                                    |                           |
| --------------- | ---------------------------------------------------------- | ------------------------- |
| `_helpers.tpl`  | Stores reusable helper functions                           |                           |
| `define`        | Creates a helper                                           |                           |
| `include`       | Calls a helper                                             |                           |
| `.Chart.Name`   | Name from Chart.yaml                                       |                           |
| `.Release.Name` | Release name (given during install)                        |                           |
| `               | nindent N`                                                 | Indents lines by N spaces |
| Benefit         | Clean, DRY (Don’t Repeat Yourself), consistent Helm charts |                           |

---

## 💪 Practice Task for You

1️⃣ Create a chart `helm create coinapp`
2️⃣ Edit `_helpers.tpl` to include:

* `coinapp.name`
* `coinapp.fullname`
* `coinapp.labels`
  3️⃣ Use these in both `deployment.yaml` and `service.yaml`.
  4️⃣ Run:

```bash
helm template coinapp/
```

and see if names & labels render correctly.

---

Perfect, Mounisha 🌻
No worries at all — you’re doing **exactly the right thing** by asking again. Helm has many moving parts, and truly understanding the **difference between `values.yaml` and `_helpers.tpl`** is key to becoming confident.

Let’s go **step by step slowly and visually**, from **zero**.
We’ll even imagine how Helm reads and combines everything.

---

# 🎓 **Day 6 – In-Depth: Difference Between `values.yaml` and `_helpers.tpl`**

---

## 🧩 Step 1: What Are These Files?

When you create a Helm chart using:

```bash
helm create mychart
```

You get a structure like this:

```
mychart/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── _helpers.tpl
    ├── deployment.yaml
    ├── service.yaml
```

Let’s understand **each**:

| File              | Purpose                                                                   |
| ----------------- | ------------------------------------------------------------------------- |
| `values.yaml`     | Stores **data** — simple values (replica count, port, image name, etc.)   |
| `_helpers.tpl`    | Stores **logic** — reusable template functions (for naming, labels, etc.) |
| `deployment.yaml` | Uses both of them to build the final YAML sent to Kubernetes              |

---

## 🧠 Step 2: What Does `values.yaml` Do?

`values.yaml` is like a **settings/configuration file**.
It contains **key-value pairs** — no logic, just plain data.

Example 👇

```yaml
replicaCount: 3

image:
  repository: nginx
  tag: latest

service:
  type: ClusterIP
  port: 80

app:
  name: coinapp
```

These values tell Helm things like:

* How many replicas? → 3
* Which image? → nginx:latest
* What service port? → 80
* What is app name? → coinapp

🧠 Important: `values.yaml` does **not** have logic, loops, or conditions.
It’s just *data that can be replaced anywhere in templates.*

---

## 🧩 Step 3: How to Use `values.yaml` in Templates

Now, go to `deployment.yaml`.

Here’s how you can use those values:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.app.name }}
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
        - name: {{ .Values.app.name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: {{ .Values.service.port }}
```

When you run:

```bash
helm install myrelease ./mychart
```

Helm will take values from `values.yaml` and replace them:

```yaml
metadata:
  name: coinapp
spec:
  replicas: 3
  containers:
    - name: coinapp
      image: "nginx:latest"
      ports:
        - containerPort: 80
```

✅ So — `values.yaml` is used for **user-provided input**
and helps make charts **configurable** without touching the YAML logic.

---

## 🧩 Step 4: What is `_helpers.tpl`?

`_helpers.tpl` is different.
It’s not for values — it’s for **reusable template logic**.

It’s like a **function library** where you define small reusable pieces of code to avoid repetition.

Example `_helpers.tpl` 👇

```yaml
{{/*
Return the chart name
*/}}
{{- define "mychart.name" -}}
{{ .Chart.Name }}
{{- end -}}

{{/*
Return a full release name like release-name + chart-name
*/}}
{{- define "mychart.fullname" -}}
{{ .Release.Name }}-{{ .Chart.Name }}
{{- end -}}
```

Here:

* `define` → creates a helper function
* `"mychart.name"` → function name
* `.Chart.Name` → accesses the chart name from Chart.yaml
* `.Release.Name` → name given when you install the release

---

## 🧩 Step 5: Using `_helpers.tpl` in Other Templates

In your `deployment.yaml`, instead of writing hardcoded names,
you can **call** these helpers like this 👇

```yaml
metadata:
  name: {{ include "mychart.fullname" . }}
```

When you install:

```bash
helm install coinapp ./mychart
```

Helm reads `_helpers.tpl` and replaces that line with:

```yaml
metadata:
  name: coinapp-mychart
```

That’s because:

* `.Release.Name` = coinapp
* `.Chart.Name` = mychart
  ✅ So it builds: coinapp-mychart automatically.

---

## 🧩 Step 6: Combining Both Together

Let’s see both `values.yaml` and `_helpers.tpl` working in one chart.

### 🧾 `values.yaml`

```yaml
replicaCount: 2
image:
  repository: nginx
  tag: stable
service:
  port: 8080
app:
  name: coinapp
```

### 🧾 `_helpers.tpl`

```yaml
{{/*
Function to build full name
*/}}
{{- define "coinapp.fullname" -}}
{{ .Release.Name }}-{{ .Values.app.name }}
{{- end -}}
```

### 🧾 `deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "coinapp.fullname" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
        - name: {{ .Values.app.name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: {{ .Values.service.port }}
```

When you install:

```bash
helm install demo ./mychart
```

It becomes:

```yaml
metadata:
  name: demo-coinapp
spec:
  replicas: 2
  containers:
    - name: coinapp
      image: nginx:stable
      ports:
        - containerPort: 8080
```

💡 Helm has now:

* Taken **data** from `values.yaml`
* Applied **logic** from `_helpers.tpl`

Together, they built a perfect Kubernetes YAML!

---

## 🧠 Step 7: Simple Analogy — Everyday Example

Let’s imagine you are filling out an online **pizza order form 🍕**.

| Role             | Description                                                                                                                                         |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| **values.yaml**  | Like the form fields you fill — “size = medium”, “toppings = cheese”, “quantity = 2”. These are **static values**.                                  |
| **_helpers.tpl** | Like the website’s code that automatically creates your **order number**, **invoice name**, and **address format** — it’s **logic/code**, not data. |

So you (the user) give **values**, and the system uses **logic** to combine them.

---

## 🧩 Step 8: Key Differences Summarized

| Feature        | `values.yaml`                        | `_helpers.tpl`                                  |
| -------------- | ------------------------------------ | ----------------------------------------------- |
| Type           | Static file (key-value pairs)        | Dynamic template file                           |
| Purpose        | Holds input values for customization | Holds reusable logic / functions                |
| Syntax         | YAML                                 | Go templating (`{{ define }}`, `{{ include }}`) |
| Used in        | Templates (via `.Values.key`)        | Templates (via `include "helper.name"`)         |
| Example Use    | `.Values.image.repository`           | `include "mychart.fullname"`                    |
| Who edits it   | User / DevOps engineer               | Chart developer                                 |
| Example Output | Replica count, image, ports          | Names, labels, computed text                    |

---

## 🧩 Step 9: Why Helm Separates Them

👉 Because in real-world projects:

* DevOps users want to easily change values (like image tags or ports) **without touching code**
* Developers want a way to **reuse and standardize logic** (like names or labels)

So Helm gives both:

* `values.yaml` → for configuration
* `_helpers.tpl` → for reusable templating logic

---

## 🧪 Step 10: Hands-On Practice for You

### 🧱 Step 1: Create a new chart

```bash
helm create demoapp
```

### 🧱 Step 2: Edit `values.yaml`

```yaml
app:
  name: demoapp
image:
  repository: nginx
  tag: latest
replicaCount: 2
service:
  port: 80
```

### 🧱 Step 3: Edit `_helpers.tpl`

```yaml
{{- define "demoapp.fullname" -}}
{{ .Release.Name }}-{{ .Values.app.name }}
{{- end -}}
```

### 🧱 Step 4: Use both in `deployment.yaml`

```yaml
metadata:
  name: {{ include "demoapp.fullname" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
        - name: {{ .Values.app.name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: {{ .Values.service.port }}
```

### 🧱 Step 5: Render the output

```bash
helm template demo ./demoapp
```

Output:

```yaml
metadata:
  name: demo-demoapp
spec:
  replicas: 2
  containers:
    - name: demoapp
      image: nginx:latest
      ports:
        - containerPort: 80
```

✅ Final result = `.Values` provided static values
✅ `_helpers.tpl` added computed naming logic

---

## 🎯 Final Comparison Summary

| Feature        | `values.yaml`               | `_helpers.tpl`                   |
| -------------- | --------------------------- | -------------------------------- |
| Holds          | User-provided values        | Reusable logic                   |
| Example Data   | Image name, ports, replicas | Naming conventions, labels       |
| Syntax         | Plain YAML                  | Go templating syntax             |
| Accessed via   | `.Values.<key>`             | `include "helper.name" .`        |
| Changes often? | Yes, by user                | Rarely, only by chart maintainer |
| Output Type    | Raw values                  | Processed text/snippet           |
| Example Output | `replicas: 3`               | `metadata.name: coinapp-mychart` |

---

so you can **see how `_helpers.tpl` changes the output**?
It will help make everything crystal clear.
