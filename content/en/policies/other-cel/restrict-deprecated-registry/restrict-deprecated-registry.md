---
title: "Restrict Deprecated Registry in CEL expressions"
category: Best Practices, EKS Best Practices in CEL
version: 1.11.0
subject: Pod
policyType: "validate"
description: >
    Legacy k8s.gcr.io container image registry will be frozen in early April 2023 k8s.gcr.io image registry will be frozen from the 3rd of April 2023.   Images for Kubernetes 1.27 will not be available in the k8s.gcr.io image registry. Please read our announcement for more details. https://kubernetes.io/blog/2023/02/06/k8s-gcr-io-freeze-announcement/     
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other-cel/restrict-deprecated-registry/restrict-deprecated-registry.yaml" target="-blank">/other-cel/restrict-deprecated-registry/restrict-deprecated-registry.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: restrict-deprecated-registry
  annotations:
    policies.kyverno.io/title: Restrict Deprecated Registry in CEL expressions
    policies.kyverno.io/category: Best Practices, EKS Best Practices in CEL 
    policies.kyverno.io/severity: high
    policies.kyverno.io/minversion: 1.11.0
    kyverno.io/kubernetes-version: "1.27-1.28"
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      Legacy k8s.gcr.io container image registry will be frozen in early April 2023
      k8s.gcr.io image registry will be frozen from the 3rd of April 2023.  
      Images for Kubernetes 1.27 will not be available in the k8s.gcr.io image registry.
      Please read our announcement for more details.
      https://kubernetes.io/blog/2023/02/06/k8s-gcr-io-freeze-announcement/     
spec:
  validationFailureAction: Enforce
  background: true
  rules:
  - name: restrict-deprecated-registry
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
        expressions:
          - expression: "variables.allContainers.all(container, !container.image.startsWith('k8s.gcr.io/'))"
            message: "The \"k8s.gcr.io\" image registry is deprecated. \"registry.k8s.io\" should now be used."
    

```
