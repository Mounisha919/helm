Perfect, Mounisha! 🌟
Let’s begin **📘 Day 8 — Helm Template Logic (if, else, range, and conditions)**.
This is a *very powerful* and *super interesting* day — by the end of this, you’ll know how to make your Helm charts **smart and dynamic** 💪

---

# 🎓 **Day 8 — Helm Template Logic: if, else, range, conditions**

Until now, your templates have been static —
they just inserted values from `values.yaml`.

But what if you want to:

* Deploy something **only if a flag is true** ✅
* Or loop over a **list of ports or env vars** 🔁

Helm lets you do that using **Go template logic** — which means you can add conditions (`if`, `else`) and loops (`range`).

---

## 🧩 Step 1: Why We Need Logic

Let’s imagine you have a chart that sometimes needs an **Ingress**, sometimes doesn’t.

You could use an `if` condition like this:

```yaml
{{- if .Values.ingress.enabled }}
# create ingress resource
{{- end }}
```

If `ingress.enabled` is `false`, Helm won’t even create that YAML section.

👉 This makes Helm charts reusable across environments — no need for multiple charts.

---

## 🧠 Step 2: Understanding Template Syntax

Helm uses **Go templating**, so logic looks like:

| Type    | Syntax                                            | Description                         |
| ------- | ------------------------------------------------- | ----------------------------------- |
| If      | `{{ if CONDITION }} ... {{ end }}`                | Execute only if true                |
| If-Else | `{{ if CONDITION }} ... {{ else }} ... {{ end }}` | Run one of two blocks               |
| Range   | `{{ range ITEMS }} ... {{ end }}`                 | Loop over lists or maps             |
| With    | `{{ with OBJECT }} ... {{ end }}`                 | Simplify context for nested objects |

---

## 🧩 Step 3: `if` and `else` Example

### 🧾 `values.yaml`

```yaml
ingress:
  enabled: true
  host: myapp.example.com
```

### 🧾 `templates/ingress.yaml`

```yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ .Release.Name }}-ingress
spec:
  rules:
  - host: {{ .Values.ingress.host }}
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: {{ .Release.Name }}-svc
            port:
              number: 80
{{- else }}
# Ingress is disabled
{{- end }}
```

---

### 🧪 When you run:

```bash
helm template myapp ./mychart
```

✅ Output:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-svc
            port:
              number: 80
```

Now if you set:

```yaml
ingress:
  enabled: false
```

✅ Output:

```yaml
# Ingress is disabled
```

🎯 Helm doesn’t even create that Kubernetes object — so your deployment becomes environment-specific!

---

## 🧩 Step 4: Using `range` (Loops)

What if your app has multiple ports or environment variables?
You don’t want to write each manually.

### 🧾 `values.yaml`

```yaml
containerPorts:
  - 80
  - 443
  - 8080
```

### 🧾 `templates/deployment.yaml`

```yaml
containers:
- name: myapp
  image: nginx
  ports:
  {{- range .Values.containerPorts }}
  - containerPort: {{ . }}
  {{- end }}
```

✅ Rendered output:

```yaml
containers:
- name: myapp
  image: nginx
  ports:
  - containerPort: 80
  - containerPort: 443
  - containerPort: 8080
```

Helm automatically loops through all values in the list! 🔁

---

## 🧩 Step 5: Using `range` with Key-Value Pairs

You can also loop through maps (dictionaries).

### 🧾 `values.yaml`

```yaml
env:
  DB_USER: admin
  DB_PASS: secret
```

### 🧾 Template:

```yaml
env:
{{- range $key, $value := .Values.env }}
- name: {{ $key }}
  value: "{{ $value }}"
{{- end }}
```

✅ Rendered output:

```yaml
env:
- name: DB_USER
  value: "admin"
- name: DB_PASS
  value: "secret"
```

🎯 So, if you add or remove environment variables in `values.yaml`,
you don’t touch the template at all — it automatically adapts!

---

## 🧩 Step 6: Using `with` for Simplicity

`with` helps when you have nested structures.

### 🧾 `values.yaml`

```yaml
image:
  repository: nginx
  tag: latest
```

### 🧾 Template:

```yaml
{{- with .Values.image }}
image: "{{ .repository }}:{{ .tag }}"
{{- end }}
```

✅ Output:

```yaml
image: "nginx:latest"
```

Without `with`, you’d have to write:

```yaml
image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

