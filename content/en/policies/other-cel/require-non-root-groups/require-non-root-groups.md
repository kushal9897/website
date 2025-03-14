---
title: "Require Non-Root Groups in CEL expressions"
category: Sample, EKS Best Practices in CEL
version: 1.11.0
subject: Pod
policyType: "validate"
description: >
    Containers should be forbidden from running with a root primary or supplementary GID. This policy ensures the `runAsGroup`, `supplementalGroups`, and `fsGroup` fields are set to a number greater than zero (i.e., non root). A known issue prevents a policy such as this using `anyPattern` from being persisted properly in Kubernetes 1.23.0-1.23.2.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other-cel/require-non-root-groups/require-non-root-groups.yaml" target="-blank">/other-cel/require-non-root-groups/require-non-root-groups.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-non-root-groups
  annotations:
    policies.kyverno.io/title: Require Non-Root Groups in CEL expressions
    policies.kyverno.io/category: Sample, EKS Best Practices in CEL 
    policies.kyverno.io/severity: medium
    policies.kyverno.io/minversion: 1.11.0
    kyverno.io/kyverno-version: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      Containers should be forbidden from running with a root primary or supplementary GID.
      This policy ensures the `runAsGroup`, `supplementalGroups`, and `fsGroup` fields are set to a number
      greater than zero (i.e., non root). A known issue prevents a policy such as this
      using `anyPattern` from being persisted properly in Kubernetes 1.23.0-1.23.2.
spec:
  validationFailureAction: Audit
  background: true
  rules:
    - name: check-runasgroup
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
            - name: allContainers
              expression: "object.spec.containers + object.spec.?initContainers.orValue([]) + object.spec.?ephemeralContainers.orValue([])"
          expressions:
            - expression: >-
                (
                  object.spec.?securityContext.?runAsGroup.orValue(-1) > 0 &&
                  variables.allContainers.all(container, container.?securityContext.?runAsGroup.orValue(1) > 0)
                ) ||
                (
                  variables.allContainers.all(container, container.?securityContext.?runAsGroup.orValue(-1) > 0) 
                )
              message: >-
                Running with root group IDs is disallowed. The fields
                spec.securityContext.runAsGroup, spec.containers[*].securityContext.runAsGroup,
                spec.initContainers[*].securityContext.runAsGroup, and
                spec.ephemeralContainers[*].securityContext.runAsGroup must be
                set to a value greater than zero.
    - name: check-supplementalgroups
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
          expressions:
            - expression: >-
                object.spec.?securityContext.?supplementalGroups.orValue([]).all(group, group > 0)
              message: >-
                Containers cannot run with a root primary or supplementary GID. The field
                spec.securityContext.supplementalGroups must be unset or
                set to a value greater than zero.
    - name: check-fsgroup
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
          expressions:
            - expression: >-
                object.spec.?securityContext.?fsGroup.orValue(1) > 0
              message: >-
                Containers cannot run with a root primary or supplementary GID. The field
                spec.securityContext.fsGroup must be unset or set to a value greater than zero.


```
