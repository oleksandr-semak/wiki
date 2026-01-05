---
layout: wiki
title: IAM Permissions Boundaries
permalink: /wiki/aws/topics/security/iam-permissions-boundaries/
tags: [aws, iam, security, permissions]
---

## The Problem

Organizations need to delegate IAM administration to developers or teams while preventing privilege escalation. Without guardrails, delegated administrators could grant themselves or others more permissions than intended — potentially gaining full administrative access.

> **Warning:** Without permissions boundaries, a user with `iam:CreateUser` and `iam:AttachUserPolicy` permissions could create a new user and attach the `AdministratorAccess` policy, effectively escalating privileges beyond their intended scope.
{: .warning}

## The Solution: IAM Permissions Boundaries

**Permissions Boundaries** are an advanced IAM feature that sets the maximum permissions an IAM entity (user or role) can have. They act as a guardrail — even if an identity policy grants broader permissions, the effective permissions are limited to the intersection of both policies.

## How It Works

The effective permissions are calculated as:

```
Effective Permissions = Identity Policy ∩ Permissions Boundary
```

| Policy Type | Purpose |
|-------------|---------|
| Identity Policy | What the entity *wants* to do |
| Permissions Boundary | What the entity *is allowed* to do (maximum) |
| Effective Permissions | The intersection of both |

## Key Use Cases

1. **Delegated Administration** — Allow developers to create IAM roles for their applications without granting full IAM admin access
2. **Self-Service IAM** — Enable teams to manage their own users/roles within defined boundaries
3. **Prevent Privilege Escalation** — Ensure delegated admins cannot grant permissions beyond their own scope
4. **Multi-Tenant Environments** — Isolate permissions between different teams or projects

## Example Scenario

A central admin wants developers to create Lambda execution roles:

1. Create a permissions boundary policy limiting actions to specific services (e.g., S3, DynamoDB)
2. Grant developers `iam:CreateRole` and `iam:AttachRolePolicy` with a condition requiring the boundary
3. Developers can create roles, but those roles cannot exceed the boundary permissions

## Practical Example

### Step 1: Create the Permissions Boundary

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "*"
    }
  ]
}
```

### Step 2: Grant Developer IAM Permissions (with boundary enforcement)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "iam:CreateRole",
        "iam:AttachRolePolicy",
        "iam:PutRolePolicy"
      ],
      "Resource": "arn:aws:iam::123456789012:role/dev-*",
      "Condition": {
        "StringEquals": {
          "iam:PermissionsBoundary": "arn:aws:iam::123456789012:policy/DevBoundary"
        }
      }
    }
  ]
}
```

### Step 3: Developer Creates a Role

Developer creates `dev-lambda-role` and attaches `AmazonS3FullAccess`:

| Policy | Allows |
|--------|--------|
| Identity Policy (S3FullAccess) | `s3:*` on all resources |
| Permissions Boundary | `s3:GetObject`, `s3:PutObject` only |
| **Effective Permissions** | `s3:GetObject`, `s3:PutObject` only |

Even though `AmazonS3FullAccess` grants `s3:DeleteObject`, the boundary blocks it.

## Best Practices

- Always require permissions boundaries when delegating IAM administration
- Use condition keys like `iam:PermissionsBoundary` to enforce boundary attachment
- Combine with SCPs for organization-wide guardrails
- Test boundary policies thoroughly before deployment

---

> **Read the full guide:** [When and where to use IAM permissions boundaries](https://aws.amazon.com/blogs/security/when-and-where-to-use-iam-permissions-boundaries/) — AWS Security Blog
{: .article-link}
