# GNX Solutions — DevOps Engineer Machine Test
**Level:** Junior (1–3 years)
**Time Limit:** 24 Hours

---

## Instructions

1. **Fork this repository** to your own GitHub or GitLab account — make it public
2. Run the app locally first using Docker Compose to understand it
3. Complete all 7 tasks below
4. Submit by emailing your **repository URL** and **live app URL**

---

## Understanding the App

This is a simple Todo application with 3 services:

```
Browser → Frontend (Nginx) → /api/* → Backend (Node.js) → PostgreSQL
```

**Run it locally before starting:**
```bash
docker compose up --build
# Open http://localhost:8080
```

You should be able to add and delete todos. The data is stored in PostgreSQL.

---

## Task 1 — Docker

The Dockerfiles for `frontend` and `backend` are already written.

1. Build both images locally and verify the app works end-to-end via Docker Compose

2. Push both images to **Docker Hub** or **GitHub Container Registry (ghcr.io)**:
   ```
   <your-username>/gnx-frontend:v1
   <your-username>/gnx-backend:v1
   ```

**Deliverables:**
- Screenshot of `docker compose up` with all 3 services running
- Screenshot of the app working in your browser at `http://localhost:8080`
- Links to your pushed images on the registry

---

## Task 2 — Kubernetes Deployment

Deploy the full application to a Kubernetes cluster on any free cloud provider.

**Free options:**
- GKE Autopilot free tier
- AWS EKS with t3.micro free tier nodes
- Civo or Linode free trial

Create the following files inside the `k8s/` folder:

### k8s/namespace.yaml
- Create a namespace called `gnx-app`

### k8s/secret.yaml
- Create a Kubernetes Secret with these keys:
  - `DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`
- Use base64 encoded values (e.g. `postgres` → `cG9zdGdyZXM=`)
- ⚠️ Use dummy/test values only — do not commit real credentials

### k8s/postgres.yaml
- Deployment + Service + PersistentVolumeClaim
- PVC should request **1Gi** of storage
- Mount PVC to `/var/lib/postgresql/data`
- DB credentials must come from the Secret (not hardcoded)

### k8s/backend.yaml
- Deployment + Service
- 2 replicas
- Inject DB credentials from the Secret as environment variables
- Readiness probe on `/health` at port 3000

### k8s/frontend.yaml
- Deployment + Service
- 2 replicas
- Readiness probe on `/` at port 80

### k8s/ingress.yaml
- Expose the app externally
- Route `/api/` → backend service
- Route `/` → frontend service
- Use Nginx Ingress Controller (or whatever is available on your cluster)

**Deliverables:**
- All manifest files in `k8s/` folder
- Screenshot of `kubectl get all -n gnx-app` showing all pods Running
- Screenshot of the live app in your browser via the Ingress IP/URL
- Add a todo on the live app, refresh the page — it should still be there (proves DB persistence)

---

## Task 3 — CI/CD Pipeline

Set up a CI/CD pipeline using **GitHub Actions** or **GitLab CI**.

Trigger: every push to the `main` branch.

The pipeline must have 3 stages in this order:

```
build → push → deploy
```

**Build:** Build both Docker images, tag with the Git commit SHA:
```
<registry>/gnx-frontend:<commit-sha>
<registry>/gnx-backend:<commit-sha>
```

**Push:** Push both images to your container registry.
Registry credentials must be stored as **CI/CD secrets** — never hardcoded in the pipeline file.

**Deploy:** Update the Kubernetes deployment to use the new image tags.
The cluster kubeconfig must be stored as a **CI/CD secret**.

**Deliverables:**
- `.github/workflows/deploy.yml` or `.gitlab-ci.yml` in your repo
- Screenshot of a successful pipeline run with all 3 stages green
- Screenshot of `kubectl get pods -n gnx-app` after the pipeline run

---

## Task 4 — Monitoring

Deploy Prometheus and Grafana to your cluster using Helm.

### Step 1 — Install kube-prometheus-stack

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --set grafana.adminPassword=gnxadmin123
```

Verify everything is running:
```bash
kubectl get pods -n monitoring
```
Wait until all pods show `Running` before continuing.

### Step 2 — Expose Grafana via Ingress

Create a file `k8s/grafana-ingress.yaml` to expose Grafana publicly through your cluster's Ingress controller:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana-ingress
  namespace: monitoring
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /grafana
        pathType: Prefix
        backend:
          service:
            name: monitoring-grafana
            port:
              number: 80
```

Apply it:
```bash
kubectl apply -f k8s/grafana-ingress.yaml
```

Grafana will be accessible at:
```
http://<your-ingress-ip>/grafana
```

Login credentials:
- **Username:** `admin`
- **Password:** `gnxadmin123`

### Step 3 — View the Dashboard

