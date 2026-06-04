# ocp4_workload_tenant_gpu_machineset

Creates a tenant-specific GPU machineset for on-demand GPU resource provisioning.

## Purpose

This workload provisions a dedicated GPU worker node for a tenant when they order their lab environment. The GPU machineset is created dynamically and removed when the tenant environment is destroyed, enabling cost-effective GPU resource allocation.

## Features

- Creates AWS GPU machineset (g6.xlarge/2xlarge/4xlarge) per tenant
- Configurable GPU instance type selection
- Waits for GPU node to join cluster and be ready
- Labels nodes with tenant identifier for tracking
- Applies GPU taints to prevent non-GPU workloads
- Automatic cleanup on tenant destruction

## Requirements

- OpenShift cluster on AWS
- Cluster-admin service account credentials
- NVIDIA GPU Operator installed on cluster
- AWS GPU quota (pgpu reservation)

## Variables

### Required Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `sandbox_openshift_api_url` | Cluster API URL | `https://api.cluster.example.com:6443` |
| `cluster_admin_agnosticd_sa_token` | Cluster admin SA token | `sha256~...` |
| `guid` | Tenant GUID | `a1b2c3` |

### Optional Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `ocp4_workload_tenant_gpu_machineset_instance_type` | `g6.xlarge` | AWS GPU instance type |
| `ocp4_workload_tenant_gpu_machineset_replicas` | `1` | Number of GPU nodes |
| `ocp4_workload_tenant_gpu_machineset_root_volume_size` | `250` | Root volume size (GB) |
| `ocp4_workload_tenant_gpu_machineset_wait_for_nodes` | `true` | Wait for nodes to be ready |

## Example Usage

```yaml
workloads:
  - agnosticd.namespaced_workloads.ocp4_workload_tenant_gpu_machineset

ocp4_workload_tenant_gpu_machineset_instance_type: g6.2xlarge
ocp4_workload_tenant_gpu_machineset_replicas: 1
```

## Provisioning Flow

1. Discovers cluster infrastructure (region, AZ, VPC, etc.)
2. Uses existing worker machineset as template
3. Creates GPU machineset with tenant labels
4. Waits for GPU machine to provision (~5-10 minutes)
5. Waits for node to join cluster and be Ready
6. Verifies GPU resources are available

## Removal Flow

1. Deletes GPU machineset
2. Waits for machines to terminate
3. AWS automatically cleans up EC2 instances

## Node Labels

GPU nodes are labeled with:
- `node-role.kubernetes.io/worker-gpu=""`
- `tenant={{ username }}`

## Node Taints

GPU nodes are tainted to prevent non-GPU workloads:
- `nvidia.com/gpu=true:NoSchedule`

## GPU Workload Requirements

**This role only provisions the GPU node.** GPU workloads (InferenceService, Notebook, etc.) **must** include:

### 1. Toleration (to overcome taint):
```yaml
tolerations:
  - effect: NoSchedule
    key: nvidia.com/gpu
    operator: Exists
```

### 2. Node Selector (for tenant isolation):
```yaml
nodeSelector:
  tenant: user-{guid}  # Match the tenant's username
```

### 3. GPU Resource Request:
```yaml
resources:
  limits:
    nvidia.com/gpu: 1
```

### Example: KServe InferenceService

```yaml
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: my-model
  namespace: user-abc123-dsproject
spec:
  predictor:
    model:
      resources:
        limits:
          nvidia.com/gpu: '1'
    nodeSelector:
      tenant: user-abc123
    tolerations:
      - effect: NoSchedule
        key: nvidia.com/gpu
        operator: Exists
```

**Without the toleration:** Pod stays pending (blocked by taint)  
**Without the nodeSelector:** Pod could schedule on another tenant's GPU node  
**Without GPU request:** Pod schedules on regular workers (no GPU access)

## Author

Tyrell Reddy (treddy@redhat.com)
