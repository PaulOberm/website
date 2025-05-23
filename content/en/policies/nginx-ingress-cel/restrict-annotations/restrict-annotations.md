---
title: "Restrict NGINX Ingress annotation values in CEL expressions"
category: Security, NGINX Ingress in CEL
version: 1.11.0
subject: Ingress
policyType: "validate"
description: >
    This policy mitigates CVE-2021-25746 by restricting `metadata.annotations` to safe values. See: https://github.com/kubernetes/ingress-nginx/blame/main/internal/ingress/inspector/rules.go. This issue has been fixed in NGINX Ingress v1.2.0. For NGINX Ingress version 1.0.5+ the  "annotation-value-word-blocklist" configuration setting is also recommended.  Please refer to the CVE for details. 
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//nginx-ingress-cel/restrict-annotations/restrict-annotations.yaml" target="-blank">/nginx-ingress-cel/restrict-annotations/restrict-annotations.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: restrict-annotations
  annotations:
    policies.kyverno.io/title: Restrict NGINX Ingress annotation values in CEL expressions 
    policies.kyverno.io/category: Security, NGINX Ingress in CEL 
    policies.kyverno.io/severity: high
    policies.kyverno.io/subject: Ingress
    policies.kyverno.io/minversion: "1.11.0"
    kyverno.io/kyverno-version: "1.11.0"
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/description: >-
      This policy mitigates CVE-2021-25746 by restricting `metadata.annotations` to safe values.
      See: https://github.com/kubernetes/ingress-nginx/blame/main/internal/ingress/inspector/rules.go.
      This issue has been fixed in NGINX Ingress v1.2.0. For NGINX Ingress version 1.0.5+ the 
      "annotation-value-word-blocklist" configuration setting is also recommended. 
      Please refer to the CVE for details. 
spec:
  validationFailureAction: Enforce
  rules:
    - name: check-ingress
      match:
        any:
        - resources:
            kinds:
            - networking.k8s.io/v1/Ingress
            operations:
            - CREATE
            - UPDATE
      validate:
        cel:
          expressions:
            - expression: >-
                !has(object.metadata.annotations) ||
                (
                  !object.metadata.annotations.exists(annotation, object.metadata.annotations[annotation].matches('\\s*alias\\s*.*;')) &&
                  !object.metadata.annotations.exists(annotation, object.metadata.annotations[annotation].matches('\\s*root\\s*.*;')) &&
                  !object.metadata.annotations.exists(annotation, object.metadata.annotations[annotation].matches('/etc/(passwd|shadow|group|nginx|ingress-controller)')) &&
                  !object.metadata.annotations.exists(annotation, object.metadata.annotations[annotation].matches('/var/run/secrets')) &&
                  !object.metadata.annotations.exists(annotation, object.metadata.annotations[annotation].matches('.*_by_lua.*'))
                )
              message: "spec.rules[].http.paths[].path value is not allowed"


```
