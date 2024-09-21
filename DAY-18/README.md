
# Kubernetes Pod Health Check Examples

This project demonstrates how to configure **liveness** and **readiness** probes for Kubernetes pods using different methods, including HTTP, TCP, and Exec-based probes.

## Pods

1. **BusyBox Pod with Exec-based Liveness Probe**
2. **Agnhost Pod with HTTP-based Liveness and Readiness Probes**
3. **Agnhost Pod with HTTP-based Probes (Example 2)**
4. **Goproxy Pod with TCP-based Liveness Probe**

## Prerequisites

- Kubernetes cluster
- `kubectl` installed and configured to interact with your cluster



## 1. BusyBox Pod with Exec-based Liveness Probe

This pod uses the `registry.k8s.io/busybox` image and has a liveness probe that checks if the file `/tmp/healthy` exists inside the container.

**YAML:**

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat 
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

### Apply:

```
kubectl apply -f liveness-exec.yaml
```

This pod will be restarted if the `/tmp/healthy` file is not found after the liveness probe is executed.



## 2. Agnhost Pod with HTTP-based Liveness and Readiness Probes

This pod uses the `registry.k8s.io/e2e-test-images/agnhost:2.40` image and defines both liveness and readiness probes. The probes check the `/healthz` path on port `8080`.

**YAML:**

```
apiVersion: v1
kind: Pod
metadata:
  name: hello
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/e2e-test-images/agnhost:2.40
    args:
    - liveness
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 3
      periodSeconds: 3
    readinessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 10
```

### Apply:

```
kubectl apply -f hello-pod.yaml
```

This pod will be checked for both liveness and readiness using HTTP GET requests to `/healthz` on port `8080`.


## 3. Goproxy Pod with TCP-based Liveness Probe

This pod uses the `registry.k8s.io/goproxy:0.1` image and defines a TCP-based liveness probe. It checks if the container is listening on port `3000`.

**YAML:**

```
apiVersion: v1
kind: Pod
metadata:
  name: tcp-pod
  labels:
    app: tcp-pod
spec:
  containers:
  - name: goproxy
    image: registry.k8s.io/goproxy:0.1
    ports:
    - containerPort: 8080
    livenessProbe:
      tcpSocket:
        port: 3000
      initialDelaySeconds: 10
      periodSeconds: 5
```

### Apply:

```
kubectl apply -f tcp-pod.yaml
```

This pod will be restarted if it is not listening on port `3000`.


## Verifying the Pods

You can verify the status of the pods and the probes by running the following commands:

### Check Pod Status:

```
kubectl get pods
```

### Check Pod Details:

```
kubectl describe pod <pod-name>
```

### Check Pod Logs:

```
kubectl logs <pod-name>
```

### Example:

```
kubectl describe pod liveness-exec
```

## Conclusion

These YAML configurations demonstrate how to configure different types of liveness and readiness probes for Kubernetes pods. Each probe type (HTTP, TCP, and Exec) ensures that the Kubernetes system can check the health of running applications and take appropriate actions to maintain the system's stability.