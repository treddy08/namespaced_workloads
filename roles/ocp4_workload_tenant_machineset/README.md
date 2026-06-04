# ocp4_workload_tenant_machineset

Tenant-specific machineset provisioning using the proven `ocp4_workload_machinesets` logic.

## Purpose

This role is a copy of `agnosticd.core_workloads.ocp4_workload_machinesets` adapted for tenant use. It provisions dedicated worker nodes for a tenant when they order their lab environment.

## Key Differences from Cluster Role

- **Usage**: Tenant-level (namespace config) vs cluster-level
- **Naming**: Machinesets named with tenant GUID
- **Removal**: Implements proper cleanup in remove_workload.yml
- **Labels**: Adds tenant labels to nodes for isolation

## Variables

Uses the same variable structure as the original role:

```yaml
ocp4_workload_machinesets_machineset_groups:
  - name: "gpu-{{ guid }}"
    autoscale: false
    total_replicas: 1
    role: worker-gpu
    taints:
      - key: nvidia.com/gpu
        value: reserved
        effect: NoSchedule
    node_labels:
      tenant: "user-{{ guid }}"
    instance_type: g6.xlarge
    root_volume_size: 250
```

See `agnosticd.core_workloads.ocp4_workload_machinesets` for full documentation.

## Author

Based on `ocp4_workload_machinesets` by Wolfgang Kulhanek  
Adapted for tenant use by Tyrell Reddy (treddy@redhat.com)
