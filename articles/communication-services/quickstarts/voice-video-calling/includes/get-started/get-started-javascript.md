---
title: Quickstart - Add VOIP calling to a web app using Azure Communication Services
description: In this tutorial, you learn how to use the Azure Communication Services Calling SDK for JavaScript
author: ddematheu
ms.author: nimag
ms.date: 03/10/2021
ms.topic: include
ms.service: azure-communication-services
---

In this quickstart, you'll learn how start a call using the Azure Communication Services Calling SDK for JavaScript.

## Sample code

You can download the sample app from [GitHub](https://github.com/Azure-Samples/communication-services-javascript-quickstarts/tree/main/add-1-on-1-voice-calling).

## Prerequisites

- An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Node.js](https://nodejs.org/) Active LTS and Maintenance LTS versions (8.11.1 and 10.14.1 recommended).
- An active Communication Services resource. [Create a Communication Services resource](../../../create-communication-resource.md).
- A User Access Token to instantiate the call client. Learn how to [create and manage user access tokens](../../../access-tokens.md).

[!INCLUDE [Calling with JavaScript](./get-started-javascript-setup.md)]

Here's the code:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Communication Client - Calling Sample</title>
  </head>
  <body>
    <h4>Azure Communication Services</h4>
    <h1>Calling Quickstart</h1>
    <input 
      id="token-input"
      type="text"
      placeholder="User access token"
      style="margin-bottom:1em; width: 200px;"
    />
    </div>
    <button id="token-submit" type="button">
        Submit
    </button>
    <input 
      id="callee-id-input"
      type="text"
      placeholder="Who would you like to call?"
      style="margin-bottom:1em; width: 200px; display: block;"
    />
    <div>
      <button id="call-button" type="button" disabled="true">
        Start Call
      </button>
      &nbsp;
      <button id="hang-up-button" type="button" disabled="true">
        Hang Up
      </button>
    </div>
    <script src="./bundle.js"></script>
  </body>
</html>
```

Create a file in the root directory of your project called **client.js** to contain the application logic for this quickstart. Add the following code to import the calling client and get references to the DOM elements so we can attach our business logic. 

```javascript
import { CallClient, CallAgent } from "@azure/communication-calling";
import { AzureCommunicationTokenCredential } from '@azure/communication-common';

let call;
let callAgent;
let tokenCredential = "";
const userToken = document.getElementById("token-input");
const calleeInput = document.getElementById("callee-id-input");
const submitToken = document.getElementById("token-submit");
const callButton = document.getElementById("call-button");
const hangUpButton = document.getElementById("hang-up-button");
```

## Object model

The following classes and interfaces handle some of the major features of the Azure Communication Services Calling SDK:

| Name                             | Description                                                                                                                                 |
| ---------------------------------| ------------------------------------------------------------------------------------------------------------------------------------------- |
| CallClient                       | The CallClient is the main entry point to the Calling SDK.                                                                       |
| CallAgent                        | The CallAgent is used to start and manage calls.                                                                                            |
| AzureCommunicationTokenCredential | The AzureCommunicationTokenCredential class implements the CommunicationTokenCredential interface which is used to instantiate the CallAgent. |

## Authenticate the client

You need to input a valid user access token for your resource into the text field and click 'Submit'. Refer to the [user access token](../../../access-tokens.md) documentation if you don't already have a token available. Using the `CallClient`, initialize a `CallAgent` instance with a `CommunicationTokenCredential` which will enable us to make and receive calls. 

Add the following code to **client.js**:

```javascript
submitToken.addEventListener("click", async () => {
  const callClient = new CallClient(); 
  const userTokenCredential = userToken.value;
    try {
      tokenCredential = new AzureCommunicationTokenCredential(userTokenCredential);
      callAgent = await callClient.createCallAgent(tokenCredential);
      callButton.disabled = false;
      submitToken.disabled = true;
    } catch(error) {
      window.alert("Please submit a valid token!");
    }
})
```

## Start a call

Add an event handler to initiate a call when the `callButton` is clicked:

```javascript
callButton.addEventListener("click", () => {
    // start a call
    const userToCall = calleeInput.value;
    // To call an Azure Communication Services communication user, use {communicationUserId: 'ACS_USER_ID'}.
    // To call echobot, use {id: '8:echo123'}.
    call = callAgent.startCall(
        [{ communicationUserId: userToCall }],
        {}
    );
    // toggle button states
    hangUpButton.disabled = false;
    callButton.disabled = true;
});
```

## End a call

Add an event listener to end the current call when the `hangUpButton` is clicked:

```javascript
hangUpButton.addEventListener("click", () => {
  // end the current call
  call.hangUp({ forEveryone: true });

  // toggle button states
  hangUpButton.disabled = true;
  callButton.disabled = false;
  submitToken.disabled = false;
});
```

The `forEveryone` property ends the call for all call participants.

## Run the code

Use the `webpack-dev-server` to build and run your app. Run the following command to bundle application host in on a local webserver:

```console
npx webpack-dev-server --entry ./client.js --output bundle.js --debug --devtool inline-source-map
```

Open your browser and navigate to http://localhost:8080/. You should see the following:

:::image type="content" source="../../media/javascript/calling-javascript-app-2.png" alt-text="Screenshot of the completed JavaScript Application.":::

You can make an outbound VOIP call by providing a valid user access token and user ID in the corresponding text fields and clicking the **Start Call** button.

Calling `8:echo123` connects you with an echo bot, this is great for getting started and verifying your audio devices are working. Pass `{id: '8:echo123'}` to the CallAgent.startCall() API to call echobot.
To call an Azure Communication Services communication user, pass `{communicationUserId: 'ACS_USER_ID'}` to the `CallAgent.startCall()` API.
