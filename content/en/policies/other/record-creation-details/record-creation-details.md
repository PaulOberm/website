---
title: "Record Creation Details"
category: Other
version: 1.6.0
subject: Annotation
policyType: "mutate"
description: >
    Kubernetes by default does not make a record of who or what created a resource in that resource itself. It must be retrieved from an audit log, if enabled, which can make it difficult for cluster operators to know who was responsible for an object's creation. This policy writes an annotation with the key `kyverno.io/created-by` having all the userInfo fields present in the AdmissionReview request for any object being created. It then protects this annotation from tampering or removal making it immutable. Although this policy matches on all kinds ("*") it is highly recommend to more narrowly scope it to only the resources which should be labeled.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/record-creation-details/record-creation-details.yaml" target="-blank">/other/record-creation-details/record-creation-details.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: record-creation-details
  annotations:
    policies.kyverno.io/title: Record Creation Details
    policies.kyverno.io/category: Other
    policies.kyverno.io/severity: medium
    kyverno.io/kyverno-version: 1.6.2
    policies.kyverno.io/minversion: 1.6.0
    kyverno.io/kubernetes-version: "1.23"
    policies.kyverno.io/subject: Annotation
    policies.kyverno.io/description: >-
      Kubernetes by default does not make a record of who or what
      created a resource in that resource itself. It must be retrieved from
      an audit log, if enabled, which can make it difficult for cluster
      operators to know who was responsible for an object's creation.
      This policy writes an annotation with the key `kyverno.io/created-by`
      having all the userInfo fields present in the AdmissionReview request
      for any object being created. It then protects this annotation from
      tampering or removal making it immutable. Although this policy matches on
      all kinds ("*") it is highly recommend to more narrowly scope it to only
      the resources which should be labeled.
spec:
  validationFailureAction: Enforce
  background: false
  rules:
  - name: add-userinfo
    match:
      any:
      - resources:
          kinds:
          - '*'
    preconditions:
      any:
      - key: "{{request.operation || 'BACKGROUND'}}"
        operator: Equals
        value: CREATE
    mutate:
      patchStrategicMerge:
        metadata:
          annotations:
            kyverno.io/created-by: "{{ request.userInfo.{groups: groups, username: username} | to_string(@) }}"
  - name: prevent-updates-deletes-userinfo-annotations
    match:
      any:
      - resources:
          kinds:
          - '*'
    preconditions:
      any:
      - key: "{{request.operation || 'BACKGROUND'}}"
        operator: Equals
        value: UPDATE
      - key: "{{ request.oldObject.metadata.annotations.\"kyverno.io/created-by\" || '' }}"
        operator: Equals
        value: "?*"
    validate:
      message: The annotation kyverno.io/created-by is protected and may not be altered or removed.
      deny:
        conditions:
          any:
          - key: "{{ request.object.metadata.annotations.\"kyverno.io/created-by\" || '' }}"
            operator: Equals
            value: ""
          - key: "{{ request.object.metadata.annotations.\"kyverno.io/created-by\" || '' }}"
            operator: NotEquals
            value: "{{ request.oldObject.metadata.annotations.\"kyverno.io/created-by\" }}"

```
