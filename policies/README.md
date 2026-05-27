# Lab 3.3 Policy Library

This directory contains the Lab 3.3 Policy-as-Code library for evaluating Terraform plan JSON against GRC control requirements.

The policies are written in Rego for Open Policy Agent (OPA). Each policy includes a `# METADATA` block that maps the rule to a control, severity level, and remediation guidance. The test files in `policies/tests/` include at least one passing fixture and one failing fixture per policy.

## Purpose

The goal of this policy library is to demonstrate how compliance requirements can be expressed as automated checks before infrastructure is deployed.

The workflow is:

1. Generate a Terraform plan.
2. Convert the plan to JSON.
3. Evaluate the plan with OPA.
4. Return deny messages for non-compliant resources.
5. Fix the Terraform and re-run the evaluation until all deny sets are empty.

This creates a developer feedback loop where compliance issues are caught before resources are applied.

## Policy Summary

| Policy File | Package | Control | Severity | What It Checks | Remediation |
|---|---|---:|---|---|---|
| `sc28_encryption.rego` | `compliance.sc28` | SC-28 | High | Every `google_storage_bucket` must have a customer-managed encryption key configured through an `encryption` block. | Add `encryption { default_kms_key_name = ... }` to the bucket using a controlled KMS key. |
| `ac3_no_public.rego` | `compliance.ac3` | AC-3 | Critical | GCS buckets must enforce uniform bucket-level access and public access prevention. Firewall rules must not expose management ports like 22 or 3389 to public source ranges. | Set `uniform_bucket_level_access = true`, set `public_access_prevention = "enforced"`, and narrow firewall `source_ranges` or remove public management access. |
| `cm6_required_tags.rego` | `compliance.cm6` | CM-6 | Medium | Labelable resources must include the required compliance labels: `project`, `environment`, `managed_by`, and `compliance_scope`. | Add the missing required labels to the resource. |

## Directory Structure

```text
policies/
├── README.md
├── sc28_encryption.rego
├── ac3_no_public.rego
├── cm6_required_tags.rego
└── tests/
    ├── sc28_encryption_test.rego
    ├── ac3_no_public_test.rego
    └── cm6_required_tags_test.rego