# apps-api

## Description

External applications (installed on the same server 
or from the Internet) can communicate with AwesomeTeamPlayer using this microservice. 
Communication is  bidirectional: apps-api provide **POST /execute** endpoint to 
execute available methods and application has to provide an endpoint (webhook) 
for receiving informations from AwesomeTeamPlayer instance.
 
## Registering application process 

1. Application sends POST request to /register endpoint:
```json
{
  "name":"Application name",
  "description":"Application description",
  "iconUrl": "http://address.to/icon/image.png",
  "scopes": 
      [ 
        //  ... list of scopes ...
      ],
  "webhookUrl": "http://address.to/post/webhook"
}
```

**name** - Application's name. Not empty string, maximum length is 255 characters.

**description** - Application's description. This is string, it can be empty, 
has no length limit. The application's description can be formatted with Markdown rules.

**iconUrl** - URL to icon in .png format. The image has to have size 50x50 pixels.
 
**scopes** - List of strings. More details below.

**webhookUrl** - URL available for POST requests. Before application is registered,
 ATP sends below request and waits for below response to test webhook.
  
Request from ATP to webhook:
```json
{
  "name":"ping-pong-test",
  "confirmationHash": "abc123",
}

```
Response from application to ATP:
```json
{
  "name":"ping-pong-test-reply",
  "confirmationHash": "abc123",
}
```

2. AwesomeTeamPlayer validates incoming JSON. If everything is OK it returns:

```json
{
  "status": "success",
  "appId": "abc123",
  "secretHash": "xxx"
}
```

where:

**appId** - is unique application ID. This is a string. It contains [10-30] chars.

**secretHash** - is string for random hash. It contains 50 chars. 
  
3. If given data is incorrect apps-service returns validation errors:
 
```json
{
  "status":"failed",
  "name": [
    // list fo name errors
  ],
  "iconUrl": [
    // list of iconUrl errors
  ],
  "scopes": [
    // list of scopes errors
  ],
  "webhookUrl": [
    // list of webhook errors
  ]
}
```

## Available scopes

* users.read - returning all users' data
* users.create - creating user (regular user)
* users.update - updating any user data
* users.activation - changing any regular user to admin user and admin user to regular user
* loggedUser.read - returning logged user's data
* loggedUser.update - updating logged user's data
* ...

## Making requests to AwesomeTeamPlayer

To make a request to AwesomeTeamPlayer, the application should send a json to 
**POST /execute** endpoint:
```json
{
  "appId": "abc123",
  "secretHash": "xxx",
  "methodName": "<method_name>",
  "methodData": {
    // JSON method data
  }
}
```

Where **methodName** is one of the available method listed below.

The result for correct request looks like this:
 
```json
{
  "status":"success",
  "response": {
    // JSON data
  }
}
```

The result for incorrect request looks like this:
 
```json
{
  "status":"failed",
  "methodName": [
    //  errors list
  ],
  "methodData": [
    //  errors list
  ]
}
```

## Available methods

* GetUsers (parameters: limit: offset)
* CountUsers
* ...

## Webhooks
