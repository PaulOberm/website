---
title: "Allowed Annotations in CEL expressions"
category: Other in CEL
version: 1.11.0
subject: Pod, Annotation
policyType: "validate"
description: >
    Rather than creating a deny list of annotations, it may be more useful to invert that list and create an allow list which then denies any others. This policy demonstrates how to allow two annotations with a specific key name of fluxcd.io/ while denying others that do not meet the pattern.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other-cel/allowed-annotations/allowed-annotations.yaml" target="-blank">/other-cel/allowed-annotations/allowed-annotations.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: allowed-annotations
  annotations:
    policies.kyverno.io/title: Allowed Annotations in CEL expressions
    policies.kyverno.io/category: Other in CEL 
    policies.kyverno.io/severity: medium
    kyverno.io/kyverno-version: 1.11.0
    policies.kyverno.io/minversion: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/subject: Pod, Annotation
    policies.kyverno.io/description: >-
      Rather than creating a deny list of annotations, it may be more useful
      to invert that list and create an allow list which then denies any others.
      This policy demonstrates how to allow two annotations with a specific key
      name of fluxcd.io/ while denying others that do not meet the pattern.
spec:
  validationFailureAction: Audit
  background: true
  rules:
  - name: allowed-fluxcd-annotations
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
              object.metadata.?annotations.orValue([]).all(annotation, !annotation.contains('fluxcd.io/') || annotation in ['fluxcd.io/cow', 'fluxcd.io/dog'])
            message: The only approved FluxCD annotations are `fluxcd.io/cow` and `fluxcd.io/dog`.
      

```
