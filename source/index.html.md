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

Welcome to ConsCent! This documentation covers all the steps for you to get set up - from optionally registering your premium content with ConsCent - which priced for a Pay per View basis, followed by integrating the paywall on your content pages and initialising the overlay to ensure all premium content is paid for.

You can get started by following the steps mentioned on the [Using ConsCent Page](#using-conscent).

# Registration

The first step is registering your client with ConsCent. This is done by the ConsCent Team internally and your administrator will recieve the login credenitals (email & password) for the ConsCent Client Dashboard on the email they chose to register with. The client must provide a default value for the 'price', 'duration' and optionally geolocation overrides if the client wants geo-fenced pricing options. This will ensure that all the premium content on the clients website/app is micro-priced - even if the client doesn't register the content with ConsCent via API. (Price values are in INR)

By logging in to your ConsCent Client Dashboard and navigating to the [Integrations Page](https://client.conscent.in/dashboard/integration). You will be able to view your active ClientId, API Key and API Secret.

# Using ConsCent

- In order to start creating content on ConsCent you need to ensure you follow the [Authentication Guidelines](#authentication). ConsCent will only allow authorised clients to create, view and edit content. For using these client APIs, the Client API Key and Secret must be passed in Authorization Headers using Basic Auth. With API Key as the Username and API Secret as the password.

Whenever you're utilising the Web Integration Code or Calling any ConsCent APIs - you to update the API_URL and SDK_URL variables based on the environment you're operating in.

<ul class = 'instruction-bg-url'>
  <li>
    Sandbox Environment: 
    <ul>
      <li>API_URL:  https://sandbox-api.conscent.in</li>
      <li>SDK_URL:  https://sandbox-sdk.conscent.in</li>
    </ul>
  </li>
</ul>
<ul class = 'instruction-bg-url'>
  <li>
    Production Environment: 
    <ul>
      <li>API_URL:  https://api.conscent.in</li>
      <li>SDK_URL:  https://sdk.conscent.in</li>
    </ul>
  </li>
</ul>

- If you wish to ONLY use blanket pricing - which allows you to set all your premium content at the same default price and priceOverrides with which your client is registered - you can skip the registering/editing content steps and move on to integrating the paywall on all your premium content pages.

- Any content that you don't register with ConsCent will be priced using the Blanket Pricing parameters - which are determined when the client is registered.

- Now you can start registering your premium content with ConsCent! Call the [Create Content API](#create-content) for any content that you wish to register with ConsCent.

Once you have registered your digital content on ConsCent, you need to integrate the paywall on all your Premium Content Pages.

- Follow the steps in the [Web Integration Section](#web-integration) to add the ConsCent paywall to all the premium content pages on your website.

<!-- - Additionally, Follow the steps in the [React Native Integration Section](#react-native-integration) to add the ConsCent paywall to all the premium content pages on your Android/iOS application. -->

Ensuring that the paywall appears each time any premium content is opened, as well as allowing users to purchase the content via ConsCent.

- Next, you need to call the [Validate Content Consumption API](#validate-content-consumption) using the recieved "consumptionId" in the successCallback response, when a user purchases any of the client's content via ConsCent. This will assist the client in authenticating valid and unique transactions on their premium content.

Integrating the paywall allows the users to purchase any of the Client's Content via ConsCent at Micropriced rates. The Web Integration section includes the steps involved in handling the Success and Failure Callbacks after the users go through the Concent Paywall, followed by validating each unique transaction incurred by a user on the any of the Clients Content.

- To view a list of all the client's premium content registered on ConsCent you can call the [View All Content API](#view-all-content).

- To view the details of any individual client content you can call the [View Content Details API](#view-content-details).

- To edit any content registered with ConsCent you can call [Edit Content API](#edit-content).

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
  })(window, document, 'script', '_csc', {SDK_URL}, clientId);
</script>
```

> Ensure you replace the 'clientId' with your actual Client ID retrieved from the Conscent Client Dashboard Integration Page and the {SDK_URL} with the sdk url of an environment you want to use, e.g. 'https://sandbox-sdk.conscent.in' for sandbox

Integrating ConsCent on your Website is a simple and direct process. You start by copying the code on the right hand side within the script tags - and adding it to the header section of your Route Index file.

You can get your ConsCent Client Id from the [Client Integrations Page](https://client.conscent.in/dashboard/integration) of the ConsCent Client Dashboard - as mentioned in the [Registration Section](#registration).

<p class = 'instruction-bg'>Including this code in the header section allows the ConsCent Paywall to be initalised and further used whenever required on any Premium Content Page. </p>

<p class = 'instruction-bg'>Finally, In order to ensure that the ConsCent Paywall appears on all pages which contain Premium Content. You need to implement the following function on the content page - included on the right hand side which calls the initalisation function '_csc' of the ConsCent Paywall and enables a user to purchase the particular premium content at a Micropriced value through ConsCent. Please ensure that you call this function anytime you want the ConsCent paywall to show up on your content page and that you have filtered for users that have subscribed to your platform beforehand and already have access to view the content and thus will not be going through the ConsCent paywall. </p>

> Include ConsCent Paywall on Premium Content Content Page:

```
const csc = window._csc;
csc('init', {
  debug: true, // can be set to false to remove sdk non-error log output
  contentId: contentId,
  subscriptionUrl: {clientSubscriptionUrl},
  signInUrl: {clientSignInUrl},
  clientId: clientId,
  title: contentTitle,
  isMobile: 'false',
  buttonMode,
  successCallback: yourSuccessCallbackFunction,
  wrappingElementId: 'csc-paywall',
  fullScreenMode: 'false', // if set to true, the entire screen will be covered
})
```

> Success Callback example:

```
async function yourSuccessCallbackFunction(validationObject: any) {
  // Function to show the premium content to the User since they have paid for it via ConsCent
  // Here you should verify the  validationObject with our backend
  // And then you must provide access to the user for the complete content

  // example verification code:
  console.log('Initiating verification with conscent backend');
  const xhttp = new XMLHttpRequest(); // using vanilla javascript to make a post request
  const url = `${API_URL}/api/v1/content/consumption/${validationObject.consumptionId}`;
  xhttp.open('POST', url, true);
  // e is the response event
  xhttp.onload = (e) => {
    const backendConfirmationData = JSON.parse(e.target.response);

    // verifying that the validation received matches the backend data
    if (
      validationObject.consumptionId === backendConfirmationData.consumptionId &&
      validationObject.payloadclientId === backendConfirmationData.payload.clientId &&
      validationObject.payload.contentId === backendConfirmationData.payload.contentId
    ) {
      // Validation successful
      console.log('successful validation');
      // accessContent would be your function that will do all the actions that need to be done to unlock the entire content
      accessContent(true);
    }
  };
  xhttp.send();
}
```

We import the initalisation script using the unique '\_csc' identifier and run the 'init' function by passing a number of parameters:

- The 'clientId' which is retrieved from the [Client Integrations Page](https://client.conscent.in/dashboard/integration) of the ConsCent Client Dashboard.

- The 'contentId' which should be identical to the Content Id by which the particular content is registered with - in the Client CMS. This allows us to identify each piece of unique content for a client.

- The 'title' which should be the Content Title by which the particular content is registered with in the Client CMS.

- The 'subscriptionUrl' which is the link to the Subscription page of the clients website - in cases when a user would like to subscribe to the clients website for accessing the premium content offered.

- You can optionally provide the 'signInUrl' which is the link to the login page for already subscribed users on the clients platform - in cases when a user has already registered and paid for the clients subscription and would like to access premium content using their login credentials. Doing this will add a "Sign in here" text to be displayed below the "Subscribe" button.

- 'wrappingElementId' is a mandatory string which is the id of an element (e.g. a div with absolute positioning on your website) within which you want the paywall to be embedded. Your element should have a minimum width of 320 pixels and a minimum height of 550 pixels for the conscent paywall to fit properly.

- 'buttonMode' can be set to 'true' or 'false' (boolean) -- if true, the paywall will appear only as a button. This may be useful for content such as videos, songs and podcasts. Moreover, the 'button' paywall can be styled from the client dashboard directly - by passing the required CSS there.

- 'fullScreenMode' can be set to 'true' or 'false' (strings) -- if true, the first screen of the paywall will cover the entire webpage. This is useful if you don't want the premium content page to be visible at all once the user proceeds with the payment.

 <p class = 'instruction-bg'>Once the ConsCent Paywall has been initalised and the user has gone through the necessary steps needed to purchase the content via ConsCent - you need to implement a 'successCallback' function which will recieve a response containing a validationObject shown below - indicating whether the user has purchased the content, or if the user has access to the content already since they have purchased it before, or whether the transaction has failed and the user has not purchased the content. </p>

<code> 
{
    "message": "Content Purchased Successfully",
    "payload": {
        "clientId": "5fbb40b07dd98b0e89d90a25",
        "contentId": "Client Content Id 5",
        "createdAt": "2020-12-29T05:51:31.116Z"
    },
    "consumptionId": "a0c433af-a413-49e1-9f40-ce1fbd63f568",
    "signature": "74h9xm2479m7x792nxx247998975393x08y9hubrufyfy3348oqpqqpyg78fhfurifr3",
}
</code>

The message "Content Purchased Successfully" appears in the response only when the user has purchased content via ConsCent and the "accessTimeLeft" field appears in the response only when the user has purchased the content previously and still has free access to consume the content. Moreover, the response contains a "consumptionId" field which will be used to verify each unique transaction by a user on the clients content with ConsCent.

Calling the [Validate Content Consumption](#validate-content-consumption) API using the recieved "consumptionId" in the successCallback response can assist the client in authenticating valid and unique transactions on their premium content.

<p class = 'instruction-bg'>The response validationObject from the 'Validate Content Consumption' API includes the same fields as mentioned in the validationObject above and matching the payload from the 'successCallback' response and 'Validate Content Consumption' response allows the client to ensure each transaction of any premium content via ConsCent is validated by matching the clientId, contentId, and createdAt (Date of Consumption/Transaction); </p>
 
If the case arrives when the user tries to purchase any content via ConsCent on the client's website and the transaction fails. The client can handle that case as well in a 'failedCallback' function or redirect to any page - as the Client wishes.

Lastly, once the transaction has been validated by the Client - whether they choose to do it in the frontend or the backend - the client needs to provide access to the premium content purchased by the user. You can do this on the same page, or redirect the user to a different page containing the entirety of the premium content.

<!-- # React Native Integration

This section covers integrating the ConsCent paywall into your react native application. This works for both iOS and Android.

You start by including ConsCent's React Native plugin. Using the following command

<p class = 'instruction-bg'>
npm i -s react-native-conscent
</p>

This is a package written in react-native with no native code - it is a pure javascript implementation and can be used in both iOS and Android applications with no extra configuration.

> Include ConsCent Paywall on Premium Content Story Page of your Android/iOS Application:

```
import Conscent from 'react-native-conscent';

export default function App() {
  return (
    // When you want to render conscent's paywall, simply use the Conscent component
    <View style={{ flex: 1 }}>
      <Conscent
        clientId="your-client-id"
        storyId="your-story-id"
        successCallback={successCallback}
        subscriptionUrl="https://www.yoursite.com/yourstory"
        signInUrl="https://www.yoursite.com/login"
        subscriptionCallback={subscriptioncallback}
        mode="sandbox"
      />
    </View>
  );
}
```

You can get your ConsCent Client Id from the [Client Integrations Page](https://ConsCent.vercel.app/client/dashboard/integration) of the ConsCent Client Dashboard - as mentioned in the [Registration Section](#registration).

<p class = 'function-params'> Function Parameters </p>

| Parameter            | Description                                                                                                                                                     |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| clientId             | Retrieved from the Integrations Page on the Client Dashboard                                                                                                    |
| storyId              | Story Id by which the Story has been registered on the Client CMS                                                                                               |
| successCallback      | Function which will receive the success payload. See the success payload structure below.                                                                       |
| subscriptionUrl      | You may either set the subscriptionUrl or the subscriptionCallback. subscriptionUrl will open the given url in the browser when the subscribe button is clicked |
| signInUrl      | You can optionally provide the link to the login page for already subscribed users on the clients platform - in cases when a previously subscribed user would like to access premium content using their login credentials. Doing this will add a "Sign in here" text to be displayed below the "Subscribe" button. |
| subscriptionCallback | function which receives no arguments. This function is called when the user clicks the subscribe button                                                         |
| mode                 | either 'sandbox' or 'production' based on your credentials                                                                                                      |

In order to show the ConsCent paywall before your premium content, you need to include the &lt;Conscent &gt; component with the required props. Usually, you would like to show the paywall before your registered premium stories to users who haven't subscribed to your service.

<p class = 'instruction-bg'>An example of how to initiate the paywall is show on the right-hand side. Include the &lt;Conscent &gt; component with the required props.</p>

Once the ConsCent Paywall has been initalised and the User has gone through the necessary steps needed to purchase the story via ConsCent - the 'successCallback' function you have provided as a prop will be called containing an object shown below - indicating whether the user has purchased the story, or if the user has access to the story already since they have purchased it before recently.

Additionally, you must set either the Subscription URL or the Subscription Callback function to determine the what happens when the user clicks on the 'Subscribe' button present on the conscent paywall and instead of paying per view.

<code>
{
    "message": "Story Purchased Successfully",
    "payload": {
        "clientId": "5fbb40b07dd98b0e89d90a25",
        "storyId": "Client Story Id 5",
        "transactionAmount": 5,
        "earningAmount": 4,
        "createdAt": "2020-12-29T05:51:31.116Z"
    },
    "readId": "a0c433af-a413-49e1-9f40-ce1fbd63f568",
}
</code>

The message "Story Purchased Successfully" appears in the response only when the user has purchased a story via ConsCent and the "accessTimeLeft" field appears in the response only when the user has purchased this story previously and still has free access to view the content. Moreover, the response contains a "readId" field which can be used to verify each unique transaction by a user on the clients registered stories on ConsCent. Moreover, the payload contains the 'transactionAmount' that the user pays for the story as well as the 'earningAmount' that the client earns for each of their stories that a user purchases through ConsCent.

Call the [Validate Story Read](#validate-story-read) API using the recieved "readId" in the successCallback response to verify that the transaction was unique and valid. The validate story read endpoint can only be used once per readId to ensure authenticity. Basically, readId functions as a one-time token.

<p class = 'instruction-bg'>The response payload from the 'Validate Story Read' API includes the same fields as mentioned in the payload above and matching the payload from the 'successCallback' response and 'Validate Story Read' response allows the client to ensure each transaction of premium content stories via ConsCent is validated by matching the clientId, storyId, transactionAmount and createdAt(Date of Read/Transaction); </p>

<p class='instruction-bg'>If any field  received in the success callback payload doesn't match the backend data, it is a fraudulent transaction & access to the premium content must not be provided</p>

Lastly, once the transaction has been validated by the Client - whether they choose to do it in the frontend or the backend - the client needs to show the premium content purchased by the User. You can do this on the same screen, or in another screencontaining the full content of the premium story. In the screen where you show the premium content, you MUST also show the ConsCent floating action button, you can do this by including the &lt;Conscent.Fab&gt; component anywhere in your view.

<p class='instruction-bg'>The ConsCent floating action button must be visible when displaying your premium content that is priced on ConsCent. Do so by  including the &lt;Conscent.Fab&gt; component anywhere in your view</p> -->

# API Introduction

Welcome to the ConsCent Client API! You can use our APIs to access ConsCent Client API endpoints, which can help you get information on various tasks such as how to create content, verify payment for any content etc.

We have language bindings in Shell (cURL), PHP, and JavaScript! You can view code examples in the dark area to the right, and you can switch the programming language of the examples with the tabs in the top right.

# Authentication

ConsCent uses API keys to allow access to the API for the Client to Create and Register any Content with ConsCent. You can view your API Key and Secret by logging in to your ConsCent Client Dashboard and navigating to the Integrations Page [Client Integrations Page](https://client.conscent.in/dashboard/integration). Please contact your administrator for the Login Credentials to access the ConsCent Client Dashboard - provided on the official email address registered with ConsCent.

ConsCent expects for the API Key and Secret to be included as Basic Authentication in certain API requests (Ex. Create Content API utilised by the Client).

`Authorization: Basic RDZXN1Y4US1NTkc0V1lDLVFYOUJQMkItOEU3QjZLRzpUNFNHSjlISDQ3TVpRWkdTWkVGVjZYUk5TS1E4RDZXN1Y4UU1ORzRXWUNRWDlCUDJCOEU3QjZLRw==`

<aside class="notice">
You must replace <code>RDZXN1Y4US1NTkc0V1lDLVFYOUJQMkItOEU3QjZLRzpUNFNHSjlISDQ3TVpRWkdTWkVGVjZYUk5TS1E4RDZXN1Y4UU1ORzRXWUNRWDlCUDJCOEU3QjZLRw==</code> with your personal Client API Key and Secret
</aside>

# Client Content

## Create Content

```php

<?php

$curl = curl_init();

curl_setopt_array($curl, array(
  CURLOPT_URL => "{API_URL}/api/v1/content/",
  CURLOPT_RETURNTRANSFER => true,
  CURLOPT_ENCODING => "",
  CURLOPT_MAXREDIRS => 10,
  CURLOPT_TIMEOUT => 0,
  CURLOPT_FOLLOWLOCATION => true,
  CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
  CURLOPT_CUSTOMREQUEST => "POST",
  CURLOPT_POSTFIELDS =>'{
    "contentId" : "testingID For Client Content",
    "duration" : 30,
    "title" : "Test content for API functionality",
    "price" : 1,
    "currency": "INR",
    "url": "www.google.com",
    contentType: "STORY",
    "priceOverrides": {
        "country": [
            {
                "name": "GL",
                "price": 3
            },
            {
                "name": "IN",
                "price": 5
            },
            {
                "name": "US",
                "price": 7
            }
        ]
    },
    "download": {
      "url": "https://yourdownloadurl.com",
      "fileName": "Download File - Name",
      "fileType": "PDF"
    }
  }',
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
curl -X POST '{API_URL}/api/v1/content/' \
-H 'Authorization: Basic RDZXN1Y4US1NTkc0V1lDLVFYOUJQMkItOEU3QjZLRzpUNFNHSjlISDQ3TVpRWkdTWkVGVjZYUk5TS1E4RDZXN1Y4UU1ORzRXWUNRWDlCUDJCOEU3QjZLRw==' \
-H 'Content-Type: application/json' \
-d '{
    "contentId" : "testingID For Client Content",
    "duration" : 30,
    "title" : "Test content for API functionality",
    "price" : 1,
    "currency": "INR",
    contentType: "STORY",
    "url": "www.google.com",
    "priceOverrides": {
        "country": [
            {
                "name": "GL",
                "price": 3
            },
            {
                "name": "IN",
                "price": 5
            },
            {
                "name": "US",
                "price": 7
            }
        ]
    },
    "download": {
      "url": "https://yourdownloadurl.com",
      "fileName": "Download File - Name",
      "fileType": "PDF"
    }
}'
```

```javascript
var axios = require("axios");
var data = JSON.stringify({
  contentId: "testingID For Client",
  duration: 30,
  title: "Test content for API functionality",
  price: 1,
  currency: "INR",
  url: "www.google.com",
  contentType: "STORY",
  priceOverrides: {
    country: [
      { name: "GL", price: 3 },
      { name: "IN", price: 5 },
      { name: "US", price: 7 },
    ],
  },
  download: {
    url: "https://yourdownloadurl.com",
    fileName: "Download File - Name",
    fileType: "PDF",
  },
});

var config = {
  method: "post",
  url: "{API_URL}/api/v1/content/",
  headers: {
    Authorization:
      "Basic RDZXN1Y4US1NTkc0V1lDLVFYOUJQMkItOEU3QjZLRzpUNFNHSjlISDQ3TVpRWkdTWkVGVjZYUk5TS1E4RDZXN1Y4UU1ORzRXWUNRWDlCUDJCOEU3QjZLRw==",
    "Content-Type": "application/json",
  },
  data: data,
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
{
  "message": "New Content Created!",
  "content": {
    "title": "Test content for API functionality",
    "price": 1,
    "currency": "INR",
    "contentId": "testingID For Client Content",
    "duration": 30,
    "url": "www.google.com",
    "contentType": "STORY",
    "priceOverrides": {
      "country": [
        {
          "_id": "605b25824646e9233aef61b4",
          "name": "GL",
          "price": 3
        },
        {
          "_id": "605b25824646e9233aef61b5",
          "name": "IN",
          "price": 5
        },
        {
          "_id": "605b25824646e9233aef61b6",
          "name": "US",
          "price": 7
        }
      ]
    },
    "download": {
      "url": "https://yourdownloadurl.com",
      "fileName": "Download File - Name",
      "fileType": "PDF",
      "s3Key": "KeyForDownloadedFileOnS3"
    }
  }
}
```

This endpoint allows the Client to Register their Content on ConsCent - with the Content Title, ContentId, Content URL, Price as well as any specific Price Overrides for a country - In order to set a different price for the content in the relevant country. Moreover, the Client can also set the duration of a content - which means that if a content if purchased by a user on ConsCent, then that user can have free access to the content for {duration} amount of time. By Default we use a 1 Day duration. Lastly, the price & priceOverrides of the content can only be set as a distinct value chosen by the client - which can be any out of [0, 0.01, 1, 2, 3, 5, 7, 10]. These prices are in INR and ONLY these values can be set as the price of the content - otherwise the API call for creating the content will throw a 400 (Bad Request) Error. Moreover, the ContentType field is optional - and if no 'contentType' is provided then the default 'contentType' of the client will be treated as the 'contentType' of the content being registered. While category based pricing can be used for any content, by passing the category field on registering the content - as long as the category has been registered by the client on the ConsCent dashboard along with its respective price, duration and priceOverrides; however, category based pricing only comes into effect if the content does not have a pre-determined price field (price must be null); Moreover, if the content is downloadable on purchase, then the client can pass the download "url", "fileName" and "fileType" in the download object of the request body while creating the content.

### HTTP Request

`POST {API_URL}/api/v1/content`

### Authorization

Client API Key and Secret must be passed in Authorization Headers using Basic Auth. With API Key as the Username and API Secret as the password.

### Request Body

| Parameter   | Default  | Description                                                                                                                                                                                                                                                                                  |
| ----------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| contentId   | required | Content Id by which the Content has been registered on the Client CMS                                                                                                                                                                                                                        |
| title       | required | Title of the Content                                                                                                                                                                                                                                                                         |
| price       | optional | Content Price for pay-per-use pricing                                                                                                                                                                                                                                                        |
| currency    | optional | Currency in which price is determined. Must be an ISO 4217 supported - 3 Digit currency Code. INR is used as the default if no value is provided.                                                                                                                                            |
| url         | required | URL where the content is available on your website                                                                                                                                                                                                                                           |
| duration    | optional | Free content access time for user once the user has purchased the content. (Standard Practice - 1 Day);                                                                                                                                                                                      |
| authorId    | optional | Id of the Author of the content - Mandatory if authorName is present                                                                                                                                                                                                                         |
| authorName  | optional | Name of the Author of the content                                                                                                                                                                                                                                                            |
| contentType | optional | Must be an ENUM from one of the following - STORY, VIDEO, SONG, PODCAST, PREMIUM CONTENT                                                                                                                                                                                                     |
| category    | optional | The category of the content, which has been registered by the client on the ConsCent Client Dashboard - in order to invoke category based pricing (only valid if story doesn't have a price). Each registered category will have an associated price, currency, duration and priceOverrides. |

| priceOverrides | optional | Price Overrides for any particular country with the relevant country code as the name and the ENUM value in the price. The country code list is located the end of this document |

| download | optional | Object containing the "url", "fileName" and "fileType". All download parameters must be provided if the content is downloadable on purchase. Also, the "fileType" is an ENUM and only accepts "PDF" currently. |

<aside class="notice">
Remember — A content must be registered by including all the required fields mentioned above! Ensure you provide all the required fields for creating the content - including the Content ID with which the content is registered on your Client CMS as well as the title and content URL. Moreover, depending on the pricing model you wish to use - you must either pass a price and currency associated with the content, or a pre-defined category to determine the pricing of the content. If neither of these are provided, the content price will be determined on the default price parameters set for the client. (Blanket Pricing). Lastly, you can pass the download object with the required fields for the content to be downloaded on purchase by a user. 
</aside>

## Edit Content

> Please ensure you URL encode the contentId in the Path Parameters
> Replace the {contentId} in the API path with your actual Content Id

```php
<?php

$curl = curl_init();

curl_setopt_array($curl, array(
  CURLOPT_URL => "{API_URL}/api/v1/content/Client%20Content%20Id%2011",
  CURLOPT_RETURNTRANSFER => true,
  CURLOPT_ENCODING => "",
  CURLOPT_MAXREDIRS => 10,
  CURLOPT_TIMEOUT => 0,
  CURLOPT_FOLLOWLOCATION => true,
  CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
  CURLOPT_CUSTOMREQUEST => "PATCH",
   CURLOPT_POSTFIELDS =>'{
    "contentId" : "testingID For Client Content",
    "duration" : 30,
    "title" : "Test content for API functionality Edited",
    "price" : 1,
    "currency": "INR",
    "url": "www.google.com",
    "contentType": "PREMIUM CONTENT",
    "priceOverrides": {
        "country": [
            {
                "name": "GL",
                "price": 2
            },
            {
                "name": "IN",
                "price": 1
            },
            {
                "name": "US",
                "price": 0
            }
        ]
    },
    "download": {
      "url": "https://yourdownloadurl.com",
      "fileName": "Download File - Edited Name",
      "fileType": "PDF"
    }
}',
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
curl -X PATCH '{API_URL}/api/v1/content/{contentId}' \
-H 'Authorization: Basic RDZXN1Y4US1NTkc0V1lDLVFYOUJQMkItOEU3QjZLRzpUNFNHSjlISDQ3TVpRWkdTWkVGVjZYUk5TS1E4RDZXN1Y4UU1ORzRXWUNRWDlCUDJCOEU3QjZLRw==' \
-H 'Content-Type: application/json' \
-d '{
    "contentId" : "testingID For Client Content",
    "duration" : 30,
    "title" : "Test content for API functionality Edited",
    "price" : 1,
    "currency": "INR",
    "url": "www.google.com",
    "contentType": "PREMIUM CONTENT",
    "priceOverrides": {
        "country": [
            {
                "name": "GL",
                "price": 2
            },
            {
                "name": "IN",
                "price": 1
            },
            {
                "name": "US",
                "price": 0
            }
        ]
    },
    "download": {
      "url": "https://yourdownloadurl.com",
      "fileName": "Download File - Edited Name",
      "fileType": "PDF"
    }
}'
```

```javascript
var axios = require("axios");
var data = JSON.stringify({
  contentId: "testingID For Client Content",
  duration: 30,
  title: "Test content for API functionality Edited",
  price: 1,
  currency: "INR",
  url: "www.google.com",
  contentType: "PREMIUM CONTENT",
  priceOverrides: {
    country: [
      { name: "GL", price: 2 },
      { name: "IN", price: 1 },
      { name: "US", price: 0 },
    ],
  },
  download: {
    url: "https://yourdownloadurl.com",
    fileName: "Download File - Edited Name",
    fileType: "PDF",
  },
});

var config = {
  method: "patch",
  url: "{API_URL}/api/v1/content/{contentId}",
  headers: {
    Authorization:
      "Basic RDZXN1Y4US1NTkc0V1lDLVFYOUJQMkItOEU3QjZLRzpUNFNHSjlISDQ3TVpRWjnJL877NJSjnkHSk5TS1E4RDZXN1Y4UU1ORzRXWUNRWDlCUDJCOEU3QjZLRw==",
    "Content-Type": "application/json",
  },
  data: data,
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
{
  "message": "Content Edited Successfully",
  "editedContent": {
    "title": "Client Content Id Test Edited",
    "contentId": "testingID For Client Content",
    "price": 1,
    "currency": "INR",
    "duration": 30,
    "url": "https://www.yoursite.com/yournewcontent",
    "contentType": "PREMIUM CONTENT",
    "authorName": "changed-author-name",
    "authorId": "changed-unique-author-id",
    "priceOverrides": {
      "country": [
        {
          "_id": "605b25824646e9233aef61b4",
          "name": "GL",
          "price": 2
        },
        {
          "_id": "605b25824646e9233aef61b5",
          "name": "IN",
          "price": 1
        },
        {
          "_id": "605b25824646e9233aef61b6",
          "name": "US",
          "price": 0
        }
      ]
    },
    "download": {
      "url": "https://yourdownloadurl.com",
      "fileName": "Download File - Edited Name",
      "fileType": "PDF",
      "s3Key": "KeyForDownloadedFileOnS3"
    }
  }
}
```

This endpoint allows the Client to Edit their Registered Content on ConsCent - with the editable fields being the Content title, price, currency, priceOverrides, URL, duration, contentType, download and category . Content ID CANNOT be edited!

### HTTP Request

`PATCH {API_URL}/api/v1/content/{contentId}`

### Authorization

Client API Key and Secret must be passed in Authorization Headers using Basic Auth. With API Key as the Username and API Secret as the password.

### URL Parameters

| Parameter | Description                                                        |
| --------- | ------------------------------------------------------------------ |
| contentId | The ID of the Content you wish to edit ( Ensure it is URL Encoded) |

### Request Body

| Parameter   | Default  | Description                                                                                                                                                                                                                                                                                  |
| ----------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| title       | optional | Title of the Content                                                                                                                                                                                                                                                                         |
| price       | optional | Content Price for Pay-per-use pricing                                                                                                                                                                                                                                                        |
| currency    | optional | Currency in which price is determined. Must be an ISO 4217 supported - 3 Digit currency Code. INR is used as the default if no value is provided.                                                                                                                                            |
| url         | optional | URL where the content is available on your website                                                                                                                                                                                                                                           |
| duration    | optional | Free content access time for user once the user has purchased the content. (Standard Practice - 1 Day);                                                                                                                                                                                      |
| authorId    | optional | Id of the Author of the content - Mandatory if authorName is present                                                                                                                                                                                                                         |
| authorName  | optional | Name of the Author of the content                                                                                                                                                                                                                                                            |
| contentType | optional | Must be an ENUM from one of the following - STORY, VIDEO, SONG, PODCAST, PREMIUM CONTENT                                                                                                                                                                                                     |
| category    | optional | The category of the content, which has been registered by the client on the ConsCent Client Dashboard - in order to invoke category based pricing (only valid if story doesn't have a price). Each registered category will have an associated price, currency, duration and priceOverrides. |

| priceOverrides | optional | Price Overrides for any particular country with the relevant country code as the name and the ENUM value in the price. The country code list is located the end of this document  
 |
| download | optional | Object containing the "url", "fileName" and "fileType". All download parameters must be provided if the content is downloadable on purchase. Also, the "fileType" is an ENUM and only accepts "PDF" currently.

<aside class="notice">
Remember — Either/All of the fields of a content - title, price, currency, priceOverrides, URL, authorName, authorId, contentType, category and download - can be edited. Only pass the fields you wish to edit in the request body. 
</aside>

## View All Content

> API to retrieve all of the client's content registered on ConsCent.

```php
<?php

$curl = curl_init();

curl_setopt_array($curl, array(
  CURLOPT_URL => "{API_URL}/api/v1/content/client",
  CURLOPT_RETURNTRANSFER => true,
  CURLOPT_ENCODING => "",
  CURLOPT_MAXREDIRS => 10,
  CURLOPT_TIMEOUT => 0,
  CURLOPT_FOLLOWLOCATION => true,
  CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
  CURLOPT_CUSTOMREQUEST => "GET",
  CURLOPT_HTTPHEADER => array(
    "Authorization: Basic RDZXN1Y4US1NTkc0V1lDLVFYOUJQMkItOEU3QjZLRzpUNFNHSjlISDQ3TVpRWkdTWkVGVjZYUk5TS1E4RDZXN1Y4UU1ORzRXWUNRWDlCUDJCOEU3QjZLRw=="
  ),
));

$response = curl_exec($curl);

curl_close($curl);
echo $response;


```

```shell
curl -X GET '{API_URL}/api/v1/content/client' \
-H 'Authorization: Basic RDZXN1Y4US1NTkc0V1lDLVFYOUJQMkItOEU3QjZLRzpUNFNHSjlISDQ3TVpRWkdTWkVGVjZYUk5TS1E4RDZXN1Y4UU1ORzRXWUNRWDlCUDJCOEU3QjZLRw=='
```

```javascript
var axios = require("axios");

var config = {
  method: "get",
  url: "{API_URL}/api/v1/content/client",
  headers: {
    Authorization:
      "Basic RDZXN1Y4US1NTkc0V1lDLVFYOUJQMkItOEU3QjZLRzpUNFNHSjlISDQ3TVpRWkdTWkVGVjZYUk5TS1E4RDZXN1Y4UU1ORzRXWUNRWDlCUDJCOEU3QjZLRw==",
  },
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
{
  "content": [
    {
      "title": "Client Content Id Test Edit 1",
      "contentId": "Client Content Id 11",
      "price": 1,
      "currency": "INR",
      "duration": 2,
      "url": "https://www.yoursite.com/yourcontent",
      "contentType": "STORY",
      "priceOverrides": {
        "country": [
          {
            "_id": "605b25824646e9233aef61b4",
            "name": "GL",
            "price": 2
          }
        ]
      }
    },
    {
      "title": "Test content for API functionality",
      "contentId": "testingID for Client Content",
      "price": 3,
      "currency": "INR",
      "duration": 2,
      "url": "https://www.yoursite.com/yourcontent",
      "contentType": "STORY",
      "download": {
        "url": "https://yourdownloadurl.com",
        "fileName": "Download File - Name",
        "fileType": "PDF",
        "s3Key": "KeyForDownloadedFileOnS3"
      }
    }
  ],
  "pagination": {
    "pageNumber": 1,
    "pageSize": 20,
    "totalRecords": 2,
    "totalPages": 1
  }
}
```

This endpoint allows the Client to view their entire collection of Registered Content on ConsCent. With each retrieved content containing the following details - Title, Price, URL, Content ID and duration.

### HTTP Request

`GET {API_URL}/api/v1/content/client`

### Authorization

Client API Key and Secret must be passed in Authorization Headers using Basic Auth. With API Key as the Username and API Secret as the password.

### URL Parameters

| Parameter  | Default  | Description                                                                                                                                                        |
| ---------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| pageNumber | optional | Pagination Parameters - which page of the contents list you would like to retrieve (default = 1). Since each page will have 20 (default) individual contents ONLY. |
| pageSize   | optional | Pagination Parameters - the number of individual contents to retrieve on each page (default = 20).                                                                 |

<aside class="notice">
Remember — Pagination parameters are optional. The dafault values are - pageNumber = 1 & pageSize = 20. If you would like to access more content in a single call then you will have to pass the value as a query parameter (pageSize). Max. value for pageSize is 499. Moreover, if you would like to access content that isn't included in the first page - then you will have to pass the pageNumber as a query parameter for any of the subsequent pages.
</aside>

## View Content Details

> Please ensure you URL encode the contentId in the Path Parameters
> Replace the {contentId} in the API path with your actual Content Id

```php
<?php

$curl = curl_init();

curl_setopt_array($curl, array(
  CURLOPT_URL => "{API_URL}/api/v1/content/client/Client%20Content%20Id%206",
  CURLOPT_RETURNTRANSFER => true,
  CURLOPT_ENCODING => "",
  CURLOPT_MAXREDIRS => 10,
  CURLOPT_TIMEOUT => 0,
  CURLOPT_FOLLOWLOCATION => true,
  CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
  CURLOPT_CUSTOMREQUEST => "GET",
  CURLOPT_HTTPHEADER => array(
    "Authorization: Basic RDZXN1Y4US1NTkc0V1lDLVFYOUJQMkItOEU3QjZLRzpUNFNHSjlISDQ3TVpRWkdTWkVGVjZYUk5TS1E4RDZXN1Y4UU1ORzRXWUNRWDlCUDJCOEU3QjZLRw=="
  ),
));

$response = curl_exec($curl);

curl_close($curl);
echo $response;

```

```shell
curl -X GET '{API_URL}/api/v1/content/client/Client%20Content%20Id%206' \
-H 'Authorization: Basic RDZXN1Y4US1NTkc0V1lDLVFYOUJQMkItOEU3QjZLRzpUNFNHSjlISDQ3TVpRWkdTWkVGVjZYUk5TS1E4RDZXN1Y4UU1ORzRXWUNRWDlCUDJCOEU3QjZLRw=='
```

```javascript
var axios = require("axios");

var config = {
  method: "get",
  url: "{API_URL}/api/v1/content/client/Client%20Content%20Id%206",
  headers: {
    Authorization:
      "Basic RDZXN1Y4US1NTkc0V1lDLVFYOUJQMkItOEU3QjZLRzpUNFNHSjlISDQ3TVpRWkdTWkVGVjZYUk5TS1E4RDZXN1Y4UU1ORzRXWUNRWDlCUDJCOEU3QjZLRw==",
  },
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
{
  "title": "Tesla Tequila",
  "contentId": "Client Content Id 6",
  "price": 1,
  "currency": "INR",
  "url": "https://www.yoursite.com/yourcontent",
  "duration": 1,
  "contentType": "STORY",
  "priceOverrides": {
    "country": [
      {
        "_id": "605b25824646e9233aef61b4",
        "name": "GL",
        "price": 2
      }
    ]
  }
}
```

This endpoint allows the Client to retrieve a particular content which they registered with ConsCent - including the details of the content such as the Content ID, title, price, currency, URL, duration, contentType and category.

### HTTP Request

`GET {API_URL}/api/v1/content/client/{contentId}`

### Authorization

Client API Key and Secret must be passed in Authorization Headers using Basic Auth. With API Key as the Username and API Secret as the password.

### URL Parameters

| Parameter | Description                                                                |
| --------- | -------------------------------------------------------------------------- |
| contentId | The ID of the Content you wish to retrieve (Please ensure its URL Encoded) |

<aside class="notice">
API to retrieve the details of an individual content registered by the client on ConsCent. 
</aside>

# Validate Content Consumption

## Validate Content Details By Consumption ID

> Replace the {consumptionId} in the API path with the actual value/string of the consumptionId recieved

```php
<?php

$curl = curl_init();

curl_setopt_array($curl, array(
  CURLOPT_URL => "{API_URL}/api/v1/content/consumption/11c369df-2a30-4a0d-90dc-5a45797dacdd",
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
curl -X POST '{API_URL}/api/v1/content/consumption/{consumptionId}'
```

```javascript
var axios = require("axios");

var config = {
  method: "post",
  url: "{API_URL}/api/v1/content/consumption/{consumptionId}",
  headers: {},
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
    "message": "Content Consumption Confirmed",
    "payload": {
      "clientId": "5fbb40b07dd98b0e89d90a25",
      "contentId": "Client Content Id",
      "createdAt": "2020-11-15T13:55:36.659Z"
    },
    "consumptionId": "11c369df-2a30-4a0d-90dc-5a45797dacdd",
    "signature": "4379hrm47mo34m2340cny2rcn487cn209842cr474107nc409c4"
  }
]
```

This endpoint allows the Client to Validate the Content Details anytime a User Purchases any Content of the Client using ConsCent - with the Client passing the Content ID in the request and recieving details regarding the Content Id and the Date of Purchase of the Content by the User, as well as matching the unique Consumption Id to ensure it cannot be reused.

### HTTP Request

`POST {API_URL}/api/v1/content/consumption/{consumptionId}`

The client will recieve a payload when any content is purchased via ConsCent - providing the details of the content such as the ClientId to identify which client the content belongs to as well as the Client Content ID, Transaction Amount and Date of the Transaction. Moroever, they will also recieve a Consumption ID - by passing the Consumption ID in this API request - the Client can verify the authenticity of the transaction and restrict any spillage.

### URL Parameters

| Parameter     | Default  | Description                                                                                                              |
| ------------- | -------- | ------------------------------------------------------------------------------------------------------------------------ |
| consumptionId | required | consumptionId recieved by the client for each unique transaction on any of the Client's Content registered with ConsCent |

<aside class="notice">
Remember — The Consumption ID is unique and once it is used it will expire. Clients can use this endpoint to ensure the unique transactions that occur on any of their premium content registered with ConsCent. 
</aside>

# Client Stats

## Daily Content Stats

```php

<?php

$curl = curl_init();

curl_setopt_array($curl, array(
  CURLOPT_URL => '{API_URL}/api/v1/client/stats/daily?from=2021-02-12T05:45:41.389Z&to=2021-02-13T05:45:41.389Z',
  CURLOPT_RETURNTRANSFER => true,
  CURLOPT_ENCODING => '',
  CURLOPT_MAXREDIRS => 10,
  CURLOPT_TIMEOUT => 0,
  CURLOPT_FOLLOWLOCATION => true,
  CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
  CURLOPT_CUSTOMREQUEST => 'GET',
  CURLOPT_HTTPHEADER => array(
    'Authorization: Basic WTE3UkdRSy1RMlQ0UEo5LU4zWVNSWEotRFNSNERZTTpQUkVTVDdTRTZSTUNXWVBaNjRZQzdXUlA2UEpEWTE3UkdRS1EyVDRQSjlOM1lTUlhKRFNSNERZTQ=='
  ),
));

$response = curl_exec($curl);

curl_close($curl);
echo $response;


```

```shell

curl -X GET '{API_URL}/api/v1/client/stats/daily?from=2021-02-12T05:45:41.389Z&to=2021-02-13T05:45:41.389Z' \
-H 'Authorization: Basic WTE3UkdRSy1RMlQ0UEo5LU4zWVNSWEotRFNSNERZTTpQUkVTVDdTRTZSTUNXWVBaNjRZQzdXUlA2UEpEWTE3UkdRS1EyVDRQSjlOM1lTUlhKRFNSNERZTQ=='

```

```javascript
var axios = require("axios");

var config = {
  method: "get",
  url: "{API_URL}/api/v1/client/stats/daily?from=2021-02-12T05:45:41.389Z&to=2021-02-13T05:45:41.389Z",
  headers: {
    Authorization:
      "Basic WTE3UkdRSy1RMlQ0UEo5LU4zWVNSWEotRFNSNERZTTpQUkVTVDdTRTZSTUNXWVBaNjRZQzdXUlA2UEpEWTE3UkdRS1EyVDRQSjlOM1lTUlhKRFNSNERZTQ==",
  },
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
{
  "totalContentConsumed": 17,
  "totalRevenueGenerated": 29.6
}
```

This endpoint allows the Client to get Aggregate Statistics based on users consumption of their content via ConsCent. By default this API provides details of the previous days' consumption. However, the client can pass the 'from' and 'to' dates as optional query parameters to get aggregated stats for any range that they choose. The value of 'totalRevenueGenerated' in the response if given in "INR" by default.

### HTTP Request

`POST {API_URL}/api/v1/client/stats/daily`

### Authorization

Client API Key and Secret must be passed in Authorization Headers using Basic Auth. With API Key as the Username and API Secret as the password.

### Query Parameters

| Parameter | Default  | Description                                                           |
| --------- | -------- | --------------------------------------------------------------------- |
| from      | optional | ISO date string from which the aggregated stats have to be calculated |
| to        | optional | ISO date string till which the aggregated stats have to be calculated |

<aside class="notice">
Providing aggregated statistics to the client - for the previous days' (default) or provided date ranges' consumption of their content by users via ConsCent.
</aside>

# Deprecated Integration Documentation

To view the deprecated integration documentation containing the Story APIs - [Click here](https://tsbmediaventure.github.io/deprecated-docs).

Please not that we no longer provide support for this documentation and it will be removed soon. We urge you to make use of the updated integration documentation. You can email us at support@conscent.in for any queries that you might have.

# Country Code List

Afghanistan - AF

Åland Islands - AX

Albania - AL

Algeria - DZ

American Samoa - AS

Andorra - AD

Angola - AO

Anguilla - AI

Antarctica - AQ

Antigua and Barbuda - AG

Argentina - AR

Armenia - AM

Aruba - AW

Australia - AU

Austria - AT

Azerbaijan - AZ

Bahrain - BH

Bahamas - BS

Bangladesh - BD

Barbados - BB

Belarus - BY

Belgium - BE

Belize - BZ

Benin - BJ

Bermuda - BM

Bhutan - BT

Bolivia, Plurinational State of - BO

Bonaire, Sint Eustatius and Saba - BQ

Bosnia and Herzegovina - BA

Botswana - BW

Bouvet Island - BV

Brazil - BR

British Indian Ocean Territory - IO

Brunei Darussalam - BN

Bulgaria - BG

Burkina Faso - BF

Burundi - BI

Cambodia - KH

Cameroon - CM

Canada - CA

Cape Verde - CV

Cayman Islands - KY

Central African Republic - CF

Chad - TD

Chile - CL

China - CN

Christmas Island - CX

Cocos (Keeling) Islands - CC

Colombia - CO

Comoros - KM

Congo - CG

Congo, the Democratic Republic of the - CD

Cook Islands - CK

Costa Rica - CR

Côte d'Ivoire - CI

Croatia - HR

Cuba - CU

Curaçao - CW

Cyprus - CY

Czech Republic - CZ

Denmark - DK

Djibouti - DJ

Dominica - DM

Dominican Republic - DO

Ecuador - EC

Egypt - EG

El Salvador - SV

Equatorial Guinea - GQ

Eritrea - ER

Estonia - EE

Ethiopia - ET

Falkland Islands (Malvinas) - FK

Faroe Islands - FO

Fiji - FJ

Finland - FI

France - FR

French Guiana - GF

French Polynesia - PF

French Southern Territories - TF

Gabon - GA

Gambia - GM

Georgia - GE

Germany - DE

Ghana - GH

Gibraltar - GI

Greece - GR

Greenland - GL

Grenada - GD

Guadeloupe - GP

Guam - GU

Guatemala - GT

Guernsey - GG

Guinea - GN

Guinea-Bissau - GW

Guyana - GY

Haiti - HT

Heard Island and McDonald Islands - HM

Holy See (Vatican City State) - VA

Honduras - HN

Hong Kong - HK

Hungary - HU

Iceland - IS

India - IN

Indonesia - ID

Iran, Islamic Republic of - IR

Iraq - IQ

Ireland - IE

Isle of Man - IM

Israel - IL

Italy - IT

Jamaica - JM

Japan - JP

Jersey - JE

Jordan - JO

Kazakhstan - KZ

Kenya - KE

Kiribati - KI

Korea, Democratic People's Republic of - KP

Korea, Republic of - KR

Kuwait - KW

Kyrgyzstan - KG

Lao People's Democratic Republic - LA

Latvia - LV

Lebanon - LB

Lesotho - LS

Liberia - LR

Libya - LY

Liechtenstein - LI

Lithuania - LT

Luxembourg - LU

Macao - MO

Macedonia, the Former Yugoslav Republic of - MK

Madagascar - MG

Malawi - MW

Malaysia - MY

Maldives - MV

Mali - ML

Malta - MT

Marshall Islands - MH

Martinique - MQ

Mauritania - MR

Mauritius - MU

Mayotte - YT

Mexico - MX

Micronesia, Federated States of - FM

Moldova, Republic of - MD

Monaco - MC

Mongolia - MN

Montenegro - ME

Montserrat - MS

Morocco - MA

Mozambique - MZ

Myanmar - MM

Namibia - NA

Nauru - NR

Nepal - NP

Netherlands - NL

New Caledonia - NC

New Zealand - NZ

Nicaragua - NI

Niger - NE

Nigeria - NG

Niue - NU

Norfolk Island - NF

Northern Mariana Islands - MP

Norway - NO

Oman - OM

Pakistan - PK

Palau - PW

Palestine, State of - PS

Panama - PA

Papua New Guinea - PG

Paraguay - PY

Peru - PE

Philippines - PH

Pitcairn - PN

Poland - PL

Portugal - PT

Puerto Rico - PR

Qatar - QA

Réunion - RE

Romania - RO

Russian Federation - RU

Rwanda - RW

Saint Barthélemy - BL

Saint Helena, Ascension and Tristan da Cunha - SH

Saint Kitts and Nevis - KN

Saint Lucia - LC

Saint Martin (French part) - MF

Saint Pierre and Miquelon - PM

Saint Vincent and the Grenadines - VC

Samoa - WS

San Marino - SM

Sao Tome and Principe - ST

Saudi Arabia - SA

Senegal - SN

Serbia - RS

Seychelles - SC

Sierra Leone - SL

Singapore - SG

Sint Maarten (Dutch part) - SX

Slovakia - SK

Slovenia - SI

Solomon Islands - SB

Somalia - SO

South Africa - ZA

South Georgia and the South Sandwich Islands - GS

South Sudan - SS

Spain - ES

Sri Lanka - LK

Sudan - SD

Suriname - SR

Svalbard and Jan Mayen - SJ

Swaziland - SZ

Sweden - SE

Switzerland - CH

Syrian Arab Republic - SY

Taiwan, Province of China - TW

Tajikistan - TJ

Tanzania, United Republic of - TZ

Thailand - TH

Timor-Leste - TL

Togo - TG

Tokelau - TK

Tonga - TO

Trinidad and Tobago - TT

Tunisia - TN

Turkey - TR

Turkmenistan - TM

Turks and Caicos Islands - TC

Tuvalu - TV

Uganda - UG

Ukraine - UA

United Arab Emirates - AE

United Kingdom - GB

United States - US

United States Minor Outlying Islands - UM

Uruguay - UY

Uzbekistan - UZ

Vanuatu - VU

Venezuela, Bolivarian Republic of - VE

Viet Nam - VN

Virgin Islands, British - VG

Virgin Islands, U.S. - VI

Wallis and Futuna - WF

Western Sahara - EH

Yemen - YE

Zambia - ZM

Zimbabwe - ZW
