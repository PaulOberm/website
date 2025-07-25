---
title: "Verify Image"
category: Software Supply Chain Security, EKS Best Practices
version: 1.14.0
subject: Pod
policyType: "verifyImages"
description: >
    Using the Cosign project, OCI images may be signed to ensure supply chain security is maintained. Those signatures can be verified before pulling into a cluster. This policy checks the signature of an image repo called ghcr.io/kyverno/test-verify-image to ensure it has been signed by verifying its signature against the provided public key. This policy serves as an illustration for how to configure a similar rule and will require replacing with your image(s) and keys.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/verify-image-ivpol/verify-image-ivpol.yaml" target="-blank">/other/verify-image-ivpol/verify-image-ivpol.yaml</a>

```yaml
apiVersion: policies.kyverno.io/v1alpha1
kind: ImageValidatingPolicy
metadata:
  name: verify-image-ivpol
  annotations:
    policies.kyverno.io/title: Verify Image
    policies.kyverno.io/category: Software Supply Chain Security, EKS Best Practices
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/minversion: 1.14.0
    policies.kyverno.io/description: >-
      Using the Cosign project, OCI images may be signed to ensure supply chain
      security is maintained. Those signatures can be verified before pulling into
      a cluster. This policy checks the signature of an image repo called
      ghcr.io/kyverno/test-verify-image to ensure it has been signed by verifying
      its signature against the provided public key. This policy serves as an illustration for
      how to configure a similar rule and will require replacing with your image(s) and keys.
spec:
  webhookConfiguration:
    timeoutSeconds: 30
  evaluation:
   background:
    enabled: false
  validationActions: [Deny]
  matchConstraints:
    resourceRules:
      - apiGroups: [""]
        apiVersions: ["v1"]
        operations: ["CREATE", "UPDATE"]
        resources: ["pods"]
  matchImageReferences:
        - glob : "docker.io/mohdcode/kyverno*"
  attestors:
  - name: cosign
    cosign:
     key:
      data: |
                -----BEGIN PUBLIC KEY-----
                MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE6QsNef3SKYhJVYSVj+ZfbPwJd0pv
                DLYNHXITZkhIzfE+apcxDjCCkDPcJ3A3zvhPATYOIsCxYPch7Q2JdJLsDQ==
                -----END PUBLIC KEY-----
  validations:
    - expression: >-
       images.containers.map(image, verifyImageSignatures(image, [attestors.cosign])).all(e ,e > 0)
      message: >-
       failed the verification

```
