# ngrok-sdk-serverless

An example of using the ngrok-js NodeJS SDK in an [AWS App Runner](https://aws.amazon.com/apprunner/) serverless application.

# Prerequisites

1. A fork of this repository made into your own Github account user
1. An AWS account with App Runner enabled

# Non-Production Use Case
This method will get an app up and running as quickly as possible, but will not protect the Auth Token to a high enough standard for Production use cases.

1. Navigate to the App Runner service within the AWS console
1. Click `Create service`
1. Source: Choose `Source code repository`
    1. Click `Add New`
        1. Follow the prompts to connect AWS to your forked repository. More information is available [here](https://docs.aws.amazon.com/apprunner/latest/dg/getting-started.html#getting-started.create)
    1. It should automatically choose the connected repository, otherwise select it
    1. Click `Next` at the bottom
1. Build settings: Choose `Configure all settings here`
    1. Runtime: `Nodejs 16`
    1. Build command: `npm install`
    1. Start command: `npm start`
    1. Click `Next` at the bottom
1. Service name: `ngrok-serverless`
    1. Environment variables: Add environment variable:
        1. Plain text (not for Prod use): name: `NGROK_AUTHTOKEN`, value: paste in your ngrok Auth Token
    1. Networking: Incoming: Choose `Private endpoint`
        1. Click `Create new endpoint`
            1. Select VPC, Subnets: select all availability zones
            1. Click `Create`
        1. Choose the newly created VPC Endpoint
    1. Click `Next` at the bottom
1. Click `Create & Deploy` at the bottom
1. After the service is deployed, you can find the ngrok ingress URL by clicking on the link in the `Application logs` section of the service details page. You may need to refresh using the circular arrow button

# Production Use Case
For production use the Auth Token must be pulled from an encrypted store. This can be done through AWS Secrets Manager or SSM Parameter Store, the latter is free so the example will use SSM Parameter Store. See the [AWS documentation](https://docs.aws.amazon.com/apprunner/latest/dg/env-variable.html#env-variable.sensitivedata) for more information on how to do this.

## Add secret
1. Create secrets in the AWS Systems Manager Parameter Store
    1. Navigate to the Parameter Store service within the AWS console
    1. Click `Create parameter`
    1. Name: `/ngrok-serverless/ngrok-authtoken`
    1. Type: SecureString
    1. Value: paste in your ngrok auth token
    1. Click `Create parameter`
1. In the forked Github repository: Edit the `apprunner.yaml` file to change the `NGROK_AUTHTOKEN` environment variable to use the AWS Secrets Manager secret
    1. Uncomment and fill in the region, account_id, and parameter_name of the secret for this line: `value-from: "arn:aws:ssm:<region>:<aws_account_id>:parameter/<parameter_name>`
    1. Commit and push the changes to your forked repository
1. Create IAM role for App Runner to access the secret
    1. Navigate to the IAM service within the AWS console
    1. Click `Roles`
    1. Click `Create role`
        1. Select `Custom trust policy`
        1. Paste in the `Example IAM Role Trust Policy` from below, replacing the existing JSON text
        1. Click `Next`
    1. Click `Create policy`. In the new tab:
        1. Click `JSON`
            1. Paste in the `Example IAM Policy` from below, replacing the region, account_id, and parameter_name with the values from the secret
            1. Click `Next`
        1. Policy name: `ngrok-serverless-policy`
            1. Click `Create policy`
    1. Back in the Create role tab:
        1. Click the two-arrows refresh button to update the list of policies
        1. Search for `ngrok`
        1. Check the box next to the `ngrok-serverless-policy` that was created in the previous step
        1. Click `Next`
    1. Role name: `ngrok-serverless-role`
        1. Click `Create role`

### Example IAM Role Trust Policy
Example IAM Role Trust Policy from the [AWS documentation](https://docs.aws.amazon.com/apprunner/latest/dg/security_iam_service-with-iam.html#security_iam_service-with-iam-roles):
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "tasks.apprunner.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

### Example IAM Policy
Example IAM Policy from the [AWS documentation](https://docs.aws.amazon.com/apprunner/latest/dg/env-variable.html#env-variable.sensitivedata.permissions):
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ssm:GetParameters"
      ],
      "Resource": [
        "arn:aws:ssm:<region>:<aws_account_id>:parameter/<parameter_name>"
      ]
    }
  ]
}
```

## Add App Runner service
1. Navigate to the App Runner service within the AWS console
1. Click `Create service`
1. Source: Choose `Source code repository`
    1. Click `Add New`
        1. Follow the prompts to connect AWS to your forked repository. More information is available [here](https://docs.aws.amazon.com/apprunner/latest/dg/getting-started.html#getting-started.create)
    1. It should automatically choose the connect repository, otherwise select it
    1. Click `Next` at the bottom
1. Build settings: Choose `Use a configuration file`
    1. Click `Next` at the bottom
1. Service name: `ngrok-serverless`
    1. Security: Instance role
        1. Select the IAM role created in the previous step, e.g. `ngrok-serverless-role`
    1. Networking: Incoming: Choose `Private endpoint`
        1. Click `Create new endpoint`
            1. Select VPC, Subnets: select all availability zones
            1. Click `Create`
        1. Choose the newly created VPC Endpoint
    1. Click `Next` at the bottom
1. Click `Create & Deploy` at the bottom
1. After the service is deployed, you can find the ngrok ingress URL within your [ngrok dashboard](https://dashboard.ngrok.com) as it will show up as a connected agent

# Next Steps

See the [App Runner documentation](https://docs.aws.amazon.com/apprunner/latest/dg/) for more information on how to use the service.

See the [ngrok-nodejs documentation](https://github.com/ngrok/ngrok-nodejs) for more information on how to use the ngrok NodeJS SDK library.

# License

This project is licensed under either of

 * Apache License, Version 2.0, ([LICENSE-APACHE](LICENSE-APACHE) or
   http://www.apache.org/licenses/LICENSE-2.0)
 * MIT license ([LICENSE-MIT](LICENSE-MIT) or
   http://opensource.org/licenses/MIT)

at your option.

### Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in ngrok-nodejs by you, as defined in the Apache-2.0 license, shall be
dual licensed as above, without any additional terms or conditions.