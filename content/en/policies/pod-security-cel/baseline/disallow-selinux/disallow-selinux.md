---
title: "Disallow SELinux in CEL expressions"
category: Pod Security Standards (Baseline) in CEL
version: 1.11.0
subject: Pod
policyType: "validate"
description: >
    SELinux options can be used to escalate privileges and should not be allowed. This policy ensures that the `seLinuxOptions` field is undefined.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//pod-security-cel/baseline/disallow-selinux/disallow-selinux.yaml" target="-blank">/pod-security-cel/baseline/disallow-selinux/disallow-selinux.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: disallow-selinux
  annotations:
    policies.kyverno.io/title: Disallow SELinux in CEL expressions
    policies.kyverno.io/category: Pod Security Standards (Baseline) in CEL
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/minversion: 1.11.0
    kyverno.io/kyverno-version: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/description: >-
      SELinux options can be used to escalate privileges and should not be allowed. This policy
      ensures that the `seLinuxOptions` field is undefined.
spec:
  validationFailureAction: Audit
  background: true
  rules:
    - name: selinux-type
      match:
        any:
        - resources:
            kinds:
              - Pod
            operations:
            - CREATE
            - UPDATE
      validate:
        cel:
          variables:
            - name: allContainerTypes
              expression: "(object.spec.containers + (has(object.spec.initContainers) ? object.spec.initContainers : []) + (has(object.spec.ephemeralContainers) ? object.spec.ephemeralContainers : []))"
            - name: seLinuxTypes
              expression: "['container_t', 'container_init_t', 'container_kvm_t']"
          expressions:
            - expression: >-
                (!has(object.spec.securityContext) ||
                !has(object.spec.securityContext.seLinuxOptions) ||
                !has(object.spec.securityContext.seLinuxOptions.type) ||
                variables.seLinuxTypes.exists(type, type == object.spec.securityContext.seLinuxOptions.type)) &&
                variables.allContainerTypes.all(container, 
                !has(container.securityContext) ||
                !has(container.securityContext.seLinuxOptions) ||
                !has(container.securityContext.seLinuxOptions.type) ||
                variables.seLinuxTypes.exists(type, type == container.securityContext.seLinuxOptions.type))
              message: >-
                Setting the SELinux type is restricted. The field securityContext.seLinuxOptions.type must either be unset or set to one of the allowed values (container_t, container_init_t, or container_kvm_t).
    - name: selinux-user-role
      match:
        any:
        - resources:
            kinds:
              - Pod
            operations:
            - CREATE
            - UPDATE
      validate:
        cel:
          variables:
            - name: allContainerTypes
              expression: "(object.spec.containers + (has(object.spec.initContainers) ? object.spec.initContainers : []) + (has(object.spec.ephemeralContainers) ? object.spec.ephemeralContainers : []))"
          expressions:
            - expression: >-
                (!has(object.spec.securityContext) ||
                !has(object.spec.securityContext.seLinuxOptions) ||
                (!has(object.spec.securityContext.seLinuxOptions.user) && !has(object.spec.securityContext.seLinuxOptions.role))) &&
                variables.allContainerTypes.all(container,
                !has(container.securityContext) ||
                !has(container.securityContext.seLinuxOptions) ||
                (!has(container.securityContext.seLinuxOptions.user) && !has(container.securityContext.seLinuxOptions.role)))
              message: >-
                Setting the SELinux user or role is forbidden. The fields seLinuxOptions.user and seLinuxOptions.role must be unset.
          
```
