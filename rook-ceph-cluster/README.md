# Rook Ceph Cluster Configuration

## ⚠️ IMPORTANT WARNINGS

### Single Replica Configuration
This cluster is configured with `replicated.size: 1` for all pools. This means:
- **Any disk failure will result in data loss**
- **No high availability**
- **Suitable only for development/testing**

### Before Production Use
When adding more nodes, update all pool configurations:

```yaml
replicated:
  size: 3  # Change from 1 to 3 for HA
  requireSafeReplicaSize: true
```

Also update monitor configuration:
```yaml
mon:
  count: 3  # Change from 1 to 3
  allowMultiplePerNode: false  # Change from true to false
```

## Device Discovery
- Uses `deviceFilter: ^sd[b-z]$` to avoid root disk (sda)
- Automatically discovers new devices on new nodes
- No manual configuration required for device mapping

## Storage Classes Created
1. **ceph-block** - Block storage (RBD)
2. **ceph-filesystem** - File system storage (CephFS)
3. **ceph-bucket** - Object storage (RGW)

## Prerequisites
- Kubernetes 1.28+ (required for Rook 1.17)
- Raw disk available (not partitioned)
- Rook Ceph Operator must be deployed first

## Deployment Order
1. **rook-ceph-operator** deploys first (sync-wave: 0)
2. **rook-ceph-cluster** deploys second (sync-wave: 1)

## Usage Examples

### Block Storage (RBD)
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-block-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: ceph-block
  resources:
    requests:
      storage: 1Gi
```

### File System Storage (CephFS)
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-filesystem-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: ceph-filesystem
  resources:
    requests:
      storage: 1Gi
```

### Object Storage (RGW)
```yaml
apiVersion: objectbucket.io/v1alpha1
kind: ObjectBucketClaim
metadata:
  name: test-bucket-claim
spec:
  generateBucketName: test-bucket
  storageClassName: ceph-bucket
```

## Monitoring and Debugging

### Check Cluster Status
```bash
# Check Ceph cluster health
kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- ceph status

# Check OSD status
kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- ceph osd status

# Check storage classes
kubectl get storageclass
```

### Access Ceph Dashboard
```bash
# Port forward to dashboard
kubectl -n rook-ceph port-forward svc/rook-ceph-mgr-dashboard 8443:8443
```

## Troubleshooting

### Device Discovery Issues
If devices are not being discovered:
1. Check if devices are partitioned: `lsblk -f`
2. Verify device filter regex: `deviceFilter: ^sd[b-z]$`
3. Check rook-discover pods: `kubectl -n rook-ceph get pods -l app=rook-discover`

### Storage Class Issues
If PVCs are stuck in Pending:
1. Check Ceph cluster health
2. Verify storage class parameters
3. Check CSI driver logs: `kubectl -n rook-ceph logs -l app=csi-rbdplugin`

## Scaling to Production

### Step 1: Add More Nodes
1. Add new nodes to your Kubernetes cluster
2. Ensure raw disks are available on new nodes
3. Rook will automatically discover and use new devices

### Step 2: Update Replica Count
1. Update all pool configurations to use `size: 3`
2. Update monitor count to `3`
3. Set `allowMultiplePerNode: false`

### Step 3: Enable Monitoring
1. Install Prometheus Operator
2. Set `monitoring.enabled: true` in values.yaml
3. Configure alerting rules

## Security Considerations

### Network Security
- Dashboard SSL is disabled by default
- Enable SSL before exposing dashboard externally
- Consider network policies for RGW access

### Access Control
- Toolbox runs as non-root user (2016)
- RBAC is enabled for operator
- Consider additional RBAC for application access

## Performance Tuning

### BlueStore Optimization
- Compression is enabled (`aggressive` mode)
- OSD memory target: 4GB
- Consider adjusting based on workload

### Network Optimization
- Host networking is disabled by default
- Enable for bare metal deployments if needed
- Consider dedicated network for Ceph traffic 