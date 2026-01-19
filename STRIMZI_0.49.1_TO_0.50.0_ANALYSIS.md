# Strimzi Kafka Operator: Analysis of Changes from 0.49.1 to 0.50.0

## Executive Summary

This document provides a comprehensive analysis of changes between Strimzi versions 0.49.1 and 0.50.0, including both documented changes and indirect impacts from dependency updates and configuration changes.

**Critical Changes:**
- Java 21 runtime upgrade (from Java 11)
- Breaking configuration change: `connector.plugin.version` now forbidden
- Strimzi Drain Cleaner updated to 1.5.0
- Support for Linux user namespaces added

---

## 1. Major Changes from Release Notes

### 1.1 Java Runtime Upgrade to Java 21

**Impact Level:** HIGH

**Change Description:**
- Strimzi Operators now use **Java 21** as both runtime and language level
- **Exceptions:** The following modules remain on Java 11 for backward compatibility:
  - `api` module
  - `test` module
  - `crd-annotations` module
  - `crd-generator` module

**Rationale:**
These modules will remain on Java 11 language level through Strimzi 0.51.x and will upgrade to Java 21 in Strimzi 1.0.0.

**Migration Impact:**
- If you use any of the above modules as dependencies in your Java projects, you will need to upgrade to Java 21 when moving to Strimzi 1.0.0
- Existing projects using operator modules as dependencies must upgrade to Java 21 immediately

**Technical Details:**
From `pom.xml`:
```xml
<maven.compiler.release>21</maven.compiler.release>
```

From `docker-images/base/Dockerfile`:
```dockerfile
ARG JAVA_VERSION=21
ENV JAVA_HOME=/usr/lib/jvm/jre-${JAVA_VERSION}
```

**Dependencies Affected:**
- All operator components (cluster-operator, topic-operator, user-operator)
- Kafka-init, certificate-manager, and other runtime components
- Container images now bundle Java 21 runtime

### 1.2 Strimzi Drain Cleaner Update

**Impact Level:** MEDIUM

**Change Description:**
- Updated from previous version to **1.5.0**
- Included in Strimzi installation files

**What is Drain Cleaner:**
A component that helps with node draining in Kubernetes during cluster maintenance operations.

**Migration Impact:**
- Review Drain Cleaner 1.5.0 release notes for specific changes
- Test drain operations in non-production environments
- Update any custom configurations that reference Drain Cleaner

### 1.3 Linux User Namespaces Support

**Impact Level:** LOW to MEDIUM

**Change Description:**
- New support for Linux user namespaces through the `hostUsers` Pod option
- Allows better security isolation for Strimzi Pods

**Use Case:**
Enables running Pods with user namespace remapping, providing additional security boundaries between containers and the host system.

**Migration Impact:**
- Optional feature - no action required unless you want to enable it
- Review Kubernetes user namespace documentation before enabling
- Test thoroughly in development environments first

### 1.4 Breaking Change: connector.plugin.version Forbidden

**Impact Level:** HIGH (Breaking Change)

**Change Description:**
The `connector.plugin.version` option is now **FORBIDDEN** in:
- `KafkaConnect` CR in `.spec.config`
- `KafkaMirrorMaker2` CR in:
  - `.spec.mirrors[].sourceConnector.config`
  - `.spec.mirrors[].checkpointConnector.config`

**Required Action:**
Use the dedicated `version` field instead of configuring connector plugin version in the config section.

**Migration Steps:**
1. Identify all KafkaConnect and KafkaMirrorMaker2 resources using `connector.plugin.version`
2. Extract the version value
3. Move it to the new dedicated `version` field
4. Remove from config section

**Example Migration:**
```yaml
# Before (0.49.1)
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaConnect
spec:
  config:
    connector.plugin.version: "3.5.0"

# After (0.50.0)
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaConnect
spec:
  version: "3.5.0"
  config:
    # connector.plugin.version removed
```

---

## 2. Dependency Version Changes

### 2.1 Core Java Dependencies

Based on analysis of `/home/runner/work/strimzi-kafka-operator/strimzi-kafka-operator/pom.xml`, the current dependency versions (as of 0.51.0-SNAPSHOT, which inherits from 0.50.0):

| Dependency | Version | Notes |
|-----------|---------|-------|
| **Kafka** | 4.1.1 | Latest stable Kafka version |
| **Fabric8 Kubernetes Client** | 7.5.0 | Core Kubernetes API interaction |
| **Fabric8 OpenShift Client** | 7.5.0 | OpenShift-specific features |
| **Jackson (all modules)** | 2.18.3 | JSON processing |
| **Vert.x** | 5.0.7 | Reactive toolkit |
| **Log4j** | 2.25.3 | Logging framework |
| **SLF4J** | 2.0.17 | Logging facade |
| **Netty** | 4.2.9.Final | Network application framework |
| **Micrometer** | 1.14.5 | Metrics instrumentation |
| **Strimzi OAuth** | 0.17.1 | OAuth authentication |
| **OpenTelemetry** | 1.34.1 | Observability framework |
| **Jetty** | 12.0.22 | HTTP server/client |
| **Snappy** | 1.1.10.5 | Compression library |
| **Quartz** | 2.5.0 | Job scheduling |

