---
title: "Enforce Istio Sidecar Injection in CEL expressions"
category: Istio in CEL
version: 1.11.0
subject: Namespace
policyType: "validate"
description: >
    In order for Istio to inject sidecars to workloads deployed into Namespaces, the label `istio-injection` must be set to `enabled`. This policy ensures that all new Namespaces set `istio-inject` to `enabled`.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//istio-cel/enforce-sidecar-injection-namespace/enforce-sidecar-injection-namespace.yaml" target="-blank">/istio-cel/enforce-sidecar-injection-namespace/enforce-sidecar-injection-namespace.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: enforce-sidecar-injection-namespace
  annotations:
    policies.kyverno.io/title: Enforce Istio Sidecar Injection in CEL expressions
    policies.kyverno.io/category: Istio in CEL 
    policies.kyverno.io/severity: medium
    kyverno.io/kyverno-version: 1.11.0
    policies.kyverno.io/minversion: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/subject: Namespace
    policies.kyverno.io/description: >-
      In order for Istio to inject sidecars to workloads deployed into Namespaces, the label
      `istio-injection` must be set to `enabled`. This policy ensures that all new Namespaces
      set `istio-inject` to `enabled`.
spec:
  validationFailureAction: Audit
  background: true
  rules:
  - name: check-istio-injection-enabled
    match:
      any:
      - resources:
          kinds:
          - Namespace
          operations:
          - CREATE
    validate:
      cel:
        expressions:
          - expression: "object.metadata.?labels[?'istio-injection'].orValue('') == 'enabled'"
            message: "All new Namespaces must have Istio sidecar injection enabled."


```
