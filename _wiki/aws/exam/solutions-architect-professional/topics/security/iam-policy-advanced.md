---
layout: wiki
title: IAM Policy Advanced
permalink: /wiki/aws/exam/solutions-architect-professional/topics/security/iam-policy-advanced/
tags: [aws, iam, security, exam, sap]
---

## Condition Operators

| Operator | Example |
|----------|---------|
| `StringEquals` | `"aws:username": "admin"` |
| `StringLike` | `"s3:prefix": "home/*"` |
| `NumericLessThan` | `"s3:max-keys": "10"` |
| `DateGreaterThan` | `"aws:CurrentTime": "2024-01-01"` |
| `Bool` | `"aws:SecureTransport": "true"` |
| `IpAddress` | `"aws:SourceIp": "192.168.0.0/24"` |
| `ArnLike` | `"aws:SourceArn": "arn:aws:s3:::bucket/*"` |
| `Null` | `"aws:TokenIssueTime": "true"` |

Allow access only from specific IP range:
```json
{
  "Effect": "Allow",
  "Action": "s3:*",
  "Resource": "*",
  "Condition": {
    "IpAddress": {"aws:SourceIp": "10.0.0.0/8"}
  }
}
```

### Set Modifiers

| Modifier | Use |
|----------|-----|
| `ForAllValues:` | All values must match |
| `ForAnyValue:` | At least one must match |
| `IfExists` | Apply only if key exists |

Allow only if all requested tags are in allowed list:
```json
{
  "Condition": {
    "ForAllValues:StringEquals": {
      "aws:TagKeys": ["env", "team", "owner"]
    }
  }
}
```

---

## Global Condition Keys

| Key | Description |
|-----|-------------|
| `aws:SourceIp` | Requester IP |
| `aws:SourceVpc` | VPC ID |
| `aws:SourceVpce` | VPC endpoint ID |
| `aws:PrincipalArn` | Caller ARN |
| `aws:PrincipalOrgID` | Organization ID |
| `aws:RequestedRegion` | Target region |
| `aws:SecureTransport` | HTTPS used |
| `aws:MultiFactorAuthPresent` | MFA used |
| `aws:CurrentTime` | Request time |

Deny access outside allowed regions:
```json
{
  "Effect": "Deny",
  "Action": "*",
  "Resource": "*",
  "Condition": {
    "StringNotEquals": {
      "aws:RequestedRegion": ["eu-west-1", "eu-central-1"]
    }
  }
}
```

---

## Policy Variables

**Syntax:** `${variable}`

| Variable | Example |
|----------|---------|
| `${aws:username}` | IAM user name |
| `${aws:userid}` | Unique user ID |
| `${aws:PrincipalTag/key}` | Tag on caller |
| `${s3:prefix}` | S3 object prefix |

Give users access to their own S3 folder:
```json
{
  "Effect": "Allow",
  "Action": "s3:*",
  "Resource": [
    "arn:aws:s3:::company-bucket/${aws:username}/*"
  ]
}
```

---

## Tag Condition Keys

| Key | Use |
|-----|-----|
| `aws:RequestTag/key` | Tag in create request |
| `aws:ResourceTag/key` | Tag on resource |
| `aws:PrincipalTag/key` | Tag on caller |
| `aws:TagKeys` | List of tag keys |

Allow EC2 access only to resources with matching team tag (ABAC):
```json
{
  "Effect": "Allow",
  "Action": "ec2:*",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "aws:ResourceTag/team": "${aws:PrincipalTag/team}"
    }
  }
}
```

---

## Common Patterns

**Require MFA:**
```json
"Condition": {"Bool": {"aws:MultiFactorAuthPresent": "true"}}
```

**Org Only Access:**
```json
"Condition": {"StringEquals": {"aws:PrincipalOrgID": "o-xxxxxxxxxx"}}
```

**VPC Endpoint Only:**
```json
"Condition": {"StringEquals": {"aws:SourceVpce": "vpce-1234567890abcdef0"}}
```

**Require HTTPS:**
```json
"Condition": {"Bool": {"aws:SecureTransport": "true"}}
```