### 2.2 Build Tool Dependencies

| Tool | Version | Purpose |
|------|---------|---------|
| **Maven Compiler** | 3.10.1 | Java compilation |
| **Maven Surefire** | 3.5.4 | Unit test execution |
| **Maven Failsafe** | 3.5.4 | Integration test execution |
| **Maven Checkstyle** | 3.5.0 | Code style checking |
| **Maven SpotBugs** | 4.7.3.4 | Static analysis |
| **Maven Jacoco** | 0.8.12 | Code coverage |
| **Checkstyle** | 10.18.1 | Style checker |
| **Lombok** | 1.18.32 | Code generation |
| **Sundrio** | 0.200.0 | Builder pattern generation |

### 2.3 Test Dependencies

| Dependency | Version | Purpose |
|-----------|---------|---------|
| **JUnit Jupiter** | 6.0.0 | Testing framework |
| **Mockito** | 5.21.0 | Mocking framework |
| **Hamcrest** | 2.2 | Assertion matchers |
| **Testcontainers** | 2.0.2 | Container testing |
| **WireMock** | 3.13.2 | HTTP mocking |
| **Strimzi Test Container** | 0.114.0 | Strimzi-specific testing |
| **Kind Container** | 2.0.0 | Kubernetes in Docker |
| **Bouncy Castle** | 1.78.1 | Cryptography |

### 2.4 Indirect Impacts from Dependency Updates

#### 2.4.1 Jackson 2.18.3
**Potential Impact:**
- JSON serialization/deserialization changes
- New features and bug fixes
- Performance improvements

**Areas to Monitor:**
- Custom resource parsing
- Configuration handling
- API request/response processing

#### 2.4.2 Vert.x 5.0.7
**Potential Impact:**
- Reactive programming model updates
- HTTP/2 and WebSocket handling
- Event bus improvements

**Areas to Monitor:**
- Operator reconciliation loops
- Kubernetes API watch operations
- Internal event handling

#### 2.4.3 Fabric8 Kubernetes Client 7.5.0
**Potential Impact:**
- Kubernetes API compatibility
- Resource model updates
- Client behavior changes

**Areas to Monitor:**
- CRD interactions
- Resource watching and patching
- API server communication

#### 2.4.4 Log4j 2.25.3
**Potential Impact:**
- Security fixes
- Logging performance
- Configuration changes

**Areas to Monitor:**
- Log output format
- Log level behavior
- Performance in high-volume logging scenarios

#### 2.4.5 Netty 4.2.9.Final
**Potential Impact:**
- Network I/O performance
- TLS/SSL handling
- Buffer management

**Areas to Monitor:**
- Kafka broker communication
- Kubernetes API connections
- Memory usage patterns

---

## 3. Kafka Version Support Changes

From `kafka-versions.yaml`, the currently supported Kafka versions are:

| Kafka Version | Metadata Version | Supported | Default | Third-Party Libs |
|--------------|------------------|-----------|---------|------------------|
| 4.0.0 | 4.0 | ✅ | ❌ | 4.0.x |
| 4.0.1 | 4.0 | ✅ | ❌ | 4.0.x |
| 4.1.0 | 4.1 | ✅ | ❌ | 4.1.x |
| 4.1.1 | 4.1 | ✅ | ✅ | 4.1.x |

**Previous versions (3.9.x and earlier) are no longer supported.**

**Impact:**
- Clusters must upgrade to Kafka 4.0.x or 4.1.x
- Default Kafka version is now 4.1.1
- Third-party library dependencies updated for Kafka 4.x compatibility

---

## 4. Configuration and API Changes

### 4.1 Docker Image Changes

**Base Image:**
- Uses Red Hat UBI9 minimal: `registry.access.redhat.com/ubi9/ubi-minimal:latest`
- Java 21 JRE bundled
- Tini v0.19.0 as init system

**Security:**
- Multi-architecture support (amd64, arm64, ppc64le, s390x)
- SHA256 checksums for all downloaded components
- Non-root user (UID 1001) for operator processes

### 4.2 Build Configuration Changes

**Maven Compiler Settings:**
```xml
<maven.compiler.release>21</maven.compiler.release>
```

**Modules with Java 11:**
```xml
<!-- In api/pom.xml, test/pom.xml, crd-annotations/pom.xml, crd-generator/pom.xml -->
<maven.compiler.release>11</maven.compiler.release>
```

