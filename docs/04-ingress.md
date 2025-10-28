# Kubernetes Networking: Make Your App Accessible from the Internet

*Learn to route internet traffic to your Kubernetes application using production-grade networking*

## 🎯 **What You'll Learn**

By the end of this tutorial, you'll know how to:
- **Route internet traffic** to your Kubernetes services
- **Set up custom domains** for your application
- **Configure load balancing** for high availability
- **Handle SSL/TLS** for secure connections
- **Debug networking issues** like a professional

## ⏱️ **Time Required: 20-40 minutes**

## Why This Matters

An Ingress Controller is like the front door to your application. It routes internet traffic to your services, handles SSL certificates, and provides load balancing. This is how real applications become accessible to users worldwide.

**What this means for you**: Every production application needs networking. Learning Ingress teaches you how companies like Netflix and Airbnb make their services accessible to millions of users.

ℹ️ **Simple Explanation:** An Ingress Controller is like a smart traffic director. It looks at incoming requests (like "go to gameapp.local") and routes them to the right service in your Kubernetes cluster.

## Do This

### Step 1: Set Up Ingress Controller for External Access

```bash
# Install nginx-ingress controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/baremetal/deploy.yaml
```

**Expected Output:**
```bash
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
configmap/ingress-nginx-controller created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
service/ingress-nginx-controller-admission created
service/ingress-nginx-controller created
deployment.apps/ingress-nginx-controller created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
```

```bash
# Wait for ingress controller to be ready
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s
```

**Expected Output:**
```bash
pod/ingress-nginx-controller-xxx condition met
```

```bash
# Configure ingress controller to use hostPorts for k3d compatibility
kubectl patch deployment ingress-nginx-controller -n ingress-nginx -p '{"spec":{"template":{"spec":{"containers":[{"name":"controller","ports":[{"containerPort":80,"hostPort":80,"name":"http","protocol":"TCP"},{"containerPort":443,"hostPort":443,"name":"https","protocol":"TCP"},{"containerPort":8443,"name":"webhook","protocol":"TCP"}]}]}}}}'
```

**Expected Output:**
```bash
deployment.apps/ingress-nginx-controller patched
```

```bash
# Wait for deployment rollout to complete
kubectl rollout status deployment/ingress-nginx-controller -n ingress-nginx
```

**Expected Output:**
```bash
deployment "ingress-nginx-controller" successfully rolled out
```

```bash
# Deploy your application's ingress rules
kubectl apply -f k8s/ingress.yaml
```

**Expected Output:**
```bash
ingress.networking.k8s.io/humor-game-ingress configured
```

```bash
# Verify ingress is configured
kubectl get ingress -n humor-game
```

**Expected Output:**
```bash
NAME                 CLASS   HOSTS           ADDRESS      PORTS   AGE
humor-game-ingress   nginx   gameapp.local   172.18.0.3   80      2m
```

### Step 2: Configure Local Domain and Test Access

**Set up local domain for development:**
```bash
# Add local domain to your hosts file
echo "127.0.0.1 gameapp.local" | sudo tee -a /etc/hosts
```

**Expected Output:**
```bash
127.0.0.1 gameapp.local
```

```bash
# Verify DNS resolution works
ping gameapp.local
# Should ping 127.0.0.1 successfully
```

**Expected Output:**
```bash
PING gameapp.local (127.0.0.1): 56 data bytes
64 bytes from gameapp.local (127.0.0.1): icmp_seq=1 time=0.037 ms
64 bytes from gameapp.local (127.0.0.1): icmp_seq=2 time=0.034 ms
64 bytes from gameapp.local (127.0.0.1): icmp_seq=3 time=0.033 ms
--- gameapp.local ping statistics ---
3 packets transmitted, 3 received, 0% packet loss
```

**Test your Kubernetes application:**
```bash
# Test frontend through Ingress
curl -H "Host: gameapp.local" -I http://localhost:8080/
```

