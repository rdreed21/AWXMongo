# Percona MongoDB Sharded Cluster - AWX/Ansible Automation

This repository contains Ansible playbooks for deploying and managing Percona MongoDB sharded clusters on OpenShift using AWX/Ansible Tower.

## Repository Structure

```
percona-mongodb-ansible/
├── ansible.cfg                    # Ansible configuration
├── inventory.yml                  # Ansible inventory
├── requirements.yml               # Required Ansible collections
├── deploy-percona-mongodb.yml     # Main deployment playbook
├── backup-mongodb.yml             # Manual backup playbook
├── restore-mongodb.yml            # Restore from backup playbook
├── delete-mongodb.yml             # Cleanup playbook
├── scale-mongodb.yml              # Scale cluster components
├── upgrade-mongodb.yml            # Upgrade MongoDB versions
├── setup-monitoring.yml           # Production monitoring setup
├── tune-mongodb.yml               # Performance tuning playbook
├── group_vars/
│   ├── dev.yml                   # Development environment variables
│   └── prod.yml                  # Production environment variables
└── README-AWX.md                 # This file
```

## Prerequisites

### AWX/Ansible Tower Requirements
- AWX 21.0+ or Ansible Tower 3.8+
- OpenShift cluster credentials configured
- Python 3.8+

### Required Collections
The following Ansible collections are required (defined in `requirements.yml`):
- `kubernetes.core` >= 2.4.0
- `community.general` >= 5.0.0

## AWX Setup Instructions

### 1. Create Project in AWX

1. Navigate to **Resources → Projects**
2. Click **Add**
3. Configure:
   - **Name**: `Percona MongoDB Deployment`
   - **Organization**: Select your organization
   - **Source Control Type**: `Git`
   - **Source Control URL**: Your Git repository URL
   - **Source Control Branch/Tag/Commit**: `main` (or your branch)
   - Check **Update Revision on Launch**
4. Click **Save**
5. Click **Sync** to pull the repository

### 2. Create Credentials

#### OpenShift/Kubernetes Credential
1. Navigate to **Resources → Credentials**
2. Click **Add**
3. Configure:
   - **Name**: `OpenShift Cluster`
   - **Credential Type**: `OpenShift or Kubernetes API Bearer Token`
   - **OpenShift or Kubernetes API Endpoint**: Your cluster API URL (e.g., `https://api.crc.testing:6443`)
   - **API Authentication Bearer Token**: Your token (get with `oc whoami -t`)
   - Check **Verify SSL** or uncheck for self-signed certs
4. Click **Save**

### 3. Create Inventory

1. Navigate to **Resources → Inventories**
2. Click **Add → Add inventory**
3. Configure:
   - **Name**: `Percona MongoDB Hosts`
   - **Organization**: Select your organization
4. Click **Save**
5. Go to **Sources** tab → **Add**
6. Configure:
   - **Name**: `Project Inventory`
   - **Source**: `Sourced from a Project`
   - **Project**: Select `Percona MongoDB Deployment`
   - **Inventory file**: `inventory.yml`
   - Check **Update on launch**
7. Click **Save**

### 4. Create Job Templates

#### Deploy MongoDB Cluster Template

1. Navigate to **Resources → Templates**
2. Click **Add → Add job template**
3. Configure:
   - **Name**: `Deploy Percona MongoDB Cluster`
   - **Job Type**: `Run`
   - **Inventory**: `Percona MongoDB Hosts`
   - **Project**: `Percona MongoDB Deployment`
   - **Playbook**: `deploy-percona-mongodb.yml`
   - **Credentials**: Select `OpenShift Cluster`
   - **Variables**: (See Variables section below)
   - Check **Prompt on launch** for: Variables
4. Click **Save**

#### Backup MongoDB Template

