---
title: "Restrict control plane scheduling"
category: Sample
version: 1.6.0
subject: Pod
policyType: "validate"
description: >
    Scheduling non-system Pods to control plane nodes (which run kubelet) is often undesirable because it takes away resources from the control plane components and can represent a possible security threat vector. This policy prevents users from setting a toleration in a Pod spec which allows running on control plane nodes with the taint key `node-role.kubernetes.io/master`.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/restrict-controlplane-scheduling/restrict-controlplane-scheduling.yaml" target="-blank">/other/restrict-controlplane-scheduling/restrict-controlplane-scheduling.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: restrict-controlplane-scheduling
  annotations:
    policies.kyverno.io/title: Restrict control plane scheduling
    policies.kyverno.io/category: Sample
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/minversion: 1.6.0
    policies.kyverno.io/description: >-
      Scheduling non-system Pods to control plane nodes (which run kubelet) is often undesirable
      because it takes away resources from the control plane components and can represent
      a possible security threat vector. This policy prevents users from setting a toleration
      in a Pod spec which allows running on control plane nodes
      with the taint key `node-role.kubernetes.io/master`.
spec:
  validationFailureAction: Audit
  background: true
  rules:
  - name: restrict-controlplane-scheduling-master
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: Pods may not use tolerations which schedule on control plane nodes.
      pattern:
        spec:
          =(tolerations):
            - key: "!node-role.kubernetes.io/master"
  - name: restrict-controlplane-scheduling-control-plane
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: Pods may not use tolerations which schedule on control plane nodes.
      pattern:
        spec:
          =(tolerations):
            - key: "!node-role.kubernetes.io/control-plane"
```
