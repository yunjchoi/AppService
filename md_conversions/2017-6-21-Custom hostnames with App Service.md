---
title: "Custom hostnames with App Service"
author_name: akurmi
layout: post
hide_excerpt: true
---
      [akurmi](https://social.msdn.microsoft.com/profile/akurmi)  6/21/2017 11:47:11 AM  Introduction
------------

 App Service has been supporting custom hostnames for a while now, you can read about this feature [here](https://docs.microsoft.com/en-us/azure/app-service-web/web-sites-custom-domain-name). App Service has a shared front-end so before you can use a custom hostname with your app, you need to verify domain ownership by creating the required DNS records. This is done to prevent hostname squatting vulnerability. To add a custom hostname to your app, you need to create DNS records for ownership verification and routing. Details
-------

 There are a few ways to add custom hostnames, most commonly used methods are given below:  2. CNAME: This is the recommended approach and easier of the two as it requires only one DNS record. To add a custom hostname using CNAME method, you need to create a CNAME record on the custom hostname pointing to the default hostname of the App. For example, if you want to add ‘support.api.contoso.com’ to a web app named ‘contoso’ then you should create a CNAME record on ‘support.api.contoso.com’ pointing to ‘contoso.azurewebsites.net’. In this case, the CNAME record is used for both, verification and routing.    DNS Record Type Sub-domain Value   CNAME support.api contoso.azurewebsites.net    
 4. A with TXT: In certain scenarios, a customer may not able to create the required CNAME record. For example, a lot of DNS providers don’t allow CNAME record on the root hostname. In that case, a customer can use this verification method. This requires creating two DNS records on the hostname being added. For example, if a customer wants to add ‘contoso.com’ to a web app named ‘contoso’ then he should create: 
	 - An A record on ‘contoso.com’ pointing to web app’s IP address. You can get this IP address from Azure portal or by resolving the default hostname locally. One of the convenient ways to resolve hostname is to simply ping it: ping contoso.azurewebsites.net. A record is used for routing traffic to your app.
	 - A TXT record on ‘contoso.com’ with value set to the default hostname. In this case, the value should be contoso.azurewebsites.net. TXT record is used for verification.
	     DNS Record Type Sub-domain Value   A @ <IP Address>   TXT @ contoso.azurewebsites.net    
  Recent updates
--------------

 We recently made the following two changes in our hostname management model:  2. One to many associations between hostname and Azure subscription: Earlier, a hostname could only be used with one subscription on App Service platform. With this change, now a hostname can be shared by multiple subscriptions. Each subscription needs to validate the domain ownership separately by creating the required DNS records.
 4. Allowing a subscription to reclaim a hostname by validating domain ownership: App Service platform consists of multiple scale units. A hostname can only be associated with one app on a scale unit. Even two apps in the same subscription cannot share the same hostname on a scale unit. This is by design as App Service runtime should be able to resolve a custom hostname to an App uniquely. We have received multiple customer reported incidents in the past where a customer had a custom hostname in his old subscription that he could not access anymore for some reason and wanted to use the same hostname with another app on the scale unit. This was not allowed earlier even though the customer could revalidate domain ownership. With this change, now App Service customers can do this themselves.
  Apart from these two changes, I would also like to cover some of the most common domain related questions we receive our customers: Migrating a live custom hostname without any downtime
-----------------------------------------------------

 If you would like to move a live custom hostname to App Service, then following the typical verification method would cause downtime between updating the DNS records and adding the corresponding hostname to your web app. If you would like to prevent the downtime, then you can use the alternate ‘awverify’ subdomain instead for verification. Say you would like to add ‘www.contoso.com’ to a web app named ‘contoso’ without any downtime. You can follow these steps to achieve this objective.  2. Create a TXT record on awverify.www.contoso.com with value set to ‘contoso.azurewebsites.net’ for verification    DNS Record Type Sub-domain Value   TXT awverify.www contoso.azurewebsites.net    
 4. Add the custom hostname through Azure portal. Once this is done, you can delete the TXT record created in step 1 if you want.
 6. Create a CNAME/A record on www.contoso.com to point it to ‘contoso.azurewebsites.net’. This would start routing traffic to your App Service App.    DNS Record Type Subdomain Value   CNAME www contoso.azurewebsites.net    Or    DNS Record Type Subdomain Value   A www <IP Address>    
  Adding the same custom hostname to multiple web apps
----------------------------------------------------

 There are scenarios where a customer would like to add the same hostname to multiple web apps in the same subscription, having a geo distributed website is one example. Our custom hostname feature allows you to bypass validation for hostnames that have already been validated. You only need to verify domain ownership when you add a hostname for the first time. For all other apps in the same subscription, you can add the same hostname without creating any DNS records. Programmatically manage custom hostnames
----------------------------------------

 We have updated our API surface to expose each custom hostname as a separate resource, giving App Service customers the ability to manage each hostname individually. A custom hostname is represented as ‘Microsoft.Web/site/hostnameBindings’ resource. For example, if you want to add ‘www.contoso.com’ to a web app called ‘contoso’ then first you need to create a CNAME record on this hostname pointing to ‘contoso.azurewebsites.net’. After that, you can add the hostname by calling the following ARM API: *ARMClient PUT /subscriptions/fb2c25dc-6bab-45c4-8cc9-cece7c42a95a/resourceGroups/Default-Web-EastAsia/providers/Microsoft.Web/sites/contoso/hostnameBindings/www.contoso.com?api-version=2016-03-01 "{'Location':'West US','Properties':{}}"* You can also use the following ARM template for adding custom hostnames: <https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-custom-domain>     