1. Click **Add → Add job template**
2. Configure:
   - **Name**: `Backup MongoDB Cluster`
   - **Job Type**: `Run`
   - **Inventory**: `Percona MongoDB Hosts`
   - **Project**: `Percona MongoDB Deployment`
   - **Playbook**: `backup-mongodb.yml`
   - **Credentials**: Select `OpenShift Cluster`
   - Check **Prompt on launch** for: Variables
3. Click **Save**

#### Restore MongoDB Template

1. Click **Add → Add job template**
2. Configure:
   - **Name**: `Restore MongoDB Cluster`
   - **Job Type**: `Run`
   - **Inventory**: `Percona MongoDB Hosts`
   - **Project**: `Percona MongoDB Deployment`
   - **Playbook**: `restore-mongodb.yml`
   - **Credentials**: Select `OpenShift Cluster`
   - Check **Prompt on launch** for: Variables
3. Click **Save**

#### Delete MongoDB Template

1. Click **Add → Add job template**
2. Configure:
   - **Name**: `Delete MongoDB Cluster`
   - **Job Type**: `Run`
   - **Inventory**: `Percona MongoDB Hosts`
   - **Project**: `Percona MongoDB Deployment`
   - **Playbook**: `delete-mongodb.yml`
   - **Credentials**: Select `OpenShift Cluster`
   - Check **Prompt on launch** for: Variables
   - Check **Enable Fact Storage**
   - Check **Ask variables on launch**
3. Click **Save**

#### Scale MongoDB Template

1. Click **Add → Add job template**
2. Configure:
   - **Name**: `Scale MongoDB Cluster`
   - **Job Type**: `Run`
   - **Inventory**: `Percona MongoDB Hosts`
   - **Project**: `Percona MongoDB Deployment`
   - **Playbook**: `scale-mongodb.yml`
   - **Credentials**: Select `OpenShift Cluster`
   - Check **Prompt on launch** for: Variables
3. Click **Save**

#### Upgrade MongoDB Template

1. Click **Add → Add job template**
2. Configure:
   - **Name**: `Upgrade MongoDB Cluster`
   - **Job Type**: `Run`
   - **Inventory**: `Percona MongoDB Hosts`
   - **Project**: `Percona MongoDB Deployment`
   - **Playbook**: `upgrade-mongodb.yml`
   - **Credentials**: Select `OpenShift Cluster`
   - Check **Prompt on launch** for: Variables
   - Check **Enable Privilege Escalation**
3. Click **Save**

#### Setup Production Monitoring Template

1. Click **Add → Add job template**
2. Configure:
   - **Name**: `Setup MongoDB Monitoring`
   - **Job Type**: `Run`
   - **Inventory**: `Percona MongoDB Hosts`
   - **Project**: `Percona MongoDB Deployment`
   - **Playbook**: `setup-monitoring.yml`
   - **Credentials**: Select `OpenShift Cluster`
   - Check **Prompt on launch** for: Variables
   - **Limit**: Only allow on production environments
3. Click **Save**

#### Performance Tuning Template

1. Click **Add → Add job template**
2. Configure:
   - **Name**: `Tune MongoDB Performance`
   - **Job Type**: `Run`
   - **Inventory**: `Percona MongoDB Hosts`
   - **Project**: `Percona MongoDB Deployment`
   - **Playbook**: `tune-mongodb.yml`
   - **Credentials**: Select `OpenShift Cluster`
   - Check **Prompt on launch** for: Variables
   - **Survey** (optional): Add dropdown for tuning_profile
3. Click **Save**

## Variables

### Deployment Variables (deploy-percona-mongodb.yml)

#### Development Environment
Use the provided `group_vars/dev.yml` or override with these in AWX:

```yaml
mongodb_namespace: percona-mongo-dev
cluster_name: mongo-cluster-dev
config_server_size: 1
mongos_size: 1
shard_replica_size: 1
deploy_minio: true
enable_backups: true
```

#### Production Environment
Use the provided `group_vars/prod.yml` or override:

