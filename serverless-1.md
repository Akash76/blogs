# Baby steps with serverless

Hello everyone, This is Akash. I've seen lot of hype about serverless in the recent days. I think it is time I tell a few things about Serverless. I first heard about serverless in 2018 when I was in college. Back then, I have zero knowledge about software development. I've come pretty long way since then and have pretty good experience with development. I'd like to talk about how I started with serverless and some basic things you need to know to get started. Lets jump in!!

## First step - Introduction

This pandemic has given everyone a lot of time to familiarize themselves with new technologies and latest advancements. In any application development, the developer should take care of everything from the scratch.Right from writing code to deploying it, developer is solely responsible. And when we want to scale our application we may need to use docker, kubernetes, aws load balancers etc which require some provision. Now how easy it would be if there is a way that allows you to concentrate only on developing business logic without worrying about scaling infrastructure? This is what serverless offers us.

If you think serverless works independent of the servers, then you are wrong. I've been there too. The main purpose of serverless is to make the development work much easier without worrying about infrastructure. All we have to do is to design good work flow, write an efficient code, test it and deploy it. Our work ends there and service providers will take care of rest. Do you want to know how? Lets see...

## Second step - Making hands dirty

AWS Lambda is a serverless compute service that helps you run code without provisioning or managing servers. The Serverless Framework helps you develop and deploy your AWS Lambda functions, along with the AWS infrastructure. To make things simpler, I'll show you how to install serverless using npm. Note that you install npm in your system beforehand!

```sh
$ npm install -g serverless
```
This command will install serverless globally so that you can initialise project wherever you want in your pc. Serverless offers us some templates to kickstart application development. These templates differ from provider to provider.

```sh
$ sls create --template aws-nodejs --path helloworld
```
Here I am using "aws-nodejs" template. --path argument creates a directory to hold our files. If the above command is executed perfectly, the output will be as shown.

```
Serverless: Generating boilerplate...
Serverless: Generating boilerplate in "/home/akash/Desktop/serverless/helloworld"
 _______                             __
|   _   .-----.----.--.--.-----.----|  .-----.-----.-----.
|   |___|  -__|   _|  |  |  -__|   _|  |  -__|__ --|__ --|
|____   |_____|__|  \___/|_____|__| |__|_____|_____|_____|
|   |   |             The Serverless Application Framework
|       |                           serverless.com, v2.43.1
 -------'

Serverless: Successfully generated boilerplate for template: "aws-nodejs"
```

Go into helloworld folder and see the generated files. You have serverless.yml file which is heart of our project. The code that we write will be evaluated and deployed depending on how it was configured in serverless.yml file. You can get a lot of information there and if you plan to know even more, check [this](https://www.serverless.com/framework/docs/providers/aws/guide/serverless.yml/)

Next is handler.js. All our business logic resides here and will be exported from here in the future. A simple hello world code can be like this.

```js
module.exports.hello = async (event) => {
  return {
    statusCode: 200,
    body: JSON.stringify(
      {
        message: 'Hello World!!!',
      },
      null,
      2
    ),
  };
};
```

## Third step - Functions and Events

The concept of serverless is Function-as-a-Service(FaaS). Aforementioned, all you need to do is write your code as functions and manage it.

Anything that triggers and executes the AWS Lambda Function is regarded by the framework as an Event. There are lots of events such as HTTP API gateway, S3, Cloudwatch, dynamoDB etc. So note that a function will be executed based on the triggered event. Lets configure our hello world with API gateway. Under 'functions' in serverless.yml, add this

```yaml
functions:
  hello:
    handler: handler.hello
    events:
      - httpApi:
          path: /hello
          method: get
```

Lets see what happens here. If we hit API gateway with given path '/hello', execute handler.hello function that is exported from handler.js. Our serverless.yml file looks likes this after the subjected changes.

```yaml
service: helloworld
frameworkVersion: '2'

provider:
  name: aws
  runtime: nodejs12.x
  lambdaHashingVersion: 20201221

functions:
  hello:
    handler: handler.hello
    events:
      - httpApi:
          path: /hello
          method: get
```

Lets deploy the code.

```sh
$ sls deploy
```

This will bundle our code and will deploy into AWS. At last we will get an endpoint. If the deployment is successful, the output looks as follows.

```
Serverless: Stack update finished...
Service Information
service: helloworld
stage: dev
region: us-east-1
stack: helloworld-dev
resources: 11
api keys:
  None
endpoints:
  GET - https://ij4kp0zahl.execute-api.us-east-1.amazonaws.com/hello
functions:
  hello: helloworld-dev-hello
layers:
  None
```

You can see the endpoint section in the above output. Lets use curl to hit that end point.

```sh
$ curl https://ij4kp0zahl.execute-api.us-east-1.amazonaws.com/hello
```

```
{
  "message": "Hello World!!!"
}
```

You have created your first API. Simple, isn't it?

## Conclusion

What you've just seen is a very basic 'hello world'. Serverless helps you in ways more than you can imagine. Startups, SMBs, and multi-national companies like Netflix are dependent on serverless. AWS Lambda is a metered service (pay-per-use). You have access to unlimited resources but are charged based on the number of requests and the time taken for your code to execute. So all you have to do is to write an optimised code. In my coming blogs, I'll brief you how to use plugins, resource creation, using webpack, writing middlewares and many more.

Stay safe!!!