---
layout: post
title: "Use the Windows Azure REST APIs to manage your subscriptions - Part 1"
date: 2013-06-06 7:55:46 +02:00
categories: Windows Azure
author: "Julien Corioland"
identifier: 5d97b88b-dcec-42af-9890-26e40575a053
redirect_from:
  - /archives/use-the-windows-azure-rest-apis-to-manage-your-subscriptions-part-1
  - /Archives/use-the-windows-azure-rest-apis-to-manage-your-subscriptions-part-1
---

Maybe you are familiar with the Windows Azure portal that allows you to manage your hosted services, deployments, storage accounts, service bus namespaces or anything you want to do on your Azure subscription. Actually, all the operations you can perform on this portal are also available in sets of http management REST APIs. All you need to access these operations is a development platform that is able to create and send an http request with a client certificate !

In this post, I will use C# and .NET to manipulate the Windows Azure REST APIs but you really can use any other platforms, that is the power of Azure <img class="wlEmoticon wlEmoticon-winkingsmile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Winking smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-winkingsmile_75B1B9BD.png">

### First step : create and upload the client certificate

In a running in Administrator Visual Studio Command Prompt, type the following command :

```plain
makecert -sky exchange -r -n "CN=<CertificateName>" -pe -a sha1 -len 2048 -ss My "<CertificateName>.cer"
```
This command makes two things :

- It generates a certificate file named `<CertificateName>.cer`

- It automatically register the certificate in your `My` store certificate for the current user on the machine you are running this command.

Now, you have to upload the management certificate to the Windows Azure subscription you want manage through REST APIs. To do that, log in to the management portal at [http://manage.windowsazure.com](http://manage.windowsazure.com).

Click on the Settings section at the bottom of the left menu to access the management certificates. Now you can download your certificate from your computer. Choose the generated certificate and upload it to your subscription. Once the certificate is uploaded the list is refreshed and you can copy the thumbprint of the certificate. You will use this thumbprint to access the management REST API. You can also copy the id of the subscription that is needed to manage the subscription.

### Second step : write the code to retrieve the management certificate from store

Because you have to add the management certificate in the client certificates of the http web request, you have to retrieve it from you local store. The thumbprint is used to get the certificate :

```csharp
private X509Certificate2 GetManagementCertificateFromStore(string certThumbprint)
{
X509Store certStore = new X509Store(StoreName.My, StoreLocation.CurrentUser);
certStore.Open(OpenFlags.ReadOnly);
certStore.Open(OpenFlags.ReadOnly);

X509Certificate2Collection certCollection =
certStore.Certificates.Find(X509FindType.FindByThumbprint, certThumbprint, false);

certStore.Close();

if (certCollection.Count == 0)
{
throw new InvalidOperationException("The certificate with thumbprint {0} was not found in the My store of the current user");
}

return certCollection[0];
}
```

It’s also possible to load the client certificate from any other place as soon as you get it as a X509Certificate2 reference !

### Third step : access the management API with an HttpWebRequest

To begin, you have to choose which operation you want execute. A complete list of all operations that are available in the management REST APIs and the description of each operation is available [on this page](http://msdn.microsoft.com/en-us/library/windowsazure/ee460799.aspx). For example, you may want to list all the hosted services that are declared in your subscription.

If you read the documentation, you will see that you just have to execute a GET http request on the following URI:

```plain
https://management.core.windows.net/<subscription-id>/services/hostedservices
```
Where <subscription-id> is your subscription id…

Each http request to the management API should be authenticated with the client certificate, so you just have to retrieve the certificate from store and append it to the client certificates collection of the web request :

```csharp
var certificate = GetManagementCertificateFromStore("<thumbprint>");
HttpWebRequest request = (HttpWebRequest)HttpWebRequest.Create([https://management.core.windows.net/<subscription-id>/services/hostedservices](https://management.core.windows.net/<subscription-id>/services/hostedservices));
request.ClientCertificates.Add(certificate);
```
You also have to add some headers to make the http request valid :

- x-ms-version : represents the version of the management API you want to use. For example the last version is 2013-03-01

- ContentType, most of the time application/xml

```csharp
request.Headers.Add("x-ms-version", "2013-03-01");
request.ContentType = "application/xml";
```
And finally you can execute the web request and wait for the response !

### Fourth step : handle the http response

Depending on the operation you have executed the response status code is not the same. In the example of listing hosted services, success will be represented by a 200 (OK) http status code. If you had choose to create a new deployment you have been received an http CREATED status code. The expected status code is given on the description of each http operation in the MSDN documentation.

If the request was not well formed, you will received a BAD REQUEST http status code.

In both success or error cases, the response content is described as an XML document. The response body is also described in the description of each operation on MSDN. For the listing hosted services example, the response body will look like the following :

```xml
<?xml version="1.0" encoding="utf-8"?>
<HostedServices xmlns=`http://schemas.microsoft.com/windowsazure`>
<HostedService>
<Url>hosted-service-address</Url>
<ServiceName>hosted-service-name</ServiceName>
<HostedServiceProperties>
<Description>description</Description>
<AffinityGroup>affinity-group</AffinityGroup>
<Location>service-location</Location>
<Label>label</Label>
<Status>status</Status>
<DateCreated>date-created</DateCreated>
<DateLastModified>date-modified</DateLastModified>
<ExtendedProperties>
<ExtendedProperty>
<Name>property-name</Name>
<Value>property-value</Value>
</ExtendedProperty>
</ExtendedProperties>
</HostedServiceProperties>
</HostedService>
</HostedServices>
```
In case you get an error status code, the response body will look like the following :

```xml
<?xml version="1.0" encoding="utf-8"?>
<Error>
<Code>string-code</Code>
<Message>detailed-error-message</Message>
</Error>
```
You can find a list of all error codes you may received on [this page](http://msdn.microsoft.com/en-us/library/windowsazure/ee460801.aspx).

### Conclusion

You are now able to use the REST APIs to manage your Windows Azure subscriptions.

In the next post, I will discuss about more advanced operations like creating deployments or hosted services.

Hope this helps ! <img class="wlEmoticon wlEmoticon-smile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-smile_669ADAE3.png">

Julien

