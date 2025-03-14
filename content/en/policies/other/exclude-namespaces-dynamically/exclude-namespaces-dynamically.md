---
title: "Exclude Namespaces Dynamically"
category: Sample
version: 1.6.0
subject: Namespace, Pod
policyType: "validate"
description: >
    It's common where policy lookups need to consider a mapping to many possible values rather than a static mapping. This is a sample which demonstrates how to dynamically look up an allow list of Namespaces from a ConfigMap where the ConfigMap stores an array of strings. This policy validates that any Pods created outside of the list of Namespaces have the label `foo` applied.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/exclude-namespaces-dynamically/exclude-namespaces-dynamically.yaml" target="-blank">/other/exclude-namespaces-dynamically/exclude-namespaces-dynamically.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: exclude-namespaces-example
  annotations:
    policies.kyverno.io/title: Exclude Namespaces Dynamically
    policies.kyverno.io/category: Sample
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Namespace, Pod
    policies.kyverno.io/minversion: 1.6.0
    pod-policies.kyverno.io/autogen-controllers: none
    kyverno.io/kyverno-version: 1.6.0
    kyverno.io/kubernetes-version: "1.23"
    policies.kyverno.io/description: >-
      It's common where policy lookups need to consider a mapping to many possible values rather than a
      static mapping. This is a sample which demonstrates how to dynamically look up an allow list of Namespaces from a ConfigMap
      where the ConfigMap stores an array of strings. This policy validates that any Pods created
      outside of the list of Namespaces have the label `foo` applied.
spec:
  validationFailureAction: Audit
  background: true
  rules:
  - name: exclude-namespaces-dynamically
    context:
      - name: namespacefilters
        configMap:
          name: namespace-filters
          namespace: default
    match:
      any:
      - resources:
          kinds:
          - Deployment
          - DaemonSet
          - StatefulSet
          - Job
    preconditions:
      all:
      - key: "{{request.object.metadata.namespace}}"
        operator: AnyNotIn
        value: "{{namespacefilters.data.exclude}}"
    validate:
      message: >
        Creating Pods in the {{request.namespace}} namespace,
        which is not in the excluded list of namespaces {{ namespacefilters.data.exclude }},
        is forbidden unless it carries the label `foo`.
      pattern:
        spec:
          template:
            metadata:
              labels:
                foo: "*"
  - name: exclude-namespaces-dynamically-pods
    context:
      - name: namespacefilters
        configMap:
          name: namespace-filters
          namespace: default
    match:
      any:
      - resources:
          kinds:
          - Pod
    preconditions:
      all:
      - key: "{{request.object.metadata.namespace}}"
        operator: AnyNotIn
        value: "{{namespacefilters.data.exclude}}"
    validate:
      message: >
        Creating Pods in the {{request.namespace}} namespace,
        which is not in the excluded list of namespaces {{ namespacefilters.data.exclude }},
        is forbidden unless it carries the label `foo`.
      pattern:
        metadata:
          labels:
            foo: "*"
  - name: exclude-namespaces-dynamically-cronjobs
    context:
      - name: namespacefilters
        configMap:
          name: namespace-filters
          namespace: default
    match:
      any:
      - resources:
          kinds:
          - CronJob
    preconditions:
      all:
      - key: "{{request.object.metadata.namespace}}"
        operator: AnyNotIn
        value: "{{namespacefilters.data.exclude}}"
    validate:
      message: >
        Creating Pods in the {{request.namespace}} namespace,
        which is not in the excluded list of namespaces {{ namespacefilters.data.exclude }},
        is forbidden unless it carries the label `foo`.
      pattern:
        spec:
          jobTemplate:
            spec:
              template:
                metadata:
                  labels:
                    foo: "*"
```
