### 🚀 **Helm Day 5 — Understanding Helm Template Engine (Deep Dive into Templates)**

Welcome back, Mounisha! 😊
Today we’ll focus on **Helm templates**, the *heart* of Helm charts — this is where real magic happens.

---

## 🧠 What Are Helm Templates?

Helm uses a **template engine (Go templates)** to dynamically generate Kubernetes YAML manifests.

When you install a chart, Helm **renders** all templates (like Deployment, Service, ConfigMap, etc.) by replacing variables and logic defined inside `{{ ... }}`.

---

### 🏗️ Template Structure Example

Let’s open your chart structure again:

```
mychart/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── _helpers.tpl
```

---

### 🧩 Example: deployment.yaml (Template)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-deployment
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Values.app.name }}
  template:
    metadata:
      labels:
        app: {{ .Values.app.name }}
    spec:
      containers:
        - name: {{ .Values.app.name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: {{ .Values.service.port }}
```

---

### 📘 Corresponding values.yaml

```yaml
replicaCount: 2

app:
  name: myapp

image:
  repository: nginx
  tag: latest

service:
  port: 80
```

---

### 🧮 How Helm Renders Templates

When you run:

```bash
helm install demo ./mychart
```

Helm will replace the template variables with values:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: "nginx:latest"
          ports:
            - containerPort: 80
```

So, Helm dynamically generates YAMLs that Kubernetes can understand.

---

## 🧠 Key Template Symbols You’ll See Often

| Syntax                                | Meaning                          |
| ------------------------------------- | -------------------------------- |
| `{{ .Values.key }}`                   | Access values from `values.yaml` |
| `{{ .Release.Name }}`                 | Name of the Helm release         |
| `{{ .Chart.Name }}`                   | Chart name from `Chart.yaml`     |
| `{{ include "template.name" . }}`     | Import a helper template         |
| `{{ if }}`, `{{ else }}`, `{{ end }}` | Conditional logic                |
| `{{ range }}`                         | Loops (for arrays or maps)       |

---

### 🧠 Example: Using `if` condition

```yaml
{{- if .Values.autoscaling.enabled }}
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: {{ .Release.Name }}-hpa
spec:
  minReplicas: {{ .Values.autoscaling.min }}
  maxReplicas: {{ .Values.autoscaling.max }}
{{- end }}
```

If `autoscaling.enabled` is `true` in `values.yaml`, Helm includes this YAML.
If it’s `false`, Helm **skips** it.

---

### 🧠 Example: Using `range` loop

```yaml
{{- range .Values.env }}
- name: {{ .name }}
  value: {{ .value }}
{{- end }}
```

With this in `values.yaml`:

```yaml
env:
  - name: MODE
    value: "prod"
  - name: VERSION
    value: "v1"
```

Helm renders:

```yaml
- name: MODE
  value: "prod"
- name: VERSION
  value: "v1"
```

---

## ⚙️ Preview Templates Without Installing

You can see what Helm will generate *before deploying* using:

```bash
helm template mychart/
```

This shows the full YAML output after all templating — **no need to install**.

---

## 🧩 Homework for Day 5

✅ 1. Create a chart `helm create day5chart`
✅ 2. Edit `values.yaml` and `deployment.yaml` to include variables (like app name, image, port).
✅ 3. Run `helm template day5chart/` to see the rendered YAML.
✅ 4. Try adding an `if` condition for autoscaling or resource limits.

---

Would you like me to **show you step-by-step** how to write a small custom template (e.g., Deployment + Service using your own app like `coinapp`)?
That will be your *hands-on practical for Day 5*.
