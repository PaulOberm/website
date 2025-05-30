---
title: "Require Kubecost Labels in CEL expressions"
category: Kubecost in CEL
version: 
subject: Pod, Label
policyType: "validate"
description: >
    Kubecost can use labels assigned to Pods in order to track and display cost allocation in a granular way. These labels, which can be customized, can be used to organize and group workloads in different ways. This policy requires that the labels `owner`, `team`, `department`, `app`, and `env` are all defined on Pods. With Kyverno autogen enabled (absence of the annotation `pod-policies.kyverno.io/autogen-controllers=none`), these labels will also be required for all Pod controllers.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//kubecost-cel/require-kubecost-labels/require-kubecost-labels.yaml" target="-blank">/kubecost-cel/require-kubecost-labels/require-kubecost-labels.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-kubecost-labels
  annotations:
    policies.kyverno.io/title: Require Kubecost Labels in CEL expressions
    policies.kyverno.io/category: Kubecost in CEL 
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod, Label
    kyverno.io/kyverno-version: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/description: >-
      Kubecost can use labels assigned to Pods in order to track and display
      cost allocation in a granular way. These labels, which can be customized, can be used
      to organize and group workloads in different ways. This policy requires that the labels
      `owner`, `team`, `department`, `app`, and `env` are all defined on Pods. With Kyverno
      autogen enabled (absence of the annotation `pod-policies.kyverno.io/autogen-controllers=none`),
      these labels will also be required for all Pod controllers.
spec:
  validationFailureAction: Audit
  background: true
  rules:
  - name: require-labels
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
              object.metadata.?labels.?owner.orValue('') != '' && 
              object.metadata.?labels.?team.orValue('') != '' &&
              object.metadata.?labels.?department.orValue('') != '' &&
              object.metadata.?labels.?app.orValue('') != '' &&
              object.metadata.?labels.?env.orValue('') != ''
            message: "The Kubecost labels `owner`, `team`, `department`, `app`, and `env` are all required for Pods."


```
