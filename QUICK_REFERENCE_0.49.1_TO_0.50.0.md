# Quick Reference: Strimzi 0.49.1 → 0.50.0

## Critical Breaking Changes ⚠️

### 1. connector.plugin.version is NOW FORBIDDEN
- **Old way (0.49.1):**
  ```yaml
  spec:
    config:
      connector.plugin.version: "3.5.0"
  ```
- **New way (0.50.0):**
  ```yaml
  spec:
    version: "3.5.0"
    config: {}
  ```
- **Impact:** Affects `KafkaConnect` and `KafkaMirrorMaker2` resources
- **Action Required:** Migrate all configurations before upgrading

### 2. Java 21 Runtime Required
- **Old:** Java 11
- **New:** Java 21
- **Impact:** All operator components run on Java 21
- **Exception:** `api`, `test`, `crd-annotations`, `crd-generator` modules still use Java 11 (until v1.0.0)
- **Action Required:** If using these modules as dependencies, verify Java 11+ compatibility

## Feature Updates

### Drain Cleaner 1.5.0
- Updated from previous version
- Included in installation files
- Review release notes for specific changes

### Linux User Namespaces
- New `hostUsers` Pod option available
- Provides enhanced security isolation
- Optional - no action required

## Key Dependency Updates

| Component | Version | Notes |
|-----------|---------|-------|
| Kafka | 4.1.1 | Default version |
| Fabric8 K8s Client | 7.5.0 | Core K8s interaction |
| Jackson | 2.18.3 | JSON processing |
| Vert.x | 5.0.7 | Reactive framework |
| Log4j | 2.25.3 | Logging |
| Netty | 4.2.9.Final | Network I/O |
| Micrometer | 1.14.5 | Metrics |
| Strimzi OAuth | 0.17.1 | Authentication |

## Supported Kafka Versions

✅ **Supported:**
- 4.1.1 (default)
- 4.1.0
- 4.0.1
- 4.0.0

❌ **No longer supported:**
- 3.9.x and earlier

## Pre-Upgrade Checklist

- [ ] Search for `connector.plugin.version` in all YAML files
- [ ] Prepare migration for connector configurations
- [ ] Verify Java 21 availability in container runtime
- [ ] Backup all custom resources
- [ ] Test in non-production environment
- [ ] Review resource limits (may need adjustment for Java 21)
- [ ] Plan rollback strategy

## Quick Migration Steps

### Step 1: Audit Configurations
```bash
# Find all uses of connector.plugin.version
grep -r "connector.plugin.version" /path/to/yaml/files/
```

### Step 2: Update KafkaConnect Resources
```bash
# Before upgrade, manually edit each resource:
# 1. Extract connector.plugin.version value from spec.config
# 2. Add spec.version field with that value
# 3. Remove connector.plugin.version from spec.config
```

### Step 3: Upgrade Operator
```bash
# Follow standard Strimzi upgrade procedure
kubectl apply -f install/cluster-operator/0.50.0/
```

### Step 4: Verify
```bash
# Check operator is running
kubectl get pods -n kafka

# Check all resources reconciled
kubectl get kafka,kafkaconnect,kafkamirrormaker2 -A

# Review operator logs
kubectl logs -n kafka deployment/strimzi-cluster-operator
```

## Monitoring After Upgrade

### Critical Metrics
- Operator pod restart count
- Reconciliation loop duration
- Memory usage (Java 21 may differ from Java 11)
- Error rates in logs

### Key Log Searches
```bash
# Java version verification
kubectl logs -n kafka deployment/strimzi-cluster-operator | grep "Java version"

# Configuration errors
kubectl logs -n kafka deployment/strimzi-cluster-operator | grep -i "error\|exception"

# Connector issues
kubectl logs -n kafka deployment/strimzi-cluster-operator | grep "connector.plugin.version"
```

## Common Issues and Solutions

### Issue: "connector.plugin.version is not allowed"
**Cause:** Configuration not migrated to new format  
**Solution:** Move version to `spec.version` field

### Issue: Increased memory usage
**Cause:** Java 21 runtime differences  
**Solution:** Adjust resource limits in operator deployment

### Issue: Operator fails to start
**Cause:** Java 21 not available in container runtime  
**Solution:** Verify container image has Java 21

## Rollback Procedure

If needed, rollback to 0.49.1:
```bash
# 1. Restore operator to 0.49.1
kubectl apply -f install/cluster-operator/0.49.1/

# 2. Revert connector configurations
#    (Put connector.plugin.version back in spec.config if needed)

# 3. Verify all resources reconcile
kubectl get kafka,kafkaconnect,kafkamirrormaker2 -A

# 4. Check operator logs
kubectl logs -n kafka deployment/strimzi-cluster-operator
```

## Resources

- **Full Analysis:** See `STRIMZI_0.49.1_TO_0.50.0_ANALYSIS.md`
- **Official Release Notes:** https://github.com/strimzi/strimzi-kafka-operator/releases/tag/0.50.0
- **CHANGELOG:** `/CHANGELOG.md` in repository
- **Community Support:** https://strimzi.io/join-us/

## Risk Assessment

| Change | Risk Level | Effort | Testing Priority |
|--------|-----------|--------|------------------|
| connector.plugin.version migration | 🔴 HIGH | Medium | Critical |
| Java 21 upgrade | 🟡 MEDIUM | Low | High |
| Drain Cleaner update | 🟢 LOW | Low | Medium |
| Dependency updates | 🟡 MEDIUM | Low | High |
| User namespaces support | 🟢 LOW | Low (optional) | Low |

## Timeline Recommendation

1. **Week 1:** Test in development
2. **Week 2:** Test in staging
3. **Week 3:** Migration preparation (audit configs)
4. **Week 4:** Production upgrade (off-peak hours)

---

**For detailed analysis, see:** `STRIMZI_0.49.1_TO_0.50.0_ANALYSIS.md`
