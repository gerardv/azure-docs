---
title: Bind an Azure Cosmos DB to your application in Azure Spring Apps
description: Learn how to bind Azure Cosmos DB to your application in Azure Spring Apps
author: karlerickson
ms.service: spring-apps
ms.topic: how-to
ms.date: 11/09/2022
ms.author: karler
ms.custom: devx-track-java, event-tier1-build-2022, ignite-2022, service-connector
---

# Bind an Azure Cosmos DB database to your application in Azure Spring Apps

> [!NOTE]
> Azure Spring Apps is the new name for the Azure Spring Cloud service. Although the service has a new name, you'll see the old name in some places for a while as we work to update assets such as screenshots, videos, and diagrams.

**This article applies to:** ✔️ Java ❌ C#

**This article applies to:** ✔️ Basic/Standard ✔️ Enterprise

Instead of manually configuring your Spring Boot applications, you can automatically bind select Azure services to your applications by using Azure Spring Apps. This article demonstrates how to bind your application to an Azure Cosmos DB database.

## Prerequisites

* A deployed Azure Spring Apps instance.
* An Azure Cosmos DB account and a database.
* The Azure Spring Apps extension for the Azure CLI.

If you don't have a deployed Azure Spring Apps instance, follow the steps in the [Quickstart: Deploy your first application to Azure Spring Apps](./quickstart.md).

## Prepare your Java project

1. Add one of the following dependencies to your application's pom.xml pom.xml file. Choose the dependency that is appropriate for your API type.

   * API type: NoSQL

     ```xml
     <dependency>
         <groupId>com.azure.spring</groupId>
         <artifactId>spring-cloud-azure-starter-data-cosmos</artifactId>
         <version>4.7.0</version>
     </dependency>
     ```

   * API type: MongoDB

     ```xml
     <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-data-mongodb</artifactId>
     </dependency>
     ```

   * API type: Cassandra

     ```xml
     <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-data-cassandra</artifactId>
     </dependency>
     ```

   * API type: Azure Table

     ```xml
     <dependency>
         <groupId>com.azure.spring</groupId>
         <artifactId>spring-cloud-azure-starter-storage-blob</artifactId>
         <version>4.7.0</version>
     </dependency>
     ```

1. Update the current app by running `az spring app deploy`, or create a new deployment for this change by running `az spring app deployment create`.

## Bind your app to the Azure Cosmos DB

### [Service Connector](#tab/Service-Connector)

#### Use the Azure CLI

Use the following command to configure your Spring app to connect to a Cosmos SQL Database with a system-assigned managed identity:

> [!NOTE]
> Updating Azure Cosmos DB database settings can take a few minutes to complete.

```azurecli
az spring connection create cosmos-sql \
    --resource-group $AZURE_SPRING_APPS_RESOURCE_GROUP \
    --service $AZURE_SPRING_APPS_SERVICE_INSTANCE_NAME \
    --app $APP_NAME \
    --deployment $DEPLOYMENT_NAME \
    --target-resource-group $COSMOSDB_RESOURCE_GROUP \
    --account $COSMOSDB_ACCOUNT_NAME \
    --database $DATABASE_NAME \
    --system-assigned-identity
```

> [!NOTE]
> If you're using [Service Connector](../service-connector/overview.md) for the first time, start by running the command `az provider register --namespace Microsoft.ServiceLinker` to register the Service Connector resource provider.
>
> If you're using Cosmos Cassandra, use a `--key_space` instead of `--database`.

> [!TIP]
> Run the command `az spring connection list-support-types --output table` to get a list of supported target services and authentication methods for Azure Spring Apps. If the `az spring` command isn't recognized by the system, check that you have installed the required extension by running `az extension add --name spring`.

#### Use the Azure portal

Alternately, you can use the Azure portal to configure this connection by completing the following steps. The Azure portal provides the same capabilities as the Azure CLI and provides an interactive experience.

1. Select your Azure Spring Apps instance in the Azure portal and select **Apps** from the navigation menu. Choose the app you want to connect and select **Service Connector** on the navigation menu.

1. Select **Create**.

1. On the **Basics** tab, for service type, select Cosmos DB, then choose a subscription. For API type, select Core (SQL), choose a Cosmos DB account, and a database. For client type, select Java, then select **Next: Authentication**. If you haven't created your database yet, see [Quickstart: Create an Azure Cosmos DB account, database, container, and items from the Azure portal](../cosmos-db/nosql/quickstart-portal.md).

