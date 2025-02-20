---
title: Ingest data from a container or Azure Data Lake Storage into Data Explorer
description: Ingest (load) data into a new Azure Data Explorer table from a container or ADLS, either as a one-time or continuous operation.
ms.reviewer: tzgitlin
ms.topic: how-to
ms.date: 09/04/2022
---

# Ingest data from a container into Azure Data Explorer

> [!div class="op_single_selector"]
> * [Ingestion wizard](ingestion-wizard-new-table.md)
> * [Portal](ingest-data-event-grid.md)
> * [C#](data-connection-event-grid-csharp.md)
> * [Python](data-connection-event-grid-python.md)
> * [Azure Resource Manager template](data-connection-event-grid-resource-manager.md)

The [ingestion wizard](./ingest-data-wizard.md) enables you to quickly ingest data in JSON, CSV, and other formats into a table and easily create mapping structures. The data can be ingested either from storage, from a local file, or from a container, or as a one-time or continuous ingestion process.

This document describes using the intuitive ingestion wizard to ingest **CSV** data from a **container** into a **new table**. Ingestion can be done as a one-time operation, or as a continuous method by [setting up an Event Grid ingestion pipeline](#create-continuous-ingestion) that responds to new files in the source container and ingests qualifying data into your table. This process can be used with slight adaptations to cover a variety of different use cases.

For an overview of the ingestion wizard, see [What is the ingestion wizard?](./ingest-data-wizard.md).
For information about ingesting data into an existing table in Azure Data Explorer, see [Ingest data to an existing table](./ingestion-wizard-existing-table.md)

## Prerequisites

* An Azure subscription. Create a [free Azure account](https://azure.microsoft.com/free/).
* An Azure Data Explorer cluster and database. [Create a cluster and database](create-cluster-database-portal.md).
* A [storage account](/azure/storage/common/storage-quickstart-create-account?tabs=azure-portal). Event Grid notification subscription can be set on Azure Storage accounts for `BlobStorage`, `StorageV2`, or [Data Lake Storage Gen2](/azure/storage/blobs/data-lake-storage-introduction).

> [!NOTE]
> To enable access between a cluster and a storage account without public access (restricted to private endpoint/service endpoint), see [Create a Managed Private Endpoint](security-network-managed-private-endpoint-create.md).

## Ingest data

1. In the left menu of the Azure Data Explorer web UI, select **Data**.

1. From the **Quick actions** section, select **Ingest data**. Alternatively, from the **All** section, select **Ingest data** and then **Ingest**.

    :::image type="content" source="media/ingestion-wizard-new-table/ingest-new-data.png" alt-text="Screenshot for the Azure Data Explorer web UI where you select ingestion for a table.":::

1. In the **Ingest data** window, the **Destination** tab is selected. The **Cluster** and **Database** fields are automatically populated.

    [!INCLUDE [ingestion-wizard-cluster](includes/ingestion-wizard-cluster.md)]

1. In **Table**, check **New table** and enter a name for the new table. You can use alphanumeric, hyphens, and underscores. Special characters aren't supported.

    > [!NOTE]
    > Table names must be between 1 and 1024 characters.

    :::image type="content" source="media/ingestion-wizard-new-table/create-new-table.png" alt-text="Screenshot of Azure Data Explorer to create a new table to ingest data":::

1. Select **Next: Source**

## Select an ingestion type

Under **Source type**, do the following steps:

1. Select **From blob container** (blob container, ADLS Gen2 container). You can ingest up to 5000 blobs from a single container.

1. For **Select source**, select **Add URL**.

   > [!NOTE]
   > Alternatively, you can select **Select container** and choose information from the dropdown menus to connect to the container.

1. In the **Link to source** field, add the [blob URI with SAS token or Account key](kusto/api/connection-strings/generate-sas-token.md) of the container, and optionally enter the sample size. A list is populated with files from the container.

   > [!NOTE]
   > The SAS URL can be created [manually](/azure/vs-azure-tools-storage-explorer-blobs#get-the-sas-for-a-blob-container) or [automatically](kusto/api/connection-strings/storage-connection-strings.md).

   :::image type="content" source="media/ingestion-wizard-new-table/from-container.png" alt-text="Screenshot selecting ingestion from container in the ingestion wizard.":::

   > [!TIP]
   > For ingestion **from file**, see [Use the ingestion wizard to ingest JSON data from a local file to an existing table in Azure Data Explorer](./ingestion-wizard-existing-table.md#select-an-ingestion-type)

## Filter data

Optionally, you can filter data to be ingested with **File filters**. You can filter by file extension, file location, or both.

### Filter by file extension

You can filter data to ingest only files with a specific file extension.

* For example, filter for all files with a **CSV** extension.

  :::image type="content" source="media/ingestion-wizard-new-table/from-container-with-filter.png" alt-text="Screenshot of Ingest data tab showing the ingestion filter.":::

  The system will select one of the files at random and the schema will be generated based on that **Schema defining file**. You can select a different file.

### Filter by folder path

You can also filter files with the full or partial **Folder path**.

* You can enter a partial folder path, or folder name.

  :::image type="content" source="media/ingestion-wizard-new-table/filter-folder-path-search.png" alt-text="Screenshot of the folder path search to filter files when ingesting data with the ingestion wizard.":::

* Alternatively, enter the full folder path.

    1. Go to the storage account, and select **Storage Explorer > Blob Containers**

       :::image type="content" source="media/ingestion-wizard-new-table/storage-browser-blob-containers.png" alt-text="Screenshot access blob containers in Azure Storage account.":::

    1. Browse to the selected folder, and select full folder path.

       :::image type="content" source="media/ingestion-wizard-new-table/copy-path.png" alt-text="Screenshot of a folder path to folder in blob container - Azure Storage account.":::

    1. Copy the full folder path and paste it into a temporary file.
    1. Insert `/` in between each folder to create the folder path and enter this path into the **Folder path** field to select this folder.

## Edit the schema

Select **Next: Schema** to view and edit your table column configuration. The service automatically identifies if the schema is compressed by looking at the name of the source.

In the **Schema** tab:

1. Confirm the format selected in **Data format**:

    In this case, the data format is **CSV**

    > [!TIP]
    > If you want to use **JSON** files, see [Use the ingestion wizard to ingest JSON data from a local file to an existing table in Azure Data Explorer](./ingestion-wizard-existing-table.md#edit-the-schema).

1. You can select the check box **Ignore the first record** to ignore the heading row of the file.

    :::image type="content" source="media/ingestion-wizard-new-table/non-json-format.png" alt-text="Screenshot showing how to select the option not to include column names in the ingestion wizard.":::

1. In the **Mapping name** field, enter a mapping name. You can use alphanumeric characters and underscores. Spaces, special characters, and hyphens aren't supported.

### Edit the table

When ingesting to a new table, alter various aspects of the table when creating the table.

[!INCLUDE [data-explorer-ingestion-wizard-column-table](includes/data-explorer-ingestion-wizard-column-table.md)]

> [!NOTE]
> For tabular formats, you can't map a column twice. To map to an existing column, first delete the new column.

[!INCLUDE [data-explorer-ingestion-wizard-command-editor](includes/data-explorer-ingestion-wizard-command-editor.md)]

Select **Next: Summary** to create a table and mapping and to begin data ingestion.

## Complete data ingestion

In the **Data ingestion completed** window, all three steps will be marked with green check marks when data ingestion finishes successfully.

:::image type="content" source="media/ingestion-wizard-new-table/one-click-data-ingestion-complete.png" alt-text="Screenshot showing ingested complete dialog box with data preview.":::

[!INCLUDE [data-explorer-ingestion-wizard-query-data](includes/data-explorer-ingestion-wizard-query-data.md)]

## Create continuous ingestion

Continuous ingestion enables you to create an Event Grid that listens for new files in the source container. Any new file that meets the criteria of the pre-defined parameters (prefix, suffix, and so on) will be automatically ingested into the destination table.

1. Select **Event Grid** in the **Continuous ingestion** tile to open the Azure portal. The data connection page opens with the Event Grid data connector opened and with source and target parameters already entered (source container, tables, and mappings).

    :::image type="content" source="media/ingestion-wizard-new-table/continuous-button.png" alt-text="Screenshot showing the continuous ingestion button.":::

### Data connection: Basics

1. The **Data connection** blade opens with the **Basics** tab selected.
1. Enter the **Storage account**.
1. Choose the **Event type** that will trigger ingestion.
1. Select **Next: Ingest properties**

:::image type="content" source="media/ingestion-wizard-new-table/data-connection-basics-tab.png" alt-text="Screenshot of Data connection blade with Basics tab selected. Fields that should be selected are highlighted by a red box.":::

### Ingest properties

The **Ingest properties** tab opens with pre-filled routing settings. The target table name, format, and mapping name are taken from the table created above.

:::image type="content" source="media/ingestion-wizard-new-table/ingest-properties.png" alt-text="Screenshot of Ingest properties blade.":::

Select **Next: Review + create**

### Review + create

Review the resources, and select **Create**.

:::image type="content" source="media/ingestion-wizard-new-table/review-create.png" alt-text="Screenshot of review and create blade.":::

## Next steps

* [Query data in Azure Data Explorer web UI](web-query-data.md)
* [Write queries for Azure Data Explorer using Kusto Query Language](write-queries.md)