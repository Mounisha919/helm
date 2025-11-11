Excellent consistency, Mounisha! 🌟👏
You’re progressing beautifully — today’s **Day 9** will take you one big step closer to becoming a **Helm pro** 🚀

---

# 🧭 **Day 9 — Helm Template Functions (Advanced Template Power)**

Now that you understand `if`, `range`, and conditions, it’s time to learn the **functions** Helm gives you to make your templates even smarter 💡

---

## 🧠 What Are Helm Template Functions?

Helm templates use **Go template functions**, plus **extra Helm functions**, to transform or compute values.
They help you:

* Format text
* Convert data types
* Join lists
* Set defaults
* Do arithmetic
* And more!

Think of them like **mini tools inside your template** 🔧

---

## 🧩 **Categories of Functions**

Let’s divide them into simple categories with examples 👇

---

### 🧾 1️⃣ String Functions

| Function  | Description              | Example                            | Output       |
| --------- | ------------------------ | ---------------------------------- | ------------ |
| `upper`   | Converts to uppercase    | `{{ upper "coinapp" }}`            | `COINAPP`    |
| `lower`   | Converts to lowercase    | `{{ lower "CoinApp" }}`            | `coinapp`    |
| `title`   | Capitalizes first letter | `{{ title "helm chart" }}`         | `Helm Chart` |
| `trim`    | Removes spaces           | `{{ trim " coinapp " }}`           | `coinapp`    |
| `replace` | Replace substring        | `{{ replace "-" "_" "coin-app" }}` | `coin_app`   |

💬 **Usage Example in Deployment:**

```yaml
metadata:
  name: {{ replace "-" "_" .Chart.Name }}
```

---

### 🔢 2️⃣ Numeric Functions

| Function | Description  | Example          | Output |
| -------- | ------------ | ---------------- | ------ |
| `add`    | Adds numbers | `{{ add 2 3 }}`  | `5`    |
| `sub`    | Subtracts    | `{{ sub 10 4 }}` | `6`    |
| `mul`    | Multiply     | `{{ mul 2 4 }}`  | `8`    |
| `div`    | Divide       | `{{ div 10 2 }}` | `5`    |

💬 Example:

```yaml
replicas: {{ add .Values.replicaCount 1 }}
```

→ If `replicaCount: 2`, Helm renders `replicas: 3`.

---

### 🧩 3️⃣ Default Function

This is **one of the most used functions** in Helm.

💡 It sets a fallback value if something is missing.

Example:

```yaml
{{ default "nginx" .Values.image.repository }}
```

➡️ If user didn’t provide `.Values.image.repository`, Helm uses `"nginx"` as default.

---

### 🔗 4️⃣ Join and Split

Used to handle lists or strings.

| Function | Description            | Example                                   | Output          |
| -------- | ---------------------- | ----------------------------------------- | --------------- |
| `join`   | Combine a list         | `{{ join "," (list "dev" "qa" "prod") }}` | `dev,qa,prod`   |
| `split`  | Break string into list | `{{ split "," "dev,qa,prod" }}`           | `[dev qa prod]` |

💬 Example:

```yaml
labels:
  envs: "{{ join "," .Values.environments }}"
```

➡️ If environments = `["dev","qa","prod"]`, output = `envs: "dev,qa,prod"`

---

### 📦 5️⃣ ToYaml and Indent

Used to print structured YAML neatly.

```yaml
env:
{{ toYaml .Values.env | indent 2 }}
```

If `values.yaml` has:

```yaml
env:
  - name: ENV
    value: dev
```

Output:

```yaml
env:
  - name: ENV
    value: dev
```

➡️ `toYaml` converts map/list → YAML
➡️ `indent 2` adds two spaces (to keep structure proper)

---

### 🧮 6️⃣ Lookup Function (Important!)

Used to **fetch live data from Kubernetes cluster** during rendering.
⚠️ Works only if cluster is connected.

Example:

```yaml
{{- $svc := lookup "v1" "Service" "default" "my-svc" -}}
{{- if $svc }}
service exists
{{- else }}
service not found
{{- end }}
```

It checks whether Service `my-svc` exists in default namespace.

---

### 🔍 7️⃣ Required Function

Used to **enforce that a value must be set**, otherwise Helm fails.

Example:

```yaml
image: "{{ required "Image repo is required!" .Values.image.repository }}"
```

If user didn’t give `image.repository`, Helm throws an error:

```
Error: Image repo is required!
```

---

### 🧰 8️⃣ Include and Template

We’ve already used these — but let’s recap clearly.

| Function   | Purpose                         | Example                              |
| ---------- | ------------------------------- | ------------------------------------ |
| `include`  | Call helper from `_helpers.tpl` | `{{ include "mychart.fullname" . }}` |
| `template` | Define inline sub-template      | `{{ template "mychart.labels" . }}`  |

---

## ⚙️ Combining Multiple Functions

You can **pipe functions together** using `|` (pipe operator)
→ just like in Linux commands.

Example:

```yaml
metadata:
  name: {{ .Chart.Name | lower | replace "-" "_" }}
```

➡️ Takes chart name → lowercase → replace “-” with “_”

---

## 🧩 Practice Example — Helm Template Functions in Action

### 🧾 `values.yaml`

```yaml
app:
  name: Coin-App
replicaCount: 2
env:
  - name: MODE
    value: "dev"
  - name: VERSION
    value: "v1"
```

### 🧾 `deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.app.name | lower | replace "-" "" }}
spec:
  replicas: {{ add .Values.replicaCount 1 }}
  template:
    spec:
      containers:
        - name: {{ default "coinapp" .Values.app.name | lower }}
          image: nginx
          env:
{{ toYaml .Values.env | indent 12 }}
```

### 🧾 Output after rendering:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: coinapp
spec:
  replicas: 3
  template:
    spec:
      containers:
        - name: coinapp
          image: nginx
          env:
            - name: MODE
              value: dev
            - name: VERSION
              value: v1
```

You can see how the **functions** shaped the output.

---

## 🧠 Summary Table

| Category   | Common Functions                    | Usage              |
| ---------- | ----------------------------------- | ------------------ |
| String     | `upper`, `lower`, `replace`, `trim` | Formatting text    |
| Numbers    | `add`, `sub`, `mul`, `div`          | Calculations       |
| YAML       | `toYaml`, `indent`                  | Formatting YAML    |
| Validation | `required`, `default`               | Ensuring safety    |
| Logic      | `include`, `lookup`                 | Reuse or query     |
| Lists      | `join`, `split`                     | Combine/split data |

---

## ✅ Practice Tasks for You

1️⃣ In your chart, use `default` to give a fallback for image name.
2️⃣ Use `add` to increase replicas by 1.
3️⃣ Use `toYaml` to print env variables properly.
4️⃣ Try combining `upper`, `replace`, `indent`.
5️⃣ Finally, run:

```bash
helm template mychart/
```

and observe the rendered YAML.

---

Would you like me to show you **a side-by-side visual example** (values.yaml, template.yaml, and rendered output) using multiple functions (like `default`, `replace`, `toYaml`, `indent`)?
That will make Day 9 crystal clear visually 🔍
