# GNX DevOps Machine Test — Answers
**Candidate Name:**
**Date:**

---

## Q1. CrashLoopBackOff Diagnosis

> Your `gnx-backend` pod is in `CrashLoopBackOff`. Write the exact `kubectl` commands
> you would run to diagnose it and explain what you look for in each output.

**Answer:**

_(Write your answer here)_

---

## Q2. Frontend Cannot Reach Backend

> The frontend can't connect to the backend inside Kubernetes. The backend Service exists.
> What are the possible causes and how do you debug each one?

**Answer:**

_(Write your answer here)_

---

## Q3. Why PersistentVolumeClaim for PostgreSQL?

> Why did you use a PersistentVolumeClaim for PostgreSQL? What would happen to your
> todo data if you removed the PVC and used a plain Deployment without it?

**Answer:**

_(Write your answer here)_

---

## Q4. Pipeline Ran But Pods Still Show Old Image

> Your CI/CD pipeline built and pushed a new image, but after the deploy stage,
> the pods are still running the old image. What are the possible reasons?

**Answer:**

_(Write your answer here)_

---

## Q5. Secret vs ConfigMap

> What is the difference between a Kubernetes Secret and a ConfigMap?
> Why did you use a Secret for the database password instead of a ConfigMap?

**Answer:**

_(Write your answer here)_

---

## Q6. What is a Readiness Probe?

> Explain what a Readiness Probe does. What happens to a pod's traffic
> if the readiness probe fails?

**Answer:**

_(Write your answer here)_

---

## Q7. Rolling Update vs Recreate

> What is the difference between a RollingUpdate and a Recreate deployment strategy?
> When would you choose one over the other?

**Answer:**

_(Write your answer here)_
