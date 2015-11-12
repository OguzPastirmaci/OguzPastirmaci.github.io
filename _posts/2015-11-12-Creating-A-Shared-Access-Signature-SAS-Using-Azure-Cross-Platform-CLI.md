---
layout: post
title: Creating a Shared Access Signature (SAS) using Azure Cross Plaftorm CLI
---

A Shared Access Signature allows you to grant a client limited permissions to objects in your storage account for a specified period of time and with a specified set of permissions, without having to share your account access keys. Visit [this page ](https://azure.microsoft.com/en-us/documentation/articles/storage-dotnet-shared-access-signature-part-1/)for additional information.

Recently I needed to create a SAS token using Azure Cross Platform CLI. The Azure CLI provides a set of open source, cross-platform commands for working with the Azure Platform. For more information, you may visit the [GitHub repo](https://github.com/Azure/azure-xplat-cli).

Below is a very simple shell script to create the token:

{% gist 628a907f0be57ed4b47c %}
