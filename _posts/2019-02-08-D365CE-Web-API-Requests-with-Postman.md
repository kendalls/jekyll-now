---
layout: post
title: "D365 CE Web API Requests with Postman"
description: "How to easily make requests to the D365 CE Web API with Postman."
thumb_image: ""
tags: [dynamics]
published: true
---

## Step 1: Create an Application User
This allows us to connect using a client id and secret. 

You can use the default client id for CRM Online (2ad88395-b77d-4561-9441-d0e40824f9bc) and a username and password though (with a grant_type of password).

Follow the steps in the [Application Authentication to Dynamics 365]({{ site.url }}/crm-server-to-server-authentication/) post to register an application and user.

## Step 2: Create Postman Environment
Click the gear-shaped icon in the top right of the Postman window to open the **Manage Environments** dialog.

Create a new environment with the following variables:
  * identity_host (with value *https://login.microsoftonline.com*)
  * BearerToken
  * BearerTokenExpiry
  * TokenEnvironmentName
  * EnvironmentName (with value corresponding to the name of your environment i.e. CRM organization name)
  
Note, values are blank unless they are listed above.

Create a folder for your requests as well.

## Step 3: Edit Folder Settings
Click the ellipsis at the righthand end of the folder name and then **Edit** to open the **Edit Collection** dialog.

Select the **Authorization** tab. Select **Bearer Token** for the **Type** and set the **Token** value to {% raw %}{{BearerToken}}{% endraw %}.

Then select the **Pre-request Scripts** tab and enter the following script, replacing the values for **client_id**, **client_secret**, and **resource** with your own values (from step one).

The authentication request url can also be replaced with the OAuth 2.0 token endpoint (v1) value from your application registration.

{% highlight javascript %}
const authenticationRequest = {
    url: pm.environment.get('identity_host') + '/common/oauth2/token', method: 'POST',
    header: 'Content-Type:application/x-www-form-urlencoded',
    body: {
        mode: 'urlencoded',
        urlencoded: [
            {key: "grant_type", value:"client_credentials", disabled: false},
            {key: "client_id", value:"**************************", disabled: false},
            {key: "client_secret", value:"********************************", disabled: false},
            {key: "resource", value:"https://***********.api.crm6.dynamics.com", disabled:false}
        ]
    }
};

if(!pm.environment.get('BearerToken') ||
    !pm.environment.get('BearerTokenExpiry') ||
    !pm.environment.get('TokenEnvironmentName') ||
    !pm.environment.get('EnvironmentName') ||
    pm.environment.get('TokenEnvironmentName') != pm.environment.get('EnvironmentName') ||
    pm.environment.get('BearerTokenExpiry') <= (new Date()).getTime()) {
    console.log('Retrieving new token...');
    pm.sendRequest(authenticationRequest, function (err, res) {
        if(err === null) {
            var responseJson = res.json();
            pm.environment.set('BearerToken', responseJson.access_token);
            pm.environment.set('TokenEnvironmentName', pm.environment.get('EnvironmentName'));
            
            var expiryDate = new Date();
            expiryDate.setSeconds(expiryDate.getSeconds() + responseJson.expires_in);
            pm.environment.set('BearerTokenExpiry', expiryDate.getTime());
        }
    });
}
{% endhighlight %}

Click **Update** to close the dialog.

Note that you can use the refresh_token value from the original authentication request to obtain a new access token when it expires, but we just authenticate again to get a new token since it's less work to setup.

## Step 4: Create Request
Create a new GET request to the D365 CE Web API endpoint.

On the **Authorization** tab, set **Type** to **Inherit auth from parent**.

Set the following headers:
 * Accept: application/json
 * Cache-Control: no-cache
 * Content-Type: application/json; charset=utf-8
 * OData-MaxVersion: 4.0
 * OData-Version: 4.0
 
 If you don't have the **Postman Console** window open, click **View** > **Show Postman Console**.
 
 When you send the request you'll first see the POST request for the authentication token in the console window.
 
 When you resend the request it shouldn't make the authentication request again (until the token expires).
