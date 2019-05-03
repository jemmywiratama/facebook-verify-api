# HyperVerge Facebook Verification API -Documentation 

## Overview

This documentation describes facebook verification API v1. If you have any queries please contact support. The postman collection can be found here : [LINK](https://www.getpostman.com/collections/4485beb30c74882c99b3)

1. Schema
1. Parameters
1. Root Endpoint
1. Authentication
1. Media Types
3. API Endpoint
4. Hook URL


## Schema

Currently, HTTP and HTTPS are supported. All data is received as JSON, and all image uploads are to be performed as form-data (POST request). 
It is recommended that HTTPS is used for the API calls. For HTTPS, please ensure TLS v1.2 is used. In the future versions support for previous version will be deprecated.

## Parameters
All optional and compulsory parameters are passed as part of the request body.

## Root Endpoint
A `GET` request can be issued to the root endpoint to check for successful connection. Actual endpoint URL will be published later.

	 curl https://docs.hyperverge.co/v1

The `plain/text` reponse of `"AoK!"` should be received.

## Authentication

Currently, a simple appId, appKey combination is passed in the request header. The appId and appKey are provided on request by the HyperVerge team. If you would like to try the API, please reach out to contact@hyperverge.co

	curl -X POST \
	  https://docs.hyperverge.co/v1/verifyFB \
	  -H 'appid: appId-Value' \
	  -H 'appkey: appKey-Value' \
	  -H 'content-type: multipart/form-data' \
	  -F fb_access_token=String \
	  -F selfie=@Imagepath.jpg \
	  -F completion_hook=https://www.google.com

On failed attempt with invalid credentials or unauthorized access the following error message should be received :

	{
	  "status": "failure",
	  "statusCode": "401"
	}

**Please do not expose the appid and appkey on browser applications. In case of a browser application, set up the API calls from the server side.**

## Media Types

Currently, `jpeg, png and tiff` images are supported by the HyperDocs image extraction APIs. 

## Supported API

* **URL**

    - /verifyFB :
      
* **Method:**

    `POST`

* **Header**
	
	- content-type : 'formdata'
	- appid 
	- appkey
	
* **Request Body**

	- selfie
	- fb\_access\_token
	- completion\_hook 
  
* **Success Response:**

    * **Code:** 200 <br />
    * Incase of a properly made request, the response would follow schema.

		
		```
		{
			"status" : "success",
			"statusCode" : "200"
		}
		```
		
	
* **Error Response:**

 Invalid Request Error :

         
	        {
	          "status": "failure",
	          "statusCode": "400",
	          "error": "E<ERROR-CODE> <ERROR-MESSAGE>"
	        }
	                 
The different error codes   
		
|Error Code| Description |
|---|---|		
|1 | Invalid multipart-form
|11 | Missing selfie
|12 | Missing fb\_access_token
|51 | Invalid fb\_access_token
|53 | Critical FB Permissions Not Granted
|202 | No face detected
|203 | Multiple faces detected

All error messages follow the same syntax with the statusCode and status also being a part of the response body, and `string` error message with the description of the error.
	
**Server Errors**
We try our best to avoid these errors, but if by chance they do occur the response code will be 5xx.


* **Sample Calls:**

			curl -X POST \
			  https://docs.hyperverge.co/v1/verifyFB \
			  -H 'appid: appId-Value' \
			  -H 'appkey: appKey-Value' \
			  -H 'content-type: multipart/form-data' \
			  -F fb_access_token=String \
			  -F selfie=@Imagepath.jpg \
			  -F completion_hook=https://www.google.com

### Hook or Callback URL

Since there could be many photos that would have to be processed for a user, this could take upto 1 minute. Hence, the callback URL is used to asynchronously post the response after the task is completed. The result JSON posted to completion hook has the following format.


```
{
    "matchedImage": {
        "timeValidationPassed": <Boolean - digitally qualified profile>,
        "createdTime": <Number - Epoch time of the oldest matched image, exists only when time validation is passed>,
        "id": <String - ID of the oldest matched image, exists only when time validation is passed>,
        "url": <String - URL of the above image>
        },
        
    "info": {
        "firstImageActivity": <Number - Epoch time of the oldest image uploaded/tagged image of the user>,
        "id": <String, Facebook User ID>,
        "name": <String>,
	"last_name":<String>,
	"short_name":<String>,
	"is_verified":<0 or 1 - This field indicates whether the person's profile is verified manually by facebook>,
	"verified":<0 or 1 - Indicates whether the account has been verified by the user via SMS etc>,
        "mostCoTaggedUsers": <Array of user objects who have been tagged in photos with this user the most>,
	"tagged_places": <Dictionary with an array of few tagged places and corresponding paging to get rest of the tagged places>,
	"age_range": <Dictionary with minimum and maximum age of the user>,
	"birthday": <String>,
	"email":<String>,
	"friends:<Dictionary with an array of few friends(names and ids), corresponding paging to get rest of the friends and total friend count>,
	"gender":<String>,
	"hometown": <Dictionary with place id and name>,
	"location": <Dictionary with place id and name>
        }
    }
}
```

*Incase the selfie not match with any of the photos the user has provided access to. The "matchedImage" will have only one key, `timeValidationPassed` and this will be set to false.*
	
Depending on the permissions given by the user and information available in the profile, additional fields might be present in the 'info' object of the result. Eg: Devices, significant_other etc.

**Retry policy**

The application expects the hook url to respond with a 200 status code. In case the status code is something other than 200, the application will retry hitting the hook url upto 5 times with a delay of 1 min between each retries. Post the 5 retries, the application will assume that hook url was invalid and will drop the request.
