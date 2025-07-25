---
title: "Require Run As Non-Root User in ValidatingPolicy"
category: Pod Security Standards (Restricted) in ValidatingPolicy
version: 1.14.0
subject: Pod
policyType: "validate"
description: >
    Containers must be required to run as non-root users. This policy ensures `runAsUser` is either unset or set to a number greater than zero.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//pod-security-vpol/restricted/require-run-as-non-root-user/require-run-as-non-root-user.yaml" target="-blank">/pod-security-vpol/restricted/require-run-as-non-root-user/require-run-as-non-root-user.yaml</a>

```yaml
apiVersion: policies.kyverno.io/v1alpha1
kind: ValidatingPolicy
metadata:
  name: require-run-as-non-root-user
  annotations:
    policies.kyverno.io/title: Require Run As Non-Root User in ValidatingPolicy
    policies.kyverno.io/category: Pod Security Standards (Restricted) in ValidatingPolicy
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/minversion: 1.14.0
    kyverno.io/kyverno-version: 1.14.0
    kyverno.io/kubernetes-version: "1.30+"
    policies.kyverno.io/description: >-
      Containers must be required to run as non-root users. This policy ensures
      `runAsUser` is either unset or set to a number greater than zero.
spec:
  validationActions:
     - Audit
  evaluation:
    background:
      enabled: true
  matchConstraints:
    resourceRules:
      - apiGroups:   [""]
        apiVersions: ["v1"]
        operations:  ["CREATE", "UPDATE"]
        resources:   ["pods"]
  variables:
  - name: allContainers
    expression: >-
      object.spec.containers + 
      object.spec.?initContainers.orValue([]) + 
      object.spec.?ephemeralContainers.orValue([])
  validations:
  - expression: >-
      object.spec.?securityContext.?runAsUser.orValue(1) > 0
    message: >-
      Running as root is not allowed. The field spec.securityContext.runAsUser 
      must be unset or set to a number greater than zero.

  - expression: >-
      variables.allContainers.all(container, 
        container.?securityContext.?runAsUser.orValue(1) > 0)
    message: >-
      Running as root is not allowed. The field spec.containers[*].securityContext.runAsUser, 
      spec.initContainers[*].securityContext.runAsUser, and 
      spec.ephemeralContainers[*].securityContext.runAsUser must be unset or set to a number greater than zero.

```
