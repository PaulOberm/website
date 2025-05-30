---
title: "Disallow use of the SecurityContextConstraint (SCC) anyuid"
category: Security
version: 1.6.0
subject: Role,ClusterRole,RBAC
policyType: "validate"
description: >
    Disallow the use of the SecurityContextConstraint (SCC) anyuid which allows a pod to run with the UID as declared in the image instead of a random UID
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//openshift/disallow-security-context-constraint-anyuid/disallow-security-context-constraint-anyuid.yaml" target="-blank">/openshift/disallow-security-context-constraint-anyuid/disallow-security-context-constraint-anyuid.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: disallow-security-context-constraint-anyuid
  annotations:
    policies.kyverno.io/title: Disallow use of the SecurityContextConstraint (SCC) anyuid
    policies.kyverno.io/category: Security
    policies.kyverno.io/severity: high
    kyverno.io/kyverno-version: 1.6.0
    policies.kyverno.io/minversion: 1.6.0
    kyverno.io/kubernetes-version: "1.20"
    policies.kyverno.io/subject: Role,ClusterRole,RBAC
    policies.kyverno.io/description: >-
      Disallow the use of the SecurityContextConstraint (SCC) anyuid which allows a pod to run with the UID as declared in the image instead of a random UID
spec:
  validationFailureAction: Enforce
  background: true
  rules:
  - name: check-security-context-constraint
    match:
      any:
      - resources:
          kinds:
          - ClusterRole
          - Role
    validate:
      message: >-
        Use of the SecurityContextConstraint (SCC) anyuid is not allowed
      foreach:
      - list: request.object.rules[]
        deny:
          conditions:
            all:
            - key: anyuid
              operator: AnyIn
              value: "{{element.resourceNames[]}}"
            - key: "{{ element.verbs[]  | contains(@, 'use') || contains(@, '*') }}"
              operator: Equals
              value: true
  - name: check-security-context-roleref
    match:
      any:
      - resources:
          kinds:
          - ClusterRoleBinding
          - RoleBinding
    validate:
      message: >-
        Use of the SecurityContextConstraint (SCC) anyuid is not allowed
      deny:
        conditions:
          all:
          - key: system:openshift:scc:anyuid
            operator: Equals
            value: "{{request.object.roleRef.name}}"

```