```yaml
mongodb_namespace: percona-mongo-prod
cluster_name: mongo-cluster-prod
config_server_size: 3
mongos_size: 3
shard_replica_size: 3
deploy_minio: true
enable_backups: true
enable_pitr: true
```

### Backup Variables (backup-mongodb.yml)

```yaml
mongodb_namespace: percona-mongo
cluster_name: mongo-cluster
backup_storage: filesystem-local  # or minio-local
```

### Restore Variables (restore-mongodb.yml)

```yaml
mongodb_namespace: percona-mongo
cluster_name: mongo-cluster
backup_name: "daily-backup-mongo-cluster-20260116020000"
pitr_enabled: false
# pitr_date: "2026-01-16T14:30:00Z"  # Only if pitr_enabled: true
```

### Delete Variables (delete-mongodb.yml)

```yaml
mongodb_namespace: percona-mongo
cluster_name: mongo-cluster
delete_namespace: false  # Set to true to delete entire namespace
delete_pvcs: false       # Set to true to delete persistent volumes
```

### Scale Variables (scale-mongodb.yml)

```yaml
mongodb_namespace: percona-mongo
cluster_name: mongo-cluster
config_server_size: 3    # Target size for config servers
mongos_size: 3           # Target size for mongos routers
shard_replica_size: 3    # Target replica size per shard

# Advanced: Add a new shard
add_new_shard: false
new_shard_name: "rs3"
new_shard_replica_size: 3
```

### Upgrade Variables (upgrade-mongodb.yml)

```yaml
mongodb_namespace: percona-mongo
cluster_name: mongo-cluster
target_operator_version: "1.21.1"
target_mongodb_version: "8.0.12-4-multi"
target_backup_version: "2.11.0-multi"
require_backup_before_upgrade: true
upgrade_strategy: SmartUpdate  # SmartUpdate, RollingUpdate, or OnDelete

# Optional maintenance window
maintenance_window: false
maintenance_start: "2026-01-20T02:00:00Z"
maintenance_end: "2026-01-20T04:00:00Z"
```

### Monitoring Setup Variables (setup-monitoring.yml)

```yaml
mongodb_namespace: percona-mongo-prod
cluster_name: mongo-cluster-prod

# PMM Configuration
enable_pmm: true
pmm_server_url: "https://pmm.example.com"
pmm_api_key: "YOUR_PMM_API_KEY"

# Prometheus Integration
enable_servicemonitor: true
prometheus_namespace: openshift-monitoring

# Grafana Dashboard
enable_grafana_dashboard: true
grafana_namespace: openshift-monitoring

# Alert Rules
enable_alerts: true
```

### Performance Tuning Variables (tune-mongodb.yml)

```yaml
mongodb_namespace: percona-mongo
cluster_name: mongo-cluster

# Tuning Profile (balanced, throughput, latency, memory_optimized)
tuning_profile: balanced

# Cache Configuration
wiredtiger_cache_size_ratio: 0.5  # 50% of pod memory

# Connection Settings
max_connections: 65536

# OpLog Settings
oplog_size_mb: 2048

# Profiling
enable_profiling: true
profiling_level: 1  # 0=off, 1=slow, 2=all
profiling_slow_ms: 100

# Sharding Optimization
optimize_chunk_size: true
chunk_size_mb: 64

# Balancer Window
balancer_active_window_start: "01:00"
balancer_active_window_end: "05:00"

# Resource Adjustment (optional)
adjust_resources: false
new_cpu_limit: "4000m"
new_memory_limit: "8Gi"
```

## Running Playbooks from AWX

### Deploy a Cluster

1. Go to **Resources → Templates**
2. Click the **Launch** button for `Deploy Percona MongoDB Cluster`
3. If prompted, provide/modify variables
4. Click **Next** → **Launch**
5. Monitor the job execution

### Create a Manual Backup

1. Launch the `Backup MongoDB Cluster` template
2. Provide variables if needed
3. Click **Launch**

### Restore from Backup

1. First, list available backups:
   ```bash
   oc get psmdb-backup -n percona-mongo
   ```