### 4.3 Feature Flags and Capabilities

While not explicitly mentioned in the 0.50.0 release notes, review:
- Any new feature gates introduced
- Changes to existing feature gate defaults
- Deprecated features slated for removal

---

## 5. Indirect Impact Analysis

### 5.1 Performance Implications

**Java 21 Benefits:**
- Virtual threads (Project Loom) available but not yet utilized
- Improved garbage collection
- Better compiler optimizations
- Potential for future performance improvements

**Dependency Updates:**
- Jackson 2.18.3: JSON processing performance improvements
- Netty 4.2.9: Network I/O optimizations
- Vert.x 5.0.7: Reactive performance enhancements

### 5.2 Security Implications

**Java 21:**
- Latest security patches and fixes
- Modern cryptography support
- Enhanced security features

**Log4j 2.25.3:**
- Contains fixes for any recent security vulnerabilities
- Improved security defaults

**User Namespaces:**
- Enhanced container isolation
- Reduced attack surface
- Better multi-tenant security

### 5.3 Operational Considerations

**Monitoring:**
- Micrometer 1.14.5 brings new metrics capabilities
- OpenTelemetry 1.34.1 for distributed tracing

**Resource Usage:**
- Java 21 memory footprint may differ from Java 11
- Monitor container resource usage after upgrade
- Adjust resource limits if needed

**Logging:**
- Log4j 2.25.3 may have different default formats
- Review log aggregation configurations
- Validate log parsing pipelines

### 5.4 Compatibility Considerations

**Kubernetes:**
- Fabric8 client 7.5.0 supports Kubernetes 1.28+
- Test with your specific Kubernetes version
- Verify CRD compatibility

**OpenShift:**
- OpenShift client 7.5.0 compatibility
- Test with your OpenShift version
- Verify operator deployment

**Java Client Dependencies:**
- Projects using `api`, `crd-annotations`, etc. modules
- Must support Java 11 minimum (Java 21 from Strimzi 1.0.0)
- Review transitive dependency compatibility

---

## 6. Testing and Validation Strategy

### 6.1 Pre-Upgrade Testing

1. **Environment Validation**
   - Verify Kubernetes/OpenShift version compatibility
   - Check Java 21 availability in container runtime
   - Validate network policies for new communication patterns

2. **Configuration Audit**
   - Identify all uses of `connector.plugin.version`
   - Review custom configurations
   - Document current state

3. **Backup**
   - Backup all Kafka custom resources
   - Export current operator configurations
   - Document current state and versions

### 6.2 Test Scenarios

1. **Basic Functionality**
   - Operator startup and reconciliation
   - CRD creation and updates
   - Resource status reporting

2. **Connector Configuration**
   - Test migration of `connector.plugin.version` to `version` field
   - Validate KafkaConnect deployments
   - Test KafkaMirrorMaker2 configurations

3. **Performance Testing**
   - Compare reconciliation times
   - Monitor resource usage (CPU, memory)
   - Validate throughput and latency

4. **Integration Testing**
   - Kafka broker connectivity
   - Topic and user management
   - Authentication and authorization

5. **Upgrade/Downgrade Testing**
   - Test upgrade path from 0.49.1
   - Validate rollback procedures
   - Test incremental upgrades

### 6.3 Monitoring During Upgrade

**Key Metrics:**
- Operator pod restarts
- Reconciliation loop duration
- Error rates in operator logs
- Kafka cluster health
- Topic and partition status

**Critical Logs:**
- Java version verification
- Dependency loading
- Configuration parsing
- API compatibility warnings

---

## 7. Migration Checklist

### Pre-Migration

- [ ] Review this analysis document completely
- [ ] Audit all KafkaConnect and KafkaMirrorMaker2 resources for `connector.plugin.version`
- [ ] Test in development environment
- [ ] Backup all custom resources and configurations
- [ ] Verify Java 21 availability in target environment
- [ ] Review Strimzi 0.50.0 release notes
- [ ] Check Drain Cleaner 1.5.0 compatibility
- [ ] Plan rollback strategy

### Migration

- [ ] Update Strimzi operator to 0.50.0
- [ ] Verify operator pods start successfully
- [ ] Check operator logs for errors or warnings
- [ ] Migrate `connector.plugin.version` configurations
- [ ] Update KafkaConnect resources
- [ ] Update KafkaMirrorMaker2 resources
- [ ] Verify all custom resources reconcile successfully
- [ ] Test Kafka cluster operations

### Post-Migration

- [ ] Monitor operator resource usage
- [ ] Verify all Kafka brokers are healthy
- [ ] Check topic and partition status
- [ ] Validate connector deployments
- [ ] Test user authentication and authorization
- [ ] Review metrics and monitoring dashboards
- [ ] Update documentation with new version
- [ ] Document any issues or observations

