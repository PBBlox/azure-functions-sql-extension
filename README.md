# Azure SQL bindings for Azure Functions - Preview

[![Build Status](https://mssqltools.visualstudio.com/CrossPlatBuildScripts/_apis/build/status/SQL%20Bindings/SQL%20Bindings%20-%20Nightly?branchName=main)](https://mssqltools.visualstudio.com/CrossPlatBuildScripts/_build/latest?definitionId=481&branchName=main)

## Table of Contents
- [Azure SQL bindings for Azure Functions - Preview](#azure-sql-bindings-for-azure-functions---preview)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Supported SQL Server Versions](#supported-sql-server-versions)
  - [Known Issues](#known-issues)
    - [Output Bindings](#output-bindings)
    - [Input Bindings](#input-bindings)
    - [Trigger Bindings](#trigger-bindings)
  - [Telemetry](#telemetry)
  - [Trademarks](#trademarks)

## Introduction

This repository contains the Azure SQL bindings for Azure Functions extension code as well as a quick start tutorial and samples illustrating how to use the binding in different ways. The types of bindings supported are:

- **Input Binding**: takes a SQL query to run and returns the output of the query in the function.
- **Output Binding**: takes a list of rows and upserts them into the user table (i.e. If a row doesn't already exist, it is added. If it does, it is updated).
- **Trigger Binding**: monitors the user table for changes (i.e., row inserts, updates, and deletes) and invokes the function with updated rows.

For a more detailed overview of the different types of bindings see the [Bindings Overview](./docs/BindingsOverview.md).

For further details on setup, usage and samples of the bindings see the language-specific guides below:

- [.NET (C# in-process)](./docs/SetupGuide_Dotnet.md)
- [.NET (C# out-of-proc)](./docs/SetupGuide_DotnetOutOfProc.md)
- [Java](./docs/SetupGuide_Java.md)
- [Javascript](./docs/SetupGuide_Javascript.md)
- [PowerShell](./docs/SetupGuide_PowerShell.md)
- [Python](./docs/SetupGuide_Python.md)

Further information on the Azure SQL binding for Azure Functions is also available in the [docs](https://aka.ms/sqlbindings).

## Supported SQL Server Versions

This extension uses the [OPENJSON](https://learn.microsoft.com/sql/t-sql/functions/openjson-transact-sql) statement which requires a database compatibility level of 130 or higher (2016 or higher). To view or change the compatibility level of your database, see [this documentation article](https://learn.microsoft.com/sql/relational-databases/databases/view-or-change-the-compatibility-level-of-a-database) for more information.

Databases on SQL Server, Azure SQL Database, or Azure SQL Managed Instance which meet the compatibility level requirement above are supported.

## Known Issues

### Output Bindings
- Output bindings against tables with columns of data types `NTEXT`, `TEXT`, or `IMAGE` are not supported and data upserts will fail. These types [will be removed](https://docs.microsoft.com/sql/t-sql/data-types/ntext-text-and-image-transact-sql) in a future version of SQL Server and are not compatible with the `OPENJSON` function used by this Azure Functions binding.
- Output bindings execution order is not deterministic ([azure-webjobs-sdk#1025](https://github.com/Azure/azure-webjobs-sdk/issues/1025)) and so the order that data is upserted is not guaranteed. This can be problematic if, for example, you upsert rows to two separate tables with one having a foreign key reference to another. The upsert will fail if the dependent table does its upsert first.

    Some options for working around this :
    * Have multiple functions, with dependent functions being triggered by the initial functions (through a trigger binding or other such method)
    * Use [dynamic (imperative)](https://learn.microsoft.com/azure/azure-functions/functions-bindings-expressions-patterns#binding-at-runtime) bindings (.NET only)
    * Use [IAsyncCollector](https://learn.microsoft.com/azure/azure-functions/functions-dotnet-class-library?tabs=v2%2Ccmd#writing-multiple-output-values) and call `FlushAsync` in the order desired (.NET only)

### Input Bindings
- Input bindings against tables with columns of data types 'DATETIME', 'DATETIME2', or 'SMALLDATETIME' will assume that the values are in UTC format.
- For Java Functions using Output bindings against a table with columns of data types 'DATETIME', 'DATETIME2', or 'SMALLDATETIME', use java.util.Date type to ensure the datetime is formatted correctly.

### Trigger Bindings
- Trigger bindings will exhibit undefined behavior if the SQL table schema gets modified while the user application is running, for example, if a column is added, renamed or deleted or if the primary key is modified or deleted. In such cases, restarting the application should help resolve any errors.

## Telemetry

This extension collect usage data in order to help us improve your experience. The data is anonymous and doesn't include any personal information. You can opt-out of telemetry by setting the `AZUREFUNCTIONS_SQLBINDINGS_TELEMETRY_OPTOUT` environment variable or the `AzureFunctionsSqlBindingsTelemetryOptOut` app setting (in your `*.settings.json` file) to '1', 'true' or 'yes';

## Trademarks

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft trademarks or logos is subject to and must follow [Microsoft’s Trademark & Brand Guidelines](https://www.microsoft.com/legal/intellectualproperty/trademarks/usage/general). Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship. Any use of third-party trademarks or logos are subject to those third-party’s policies.