2. Launch the `Restore MongoDB Cluster` template
3. Provide the `backup_name` variable
4. Click **Launch**

### Scale the Cluster

1. Launch the `Scale MongoDB Cluster` template
2. Set target sizes:
   - `config_server_size: 5`
   - `mongos_size: 5`
   - `shard_replica_size: 5`
3. Or add a new shard:
   - `add_new_shard: true`
   - `new_shard_name: rs3`
4. Click **Launch**

### Upgrade MongoDB

1. Launch the `Upgrade MongoDB Cluster` template
2. Set target versions:
   - `target_mongodb_version: "8.0.14-5-multi"`
   - `require_backup_before_upgrade: true`
3. Optional: Set maintenance window
4. Click **Launch**
5. Monitor upgrade progress carefully

### Setup Production Monitoring

1. Ensure you have PMM server deployed (optional)
2. Launch the `Setup MongoDB Monitoring` template
3. Provide PMM credentials if using PMM
4. Configure Prometheus/Grafana settings
5. Click **Launch**

### Performance Tuning

1. Launch the `Tune MongoDB Performance` template
2. Select tuning profile:
   - **balanced**: General purpose workloads
   - **throughput**: High-volume writes, batch processing
   - **latency**: Low-latency reads, user-facing apps
   - **memory_optimized**: Resource-constrained environments
3. Configure additional settings:
   - Enable profiling for slow query analysis
   - Adjust chunk size for sharding optimization
   - Set balancer window for off-peak hours
4. Click **Launch**
5. Monitor results in generated ConfigMap

### Delete a Cluster

1. Launch the `Delete MongoDB Cluster` template
2. Set variables:
   - `delete_pvcs: true` to delete data
   - `delete_namespace: true` to delete namespace
3. Click **Launch**
4. Confirm the deletion in the playbook prompt

## Running Playbooks via CLI

If you prefer to run outside of AWX:

### Install Requirements
```bash
ansible-galaxy collection install -r requirements.yml
```

### Deploy Cluster
```bash
# Development environment
ansible-playbook deploy-percona-mongodb.yml -e @group_vars/dev.yml

# Production environment
ansible-playbook deploy-percona-mongodb.yml -e @group_vars/prod.yml

# Custom variables
ansible-playbook deploy-percona-mongodb.yml \
  -e mongodb_namespace=my-mongo \
  -e cluster_name=my-cluster
```

### Create Backup
```bash
ansible-playbook backup-mongodb.yml \
  -e mongodb_namespace=percona-mongo \
  -e cluster_name=mongo-cluster
```

### Restore from Backup
```bash
ansible-playbook restore-mongodb.yml \
  -e mongodb_namespace=percona-mongo \
  -e cluster_name=mongo-cluster \
  -e backup_name=daily-backup-mongo-cluster-20260116020000
```

### Delete Cluster
```bash
ansible-playbook delete-mongodb.yml \
  -e mongodb_namespace=percona-mongo \
  -e cluster_name=mongo-cluster \
  -e delete_pvcs=true \
  -e delete_namespace=false
```

### Scale Cluster
```bash
# Scale up to 5 replicas per shard
ansible-playbook scale-mongodb.yml \
  -e mongodb_namespace=percona-mongo \
  -e cluster_name=mongo-cluster \
  -e shard_replica_size=5

# Add a new shard
ansible-playbook scale-mongodb.yml \
  -e mongodb_namespace=percona-mongo \
  -e cluster_name=mongo-cluster \
  -e add_new_shard=true \
  -e new_shard_name=rs3 \
  -e new_shard_replica_size=3
```

### Upgrade Cluster
```bash
ansible-playbook upgrade-mongodb.yml \
  -e mongodb_namespace=percona-mongo \
  -e cluster_name=mongo-cluster \
  -e target_mongodb_version="8.0.14-5-multi" \
  -e require_backup_before_upgrade=true
```

