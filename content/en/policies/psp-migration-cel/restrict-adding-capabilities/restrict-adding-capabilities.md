---
title: "Restrict Adding Capabilities in CEL expressions"
category: PSP Migration in CEL
version: 1.11.0
subject: Pod
policyType: "validate"
description: >
    Adding capabilities is a way for containers in a Pod to request higher levels of ability than those with which they may be provisioned. Many capabilities allow system-level control and should be prevented. Pod Security Policies (PSP) allowed a list of "good" capabilities to be added. This policy checks ephemeralContainers, initContainers, and containers to ensure the only capabilities that can be added are either NET_BIND_SERVICE or CAP_CHOWN.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//psp-migration-cel/restrict-adding-capabilities/restrict-adding-capabilities.yaml" target="-blank">/psp-migration-cel/restrict-adding-capabilities/restrict-adding-capabilities.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: psp-restrict-adding-capabilities
  annotations:
    policies.kyverno.io/title: Restrict Adding Capabilities in CEL expressions
    policies.kyverno.io/category: PSP Migration in CEL 
    policies.kyverno.io/severity: medium
    kyverno.io/kyverno-version: 1.11.0
    policies.kyverno.io/minversion: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      Adding capabilities is a way for containers in a Pod to request higher levels
      of ability than those with which they may be provisioned. Many capabilities
      allow system-level control and should be prevented. Pod Security Policies (PSP)
      allowed a list of "good" capabilities to be added. This policy checks
      ephemeralContainers, initContainers, and containers to ensure the only
      capabilities that can be added are either NET_BIND_SERVICE or CAP_CHOWN.
spec:
  validationFailureAction: Audit
  background: true
  rules:
    - name: allowed-capabilities
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
          variables:
          - name: allContainers
            expression: "object.spec.containers + object.spec.?initContainers.orValue([]) + object.spec.?ephemeralContainers.orValue([])"
          - name: allowedCapabilities
            expression: "['NET_BIND_SERVICE', 'CAP_CHOWN']"
          expressions:
            - expression: >-
                variables.allContainers.all(container, 
                container.?securityContext.?capabilities.?add.orValue([]).all(capability, capability in variables.allowedCapabilities))
              message: >-
                Any capabilities added other than NET_BIND_SERVICE or CAP_CHOWN are disallowed.


```
