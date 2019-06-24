# Serverless Airline Reservation Application

This is inspired by Heitor Lessa's Serverless Airline Reservation system on AWS (https://github.com/aws-samples/aws-serverless-airline-booking). 

     It is a complete web application that provides Flight Search, Flight Payment, Flight Preference, and Loyalty Points 
     including end-to-end testing and CI/CD. 
     
However I will be building custom tooling as we go along to simplify as many repeated activities as possible.

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

Now that we have a Swagger, let's create an API Gateway endpoint using the Swagger.