`with` helps shorten things.

---

## 🧩 Step 7: Combining `if`, `range`, and `with`

You can combine them to create **powerful dynamic logic** 💪

Example:

```yaml
{{- if .Values.env }}
env:
{{- range $key, $value := .Values.env }}
- name: {{ $key }}
  value: "{{ $value }}"
{{- end }}
{{- else }}
# No environment variables defined
{{- end }}
```

✅ If `.Values.env` exists → prints env vars
✅ If not → prints comment line

---

## 🧠 Step 8: Real-World Example — Optional Service Monitor

```yaml
{{- if .Values.serviceMonitor.enabled }}
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: {{ include "mychart.fullname" . }}
spec:
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  endpoints:
  - port: metrics
{{- end }}
```

And in `values.yaml`:

```yaml
serviceMonitor:
  enabled: false
```

👉 You can simply toggle `enabled: true` or `false` to control creation of the ServiceMonitor resource — no need to delete YAML files!

---

## 🧩 Step 9: Helpful Operators in Helm

| Operator | Meaning     | Example                                                             |
| -------- | ----------- | ------------------------------------------------------------------- |
| `eq`     | equals      | `{{ if eq .Values.env "prod" }}`                                    |
| `ne`     | not equals  | `{{ if ne .Values.env "dev" }}`                                     |
| `and`    | logical AND | `{{ if and (eq .Values.env "prod") (.Values.monitoring.enabled) }}` |
| `or`     | logical OR  | `{{ if or .Values.debug .Values.devMode }}`                         |
| `not`    | negate      | `{{ if not .Values.disabled }}`                                     |

You can use these to make smart condition checks.

---

## 🧭 Step 10: Summary

| Concept                 | Description                       | Example                                |
| ----------------------- | --------------------------------- | -------------------------------------- |
| `if` / `else`           | Conditional rendering             | `{{ if .Values.enabled }}...{{ end }}` |
| `range`                 | Loop through list or map          | `{{ range .Values.list }}`             |
| `with`                  | Shorten nested access             | `{{ with .Values.image }}`             |
| `eq`, `ne`, `and`, `or` | Comparison operators              | `{{ if eq .Values.env "prod" }}`       |
| Real use cases          | Optional ingress, env vars, loops | ✅                                      |

---

## 💪 Practice Task for You

1️⃣ Create a chart `day8-demo`:

```bash
helm create day8-demo
```

2️⃣ In `values.yaml`, add:

```yaml
env:
  NAME: Mounisha
  PROJECT: HelmLearning
containerPorts:
  - 80
  - 443
showMessage: true
```

3️⃣ In `deployment.yaml`, add:

```yaml
{{- if .Values.showMessage }}
# Mounisha’s Helm learning chart 💫
{{- end }}
containers:
- name: app
  image: nginx
  env:
  {{- range $key, $value := .Values.env }}
  - name: {{ $key }}
    value: "{{ $value }}"
  {{- end }}
  ports:
  {{- range .Values.containerPorts }}
  - containerPort: {{ . }}
  {{- end }}
```

4️⃣ Render output:

```bash
helm template mytest ./day8-demo
```

✅ You’ll see:

```yaml
# Mounisha’s Helm learning chart 💫
containers:
- name: app
  image: nginx
  env:
  - name: NAME
    value: "Mounisha"
  - name: PROJECT
    value: "HelmLearning"
  ports:
  - containerPort: 80
  - containerPort: 443
```

---

## 🎯 Day 8 Recap

| Concept           | Example                         | Purpose            |
| ----------------- | ------------------------------- | ------------------ |
| `if`              | Create resource only if enabled | Control flow       |
| `else`            | Fallback logic                  | Default case       |
| `range`           | Loop through list/map           | Dynamic repetition |
| `with`            | Simplify context                | Clean templates    |
| `eq`, `and`, `or` | Logic operators                 | Smart conditions   |

---

Would you like me to continue next with **Day 9 — Built-in Objects & Functions in Helm** (like `.Release`, `.Chart`, `.Files`, `.Capabilities`, and built-in template functions such as `default`, `quote`, `upper`, etc.)?
That’s where we learn how Helm automatically gives you access to information about your chart and release 💡
