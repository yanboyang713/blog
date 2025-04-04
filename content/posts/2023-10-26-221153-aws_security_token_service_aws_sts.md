---
title: "AWS Security Token Service (AWS STS)"
draft: false
---

The [AWS]({{< relref "20230314144515-aws.md" >}}) STS is a web service that enables you to request temporary, limited-privilege credentials for [IAM Users]({{< relref "2023-10-26-011820-aws_identity_and_access_management_iam.md#iam-users" >}}) or for users that you authenticate (federated users).

Temporary security credentials work almost identically to long-term access key credentials that [IAM Users]({{< relref "2023-10-26-011820-aws_identity_and_access_management_iam.md#iam-users" >}}) can use, with the following differences:

-   Temporary security credentials are short-term.
-   They can be configured to last anywhere from a few minutes to several hours.
-   After the credentials expire, AWS no longer recognizes them or allows any kind of access to API requests made with them.
-   Temporary security credentials are not stored with the user but are generated dynamically and provided to the user when requested.
-   When (or even before) the temporary security credentials expire, the user can request new credentials, if the user requesting them still has permission to do so.


## Advantages of STS are: {#advantages-of-sts-are}

-   You do not have to distribute or embed long-term AWS security credentials with an application.
-   You can provide access to your AWS resources to users without having to define an AWS identity for them (temporary security credentials are the basis for IAM Roles and ID Federation).
-   The temporary security credentials have a limited lifetime, so you do not have to rotate them or explicitly revoke them when they’re no longer needed.
-   After temporary security credentials expire, they cannot be reused (you can specify how long the credentials are valid for, up to a maximum limit)

Users can come from three sources.

1.  Federation (typically AD):
2.  Uses SAML 2.0.
3.  Grants temporary access based on the users AD credentials.
4.  Does not need to be a user in IAM.
5.  Single sign-on allows users to login to the AWS console without assigning IAM credentials.
6.  Federation with Mobile Apps:
7.  Use Facebook/Amazon/Google or other OpenID providers to login.
8.  Cross Account Access:
9.  Allows users from one AWS account access resources in another.
10. To make a request in a different account the resource in that account must have an attached resource-based policy with the permissions you need.
11. Or you must assume a role (identity-based policy) within that account with the permissions you need.
