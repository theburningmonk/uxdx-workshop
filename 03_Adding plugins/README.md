# Module 3: Adding plugins

**Goal:** Use plugins to extend Serverless framework

One of the most powerful things about the Serverless framework is the fact that it has a rich ecosystem of plugins that can extend the framework's capabilities.

For example, to follow best practices for security, you should assign a separate IAM role for each function and restrict the permissions to just what the function needs.

By default, the Serverless framework uses a shared IAM role. But this can be changed with the `serverless-iam-roles-per-function` plugin.

<details>
<summary><b>Configure per-function IAM role</b></summary><p>

1. Install the `serverless-iam-roles-per-function` plugin as a dev dependency.

`npm i --save-dev serverless-iam-roles-per-function`

2. In the `serverless.yml` add the following to the end of the file

```yml
plugins:
  - serverless-iam-roles-per-function
```

This plugin allows you to specify the IAM permissions for each function.

3. Remove the `iamRoleStatements` block from the `provider` section.

4. Add an `iamRoleStatements` block to the `addRestaurant` function instead:

```yml
iamRoleStatements:
  - Effect: Allow
    Action: dynamodb:PutItem
    Resource: !GetAtt RestaurantsTable.Arn
```

The `addRestaurant` function should look like this afterwards

```yml
addRestaurant:
  handler: functions/add-restaurant.handler
  environment:
    RESTAURANTS_TABLE_NAME: !Ref RestaurantsTable
  events:
    - http:
        path: /restaurants
        method: post
  iamRoleStatements:
    - Effect: Allow
      Action: dynamodb:PutItem
      Resource: !GetAtt RestaurantsTable.Arn
```

5. Deploy the project again

`npm run sls -- deploy`

and make sure the `POST` endpoint still works.

`curl -d '{"name":"myOtherRestaurant"}' -H "Content-Type: application/json" -X POST https://xxx.execute-api.us-east-1.amazonaws.com/dev/restaurants`

</p></details>

You can go to the official plugins list [here](https://www.serverless.com/plugins/) to browse the available plugins, curated from the community. Here are some of the most popular plugins:

* [serverless-offline](https://github.com/dherault/serverless-offline): simulates running API Gateway locally
* [serverless-webpack](https://github.com/serverless-heaven/serverless-webpack): use webpack to bundle functions before deployment
* [serverless-domain-manager](https://github.com/amplify-education/serverless-domain-manager): creates custom domain names and create Route53 record set for API Gateway
* [serverless-python-requirements](https://github.com/UnitedIncome/serverless-python-requirements): bundles Python functions's dependencies
* [serverless-step-functions](https://github.com/serverless-operations/serverless-step-functions): configure Step Functions state machines in your serverless.yml
* [serverless-split-stacks](https://github.com/dougmoscrop/serverless-plugin-split-stacks): split the CloudFormation stack into nested stacks to get beyond CloudFormation's 200 resources limit per stack
* [serverless-finch](https://github.com/fernando-mc/serverless-finch): host static websites in S3
* [serverless-appsync-plugin](https://github.com/sid88in/serverless-appsync-plugin): configure AppSync APIs

and many many more.