---

## 8. Known Issues and Considerations

### 8.1 Java 21 Specific

- **Memory Usage:** Java 21 may have different memory profiles than Java 11. Monitor and adjust resource limits.
- **GC Behavior:** Garbage collection patterns may change. Review GC logs if performance issues occur.
- **Startup Time:** Initial startup may be slower or faster depending on JIT compilation.

### 8.2 Configuration Changes

- **Validation:** The operator may enforce stricter validation on custom resources.
- **Defaults:** Some default values may have changed. Review resource specifications.
- **Deprecated Fields:** Watch for deprecation warnings in logs.

### 8.3 Dependency Conflicts

- **Transitive Dependencies:** Projects using Strimzi modules may encounter dependency conflicts.
- **Version Pinning:** May need to explicitly manage dependency versions in your projects.
- **API Changes:** Some dependencies may have breaking API changes.

---

## 9. Additional Resources

### Official Documentation

- [Strimzi 0.50.0 Release Notes](https://github.com/strimzi/strimzi-kafka-operator/releases/tag/0.50.0)
- [Strimzi Documentation](https://strimzi.io/documentation/)
- [Kafka 4.1.1 Documentation](https://kafka.apache.org/documentation/)

### Dependency Documentation

- [Java 21 Release Notes](https://www.oracle.com/java/technologies/javase/21-relnote-issues.html)
- [Fabric8 Kubernetes Client](https://github.com/fabric8io/kubernetes-client)
- [Vert.x Documentation](https://vertx.io/docs/)
- [Jackson Documentation](https://github.com/FasterXML/jackson)

### Community Resources

- [Strimzi Slack Channel](https://strimzi.io/join-us/)
- [Strimzi GitHub Issues](https://github.com/strimzi/strimzi-kafka-operator/issues)
- [Strimzi Mailing List](https://lists.cncf.io/g/cncf-strimzi-users)

---

## 10. Summary of Critical Actions

### Must Do (Breaking Changes)

1. **Migrate connector.plugin.version configurations** - This is now forbidden and will cause deployment failures
2. **Verify Java 21 compatibility** - Especially if using Strimzi modules as dependencies
3. **Test in non-production** - Significant runtime changes require thorough testing

### Should Do (Recommended)

1. **Review dependency updates** - Understand implications of updated libraries
2. **Update monitoring** - Adjust dashboards for Java 21 metrics
3. **Review resource limits** - May need adjustment for Java 21
4. **Update documentation** - Reflect new version and configuration changes

### Optional (Enhancement)

1. **Enable user namespaces** - If enhanced security isolation is needed
2. **Leverage new Drain Cleaner features** - Review 1.5.0 capabilities
3. **Optimize configurations** - Take advantage of any new options

---

## Appendix A: Dependency Version Matrix

### Complete Dependency List from pom.xml

```xml
<!-- Runtime -->
fabric8.kubernetes-client.version: 7.5.0
fabric8.openshift-client.version: 7.5.0
fabric8.kubernetes-model.version: 7.5.0
fasterxml.jackson-core.version: 2.18.3
vertx.version: 5.0.7
kafka.version: 4.1.1
slf4j.version: 2.0.17
log4j.version: 2.25.3
netty.version: 4.2.9.Final
micrometer.version: 1.14.5
opentelemetry.version: 1.34.1
strimzi-oauth.version: 0.17.1

<!-- Build Tools -->
maven.compiler.version: 3.10.1
maven.surefire.version: 3.5.4
maven.checkstyle.version: 3.5.0
checkstyle.version: 10.18.1
lombok.version: 1.18.32

<!-- Testing -->
jupiter.version: 6.0.0
mockito.version: 5.21.0
testcontainer.version: 2.0.2
```

---

## Appendix B: Module Java Version Matrix

| Module | Java Language Level | Reason |
|--------|-------------------|---------|
| cluster-operator | 21 | Operator runtime |
| topic-operator | 21 | Operator runtime |
| user-operator | 21 | Operator runtime |
| operator-common | 21 | Operator runtime |
| kafka-init | 21 | Runtime component |
| certificate-manager | 21 | Runtime component |
| kafka-agent | 21 | Runtime component |
| tracing-agent | 21 | Runtime component |
| **api** | **11** | **Library compatibility** |
| **test** | **11** | **Library compatibility** |
| **crd-annotations** | **11** | **Library compatibility** |
| **crd-generator** | **11** | **Library compatibility** |
| config-model | 21 | Operator runtime |
| config-model-generator | 21 | Build tool |
| v1-api-conversion | 21 | Operator runtime |

---

**Document Version:** 1.0  
**Analysis Date:** January 19, 2026  
**Strimzi Version Analyzed:** 0.49.1 → 0.50.0  
**Current Repository Version:** 0.51.0-SNAPSHOT  
