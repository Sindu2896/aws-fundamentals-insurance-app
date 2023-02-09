# Serverless Framework Node Insurance Example

This template demonstrates how to develop and deploy a simple Node lambda service, backed by DynamoDB database, running on AWS Lambda using the traditional Serverless Framework.


## Usage

### Deployment

Install dependencies with:

```
npm install
```

and then deploy with:

```
serverless deploy
```

After running deploy, you should see output similar to:

```bash
Deploying insurance-app-demo to stage dev (us-east-1)

✔ Service deployed to stack insurance-app-demo-dev (196s)

endpoint: ANY - https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com
functions:
  api: aws-node-express-dynamodb-api-project-dev-api (766 kB)
```

_Note_: In current form, after deployment, your API is public and can be invoked by anyone. For production deployments, you might want to configure an authorizer. For details on how to do that, refer to [`httpApi` event docs](https://www.serverless.com/framework/docs/providers/aws/events/http-api/). Additionally, in current configuration, the DynamoDB table will be removed when running `serverless remove`. To retain the DynamoDB table even after removal of the stack, add `DeletionPolicy: Retain` to its resource definition.

### Invocation

After successful deployment, you can create a new user by calling the corresponding endpoint:

```bash
curl --request POST 'https://xxxxxx.execute-api.us-east-1.amazonaws.com/users' --header 'Content-Type: application/json' --data-raw '{"username": "John", "role": "member", "orgName": "Insighture"}'
```

Which should result in the following response:

```bash
{"userId":"someUserId","username":"John","role": "member", "orgName": "Insighture"}
```

You can later retrieve the user by `userId` by calling the following endpoint:

```bash
curl https://xxxxxxx.execute-api.us-east-1.amazonaws.com/users/someUserId
```

Which should result in the following response:

```bash
{"userId":"someUserId","username":"John","role": "member", "orgName": "Insighture"}
```

If you try to retrieve user that does not exist, you should receive the following response:

```bash
{"error":"Record Not Found for userId: \"userId\""}
```

### Local development

It is also possible to emulate DynamoDB, API Gateway and Lambda locally using the `serverless-dynamodb-local` and `serverless-offline` plugins. In order to do that, run:

```bash
serverless plugin install -n serverless-dynamodb-local
serverless plugin install -n serverless-offline
```

It will add both plugins to `devDependencies` in `package.json` file as well as will add it to `plugins` in `serverless.yml`. Make sure that `serverless-offline` is listed as last plugin in `plugins` section:

```
plugins:
  - serverless-dynamodb-local
  - serverless-offline
```

You should also add the following config to `custom` section in `serverless.yml`:

```
custom:
  (...)
  dynamodb:
    start:
      migrate: true
    stages:
      - dev
```

Additionally, we need to reconfigure `AWS.DynamoDB.DocumentClient` to connect to our local instance of DynamoDB. We can take advantage of `IS_OFFLINE` environment variable set by `serverless-offline` plugin and replace:

```javascript
const dynamoDbClient = new AWS.DynamoDB.DocumentClient();
```

with the following:

```javascript
const dynamoDbClientParams = {};
if (process.env.IS_OFFLINE) {
  dynamoDbClientParams.region = 'localhost'
  dynamoDbClientParams.endpoint = 'http://localhost:8000'
}
const dynamoDbClient = new AWS.DynamoDB.DocumentClient(dynamoDbClientParams);
```

After that, running the following command with start both local API Gateway emulator as well as local instance of emulated DynamoDB:

```bash
serverless offline start
```

To learn more about the capabilities of `serverless-offline` and `serverless-dynamodb-local`, please refer to their corresponding GitHub repositories:
- https://github.com/dherault/serverless-offline
- https://github.com/99x/serverless-dynamodb-local
