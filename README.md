# StorageQueueDemo

A .NET console application that reads and processes messages from an **Azure Storage Queue**.

Built as part of the course **Microsoft Azure From Zero to Hero - The Complete Guide**.

## Overview

The application connects to an Azure Storage account, checks that the queue `store-messages` exists, and then runs in a continuous loop that:

1. Receives the next message from the queue (one message at a time).
2. Prints the message text to the console.
3. Deletes the message from the queue after it has been processed.

## Tech Stack

| Item | Value |
|------|-------|
| Language | C# |
| Framework | .NET 10 (`net10.0`) |
| Project type | Console application |
| NuGet package | `Azure.Storage.Queues` 12.21.0 |

## Project Structure

```
StorageQueueDemo/
├── .vscode/
│   ├── launch.json          # VS Code debug configuration
│   └── tasks.json           # VS Code build / publish tasks
├── Program.cs               # Application entry point and queue processing logic
└── StorageQueueDemo.csproj  # Project file and package references
```

## Configuration

| Setting | Where | Description |
|---------|-------|-------------|
| `AZURE_STORAGE_CONNECTION_STRING` | Environment variable | Connection string of the Azure Storage account (Azure Portal → Storage account → **Access keys**) |
| `queueName` | Constant in `Program.cs` | Name of the queue to read from (`store-messages`) |

Set the environment variable before running the application.

PowerShell:

```powershell
$env:AZURE_STORAGE_CONNECTION_STRING = "<your-connection-string>"
```

Bash:

```bash
export AZURE_STORAGE_CONNECTION_STRING="<your-connection-string>"
```

If the variable is not set, the application prints a message and exits.

## How It Works

```csharp
string storageConnectionString = Environment.GetEnvironmentVariable("AZURE_STORAGE_CONNECTION_STRING");

QueueClient queueClient = new QueueClient(storageConnectionString, queueName);

if (queueClient.Exists())
{
    while (true)
    {
        QueueMessage[] retrievedMessages = queueClient.ReceiveMessages(1);

        if (retrievedMessages.Length > 0)
        {
            Console.WriteLine($"Dequeued message: '{retrievedMessages[0].MessageText}'");
            queueClient.DeleteMessage(retrievedMessages[0].MessageId, retrievedMessages[0].PopReceipt);
        }
    }
}
```

- **`QueueClient`** – the client used to access and manipulate the queue.
- **`Exists()`** – verifies the queue exists before processing starts.
- **`ReceiveMessages(1)`** – retrieves one message and hides it from other consumers for the default visibility timeout (30 seconds), so it must be processed within that time.
- **`DeleteMessage(MessageId, PopReceipt)`** – permanently removes the message once it has been processed.

## Prerequisites

- [.NET 10 SDK](https://dotnet.microsoft.com/download)
- An Azure Storage account with a queue named `store-messages`
- The `AZURE_STORAGE_CONNECTION_STRING` environment variable set (see [Configuration](#configuration))

## Running the Application

Restore, build, and run:

```bash
dotnet run
```

Or in VS Code, press **F5** to use the `.NET Core Launch (console)` configuration.

Add messages to the `store-messages` queue (e.g. from the Azure Portal), and the application will print them as they are dequeued:

```
Dequeued message: 'Hello from Azure Queue'
```

Stop the application with **Ctrl + C**.
