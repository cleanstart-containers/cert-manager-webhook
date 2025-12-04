# Cert-Manager Webhook - CleanStart Container

The `cleanstart/cert-manager-webhook:latest-dev` image is a security-hardened, production-ready container image for cert-manager's Webhook server. This CleanStart image provides the cert-manager webhook functionality with enhanced security features, including non-root execution, dropped capabilities, and privilege escalation prevention, making it suitable for security-conscious Kubernetes deployments.

**📌 CleanStart Foundation:** Security-hardened, minimal base OS designed for enterprise containerized environments.

**Image Path:** `cleanstart/cert-manager-webhook`

**Registry:** CleanStart Registry

---

## Overview

The `cleanstart/cert-manager-webhook:latest-dev` image provides a drop-in replacement for the standard cert-manager webhook with security hardening applied. The Webhook is a validating and mutating admission webhook server that performs admission control for cert-manager resources, ensuring resource validation and mutation before they are persisted to the Kubernetes API server.

**Image:** `cleanstart/cert-manager-webhook:latest-dev`  
**Digest:** `sha256:d1bc72af8737c8087b7924ee996665fb8dc82bbba9075d392bcc1e59a8a0bc6a`

**Technical Specifications:**
- **Binary Location:** `/usr/bin/webhook`
- **User:** `clnstrt` (UID 1000) - CleanStart non-root user
- **Architecture:** `amd64`
- **OS:** `linux`

---

## Key Features

Core capabilities and strengths of this container:

- Validating admission webhook for cert-manager resources
- Mutating admission webhook for cert-manager resources
- Certificate resource validation and mutation
- CertificateRequest validation
- Issuer and ClusterIssuer validation
- Integration with Kubernetes admission control system
- Metrics endpoint for monitoring and observability (Prometheus compatible)
- Health check endpoints for liveness and readiness probes
- Secure TLS communication with Kubernetes API server
- Lightweight and efficient operation with minimal resource requirements
- Security-first design with minimal attack surface
- Enterprise compliance and regular security updates
- Multi-architecture support (AMD64 and ARM64)

---

## Common Use Cases

Typical scenarios where this container excels:

- Validating cert-manager Certificate resources before creation
- Mutating cert-manager resources with default values
- Enforcing cert-manager resource policies and constraints
- Integration with cert-manager controller for complete certificate management
- Production Kubernetes deployments requiring admission control
- Security-conscious environments requiring resource validation
- Multi-tenant clusters requiring policy enforcement
- Compliance and governance requirements

---

## What Cert-Manager Webhook Does

The Webhook operates as an admission controller that validates and mutates cert-manager resources:

1. **Validates Certificates**: Checks Certificate resources for correctness before they are created or updated
2. **Mutates Resources**: Adds default values and applies mutations to cert-manager resources
3. **Validates CertificateRequests**: Ensures CertificateRequest resources are properly configured
4. **Validates Issuers**: Verifies Issuer and ClusterIssuer configurations
5. **Enforces Policies**: Applies validation rules and constraints to cert-manager resources
6. **Admission Control**: Integrates with Kubernetes admission control system to intercept resource creation/updates
7. **TLS Communication**: Securely communicates with Kubernetes API server over TLS
8. **Health Monitoring**: Exposes health check endpoints for Kubernetes probes

---

## CleanStart Security Features

The `cleanstart/cert-manager-webhook:latest-dev` image implements multiple layers of security:

### CleanStart Security Enhancements

1. **Non-Root Execution with Dedicated User**: The image runs as the `clnstrt` user (UID 1000), a dedicated non-root user specifically created for CleanStart containers. This eliminates the risk of root privilege exploitation.

2. **Complete Capability Dropping**: All Linux capabilities are dropped at the container level, ensuring the webhook cannot perform privileged operations even if compromised.

3. **Privilege Escalation Prevention**: The image is configured with `allowPrivilegeEscalation: false`, preventing any process from escalating privileges through setuid binaries or other mechanisms.

4. **Pre-Configured SSL/TLS**: The image includes a complete SSL certificate bundle at `/etc/ssl/certs/ca-certificates.crt`, enabling secure communication with the Kubernetes API server without requiring additional configuration.

5. **Kubernetes Security Context Integration**: The deployment uses Kubernetes security contexts to enforce non-root execution and prevent privilege escalation at the pod level, providing defense in depth.

### Security Features Summary

- **Non-Root Execution**: Runs as dedicated `clnstrt` user (UID 1000), eliminating root privilege risks
- **Capability Dropping**: All Linux capabilities are dropped, preventing privileged operations
- **Privilege Escalation Prevention**: `allowPrivilegeEscalation: false` prevents privilege escalation attacks
- **Pre-Configured SSL/TLS**: Complete SSL certificate bundle pre-configured for secure communication
- **Kubernetes Security Context**: Pod-level security context enforces non-root execution and prevents privilege escalation
- **Defense in Depth**: Multiple security layers provide comprehensive protection
- **Least Privilege RBAC**: ClusterRole permissions follow the principle of least privilege

These security features make the CleanStart image suitable for production environments with strict security requirements, compliance needs, or security-sensitive workloads.

---

## Getting Started

### Pull Commands

Download the runtime container images:
```bash
docker pull cleanstart/cert-manager-webhook:latest
docker pull cleanstart/cert-manager-webhook:latest-dev
```

### Basic Run

Run the container with basic configuration:
```bash
docker run -it --name cert-manager-webhook-test cleanstart/cert-manager-webhook:latest-dev
```

### Production Deployment

Deploy with production security settings:
```bash
docker run -d --name cert-manager-webhook-prod \
  --read-only \
  --security-opt=no-new-privileges \
  --user 1000:1000 \
  cleanstart/cert-manager-webhook:latest
```

