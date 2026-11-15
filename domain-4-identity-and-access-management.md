# Domain 4: Identity and Access Management

### NOTE:
A general IAM entity ARN looks like:
```
arn:aws:iam::<account-id>:<entity-type>/<division>/<sub-division>/<entity-name-or-id>
```

1. In an IAM policy, you cannot use special characters (NOT even underscore) in "Sid", but you can in "id"

2. Significance of the below statement is that the policy explicitly denies every non-IAM action when MFA is NOT present:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "NotAction": "iam:*",
            "Resource": "*",
            "Condition": {
                "BoolIfExists": {
                    "aws:MultiFactorAuthPresent": "false"
                }
            }
        }
    ]
}
```