### Setup Production Monitoring
```bash
ansible-playbook setup-monitoring.yml \
  -e mongodb_namespace=percona-mongo-prod \
  -e cluster_name=mongo-cluster-prod \
  -e enable_pmm=true \
  -e pmm_server_url="https://pmm.example.com" \
  -e pmm_api_key="YOUR_API_KEY"
```

### Performance Tuning
```bash
# Apply balanced profile
ansible-playbook tune-mongodb.yml \
  -e mongodb_namespace=percona-mongo \
  -e cluster_name=mongo-cluster \
  -e tuning_profile=balanced

# High throughput optimization
ansible-playbook tune-mongodb.yml \
  -e tuning_profile=throughput \
  -e oplog_size_mb=4096 \
  -e enable_profiling=true

# Low latency optimization
ansible-playbook tune-mongodb.yml \
  -e tuning_profile=latency \
  -e profiling_slow_ms=50
```

## Security Best Practices

### Using Ansible Vault for Secrets

For production, store sensitive variables in Ansible Vault:

1. Create a vault file:
```bash
ansible-vault create group_vars/prod_vault.yml
```

2. Add sensitive variables:
```yaml
vault_mongodb_backup_password: "SecurePassword123!"
vault_mongodb_cluster_admin_password: "SecureAdmin456!"
vault_mongodb_cluster_monitor_password: "SecureMonitor789!"
vault_mongodb_user_admin_password: "SecureUser012!"
vault_minio_access_key: "SecureMinioKey345"
vault_minio_secret_key: "SecureMinioSecret678"
```

3. Reference in `group_vars/prod.yml`:
```yaml
mongodb_backup_password: "{{ vault_mongodb_backup_password }}"
mongodb_cluster_admin_password: "{{ vault_mongodb_cluster_admin_password }}"
# ... etc
```

4. In AWX, create a **Credential Type** for the vault password:
   - Navigate to **Administration → Credential Types**
   - Click **Add**
   - Create custom credential type for Ansible Vault

5. Add vault credential to your job template

## Monitoring Deployments

### Check Cluster Status
```bash
oc get psmdb -n percona-mongo
oc get pods -n percona-mongo
```

### Check Backup Status
```bash
oc get psmdb-backup -n percona-mongo
oc describe psmdb-backup <backup-name> -n percona-mongo
```

### View Logs
```bash
# Operator logs
oc logs -n percona-mongo -l app.kubernetes.io/name=percona-server-mongodb-operator

# MongoDB pod logs
oc logs -n percona-mongo mongo-cluster-rs0-0
```

### Connect to MongoDB
```bash
# Port forward
oc port-forward -n percona-mongo svc/mongo-cluster-mongos 27017:27017

# Connect with mongosh
mongosh "mongodb://clusterAdmin:PASSWORD@localhost:27017/admin"
```

## Troubleshooting

### Playbook Fails During Operator Installation
- Check that the certified-operators catalog is available
- Verify network connectivity from AWX to the cluster
- Check operator logs: `oc logs -n percona-mongo -l app.kubernetes.io/name=percona-server-mongodb-operator`

### Cluster Not Reaching Ready State
- Verify sufficient resources in the cluster
- Check pod status: `oc get pods -n percona-mongo`
- Check pod events: `oc describe pod <pod-name> -n percona-mongo`
- Review cluster status: `oc describe psmdb mongo-cluster -n percona-mongo`

### Backup Fails
- Verify storage backend is available (MinIO or filesystem)
- Check PVC status: `oc get pvc -n percona-mongo`
- Review backup logs: `oc logs -n percona-mongo -l app.kubernetes.io/component=backup`

### Scaling Issues
- Ensure sufficient cluster resources before scaling up
- Check pod events: `oc describe pod <pod-name> -n percona-mongo`
- For new shards, manually add to cluster: `sh.addShard('rs3/...')`
- Verify data rebalancing: `sh.status()`

