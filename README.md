# Build Your First Azure SignalR Service Application

In [ChatRoomLocal sample](https://github.com/aspnet/AzureSignalR-samples/tree/main/samples/ChatRoomLocal) you have learned how to use SignalR to build a chat room application. In that sample, the SignalR runtime (which manages the client connections and message routing) is running on your local machine. As the number of the clients increases, you'll eventually hit a limit on your machine and you'll need to scale up your machine to handle more clients. This is usually not an easy task. In this tutorial, you'll learn how to use Azure SignalR Service to offload the connection management part to the service so that you don't need to worry about the scaling problem.

## Provision a SignalR Service

First let's provision a SignalR service on Azure.

1. Open Azure portal, click "Create a resource" and search "SignalR Service".
2. Navigate to "SignalR Service" and click "Create".
3. Fill in basic information including resource name, resource group and location.
4. Click "Create", your SignalR service will be created in a few minutes.
5. Create another resource in a different location.

After your service is ready, go to the **Keys** page of your service instance and you'll get two connection strings that your application can use to connect to the service.

## Set Chat Rooms to Use Azure SignalR Service

Then, let's update the chat room sample to use the new service you just created.

Start the first Chatroom:

1. In [Startup.cs](Startup.cs), in `ConfigureServices()` method, update the connection strings of the `ServiceEndpoint`. You can copy the connection string from Azure portal.

    ```cs
    public void ConfigureServices(IServiceCollection services)
    {
        ...
        services.AddSignalR()
                .AddAzureSignalR(options =>
                    {
                        options.Endpoints = new ServiceEndpoint[2]
                        {
                            new ServiceEndpoint("conn1", EndpointType.Primary, "Primary"),
                            new ServiceEndpoint("conn2", EndpointType.Secondary, "Secondary"),
                        };
                    });
    }
    ```

Now run this app:

```
dotnet build
dotnet run
```

2. Run another chat room. Copy the `src` folder to a different place. And switch the `Primary` and `Secondary` endpoints.

    ```cs
    public void ConfigureServices(IServiceCollection services)
    {
        ...
        services.AddSignalR()
                .AddAzureSignalR(options =>
                    {
                        options.Endpoints = new ServiceEndpoint[2]
                        {
                            new ServiceEndpoint("conn2", EndpointType.Primary, "Primary"),
                            new ServiceEndpoint("conn1", EndpointType.Secondary, "Secondary"),
                        };
                    });
    }

 and update the `properties\launchSettings.json` to set it to a different port like `5001`:

    ```json
        "ChatRoom": {
          "commandName": "Project",
          "launchBrowser": true,
          "environmentVariables": {
            "ASPNETCORE_ENVIRONMENT": "Development"
          },
          "applicationUrl": "http://localhost:5001/"
        }
    ```

Then run with `dotnet run`.

3. Open browser and visit `http://localhost:5000` to begin a failover process. Be sure to open developer tools to trace the network.
4. Restart the Primary SignalR resource, where both Chat rooms are expected to see failure connecting to it, and the web page of the client session in browser should be keep running without any issue. The network trace shows it automatically connects to Seconary SignalR.
5. After restart operation completed, both Chat Rooms connect two both SignalR successfully again. And the web page continues working without no error.