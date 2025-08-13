# Kubernetes Cluster Service Template

A Backstage Software Template for provisioning Kubernetes clusters using Crossplane.

## Overview

This template creates the necessary Crossplane resources to provision a Kubernetes cluster in your cloud provider. It includes:

- **XRD (Composite Resource Definition)**: Defines the Cluster API
- **Composition**: Implements the cluster provisioning logic
- **Example Claim**: Shows how to request a Kubernetes cluster

## Features

- 🚀 **Multi-Cloud Support**: Deploy to AWS, Azure, GCP, or on-premises
- ⚙️ **Flexible Sizing**: Choose from small, medium, or large node sizes
- 🔢 **Scalable**: Support for 3 to 100 nodes
- 🔧 **Version Control**: Deploy specific Kubernetes versions
- 🔒 **Automatic Firewall**: Security rules configured automatically
- 📊 **Kubeconfig Access**: Connection details available in resource status

## Prerequisites

### Crossplane Installation

Ensure Crossplane is installed in your cluster:

```bash
kubectl create namespace crossplane-system
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm install crossplane --namespace crossplane-system crossplane-stable/crossplane
```

### Required Providers

This template requires the following Crossplane providers:

1. **Provider NOP** (for simulation/testing):
```bash
kubectl apply -f - <<EOF
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-nop
spec:
  package: xpkg.upbound.io/crossplane-contrib/provider-nop:v0.2.0
EOF
```

### Required Functions

Install the necessary Crossplane composition functions:

```bash
kubectl apply -f - <<EOF
apiVersion: pkg.crossplane.io/v1beta1
kind: Function
metadata:
  name: function-go-templating
spec:
  package: xpkg.upbound.io/crossplane-contrib/function-go-templating:v0.10.0
---
apiVersion: pkg.crossplane.io/v1beta1
kind: Function
metadata:
  name: function-auto-ready
spec:
  package: xpkg.upbound.io/crossplane-contrib/function-auto-ready:v0.2.1
---
apiVersion: pkg.crossplane.io/v1beta1
kind: Function
metadata:
  name: function-sequencer
spec:
  package: xpkg.upbound.io/crossplane-contrib/function-sequencer:v0.3.0
---
apiVersion: pkg.crossplane.io/v1beta1
kind: Function
metadata:
  name: function-environment-configs
spec:
  package: xpkg.upbound.io/crossplane-contrib/function-environment-configs:v0.1.0
EOF
```

## Usage

### Through Backstage

1. Navigate to the Software Catalog
2. Click "Create Component"
3. Select "Kubernetes Cluster"
4. Fill in the required parameters:
   - **Cluster Name**: Unique identifier for your cluster
   - **Namespace**: Kubernetes namespace (default: `default`)
   - **Node Size**: Small, medium, or large
   - **Node Count**: Number of nodes (3-100)
   - **Storage Size**: Storage per node in GB
   - **Kubernetes Version**: Select from available versions
   - **Cloud Provider**: AWS, Azure, GCP, or on-premises
   - **Owner**: Team or user who owns this resource

### Manual Deployment

1. Apply the XRD:
```bash
kubectl apply -f content/definition.yaml
```

2. Apply the Composition:
```bash
kubectl apply -f content/composition.yaml
```

3. Create a claim:
```bash
kubectl apply -f content/example.yaml
```

## Parameters

| Parameter | Description | Type | Default | Required |
|-----------|-------------|------|---------|----------|
| `name` | Cluster name | string | - | Yes |
| `namespace` | Kubernetes namespace | string | `default` | No |
| `nodeSize` | Size of nodes | string | `medium` | No |
| `nodeCount` | Number of nodes | integer | `3` | No |
| `storageSize` | Storage per node (GB) | integer | `10` | No |
| `version` | Kubernetes version | string | `1.31.1` | Yes |
| `cloudProvider` | Cloud provider | string | `azure` | No |
| `owner` | Resource owner | string | `group:platform` | No |

## Node Sizes

| Size | vCPU | Memory | Storage |
|------|------|--------|---------|
| Small | 2 | 4 GB | Configurable |
| Medium | 4 | 8 GB | Configurable |
| Large | 8 | 16 GB | Configurable |

## Architecture

The template creates a composite resource that provisions:

1. **ConfigMap**: Stores cluster configuration
2. **Cluster Resource**: The actual Kubernetes cluster (simulated)
3. **Firewall Rule**: Network security configuration
4. **NOP Resource**: Simulates cluster provisioning delay (30s)

## Accessing the Cluster

After provisioning, retrieve the kubeconfig:

```bash
# Get the base64-encoded kubeconfig
kubectl get cluster <cluster-name> -n <namespace> -o jsonpath='{.status.kubeconfig}' | base64 -d > kubeconfig.yaml

# Use the kubeconfig
export KUBECONFIG=kubeconfig.yaml
kubectl get nodes
```

Get the cluster IP address:

```bash
kubectl get cluster <cluster-name> -n <namespace> -o jsonpath='{.status.ipaddress}'
```

## Development

### Testing Locally

1. Clone this repository
2. Install the template in Backstage:
```yaml
catalog:
  locations:
    - type: url
      target: https://github.com/open-service-portal/service-cluster-template/blob/main/template.yaml
```

### Customization

To customize the cluster provisioning:

1. Edit `content/definition.yaml` to add parameters
2. Modify `content/composition.yaml` to change provisioning logic
3. Update `template.yaml` to expose new parameters in Backstage

## Supported Kubernetes Versions

- 1.31.1 (latest)
- 1.30.5
- 1.29.x series
- 1.28.x series

## Troubleshooting

### Common Issues

**Cluster stuck in Creating state**
- Check if all required functions are installed
- Verify the NOP provider is running: `kubectl get providers`
- Check composition status: `kubectl describe composition cluster`

**Kubeconfig not appearing**
- Ensure environment config is properly set
- Check if cluster resource reached Ready state
- Verify composition pipeline completed successfully

**Firewall rule errors**
- Check if FirewallRule XRD is installed
- Verify network policies allow rule creation

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Submit a pull request

## License

MIT License - see LICENSE file for details

## Support

For issues and questions:
- GitHub Issues: [open-service-portal/service-cluster-template](https://github.com/open-service-portal/service-cluster-template/issues)
- Discussions: [open-service-portal/discussions](https://github.com/orgs/open-service-portal/discussions)