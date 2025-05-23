---
title: "Require imagePullPolicy Always"
category: Sample
version: 1.6.0
subject: Pod
policyType: "validate"
description: >
    If the `latest` tag is allowed for images, it is a good idea to have the imagePullPolicy field set to `Always` to ensure should that tag be overwritten that future pulls will get the updated image. This policy validates the imagePullPolicy is set to `Always` when the `latest` tag is specified explicitly or where a tag is not defined at all.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/imagepullpolicy-always/imagepullpolicy-always.yaml" target="-blank">/other/imagepullpolicy-always/imagepullpolicy-always.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: imagepullpolicy-always
  annotations:
    policies.kyverno.io/title: Require imagePullPolicy Always
    policies.kyverno.io/category: Sample
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/minversion: 1.6.0
    policies.kyverno.io/description: >-
      If the `latest` tag is allowed for images, it is a good idea to have the
      imagePullPolicy field set to `Always` to ensure should that tag be overwritten that future
      pulls will get the updated image. This policy validates the imagePullPolicy is set to `Always`
      when the `latest` tag is specified explicitly or where a tag is not defined at all.
spec:
  validationFailureAction: Audit
  background: true
  rules:
  - name: imagepullpolicy-always
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: >-
        The imagePullPolicy must be set to `Always` when the tag `latest` is used.
      pattern:
        spec:
          containers:
          - (image): "*:latest | !*:*"
            imagePullPolicy: "Always"
```
