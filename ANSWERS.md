# GNX DevOps Machine Test — Answers
**Candidate Name:**
**Date:**

---

## Q1. CrashLoopBackOff Diagnosis

> Your `gnx-backend` pod is in `CrashLoopBackOff`. Write the exact `kubectl` commands
> you would run to diagnose it and explain what you look for in each output.

**Answer:**

Check the Pod status using the command
"kubectl get pods".
Check for the pod named "gnx-backend" and whether it's status is CrashLoopBackOff, Number of restarts and the fequency of restarts.

Inspect the event, exit codes and termination reasons of the pod using the command
"kubectl describe pod gnx-backend".
Check the application logs using the command
"kubectl logs"  and  "kubectl logs --previous"
to identify the exact crash. 
If the logs indicate memory issue, verify using the command "kubectl top pod" and inspect the resource limits.
Review the deployment configuration, ConfigMaps, Secrets and health probes.  Also check the namespace events and also exec into the container to validate environment environment variables, DNS resolution, and connectivity to databases or external services

-------

## Q2. Frontend Cannot Reach Backend

> The frontend can't connect to the backend inside Kubernetes. The backend Service exists.
> What are the possible causes and how do you debug each one?

**Answer:**

Verify the backend pods are running using the command
"kubectl get pods -n production" .
Check if the backend pod is in running state, Is it in crashLoopBackOff or is it in ready state.
Verify the service using the command
"kubectl getb svc -n production" .
Check for the service name, Cluster IP assigned, Correct port and correct target port.

Check whether the Service has endpoints using the command "kubectl get endpoints backend -n production" . If the service isn't pointing to any pods, the possible reasons are wrong labels, selector mismatch, Backend pod isn't ready.

Verify service selectors using the command
"kubectl describe svc backend -n production" and comapare the pod labels.

Test DNS resolution using the command
"kubectl exec -t frontend-pod -n production -- sh"
and run "getnet hosts backend". If DNS fails, the possible reasons maybe wrong service name, wrong namespace, CoreDNS issue.

Test Connectivity 
"curl http://backend:8080" .
Inspect  any network policies that might block traffic, ensure both applications are using the correct namespace and finally review both the frontend and backend service logs for connection or application errors.


---

## Q3. Why PersistentVolumeClaim for PostgreSQL?

> Why did you use a PersistentVolumeClaim for PostgreSQL? What would happen to your
> todo data if you removed the PVC and used a plain Deployment without it?

**Answer:**

I used a PersistentVolumeClaim because PostgreSQL is a stateful application that stores data on disk. Kubernetes pods are ephemeral, so if PostgreSQL runs without a PVC, all database files are stored inside the container's writable layer. Whenever the pod is deleted, restarted, or recreated during deployments or node failures, that filesystem is lost, causing all database data—including the todo records—to be erased. A PVC provides persistent storage by mounting a PersistentVolume into the pod. When Kubernetes creates a replacement pod, it reattaches the same volume, allowing PostgreSQL to continue using the existing database files. This ensures that application data survives pod restarts, upgrades, and rescheduling, which is essential for production environments.

---

## Q4. Pipeline Ran But Pods Still Show Old Image

> Your CI/CD pipeline built and pushed a new image, but after the deploy stage,
> the pods are still running the old image. What are the possible reasons?

**Answer:**

If my pipeline built and pushed a new image but the pods were still running the old version, I'd first verify the image configured in the Deployment using "kubectl describe deployment" or "kubectl get deployment -o yaml" command. 

I'd confirm the Deployment references the newly built image tag. Next, I'd check whether the pipeline uses immutable image tags or reuses latest, because static tags combined with "imagePullPolicy: IfNotPresent" can cause Kubernetes to reuse a cached image. I'd then inspect the rollout status and rollout history to confirm a new ReplicaSet was created and that the rollout completed successfully. If not, I'd investigate deployment events for readiness probe failures, image pull errors, or CrashLoopBackOff issues. I'd also verify the image exists in the container registry, ensure the deployment targeted the correct cluster and namespace, and review the CI/CD pipeline logs to make sure the deploy stage used the same image tag produced by the build stage. This step-by-step process helps identify whether the issue is with image tagging, deployment configuration, rollout execution, or the CI/CD pipeline itself.

---

## Q5. Secret vs ConfigMap

> What is the difference between a Kubernetes Secret and a ConfigMap?
> Why did you use a Secret for the database password instead of a ConfigMap?

**Answer:**

A ConfigMap is used to store non-sensitive configuration such as application settings, hostnames, ports, or log levels. A Secret is used to store sensitive information like passwords, API keys, and certificates.

I used a Secret for the PostgreSQL database password because credentials should not be stored as plain configuration. Secrets provide a dedicated mechanism for managing sensitive data, can be protected with RBAC, and support encryption at rest when configured. While Secrets are only Base64-encoded by default, they are still the correct Kubernetes resource for credentials, and in production I would also enable encryption at rest and, if required, integrate with an external secrets manager.

---

## Q6. What is a Readiness Probe?

> Explain what a Readiness Probe does. What happens to a pod's traffic
> if the readiness probe fails?

**Answer:**

A Readiness Probe tells Kubernetes whether a container is ready to receive traffic. Kubernetes periodically checks a configured endpoint, command, or TCP socket. If the readiness probe succeeds, the pod is marked as Ready and added to the Service's endpoints, allowing it to receive requests. If the readiness probe fails, Kubernetes marks the pod as Not Ready and removes it from the Service's endpoints, so no new traffic is routed to it. The container continues running and is not restarted. Kubernetes keeps checking the probe, and once it succeeds again, the pod is automatically added back into the load balancer. This mechanism prevents traffic from reaching applications that are still starting up or are temporarily unable to serve requests.

---

## Q7. Rolling Update vs Recreate

> What is the difference between a RollingUpdate and a Recreate deployment strategy?
> When would you choose one over the other?

**Answer:**

A RollingUpdate deployment replaces old pods with new ones gradually, ensuring that healthy pods continue serving traffic throughout the deployment. This provides little or no downtime and is the default deployment strategy in Kubernetes. A Recreate deployment, on the other hand, terminates all existing pods before starting the new ones, which results in downtime but guarantees that old and new versions never run simultaneously. I would choose RollingUpdate for production web applications, APIs, and microservices where high availability is important. I would choose Recreate only when the application cannot safely run multiple versions at the same time, such as with certain legacy applications or workloads that require exclusive access to shared resources.
