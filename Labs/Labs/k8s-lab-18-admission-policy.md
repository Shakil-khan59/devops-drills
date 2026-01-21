# Kubernetes Lab 18: Admission Controllers and Policy

This lab explores admission controls, policy enforcement, and guardrails.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: ValidatingAdmissionPolicy (If Supported)
4. Lab 2: Enforce Image Tag Policy (Concept)
5. Lab 3: Gatekeeper/Kyverno (Optional)
6. Lab 4: Audit Mode and Exceptions
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
- Kubernetes v1.26+ for ValidatingAdmissionPolicy

```bash
kubectl create ns policy-lab
kubectl config set-context --current --namespace=policy-lab
```

---

## 2. Core Concepts
- **Admission:** Validates or mutates requests before persistence
- **Policy engines:** Gatekeeper or Kyverno enforce rules

---

## 3. Lab 1: ValidatingAdmissionPolicy (If Supported)

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingAdmissionPolicy
metadata:
  name: deny-latest-tag
spec:
  failurePolicy: Fail
  matchConstraints:
    resourceRules:
    - apiGroups: [""]
      apiVersions: ["v1"]
      operations: ["CREATE"]
      resources: ["pods"]
  validations:
  - expression: "!has(object.spec.containers) || object.spec.containers.all(c, !c.image.endsWith(':latest'))"
    message: "Image tag 'latest' is not allowed."
```

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingAdmissionPolicyBinding
metadata:
  name: deny-latest-binding
spec:
  policyName: deny-latest-tag
  validationActions: ["Deny"]
```

---

## 4. Lab 2: Enforce Image Tag Policy (Concept)

Try to create a pod with `nginx:latest` and observe denial.

---

## 5. Lab 3: Gatekeeper/Kyverno (Optional)

> If installed, apply a constraint/policy to enforce labels or resource limits.

---

## 6. Lab 4: Audit Mode and Exceptions

> Use policy bindings with `Audit` action or namespace exclusions for controlled rollout.

---

## 7. Troubleshooting Scenarios
- **Policy not enforced:** Controller not enabled or version unsupported.
- **Unexpected blocks:** Check matchConstraints and object selectors.
- **Audit noise:** Narrow scope or add exclusions.

---

## 8. Interview Questions and Answers

### Q1: What is admission control?
**Answer:** A gate that validates or mutates API requests before persistence.

### Q2: Why avoid image:latest?
**Answer:** It breaks repeatability and can pull unexpected versions.

### Q3: How to rollout new policies safely?
**Answer:** Use audit mode and narrow scope before enforcing deny.

---

## 9. Cleanup
```bash
kubectl delete ns policy-lab
kubectl delete validatingadmissionpolicy deny-latest-tag
kubectl delete validatingadmissionpolicybinding deny-latest-binding
```

---

## 10. Theory (Concise)
- **Admission is a control plane guardrail** for API changes.
- **Policies reduce risk** by enforcing standards.
- **Audit-first rollout** prevents outages.
