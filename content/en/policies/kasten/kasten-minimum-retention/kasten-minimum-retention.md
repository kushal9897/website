---
title: "Set Kasten Policy Minimum Backup Retention"
category: Veeam Kasten
version: 1.6.2
subject: Policy
policyType: "mutate"
description: >
    Example Kyverno policy to enforce common compliance retention standards by modifying Kasten Policy backup retention settings. Based on regulation/compliance standard requirements, uncomment (1) of the desired GFS retention schedules to mutate existing and future Kasten Policies. Alternatively, this policy can be used to reduce retention lengths to enforce cost optimization. NOTE: This example only applies to Kasten Policies with an '@hourly' frequency. Refer to Kasten documentation for Policy API specification if modifications are necessary: https://docs.kasten.io/latest/api/policies.html#policy-api-type
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//kasten/kasten-minimum-retention/kasten-minimum-retention.yaml" target="-blank">/kasten/kasten-minimum-retention/kasten-minimum-retention.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: kasten-minimum-retention
  annotations:
    policies.kyverno.io/title: Set Kasten Policy Minimum Backup Retention
    policies.kyverno.io/category: Veeam Kasten
    kyverno.io/kyverno-version: 1.12.1
    policies.kyverno.io/minversion: 1.6.2
    kyverno.io/kubernetes-version: "1.24-1.30"
    policies.kyverno.io/subject: Policy
    policies.kyverno.io/description: >-
      Example Kyverno policy to enforce common compliance retention standards by modifying Kasten Policy backup retention settings. Based on regulation/compliance standard requirements, uncomment (1) of the desired GFS retention schedules to mutate existing and future Kasten Policies. Alternatively, this policy can be used to reduce retention lengths to enforce cost optimization. NOTE: This example only applies to Kasten Policies with an '@hourly' frequency. Refer to Kasten documentation for Policy API specification if modifications are necessary: https://docs.kasten.io/latest/api/policies.html#policy-api-type
spec:
  rules:
  - name: kasten-minimum-retention
    match:
      any:
      - resources:
          kinds:
          - config.kio.kasten.io/v1alpha1/Policy
    preconditions:
      all:
      # Match only @hourly policies that do not use policy presets, as the
      # number of retained artifacts can only be specified for frequencies
      # of the same or lower granularity than the policy frequency. For example,
      # if the policy frequency is '@daily', then retention can have values for
      # 'daily', 'weekly', 'monthly' and 'yearly', but not for 'hourly'.
      # If the policy frequency is 'hourly', then all retention values are
      # allowed. If the policy frequency is '@onDemand' or policy preset is used
      # then retention values are not allowed.
      - key: "{{ request.object.spec.frequency || ''}}"
        operator: Equals
        value: '@hourly'
    mutate: 
      # Federal Information Security Management Act (FISMA): 3 Years 
      #patchesJson6902: |-
      #  - path: "/spec/retention"
      #    op: replace
      #    value: {"hourly":24,"daily":30,"weekly":4,"monthly":12,"yearly":3}
      
      # Health Insurance Portability and Accountability Act (HIPAA):  6 Years   
      #patchesJson6902: |-
      #  - path: "/spec/retention"
      #    op: replace
      #    value: {"hourly":24,"daily":30,"weekly":4,"monthly":12,"yearly":6}
      
      # National Energy Commission (NERC): 3 to 6 Years  
      #patchesJson6902: |-
      #  - path: "/spec/retention"
      #    op: replace
      #    value: {"hourly":24,"daily":30,"weekly":4,"monthly":12,"yearly":3}
      
      # Basel II Capital Accord: 3 to 7 Years 
      #patchesJson6902: |-
      #  - path: "/spec/retention"
      #    op: replace
      #    value: {"hourly":24,"daily":30,"weekly":4,"monthly":12,"yearly":3}
      
      # Sarbanes-Oxley Act of 2002 (SOX): 7 Years
      #patchesJson6902: |-
      #  - path: "/spec/retention"
      #    op: replace
      #    value: {"hourly":24,"daily":30,"weekly":4,"monthly":12,"yearly":7}
      
      # National Industrial Security Program Operating Manual (NISPOM): 6 to 12 Months 
      #patchesJson6902: |-
      #  - path: "/spec/retention"
      #    op: replace
      #    value: {"hourly":24,"daily":30,"weekly":4,"monthly":6}
      
      # Cost Optimization (Maximum Retention: 3 Months)
      patchesJson6902: |-
        - path: "/spec/retention"
          op: replace
          value:
            hourly: 24
            daily: 30
            weekly: 4
            monthly: 3

```
