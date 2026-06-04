# ocp4_workload_tenant_machineset

Creates tenant-specific machinesets for on-demand resource provisioning.

## Purpose

This workload provisions dedicated worker nodes for a tenant when they order their lab environment. Machinesets are created dynamically and removed when the tenant environment is destroyed, enabling cost-effective resource allocation.

## Features

- Creates AWS machinesets (any instance type) per tenant
- Supports multiple machineset groups per tenant
- Configurable instance types, taints, and node roles
- Waits for nodes to join cluster and be ready
- Labels nodes with tenant identifier for tracking
- Automatic cleanup on tenant destruction
- Flexible configuration matching core_workloads.ocp4_workload_machinesets pattern

## Requirements

- OpenShift cluster on AWS
- Cluster-admin service account credentials
- AWS quota for requested instance types

## Variables

See defaults/main.yml for full variable documentation.

**Key variable:** `ocp4_workload_tenant_machineset_groups` - list of machineset configurations

## Example Usage

### GPU Machineset

```yaml
workloads:
  - agnosticd.namespaced_workloads.ocp4_workload_tenant_machineset

ocp4_workload_tenant_machineset_groups:
  - name: gpu
    instance_type: g6.xlarge
    replicas: 1
    role: worker-gpu
    taints:
      - key: nvidia.com/gpu
        value: "true"
        effect: NoSchedule
    root_volume_size: 250
```

## Author

Tyrell Reddy (treddy@redhat.com)
