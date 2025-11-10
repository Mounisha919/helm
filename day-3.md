Awesome 🙌 Welcome to **Day 3 of Helm classes**!
You’ve done great so far — now that you understand what a chart and a release are, we’re ready to dive into the **core power of Helm**:
👉 **Templating and Values**

By the end of this lesson, you’ll understand how Helm dynamically fills values inside YAML files — so you can deploy **the same chart** to **dev, QA, and prod** with different configurations.

---

## 🧭 **Day 3 – Helm Templating Deep Dive**

---

### 🧠 1️⃣ What are Helm Templates?

Helm templates are **YAML files with dynamic placeholders** that get replaced during installation.

A normal Kubernetes deployment YAML looks like this:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 2
  template:
    spec:
      containers:
      - name: nginx
        image: nginx:latest
```

Now, instead of hardcoding values, Helm lets you **parameterize** it using variables:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-deployment
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

When Helm installs this chart, it replaces all `{{ ... }}` placeholders with actual values.

---

### ⚙️ 2️⃣ Understanding Template Variables

Let’s break them down 👇

| Variable                   | Meaning                             | Example Output |
| -------------------------- | ----------------------------------- | -------------- |
| `{{ .Values.xyz }}`        | Values from `values.yaml`           | `2`            |
| `{{ .Chart.Name }}`        | Name of the chart (from Chart.yaml) | `mychart`      |
| `{{ .Chart.AppVersion }}`  | Application version                 | `1.0.0`        |
| `{{ .Release.Name }}`      | Release name during install         | `myapp`        |
| `{{ .Release.Namespace }}` | Namespace used                      | `default`      |

---

### 🧩 3️⃣ Example: Simple Template + values.yaml

**values.yaml**

```yaml
replicaCount: 3

image:
  repository: nginx
  tag: latest

service:
  type: NodePort
  port: 80
```

**templates/deployment.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-deployment
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: {{ .Values.service.port }}
```

When you install:

```bash
helm install myapp ./mychart
```

Helm replaces the placeholders → resulting YAML looks like:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  template:
    spec:
      containers:
        - name: mychart
          image: "nginx:latest"
          ports:
            - containerPort: 80
```

✅ Done! This is how Helm templating dynamically injects values.

---

### 🧠 4️⃣ Templating Functions and Helpers

Helm templates use **Go templating language**, so we can do more than variable substitution.

#### Example – Default Value

```yaml
image: "{{ .Values.image.repository | default "nginx" }}"
```

If `image.repository` is missing, it defaults to `"nginx"`.

---

#### Example – Uppercase or Lowercase

```yaml
name: {{ .Release.Name | upper }}
```

If release name = `myapp` → output = `MYAPP`

---

#### Example – If Condition

```yaml
{{- if .Values.service.enabled }}
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
{{- end }}
```

So if in `values.yaml`:

```yaml
service:
  enabled: true
  type: ClusterIP
  port: 80
```

Then service will be created.
If `enabled: false` → Helm will skip creating it.

---

#### Example – Loop (for multiple ports)

```yaml
ports:
{{- range .Values.service.ports }}
  - port: {{ .port }}
    targetPort: {{ .targetPort }}
{{- end }}
```

**values.yaml**

```yaml
service:
  ports:
    - port: 80
      targetPort: 8080
    - port: 443
      targetPort: 8443
```

💡 Output → Helm will repeat those lines for each port.

---

### 📜 5️⃣ Helper Templates (`_helpers.tpl`)

This file defines reusable template functions.

Example – inside `_helpers.tpl`

```yaml
{{- define "mychart.labels" -}}
app.kubernetes.io/name: {{ .Chart.Name }}
app.kubernetes.io/version: {{ .Chart.AppVersion }}
{{- end -}}
```

Use it anywhere:

```yaml
metadata:
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
```

💡 `| nindent 4` → adds indentation so YAML remains valid.

---

### 🔍 6️⃣ See the Rendered YAML (Before Installing)

Very important for debugging 👇

```bash
helm install myapp ./mychart --dry-run --debug
```

This **doesn’t deploy**, it just shows the **final YAML output** Helm will apply.
Use this command to catch errors in templating or indentation before real deployment.

---

### ⚠️ 7️⃣ Common Errors & Fixes

| Error                          | Reason                      | Fix                                        |                                |
| ------------------------------ | --------------------------- | ------------------------------------------ | ------------------------------ |
| `template: unexpected EOF`     | Missing `}}` or `{%`        | Check all brackets                         |                                |
| `YAML parse error`             | Wrong indentation or spaces | Use 2 spaces, no tabs                      |                                |
| `cannot find value`            | Wrong variable path         | Check `.Values` key path                   |                                |
| `nindent` error                | Forgot to pipe correctly    | Use `                                      | nindent 4`not`nindent 4` alone |
| `helm install: already exists` | Same release name exists    | Uninstall previous: `helm uninstall myapp` |                                |

---

### 🧪 8️⃣ Practical Task for Today

1. Create a chart:

   ```bash
   helm create mychart
   ```
2. Edit `values.yaml` → change:

   ```yaml
   replicaCount: 3
   image:
     repository: nginx
     tag: latest
   service:
     enabled: true
     type: NodePort
     port: 8080
   ```
3. Edit `templates/deployment.yaml` → use templating like above.
4. Run:

   ```bash
   helm install myapp ./mychart --dry-run --debug
   ```
5. Check the rendered YAML output.

---

### 🏁 Summary

| Concept                  | Meaning                              |
| ------------------------ | ------------------------------------ |
| `{{ .Values }}`          | Pulls values from `values.yaml`      |
| `{{ .Chart }}`           | Pulls metadata from `Chart.yaml`     |
| `{{ .Release }}`         | Info about current release           |
| `if`, `range`, `include` | Go template features for logic       |
| `--dry-run --debug`      | View rendered output before applying |

---

Next class →
📘 **Day 4: Helm Chart Values and Overrides (real-world multi-environment setup)**
You’ll learn how to use `values-dev.yaml`, `values-prod.yaml`, and how to override configurations with `--set` and `-f` flags.

---

Would you like me to include **real “dev–qa–prod” environment examples** for Day 4 so you see how companies actually use Helm in production?
