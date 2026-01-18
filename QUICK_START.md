# Percona MongoDB Ansible - Quick Start Guide

## Prerequisites

1. OpenShift cluster access
2. `oc` CLI installed and logged in
3. Ansible installed (2.9+)
4. Python 3.8+

## Installation

```bash
# Install required collections
ansible-galaxy collection install -r requirements.yml
```

## Quick Deploy

### Development Environment
```bash
ansible-playbook deploy-percona-mongodb.yml -e @group_vars/dev.yml
```

### Production Environment
```bash
ansible-playbook deploy-percona-mongodb.yml -e @group_vars/prod.yml
```

## Common Operations

### Backup
```bash
ansible-playbook backup-mongodb.yml
```

### Restore
```bash
ansible-playbook restore-mongodb.yml -e backup_name=<backup-name>
```

### Scale
```bash
ansible-playbook scale-mongodb.yml -e shard_replica_size=5
```

### Upgrade
```bash
ansible-playbook upgrade-mongodb.yml -e target_mongodb_version="8.0.14-5-multi"
```

### Performance Tuning
```bash
# Balanced profile
ansible-playbook tune-mongodb.yml -e tuning_profile=balanced

# High throughput
ansible-playbook tune-mongodb.yml -e tuning_profile=throughput

# Low latency
ansible-playbook tune-mongodb.yml -e tuning_profile=latency
```

### Setup Monitoring (Production Only)
```bash
ansible-playbook setup-monitoring.yml \
  -e enable_pmm=true \
  -e pmm_server_url="https://pmm.example.com"
```

### Delete
```bash
ansible-playbook delete-mongodb.yml -e delete_pvcs=true
```

## Verification

```bash
# Check deployment
oc get pods -n percona-mongo

# Connect to MongoDB
oc port-forward -n percona-mongo svc/mongo-cluster-mongos 27017:27017
mongosh "mongodb://clusterAdmin:clusterAdmin123456@localhost:27017/admin"

# Check shard status
sh.status()
```

## AWX Setup

For AWX/Ansible Tower setup, see README-AWX.md for complete instructions on:
- Project creation
- Credential setup
- Job templates
- Surveys and workflows

## Support

- See README-AWX.md for detailed documentation
- Review individual playbook headers for specific options
- Check OpenShift logs: `oc logs -n percona-mongo <pod-name>`
