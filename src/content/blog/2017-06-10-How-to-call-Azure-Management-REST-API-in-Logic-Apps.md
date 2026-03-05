---
title: How to Call Azure Management REST API in Logic Apps
pubDate: 2017-06-10 15:26:51
tags:
- Azure
- Logic Apps
- CRM WebAPI
---
Logic Apps provide a way to simplify and implement scalable integrations and workflows in the cloud. Using [Logic Apps](https://azure.microsoft.com/en-us/services/logic-apps/), you can create an application to manage your service’s resources programmatically. But how to call the Azure Management API in Logic Aps?. In this article, we will demonstrate how to get the email address of the Azure subscription vai Azure Management REST API and send an email to in Logic Apps dynamically.
<!-- more -->
# Pre works:
## Create a management certificate
Please follow the steps in this document to create your management certificate:
https://docs.microsoft.com/en-us/azure/cloud-services/cloud-services-certs-create
## Upload your management certificate through Azure classic portal
Then you need to upload your management certificate to your subscription (public certificate .cer file) so that it is authorized to perform management operations on your behalf. Please follow this documentation for step-by-step details:
https://docs.microsoft.com/en-us/azure/azure-api-management-certs 
## Find the API you needed. 
In this sample, we will use an Azure Management REST API to get the User Accounts information (https://msdn.microsoft.com/en-us/library/azure/dn469420.aspx):
```
GET https://management.core.windows.net/<subscription-id>/principals
```

# Logic Apps:
## Part A: Create a Logic App to get the email of Admin (Parent Logic App). 
### Create a request trigger.
We created a request trigger here to start our workflow. But you can use any other triggers.
More information you can find here: https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-http-endpoint
### Create an HTTP action.
In this action, we will call Azure REST API using client certificate (pfx + password).
**_Note_**: You need to base64 encode the pfx file content and embed in the pfx textbox.
![image001.png](/images/2017-06-10-How-to-call-Azure-Management-REST-API-in-Logic-Apps/image001.png)

The format of the response body is a xml file as follows:

{% codeblock lang:xml %}
<?xml version="1.0" encoding="utf-8"?>
<Principal xmlns=”http://schemas.microsoft.com/windowsazure”>
  <Role>role-names-for-user-account</Role>
  <Email>email-address-for-user-account</Email>
</Principal>
{% endcodeblock %}

In Logic Apps, it is more convenient to pass the data as a JSON file between different actions or apps.
You can use @JSON() function to convert the XML content easily in Logic Apps as below:
![image003.png](/images/2017-06-10-How-to-call-Azure-Management-REST-API-in-Logic-Apps/image003.png)
In real word, you can define any actions after you got the response message.
Here we defined a child logic apps to parse the response and send email -- **sendEmail**.
## Part B: Create a Logic App to parse the response and send email to Admin (Child Logic Apps).
The sendEmail app takes an array of subscription admins and uses Send email action inside a ForEach loop
### Create a request trigger to accept the response.
Here is a sample of the JSON format playload:

{% codeblock lang:json %}
{
  "Principals": {
    "Principal": [
      {
        "Role": "ServiceAdministrator;AccountAdministrator",
        "Email": "user1@microsoft.com"
      },
      {
        "Role": "CoAdministrator",
        "Email": "user2@microsoft.com"
      }
    ]
  }
}
{% endcodeblock %}

Put the above JSON data to http://jsonschema.net to get the its JSON schema.
Paste the schema definition in request trigger. With the help of JSON schema, Logic App could automatically tokenize all properties e.g. Principals, EmailId etc.
![image005.png](/images/2017-06-10-How-to-call-Azure-Management-REST-API-in-Logic-Apps/image005.png)
### Create a ForEach action – Loop over all principals
**a. Send Email action** – Send email using Office 365 send email action.
![image007.png](/images/2017-06-10-How-to-call-Azure-Management-REST-API-in-Logic-Apps/image007.png)
**b. Response action** – Child workflows should have response action to be callable from another logic app use native child workflow action.

Now, when you run the application, you will get the email address of the Azure Subscription and send a customer email to him. Similarly, you can call any Azure Management API like this.

Thanks to Vinay Singh, Xiaodong Zhu.
Ray Wang 

