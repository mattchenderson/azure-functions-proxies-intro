# Introduction to Azure Functions Proxies (preview)

This a demo script for introducing an audience to Azure Functions Proxies (currently in public preview). The demo assumes familiarity with Azure Functions. It covers the basics of starting with proxies, as well as some best practices and advanced configuration options.

>
> **Level:** 200  
> **Completion Time:** 20 minutes.
>

The demo is divided into 6 sections:
1. Creating a basic proxy
2. Best practices around environment management
3. Hosting static content
4. Passing along path segment parameters
5. Wildcard paths
6. Transforming request / response data

The end of each section shows what the underlying `proxies.json` file will look like once the stage is complete. You do not need to show this to the audience at each phase, but it can be useful for checking rehearsals and resetting the demo to a particular state.

## Prerequisites

This demo requires that you have the following:

* [Azure Subscription](https://azure.microsoft.com/free-trial)
* [Postman](https://www.getpostman.com/) or another HTTP client
* [Azure Storage Explorer](http://storageexplorer.com/)

## Setup

Before the demo, you should prepare with the following:

1. Create two function apps, preferably with names that designate one as the **frontend** and one as the **backend**.
2. On the app that is the backend, create a basic HttpTrigger that just returns a "Hello, world." Make note of the **Function URL**.
3. Create an app setting in the frontend application, *HELLO_HOST*, which is set with the hostname of the backend (e.g., _backend.azurewebsites.net_). You could optionally do this as part of the demo if audience would not be familiar with app settings.
4. Create a blob container and upload a simple HTML page to it. It could just say "Hello from Azure Blob Storage." Make the blob public for simplicity. Note the host and path.
5. Create an app setting in the frontend application, *BLOB_HOST*, which is set with the storage URL of the page you uploaded.
6. Make sure 

### Recommended demo machine state

Have Postman open and cleared out. Also have Storage explorer open, and navigate to the blob container you created.

It's helpful to have the following browser tabs open:
- The Azure portal, showing the backend function app
- The Azure portal, showing the frontend function app
- The root of your frontend function app

You can use slides from the sample deck ([PPT](https://functionsreadiness.blob.core.windows.net/functionsreadiness/Azure%20Functions%20Proxies.pptx?st=2017-07-11T07%3A01%3A00Z&se=2027-07-13T07%3A01%3A00Z&sp=rl&sv=2015-12-11&sr=b&sig=0KIWOZWUnV2AWNJAsm3gUi6xsgYRFyswI9pIZOdNXCg%3D), [PDF](https://functionsreadiness.blob.core.windows.net/functionsreadiness/Azure%20Functions%20Proxies.pdf?st=2017-07-11T07%3A01%3A00Z&se=2027-07-13T07%3A01%3A00Z&sp=rl&sv=2015-12-11&sr=b&sig=A3Z%2Bb4qcsVBn6TJ%2Fnzw2JHanT0wiWlM3vSPs37BwaK4%3D)) to help set the context for the demo.

## Part 1 - Creating a basic proxy

This will walk you through creating your first proxy and demonstrating that it works.

### Steps to Complete

1. Navigate to the frontend function app in the portal.
2. Click **New proxy**. Provide a **Name** for your proxy, such as "HelloProxy".
3. Set the **Route template** to `/hello`.
4. In **Backend URL**, enter the URL for the HttpTrigger you created on the backend app. 
5. Click **Create**.
6. Copy the **Proxy URL**, and then use Postman or the HTTP client of your choosing, and send a GET request that URL.

### Success Criteria

You should see your "Hello, world" from the backend returned.

### Completed state

The underlying `proxies.json` will look like the following when this stage is complete:
```json
{
    "proxies": {
        "HelloProxy": {
            "matchCondition": {
                "route": "/hello"
            },
            "backendUri": "https://backend.azurewebsites.net/api/HttpTriggerCSharp1"
        }
    }
}
```

## Part 2 - Best practices around environment management

In this step, we update our simple proxy to leverage application settings. This is important when using multiple environments - you want to make sure that the requests are being proxied to the right site.

### Steps to Complete

1. Navigate to the frontend function app in the portal.
2. Select the proxy you created earlier.
3. Replace the hostname (e.g., _backend.azurewebsites.net_) in the "Backend URL" field with `%HELLO_HOST%`. This will reference the app setting you created during setup. Click **Save**.
4. Navigate to your app settings list and show the setting that you are referencing.
5. Repeat the call using your HTTP client to show that the proxy still works.

### Success Criteria

Again, you should see your "hello, world" from the backend returned. Be sure to explain that this is the preferred way of configuring a production proxy.

### Completed state

The underlying `proxies.json` will look like the following when this stage is complete:
```json
{
    "proxies": {
        "HelloProxy": {
            "matchCondition": {
                "route": "/hello"
            },
            "backendUri": "https://%HELLO_HOST%/api/HttpTriggerCSharp1"
        }
    }
}
```

## Part 3 - Hosting static content

In this step, we take over the root of our function app to serve a simple HTML page. This could be used to host a single page application (SPA) which calls back to APIs served by the function app. Those APIs could be other proxies, with backend fuctions.

Note that this allows you to access a blob using a custom domain and SSL. The Storage team is working on enabling this natively, but this provides a convenient workaround until that is available.

Note also that taking over the root of the site will interfere with other functions on the site. This is currently treated as a bug. You may choose to omit this section if you think it will confuse users.

### Steps to Complete

1. Navigate to the frontend function app in the portal.
2. Create a new proxy as before. This time, give it the name "root" or similar. Set the route template to "/". For the backend URL, use "https://%BLOB_HOST%/", appended with the file path you configured during setup.

    For example, if during setup you had put the file `index.html` in the "static" container, you would set your backend URL to `https://%BLOB_HOST%/static/index.html`

3. Optionally show the app setting. The audience may have seen it in the previous step.
4. Use Storage Explorer to show the file that you uploaded.
3. Navigate to a previously loaded tab for the root of your frontend (e.g., _frontend.azurewebsites.net_).
4. Reload the tab to show the new page from blob storage being hosted.

### Success Criteria

You should see the page that you uploaded to blob storage during setup.

### Completed state

The underlying `proxies.json` will look like the following when this stage is complete:
```json
{
    "proxies": {
        "HelloProxy": {
            "matchCondition": {
                "route": "/hello"
            },
            "backendUri": "https://%HELLO_HOST%/api/HttpTriggerCSharp1"
        },
        "root": {
            "matchCondition": {
                "route": "/"
            },
            "backendUri": "https://%BLOB_HOST%/static/index.html"
        }
    }
}
```

## Part 4 - Passing along path segment parameters

This section shows how to get parameters from the request that can be used to dynamically adjust the backend call. This section also demonstrates that you can proxy to something other than a function. 

### Steps to Complete

1. Navigate to the frontend function app in the portal.
2. Create a new proxy as before. This time, give it the name "graph" or similar. Set the route template to `/graph/{object}`, which includes the parameter `{object}`. For the backend URL, use `https://jsonplaceholder.typicode.com/{object}`. The `{object}` section in the backend URL will be populated based on what was provided in the original request.
3. Optionally, you may want to show the site [https://jsonplaceholder.typicode.com/](https://jsonplaceholder.typicode.com/) to give the audience an idea of what you are pinging.
4. Copy the proxy URL and test the endpoint using another GET request from your HTTP client. You will need to replace "{object}" with a valid input, such as "users".

    An example call would be `GET https://frontend.azurewebsites.net/graph/users`.

5. Repeat the test using "posts" as the object.

### Success Criteria

You should see a JSON array of sample data resembling a social network graph.

### Completed state

The underlying `proxies.json` will look like the following when this stage is complete:
```json
{
    "proxies": {
        "HelloProxy": {
            "matchCondition": {
                "route": "/hello"
            },
            "backendUri": "https://%HELLO_HOST%/api/HttpTriggerCSharp1"
        },
        "root": {
            "matchCondition": {
                "route": "/"
            },
            "backendUri": "https://%BLOB_HOST%/static/index.html"
        },
        "graph": {
            "matchCondition": {
                "route": "/graph/{object}"
            },
            "backendUri": "https://jsonplaceholder.typicode.com/{object}"
        }
    }
}
```

## Part 5 - Wildcard paths

This section shows how to get use wildcard parameters from the request. This is especially useful when a backend serves all of the APIs for a given subpath.

### Steps to Complete

1. Navigate to the frontend function app in the portal. Select the graph proxy you just created.
2. Edit the proxy's route template to be `/graph/{*restOfPath}`.
3. Copy the proxy URL and test the endpoint using another GET request from your HTTP client. You will need to replace "{*restOfPath}" with a valid input, such as "users/1". Note that this is composed of two path segments.

    An example call would be `GET https://frontend.azurewebsites.net/graph/users/1`.

### Success Criteria

You should see a JSON object of sample data resembling a single user.

### Completed state

The underlying `proxies.json` will look like the following when this stage is complete:
```json
{
    "proxies": {
        "HelloProxy": {
            "matchCondition": {
                "route": "/hello"
            },
            "backendUri": "https://%HELLO_HOST%/api/HttpTriggerCSharp1"
        },
        "root": {
            "matchCondition": {
                "route": "/"
            },
            "backendUri": "https://%BLOB_HOST%/static/index.html"
        },
        "graph": {
            "matchCondition": {
                "route": "/graph/{*restOfPath}"
            },
            "backendUri": "https://jsonplaceholder.typicode.com/{restOfPath}"
        }
    }
}
```


## Part 6 - Transforming request / response data

Azure Functions Proxies can modify the request to and response from the backend. This is particularly useful when securing the backend or integrating a legacy API. In this demo, we use a response transform to create a mock API. This keeps the scenario simple and shows a practical use of the feature. Other transforms can simply be talking points.

Because this feature is not yet supported by the UX, you will need to manually modify the proxy configuration. The script includes instructions for this.

### Steps to Complete

Transitioning to manual configuration:
1. Navigate to the frontend function app in the portal.
2. Open **Function app settings** and choose **Go to App Service Editor**.
3. In the left-hand navigation, select `proxies.json`.
4. You can explain the file structure at this point. See the [Proxies documentation](https://docs.microsoft.com/azure/azure-functions/functions-proxies#deployment-methods) for details.

Adding a data transform:
1. Locate the proxy object for the original hello world endpoint. Within the proxy object, add a "requestOverrides" object that includes a property to change the content type to "text-plain". Your proxy object should look like in the following:
```json
{
    "matchCondition": {
        "route": "/hello"
    },
    "backendUri": "https://backend.azurewebsites.net/api/HttpTriggerCSharp1",
    "responseOverrides": {
        "response.header.Content-Type": "text/plain"
    }
}
```

2. Test the endpoint using your HTTP client again to show that the response is no longer JSON, but instead just plain text.

3. Next, modify the message by changing the response body. To do this, add a "response.body" property to "responseOverrides", giving it any value that is different from the original function. Your proxy object should look like the following:
```json
{
    "matchCondition": {
        "route": "/hello"
    },
    "backendUri": "https://backend.azurewebsites.net/api/HttpTriggerCSharp1",
    "responseOverrides": {
        "response.header.Content-Type": "text/plain",
        "response.body": "Hello from response transform"
    }
}
```

4. Test the endpoint using your HTTP client again to show the new text response.
5. Back in the editor, remove the "backendUri" property. Your proxy object should look like the following:
```json
{
    "matchCondition": {
        "route": "/hello"
    },
    "responseOverrides": {
        "response.header.Content-Type": "text/plain",
        "response.body": "Hello from response transform"
    }
}
```
6. Test the endpoint again using your HTTP client, which should again show the modified response. However, no backend API was needed, which means this feature can be used to create mock endpoints.

### Success Criteria

You should see the modified message come back. If using the example above, we would see "Hello from the repsonse transform" as plain text.

It may be worth discussing the use of app settings again for request transforms. This is a good way of handling backend API keys without putting them in the config file.

### Completed state

The underlying `proxies.json` will look like the following when this stage is complete:
```json
{
    "proxies": {
        "HelloProxy": {
            "matchCondition": {
                "route": "/hello"
            },
            "responseOverrides": {
                "response.header.Content-Type": "text/plain",
                "response.body": "Hello from response transform"
            }
        },
        "root": {
            "matchCondition": {
                "route": "/"
            },
            "backendUri": "https://%BLOB_HOST%/static/index.html"
        },
        "graph": {
            "matchCondition": {
                "route": "/graph/{*restOfPath}"
            },
            "backendUri": "https://jsonplaceholder.typicode.com/{restOfPath}"
        }
    }
}
```



## Conclusion

This demo walks users through several capabilities of Azure Functions Proxies, from creating a basic "Hello, world" call to mocking APIs and transforming backend requests. The demo also includes some best practice guidance, such as leveraging app settings to make environment and secrets management easier. Be sure to emphasize these points.

It's also important to emphasize the problem that Azure Functions Proxies solves. This allows develoeprs to build APIs however they wish, but expose only a single host to client applications. It is especially powerful for customers interested in a microservice architecture.

### Next Steps

- Encourage folks to try out the Azure Functions Proxies public preview.
- Point attendees to the [documentation](https://docs.microsoft.com/azure/azure-functions/functions-proxies) for Proxies if they want to learn more.