1. Open `http://<your-ingress-ip>/grafana` in your browser
2. Navigate to **Dashboards → Kubernetes / Compute Resources / Namespace (Pods)**
3. Filter by namespace: `gnx-app`
4. Take a screenshot showing CPU and memory usage for your pods

**Deliverables:**
- `k8s/grafana-ingress.yaml` in your repo
- Screenshot of Grafana accessible in the browser via the Ingress URL (not port-forward)
- Screenshot of the **"Namespace (Pods)"** dashboard filtered to `gnx-app`
- Screenshot of `kubectl get pods -n monitoring` showing all pods Running

---

## Task 5 — Bash Scripting

Write a script `health-check.sh` in the root of the repo.

**Usage:**
```bash
./health-check.sh gnx-app
```

**Requirements:**
1. Print a table of all pods in the given namespace showing: pod name, status, restart count
2. Print a `WARNING` for any pod with more than 3 restarts
3. Exit with code `1` if any warnings exist, `0` if all pods are healthy

**Expected output:**
```
Checking pods in namespace: gnx-app
------------------------------------------
POD                            STATUS    RESTARTS
gnx-frontend-7d9f-abc          Running   0
gnx-backend-5c8d-xyz           Running   2
gnx-postgres-6b7f-def          Running   0
------------------------------------------
All pods healthy.
```

**Deliverables:**
- `health-check.sh` in the root of your repo (must be executable: `chmod +x health-check.sh`)
- Screenshot of the script running against your `gnx-app` namespace

---

## Task 6 — Architecture Diagram

Draw a diagram showing the complete system. It must include:

- Developer pushes code → CI/CD pipeline → Container Registry → Kubernetes cluster
- Inside Kubernetes: Namespace, Frontend pods, Backend pods, Postgres pod, PVC, Services, Ingress
- Monitoring: Prometheus + Grafana inside the cluster
- How external traffic flows in (Internet → Ingress → Frontend → Backend → DB)

Use any free tool: **Excalidraw** (excalidraw.com), **draw.io**, or a clear hand-drawn photo.

**Deliverables:**
- Save as `architecture.png` or `architecture.pdf` in the root of your repo

---

## Task 7 — Written Questions

Open `ANSWERS.md` and fill in your answers. All 7 questions are already written there.
Keep each answer to 5–10 lines. Write in your own words.

---

## Final Repository Structure

Your submitted repo should look like this:

```
gnx-todo-app/
├── frontend/
│   ├── Dockerfile
│   ├── index.html
│   └── nginx.conf
├── backend/
│   ├── Dockerfile
│   ├── index.js
│   └── package.json
├── k8s/
│   ├── namespace.yaml
│   ├── secret.yaml
│   ├── postgres.yaml
│   ├── backend.yaml
│   ├── frontend.yaml
│   ├── ingress.yaml
│   └── grafana-ingress.yaml
├── screenshots/
│   └── (all screenshots here)
├── .github/workflows/deploy.yml  ← OR .gitlab-ci.yml
├── docker-compose.yml
├── health-check.sh
├── architecture.png
├── ANSWERS.md                    ← fill this in
├── MACHINE_TEST.md
└── README.md
```

---

## Submission Checklist

Before submitting, verify every item:

- [ ] Repo is **public** and accessible via URL
- [ ] App is **live** — share the Ingress IP/URL (we will open it and test it)
- [ ] Add a todo on the live app, refresh — confirm it persists (DB is working)
- [ ] CI/CD pipeline has at least **one successful run** — share the pipeline run URL
- [ ] All screenshots are in `screenshots/` folder
- [ ] `health-check.sh` is executable (`chmod +x health-check.sh`)
- [ ] `ANSWERS.md` is filled in
- [ ] `architecture.png` or `architecture.pdf` is in the repo
- [ ] **No credentials, kubeconfig, or `.env` files are committed to the repo**

**Email your submission to:** hr@gnxsolutions.com
**Include:**
1. Repository URL
2. Live app URL
3. CI/CD pipeline run URL

---

## Evaluation Criteria

| Area | Weight | What We Look For |
|---|---|---|
| Docker | 10% | Images build, app runs locally |
| Kubernetes Manifests | 30% | All pods running, app live, data persists |
| Secrets handling | 10% | No hardcoded credentials anywhere |
| CI/CD Pipeline | 20% | All 3 stages pass, secrets handled correctly |
| Monitoring | 10% | Stack deployed, Grafana screenshot provided |
| Scripting | 5% | Script runs correctly, correct exit codes |
| Architecture Diagram | 5% | Accurate, covers all components |
| Written Answers | 10% | Shows genuine understanding |

**We care more about a working app with clean, readable files than a complex setup that doesn't run.**

---

*If you hit genuine blockers (cloud billing issues, cluster access problems) reach out to your interviewer — don't waste hours on account issues.*

*Good luck!*
