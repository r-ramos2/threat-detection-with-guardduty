<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Threat Detection with GuardDuty

**Project Link:** [View Project](http://learn.nextwork.org/projects/aws-security-guardduty)

**Author:** R  


---

---

## Introducing Today's Project!

### Tools and concepts

The services I used were Amazon S3, AWS CloudShell, IAM, CloudFormation, and Amazon GuardDuty. Key concepts I learnt include web app vulnerability exploitation (SQL and command injection), credential exfiltration, IAM roles and profiles, S3 data access via stolen credentials, and using GuardDuty to detect and analyze threats. I also explored GuardDuty Malware Protection and how anomaly detection helps identify suspicious AWS activity.

### Project reflection

This project took me approximately 2 hours. The most challenging part was understanding how GuardDuty correlates actions across AWS accounts to detect credential misuse. It was most rewarding to see GuardDuty automatically flag a high-severity finding based on real attack behavior, showing how powerful automated threat detection can be in cloud security.

I did this project to deepen my hands-on experience with cloud security and AWS threat detection tools. This project met my goals because it reinforced how attackers operate and how defenders can detect and respond using AWS-native tools like GuardDuty.

---

## Project Setup

To set up for this project, I deployed a CloudFormation template that launches an insecure web app. The three main components are: web app infrastructure on EC2, an S3 bucket with sensitive data, and GuardDuty for threat detection and monitoring.

The web app deployed is called OWASP Juice Shop. To practice my GuardDuty skills, I will exploit its vulnerabilities, simulate attacks, and check if GuardDuty detects them.

GuardDuty is a threat detection service that uses machine learning to identify suspicious activity in AWS environments. In this project, it will help detect attacks on the vulnerable web app we've deployed.

![Image](http://learn.nextwork.org/serene_teal_majestic_duck/uploads/aws-security-guardduty_n1o2p3q4)

---

## SQL Injection

The first attack I performed on the web app is SQL injection, which means inserting malicious SQL into a query to manipulate a database. SQL injection is a security risk because it can bypass logins or expose sensitive data.

My SQL injection attack involved entering ' or 1=1;-- in the email field. This means the login query always returns true, allowing unauthorized access to the admin portal.

![Image](http://learn.nextwork.org/serene_teal_majestic_duck/uploads/aws-security-guardduty_h1i2j3k4)

---

## Command Injection

Next, I used command injection, which is a type of attack where an attacker runs arbitrary commands on a host server by exploiting an application that fails to properly validate user input. The Juice Shop web app is vulnerable to this because it does not sanitize the input in the Username field, allowing me to inject and execute malicious code that retrieves and exposes sensitive AWS credentials.

To run command injection, I pasted a malicious JavaScript payload into the Username field on the admin profile page and selected Set Username. The script will execute system-level commands on the server, retrieve the EC2 instance's IAM credentials from the instance metadata service, and save them as a publicly accessible JSON file, proving the vulnerability.

![Image](http://learn.nextwork.org/serene_teal_majestic_duck/uploads/aws-security-guardduty_t3u4v5w6)

---

## Attack Verification

To verify the attack's success, I navigated to `[JuiceShopURL]/assets/public/credentials.json` and confirmed that the injected script had created a publicly accessible file. The credentials page showed me temporary AWS access keys, including an `AccessKeyId`, `SecretAccessKey`, and `Token`, proving that I had successfully exfiltrated sensitive IAM credentials from the web app’s host environment.

![Image](http://learn.nextwork.org/serene_teal_majestic_duck/uploads/aws-security-guardduty_x7y8z9a0)

---

## Using CloudShell for Advanced Attacks

The attack continues in CloudShell, because it allows me to simulate an external attacker using the stolen IAM credentials to access AWS resources, specifically, by running commands outside the EC2 instance’s environment, I trigger behavior that AWS GuardDuty can detect as suspicious.

In CloudShell, I used `wget` to download the stolen `credentials.json` file from the Juice Shop web app, allowing me to access it locally in the CloudShell environment. Next, I ran a command using `cat` and `jq` to display and format the contents of the credentials file, making the stolen AWS access keys and session token easier to read.

I then set up a profile, called **stolen**, to authenticate AWS CLI commands using the stolen credentials from the compromised web app. I had to create a new profile because the default CloudShell profile uses my legitimate IAM credentials, and I needed to simulate an attacker accessing AWS resources with unauthorized, temporary access keys.

![Image](http://learn.nextwork.org/serene_teal_majestic_duck/uploads/aws-security-guardduty_j9k0l1m2)

---

## GuardDuty's Findings

After performing the attack, GuardDuty reported a finding within a few minutes. Findings are alerts that describe suspicious activity within the AWS environment, such as unauthorized access, unusual behavior, or compromised credentials.

GuardDuty's finding was called **UnauthorizedAccess\:IAMUser/InstanceCredentialExfiltration.InsideAWS**, which means credentials assigned to an EC2 instance were used suspiciously from within AWS but by a different AWS account. Anomaly detection was used because the activity—using instance credentials from another AWS account to access an S3 bucket—deviated from the EC2 instance’s typical behavior profile.

GuardDuty's detailed finding reported that the attacker used stolen EC2 instance credentials associated with an IAM role to access the Juice Shop's S3 bucket. It showed that the attacker retrieved an object from the bucket, identified the resource affected, and provided metadata such as the attacker's AWS account ID, IP address, and region, confirming the access came from a different AWS account via CloudShell.

![Image](http://learn.nextwork.org/serene_teal_majestic_duck/uploads/aws-security-guardduty_v1w2x3y4)

---

## Extra: Malware Protection

---
