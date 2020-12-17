---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - python
  - javascript

toc_footers:
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true

code_clipboard: true
---

# Introduction

Welcome to the Conscent Client API! You can use our APIs to access Conscent Client API endpoints, which can help you get information on various tasks such as how to create stories, verify story payment etc.

We have language bindings in Shell (cURL), Python, and JavaScript! You can view code examples in the dark area to the right, and you can switch the programming language of the examples with the tabs in the top right.

# Authentication

Conscent uses API keys to allow access to the API for the Client to Create and Register a Story with Conscent. You can view your API Key and Secret by logging in to your Conscent Client Dashboard and navigating to the Integrations Page [Client Integrations Page](https://conscent.vercel.app/client/dashboard/integration). The Login Credentials for the Client Dashboard are provided on your official email address registered with Conscent. 

Conscent expects for the API Key and Secret to be included as Basic Authentication in certain API requests (Create Story API utilised by the Client).

`Authorization: Basic RDZXN1Y4US1NTkc0V1lDLVFYOUJQMkItOEU3QjZLRzpUNFNHSjlISDQ3TVpRWkdTWkVGVjZYUk5TS1E4RDZXN1Y4UU1ORzRXWUNRWDlCUDJCOEU3QjZLRw==`

<aside class="notice">
You must replace <code>RDZXN1Y4US1NTkc0V1lDLVFYOUJQMkItOEU3QjZLRzpUNFNHSjlISDQ3TVpRWkdTWkVGVjZYUk5TS1E4RDZXN1Y4UU1ORzRXWUNRWDlCUDJCOEU3QjZLRw==</code> with your personal Client API Key and Secret
</aside>

# Client Story

## Create Story

```python
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

This endpoint allows the Client to Register their Story on Conscent - with the Story Title, StoryId and Price. Moreover, the Client can also set the duration of a story - which means that if a story if purchased by a User on Conscent, then that User can have free access to the story for {duration} amount of time. By Default we user a 1 Day duration.

### HTTP Request

`POST stage.tsbdev.co/api/v1/story`

### Authorization

Client API Key and Secret must be passed in Authorization Headers using Basic Auth. With API Key as the Username and API Secret as the password. 

### Request Body

Parameter | Default | Description
--------- | ------- | -----------
storyId | required | Story Id by which the Story has been registered on the Client CMS
title | required | Title of the Story 
price | required | Story Price to be charged for a Pay per View basis (Micro-priced)
duration | required | Free story access time for user once the user has purchased the story. (Standard Practice - 1 Day);

<aside class="success">
Remember — A story must be registered by including all the fields mentioned above!
</aside>

## Edit Story

> Please ensure you URL encode the storyId passed in the Path Parameters

```python
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
    "message": "New Story Created!",
    "story": {
        "title": "Test story for API functionality",
        "price": 0.1,
        "storyId": "testingID For Client"
    }
  },
]
```

This endpoint allows the Client to Edit their Registered Story on Conscent - with the editable fields being the Story title, price and duration. Story ID CANNOT be edited!

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
price | optional | Story Price to be charged for a Pay per View basis (Micro-priced)
duration | optional | Free story access time for user once the user has purchased the story. (Standard Practice - 1 Day);

<aside class="success">
Remember — Either/All of the fields of a story - title, price and duration - can be edited. Only pass the fields you wish to edit in the request body.
</aside>

## Delete a Specific Kitten

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.delete(2)
```

```shell
curl "http://example.com/api/kittens/2" \
  -X DELETE \
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.delete(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "deleted" : ":("
}
```

This endpoint deletes a specific kitten.

### HTTP Request

`DELETE http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to delete

