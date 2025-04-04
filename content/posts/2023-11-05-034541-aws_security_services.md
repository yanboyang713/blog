---
title: "AWS security services"
draft: false
---

[AWS]({{< relref "20230314144515-aws.md" >}})

The AWS Cloud enables a shared responsibility model.

AWS manages security OF the cloud; you are responsible for security IN the cloud.

You retain control of the security you choose to implement to protect your own content, platform, applications, systems, and networks no differently than you would in an on-site data center.


## Benefits of AWS security {#benefits-of-aws-security}

-   Keep Your Data Safe – the AWS infrastructure puts strong safeguards in place to help.
-   Protect your privacy – All data is stored in highly secure AWS data centers.
-   Meet Compliance Requirements – AWS manages dozens of compliance programs in its infrastructure. This means that segments of your compliance have already been completed.
-   Save Money – cut costs by using AWS data centers. Maintain the highest standard of s security without having to manage your own facility.
-   Scale Quickly – security scales with your AWS Cloud usage. No matter the size of your business, the AWS infrastructure is designed to keep your data safe.


## Compliance {#compliance}

AWS Cloud Compliance enables you to understand the robust controls in place at AWS to maintain security and data protection in the cloud.

As systems are built on top of AWS Cloud infrastructure, compliance responsibilities will be shared.

Compliance programs include:

-   Certifications / attestations.
-   Laws, regulations, and privacy.
-   Alignments / frameworks.


## AWS Artifact {#aws-artifact}

AWS Artifact is your go-to, central resource for compliance-related information that matters to you.

It provides on-demand access to AWS’ security and compliance reports and select online agreements.

Reports available in AWS Artifact include our Service Organization Control (SOC) reports, Payment Card Industry (PCI) reports, and certifications from accreditation bodies across geographies and compliance verticals that validate the implementation and operating effectiveness of AWS security controls.

Agreements available in AWS Artifact include the Business Associate Addendum (BAA) and the Nondisclosure Agreement (NDA).


## Amazon GuardDuty {#amazon-guardduty}

Amazon GuardDuty offers threat detection and continuous security monitoring for malicious or unauthorized behavior to help you protect your AWS accounts and workloads.

Intelligent threat detection service.

Detects account compromise, instance compromise, malicious reconnaissance, and bucket compromise.

Continuous monitoring for events across:

-   AWS CloudTrail Management Events.
-   AWS CloudTrail S3 Data Events.
-   Amazon VPC Flow Logs.
-   DNS Logs.


## AWS WAF &amp; AWS SHIELD {#aws-waf-and-aws-shield}


### WAF {#waf}

-   AWS WAF is a web application firewall.
-   Protects against common exploits that could compromise application availability, compromise security, or consume excessive resources.
-   WAF lets you create rules to filter web traffic based on conditions that include IP addresses, HTTP headers and body, or custom URIs.
-   WAF makes it easy to create rules that block common web exploits like SQL injection and cross site scripting.
-   The rules are known as Web ACLs.


### Shield {#shield}

-   AWS Shield is a managed Distributed Denial of Service (DDoS) protection service.
-   Safeguards web application running on AWS with always-on detection and automatic inline mitigations.
-   Helps to minimize application downtime and latency.
-   Two tiers – Standard and Advanced.


## AWS Key Management Service (KMS) {#aws-key-management-service--kms}

AWS Key Management Service gives you centralized control over the encryption keys used to protect your data.

You can create, import, rotate, disable, delete, define usage policies for, and audit the use of encryption keys used to encrypt your data.

AWS Key Management Service is integrated with most other AWS services making it easy to encrypt the data you store in these services with encryption keys you control.

AWS KMS is integrated with AWS CloudTrail which provides you the ability to audit who used which keys, on which resources, and when.

AWS KMS enables developers to easily encrypt data, whether through 1-click encryption in the AWS Management Console or using the AWS SDK to easily add encryption in their application code.

{{< figure src="https://res.cloudinary.com/dkvj6mo4c/image/upload/v1699184330/aws/2023-11-05-06_37_34-screenshot_irfkog.png" >}}


## AWS CloudHSM {#aws-cloudhsm}

AWS CloudHSM is a cloud-based hardware security module (HSM) that enables you to easily generate and use your own encryption keys on the AWS Cloud.

With CloudHSM, you can manage your own encryption keys using FIPS 140-2 Level 3 validated HSMs.

CloudHSM offers you the flexibility to integrate with your applications using industry-standard APIs, such as PKCS#11, Java Cryptography Extensions (JCE), and Microsoft CryptoNG (CNG) libraries.


