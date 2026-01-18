# Percona MongoDB Ansible Bundle - File List

## Configuration Files (Provided)
- ✅ ansible.cfg - Ansible configuration
- ✅ inventory.yml - Inventory for localhost
- ✅ requirements.yml - Required Ansible collections
- ✅ group_vars/dev.yml - Development environment settings
- ✅ group_vars/prod.yml - Production environment settings

## Playbooks (Copy from Claude Artifacts)
- ⏳ deploy-percona-mongodb.yml - Main deployment playbook
- ⏳ backup-mongodb.yml - Manual backup trigger
- ⏳ restore-mongodb.yml - Restore from backup
- ⏳ delete-mongodb.yml - Cluster cleanup
- ⏳ scale-mongodb.yml - Scale cluster components
- ⏳ upgrade-mongodb.yml - Upgrade MongoDB versions
- ⏳ setup-monitoring.yml - Production monitoring setup
- ⏳ tune-mongodb.yml - Performance tuning

## Documentation (Copy from Claude Artifacts)
- ⏳ README-AWX.md - Complete AWX setup guide

## Helper Files (Provided)
- ✅ QUICK_START.md - Quick reference guide
- ✅ FILE_LIST.md - This file
- ✅ PLAYBOOKS_HERE.txt - Instructions for copying playbooks

## Total Files
- Provided: 8 files
- To Copy: 9 files
- Total: 17 files

## Verification Checklist

After copying all playbooks, verify you have:
- [ ] All 8 playbook YAML files
- [ ] README-AWX.md documentation
- [ ] Configuration files in place
- [ ] group_vars directory with dev.yml and prod.yml

Run this to check:
```bash
ls -1 *.yml | wc -l  # Should show 12 files (3 config + 8 playbooks + inventory)
ls -1 *.md | wc -l   # Should show 3 files (README-AWX + QUICK_START + FILE_LIST)
```
