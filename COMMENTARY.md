# Serverless Airline Reservation Application

This is inspired by Heitor Lessa's Serverless Airline Reservation system on AWS (https://github.com/aws-samples/aws-serverless-airline-booking). 

     It is a complete web application that provides Flight Search, Flight Payment, Flight Preference, and Loyalty Points 
     including end-to-end testing and CI/CD. 
     
However I will be building custom tooling as we go along to simplify as many repeated activities as possible.

- [Serverless Airline Reservation Application](#serverless-airline-reservation-application)
  * [Legend](#legend)
  * [Use Cases](#use-cases)
    + [Search](#search)
    + [Book](#book)
    + [Manage](#manage)
  * [24-June-2019](#24-june-2019)
    + [@activity Create and initialize a workspace](#activity-create-and-initialize-a-workspace)
    + [@script `mkdirg`: Create a new directory with .gitkeep](#script-mkdirg-create-a-new-directory-with-gitkeep)
    + [Search Design & Architecture](#search-design--architecture)
      - [Functionality](#functionality)
      - [UI Components](#ui-components)
      - [Backend Routes](#backend-routes)
      - [Data Source: Airports](#data-source-airports)
      - [Data Source: Flights Catalog](#data-source-flights-catalog)
    + [@activity Barebones website with search form](#activity-barebones-website-with-search-form)
    + [@activity Barebones HTTP backend for `/search` endpoint](#activity-barebones-http-backend-for-search-endpoint)
      - [@activity Authoring Swagger](#activity-authoring-swagger)
      - [@activity Creating HTTP Endpoint with Swagger](#activity-creating-http-endpoint-with-swagger)
        * [Create API Gateway Deployment](#create-api-gateway-deployment)
          + [Adding x-amazon-apigateway-integration` to Swagger](#adding-x-amazon-apigateway-integration-to-swagger)
      - [@activity Create a Lambda function](#activity-create-a-lambda-function)
        * [@activity Create an IAM Role for Lambda](#activity-create-an-iam-role-for-lambda)

## Legend

To call out specific activities, I will use the following annotations:

**@activity**: This is a new idea I have

**@script**: I am writing a script for a repeated activity

**@tool**: I am converting a script to a tool

**@surprise**: When my expectation from an activity did not match with the result

You can also search `git log` for these annotations. There will usually be one commit matching the annotations here
if you want to look up the code for activity described here.

## Use Cases
This app will support the following use cases: 

### Search
A customer should be able to provide a departure airport, destination airport, and date of travel to retrieve available 
flights. Works for "one way" flights only.

### Book
After a flight is selected, a customer should be able to provide information about them, their credit card, and make a
payment. 

### Manage
A customer should be able to view their existing reservations. They should be able to cancel or update an existing 
reservation.

## 24-June-2019
Today we will be building the Search use case. 

### @activity Create and initialize a workspace
```bash
AIRLINE=~/workspace/github-repos/serverless-airline
AIRLINE_APP=$AIRLINE/app
AIRLINE_SCRIPTS=$AIRLINE/scripts
AIRLINE_TOOLS=$AIRLINE/tools

mkdir $AIRLINE
mkdir $AIRLINE_APP
mkdir $AIRLINE_SCRIPTS
mkdir $AIRLINE_TOOLS

cd $AIRLINE
git init

# Add `scripts` directory to PATH so I can easily access it from my shell
echo 'AIRLINE=~/workspace/github-repos/serverless-airline' >> ~/.zshrc
echo 'export PATH="$PATH:$AIRLINE/scripts"' >> ~/.zshrc
source ~/.zshrc

# Add .gitkeep files to each directory so I can actually commit these folders
touch $AIRLINE_APP/.gitkeep
touch $AIRLINE_SCRIPTS/.gitkeep
touch $AIRLINE_TOOLS/.gitkeep


# Commit everything
git add .
git commit -m "@activity Create and initialize a workspace"
```

### @script `mkdirg`: Create a new directory with .gitkeep
Since we have created a new directory with `.gitkeep` file thrice, I am going to write a script for it. This is a 
non-serverless specific activity that I am not going to create a tool, instead use a script.

```bash
git add scripts/mkdirg
git commit -m "@script `mkdirg`: Create a new directory with .gitkeep"
```

### Search Design & Architecture
To keep the application minimal, I am going to design & implement one use-case at a time. 
If there are repeated components, I will refactor them into common modules later.

#### Functionality

* UX:
    * Enter departure and arrival airports by searching for airport code
    * Enter departure date
    * Hit the search button to display list of results
    
* Business Rules:
    * Departure & Arrival airport code cannot be the same
    * Departure date cannot be in the past or too far in future
    * Domestic US flights only
    
#### UI Components
* Search Form: Departure Airport, Arrival Airport, Departure Date, Search button. When Search Button is pressed,
  performs a HTTP POST to backend to retrieve results.
  
* *Date Picker*: To select the date of departure. For now, we can do with a `<input type="date"/>` because this 
  provides sufficient functionality
  
#### Backend Routes
* `GET /` - Returns homepage HTML that contains the search UI
* `GET /airport/search?q=prefix` - Given the prefix user entered on the text box, search and return list of matching
  airports. Used to populate the "From" and "To" text boxes
* `POST /search` - Returns search results

#### Data Source: Airports
This data source provides a list of airport names and their codes. Capable of searching by airport name & airport code 
prefix.

#### Data Source: Flights Catalog
This data source contains all operational flight routes, schedules, cancellations etc. Given a departure airport, 
arrival airport, and departure date, this data source is capable of returning all available flights.


### @activity Barebones website with search form 
**Start: 2019-06-24 09:00**
**End:   2019-06-24 09:30**

```bash
# Create separate directory for all website components because this is usually using a different software stack 
# than backend
mkdirg $AIRLINE_APP/www
touch $AIRLINE_APP/www/index.html

# Write the index.html file
# View file in browser
open index.html 

git add $AIRLINE_APP/www
git commit -m "@activity Barebones website with search form"
```

### @activity Barebones HTTP backend for `/search` endpoint
**Start: 2019-06-24 09:50**

Fun stuff begins here. We are going to:

1. Create a serverless HTTP endpoint for `POST /search` that returns a simple HTML message "form success"
1. Wire up `index.html` to submit the form to this endpoint
1. Test form submission to ensure the success message is shown

AWS API Gateway is the serverless service that provides highly available & scalable HTTP endpoints. We will first model
our API in the Swagger JSON language. We will then provide the Swagger file to API Gateway to create the endpoints
for us. For those of you that don't know, Swagger is a [open specification](https://swagger.io/resources/open-api/) 
to model RESTful APIs.

#### @activity Authoring Swagger 
**End:   2019-06-24 10:35**


```bash
# Create the search usecase folder
mkdirg $AIRLINE_APP/search

# Create an empty Swagger file
touch $AIRLINE_APP/search/swagger.json
```

I don't remember the Swagger specification. Let me Google and find a snippet for Swagger that I can copy, paste, and 
modify.

```
Google for "api gateway swagger example"

Open https://cloudonaut.io/create-a-serverless-restful-api-with-api-gateway-swagger-lambda-and-dynamodb/

Copy the Swagger snippet from this website to swagger.json
```

**@surprise PyCharm automatically checks Swagger Schema**: I edited the Swagger file to remove `info: {}` and suddenly my PyCharm 
gave a big bright error highlight. Apparently, "info" is required by Swagger schema. PyCharm automatically applied the
Swagger's JSON Schema from SchemaStore.org to validate this file.

I am now editing the Swagger file to make it work for `POST /search` endpoint. The example gave me a response schema,
but I don't know how to add a request schema. After some googling, I found
[https://swagger.io/docs/specification/2-0/describing-request-body/](https://swagger.io/docs/specification/2-0/describing-request-body/)
which contains request schema format.

```bash
git add .
git commit -m "@activity Authoring Swagger"
```

#### @activity Creating HTTP Endpoint with Swagger
**Status: ABANDONED**
**Start: 2019-06-24 11:49**
**End:   2019-06-24 12:49**

Now that we have a Swagger, let's create an API Gateway endpoint using the Swagger.


```
Google for "creating api gateway endpoint using swagger"

Open "https://docs.aws.amazon.com/apigateway/latest/developerguide/create-api-using-swagger.html"

Found Console instructions. But found CLI instructions at the end. So copy & pasting now.
```

```bash
cd $AIRLINE_APP/search 

# Import the Swagger 
aws apigateway import-rest-api --body file://`pwd`/swagger.json

# @surprise This command actually worked and gave me the following response!
{
    "id": "d0bkpi6xth",
    "name": "Serverless Airline Reservation",
    "createdDate": 1561402235,
    "version": "1.0",
    "apiKeySource": "HEADER",
    "endpointConfiguration": {
        "types": [
            "EDGE"
        ]
    }
}
```

Now that I have the API created, I need to get the URL. By browsing on APIGW documentation left panel, I found this 
page: [https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-call-api.html](https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-call-api.html).
This page says I can construct the endpoint like this: `https://{restapi_id}.execute-api.{region}.amazonaws.com/{stage_name}/`.

From response above, the `id` property looks like the `{restapi_id}` in the URL template. It is deployed to default region
which should be `us-east-1`. I still have to figure out what `{stage_name}` is. 

```
Google "apigateway stage name"

Open "https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-deploy-api.html"

Wrong page. 

But browsing the left panel on doc page, found another page

Open "https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-deployments.html"

Turns out I have to create a "Deployment", followed by "Update Stage", and then "Deploy Again to the Stage"
```

##### Create API Gateway Deployment
**Start: 2019-06-24 12:20**


```bash
# Creating the deployment

# I had to save & retrieve the APIGW ID from somewhere. So let me store it in a script now
$ aws apigateway create-deployment --rest-api-id d0bkpi6xth
# ^^FAILED: An error occurred (BadRequestException) when calling the CreateDeployment operation: Invalid deployment content specified.CreateDeploymentInput should not be null.

# Let's look at help
$ aws apigateway create-deployment help
# Looks like there is a --stage-name property

# Let's create a deployment with the --stage-name = prod
aws apigateway create-deployment --rest-api-id d0bkpi6xth --stage-name prod
# ^^FAILED: An error occurred (BadRequestException) when calling the CreateDeployment operation: No integration defined for method
```

I don't know what the error is. So let's Google!

```
Google "CreateDeployment operation: No integration defined for method"

Open https://github.com/awslabs/serverless-application-model/issues/5

By parsing, looks like I need to add `x-amazon-apigateway-integration` to Swagger. From the URL, I found another
example, which looks promising:

Open https://github.com/awslabs/serverless-application-model/blob/master/examples/2016-10-31/api_swagger_cors/swagger.yaml#L41
```

######  Adding x-amazon-apigateway-integration` to Swagger

I added the following snippet to Swagger. 

> NOTE: I know this is wrong because I know CloudFormation and Fn::Sub is a CFN construct. But being a naive user of AWS,
  I don't expect to know the nuances.

```json
  "x-amazon-apigateway-integration": {
      "uri": {
        "Fn::Sub": "arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${LambdaFunction.Arn}/invocations"
      },
      "httpMethod": "POST",
      "type": "aws_proxy"
  }
```

Now that we have updated Swagger, we should pass Swagger back to API Gateway. Based on our previous experience, 
let's go re-import the API and re-deploy.

```console

$ cd $AIRLINE_APP/search

$ aws apigateway import-rest-api --body file://`pwd`/swagger.json
An error occurred (BadRequestException) when calling the ImportRestApi operation: Unable to parse API definition because of a malformed integration at path /search.
```

Looking at the integration URI above, looks like the integration needs Lambda Function because the URI contains a 
Lambda Function ARN.

May be I am doing this all backwards. I am going to abandon this approach and start with a Lambda instead.


#### @activity Create a Lambda function
**Start: 2019-06-24 12:54**
**End:   2019-06-24 13:37**


```
Google "creating a lambda function in python"

After exploring 4 links which all gave "Console" based walkthrough, I landed at this one.
Open "https://docs.aws.amazon.com/lambda/latest/dg/python-programming-model.html"

Still talks about Console based walkthrough. 

Open "https://docs.aws.amazon.com/lambda/latest/dg/lambda-python-how-to-create-deployment-package.html"
Piecing together information from different pages, I am creating a lambda file now
```

```bash
cd $AIRLINE_APP/search
touch $AIRLINE_APP/search/function.py

# Copy paste handler code from https://docs.aws.amazon.com/lambda/latest/dg/python-programming-model-handler-types.html
# Create the zip
zip function.zip function.py

# Upload function to Lambda
aws lambda update-function-code --function-name python37 --zip-file fileb://function.zip
# ^^ FAILED: An error occurred (ResourceNotFoundException) when calling the UpdateFunctionCode operation: Function not found: arn:aws:lambda:us-east-1:1235:function:python37
# Reading help page, looks like this is not the right command to create the function with code.

aws lambda create-function --function-name search-function --zip-file fileb://function.zip --runtime python37 --handler function.handler 
# ^^FAILED: Looks like I need a Role ARN for this
```

##### @activity Create an IAM Role for Lambda

```
Google "aws lambda create function"

Open "https://medium.com/@jacobsteeves/aws-lambda-from-the-command-line-7efab7f3ebd9"

Turns out this is the only post that explains how to set everything up manually
```

```bash

cat '{
    "Version": "2012-10-17",
    "Statement": [{
        "Effect": "Allow",
        "Principal": { "AWS" : "*" },
        "Action": "sts:AssumeRole"
    }]
}' > /tmp/basic_lambda_role.json

aws iam create-role --role-name basic_lambda_role --assume-role-policy-document file://basic_lambda_role.json

# This worked and produced a Role
# arn:aws:iam::283125242610:role/basic_lambda_role

```

Now that the role is created, I can go ahead and create the function

```bash
aws lambda create-function --function-name search-function --zip-file fileb://function.zip --runtime python3.7 --handler function.handler --role arn:aws:iam::283125242610:role/basic_lambda_role

# This created the function!

git add .
git commit -m "@activity Create a Lambda function"
```

