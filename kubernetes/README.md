# Cert-Manager Webhook - Kubernetes Deployment

This directory contains Kubernetes manifests to deploy and test the `cleanstart/cert-manager-webhook:latest-dev` image.

## Overview

The cert-manager webhook is a validating and mutating webhook server that performs admission control for cert-manager resources. This deployment allows you to test the CleanStart cert-manager-webhook container image in a Kubernetes environment.

## What This Deployment Includes

- **Namespace**: `cert-manager-webhook` - Isolated environment for testing
- **ServiceAccount**: `webhook-sa` - Identity for the pod
- **ClusterRole & ClusterRoleBinding**: Permissions to manage admission webhook configurations and cert-manager resources
- **Deployment**: Runs the webhook container to handle admission requests
- **Service**: Exposes the webhook service on port 443 for admission requests and port 9402 for metrics

## Prerequisites

- Kubernetes cluster (v1.19+)
- `kubectl` configured to access your cluster
- **cert-manager CRDs installed** in your cluster (required for the webhook to function)
- RBAC permissions to create namespaces, deployments, services, service accounts, cluster roles, and cluster role bindings

## Deployment Steps

### Step 1: Verify cert-manager CRDs are Installed

Before deploying, ensure cert-manager CRDs are installed in your cluster:

```bash
# Check if cert-manager CRDs are installed
kubectl get crd | grep cert-manager.io

# Verify key CRDs exist
kubectl get crd certificates.cert-manager.io
kubectl get crd certificaterequests.cert-manager.io
kubectl get crd issuers.cert-manager.io
kubectl get crd clusterissuers.cert-manager.io
```

If cert-manager CRDs are not installed, install them first:

```bash
# Install cert-manager CRDs (example for v1.13.0)
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.crds.yaml

# Verify CRDs are installed
kubectl get crd | grep cert-manager.io
```

### Step 2: Deploy the Webhook

Apply the deployment manifest:

```bash
kubectl apply -f deployment.yaml
```

### Step 3: Verify Deployment

Check that the deployment was created successfully:

```bash
# Check namespace
kubectl get namespace cert-manager-webhook

# Check deployment status
kubectl get deployment -n cert-manager-webhook -w

# Check pod status
kubectl get pods -n cert-manager-webhook -w

# Check service
kubectl get service -n cert-manager-webhook
```

### Step 4: Check Pod Logs

View the logs to see the webhook server output:

```bash
# Get pod name
POD_NAME=$(kubectl get pods -n cert-manager-webhook -l app=cert-manager-webhook -o jsonpath='{.items[0].metadata.name}')

# View logs
kubectl logs -n cert-manager-webhook $POD_NAME -f
```

### Step 5: Verify Functionality

The webhook should:
- Start successfully and listen on port 6443 for admission requests
- Expose metrics endpoint on port 9402
- Respond to health check probes on /livez and /readyz
- Successfully validate cert-manager resources when configured

Check the pod status:

```bash
# Check pod status
kubectl get pods -n cert-manager-webhook -l app=cert-manager-webhook

# Describe pod for detailed status
kubectl describe pod -n cert-manager-webhook -l app=cert-manager-webhook
```

**Success Indicator:** The pod should be in `Running` state with `READY 1/1`. 

**Note:** With health probes temporarily disabled for initial startup, the pod will show `1/1 Running` even while generating certificates. Once the CA secret is created and certificates are generated, you can re-enable the health probes.

**Note on Initial Startup:** During the first startup, you may see log messages like:
```
E1115 12:33:33.780817       1 dynamic_source.go:221] "Failed to generate serving certificate, retrying..." err="no tls.Certificate available yet, try again later"
```

This is **expected behavior** - the webhook uses dynamic certificate serving and needs to:
1. Create the CA secret (`cert-manager-webhook-ca`) in the namespace
2. Generate the serving certificate from the CA
3. This process typically completes within 30-60 seconds

The webhook will automatically retry until the certificate is ready. Once the CA secret is created and the serving certificate is generated, the webhook will be fully operational.

**Important:** The deployment is configured with health probes temporarily disabled to allow the webhook to complete secret creation during initial startup. After the webhook is running successfully and the secret is created, you should re-enable the probes by uncommenting them in the deployment.yaml file.

### Step 6: Test Webhook Connectivity

Test that the webhook service is accessible:

```bash
# Check service endpoints
kubectl get endpoints -n cert-manager-webhook cert-manager-webhook

# Test metrics endpoint
kubectl run -it --rm --image=curlimages/curl:latest --restart=Never curl-test -- \
  curl http://cert-manager-webhook.cert-manager-webhook.svc:9402/metrics
```

## Cleanup

To remove all resources created by this deployment:

```bash
kubectl delete -f deployment.yaml
```

Or delete the namespace (this will remove all resources):

```bash
kubectl delete namespace cert-manager-webhook
```

## Notes

- This deployment is for testing the CleanStart cert-manager-webhook image functionality
- For production use, you should configure proper ValidatingWebhookConfiguration and MutatingWebhookConfiguration resources to register the webhook with the Kubernetes API server
- The webhook requires cert-manager CRDs to be installed in the cluster
- Ensure proper network policies are configured if required by your security policies
- The webhook service must be accessible from the Kubernetes API server for admission requests to work