---

## Configuration

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `SSL_CERT_FILE` | `/etc/ssl/certs/ca-certificates.crt` | SSL certificate path for secure communication with Kubernetes API server |
| `PATH` | `/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin` | Standard PATH environment variable |
| `POD_NAME` | - | Automatically populated from Kubernetes pod metadata via downward API |
| `POD_NAMESPACE` | - | Automatically populated from Kubernetes pod metadata via downward API |

### Ports

- **6443** - Webhook admission endpoint (TLS) for receiving admission requests from Kubernetes API server
- **9402** - Metrics and health check endpoint (HTTP) for monitoring and probes

### RBAC Permissions

The Webhook requires cluster-scoped permissions to:

- Manage admission webhook configurations (ValidatingWebhookConfiguration, MutatingWebhookConfiguration)
- Create and manage CertificateSigningRequests (for TLS certificates)
- Read cert-manager resources (certificates, certificaterequests, issuers, clusterissuers)
- Read secrets (for TLS certificates)
- Read services, pods, and endpoints (for service discovery)
- Read CustomResourceDefinitions (to verify cert-manager CRDs are installed)

---

## Best Practices

- Deploy the Webhook with proper resource limits to ensure predictable resource usage
- Monitor webhook logs for admission request processing and errors
- Use health check endpoints for Kubernetes liveness and readiness probes
- Keep the container image updated with the latest security patches
- Implement proper network policies if required by your security policies
- Configure ValidatingWebhookConfiguration and MutatingWebhookConfiguration resources to register the webhook
- Ensure the webhook service is accessible from the Kubernetes API server
- Monitor webhook admission latency to ensure performance requirements are met
- Use proper TLS certificates for secure communication between API server and webhook

---

## Standard Webhook Functionality

The CleanStart image maintains full compatibility with standard cert-manager webhook features:

- **Admission Control**: Processes validating and mutating admission requests from Kubernetes API server
- **Resource Validation**: Validates cert-manager resources (Certificates, CertificateRequests, Issuers, ClusterIssuers) for correctness
- **Resource Mutation**: Applies default values and mutations to cert-manager resources
- **Kubernetes Integration**: Receives pod metadata (POD_NAME, POD_NAMESPACE) through downward API for proper logging and identification
- **Resource Management**: Deployed with conservative resource requests and limits for predictable resource usage
- **Health Monitoring**: Built-in health check endpoints enable Kubernetes liveness and readiness probes
- **Metrics Endpoint**: Exposes Prometheus metrics for monitoring and observability

The `cleanstart/cert-manager-webhook:latest-dev` image can be used as a drop-in replacement for the standard cert-manager webhook, providing the same functionality with enhanced security posture suitable for production environments with strict security requirements.

---

## Observability

### Logging

The Webhook provides structured logging that includes:

- Webhook startup and initialization events
- Admission request processing events
- Resource validation results
- Resource mutation operations
- Error conditions and failure reasons
- TLS certificate management events

### Metrics

The Webhook exposes a Prometheus metrics endpoint on port 9402, providing observability into:

- Admission request counts and success rates
- Admission request latency
- Resource validation statistics
- Resource mutation statistics
- Error rates and failure metrics
- Health check status

### Health Checks

Health check functionality allows Kubernetes to monitor webhook readiness:

- **Liveness Probe**: `/livez` endpoint on port 9402
- **Readiness Probe**: `/readyz` endpoint on port 9402

These endpoints enable proper integration with Kubernetes monitoring systems and ensure the webhook is ready to process admission requests.

---

## Architecture Support

CleanStart images support multiple architectures to ensure compatibility across different deployment environments:

- **AMD64**: Intel and AMD x86-64 processors
- **ARM64**: ARM-based processors including Apple Silicon and ARM servers

### Architecture-based Pull Commands
```bash
docker pull --platform linux/amd64 cleanstart/cert-manager-webhook:latest
docker pull --platform linux/arm64 cleanstart/cert-manager-webhook:latest
```

---

## Kubernetes Deployment

The `kubernetes/` directory contains a complete, production-ready Kubernetes deployment:

- `deployment.yaml` - Complete deployment manifest (Namespace, ServiceAccount, ClusterRole, ClusterRoleBinding, Deployment, Service)
- `README.md` - Comprehensive deployment guide with step-by-step instructions, testing procedures, and troubleshooting

See the `kubernetes/README.md` file for detailed deployment instructions and testing procedures.

---

## Resources

- **Official Documentation:** https://cert-manager.io/docs/
- **Provenance / SBOM / Signature:** https://images.cleanstart.com/images/cert-manager-webhook
- **Docker Hub:** https://hub.docker.com/r/cleanstart/cert-manager-webhook
- **CleanStart All Images:** https://images.cleanstart.com
- **CleanStart Community Images:** https://hub.docker.com/u/cleanstart

---

## Disclaimer & License

### Disclaimer

**Disclaimer:** This documentation is provided for informational purposes only. Users are responsible for ensuring compliance with applicable laws, regulations, and security requirements. CleanStart makes no warranties regarding the suitability of these images for specific use cases or environments.

### License

Apache-2.0

---

## Vulnerability Disclaimer

CleanStart offers Docker images that include third-party open-source libraries and packages maintained by independent contributors. While CleanStart maintains these images and applies industry-standard security practices, it cannot guarantee the security or integrity of upstream components beyond its control.

Users acknowledge and agree that open-source software may contain undiscovered vulnerabilities or introduce new risks through updates. CleanStart shall not be liable for security issues originating from third-party libraries, including but not limited to zero-day exploits, supply chain attacks, or contributor-introduced risks.

**Security remains a shared responsibility:** CleanStart provides updated images and guidance where possible, while users are responsible for evaluating deployments and implementing appropriate controls.