1. On the **Authentication** tab, choose **Connection string**. Service Connector automatically retrieves the access key from your Cosmos DB account. Select **Next: Networking**.

1. On the **Networking** tab, select **Configure firewall rules to enable access to target service**, then select **Next: Review + Create**.

1. On the **Review + Create** tab, wait for the validation to pass and then select **Create**. The creation can take a few minutes to complete.

1. Once the connection between your Spring apps and your Cosmos DB database has been generated, you can see it in the Service Connector page and select the unfold button to view the configured connection variables.

### [Service Binding](#tab/Service-Binding)

> [!NOTE]
> We recommend using Service Connector instead of Service Binding to connect your app to your database. Service Binding is going to be deprecated in favor of Service Connector. For instructions, see the Service Connector tab.

Azure Cosmos DB has five different API types that support binding. The following procedure shows how to use them:

1. Create an Azure Cosmos DB database. Refer to the quickstart on [creating a database](../cosmos-db/create-cosmosdb-resources-portal.md) for help.

1. Record the name of your database. For this procedure, the database name is **testdb**.

1. Go to your Azure Spring Apps service page in the Azure portal. Go to **Application Dashboard** and select the application to bind to Azure Cosmos DB. This application is the same one you updated or deployed in the previous step.

1. Select **Service binding**, and select **Create service binding**. To fill out the form, select:

   * The **Binding type** value **Azure Cosmos DB**.
   * The API type.
   * Your database name.
   * The Azure Cosmos DB account.

   > [!NOTE]
   > If you are using Cassandra, use a key space for the database name.

1. Restart the application by selecting **Restart** on the application page.

1. To ensure the service is bound correctly, select the binding name and verify its details. The `property` field should be similar to this example:

   ```properties
   spring.cloud.azure.cosmos.endpoint=https://<some account>.documents.azure.com:443
   spring.cloud.azure.cosmos.key=abc******
   spring.cloud.azure.cosmos.database=testdb
   ```

### [Terraform](#tab/Terraform)

The following Terraform script shows how to set up an app deployed to Azure Spring Apps with an Azure Cosmos DB account.

```terraform
provider "azurerm" {
  features {}
}

variable "application_name" {
  type        = string
  description = "The name of your application"
  default     = "demo-abc"
}

resource "azurerm_resource_group" "example" {
  name     = "example-resources"
  location = "West Europe"
}

resource "azurerm_cosmosdb_account" "cosmosdb" {
  name                = "cosmosacct-${var.application_name}-001"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  offer_type          = "Standard"
  kind                = "GlobalDocumentDB"

  consistency_policy {
    consistency_level = "Session"
  }

  geo_location {
    failover_priority = 0
    location          = azurerm_resource_group.example.location
  }
}

resource "azurerm_cosmosdb_sql_database" "cosmosdb" {
  name                = "cosmos-${var.application_name}-001"
  resource_group_name = azurerm_cosmosdb_account.cosmosdb.resource_group_name
  account_name        = azurerm_cosmosdb_account.cosmosdb.name
}

resource "azurerm_spring_cloud_service" "example" {
  name                = "${var.application_name}"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
}

resource "azurerm_spring_cloud_app" "example" {
  name                = "${var.application_name}-app"
  resource_group_name = azurerm_resource_group.example.name
  service_name        = azurerm_spring_cloud_service.example.name
  is_public           = true
  https_only          = true
}

resource "azurerm_spring_cloud_java_deployment" "example" {
  name                = "default"
  spring_cloud_app_id = azurerm_spring_cloud_app.example.id
  quota {
    cpu    = "2"
    memory = "4Gi"
  }
  instance_count      = 2
  jvm_options         = "-XX:+PrintGC"
  runtime_version     = "Java_11"

  environment_variables = {
    "spring.cloud.azure.cosmos.endpoint" : azurerm_cosmosdb_account.cosmosdb.endpoint
    "spring.cloud.azure.cosmos.key" : azurerm_cosmosdb_account.cosmosdb.primary_key
    "spring.cloud.azure.cosmos.database" : azurerm_cosmosdb_sql_database.cosmosdb.name
  }
}

resource "azurerm_spring_cloud_active_deployment" "example" {
  spring_cloud_app_id = azurerm_spring_cloud_app.example.id
  deployment_name     = azurerm_spring_cloud_java_deployment.example.name
}
```

---

## Next steps

In this article, you learned how to bind your application in Azure Spring Apps to an Azure Cosmos DB database. To learn more about binding services to your application, see [Bind to an Azure Cache for Redis cache](./how-to-bind-redis.md).
