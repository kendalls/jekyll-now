---
layout: post
title: "Application Authentication to Dynamics 365"
description: "How to connect to Dynamics 365 from an external source without using a username and password."
thumb_image: ""
tags: [dynamics]
published: true
---
We'll register an app in Azure, create an application user in CRM, and write a bit of code.

## Step One: Register an App in Azure
1. Open the Azure Portal and navigate to **Azure Active Directory** > **App Registrations (Preview)** and click **+New registration**.
2. Fill in the required fields and click **Create**. You can use https://localhost as the sign-on URL in most cases.
3. Click the new app registration and then navigate to **API permissions** and click **+Add a permission**. Select **Dynamics CRM**, **Delegated permissions** and **user_impersonation**.
4. Now navigate to **Certificates & secrets** and click **+New client secret**. Keep the value somewhere safe because it won't be visible later.
5. Go back to the application overview and copy the values for **Application ID** and **Directory ID** as we'll need them later in code.
6. You can also click **Endpoints** in the app list and copy the **OAuth 2.0 authorization endpoint** value, but the default shown in the code will also work.

## Step Two: Create an Application User
1. In your D365 organization, navigate to **Settings** > **Security** > **Users** and click **+New**.
2. Change the form to **APPLICATION USER** and supply values for User Name, Full Name, Primary Email, and paste in the Application ID from above. The rest of the values will be populated when you save the record.
3. Create a new custom Security Role for the user and assign it to them.

## Step Three: C# Connection
This uses [David Yack's Web API Helper](https://github.com/davidyack/Xrm.Tools.CRMWebAPI) but that doesn't make much difference.
{% highlight cs %}
    
    using System.Threading.Tasks;

    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    using Xrm.Tools.WebAPI;

    public static class CRMWebApiHelper
    {
        public async static Task<CRMWebAPI> GetApi()
        {
            var authority = "https://login.microsoftonline.com/";
            var clientid = "8bdb******************";
            var crmBaseUrl = "https://********.crm6.dynamics.com";
            var clientSecret = "z2+*****************************";
            var tenantId = "6196*******************";

            var clientCredential = new ClientCredential(clientid, clientSecret);
            var authContext = new AuthenticationContext(authority + tenantId);
            var authResult = await authContext.AcquireTokenAsync(crmBaseUrl, clientCredential);

            return new CRMWebAPI(crmBaseUrl + "/api/data/v9.1/", authResult.AccessToken);
        }
    }
{% endhighlight %}
