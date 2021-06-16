# Baby steps with serverless

Hello everyone, This is Akash. I have seen lot of hype about serverless in recent days. I first heard about serverless in 2018 when I was in college and it was the time I donot have any idea about software development. Now I have pretty good experience with development and here I would like to talk about how I started with serverless and some basic things you need to know to get started. Lets jump in!!

## First step - 

This pandemic has given me lot of time to get introduced with new technologies and advancements in existing ones. In any application development, the developer should take care of everything right from writing code to deploying it. And when we want to scale our application we may need to use docker, kubernetes, aws load balancers etc which require some provision. What if there is a way in which we can concentrate only on developing business logic without worrying about scaling infrastructure? This is what serverless offers us.

If you think there are no servers involved in serverless, then you are wrong. Even I was wrong when I first heard about it. The thing is serverless makes the development work much more easier and there is no need to worry about infrastructure. Our only work is to design good work flow, write efficient code, test it and deploy it. And we are done. Our service provider will take care of rest. Do you want to see how? Lets see...

## Second step - Making hands dirty

The Serverless Framework helps you develop and deploy your AWS Lambda functions, along with the AWS infrastructure resources they require. To keep things simpler, I will show you how to install serverless using npm. You need to have npm in your system!

```sh
$ npm install -g serverless
```
Thats it. This command will install serverless globally so that you can initialise project wherever you want in your pc. Serverless gives us some templates for us to get started with and there are a lot of templates for different service providers.

```sh
$ sls create --template aws-nodejs --path helloworld
```
Here I am using "aws-nodejs" template. --path argument creates a directory to hold our files. If the above command executed perfectly you have to see below output.

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

Go into helloworld folder and see what files are generated for us. You have serverless.yml file which is heart of our project. The code that we write will be evaluated and deployed by seeing how it was configured in serverless.yml file. There is a lot of information that you can see there and if you want to know even more, check [this](https://www.serverless.com/framework/docs/providers/aws/guide/serverless.yml/)

Next is handler.js. All our code goes here. The business logic that we write will be exported from here. A simple hello world code can be like this.

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

The concept of serverless is Function-as-a-Service(FaaS). As I stated before, all you need to do is write your code as functions and manage it.

Anything that triggers an AWS Lambda Function to execute is regarded by the framework as an Event. There are lots of events like HTTP API gateway, S3, Cloudwatch, dynamoDB etc. So to understand much clearly, a function will be executed based on what event got triggered. Lets configure our hello world with API gateway. Under 'functions' in serverless.yml, add this

```yaml
functions:
  hello:
    handler: handler.hello
    events:
      - httpApi:
          path: /hello
          method: get
```

Here what we are saying is if we hit API gateway with given path '/hello', execute handler.hello function which we exported from handler.js. Our serverless.yml file looks likes this after we made our changes.

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
          path: /users/create
          method: get
```

Lets deploy the code.

```sh
$ sls deploy
```

This will bundle our code and will deploy into AWS. At last we will get an endpoint. You have to see output like below if the deployment is successful.

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

Its that simple. You have created your first API.

## Conclusion

Here what I showed is very basic and just hello world. There are a lot of things you can do with serverless. Startups and  big companies like Netflix are using serverless. With AWS Lambda, you pay only for what you use. You are charged based on the number of requests for your functions and the duration, the time it takes for your code to execute. So only thing you need to do is writing optimised code. In my next blogs I will show you how to use plugins, resource creation, using webpack, writing middlewares and many more.

Stay safe!!!