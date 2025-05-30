---
title: "Check NVIDIA GPUs"
category: Other
version: 1.6.0
subject: Pod
policyType: "validate"
description: >
    Containers which request use of an NVIDIA GPU often need to be authored to consume them via a CUDA environment variable called NVIDIA_VISIBLE_DEVICES. This policy checks the containers which request a GPU to ensure they have been authored with this environment variable.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/check-nvidia-gpu/check-nvidia-gpu.yaml" target="-blank">/other/check-nvidia-gpu/check-nvidia-gpu.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: check-nvidia-gpus
  annotations:
    policies.kyverno.io/title: Check NVIDIA GPUs
    policies.kyverno.io/category: Other
    policies.kyverno.io/severity: medium
    kyverno.io/kyverno-version: 1.6.0
    policies.kyverno.io/minversion: 1.6.0
    kyverno.io/kubernetes-version: "1.23"
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      Containers which request use of an NVIDIA GPU often need to
      be authored to consume them via a CUDA environment variable called
      NVIDIA_VISIBLE_DEVICES. This policy checks the containers which
      request a GPU to ensure they have been authored with this environment
      variable.
spec:
  validationFailureAction: Audit
  rules:
  - name: check-nvidia-gpus
    match:
      any:
      - resources:
          kinds:
          - Pod
    preconditions:
      all:
      - key: "{{request.operation || 'BACKGROUND'}}"
        operator: NotEquals
        value: DELETE
    validate:
      message: "Images which reserve NVIDIA GPUs must be built to use them."
      foreach:
      - list: "request.object.spec.containers"
        context: 
        - name: imageData
          imageRegistry: 
            reference: "{{ element.image }}"
        deny:
          conditions:
            all:
              # If a container image calls for an NVIDIA GPU in its resources.limits, it must also
              # have been built with the CUDA environment variable `NVIDIA_VISIBLE_DEVICES`.
              - key: "NVIDIA_VISIBLE_DEVICES=*?"
                operator: AnyNotIn
                value: "{{ imageData.configData.config.Env || '' }}"
              - key: "{{ element.resources.limits.\"nvidia.com/gpu\" || '' }}"
                operator: GreaterThan
                value: 0
```
