# grafana-lgtm

### 🔍 What is the **LGTM stack**?

Think of **LGTM** as a toolkit that helps you monitor and understand everything happening in your software system — like watching the health of your apps, tracking errors, and figuring out what went wrong when something breaks.

**LGTM stands for:**

* **L**oki – Logs
* **G**rafana – Graphs (Visuals)
* **T**empo – Traces (Performance tracking)
* **M**imir – Metrics (Numbers like CPU usage, traffic, etc.)

---

### 🧱 Each component in simple terms:

---

#### 📄 **Grafana Loki** – *Log Aggregation System*

**What it does:**
Imagine every app on your computer writes what it's doing into a notepad — these are **logs**.
Loki collects all these logs from different apps and systems in one place, so you can search and analyze them easily.

**Layman analogy:**
Like having **one big searchable diary** of everything your apps are doing.

---

#### 📊 **Grafana** – *Data Visualization*

**What it does:**
Grafana takes all your data — logs, metrics, traces — and shows them in nice charts, dashboards, and graphs.

**Layman analogy:**
Like a **weather dashboard** that shows temperature, wind speed, and humidity — but for your servers and apps.

---

#### 🕵️ **Grafana Tempo** – *Distributed Tracing*

**What it does:**
Tempo tracks how a request moves through your system.
Example: A user clicks a button → the request goes through service A → then B → then C → then returns the result. Tempo records that whole journey.

**Layman analogy:**
Like **tracking a food delivery** from order to delivery — so you know where delays happen.

---

#### 📈 **Grafana Mimir** – *Metrics Time Series Database (TSDB)*

**What it does:**
Mimir stores and manages numeric data over time — like CPU usage, memory, number of users, etc.

**Layman analogy:**
Like a **fitness app** recording your heartbeat, steps, and calories every day — but for servers.

---

#### 🛠️ **Grafana Agent** (Deprecated) / **Grafana Alloy** (New) – *Data Collector*

**What it does:**
This tool runs on your systems and **collects data** (logs, metrics, traces) to send to Loki, Tempo, and Mimir.

* **Grafana Agent**: Original version
* **Grafana Alloy**: New and improved version

**Layman analogy:**
Like a **mailman** who collects reports from all your devices and delivers them to the LGTM system.

---

### 🎯 Summary

| Tool            | What it handles | Real-world comparison                      |
| --------------- | --------------- | ------------------------------------------ |
| **Loki**        | Logs            | A diary of everything happening            |
| **Grafana**     | Dashboards      | A control panel to view everything         |
| **Tempo**       | Traces          | Order tracking through different steps     |
| **Mimir**       | Metrics         | Fitness tracker for your servers           |
| **Alloy/Agent** | Data collector  | Mailman gathering all your app/system data |

---

### ✅ Why use LGTM Stack?

* Easy to **see what’s happening** in your system
* Helps **debug faster** when things go wrong
* **Open-source** and works well with Kubernetes, Docker, etc.
* Scalable for both small and big systems







## 🎯 Goal:

Deploy **Grafana LGTM stack** (Loki, Grafana, Tempo, Mimir) on Minikube and a **sample app**, and view everything (logs, metrics, traces) via Grafana.

---

## 🧰 What You Need Installed

Please make sure you have these:

| Tool         | Description                        | Download Link                                                                        |
| ------------ | ---------------------------------- | ------------------------------------------------------------------------------------ |
| **Minikube** | Runs Kubernetes locally            | [https://minikube.sigs.k8s.io/docs/start/](https://minikube.sigs.k8s.io/docs/start/) |
| **kubectl**  | Command tool to talk to Kubernetes | [https://kubernetes.io/docs/tasks/tools/](https://kubernetes.io/docs/tasks/tools/)   |
| **Helm**     | Tool to install apps in Kubernetes | [https://helm.sh/docs/intro/install/](https://helm.sh/docs/intro/install/)           |

If not sure, type:

```bash
minikube version
kubectl version --client
helm version
```

---

## 🛠️ Step-by-Step Instructions

### ✅ Step 1: Start Minikube

```bash
minikube start --cpus=4 --memory=8192
```

---

### ✅ Step 2: Create Namespace

```bash
kubectl create namespace monitoring
```

---

### ✅ Step 3: Add Helm Repositories

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

---

### ✅ Step 4: Install Loki (for logs)

```bash
helm install loki grafana/loki-stack \
  --namespace monitoring \
  --set promtail.enabled=true
```

---

### ✅ Step 5: Install Tempo (for tracing)

```bash
helm install tempo grafana/tempo-distributed \
  --namespace monitoring
```

---

### ✅ Step 6: Install Mimir (for metrics)

```bash
helm install mimir grafana/mimir-distributed \
  --namespace monitoring
```

---

### ✅ Step 7: Install Grafana (Dashboard UI)

```bash
helm install grafana grafana/grafana \
  --namespace monitoring \
  --set adminPassword='admin' \
  --set service.type=NodePort
```

---

## 🌐 How to Access Grafana

Run this command to get the **Grafana dashboard URL**:

```bash
minikube service grafana --namespace monitoring --url
```

➡️ It will show something like:

```
http://127.0.0.1:32846
```

Open that URL in your browser.

**Login:**

* Username: `admin`
* Password: `admin`

✅ You are now in Grafana!

---

## 🧪 Step 8: Deploy a Sample App

This app:

* Logs output (for Loki)
* Has metrics (for Mimir)
* Sends traces (for Tempo)

Run:

```bash
kubectl apply -f https://raw.githubusercontent.com/BenedictNS/k8s-observability-demo/main/k8s-manifests/deployment.yaml
```

---

## 🔌 Step 9: Connect Data Sources in Grafana

In Grafana:

1. Click **Settings → Data Sources**
2. Click **Add data source** and add:

| Name      | Type       | URL                                                                        |
| --------- | ---------- | -------------------------------------------------------------------------- |
| **Loki**  | Loki       | `http://loki.monitoring.svc.cluster.local:3100`                            |
| **Tempo** | Tempo      | `http://tempo-query-frontend.monitoring.svc.cluster.local:3100`            |
| **Mimir** | Prometheus | `http://mimir-query-frontend.monitoring.svc.cluster.local:8080/prometheus` |

Click **Save & Test** for each.

---

## 🔍 Step 10: View Logs, Metrics, and Traces

### 📜 View Logs:

1. Go to **Explore** (from the left menu)
2. Select **Loki** from top dropdown
3. Run query:

   ```
   {app="otel-demo-app"}
   ```
4. You’ll see logs from your app.

---

### 📊 View Metrics:

1. Go to **Explore**
2. Select **Mimir (Prometheus)** from the dropdown
3. Type:

   ```
   http_requests_total
   ```
4. You’ll see HTTP request graphs.

---

### 🚀 View Traces:

1. Go to **Explore**
2. Select **Tempo**
3. You’ll see traces (filter by service name `otel-demo-app`)

---

## 🔁 Step 11: Test It

Send a request to generate logs, metrics, and traces:

```bash
POD=$(kubectl get pods -l app=otel-demo-app -o jsonpath="{.items[0].metadata.name}")
kubectl exec -it $POD -- curl http://localhost:8080
```

Now:

* Go to **Explore → Loki** to see the new log line
* Go to **Explore → Mimir** to see metrics
* Go to **Explore → Tempo** to see a trace

---

## 🏁 Done!

You’ve now deployed:

* ✅ Loki → logs
* ✅ Mimir → metrics
* ✅ Tempo → traces
* ✅ Grafana → dashboards
* ✅ A sample app to generate all of the above