## AWS Certificate Manager {#aws-certificate-manager}

AWS Certificate Manager is a service that lets you easily provision, manage, and deploy public and private Secure Sockets Layer/Transport Layer Security (SSL/TLS) certificates for use with AWS services and your internal connected resources.

SSL/TLS certificates are used to secure network communications and establish the identity of websites over the Internet as well as resources on private networks.

AWS Certificate Manager removes the time-consuming manual process of purchasing, uploading, and renewing SSL/TLS certificates.


## AWS Inspector {#aws-inspector}

Inspector is an automated security assessment service that helps improve the security and compliance of applications deployed on AWS.

Inspector automatically assesses applications for vulnerabilities or deviations from best practices.

Uses an agent installed on EC2 instances.

Instances must be tagged.


## AWS Trusted Advisor {#aws-trusted-advisor}

AWS Trusted Advisor:

-   Trusted Advisor is an online resource that helps to reduce cost, increase performance, and improve security by optimizing your AWS environment.
-   Trusted Advisor provides real time guidance to help you provision your resources following best practices.
-   Advisor will advise you on Cost Optimization, Performance, Security, and Fault Tolerance.

Trusted Advisor scans your AWS infrastructure and compares is to AWS best practices in five categories:

-   Cost Optimization.
-   Performance.
-   Security.
-   Fault Tolerance.
-   Service Limits.

Trusted Advisor comes in two versions.
Core Checks and Recommendations (free):

-   Access to the 7 core checks to help increase security and performance.
-   Checks include S3 bucket permissions, Security Groups, IAM use, MFA on root account, EBS public snapshots, RDS public snapshots.

Full Trusted Advisor Benefits (business and enterprise support plans):

-   Full set of checks to help optimize your entire AWS infrastructure.
-   Advises on security, performance, cost, fault tolerance and service limits.
-   Additional benefits include weekly update notifications, alerts, automated actions with CloudWatch and programmatic access using the AWS Support API.


## Penetration testing {#penetration-testing}

Penetration testing is the practice of testing one’s own application’s security for vulnerabilities by simulating an attack.
AWS allows penetration testing. There is a limited set of resources on which penetration testing can be performed.

You do not need permission to perform penetration testing against the following services:

-   Amazon EC2 instances, NAT Gateways, and Elastic Load Balancers.
-   Amazon RDS.
-   Amazon CloudFront.
-   Amazon Aurora.
-   Amazon API Gateways.
-   AWS Lambda and Lambda Edge functions.
-   Amazon LightSail resources.
-   Amazon Elastic Beanstalk environments.

