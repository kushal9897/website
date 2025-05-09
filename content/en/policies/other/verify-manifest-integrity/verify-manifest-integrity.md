---
title: "Verify Manifest Integrity"
category: Other
version: 1.8.0
subject: Deployment
policyType: "validate"
description: >
    Verifying the integrity of resources is important to ensure no tampering has occurred, and in some cases this may need to be extended to certain YAML manifests deployed to Kubernetes. Starting in Kyverno 1.8, these manifests may be signed with Sigstore and the signature(s) validated to prevent this tampering while still allowing some exceptions on a per-field basis. This policy verifies Deployments are signed with the expected key but ignores the `spec.replicas` field allowing other teams to change just this value.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/verify-manifest-integrity/verify-manifest-integrity.yaml" target="-blank">/other/verify-manifest-integrity/verify-manifest-integrity.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: verify-manifest-integrity
  annotations:
    policies.kyverno.io/title: Verify Manifest Integrity
    policies.kyverno.io/category: Other
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Deployment
    kyverno.io/kyverno-version: 1.8.0
    policies.kyverno.io/minversion: 1.8.0
    kyverno.io/kubernetes-version: "1.24"
    policies.kyverno.io/description: >-
      Verifying the integrity of resources is important to ensure no tampering has
      occurred, and in some cases this may need to be extended to certain YAML manifests
      deployed to Kubernetes. Starting in Kyverno 1.8, these manifests may be signed with
      Sigstore and the signature(s) validated to prevent this tampering while still allowing
      some exceptions on a per-field basis. This policy verifies Deployments are signed with
      the expected key but ignores the `spec.replicas` field allowing other teams to change just
      this value.
spec:
  validationFailureAction: Audit
  background: true
  rules:
    - name: verify-deployment-allow-replicas
      match:
        any:
        - resources:
            kinds:
              - Deployment
      validate:
        manifests:
          attestors:
          - count: 1
            entries:
            - keys:
                publicKeys: |-
                  -----BEGIN PUBLIC KEY-----
                  MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEStoX3dPCFYFD2uPgTjZOf1I5UFTa
                  1tIu7uoGoyTxJqqEq7K2aqU+vy+aK76uQ5mcllc+TymVtcLk10kcKvb3FQ==
                  -----END PUBLIC KEY-----
          ignoreFields:
          - objects:
            - kind: Deployment
            fields:
            - spec.replicas

```
