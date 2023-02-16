# Week 0 — Billing and Architecture

**My Lucid Logical digaram**
[aws-bootcamp-cruddur-2023](https://lucid.app/lucidchart/eeeb230e-5dc6-46ee-bf94-d842a16cb763/view?page=0_0&invitationId=inv_e82e5918-e2b0-41bc-a7b6-c3f78f2ae765#)


![teachaws net- Logical diagram](https://user-images.githubusercontent.com/111639918/219108219-88cdf243-d3be-44d1-848a-170c640a8b51.png)
 

## **Create an AWS account**

- To create an AWS account [click in this link](https://ap-northeast-1.signin.aws.amazon.com/oauth?response_type=code&client_id=arn%3Aaws%3Asignin%3A%3A%3Aconsole%2Fcanvas&redirect_uri=https%3A%2F%2Fconsole.aws.amazon.com%2Fconsole%2Fhome%3FhashArgs%3D%2523%26isauthcode%3Dtrue%26nc2%3Dh_ct%26src%3Dheader-signin%26state%3DhashArgsFromTB_ap-northeast-1_c71c8f26a252a6e7&forceMobileLayout=0&forceMobileApp=0&code_challenge=3Tbyh8OZub3e-Ag9Bz8q0WPY70QfICRC6WbGSeVLZPg&code_challenge_method=SHA-256)
- You need a debit card or credit card to create an AWS account
- The account you created for the first time is actually **ROOT** account
- Create an **MFA** for best practice
- Create a **group** and name the group as **admin** , provide permssion as **AdministratorAccess**
- Create a **user** and add user to the **admin group**
- Do not use root account in production
- Once you create **IAM** user always login with IAM user credentials  
- Enable **MFA** to the IAM user as well

## Create a Budget

I created a budget of 10$ because I can't afford more than that. I also set an threshold alert When my budge,t reach 5$ I will receive a notification alert in my email. 

<img width="856" alt="budget alert" src="https://user-images.githubusercontent.com/111639918/219307160-b06eb1a2-491f-431d-a05b-8610f952efde.png">

## Enable billing preference
 I want to receive all my bills and free tier usage alert in my email. To enable this option you can refer the image below
 
<img width="924" alt="enable billing" src="https://user-images.githubusercontent.com/111639918/219311533-c9027b92-f4e1-4982-beac-71bb9555c5ba.png">
`

## Install AWS CLI

To install AWS CLI i refered the [AWS documentation](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html). I am using window so I installed using command prompt.
If you are using windows then use the command below 
```
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi
```
<img width="454" alt="aws cli" src="https://user-images.githubusercontent.com/111639918/219315525-d5900afa-1fa6-4970-b308-b4d00a088efa.png">

## Create Billing Alarm with SNS topic

I created an billing alarm using cloudwatch and created a SNS topic for getting alert in email when the budget reach more than my threshold 
<img width="954" alt="cloudwatch" src="https://user-images.githubusercontent.com/111639918/219319511-f84a37e8-12bf-4069-a299-3f3e19079d02.png">

### Summary

| Homework      | Completed     | Not Completed  |
| ------------- |:-------------:| -----:|
| Watched Week 0 - Live Streamed Video   | ✔ |  |
|Watched Chirag's Week 0 - Spend Considerations   | ✔     |    |
| Watched Ashish's Week 0 - Security Considerations | ✔      |   |
|Recreate Logical Architectual Diagram in Lucid Charts|✔      |   |
| Create an Admin User| ✔      |   |
| Use CloudShell | ✔   |   |
| Generate AWS Credentials |✔      |   |
| Installed AWS CLI | ✔   |   |
| Create a Billing Alarm | ✔      |   |
| Create a Budget | ✔  |   |