**Expected Output:**
```bash
HTTP/1.1 200 OK
Date: Sat, 30 Aug 2025 00:33:20 GMT
Content-Type: text/html
Content-Length: 16910
Connection: keep-alive
Last-Modified: Sat, 30 Aug 2025 00:01:16 GMT
ETag: "68b23f4c-420e"
Expires: Sat, 30 Aug 2025 00:38:20 GMT
Cache-Control: max-age=300
Cache-Control: public, must-revalidate
Accept-Ranges: bytes
```

```bash
# Test API health through Ingress
curl -H "Host: gameapp.local" http://localhost:8080/api/health
```

**Expected Output:**
```json
{
  "status": "healthy",
  "timestamp": "2025-08-30T00:33:26.867Z",
  "services": {
    "database": "connected",
    "redis": "connected",
    "api": "running"
  },
  "version": "1.0.0",
  "environment": "development"
}
```

```bash
# Open in browser with domain
open http://gameapp.local:8080
```

### Step 3: Verify Ingress Configuration

```bash
# Check ingress status
kubectl get ingress -n humor-game
# Should show: humor-game-ingress with nginx class
```

**Expected Output:**
```bash
NAME                 CLASS   HOSTS           ADDRESS      PORTS   AGE
humor-game-ingress   nginx   gameapp.local   172.18.0.3   80      13m
```

```bash
# Check ingress controller pods
kubectl get pods -n ingress-nginx
# Should show: nginx-ingress-controller pod with "1/1 Running"
```

**Expected Output:**
```bash
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-md6pr        0/1     Completed   0          23m
ingress-nginx-admission-patch-b2rx9         0/1     Completed   0          23m
ingress-nginx-controller-5445788fcd-qn4x2   1/1     Running     0          73s
```

```bash
# Verify ingress rules
kubectl describe ingress humor-game-ingress -n humor-game
# Should show rules for gameapp.local
```

### Step 4: Test Full Application Functionality

```bash
# Test frontend loads
curl -H "Host: gameapp.local" -I http://localhost:8080/
# Should return: HTTP/1.1 200 OK
```

**Expected Output:**
```bash
HTTP/1.1 200 OK
Date: Sat, 30 Aug 2025 00:33:20 GMT
Content-Type: text/html
Content-Length: 16910
Connection: keep-alive
Last-Modified: Sat, 30 Aug 2025 00:01:16 GMT
ETag: "68b23f4c-420e"
Expires: Sat, 30 Aug 2025 00:38:20 GMT
Cache-Control: max-age=300
Cache-Control: public, must-revalidate
Accept-Ranges: bytes
```

```bash
# Test API endpoints
curl -H "Host: gameapp.local" http://localhost:8080/api/health
# Should return: {"status":"healthy",...}
```

```bash
# Test metrics endpoint
curl -H "Host: gameapp.local" http://localhost:8080/metrics
# Should return Prometheus metrics
```

## You Should See...

**Ingress Status:**
```bash
NAME                 CLASS   HOSTS           ADDRESS      PORTS   AGE
humor-game-ingress   nginx   gameapp.local   172.18.0.3   80      13m
```

**Ingress Controller Status:**
```bash
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-controller-5445788fcd-qn4x2   1/1     Running     0          73s
```

**Host Resolution:**
```bash
PING gameapp.local (127.0.0.1): 56 data bytes
64 bytes from gameapp.local (127.0.0.1): icmp_seq=1 time=0.037 ms
```

**Frontend Response:**
```bash
HTTP/1.1 200 OK
Content-Type: text/html
```

**API Health Response:**
```json
{
  "status": "healthy",
  "services": {
    "database": "connected",
    "redis": "connected",
    "api": "running"
  },
  "timestamp": "2025-08-30T00:33:26.867Z"
}
```

## ✅ Checkpoint

Your Ingress setup is working when:
- ✅ Ingress controller pods are running in ingress-nginx namespace
- ✅ Ingress rules are configured for gameapp.local
- ✅ Frontend loads at `http://gameapp.local:8080` through Ingress
- ✅ Backend API responds to health checks through Ingress
- ✅ Ingress routes traffic correctly to both frontend and backend

## If It Fails