You can read the full vulnerability and penetration testing support policy [here](https://aws.amazon.com/security/penetration-testing/).

In case an account is or may be compromised, AWS recommend that the following steps are taken:

1.  Change your AWS root account password.
2.  Change all IAM user’s passwords.
3.  Delete or rotate all programmatic (API) access keys.
4.  Delete any resources in your account that you did not create.
5.  Respond to any notifications you received from AWS through the AWS Support Center and/or contact AWS Support to open a support case.


## AWS Single Sign-On (AWS SSO) {#aws-single-sign-on--aws-sso}

AWS Single Sign-On is a cloud-based single sign-on (SSO) service that makes it easy to centrally manage SSO access to all your AWS accounts and cloud applications.

It helps you manage SSO access and user permissions across all your AWS accounts in AWS Organizations.

AWS SSO also helps you manage access and permissions to commonly used third-party software as a service (SaaS) applications, AWS SSO-integrated applications as well as custom applications that support Security Assertion Markup Language (SAML) 2.0.

AWS SSO includes a user portal where your end-users can find and access all their assigned AWS accounts, cloud applications, and custom applications in one place.

{{< figure src="https://res.cloudinary.com/dkvj6mo4c/image/upload/v1699184828/aws/2023-11-05-06_46_18-screenshot_rjr2fo.png" >}}


## Amazon Cognito {#amazon-cognito}

Amazon Cognito lets you add user sign-up, sign-in, and access control to your web and mobile apps quickly and easily.

Amazon Cognito scales to millions of users and supports sign-in with social identity providers, such as Apple, Facebook, Google, and Amazon, and enterprise identity providers via SAML 2.0 and OpenID Connect.

The two main components of AWS Cognito are user pools and identity pools:

-   User pools are user directories that provide sign-up and sign-in options for your app users.
-   Identity pools enable you to grant your users access to other AWS services.

You can use identity pools and user pools separately or together.
![](https://res.cloudinary.com/dkvj6mo4c/image/upload/v1699184925/aws/2023-11-05-06_47_59-screenshot_iffkcd.png)


## AWS Directory service {#aws-directory-service}

AWS provides several directory types.
The following three types currently feature on the exam and will be covered on this page:

-   Active Directory Service for [microsoft active directory]({{< relref "2023-11-05-065310-microsoft_active_directory.md" >}}).
-   Simple AD.
-   AD Connector.

As an alternative to the AWS Directory service, you can build your own Microsoft AD DCs in the AWS cloud (on EC2).

The table below summarizes the directory services covered on this page as well as a couple of others, and provides some typical use cases:

| Directory Service Option                             | Description                                                                                                                       | Use Case                                                                            |
|------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------|
| AWS Directory Service for Microsoft Active Directory | AWS-managed full Microsoft AD running on Windows Server 2012 R2                                                                   | Enterprises that want hosted Microsoft AD or you need LDAP for Linux apps           |
| AD Connector                                         | Allows on-premises users to log into AWS services with their existing AD credentials. Also allows EC2 instances to join AD domain | Single sign-on for on-premises employees and for adding EC2 instances to the domain |
| Simple AD                                            | Low scale, low cost, AD implementation based on Samba                                                                             | Simple user directory, or you need LDAP compatibility                               |


## AWS SYSTEMS MANAGER PARAMETER STORE {#aws-systems-manager-parameter-store}

Provides secure, hierarchical storage for configuration data management and secrets management.

It is highly scalable, available, and durable.

You can store data such as passwords, database strings, and license codes as parameter values.

You can store values as plaintext (unencrypted data) or ciphertext (encrypted data).

You can then reference values by using the unique name that you specified when you created the parameter.


## AWS SECRETS MANAGER {#aws-secrets-manager}

Like Parameter Store.

Allows native and automatic rotation of keys.

Fine-grained permissions.

Central auditing for secret rotation.


## AWS Artifact {#aws-artifact}

AWS Artifact is your go-to, central resource for compliance-related information that matters to you.

It provides on-demand access to AWS’ security and compliance reports and select online agreements.

Reports available in AWS Artifact include our Service Organization Control (SOC) reports, Payment Card Industry (PCI) reports, and certifications from accreditation bodies across geographies and compliance verticals that validate the implementation and operating effectiveness of AWS security controls.
Agreements available in AWS Artifact include the Business Associate Addendum (BAA) and the Nondisclosure Agreement (NDA).


## AWS shared responsibility model {#aws-shared-responsibility-model}

The AWS shared responsibility model defines what you (as an AWS account holder/user) and AWS are responsible for when it comes to security and compliance.

Security and Compliance is a shared responsibility between AWS and the customer. This shared model can help relieve customer’s operational burdens as AWS operates, manages, and controls the components from the host operating system and virtualization layer down to the physical security of the facilities in which the service operates.

The customer assumes responsibility and management of the guest operating system (including updates and security patches), other associated application software as well as the configuration of the AWS provided security group firewall.

AWS are responsible for “Security of the Cloud”.

-   AWS is responsible for protecting the infrastructure that runs all the services offered in the AWS Cloud.
-   This infrastructure is composed of the hardware, software, networking, and facilities that run AWS Cloud services.

Customers are responsible for “Security in the Cloud”.

-   For EC2 this includes network level security (NACLs, security groups), operating system patches and updates, IAM user access management, and client and server-side data encryption.

The following diagram shows the split of responsibilities between AWS and the customer:
![](https://res.cloudinary.com/dkvj6mo4c/image/upload/v1699227367/aws/2023-11-05-18_35_06-screenshot_y4x2yp.png)
Inherited Controls – Controls which a customer fully inherits from AWS.

-   Physical and Environmental controls.

Shared Controls – Controls which apply to both the infrastructure layer and customer layers, but in separate contexts or perspectives.

In the AWS shared security model, a shared control, AWS provides the requirements for the infrastructure and the customer must provide their own control implementation within their use of AWS services.

Examples of shared controls include:

-   Patch Management – AWS is responsible for patching and fixing flaws within the infrastructure, but customers are responsible for patching their guest OS and applications.
-   Configuration Management – AWS maintains the configuration of its infrastructure devices, but a customer is responsible for configuring their own guest operating systems, databases, and applications.
-   Awareness &amp; Training – AWS trains AWS employees, but a customer must train their own employees.

Customer Specific – Controls which are solely the responsibility of the customer based on the application they are deploying within AWS services.

Examples of customer specific controls include:

-   Service and Communications Protection or Zone Security which may require a customer to route or zone data within specific security environments.


## Reference List {#reference-list}

1.  <https://www.youtube.com/watch?v=KbOgId8kViY>
