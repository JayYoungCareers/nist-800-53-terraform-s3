# NIST 800-53 Controls as Terraform Resources

Terraform for a single S3 bucket (plus its companion access-log bucket) that
satisfies five NIST 800-53 controls, with machine-readable evidence of each.

Deployed bucket: `cgep-lab-1-dev-data-a2107ca2` (`us-east-1`, account `795782340501`).

## Control mapping

| Control | Requirement | Implementation |
|---|---|---|
| SC-28 | Protection of information at rest | `aws_s3_bucket_server_side_encryption_configuration` — AES-256 SSE on both buckets |
| AU-3 | Content of audit records | `aws_s3_bucket_logging` on the primary bucket targets the log bucket with prefix `access-logs/` |
| AU-6 | Audit review, analysis, and reporting | Dedicated `log` bucket with `log-delivery-write` ACL and ownership controls so access logs are collected for review |
| CM-6 | Configuration settings | `default_tags` on the provider enforce compliance tags on every taggable resource; `aws_s3_bucket_versioning` preserves prior object states |
| AC-3 | Access enforcement | `aws_s3_bucket_public_access_block` on both buckets blocks/ignores public ACLs and policies |

## Layout

```
terraform/primitives/compliant-s3/   Terraform source (main.tf, variables.tf, outputs.tf)
evidence/plan.json                   terraform show -json of the last plan
evidence/state.json                  terraform show -json of the last apply (machine-readable control evidence)
```

`evidence/*.json` is the machine-readable proof each control was actually applied — generated via
`terraform show -json`, not hand-written. Local state (`.tfstate`) and plan binaries are not committed
(see `.gitignore`); only the JSON evidence exports are tracked.

## Verified

Bucket confirmed live in AWS on 2026-08-15: versioning Enabled, encryption SSE-S3 (AES256), and tags
(`ComplianceScope`, `environment`, `ManagedBy`, `project_name`) all match this Terraform config.