### Upgrade Fails or Hangs
- Check operator logs for upgrade status
- Verify all pods are in Running state before upgrade
- Review MongoDB compatibility matrix
- Check for sufficient storage and resources
- Rollback if needed: patch cluster with previous versions

### Monitoring Not Working
- Verify Prometheus Operator is installed
- Check ServiceMonitor is created: `oc get servicemonitor -n percona-mongo`
- Verify metrics endpoint: `curl http://pod-ip:9216/metrics`
- Check Grafana can access Prometheus data source
- Review alert rules: `oc get prometheusrule -n percona-mongo`

### Performance Issues After Tuning
- Review tuning recommendations ConfigMap: `oc get configmap <cluster>-tuning-recommendations`
- Check slow query logs: `db.system.profile.find().sort({ts:-1})`
- Verify WiredTiger cache hit ratio: `db.serverStatus().wiredTiger.cache`
- Monitor connection usage: `db.serverStatus().connections`
- Review balancer activity during peak hours
- Consider different tuning profile for workload type

### AWX Job Hangs
- Check AWX execution environment has required Python packages
- Verify kubernetes.core collection is installed
- Check AWX → Jobs → <job> → Output for detailed errors

## Architecture Overview

The deployment creates:
- **3 Config Server nodes** (or 1 in dev)
- **3 Mongos routers** (or 1 in dev)
- **3 Shards** with **3 replicas each** (or 1 in dev)
- **Optional MinIO** for S3-compatible backup storage
- **Automated backup schedules**
- **Point-in-Time Recovery** (production only)

## Complete Operations Suite

### Available Playbooks

1. **deploy-percona-mongodb.yml** - Deploy full sharded cluster
   - Installs Percona operator
   - Creates namespace and secrets
   - Deploys config servers, mongos, and shards
   - Optional MinIO for backups
   - Configurable for dev/prod environments

2. **backup-mongodb.yml** - Create manual backups
   - On-demand backup creation
   - Filesystem or S3 storage
   - Compression support
   - Status monitoring

3. **restore-mongodb.yml** - Restore from backup
   - Restore from named backup
   - Point-in-time recovery support
   - Progress monitoring
   - Verification steps

4. **scale-mongodb.yml** - Scale cluster components
   - Scale config servers (1-5 nodes)
   - Scale mongos routers (1-5 nodes)
   - Scale shard replicas (1-5+ per shard)
   - Add new shards dynamically
   - Data rebalancing support

5. **upgrade-mongodb.yml** - Upgrade MongoDB versions
   - Safe version upgrades
   - Pre-upgrade backup (mandatory)
   - Version compatibility checks
   - Maintenance window support
   - Rolling update strategy
   - Rollback guidance

6. **setup-monitoring.yml** - Production monitoring (PROD ONLY)
   - PMM integration
   - Prometheus ServiceMonitor
   - Grafana dashboards
   - Alert rules (critical & warning)
   - Network policies
   - Health checks

7. **tune-mongodb.yml** - Performance optimization
   - Pre-built tuning profiles (balanced, throughput, latency, memory_optimized)
   - WiredTiger cache optimization
   - Connection pool tuning
   - Query profiling
   - Sharding optimization
   - Resource adjustment
   - Performance analysis

8. **delete-mongodb.yml** - Safe cluster removal
   - Confirmation prompts
   - Optional PVC deletion
   - Optional namespace deletion
   - MinIO cleanup

### Tuning Profiles

| Profile | Best For | Cache | Connections | Journal |
|---------|----------|-------|-------------|---------|
| **balanced** | General workloads | 50% | 65K | 100ms |
| **throughput** | High writes, ETL | 60% | 100K | 50ms |
| **latency** | Real-time queries | 70% | 50K | 100ms |
| **memory_optimized** | Limited resources | 30% | 10K | 200ms |

## Support and Contributions

For issues or questions:
1. Check the playbook output in AWX
2. Review OpenShift logs
3. Consult Percona MongoDB Operator documentation

## License

This project is provided as-is for deploying Percona MongoDB clusters.