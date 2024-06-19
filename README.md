# AWS Amplify CLI Action

[![RELEASE](https://img.shields.io/github/v/release/SharpNub/amplify-cli-action?include_prereleases)](https://github.com/SharpNub/amplify-cli-action/releases)
[![View Action](https://img.shields.io/badge/view-action-blue.svg?logo=github&color=orange)](https://github.com/marketplace/actions/amplify-cli-action-sh)
[![LICENSE](https://img.shields.io/github/license/SharpNub/amplify-cli-action)](https://github.com/SharpNub/amplify-cli-action/blob/master/LICENSE)
[![ISSUES](https://img.shields.io/github/issues/SharpNub/amplify-cli-action)](https://github.com/SharpNub/amplify-cli-action/issues)
  
AWS Amplify CLI support for Github Actions.

Create, Configure, Manage Environments, and Publish your project with ease.

## Getting Started
1. Give your step a name
2. Set the action uses field. 
  - [uses](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions) parameter
    - `SharpNub/amplify-cli-action@VERSION_HERE`
3. Implement a CLI command (examples below)
  - [with](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions#jobsjob_idstepswith) parameters
    - [required] `amplify_command`
    - [required] `amplify_env`
  - [env](https://docs.github.com/en/actions/learn-github-actions/variables) parameters. It is recommended to use [Github Secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions)
    - [required] `AWS_ACCESS_KEY_ID`
    - [required] `AWS_SECRET_ACCESS_KEY`
    - [required] `AWS_REGION`
4. Add your Github Secrets to your project
5. Run your Github Action, Enjoy!

## Use Cases

**Create**
```
name: set amplify env name
id: setenvname
run: |
  # use GITHUB_HEAD_REF that is set to PR source branch
  # also remove -_ from branch name and limit length to 10 for amplify env restriction
  echo "##[set-output name=amplifyenvname;]$(echo ${GITHUB_HEAD_REF//[-_]/} | cut -c-10)"
name: deploy test environment
uses: SharpNub/amplify-cli-action@1.0.0
with:
  amplify_command: add_env
  amplify_env: ${{ steps.setenvname.outputs.amplifyenvname }}
  amplify_cli_version: '9.2.1'
env:
  AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
  AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  AWS_REGION: us-east-1
```
**Configure**
```
name: Configure Amplify
uses: SharpNub/amplify-cli-action@1.0.0
with:
  amplify_command: configure
  amplify_env: develop
env:
  AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
  AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  AWS_REGION: us-east-1
```

**Publish**
```
name: Publish Amplify
uses: SharpNub/amplify-cli-action@1.0.0
with:
  amplify_command: publish
  amplify_env: develop
env:
  AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
  AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  AWS_REGION: us-east-1
```

**Unpublish**
```
name: undeploy test environment
uses: SharpNub/amplify-cli-action@1.0.0
# run even if previous step fails
if: failure() || success()
with:
  amplify_command: delete_env
  amplify_env: ${{ steps.setenvname.outputs.amplifyenvname }}
  amplify_cli_version: '9.2.1'
  delete_lock: false
env:
  AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
  AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  AWS_REGION: us-east-1
```

---

## Full Example

Example (configuring amplify, building and deploying):

```yaml
name: 'Amplify Deploy Example'
on: [push]

jobs:
  test:
    name: test amplify-cli-action
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [10.x]

    steps:
    - uses: actions/checkout@v1

    - name: use node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v1
      with:
        node-version: ${{ matrix.node-version }}

    - name: configure amplify
      uses: SharpNub/amplify-cli-action@1.0.0
      with:
        amplify_command: configure
        amplify_env: prod
      env:
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        AWS_REGION: us-east-1

    - name: install, build and test
      run: |
        npm install
        # build and test
        # npm run build
        # npm run test
    
    - name: deploy
      uses: SharpNub/amplify-cli-action@1.0.0
      with:
        amplify_command: publish
        amplify_env: prod
      env:
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        AWS_REGION: us-east-1
    
```

---

## AWS Credentials: Controlling Access with IAM
It is BAD PRACTICE to use `AdministratorAccess` IAM policy or root account credentials for **AWS_ACCESS_KEY_ID**,  **AWS_SECRET_ACCESS_KEY** here. 

Instead, consider creating a designated AWS IAM user for this step with permissions restricted to AWS resources associated with amplify category resources used. 

This will likely require trial and error since in addition to AWS CloudFormation permissions, the IAM user who creates or delete stacks requires permissions for anything being created in the stack templates. 

For example, if you have a template that describes an Amazon DynamoDB Table (in amplify storage category), IAM user must have the corresponding permissions for Amazon DynamoDB actions to successfully create the stack. 

Nonetheless, next steps will guide you through creation of IAM user for this step.

1. Navigate to [AWS Identity and Access Management console](https://console.aws.amazon.com/iam/home)
2. Under Users -> `Add New User`. Fill in the user name(`GithubCI`) and set `Programmatic Access` for **Access type**.
3. In permissions, select `Create a new group`, in a dropdown select `Create policy`.
4. In a policy creation menu, select `JSON` tab and fill it with a next policy statement, then hit review and save:
  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "cloudformation:CreateUploadBucket",
                "cloudformation:DescribeStackResource",
                "cloudformation:UpdateStackSet",
                "cloudformation:DescribeStackEvents",
                "cloudformation:UpdateStack",
                "cloudformation:CreateStackSet",
                "cloudformation:DescribeStackResources",
                "cloudformation:DeleteStackSet",
                "cloudformation:DescribeStacks",
                "cloudformation:CreateStack",
                "cloudformation:DeleteStack"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "iam:CreateRole",
                "iam:GetRole",
                "iam:PassRole",
                "iam:PutRolePolicy",
                "iam:GetPolicy",
                "iam:DeleteRole",
                "iam:DeleteRolePolicy",
                "iam:CreatePolicy",
                "iam:UpdateRole",
                "iam:GetRolePolicy"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:CreateBucket",
                "s3:PutObject",
                "s3:GetObject"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "dynamodb:*",
                "cloudfront:*",
                "cognito-identity:*",
                "s3:*",
                "appsync:*",
                "lambda:*",
                "cognito-idp:*"
            ],
            "Resource": "*"
        }
    ]
  }
  ```

  The first 3 policy statement blocks contain neccessary IAM permissions for Amplify CloudFormation deployments to work. 

  The last policy statement contains permissions corresponding to AWS resources that are commonly used in Amplify (as an example): `auth`, `api`, `hosting`, `storage`, `function`. 
  
  You will **NEED TO ADD** more permissions corresponding to the resources your amplify application uses. 
  
  You may further constraint it down to specific service actions. You will find this will require you to iteratively deploy while tweaking IAM permissions until deployment succeeds. This is normal - if it was easy, your IAM policy is probably not very secure.

5. In the previous page group creation dropdown, find a newly created policy in the list, add a name (`AmplifyDeploy`) and click on Create Group.
6. Select a newly created group for this new user, click through the other steps and finish creating a new user.
7. Copy the access key and secret access key into your github repository Secrets (in repo's Settings) as **AWS_ACCESS_KEY_ID**,  **AWS_SECRET_ACCESS_KEY**.
8. You can also learn more at [Controlling Access with AWS Identity and Access Management](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-iam-template.html)

## Inputs

### amplify_command

**type**: `string`  
**values**: `configure | push | publish | status | add_env | delete_env`

#### configure
**required parameters**: `amplify_env`

Configures amplify and initializes specified amplify environment, which is required to exist prior to running this command.

#### push

Perform cloudformation deployment of your latest amplify updates. You are required to run this step with `configure` command prior to running this. 

**Note #0:** this won't additionally build and deploy your front-end artifacts. Use [publish](#publish) for this.  
**Note #1:** don't forget to run `amplify env pull` locally to synchronize the stacks status aftwards

#### publish

Perform cloudformation deployment of your latest amplify updates as well as front-end artifacts if hosting category is used in your project. You are required to run this step with `configure` command prior to running this.

**Note:** don't forget to run `amplify env pull` locally to synchronize the stacks status aftwards

#### status

Shows the state of local resources not yet pushed to the cloud (Create/Update/Delete). You are required to run this step with `configure` command prior to running this.

#### add_env
**required parameters**: `amplify_env`

Creates and initialized a new amplify environment. You would likely need this if you want to create a full replica of production environment for running integration tests (Refer to [Replicating Environment for Integration Tests](#replicating-environment-for-integration-tests)). **IMPORTANT**: make sure to always run this step together with [delete_env](#delete_env) command since this new environment won't be added to your project's configuration and you would need to manually delete the leftover cloudformation stack and S3 bucket otherwise.
  
**Note #0**: **WILL FAIL** with `resource already exists` exception if you repeatedly populate the environment that you have undeployed previously **WHEN** you are using storage category in your project and its CF `AWS::S3::Bucket` resource has **Retain** `DeletionPolicy`, since `delete_env` step won't remove such S3 bucket.  
**Note #1**: may take significant time if you are utilizing `AWS CloudFront` in your hosting category.

#### delete_env
**required parameters**: `amplify_env, delete_lock`

Undeploys cloudformation stack(removes all resources) for a selected amplify environment. To prevent accidental deletion, you are required to explicitly set [delete_lock](#delete_lock) input parameter with `false`. For the same reason, this step will fail if you try running it on the enivonment with name containing `prod/release/master`. 

**Note #0**: results in leftover amplify environment S3 bucket since `amplify env delete` won't remove this S3 bucket. (this will not affect repeated population of the environment with the same name as new population will create S3 bucket with different name)  
**Note #1**: repeated population of environment with the same name **WILL FAIL** with `resource already exists` exception if you repeatedly populate the environment that you have undeployed previously **WHEN** you are using storage category in your project and its CF `AWS::S3::Bucket` resource has **Retain** `DeletionPolicy`, since `delete_env` step won't remove such S3 bucket.  
**Note #2**: may take significant time if you are utilizing `AWS CloudFront` in your hosting category.

### amplify_env
**type**: `string`  
**required**: `YES` for amplify_commands: `configure, add_env, delete_env`.

Name of amplify environment used in this step.

### amplify_cli_version
**type**: `string`  
**required** `NO`

Use custom amplify version instead of latest stable (npm's `@latest`) when parameter is not specified.

### project_dir
**type**: `string`  
**required**: `NO`

the root amplify project directory (contains `/amplify`): use it if you amplify project is not this repo root directory.

### delete_lock
**type**: `bool`  
**required** `YES` for `delete_env` amplify_command  
**default**: true

deletion protection: explicitly set this to false if you want `delete_env` step to work.

### amplify_arguments
**type**: `string`  
**required** `NO`

additional arguments to pass to ampify_command's defined command

## Advanced Examples

### Replicating environment for integration tests

You may soon find the need of running fully-fledged tests that would test the actual API calls and other functionality available in your infrustructure rather their mocked counterparts. This is achieved in the next example by means of populating the new amplify environment, running all the necessary tests and undeploying amplify environment back. PR branch name is used for environment name. Please note that subsequent commits to PR branch may fail with `resource already exist` if your amplify category resources use [DeletionPolicy](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-attribute-deletionpolicy.html) in its CF templates that is set to `Retain`.
Also note, each deployment results in leftover amplify environment S3 bucket (named `amplify-{PROJECT_NAME}-{ENV_NAME}-{ID}-deployment`) since `amplify env delete` won't remove this S3 bucket. (this will not affect repeated population of the environment with the same name as new population will create S3 bucket with different name)  

```yaml
name: 'Integration Tests'
on: [pull_request]

jobs:
  test:
    name: test amplify-cli-action
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [10.x]

    steps:
    - uses: actions/checkout@v1

    - name: use node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v1
      with:
        node-version: ${{ matrix.node-version }}

    - name: set amplify env name
      id: setenvname
      run: |
        # use GITHUB_HEAD_REF that is set to PR source branch
        # also remove -_ from branch name and limit length to 10 for amplify env restriction
        echo "##[set-output name=amplifyenvname;]$(echo ${GITHUB_HEAD_REF//[-_]/} | cut -c-10)"
    - name: deploy test environment
      uses: SharpNub/amplify-cli-action@1.0.0
      with:
        amplify_command: add_env
        amplify_env: ${{ steps.setenvname.outputs.amplifyenvname }}
        amplify_cli_version: '9.2.1'
      env:
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        AWS_REGION: us-east-1

    - name: install, build and run integration tests
      run: |
        # build and test
        # npm install
        # npm run build
        # npm run test
    
    - name: undeploy test environment
      uses: SharpNub/amplify-cli-action@1.0.0
      # run even if previous step fails
      if: failure() || success()
      with:
        amplify_command: delete_env
        amplify_env: ${{ steps.setenvname.outputs.amplifyenvname }}
        amplify_cli_version: '9.2.1'
        delete_lock: false
      env:
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        AWS_REGION: us-east-1
    
```

As an alternative, one practical way could be to have a fixed sandbox environment that all PRs will update regardless of the branch (and doesn't get undeployed), so it can be used as a playground to manually test and play around with upcoming updates, but kind in mind there can be potential additional costs involved as some AWS resources used in amplify have fixed by-hours costs (kinesis for example).
