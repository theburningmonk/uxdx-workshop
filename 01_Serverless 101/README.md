# Module 1: Serverless Framework 101

## Installation

Follow the instruction and install the CLI from [http://serverless.com/](http://serverless.com/).

## Create a Serverless project

**Goal:** Create and deploy AWS Lambda handler code using `serverless create` command.

<details>
<summary><b>Create a new serverless project</b></summary><p>

1. Create a directory for your serverless project.

```bash
mkdir hello-world
cd hello-world
```

2. Initialise the project:

`npm init -y`

This would initialize the project's `package.json` with default values. We'll come back to this later.

3. Install the `Serverless` framework as dev dependency.

`npm install --save-dev serverless`

Add `sls` to npm scripts by editing your `package.json` so your `scripts` section looks like this:

```json
"scripts": {
  "sls": "serverless"
},
```

Now you can run serverless using `npm run sls [-- <args>...]`

The special option `--` is used to delimit the end of the options for `npm run` and pass all the arguments after the `--` directly to your script.

**IMPORTANT: there needs to be a whitespace after `--`.** e.g. `npm run sls -- create` instead of `npm run sls --create`

> _Pro tip:_ Most examples gives steps to install and run Serverless Framework globally (allowing you to directly call `serverless` in your terminal). However, global package dependency will likely to cause issues in the future between two projects depending on incompatible major versions, especially when used by build and deploy steps on your CI.

4. Create nodejs Serverless project using one of the default templates:

`npm run sls -- create --template aws-nodejs`

See more information about `serverless create` command on [CLI documentation](https://serverless.com/framework/docs/providers/aws/cli-reference/create/) page.

</p></details>

<details>
<summary><b>Configure the serverless application</b></summary><p>

1. Modify the `serverless.yml` file, rename `service` to `hello-world-` followed by your name - e.g. `hello-world-yancui`.

2. Go to `handler.js`, and replace the whole module with the following:

```javascript
module.exports.hello = async event => {
  return {
    statusCode: 200,
    body: JSON.stringify({
      message: 'hello world'
    })
  };
}
```

3. Modify the `serverless.yml` file, under `functions`, so that the definition for the `hello` function looks like this:

```yml
hello:
  handler: handler.hello
  events:
    - http:
        path: /
        method: get
```

The `handler` is now pointing to the `hello` function.

A function can have different event triggers, which we can configure in the `events` array.

Here, we configure a REST API endpoint in API Gateway as the event source for our Lambda function.

</p></details>

<details>
<summary><b>Invoke function locally</b></summary><p>

1. Run `invoke local` command:

`npm run sls -- invoke local --function hello`

See more information about `invoke local` command on [CLI documentation](https://serverless.com/framework/docs/providers/aws/cli-reference/invoke-local/) page.

2. Verify that the function returns the following output:

```json
{
  "statusCode": 200,
  "body": "{\"message\":\"hello world\"}"
}
```

</p></details>

<details>
<summary><b>Deploy a serverless project</b></summary><p>

1. Run `deploy` command:

`npm run sls -- deploy`

See more information about `deploy` command on [CLI documentation](https://serverless.com/framework/docs/providers/aws/cli-reference/deploy/) page.

2. This creates an API in Amazon API Gateway. In the output you should see something like this:

```
endpoints:
  GET - https://xxxxx.execute-api.us-east-1.amazonaws.com/dev/
```

Curl the endpoint and see that it returns a 200 response, with the JSON payload:

```json
{
  "message": "hello world"
}
```

</p></details>

Congratulations! You have successfully successfully created and deployed your first Serverless API project.

<details>
<summary><b>Add description to the functions</b></summary><p>

1. Consult the [Serverless framework docs](https://serverless.com/framework/docs/providers/aws/guide/serverless.yml/) to see all the different configuration options available

2. Modify the `serverless.yml` and add descriptions to the functions

3. Deploy the functions with `npm run sls -- deploy`

4. Go to the Lambda console to see the functions have been updated with descriptions

</p></details>

<details>
<summary><b>Customize the function names</b></summary><p>

The Serverless framework enforces a naming convention, but you can override the convention.

1. Consult the [Serverless framework docs](https://serverless.com/framework/docs/providers/aws/guide/serverless.yml/) to see how you can override function names

2. Deploy the functions with `npm run sls -- deploy`

3. Go to the Lambda console to see the functions have been renamed

</p></details>

<details>
<summary><b>Deploy to another stage/environment</b></summary><p>

1. Deploy the project to a `test` stage (aka environment) with `npm run sls -- deploy --stage test`

2. Go to API Gateway console to see that another API has been created for the `test` stage

3. Go to the Lambda console to see the functions that been created for the `test` stage

4. Note the naming conventino the Serverless framework applies to both functions and APIs

</p></details>

<details>
<summary><b>Remove the test stage</b></summary><p>

1. Delete the `test` stage (the Lambda function and REST API) with `npm run sls -- remove --stage test`. This would delete all the resources associated with the `test` stage.

2. Go to the Lambda console to see the deployed functions are deleted

3. Go to the API Gateway console to see the deployed APIs are deleted

4. Go to the IAM console to see the IAM execution roles for the functions are deleted

5. Go to the CloudFormation console to see the CloudFormation stack is deleted

</p></details>
