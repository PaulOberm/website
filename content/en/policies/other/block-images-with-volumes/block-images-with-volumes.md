---
title: "Block Images with Volumes"
category: Other
version: 1.6.0
subject: Pod
policyType: "validate"
description: >
    OCI images may optionally be built with VOLUME statements which, if run in read-only mode, would still result in write access to the specified location. This may be unexpected and undesirable. This policy checks the contents of every container image and inspects them for such VOLUME statements, then blocks if found.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/block-images-with-volumes/block-images-with-volumes.yaml" target="-blank">/other/block-images-with-volumes/block-images-with-volumes.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: block-images-with-volumes
  annotations:
    policies.kyverno.io/title: Block Images with Volumes
    policies.kyverno.io/category: Other
    policies.kyverno.io/severity: medium
    kyverno.io/kyverno-version: 1.6.0
    policies.kyverno.io/minversion: 1.6.0
    kyverno.io/kubernetes-version: "1.23"
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      OCI images may optionally be built with VOLUME statements which, if run
      in read-only mode, would still result in write access to the specified location.
      This may be unexpected and undesirable. This policy checks the contents of every
      container image and inspects them for such VOLUME statements, then blocks if found.
spec:
  validationFailureAction: Audit
  rules:
  - name: block-images-with-vols
    match:
      any:
      - resources:
          kinds:
          - Pod
    preconditions:
      all:
      - key: "{{request.operation || 'BACKGROUND'}}"
        operator: NotEquals
        value: DELETE
    validate:
      message: "Images containing built-in volumes are prohibited."
      foreach:
      - list: "request.object.spec.containers"
        context: 
        - name: imageData
          imageRegistry: 
            reference: "{{ element.image }}"
        deny:
          conditions:
            all:
              - key: "{{ imageData.configData.config.Volumes || '' | length(@) }}"
                operator: GreaterThan
                value: 0
```
