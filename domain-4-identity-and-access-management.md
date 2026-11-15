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

3. IAM follows:  
<mark>**Explicit DENY > Explicit ALLOW > Implicit DENY**</mark>

4. Policy Structure
```
{
  "Version": "2012-10-17",
  "Id": "a3f1c2e4-7b9d-4c1a-9e2f-5d6b8a0c1e3f",
  "Statement": [
    {
      "Sid": "AllowOrgReadsOfOwnTeamPrefixOverTLS",
      "Effect": "Allow",
      "Principal": {
        "AWS": [
          "arn:aws:iam::111122223333:role/analytics-role",
          "arn:aws:iam::444455556666:root"
        ]
      },
      "Action": [
        "s3:GetObject", 
        "s3:GetObjectVersion"
        ],
      "Resource": "arn:aws:s3:::reports-bucket/${aws:PrincipalTag/team}/*",
      "Condition": {
        "StringEquals": { "aws:PrincipalOrgID": "o-a1b2c3d4e5" },
        "Bool": { "aws:SecureTransport": "true" },
        "DateLessThan": { "aws:CurrentTime": "2027-03-31T23:59:59Z" }
      }
    },
    {
      "Sid": "DenyAllButReadsForEveryoneExceptAdmin",
      "Effect": "Deny",
      "Principal": "*",
      "NotAction": [
        "s3:GetObject", 
        "s3:GetObjectVersion", 
        "s3:ListBucket"
        ],
      "Resource": [
        "arn:aws:s3:::reports-bucket",
        "arn:aws:s3:::reports-bucket/*"
      ],
      "Condition": {
        "ArnNotEquals": { "aws:PrincipalArn": "arn:aws:iam::111122223333:role/reports-admin" }
      }
    },
    {
      "Sid": "DenyDeletesOutsideTmpPrefix",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:DeleteObject",
      "NotResource": "arn:aws:s3:::reports-bucket/tmp/*"
    }
  ]
}
```

**NOTE:**  
"Id" allows alphanumeric characters, hyphens, and underscores.
"Sid" only allows alphanumeric characters

Case sensitivity in IAM policies:  
| Policy element | Case sensitive? | Details |
| --- | --- | --- |
| Effect, Action, Resourcem, Condition | NO | "Effect": "Allow" is same as  "effect": "allow" |
| Action, Service prefixes| NO | iam:ListAccessKeys is same as IAM:listaccesskeys |
| Resource ARNs | YES | |
| Principals | YES | |
| Condition key names | NO | aws:SourceIP is same as AWS:SourceIp |
| Condition values | YES, NO | YES in general. NO when using operators like StringEqualsIgnoreCase |
| Policy variables | NO | Variables names placeholders, like  ${aws:username} | 
| Policy names | NO |  |
| Tags keys | NO |  |
| Tag values | YES, NO | YES in general, unless you're using a IgnoreCase operator |

5. IAM policy effect-action table  
![IAM policy effect-action table](assets/images/iam-policy-effect-action-table.drawio.png)

6. AWS ARN Format  
```
arn:partition:service:region:account-id:resource-id
arn:partition:service:region:account-id:resource-type/resource-id
arn:partition:service:region:account-id:resource-type:resource-id
```

[+] [https://docs.aws.amazon.com/IAM/latest/UserGuide/reference-arns.html](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference-arns.html)

**NOTE:**  
An AWS Account ID is always 12 digits long.

7. Specifying the root user in an IAM policy:  
```
"Principal": {
    "AWS": "arn:aws:iam::123456789012:root"
}
```

8. IAM Role sessions
```
"Principal": { "AWS": "arn:aws:sts::111111111111:assumed-role/role-name/role-session-name" }

"Principal": { "Federated": "cognito-identity.amazonaws.com" }

"Principal": { "Federated": "arn:aws:iam::111111111111:saml-provider/provider-name" }
```
9. When to use IAM user vs. IAM role  

* An IAM user represents a specific individual or application that has long term credentials like access keys or passwords, needed for persistent access

* An IAM role is a temporary identity, assumed by entities like users, applications or AWS services, for temporary credentials via sts:AssumeRole, meant for short-term access

**NOTE:**  
IAM best practices recommend that you require human users to use federation with an identity provider to access AWS using temporary credentials instead of using IAM users with long-term credentials. We recommend that you only use IAM users for specific use cases not supported by federated users.

[+] [https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html)

10. IAM Role names must be unique:  

* For Role name, enter a role name that helps you identify the purpose of this role. Role names must be unique within your AWS account. 
* When a role name is used in a policy or as part of an ARN, the role name is case sensitive. 
When a role name appears to customers in the console, such as during the sign-in process, the role name is case insensitive. 
* Because various entities might reference the role, you can't edit the name of the role after it is created.

11. iam:PassRole  

iam:PassRole is an AWS IAM permission that allows a user, role, or group to pass an existing IAM role to an AWS service (such as EC2, Lambda, or CloudFormation) so the service can assume it and run with its permissions. 

To pass a role, you need 3 things:

* IAM permissions policy attached to the role that determines what the role can do
* A trust policy for the role that allows the service to assume the role
* An IAM permissions policy attached to the IAM user that allows the user to pass only those approved roles

[+] [https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_passrole.html](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_passrole.html)

**NOTE:**  
Unlike most other AWS service actions, IAM PassRole is NOT an API method even though it appears in the Action list of IAM policies.

[+] [https://blog.rowanudell.com/iam-passrole-explained/](https://blog.rowanudell.com/iam-passrole-explained/)

* Actions defined by IAM:  
[+] [https://docs.aws.amazon.com/service-authorization/latest/reference/list_iam.html#list_iam-actions-as-permissions](https://docs.aws.amazon.com/service-authorization/latest/reference/list_iam.html#list_iam-actions-as-permissions)

* Permission only actions defined by IAM:  
[+] [https://docs.aws.amazon.com/service-authorization/latest/reference/list_iam.html#list_iam-permission-only-actions](https://docs.aws.amazon.com/service-authorization/latest/reference/list_iam.html#list_iam-permission-only-actions)

iam:PassRole can be used to prevent the confused deputy problem:  
[+] [https://blog.rowanudell.com/iam-passrole-explained/](https://blog.rowanudell.com/iam-passrole-explained/)

BUT official AWS’ guidance for preventing the confused deputy problem is using “aws:SourceArn” / “aws:SourceAccount conditions and “sts:ExternalId” for third parties:  
[+] [https://docs.aws.amazon.com/IAM/latest/UserGuide/confused-deputy.html](https://docs.aws.amazon.com/IAM/latest/UserGuide/confused-deputy.html)





