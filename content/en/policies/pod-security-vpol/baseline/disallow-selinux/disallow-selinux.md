---
title: "Disallow SELinux in ValidatingPolicy"
category: Pod Security Standards (Baseline) in ValidatingPolicy
version: 1.14.0
subject: Pod
policyType: "validate"
description: >
    SELinux options can be used to escalate privileges and should not be allowed. This policy ensures that the `seLinuxOptions` field is undefined.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//pod-security-vpol/baseline/disallow-selinux/disallow-selinux.yaml" target="-blank">/pod-security-vpol/baseline/disallow-selinux/disallow-selinux.yaml</a>

```yaml
apiVersion: policies.kyverno.io/v1alpha1
kind: ValidatingPolicy
metadata:
  name: disallow-selinux
  annotations:
    policies.kyverno.io/title: Disallow SELinux in ValidatingPolicy
    policies.kyverno.io/category: Pod Security Standards (Baseline) in ValidatingPolicy
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/minversion: 1.14.0
    kyverno.io/kyverno-version: 1.14.0
    kyverno.io/kubernetes-version: "1.30+"
    policies.kyverno.io/description: >-
      SELinux options can be used to escalate privileges and should not be allowed. This policy
      ensures that the `seLinuxOptions` field is undefined.
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
  - name: allContainerTypes
    expression: >-
      object.spec.containers + 
      object.spec.?initContainers.orValue([]) + 
      object.spec.?ephemeralContainers.orValue([])

  - name: seLinuxTypes
    expression: "['container_t', 'container_init_t', 'container_kvm_t']"

  - name: hasValidSELinuxType
    expression: >-
      object.spec.?securityContext.?seLinuxOptions.?type.orValue('') == '' || 
      variables.seLinuxTypes.exists(type, type == object.spec.securityContext.seLinuxOptions.type)

  - name: hasValidSELinuxUserRole
    expression: >-
      object.spec.?securityContext.?seLinuxOptions.?user.orValue('') == '' &&
      object.spec.?securityContext.?seLinuxOptions.?role.orValue('') == ''

  validations:
  - expression: >-
      variables.hasValidSELinuxType &&
      variables.allContainerTypes.all(container, 
        container.?securityContext.?seLinuxOptions.?type.orValue('') == '' || 
        variables.seLinuxTypes.exists(type, type == container.securityContext.seLinuxOptions.type))
    message: >-
      Setting the SELinux type is restricted. The field securityContext.seLinuxOptions.type 
      must either be unset or set to one of the allowed values (container_t, container_init_t, or container_kvm_t).

  - expression: >-
      variables.hasValidSELinuxUserRole &&
      variables.allContainerTypes.all(container,
        container.?securityContext.?seLinuxOptions.?user.orValue('') == '' &&
        container.?securityContext.?seLinuxOptions.?role.orValue('') == '')
    message: >-
      Setting the SELinux user or role is forbidden. The fields seLinuxOptions.user and seLinuxOptions.role must be unset.

```
