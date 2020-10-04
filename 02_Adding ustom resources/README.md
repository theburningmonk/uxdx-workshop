# Module 2: Adding custom resources

**Goal:** Configure custom resources using CloudFormation

<details>
<summary><b>Add a DynamoDB table in the serverless.yml</b></summary><p>

It's possible to add custom resources to the project using raw CloudFormation. 

1. Add DynamoDB table to `serverless.yml`. Add this block to the end of the file:

```yml
resources:
  Resources:
    RestaurantsTable:
      Type: AWS::DynamoDB::Table
      Properties:
        BillingMode: PAY_PER_REQUEST
        AttributeDefinitions:
          - AttributeName: name
            AttributeType: S
        KeySchema:
          - AttributeName: name
            KeyType: HASH
```

This is how we define a DynamoDB table in CloudFormation. Couple of things to note if you're new to AWS or CloudFormation.

`RestaurantsTable` is called the **logical Id** in CloudFormation, and is the unique ID for a resource within a CloudFormation template. Other resources can reference this resource using this logical id.

You typically use either the `!Ref` or `!GetAtt` function to reference a resource and get its attributes (like name or ARN). Which, depending on the resource and the attribute you want to get, you have to choose the right function... I have a cheatsheet [here](https://theburningmonk.com/cloudformation-ref-and-getatt-cheatsheet) that list all the resources and what you get with each.

As for DynamoDB, there are two billing modes. Here, we're using the `On-Demand` mode by setting `BillingMode` to `PAY_PER_REQUEST`. This means we don't pay for the table unless we use it, which is perfect for a demo app like this. In fact, this is probably the right setting for your app in production too unless you have very stable and very predictable load.

DynamoDB operates with `HASH` and `RANGE` keys as the only schemas you need to specify, so that's what `KeySchema` is doing. When you specify an attribute in the key schema you also need to add them to the `AttributeDefinitions` list so you can specify their type (`S` for `String`), this is how we tell DynamoDB that the table has a `HASH` key called `name` and it's a `S`tring.

There are other configurations available, but they're needed here. For more details, check out the CloudFormation docs [here](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-dynamodb-table.html).

Ok, now let's deploy the serverless project again (so CloudFormation provisions the table):

`npm run sls -- deploy`

After deployment finishes, the DynamoDB table would now be provisioned.

If you look into the DynamoDB console in your AWS account, you will see that the table would have an auto-generated name like `hello-world-yancui-dev-RestaurantsTable-1RF6ESC8XRRVY`.

At this point, it's also worth peeking into the CloudFormation console, and see that the Serverless framework generated a CloudFormation stack for our project, and the name of the stack is derived from the `service` and `stage` name of our project - e.g. `hello-world-yancui-dev`.

The stack includes a no. of outputs including the deployment bucket name, and the root URL for API Gateway REST API.

![](/images/mod02-001.png)

Since we have added a DynamoDB table to the stack as a custom resource, it's useful to add its name to the stack output.

2. To add the `RestaurantsTable` name to stack output, go to `serverless.yml`. Under the `resources` section, add the following:

```yml
  Outputs:
    RestaurantsTableName:
      Value: !Ref RestaurantsTable
```

**IMPORTANT**: `Outputs` should be aligned with `Resources` (**NOT** `resources`). So after this change, the `resources` section of your `serverless.yml` should look like this:

```yml
resources:
  Resources:
    RestaurantsTable:
      Type: AWS::DynamoDB::Table
      Properties:
        BillingMode: PAY_PER_REQUEST
        AttributeDefinitions:
          - AttributeName: name
            AttributeType: S
        KeySchema:
          - AttributeName: name
            KeyType: HASH

  Outputs:
    RestaurantsTableName:
      Value: !Ref RestaurantsTable
```

Make sure the indentations are exactly the same.

3. Deploy the project again

`npm run sls -- deploy`

and check in the CloudFormation console to make sure that you see the `RestaurantsTableName` in the stack outputs

![](/images/mod02-002.png)

</p></details>

<details>
<summary><b>Add a POST endpoint to add a new restaurant</b></summary><p>

Now that we have a table to hold a list of restaurants, let's add a new `POST` endpoint to populate it.

1. In the `serverless.yml`, add a new function under `functions`:

```yml
addRestaurant:
  handler: functions/add-restaurant.handler
  events:
    - http:
        path: /restaurant
        method: post
```

This adds a new `POST /restaurant` endpoint to our API.

**IMPORTANT**: this needs to be aligned with the `hello` function we defined in the last module. The `functions` section should look like this:

```yml
functions:
  hello: ...

  addRestaurant: ...
```

2. In the project root, add a folder called `functions`.

3. In the `functions` folder, add a new file called `add-restaurant.js`.

4. Copy the following into the `functions/add-restaurant.js` module.

```js
module.exports.handler = async (event) => {
  return {
    statusCode: 200
  }
}
```

This function needs to take the `POST` body (that includes information about the restaurant) and write it into the restaurants table.

We need to use the AWS SDK for that.

5. Install the AWS SDK as a dev dependency. Run

`npm i --save-dev aws-sdk`

The reason we install it as a dev dependency instead of a production dependency is because it's already available in the excution environment. Doing this helps us keep the upload package small and helps reduce cold start time too.

The new function also needs to know the name of the DynamoDB table to write to.

We can pass the name into the function via its environment variables.

6. In the `serverless.yml`, replace the `addRestaurant` function with this

```yml
addRestaurant:
  handler: functions/add-restaurant.handler
  environment:
    RESTAURANTS_TABLE_NAME: !Ref RestaurantsTable
  events:
    - http:
        path: /restaurants
        method: post
```

This adds a `RESTAURANTS_TABLE_NAME` environment variable to this function. We can use `!Ref` (a CloudFormation intrinsic function, see [here](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html) for more detail) to get the name of the `RestaurantsTable`.

7. Replace `functions/add-restaurant.js` with the following:

```js
const DynamoDB = require('aws-sdk/clients/dynamodb')
const DocumentClient = new DynamoDB.DocumentClient()

const { RESTAURANTS_TABLE_NAME } = process.env

module.exports.handler = async (event) => {
  await DocumentClient.put({
    TableName: RESTAURANTS_TABLE_NAME,
    Item: JSON.parse(event.body)
  }).promise()

  return {
    statusCode: 200
  }
}
```

Since the DynamoDB table schema uses `name` as the `HASH` key, the `POST` payload must include a `name` attribute. This handler function would then save the payload into the table as it is.

The last thing we need is to give the `addRestaurant` function the necessary IAM permissions.

8. In the `serverless.yml`, under `provider`, add the following:

```yml
iamRoleStatements:
  - Effect: Allow
    Action: dynamodb:PutItem
    Resource: !GetAtt RestaurantsTable.Arn
```

**IMPORTANT**: make sure this block is aligned with `provider.environment`.

After this step, the `provider` section should look like

```yml
provider:
  name: aws
  runtime: nodejs12.x
  environment: ...
  iamRoleStatements: ...
```

9. Deploy the project again.

`npm run sls -- deploy`

10. Curl the new `POST` endpoint to make sure it's working.

`curl -d '{"name":"myRestaurant"}' -H "Content-Type: application/json" -X POST https://xxx.execute-api.us-east-1.amazonaws.com/dev/restaurants`

This should return a 200 response.

Have a look in the DynamoDB table in the AWS console and see that the restaurant has indeed been added.

</p></details>

<details>
<summary><b>Enable HTTP Keep-Alive for AWS SDK</b></summary><p>

The single most effective optimization for Node.js function is to enable HTTP Keep-Alive for the AWS SDK. It can easily save you 10s of ms of invocation time, every invocation (see [here](https://theburningmonk.com/2019/02/lambda-optimization-tip-enable-http-keep-alive/) for more detail).

We want to do this for ALL of our functions.

The Serverless framework lets you set environment variables that are shared by all the functions in the project.

1. In the `serverless.yml`, add the following to the `provider` section:

```yml
environment:
  AWS_NODEJS_CONNECTION_REUSE_ENABLED: '1'
```

The `AWS_NODEJS_CONNECTION_REUSE_ENABLED` tells the AWS SDK to enable HTTP Keep-Alive. By setting the environment variable at the `provider` level, it'll be inherited by all of the functions. 

The `provider` section should look like this afterwards:

```yml
provider:
  name: aws
  runtime: nodejs12.x
  environment:
    AWS_NODEJS_CONNECTION_REUSE_ENABLED: '1'
```

2. Deploy the project again.

`npm run sls -- deploy`

And make sure the `POST` endpoint still works.

`curl -d '{"name":"myRestaurant"}' -H "Content-Type: application/json" -X POST https://xxx.execute-api.us-east-1.amazonaws.com/dev/restaurants`

</p></details>
