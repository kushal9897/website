---
title: "Disallow Privilege Escalation in CEL"
category: Pod Security Standards (Restricted) in CEL
version: 1.11.0
subject: Pod
policyType: "validate"
description: >
    Privilege escalation, such as via set-user-ID or set-group-ID file mode, should not be allowed. This policy ensures the `allowPrivilegeEscalation` field is set to `false`.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//pod-security-cel/restricted/disallow-privilege-escalation/disallow-privilege-escalation.yaml" target="-blank">/pod-security-cel/restricted/disallow-privilege-escalation/disallow-privilege-escalation.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: disallow-privilege-escalation
  annotations:
    policies.kyverno.io/title: Disallow Privilege Escalation in CEL
    policies.kyverno.io/category: Pod Security Standards (Restricted) in CEL
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/minversion: 1.11.0
    kyverno.io/kyverno-version: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/description: >-
      Privilege escalation, such as via set-user-ID or set-group-ID file mode, should not be allowed.
      This policy ensures the `allowPrivilegeEscalation` field is set to `false`.
spec:
  validationFailureAction: Audit
  background: true
  rules:
    - name: privilege-escalation
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
              expression: >-
               object.spec.containers + 
               object.spec.?initContainers.orValue([]) + 
               object.spec.?ephemeralContainers.orValue([])
          expressions:
            - expression: >- 
                variables.allContainers.all(container, 
                container.?securityContext.allowPrivilegeEscalation.orValue(true) == false)
              message: >-
                Privilege escalation is disallowed. 
                All containers must set the securityContext.allowPrivilegeEscalation field to `false`.

```
