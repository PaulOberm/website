---
title: "Require Requests and Limits for emptyDir in CEL expressions"
category: Other in CEL
version: 
subject: Pod
policyType: "validate"
description: >
    Pods which mount emptyDir volumes may be allowed to potentially overrun the medium backing the emptyDir volume. This sample ensures that any initContainers or containers mounting an emptyDir volume have ephemeral-storage requests and limits set. Policy will be skipped if the volume has already a sizeLimit set.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other-cel/require-emptydir-requests-limits/require-emptydir-requests-limits.yaml" target="-blank">/other-cel/require-emptydir-requests-limits/require-emptydir-requests-limits.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-emptydir-requests-and-limits
  annotations:
    policies.kyverno.io/title: Require Requests and Limits for emptyDir in CEL expressions
    policies.kyverno.io/category: Other in CEL 
    policies.kyverno.io/severity: medium
    kyverno.io/kyverno-version: 1.12.1
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      Pods which mount emptyDir volumes may be allowed to potentially overrun
      the medium backing the emptyDir volume. This sample ensures that any
      initContainers or containers mounting an emptyDir volume have
      ephemeral-storage requests and limits set. Policy will be skipped if
      the volume has already a sizeLimit set.
spec:
  background: false
  validationFailureAction: Audit
  rules:
    - name: check-emptydir-requests-limits
      match:
        any:
          - resources:
              kinds:
                - Pod
              operations:
              - CREATE
              - UPDATE
      celPreconditions:
        - name: "has-emptydir-volume"
          expression: "object.spec.?volumes.orValue([]).exists(volume, has(volume.emptyDir))"
      validate:
        cel:
          variables:
            - name: containers
              expression: "object.spec.containers + object.spec.?initContainers.orValue([])"
            - name: emptydirnames
              expression: >-
                has(object.spec.volumes) ? 
                object.spec.volumes.filter(volume, has(volume.emptyDir) && !has(volume.emptyDir.sizeLimit)).map(volume, volume.name) : []
          expressions:
            - expression: >-
                variables.containers.all(container,
                !container.?volumeMounts.orValue([]).exists(mount, mount.name in variables.emptydirnames) || 
                container.resources.?requests[?'ephemeral-storage'].hasValue() &&
                container.resources.?limits[?'ephemeral-storage'].hasValue())
              message: Containers mounting emptyDir volumes must specify requests and limits for ephemeral-storage.


```
