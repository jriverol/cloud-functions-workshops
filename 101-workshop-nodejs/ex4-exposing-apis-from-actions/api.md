<!--
#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
-->

#  API Management of web actions

Let's now explore how web actions can be turned into an API using the [API Gateway](https://cloud.ibm.com/docs/openwhisk?topic=cloud-functions-apigateway).

At first glance, APIs do not seem to be very different from web actions since ICF already generated an HTTP endpoint. However, the key difference is that with an actual API, you are going through an API Gateway service and are able to take advantage of a variety of features outside the scope of this course including [rate limiting, authentication](https://cloud.ibm.com/docs/api-gateway?topic=api-gateway-create_api), and [custom domains](https://cloud.ibm.com/docs/api-gateway?topic=api-gateway-custom_endpoint) for your API.

Again, all these API Gateway features are provided with no changes to your action. Trying to do these things in your function code would likely be impossible and negate the many benefits of serverless.

## API create syntax

To create an API, you will use the following command syntax:

```bash
ibmcloud fn api create <BASE_PATH> <API_NAME> <HTTP_METHOD> <ACTION_NAME>
```

{% hint style="warning" %}
The example above merely shows the syntax used to create an API endpoint and should **not** be run.
{% endhint %}

## Create API endpoints

Note that all actions used in an API must be web actions. If they are not, you can run `ibmcloud fn action update <action name> --web true` prior to running the commands below.

### Greeting endpoint

1. Run the following command to create a simple HTTP `GET` endpoint for the `hello` action:

    ```bash
    ibmcloud fn api create /myapi /greeting get hello
    ```

    ```bash
    ok: created API /myapi/greeting GET for action /_/hello
    https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/d9903f40439f1a268b7dcbac42a389cdde605f3f3bef57f69789be6df438361e/myapi/greeting
    ```

    In this example, we used `/myapi` as the API base path and `greeting` as the API endpoint name. Having a base path name is useful for grouping APIs together in a logical way within ICF.

2. Verify the API was created:

    ```bash
    ibmcloud fn api list
    ```

    ```bash
    ok: APIs
    Action                                     Verb  API Name  URL
    /2ca6a304-a717-4486-ae33-1ba6be11a393/he    get    /myapi  https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/d9903f40439f1a268b7dcbac42a389cdde605f3f3bef57f69789be6df438361e/myapi/greeting
    ```

3. Now let’s invoke that `greeting` API via curl:

    ```bash
    curl "https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/d9903f40439f1a268b7dcbac42a389cdde605f3f3bef57f69789be6df438361e/myapi/greeting"
    ```

    ```json
    {
        "greeting": "Hello, undefined from Rivendell"
    }
    ```

You have now created and invoked your first IBM Cloud Functions (ICF) API endpoint!

### Other response types

If you do not return the default , we explicitly set the appropriate response type with the `--response-type <TYPE>` flag. Valid type values include `http`, `json`, `text`, and `svg`.

Let's demonstrate how to create an endpoint for your redirect web action.

1. Create the endpoint for the HTTP endpoint:

    ```bash
    ibmcloud fn api create /myapi /redirect get redirect --response-type http
    ```

    ```bash
    ok: created API /myapi/redirect GET for action /_/redirect
    https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/d9903f40439f1a268b7dcbac42a389cdde605f3f3bef57f69789be6df438361e/myapi/redirect
    ```

2. Check to make sure that the redirect works by copying and pasting the URL returned into your browser.

You’ve now successfully created an endpoint for an HTTP response type. The other response types listed above follow the same syntax. For the sake of brevity, you will not be shown the creation of the other response types or web actions created in the previous section. However, if you would like to go through the exercise of creating those API endpoints, you can walk through the optional exercise [Other APIs](other_apis.md).

## Use OpenAPI Specification

As you can begin to tell, as the number of API endpoints increases, documenting and managing them becomes increasingly difficult. One solution to this is to use the [OpenAPI Specification](https://swagger.io/specification/). This has a plethora of tools around for documenting, creating stub projects, and more, in a variety of languages. And it is supported by ICF!

1. Let's take stock of our APIs by listing out all the endpoints:

    ```bash
    ibmcloud fn api list
    ```

    ```bash
    ok: APIs
    Action                                      Verb  API Name  URL
    /2ca6a304-a717-4486-ae33-1ba6be11a393/he     get    /myapi  https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/d9903f40439f1a268b7dcbac42a389cdde605f3f3bef57f69789be6df438361e/myapi/greeting
    /2ca6a304-a717-4486-ae33-1ba6be11a393/re     get    /myapi  https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/d9903f40439f1a268b7dcbac42a389cdde605f3f3bef57f69789be6df438361e/myapi/redirect
    ```

{% hint style="info" %}
Note that your listing may have more API endpoints if you tried the optional exercises.
{% endhint %}

2. View the OpenAPI Specification for `myapi` using the following command:

    ```bash
    ibmcloud fn api get /myapi
    ```

    You should see a long JSON document that starts something like:

    ```json
    {
        "swagger": "2.0",
        "basePath": "/myapi",
        "info": {
            "title": "/myapi",
            "version": "1.0.0"
        },
        "paths": {
        "/greeting": {
            "get": {
                "operationId": "getGreeting",
                "responses": {
                    "default": {
                        "description": "Default response"
                    }
                },
                "x-openwhisk": {
                    "action": "hello",
                    "namespace": "2ca6a304-a717-4486-ae33-1ba6be11a393",
                    "package": "",
                    "url": "https://us-south.functions.cloud.ibm.com/api/v1/web/2ca6a304-a717-4486-ae33-1ba6be11a393/default/hello.json"
                }
            }
        },
        "/redirect": {
            ...
        },
        ...
    }
    ```

3. You will want to write that to a file with:

    ```bash
    ibmcloud fn api get /myapi > myapi.json
    ```

    Now you can delete the existing API you created and restore it using this document.

4. Delete the existing API:

    ```bash
    ibmcloud fn api delete /myapi
    ```

    ```bash
    ok: deleted API /myapi
    ```

5. Check that the API endpoints are gone:

    ```bash
    ibmcloud fn api list
    ```

    ```bash
    ok: APIs
    Action                            Verb             API Name  URL
    ```

    The list should be empty.

6. Restore the endpoints from the OpenAPI Specification:

    ```bash
    ibmcloud fn api create -c myapi.json
    ```

    ```bash
    ok: created API /myapi/greeting get for action /3cc8e80c-1e29-4d99-b530-a89bf13fee32/hello
    https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/d9903f40439f1a268b7dcbac42a389cdde605f3f3bef57f69789be6df438361e/myapi/greeting
    ok: created API /myapi/redirect get for action /3cc8e80c-1e29-4d99-b530-a89bf13fee32/redirect
    https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/d9903f40439f1a268b7dcbac42a389cdde605f3f3bef57f69789be6df438361e/myapi/redirect
    ```

7. You can now see that the endpoints are restored:

    ```bash
    ibmcloud fn api list
    ```

    ```bash
    ok: APIs
    Action                                      Verb  API Name  URL
    /3cc8e80c-1e29-4d99-b530-a89bf13fee32/he     get    /myapi  https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/d9903f40439f1a268b7dcbac42a389cdde605f3f3bef57f69789be6df438361e/myapi/greeting
    /3cc8e80c-1e29-4d99-b530-a89bf13fee32/re     get    /myapi  https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/d9903f40439f1a268b7dcbac42a389cdde605f3f3bef57f69789be6df438361e/myapi/redirect
    ```

This OpenAPI Specification can now be stored in your code repository and used to update endpoints, documentation, or event generate stub code!

{% hint style="success" %}
Congratulations on successfully creating serverless APIs! Exposing and managing serverless APIs takes minimal effort using ICF and allows you full access control and use of the OpenAPI specification!
{% endhint %}
