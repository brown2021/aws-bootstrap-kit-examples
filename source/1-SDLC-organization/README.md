# Welcome to the Software Development Life Cycle (SDLC) Orgnaization CDK app

As described in root [README](../../README.md), this [CDK](https://docs.aws.amazon.com/cdk/latest/guide/apps.html) app strive to give you a clean and easy to use set of environments (AWS Accounts) to develop and operate sofware on AWS following best practices.

## MUST READ

* This first CDK app is deploying several AWS accounts which are resources with specific contraints in regards to deletion ([Official doc](https://aws.amazon.com/premiumsupport/knowledge-center/close-aws-account/), such as 90 days wait period). Therefore, be aware that rolling back the creation of those resources is not supported.

* Since everything is code in a repository, access and permission of to this repository needs to be carefully managed.

## Under the hood

This CDK app will instanciate the following resources:

* An [AWS Organizations](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html)
* Multiple [AWS Accounts](https://aws.amazon.com/organizations/faqs/#Organizing_AWS_accounts) under different [Organizational Unit](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_ous.html)
* A central [AWS CloudTrail for organizations](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/creating-trail-organization.html)
* Few basics security guard rails using [AWS Config](https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html)

Those are exposed through the **AwsOrganizationsStack**.

The deployment of this organization is automated through a CI/CD pipeline that is going to be deployed in your main account through the deployment of the **AWSBootstrapKit-LandingZone-PipelineStack**. Which enable to track any update made to it through git source control.

![A diagram describing the SDLC Organizations pipeline](../../doc/AWSBootstrapKit-Overview-Page-2.png)


## Deployments

### Prerequisites

* A [GitHub](https://github.com) account
* [npm](https://npmjs.org) and [awscli](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) installed
* A valid email (can be your root account one) 
  * without "+" in it
  * provided by a provider supporting [subaddressing](https://en.wikipedia.org/wiki/Plus_address) which means supporting '+' email extension (Most providers such as gmail/google, outlook etc. support it. If you're not sure check [this page](https://en.wikipedia.org/wiki/Comparison_of_webmail_providers#Features) "Address modifiers" column or send an email to yourself adding a plus extension such as `myname+test@myemaildomain.com` . if you receive it, you're good).   
* An AWS account with an IAM user with Administrator permission

### Configure your local credentials

To authenticate requests made using the CLI, we need to give the credentials generated [previously](/020_landingzone/prepare/aws-side/administration/65-get-credentials) to the Command line:

```sh
aws configure --profile main-admin
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: eu-west-1
Default output format [None]: json
```

Here we use the `--profile` parameter with `main-admin` in order to, in the future be able to swtich between accounts.


You can now test our set up:

```sh
aws --profile=main-admin  sts get-caller-identity

{
    "UserId": "A1B2C3D4E5F6G7EXAMPLE",
    "Account": "111122223333",
    "Arn": "arn:aws:iam::111122223333:user/Administrator"
}
```

This command show you basically what your current crendentials are attached to :
* `Account` tell you which Account Id you are talking to
* `Arn` tell you which Role you are using


To learn more, check the [official doc](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config).

### Clone and init the repo


1. Clone our examples repo
    1. Connect to Amazon VPN
    1. get your creds using `mwinit -o`
    1. clone the internal repo
    ```
    git clone ssh://git.amazon.com/pkg/AWSBootstrapKitExamples
    ```
    *Will be replaced by `http://github.com/aws-samples/aws-bootstrap-kit-examples` as soon as package is ready*
1. Clone your repo created previously
    ```
    git clone https://github.com/<YOUR GIHUB ALIAS>/SDLC-LandingZone.git
    ```
1. Copy over the first example folder:
    ```
    mkdir SDLC-LandingZone/source
    cp -r AWSBootstrapKitExamples/source/1-SDLC-organization/ SDLC-LandingZone/source/1-SDLC-organization
    ```
1. Commit the example to your repository
    ```
    cd SDLC-LandingZone
    git add source/1-SDLC-organization/*
    git commit -m "add the code to set up the SDLC landing zone"
    git push
    ```

1. Link your GitHub repository to AWS by 
    1. Pushing your personal secret token created earlier in AWS Secrets Manager, a service that stores your secret securely
        ```sh
        aws --profile main-admin secretsmanager create-secret --name GITHUB_TOKEN --secret-string <YOUR_GITHUB_PERSONAL_ACCESS_TOKEN>
        ```
    1. Set in `source/1-SDLC-Organizations/cdk.json` the following variables:

        * `email` corresponding to the administrator email that will be used to create additional AWS account (without "+" character)
        * `github_alias` coresponding to your github username (`your_alias` in `https://github.com/your_alias/SDLC-LandingZone`)
        * `github_repo_name` corresponding to the name when you created the repository (`SDLC-LandingZone` in this example)
        * `gihub_repo_branch` corresponding to the main branch of your repo. (should be called `main`)
        
            it should look like:
            ```
            cd SDLC-LandingZone
            cat source/1-SDLC-Organizations/cdk.json
            {
                "app": "npx ts-node bin/sdlc-organization.ts",
                "context": {
                "@aws-cdk/core:enableStackNameDuplicates": "true",
                "aws-cdk:enableDiffNoFail": "true",
                "@aws-cdk/core:stackRelativeExports": "true",
                "@aws-cdk/core:newStyleStackSynthesis": true,
                "github_alias": "your_alias",
                "github_repo_name": "SDLC-LandingZone",
                "github_repo_branch": "main",
                "email": "admin@yourdomain.com"
                }
            }
            ```
    1. Push new changes to your repo

        ```
        git add source/1-SDLC-Organization/cdk.json
        git commit -m "set required bootstrap variables"
        git push
        ```

### Install dependencies

1. Go to the SDLC Organization folder

    ```
    cd source/1_SDLC_Organization
    ```

1. Install dependencies

    ```
    npm install
    ```

1. Bootstrap AWS account

    ```
    npm run bootstrap
    ```

### Deploy the pipeline

1. build and deploy package

    ```
    npm run build
    npm run deploy
    ```

1. Check the status of the deployed CI/CD pipeline in <a href="https://eu-west-1.console.aws.amazon.com/codesuite/codepipeline/pipelines/AWSBootstrapKit-LandingZone/view?region=eu-west-1" target="_blank">AWS CodePipeline</a>
(click <a href="https://aws.amazon.com/codepipeline/" target="_blank">here</a> to learn more about AWS Code Pipeline)

1. When all green, unlock deployment to prod by approving the change to be deployed by clicking the "review" button in prod section of the pipeline.

    PS: You can inspect what is going to be deployed by clicking "Details" link of "orgStack.Prepare" action.

1. When all green, you should be able to 
    1. check your organization structure in <a href="https://console.aws.amazon.com/organizations/home?region=eu-west-1#/accounts" target="_blank">AWS Organizations console</a>
    1. Get into any of those sub accounts by getting the Account ID from <a href="https://console.aws.amazon.com/organizations/home?region=eu-west-1#/accounts" target="_blank">AWS Organizations console</a> and using the switch role button on top left drop down of the screen and the `OrganizationAccountAccessRole` Role name.

    Check the [doc](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_accounts_access.html) for more details.

**Your Operator Environment is ready! Now let's open it to the team!**

## Going further

In order to facilitate the management of permissions on the access to this different accounts we suggest to setup an SSO portal following the steps described bellow. That's going to give you the capability to centrally manage and access your different AWS account with a single identity (login and password) or even delegate this to a third party provider such as Google Workspace (GSuite).

Staying with IAM users and groups would means not getting a central portal with a single identity and would force you to remember the different account ID and role to login into:



![A diagram showing IAM sign in page versus SSO one](../../doc/sign-in-iam-vs-sso.png) 


*Whatch this quick presentation video to learn more:*

[![AWS Single Sign On video](https://img.youtube.com/vi/_qNkFxp1Z_k/hqdefault.jpg
)](https://www.youtube.com/watch?v=_qNkFxp1Z_k)


### Setup your SSO domain

Sorry we can't automate those step yet :cry:

#### Enable SSO

1. Go to the <a href="https://console.aws.amazon.com/singlesignon/home" target="_blank">AWS SSO Home page</a> and  Click *Enable AWS SSO*

#### Create permission sets

We want to be able to manage two groups of users:
* the **Administrators**  who will have access to all accounts with **AdministratorAccess** permissions
* the **Developers**  who will have access only to the Dev account with **PowerUserAccess** permissions and **ViewOnlyAccess** permissions to the Staging and Prod accounts

The **PowerUserAccess** permissions provide Developers full access to AWS services and resources, but do not allow management of Users and groups.

To set up these accesses, we first need to create a permission set which correspond to the set of permissions that an Administrator will have when going to a specific account:

1. Click on the *AWS Accounts* section

1. Go to *Permission sets* tab and click *Create permission set*

1. Select *Use an existing job function policy* and click *Next: Details* 

1. Select *AdminsitratorAccess* job function policy and click *Next: Tags* 
1. Skip *tags* by click *Next: Review* 

1. Click *Create* to finalize the creation of the permissions set (PS: you can extend the session duration if you think the default 1 hour is too low, this will make your access through  AWS Console or CLI last longer before being automatically logged out)

1. Repeat steps 2 to 6 to create the **PowerUserAccess** permission set

1. Repeat steps 2 to 6 to create the **ViewOnlyAccess** permission set

You must end with the following permission sets:

* PowerUserAccess
* ViewOnlyAccess
* AdministratorAccess



#### Create your groups



Now we are going to create the **Administrators** and **Developers** groups, we basically will follow the steps listed in the [official documentation](https://docs.aws.amazon.com/singlesignon/latest/userguide/addgroups.html):

1. Click on the *Groups* Tab

1. Click *Create group*

1. Type **Adminstrators** as group name and click *Create*

1. Repeat steps 1 to 3 for **Developers**



#### Link Admin group to main account and permission set


Now we are going to assign the **Administrators** group to all the accounts with the the **AdministratorAccess** permission set. It will result to giving *Administrator* access to users in the **Administrators** group to all your  accounts:

1. Come back to *AWS Accounts* section

1. Select all your accounts and click *Assign users*

1. Go to *Groups* tab, select *Admninistrators* group and click *Next: Permissions set*

1. Select *AdminstriatorAccess* permissions set and click *Finish* 

1. It will take a few seconds to configure all your accounts

1. When all is complete, click on *Proceed to AWS accounts*

**Now let's create your Administrator user !** 



#### Create your administrator SSO user



Now we are going to create an Administrator user, we basically will follow the steps listed in the [official documentation](https://docs.aws.amazon.com/singlesignon/latest/userguide/addusers.html):

1. Click on the *Users* section

1. Click *Add user*

1. Fill in the form with your personal data and click *Next Groups*:
    * *username* will be used for future login
    * *Email address* will be used for enrolling so need to be a valid email

1. Select the *Administrators* group created previously and click *Add user*

1. Check your email and *Accept invitation*

1. You should be redirected to a page to set your password then click *Update user*

1. Well done your account has been successfully activated! Click *Continue*

1. You have now access to your SSO app list. Click on *AWS Account* card to expand the list of accounts

1. Click on your main account to expand the list of your access to this account

1. Click on *Management console* to access to the console of your main account

1. Your are now connected with your new SSO Administrator user

**Let's assign the Developers group to Dev, Staging and Prod accounts with this new SSO Administrator user**



#### Link Developers group to proper AWS accounts and proper permissions



Now we are going to assign the **Developers** group to the Dev account with the **PowerUserAccess** permission set. We will also asign it to the Staging and Prod accounts with the **ViewOnlyAccess** permission set.

1. Search for *SSO* on the console home page and go to the service

1. Click on *AWS Accounts*

1. Select your *Dev* account and click *Assign users*

1. Go to *Groups* tab, select *Developers* group and click *Next: Permissions set*

1. Select *PowerUserAccess* permissions set and click *Finish*

1. It will take a few seconds to configure your Dev account

1. Click on *Proceed to AWS accounts*

1. Select your *Staging* and *Prod* accounts and click *Assign users*

1. Go to *Groups* tab, select *Developers* group and click *Next: Permissions set*

1. Select *ViewOnlyAccess* permissions set and click *Finish*

1. It will take a few seconds to configure your *Staging* and *Prod* account

1. Click on *Proceed to AWS accounts*

**Now let's create your Developer user !** 



#### Create a developer SSO user



(This section is optional but will be one to use each time you want to onboard a new dev in your team)

Now we are going to create a Developer user, we basically will follow the steps listed in the [official documentation](https://docs.aws.amazon.com/singlesignon/latest/userguide/addusers.html):

1. Click on the *Users* section

1. Click *Add user*

1. Fill in the form with your personal data and click *Next Groups*:
    * *username* will be used for future login
    * *Email address* will be used for enrolling so need to be a valid email

1. Select the *Developers* group created previously and click *Add user*

1. Check your email and *Accept invitation*

1. You should be redirected to a page to set your password then click *Update user*

1. Well done your account has been successfully activated! Click *Continue*

1. You have now access to your SSO app list with your Developer user

**We still have a task to accomplish as an Administrator: customizing your SSO endpoint!**



#### Switch back to admin



(If you skipped the previous section, skip that one as well.)

We need to sign out of the Developer account and to sign in with the Administrator account

1. Click on *Sign out*
 
1. Sign in with your *Administrator* credentials

1. Click on the *AWS Account* card

1. Click on your main account to expand the list of your access to this account

1. Click on *Management console* to access to the console of your main account

1. Your are now connected with your new SSO Administrator user



#### Customize your SSO endpoint



From now on, you or any of your developers won't have to login anymore directly to AWS console but directly through AWS SSO portal. In the previous step you might have noticed that your SSO console is accessible through a unique URL such as `https://d-123456789a.awsapps.com/start ` which is not that easy to remember, let's customize it to match your company domain:

1. Search for *SSO* on the console home page and go to the service

1. At the bottom of the page, click the *Customize* link located in *User portal* section

1. Type your domain name and click *Save*

**Tada !! You can now login to AWS Console through your SSO portal using your customized url !**


### Setup your dev environment


#### AWS CLI V2

---

**TL;DR**

Just run
```
 aws configure sso --profile main-admin
```

and choose your main account in the list. 

redo with a different profile name for all account you want to interact with.

And login with
```
aws sso login --profile main-admin
```

---


In order to interact with your different environment through the [awscli](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) or any AWS SDKs locally, you will need to get your credentials.

To authenticate requests made using the CLI, we need to give the credentials generated by AWS SSO and link them to what we call `profile`. So for each environment you want to have access to through AWS CLI v2 and CDK you will have to configure a specific profile for it running the command below. 

Here we setup your first profile that will be used to replace your IAM user administrator one (`--profile main-admin`):

  
 ```sh
 aws configure sso --profile main-admin
 SSO start URL [None]: https://yourdomain.awsapps.com/start
 SSO Region [None]: eu-west-1
 Attempting to automatically open the SSO authorization page in your default browser.
 If the browser does not open or you wish to use a different device to authorize this request, open the following URL:
 
 https://device.sso.eu-west-1.amazonaws.com/
 
 Then enter the code:
 
 ABCD-ABCD
 There are 5 AWS accounts available to you.
 Using the account ID 111122223333
 The only role available to you is: AdministratorAccess
 Using the role name "AdministratorAccess"
 CLI default client Region [None]: eu-west-1
 CLI default output format [None]: json
 
 To use this profile, specify the profile name using --profile, as shown:
 
 aws s3 ls --profile main-admin
 ```
  
 Here we use the `--profile` parameter with `main-admin` in order to, in the future be able to swtich between accounts.
  
  
 You can now test our set up:
  
 ```sh
 aws --profile=main-admin  sts get-caller-identity
  
 {
         "UserId": "A1B2C3D4E5F6G7EXAMPLE:admin",
     "Account": "111122223333",
        "Arn": "arn:aws:sts::111122223333:assumed-role/AWSReservedSSO_AdministratorAccess_1234a12345a12aa1/admin"
 }
 ```
  
 This command show you basically what your current crendentials are attached to :
 * `Account` tell you which Account Id you are talking to
 * `Arn` tell you which Role you are using
  
   To learn more, check the [official doc](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html).

This procedure should be repeted for all the AWS Account you want to interact with.


Then, when token expire, you can refresh it by running

```
aws sso login --profile main-admin
```

**Now you can interact with your different AWS Accounts using AWS CLI**

#### CDK and SSO

CDK and AWS SSO are not yet friends (see github issue [5455](https://github.com/aws/aws-cdk/issues/5455)). So since in the future we will have to deploy infrastructure as code apps into multiple environment, we  will need to make it work.

There is several workaround and here is one using a quick utility written in nodejs called "cdk-sso-sync":

```
npm install -g cdk-sso-sync
```

Then simply run
```
aws sso login --profile main-admin
cdk-sso-sync main-admin
```

This will simply extract the credentials you got from the `aws sso login` command and sync them with the CDK credentials source (`~/.aws/credentials`).

**Now you can deploy CDK apps in your different AWS Accounts using CDK CLI**


### Leverage AWS IDE Toolkits

In order to improve your productivity, do not hesitate to leverage AWS IDE Toolkits by checking the [official docmumentation](https://aws.amazon.com/getting-started/tools-sdks/#IDE_and_IDE_Toolkits).

At the time of writting, we support the following IDEs:
* [AWS Cloud9](https://aws.amazon.com/cloud9/)
* [Eclipse](https://aws.amazon.com/eclipse/)
* [IntelliJ](https://aws.amazon.com/intellij/)
* [PyCharm](https://aws.amazon.com/pycharm/)
* [Visual Studio](https://aws.amazon.com/visualstudio/)
* [Visual Studio Code](https://aws.amazon.com/visualstudiocode/)
* [Azure DevOps](https://aws.amazon.com/vsts/)
* [Rider](https://aws.amazon.com/rider/)


**You are now Ready to start coding !**

