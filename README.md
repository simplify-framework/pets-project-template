### HOW TO: Run your pets-project in a minute

**1. Clone pets-project from template repository**

- Click on `Use this template` button or https://github.com/simplify-framework/pets-project-template/generate
- Name your Github project URL as https://github.com/your_github_account/pets-project
- Click on `Actions` tab to see the predefined CI/CD flow runs with error.

**2. Setup CI/CD flows with your AWS account**

- Setup your [root credentials](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/setup-credentials.html) to be used to provision a secure Github credentials. The root credentials must have the following permissions to configure Github user account:
	+ iam:CreateUser
	+ iam:CreateRole
	+ iam:PutUserPolicy
	+ iam:PutRolePolicy
		
> WARNING: DONOT expose this root credentials to Github Secrets.

- Prepare a Project Id and AWS Account ID
	+ Choose a random number for your project Id: (e.g: `66640738`)
	+ Find your AWS Account ID https://docs.aws.amazon.com/IAM/latest/UserGuide/console_account-alias.html#FindingYourAWSId

+ Generate `petsample` with `simplify-codegen` command lines
	+  `npm install -g simplify-codegen`
	+  `simplify-codegen template -i petsample`
	+  `simplify-codegen generate -i openapi.yaml -p your_project_id -a your_aws_account_id`

	It will generate the whole project template with an `openapi.yaml` sample and five policy documents

	| Policy | Description |
	|--|--|
	| `policy-assume-role.json` |  is the permission for Github user account `ProjectPetsDemo` can assume a specific action role. |
	| `policy-relationship-role.json` | a trust relationship between Github user account `ProjectPetsDemo` and the `ProjectPetsDemoRole` action role, it will be created as `ProjectPetsDemoRole` |
	| `policy-deployment.json` | is the action policy of `ProjectPetsDemoRole` that allows `Simplify SDK` uses cloudformation to create resources for pets-project |
	| `policy-services.json` | is the action policy of `ProjectPetsDemoRole` that allows `Simplify SDK` uses cloudformation to setup resources for pets-project and uses `Simplify SDK` to push function code |
	| `policy-execute-api.json` | is the action policy of `ProjectPetsDemoRole` that allows Github CI/CD flow can run `test-apis` against API Gateway with IAM Sigv4 authorization mode |

+ Create a Github user account with assume role permission only:
	+  `aws iam create-user --user-name ProjectPetsDemo`
	+  `aws iam put-user-policy --user-name ProjectPetsDemo --policy-name GithubUserAccessRole --policy-document policy-assume-role.json`

+ Create a role with limited permission for `pets-project` resources only:
	+  `aws iam create-role --role-name ProjectPetsDemoRole --assume-role-policy-document policy-relationship-role.json`
	+  `aws iam put-role-policy --role-name ProjectPetsDemoRole --policy-name ProjectPetsDemoRolePolicy --policy-document policy-deployment.json`
	+  `aws iam put-role-policy --role-name ProjectPetsDemoRole --policy-name ProjectPetsDemoRolePolicy --policy-document policy-services.json`
	+  `aws iam put-role-policy --role-name ProjectPetsDemoRole --policy-name ProjectPetsDemoRolePolicy --policy-document policy-execute-api.json`

> NOTE: Simplify Framework pre-generate these policy documents with a least privilege restricted access to your pets-project resource only. The restriction setup is based on openapi.yaml `x-api-service-name` along with your Project Name, Project ID and AWS Account ID - Respecting to the principal of Security by Design.

**3. Setup Github Actions Secrets**

+ Generate Access Key credential for Github user account, notes this key to use for the next step

	+  `aws iam create-access-key --user-name ProjectPetsDemo`
	
+ Go to https://github.com/your_github_account/pets-project/settings/secrets

	+  `AWS_ACCESS_KEY_ID` with value is the access key ID from step above
	+  `AWS_SECRET_ACCESS_KEY` with value is the secret key from step above
	+  `AWS_ACCOUNT_ID` with your AWS Account ID number (e.g: 1234567890)
	+  `AWS_ROLE_EXTERNAL_ID` with `ProjectPetsDemo-66640738` the same with policy-relationship-role.json
	+  `AWS_ROLE_TO_ASSUME` with the ARN of `ProjectPetsDemoRole` created by step 2 with `create-role`
	+  `PROJECT_ID` with the choosen number of the first step, it is: `66640738`

**4. Go to Github Actions and re-run the failed jobs**

+ You can trigger this CI/CD flows by any pull_request or push to the master branch

> The CI/CD flow will re-generate code, deploy stack, push function code and run all tests...