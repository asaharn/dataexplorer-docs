---
title: Streaming ingestion failures - Azure Data Explorer
description: This article describes the command to show streaming ingestion failures in Azure Data Explorer.
ms.topic: reference
ms.date: 02/21/2023
---

# Streaming ingestion failures

## .show streamingingestion failures

This command returns a result set that includes aggregated streaming ingestion failures that occur when [data is ingested using one of the streaming ingestion types](../../ingest-data-streaming.md#choose-the-appropriate-streaming-ingestion-type).

> [!NOTE]
> Streaming ingestion failures are grouped into buckets of short periods of time and aggregated by database, table, principal, ingestion properties, failure kind, and error code.
>
> Gateway throttling failures will not appear in the result set of this command.
>
> The retention period for streaming ingestion failures is 14 days.

## Permissions

If you have Database Admin or Database Monitor permissions, you'll see all failed operations. Otherwise, you'll only see operations that you created. For more information about permissions, see [role-based access control](access-control/role-based-access-control.md).

## Syntax

| Syntax option | Description |
|--|--|--|
| `.show` `streamingingestion` `failures` | Returns aggregated streaming ingestion failures |
| `.show` `streamingingestion` `failures` \| `where` ... | Returns a filtered set of streaming ingestion failures |

## Returns

| Output parameter | Type | Description |
|--|--|--|
| Database | String | Target database of the ingestion |
| Table | String | Target table of the ingestion |
| Principal | String | The principal whosе credentials were used for the ingestion |
| RootActivityId | Guid | The ingestion Root Activity ID |
| IngestionProperties | Dynamic | The ingestion properties |
| Count | Long | The total count of failures in the bucket |
| FirstFailureOn | DateTime | Date/time (in UTC) of the first ingestion in the bucket |
| LastFailureOn | DateTime | Date/time (in UTC) of the last ingestion in the bucket |
| FailureKind | String | Type of the failure (Permanent/Transient) |
| ErrorCode | String | The error code of the failure |
| Details | String | The details about the failure |

## Example

| Database | Table | Principal | RootActivityId | IngestionProperties | Count | FirstFailureOn | LastFailureOn | FailureKind | ErrorCode | Details |
|--|--|--|--|--|--|--|--|--|--|--|
| DB1 | Table1 | aadapp=xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx | 04411033-5ffb-410f-bc6d-a24d50790a80 | { "Mapping": "Mapping_name", "Format": "Csv", "Compressed": true, "IngestionSource": "Storage" } | 2 | 2020-10-11 12:06:35.8362967 | 2020-10-11 12:06:35.8362967 | Transient | Kusto.DataNode.Exceptions.StreamingIngestionServiceException | Server error in performing streaming ingestion into xxxx : Cannot determine row store for ingestion |
| DB1 | Table1 | aadapp=xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx | d025b3e1-8f3d-4864-be0a-de54ce5246be | { "Mapping": null, "Format": "Csv", "Compressed": false, "IngestionSource": "Stream" } | 3 | 2020-10-11 12:07:40.8362967 | 2020-10-11 12:08:35.8362967 | Permanent | Kusto.DataNode.Exceptions.StreamingIngestionServiceException | Database metadata is unavailable. |