### Symptom: Ingress controller pods not starting
**Cause:** Resource constraints or image pull issues
**Command to confirm:** `kubectl get pods -n ingress-nginx`
**Fix:**
```bash
# Check ingress controller logs
kubectl logs -n ingress-nginx -l app.kubernetes.io/component=controller

# Check resource usage
kubectl top nodes

# If resources are low, use minimal cluster
k3d cluster delete dev-cluster
k3d cluster create dev-cluster --servers 1 --agents 1 --k3s-arg --disable=traefik@server:0
```

### Symptom: Ingress not accessible through k3d load balancer
**Cause:** Ingress controller not configured with hostPorts for k3d compatibility
**Command to confirm:** `curl -H "Host: gameapp.local" -I http://localhost:8080/`
**Fix:**
```bash
# Configure ingress controller to use hostPorts
kubectl patch deployment ingress-nginx-controller -n ingress-nginx -p '{"spec":{"template":{"spec":{"containers":[{"name":"controller","ports":[{"containerPort":80,"hostPort":80,"name":"http","protocol":"TCP"},{"containerPort":443,"hostPort":443,"name":"https","protocol":"TCP"},{"containerPort":8443,"name":"webhook","protocol":"TCP"}]}]}}}}'

# Wait for rollout
kubectl rollout status deployment/ingress-nginx-controller -n ingress-nginx

# Test again
curl -H "Host: gameapp.local" -I http://localhost:8080/
```

### Symptom: Can't open gameapp.local:8080 through the browser
**Cause:** cluster’s ingress (LoadBalancer / NodePort) listens on the WSL network (if using WSL Linux), not the Windows host network.
**Fix:**
```bash

# edit your Windows hosts file
C:\Windows\System32\drivers\etc\hosts

# add
127.0.0.1 gameapp.local
```




### Symptom: SSL certificate errors in ingress controller logs
**Cause:** Ingress configured with SSL but certificates don't exist
**Command to confirm:** `kubectl logs -n ingress-nginx -l app.kubernetes.io/component=controller | grep "SSL"`
**Fix:**
```bash
# Comment out SSL configurations in k8s/ingress.yaml
# Remove or comment: cert-manager.io/cluster-issuer, tls sections, ssl-redirect

# Apply updated ingress
kubectl apply -f k8s/ingress.yaml

# Check logs again
kubectl logs -n ingress-nginx -l app.kubernetes.io/component=controller | grep "SSL"
```

## 💡 **Reset/Rollback Commands**

If you need to start over or fix issues:

```bash
# Remove ingress rules
kubectl delete ingress humor-game-ingress -n humor-game

# Remove ingress controller (nuclear option)
kubectl delete namespace ingress-nginx

# Reset hosts file
sudo sed -i '/gameapp.local/d' /etc/hosts

# Restart ingress controller
kubectl rollout restart deployment/ingress-nginx-controller -n ingress-nginx

# Check ingress controller status
kubectl get pods -n ingress-nginx
kubectl logs -n ingress-nginx -l app.kubernetes.io/component=controller
```

## Understanding the URL Patterns

The documentation shows different URLs for different purposes:

- **`localhost:8080`** - For direct service testing and curl commands with Host headers
- **`gameapp.local:8080`** - For actual user access through the browser  
- **`Host: gameapp.local`** - For testing Ingress routing

This gives you **both development and production access patterns**:

- **Developers**: Use `localhost:8080` for direct testing and debugging
- **Users**: Access via `gameapp.local:8080` through Ingress (production-style)
- **DevOps Engineers**: Can test both patterns to verify routing works correctly

**Why both?** `localhost:8080` is the local port that k3d exposes, while `gameapp.local:8080` is the domain that Ingress routes to your services.

## What You Learned

You've implemented production networking with Ingress routing:
- **External access** to your Kubernetes application through domain names
- **Traffic routing** from Ingress controller to appropriate services
- **Production patterns** used by enterprise applications
- **Domain management** for both development and production environments
- **k3d compatibility** with hostPort configuration for ingress controllers

## Professional Skills Gained

- **Ingress controller setup** and configuration
- **Domain routing** and traffic management
- **Production networking** patterns
- **Service discovery** through external access points
- **k3d cluster integration** with ingress controllers

---

*Ingress milestone completed successfully. Application accessible via gameapp.local:8080, ready for [05-observability.md](05-observability.md).*
