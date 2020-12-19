---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - javascript
  - php

includes:
  - errors

# search: true

code_clipboard: true
---

# Getting Started

Welcome to ConsCent! Now you can begin micropricing your stories and create a whole new revenue stream. This documentation covers all the steps for you to get set up - starting from registering your premium stories with ConsCent which are priced for a Pay per View basis, followed by integrating the paywall on your story pages to and initialising the overlay to ensure all premium content is paid for.

You can start by registering your stories on ConsCent following the steps mentioned in the [Registration Page](#registration).

Once you have registered your stories on ConsCent, you need to integrate the paywall on all your Premium Content Pages. Follow the steps in the [Web Integration Section](#web-integration) to add the ConsCent paywall to all your story pages that have been micropriced. Ensuring that the paywall appears each time a premium story is opened as well as to allow users to purchase this story via ConsCent. 

Integrating the paywall allows the users to purchase the Client's Stories via ConsCent at Microprices rates. The Web Integration section includes the steps involved in handling the Success and Failure Callbacks after the users go through the Concent Paywall as well as validating each unique transaction incurred by a user on the any of the Clients Stories registered with ConsCent. 

# Registration

The first step is registering your client with ConsCent. This is done by the ConsCent Team and you will recieve your login credenitals (email & password) for the ConsCent Client Dashboard on the email you chose to register with. 

By logging in to your ConsCent Client Dashboard and navigating to the Integrations Page [Client Integrations Page](https://ConsCent.vercel.app/client/dashboard/integration). You will be able to view your active ClientId, API Key and API Secret. 

<p class = 'instruction-bg'>In order to start creating stories on ConsCent you need to ensure you follow the [Authentication Guidelines](#authentication). ConsCent will only allow authorised clients to create and edit stories. For using these client APIs, the Client API Key and Secret must be passed in Authorization Headers using Basic Auth. With API Key as the Username and API Secret as the password. </p>

Now you can start registering your premium content stories with ConsCent! Call the [Create Story API](#create-story) for any story that you wish to register with ConsCent. Ensure you provide all the required fields for creating the story - including the Story ID with which the story is registered on your Client CMS as well as the title, price (Pay per View Price) and duration for which the user can access the story after purchasing it. 

Moreover, you can edit any story registered with ConsCent using the [Edit Story API](#edit-story). Editable fields include the title, price and duration of the story. However, once the story is registered - it's Story Id cannot be changed.

# Web Integration

> ConsCent Paywall Initalisation Script:

```
<script>
  const clientId = '5f92a62013332e0f667794dc';
  (function (w, d, s, o, f, cid) {
    if (!w[o]) {
      w[o] = function () {
        w[o].q.push(arguments);
      };
      w[o].q = [];
    }
    (js = d.createElement(s)), (fjs = d.getElementsByTagName(s)[0]);
    js.id = o;
    js.src = f;
    js.async = 1;
    js.title = cid;
    fjs.parentNode.insertBefore(js, fjs);
  })(window, document, 'script', '_csc', 'https://csc-sdk.netlify.app/', clientId);
</script>
```

Integrating ConsCent on your Website is a simple and direct process. You start by copying the code on the right hand side within the script tags - and adding it to the header section of your Route Index file.

You can get your ConsCent Client Id from the [Client Integrations Page](https://ConsCent.vercel.app/client/dashboard/integration) of the ConsCent Client Dashboard - as mentioned in the [Registration Section](#registration).

<p class = 'instruction-bg'>Including this code in the header section allows the ConsCent Paywall to be initalised and further used whenever required on a Premium Content Story Page. </p>

<p class = 'instruction-bg'>Finally, In order to ensure the the ConsCent Paywall appears on all Story Pages which contain Premium Content. You need to implement the following function on the story page - included on the right hand side which calls the initalisation function '_csc' of the ConsCent Paywall and enables a user to purchase the particular premium story at a Micropriced value through ConsCent. Please ensure that you call this function anytime you want the ConsCent paywall to show up on your story page and that you have filtered for users that have subscribed to your platform beforehand and already have access to view the content and thus will not be going through the ConsCent paywall. </p>

> Include ConsCent Paywall on Premium Content Story Page:

```
const csc = window._csc as any;
csc('init', {
  debug: true,
  storyId: storyId,
  subscriptionUrl: {clientSubscriptionUrl},
  clientId: clientId,
  isMobile: 'false',
  successCallback: async (payload: any) => {
    // Function to show the premium content to the User since they have paid for it via ConsCent
  },
```

We import the initalisation script using the unique '_csc' identifier and run the 'init' function by passing a number of parameters:

  - The 'clientId' which is retrieved from the [Client Integrations Page](https://ConsCent.vercel.app/client/dashboard/integration) of the ConsCent Client Dashboard. 

  - The 'storyId' which should be identical to the Story Id by which the particular story is registered with in the Client CMS. This allows us to identify each unique story for a client. 

  - The 'subscriptionUrl' which is the link to the Subscription page of the clients website - in cases when a user would like to subscribe to the clients website for viewing the premium content offered. 

 <p class = 'instruction-bg'>Once the ConsCent Paywall has been initalised and the User has gone through the necessary steps needed to purchase the story via ConsCent - the client side needs to implement a 'successCallback' function which will recieve a response containing a payload shown below - indicating whether the user has purchased the story, or if the user has access to the story already since they have purchased it before, or whether the transaction has failed and the user has not purchased the story. </p>

<code> 
{
  "message": "Story Purchased Successfully",
  "accessTimeLeft": 86397686,
  "payload": {
      "clientId": "5fbb40b07dd98b0e89d90a25",
      "storyId": "Client Story Id 5",
      "transactionAmount": 0.01,
      "createdAt": "2020-12-17T10:58:51.828Z"
  },
  "readId": "c697833f-f473-4135-9f7c-ce92892d46d9",
}
</code>

The message "Story Purchased Successfully" appears in the response only when the user has purchased a story via ConsCent and the "accessTimeLeft" field appears in the response only when the user has purchased this story previously and still has free access to view the content. Moreover, the response contains a "readId" field which can be used to verify each unique transaction by a user on the clients registered stories on ConsCent. 

<p class = 'instruction-bg'>Calling the [Validate Story Read](#validate-story-read) API using the recieved "readId" in the successCallback response can assist the client in authenticating valid and unique transactions on their stories. The response payload includes the same fields as mentioned in the payload above and matching the payload from the 'successCallback' response and 'Validate Story Read' response allows the client to ensure each transaction of premium content stories via ConsCent is validated by matching the clientId, storyId, transactionAmount and createdAt(Date of Read/Transaction); </p>
 
If the case arrives when the user tries to purchase a story via ConsCent on the client's website and the transaction fails. The client can handle that case as well in a 'failedCallback' function or redirect to any page - as the Client wishes. 

Lastly, once the transaction has been validated by the Client - whether they choose to do it in the frontend or the backend - the client needs to show the premium content purchased by the User. You can do this on the same page, or redirect the user to a different page containing the full content of the premium story. 
# API Introduction

Welcome to the ConsCent Client API! You can use our APIs to access ConsCent Client API endpoints, which can help you get information on various tasks such as how to create stories, verify story payment etc.

We have language bindings in Shell (cURL), Python, and JavaScript! You can view code examples in the dark area to the right, and you can switch the programming language of the examples with the tabs in the top right.

# Authentication

ConsCent uses API keys to allow access to the API for the Client to Create and Register a Story with ConsCent. You can view your API Key and Secret by logging in to your ConsCent Client Dashboard and navigating to the Integrations Page [Client Integrations Page](https://ConsCent.vercel.app/client/dashboard/integration). The Login Credentials for the Client Dashboard are provided on your official email address registered with ConsCent. 

ConsCent expects for the API Key and Secret to be included as Basic Authentication in certain API requests (Create Story API utilised by the Client).

`Authorization: Basic RDZXN1Y4US1NTkc0V1lDLVFYOUJQMkItOEU3QjZLRzpUNFNHSjlISDQ3TVpRWkdTWkVGVjZYUk5TS1E4RDZXN1Y4UU1ORzRXWUNRWDlCUDJCOEU3QjZLRw==`

<aside class="notice">
You must replace <code>RDZXN1Y4US1NTkc0V1lDLVFYOUJQMkItOEU3QjZLRzpUNFNHSjlISDQ3TVpRWkdTWkVGVjZYUk5TS1E4RDZXN1Y4UU1ORzRXWUNRWDlCUDJCOEU3QjZLRw==</code> with your personal Client API Key and Secret
</aside>

# Client Story

## Create Story

```php
<?php

$curl = curl_init();

curl_setopt_array($curl, array(
  CURLOPT_URL => "http://localhost:3001/api/v1/story/",
  CURLOPT_RETURNTRANSFER => true,
  CURLOPT_ENCODING => "",
  CURLOPT_MAXREDIRS => 10,
  CURLOPT_TIMEOUT => 0,
  CURLOPT_FOLLOWLOCATION => true,
  CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
  CURLOPT_CUSTOMREQUEST => "POST",
  CURLOPT_POSTFIELDS =>"{\n    \"storyId\" : \"testingID For Client\",\n    \"title\" : \"Test story for API functionality\",\n    \"price\" : 0.10,\n    \"duration\" : 2\n}",
  CURLOPT_HTTPHEADER => array(
    "Authorization: Basic RDZXN1Y4US1NTkc0V1lDLVFYOUJQMkItOEU3QjZLRzpUNFNHSjlISDQ3TVpRWkdTWkVGVjZYUk5TS1E4RDZXN1Y4UU1ORzRXWUNRWDlCUDJCOEU3QjZLRw==",
    "Content-Type: application/json"
  ),
));

$response = curl_exec($curl);

curl_close($curl);
echo $response;
```

```shell
curl -X POST 'http://stage.tsbdev.co/api/v1/story/' \
-H 'Authorization: Basic RDZXN1Y4US1NTkc0V1lDLVFYOUJQMkItOEU3QjZLRzpUNFNHSjlISDQ3TVpRWkdTWkVGVjZYUk5TS1E4RDZXN1Y4UU1ORzRXWUNRWDlCUDJCOEU3QjZLRw==' \
-H 'Content-Type: application/json' \
-d '{
    "storyId" : "testingID For Client",
    "title" : "Test story for API functionality",
    "price" : 0.10,
    "duration" : 2
}'
```

```javascript
var axios = require('axios');
var data = JSON.stringify({"storyId":"testingID For Client","title":"Test story for API functionality","price":0.1,"duration":2});

var config = {
  method: 'post',
  url: 'http://stage.tsbdev.co/api/v1/story/',
  headers: { 
    'Authorization': 'Basic RDZXN1Y4US1NTkc0V1lDLVFYOUJQMkItOEU3QjZLRzpUNFNHSjlISDQ3TVpRWkdTWkVGVjZYUk5TS1E4RDZXN1Y4UU1ORzRXWUNRWDlCUDJCOEU3QjZLRw==', 
    'Content-Type': 'application/json'
  },
  data : data
};

axios(config)
.then(function (response) {
  console.log(JSON.stringify(response.data));
})
.catch(function (error) {
  console.log(error);
});
```

> The above command returns JSON structured like this:

```json
[
  {
    "message": "New Story Created!",
    "story": {
        "title": "Test story for API functionality",
        "price": 0.1,
        "storyId": "testingID For Client"
    }
  },
]
```

This endpoint allows the Client to Register their Story on ConsCent - with the Story Title, StoryId and Price. Moreover, the Client can also set the duration of a story - which means that if a story if purchased by a User on ConsCent, then that User can have free access to the story for {duration} amount of time. By Default we user a 1 Day duration. Lastly, the price of the story can only be set as a distinct value chosen by the client - which can be any out of [0, 0.01, 0.10, 1, 5]. These prices are in INR and ONLY these values can be set as the price of the story - otherwise the API call for creating a story will throw a 400 (Bad Request) Error.

### HTTP Request

`POST stage.tsbdev.co/api/v1/story`

### Authorization

Client API Key and Secret must be passed in Authorization Headers using Basic Auth. With API Key as the Username and API Secret as the password. 

### Request Body

Parameter | Default | Description
--------- | ------- | -----------
storyId | required | Story Id by which the Story has been registered on the Client CMS
title | required | Title of the Story 
price | required | Story Price to be selected out of [0, 0.01, 0.10, 1, 5] ONLY. Values are in INR by default.
duration | required | Free story access time for user once the user has purchased the story. (Standard Practice - 1 Day);

<aside class="notice">
Remember — A story must be registered by including all the fields mentioned above!
</aside>

## Edit Story

> Please ensure you URL encode the storyId in the Path Parameters
> Replace the {storyId} in the API path with your actual Story Id

```php
<?php

$curl = curl_init();

curl_setopt_array($curl, array(
  CURLOPT_URL => "http://localhost:3001/api/v1/story/Client%20Story%20Id%2011",
  CURLOPT_RETURNTRANSFER => true,
  CURLOPT_ENCODING => "",
  CURLOPT_MAXREDIRS => 10,
  CURLOPT_TIMEOUT => 0,
  CURLOPT_FOLLOWLOCATION => true,
  CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
  CURLOPT_CUSTOMREQUEST => "PATCH",
  CURLOPT_POSTFIELDS =>"{\n    \"title\": \"Client Story Id Test Edit\",\n    \"price\": 1,\n    \"duration\": 2\n}",
  CURLOPT_HTTPHEADER => array(
    "Authorization: Basic RDZXN1Y4US1NTkc0V1lDLVFYOUJQMkItOEU3QjZLRzpUNFNHSjlISDQ3TVpRWkdTWkVGVjZYUk5TS1E4RDZXN1Y4UU1ORzRXWUNRWDlCUDJCOEU3QjZLRw==",
    "Content-Type: application/json"
  ),
));

$response = curl_exec($curl);

curl_close($curl);
echo $response;

```

```shell
curl -X PATCH 'http://stage.tsbdev.co/api/v1/story/{storyId}' \
-H 'Authorization: Basic RDZXN1Y4US1NTkc0V1lDLVFYOUJQMkItOEU3QjZLRzpUNFNHSjlISDQ3TVpRWkdTWkVGVjZYUk5TS1E4RDZXN1Y4UU1ORzRXWUNRWDlCUDJCOEU3QjZLRw==' \
-H 'Content-Type: application/json' \
-d '{
    "title": "Client Story Id Test Edit",
    "price": 1,
    "duration": 2
}'
```

```javascript
var axios = require('axios');
var data = JSON.stringify({"title":"Client Story Id Test Edit","price":1,"duration":2});

var config = {
  method: 'patch',
  url: 'http://stage.tsbdev.co/api/v1/story/{storyId}',
  headers: { 
    'Authorization': 'Basic RDZXN1Y4US1NTkc0V1lDLVFYOUJQMkItOEU3QjZLRzpUNFNHSjlISDQ3TVpRWjnJL877NJSjnkHSk5TS1E4RDZXN1Y4UU1ORzRXWUNRWDlCUDJCOEU3QjZLRw==', 
    'Content-Type': 'application/json'
  },
  data : data
};

axios(config)
.then(function (response) {
  console.log(JSON.stringify(response.data));
})
.catch(function (error) {
  console.log(error);
});
```

> The above command returns JSON structured like this:

```json
[
  {
    "message": "Story Edited Successfully",
    "editedStory": {
        "title": "Client Story Id Test Edit",
        "storyId": "Client Story Id 11",
        "price": 1,
        "duration": 2
    }
  },
]
```

This endpoint allows the Client to Edit their Registered Story on ConsCent - with the editable fields being the Story title, price and duration. Story ID CANNOT be edited!

### HTTP Request

`PATCH stage.tsbdev.co/api/v1/story/{storyId}`

### Authorization

Client API Key and Secret must be passed in Authorization Headers using Basic Auth. With API Key as the Username and API Secret as the password.

### URL Parameters

Parameter | Description
--------- | -----------
storyId | The ID of the Story you wish to edit ( Ensure its URL Encoded)

### Request Body

Parameter | Default | Description
--------- | ------- | -----------
title | optional | Title of the Story 
price | optional | Story Price to be selected out of [0, 0.01, 0.10, 1, 5] ONLY. Values are in INR by default.
duration | optional | Free story access time for user once the user has purchased the story. (Standard Practice - 1 Day);

<aside class="notice">
Remember — Either/All of the fields of a story - title, price and duration - can be edited. Only pass the fields you wish to edit in the request body. Moreover, keep in mind that story price must be one of the following - [0, 0.01, 0.10, 1, 5]. Price values are in INR by default. 
</aside>

# Validate Story Read

## Validate Story Details By Read ID

> Replace the {readId} in the API path with the actual value/string of the readId recieved

```php
<?php

$curl = curl_init();

curl_setopt_array($curl, array(
  CURLOPT_URL => "http://localhost:3001/api/v1/story/read/11c369df-2a30-4a0d-90dc-5a45797dacdd",
  CURLOPT_RETURNTRANSFER => true,
  CURLOPT_ENCODING => "",
  CURLOPT_MAXREDIRS => 10,
  CURLOPT_TIMEOUT => 0,
  CURLOPT_FOLLOWLOCATION => true,
  CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
  CURLOPT_CUSTOMREQUEST => "POST",
));

$response = curl_exec($curl);

curl_close($curl);
echo $response;
```

```shell
curl -X POST 'http://stage.tsbdev.co/api/v1/story/read/{readId}'
```

```javascript
var axios = require('axios');

var config = {
  method: 'post',
  url: 'http://stage.tsbdev.co/api/v1/story/read/{readId}',
  headers: { }
};

axios(config)
.then(function (response) {
  console.log(JSON.stringify(response.data));
})
.catch(function (error) {
  console.log(error);
});
```

> The above command returns JSON structured like this:

```json
[
  {
    "message": "Story Read Confirmed",
    "payload": {
        "clientId": "5fbb40b07dd98b0e89d90a25",
        "storyId": "Client Story Id",
        "transactionAmount": 1,
        "createdAt": "2020-11-15T13:55:36.659Z"
    },
    "readId": "11c369df-2a30-4a0d-90dc-5a45797dacdd"
  },
]
```

This endpoint allows the Client to Validate the Story Details anytime a User Purchases a Story of the Client using ConsCent - with the Client passing the Story ID in the request and recieving details regarding the Story Id, Transaction Amount and Date of Purchase of the Story by the User, as well as matching the unique Read Id to ensure it cannot be reused. 

### HTTP Request

`POST stage.tsbdev.co/api/v1/story/read/{readId}`

The client will recieve a payload when a story is purchased via ConsCent - providing the details of the story such as the ClientId to identify which client the story belongs to as well as the Client Story ID, Transaction Amount and Date of the Transaction. Moroever, they will also recieve a Read ID - by passing the Read ID in this API request - the Client can verify the authenticity of the transaction and restrict any spillage. 

### URL Parameters

Parameter | Default | Description
--------- | ------- | -----------
readId | required | readId recieved by the client for each unique transaction on any of the Client's Stories registered with ConsCent

<aside class="notice">
Remember — The Read ID is unique and once it is used it will expire. Clients can use this endpoint to ensure the unique transactions that occur on their stories registered with ConsCent. 
</aside>

