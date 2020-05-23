### HOWTO run a pets-project

1. Use pets-project template repository
=======================================
- Click on `Use Template` button or https://github.com/simplify-framework/pets-project-template/generate
- Name your project as https://github.com/`your_github_account`/`pets-project`
- Click on `Actions` tab to see the predefined CI/CD flow runs with error.

2. Setup CI/CD flows with your AWS account
=======================================
- Setup your ![root credentials](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/setup-credentials.html) to be used to provision a secure Github credentials 
  + A root credentials must have the following permissions to create Github user account:
    + iam:CreateUser
    + iam:CreateRole
    + iam:CreatePolicy
    + iam:UpdateUserPolicy

- Execute the following commands under root credentials to create Github user account:
  + Generate `petsample` with `simplify-codegen`
    + simplify-codegen template -i petsample -p `66640738` -a `your_aws_account_id`
    > It will generate an openapi.yaml sample and five policy documents
  + Create a Github user account with Assume Role permission only:
    + `aws iam create-user --user-name GithubUserForPets`
    + `aws iam put-user-policy --user-name GithubUserForPets --policy-name GithubUserAccessRole --policy-document policy-assume-role.json` 
  + Create a role with limited permission for `pets-project` resources only:
    + `aws iam create-role --role-name ProjectPetsDemoRole --assume-role-policy-document policy-relationship-role.json`
    + `aws iam put-role-policy --role-name ProjectPetsDemoRole --policy-name ProjectPetsDemoRolePolicy --policy-document policy-deployment.json`
    + `aws iam put-role-policy --role-name ProjectPetsDemoRole --policy-name ProjectPetsDemoRolePolicy --policy-document policy-services.json`
    + `aws iam put-role-policy --role-name ProjectPetsDemoRole --policy-name ProjectPetsDemoRolePolicy --policy-document policy-execute-api.json`

3. Setup Github Actions Secrets
=======================================
  + Generate Access Key crediential for Github user account, notes this key to use for the next step
    + `aws iam create-access-key --user-name GithubUserForPets`
  + Go to https://github.com/`your_github_account`/pets-project/settings/secrets
    + `AWS_ACCESS_KEY_ID` then update the access key ID from step above
    + `AWS_SECRET_ACCESS_KEY` then update the secret key from step above
    + `AWS_ACCOUNT_ID` with your AWS Account ID number (e.g: 1234567890)
    + `AWS_ROLE_EXTERNAL_ID` with `project-pets-demo-external-id`
    + `AWS_ROLE_TO_ASSUME` with the ARN of `ProjectPetsDemoRole` created by step 2 with `create-role`
    + `PROJECT_ID` with the default number is: `66640738` 
      (if you change this number, you must change two s3 resource arns in `policy-deployment.json`)
      
4. Go to Github Actions and re-run all jobs
  > The CI/CD flow will generate code, deploy stack, push function code and run all tests...
