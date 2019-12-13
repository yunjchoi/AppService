---
title: "Configure App Service Certificate to Azure Virtual machines"
author_name: "mksunitha"
layout: single
excerpt: "Learn how to use your App Service Certificate with your Azure Virtual Machines."
toc: true
toc_sticky: true
---

App Service Certificate can be used for other Azure service and not just App Service Web App. This tutorial shows you how to secure your web app by purchasing an SSL certificate using App Service Certificates, securely storing it in [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-whatis), domain verification and configuring it your virtual machine. Before your begin log in to the [Azure portal](http://portal.azure.com/).

## Create an Azure Virtual machine with IIS web server

Create an Azure virtual machine with IIS from the [Azure Marketplace](https://docs.microsoft.com/azure/virtual-machines/windows/tutorial-secure-web-server#create-a-virtual-machine) or the Azure CLI.

## Add a Custom domain to your virtual machine

[Purchase a new domain](https://blogs.msdn.microsoft.com/appserviceteam/2017/07/31/assign-app-service-domain-to-azure-vm-or-azure-storage/) and assign it your Azure virtual machine.

## Place an SSL Certificate order

You can place an SSL Certificate order by creating a new [App Service Certificate](https://portal.azure.com/#create/Microsoft.SSL) in the Azure portal. Enter a friendly name for your SSL certificate and enter the Domain Name in Step 1. **DO NOT** append the Host name with WWW.

![](https://docs.microsoft.com/azure/app-service/media/app-service-web-purchase-ssl-web-site/createssl.png)

## Store the certificate in Azure Key Vault

[Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-whatis) is an Azure service that helps safeguard cryptographic keys and secrets used by cloud applications and services. Once the SSL Certificate purchase is complete, you need to open the [App Service Certificates page](https://portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.CertificateRegistration%2FcertificateOrders). The current status of the certificate is **“Pending Issuance”**. Complete the steps below to have an active certificate ready to use.

Click **Certificate Configuration** inside the Certificate Properties page and Click on **Step 1: Store** to store this certificate in Azure Key Vault.

![store-the-cert](https://docs.microsoft.com/azure/app-service/media/app-service-web-purchase-ssl-web-site/readykv.png)

From the **Key Vault Status** page, click **Key Vault Repository** to choose an existing Key Vault to store this certificate or **Create New Key Vault** to create new Key Vault inside same subscription and resource group.

> Azure Key Vault has minimal charges for storing this certificate. For more information, see [Azure Key Vault Pricing Details](https://azure.microsoft.com/pricing/details/key-vault/).

Once you have selected the Key Vault Repository to store this certificate in, the **Store** option should show success.

![success](https://docs.microsoft.com/azure/app-service/media/app-service-web-purchase-ssl-web-site/kvstoresuccess.png)

## Verify the domain ownership

From the same **Certificate Configuration** page you used in Step 3, click **Step 2: Verify**. Choose the preferred domain verification method.

There are four types of domain verification supported by App Service Certificates: App Service, Domain, Mail, and Manual Verification. These verification types are explained in more details in the Advanced section.

## Assign certificate to Virtual machine

Before performing the steps in this section dedicated for Virtual machine, you must have done the following:

1. Associated a custom domain name with your app on the virtual machine. For more information, see [configuring a custom domain name for a web app](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain).
1. Make sure Key Vault has appropriate permissions to be used with Virtual machine. For more information, see Using MSI with Key Vault on Virtual machine

Once you have associated the domain and granted permissions to the Vm, follow the steps below to assign the certificate to an App Service App.

1. Get the Key Vault information for your SSL certificate resource under certificate configuration.
1. Prepare and Configure the virtual machine to add the Certificate.

An App Service Certificate can be used on multiple Azure Virtual Machines. [Learn more](https://docs.microsoft.com/azure/azure-stack/user/azure-stack-kv-push-secret-into-vm)

![done]({{ site.baseurl }}/media/2017/10/ASC-virtualmachine-1024x423.png)

## References

- [Internals of App Service Certificate](https://azure.microsoft.com/blog/internals-of-app-service-certificate/)
- [Get started with Azure Key Vault](https://docs.microsoft.com/azure/key-vault/)
