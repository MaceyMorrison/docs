# Job Quickstart Guide

### Getting Started

Veritone Job API allows you to easily integrate cognitive functionality such as object detection, language translation, and voice transcription with just a few lines of code. The Job API is a set of calls that takes you through the workflow for performing cognitive task processing — from file ingestion to engine processing and retrieving output results.

Veritone's API is built around the GraphQL paradigm to provide a more efficient way to deliver data with greater flexibility than a traditional REST approach. GraphQL is a [query language](http://graphql.org/learn/queries/) that operates over a single endpoint using conventional HTTP requests and returning JSON responses. The structure not only lets you call multiple nested resources in a single query, it also allows you to define requests so that only requested data is sent back.

This quickstart guide provides resources, detailed documentation, and example requests and responses to help get your integration up and running to perform the following operations:

*  **Create a TDO (recording container) with an asset**
*  **Create a job with specific tasks**
*  **Check the status of a job**
*  **Retrieve job results**
*  **Delete a TDO/recording container**

We designed this quickstart to be user friendly and example filled, but if you have any questions, please don’t hesitate to reach out to our [Developer Support Team](mailto:devsupport@veritone.com) for help.

### **Base URL**

Veritone uses a single endpoint for accessing the API. All calls to the API are *POST* requests and are served over *http* with *application/json* encoded bodies. The base URL varies based on the geographic region where the services will run. When configuring your integration, choose the base URL from the list below that supports your geographic location.

<table>
  <tr>
    <td><b>Region</b></td>
    <td><b>Base URL</b></td>
  </tr>
  <tr>
    <td>United States</td>
    <td>[https://api.veritone.com/v3/graphql](https://api.veritone.com/v3/graphql)</td>
  </tr>
  <tr>
    <td>Europe</td>
    <td>[https://api.uk.veritone.com/v3/graphql](https://api.uk.veritone.com/v3/graphql)</td>
  </tr>
</table>
  
*Note:* The above base URLs are provided for use within SaaS environments. Applications using an on-prem deployment access the API via an endpoint that's custom configured to the private network.

### **Making Sample Requests**

To make it easier to explore, write, and test the API, we set up [GraphiQL](https://api.veritone.com/v3/graphiql) — an interactive playground that gives you a code editor with autocomplete, validation, and syntax error highlighting features. Use the GraphiQL interface to construct and execute queries, experiment with different schema modifications, and browse documentation. In addition, GraphiQL bakes authorization right into the schema and automatically passes the `Authentication` header with a valid token when you’re logged into the Veritone system.

Veritone’s GraphiQL interface is the recommended method for ad hoc API requests, but calls can be made using any HTTP client. All requests must be HTTP POST to the base URL designated for your geographic region with the `query` parameter and `application/json` encoded bodies. In addition, requests must be authenticated using an API Token. Pass the token in your request using the *Authorization* header with a value `Bearer token`. If you’re using a raw HTTP client, the query body contents must be sent in a string with all quotes escaped.

The sample requests provided in this documentation are structured for use in our [GraphiQL interface](https://api.veritone.com/v3/graphiql), but we’ve also included the basic cURL structure for your reference below. Please note that the examples shown throughout this guide do not use client information and are not language specific. For fields that require account-specific data (such as a Container ID), replace the value with your own. In addition, the sample requests shown are not all-inclusive — they highlight the minimum requirements and relevant information. Additional attributes for each request can be found in our [GraphQL docs](https://api.veritone.com/v3/graphqldocs/).

#### Basic cURL Structure for Requests

```bash
curl -X POST \
  https://api.veritone.com/v3/graphql \
  -H 'authorization: Bearer 31gcf6:2e76022093e64732b4c48f202234394328abcf72d50e4981b8043a19e8d9baac' \
  -H 'content-type: application/json' \
  -d '{"query": "mutation { createTDO( input: { startDateTime: 1507128535, stopDateTime: 1507128542, name: \"My New Video\", description: \"The latest video in the series\" }) { id,  status } }" }'
```

### **Authentication**

Veritone Job API uses bearer token authentication for requests. To authenticate your calls, provide a valid API Token in the `Authentication` header of the request with the value `Bearer token`. Requests made without this header or with an invalid token will return an error code.

An API Token can be generated in the Veritone Admin App by your Organization Administrator. If your organization does not use the Admin App, please contact your Veritone Account Manager for assistance. 

**To generate an API Token:**

1. Log into the Veritone Platform and select **Admin** from the *App Picker* drop-down. The *Admin App* opens.
2. Click the **API Keys** tile. The *API Key* page opens.

![Get API Token](Get-API-Token-1.png)

3. Click **New API** Key. The *New API Key* window opens.

![Get API Token](Get-API-Token-2.png)

4. Enter a token name and select the permissions needed for the token to perform the required API tasks. Click **Generate Token** to save. The *Token Generated *window opens.
5. Copy your token and click **Close** when finished. 

*Note:* Once the *Token Generated* window closes, the token code no longer displays and it cannot be viewed again.

## Create a TDO (Recording Container) with an Asset

In Veritone, audio and video files are uploaded to a container called a Temporal Data Object (TDO). The first step in the Job workflow is to create a TDO and upload a video or audio file to it by calling the `createTDOWithAsset` mutation. Successful calls return a unique TDO ID and Asset ID that are used to create cognitive processing jobs and tasks. Calls that are unsuccessful will result in an error.

There are two available options for uploading files, which are described below. Before getting started, we recommend reviewing our [Uploading Large Files](apis/tutorials/uploading-large-files) tutorial to learn about the size limitations for queries and files.

### Option 1: Upload a Local System File 
----------

Upload a file from your local system by doing a multipart/form-data HTTP request. This option structures requests in multiple parts that represent different characteristics of the file. In each request, form field data containing information about the file is sent along with the `Authentication` and `Content-Type` headers. When structuring your request, be sure to set the `Content-Type` header to `multipart/form-data` and use the form-data keys `query`, `file`, and `filename` in the body. Currently, GraphiQL does not support multipart/form requests, so a different HTTP client must be used for making sample calls.
 

#### Option 1 Request Payload: Create a TDO with an Asset
```graphql
 -H content-type:  => A header that specifies the content type. Enter "multipart/form-data" as the value.(required)
 -F filename       => The name of the file to upload. The value must match the name of the saved file. (required)      
 -F file           => The path of the file to upload. (required)                                                          
 -F query=mutation {
-----------request fields-----------
  createTDOWithAsset(input:{  => The mutation type and input variable. (required)
    startDateTime: "string"   => The starting date and time of the file to be uploaded in [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) format. (required)
    stopDateTime: "string"    => The ending date and time in [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) format. The value is calculated by adding the length of the file to the startDateTime value. If a value is not provided, a 15-minute default value will be applied. (optional)
    contentType: "string"     => The MIME type of the file being uploaded (e.g., "audio/mp3"). If a value is not provided, the default value "video/mp4" will be applied. (optional)
    assetType: "string"       => A label that classifies the file to be uploaded, such as "transcript," "media," or "text." If a value is not specified, the default value "media" will be applied. (optional)
    addToIndex:               => A Boolean that adds the uploaded file to the search index when set to true. (optional and recommended) 
  }){
-----------return fields-----------
    id              => The unique ID associated with the TDO/container, provided by the server. (required)
    status          => The status of the request’s progress. (required)
    assets{         => The Assets object parameter that contains the TDO's assets. (required)
      records {     => The Records object parameter that contains data specific to individual assets. (required)
        id          => The unique ID of the new asset, provided by the server. (required)
        type        => A label that classifies the asset. This reflects the value specified in the request or the default value "media" if no request value was set. (required)
        contentType => The MIME type of the asset (e.g., "audio/mp3"). (required)
        signedUri   => The secure URI of the asset. (required)
      }
    }    
  }
}
```

#### Option 1 Sample Request: Create a TDO with an Asset
```bash
curl -X POST \
  https://api.veritone.com/v3/graphql \
  -H 'authorization: Bearer 31gcf6:2e76022093e64732b4c48f202234394328abcf72d50e4981b8043a19e8d9baac' \
  -H 'content-type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW' \
  -F 'filename=rescued puppies road trip.mp4' \
  -F 'file=@/Users/bobjones/Downloads/rescued puppies road trip.mp4' \
  -F 'query=mutation {
  createTDOWithAsset(
    input: {
      startDateTime: 1507128535
      stopDateTime: 1507128542
      contentType: "video/mp4"
      assetType: "media"
      addToIndex: true
    } ) {
    id
    status
    assets{
      records {
        id
        type
        contentType
        signedUri
      }
    }    
  }
}'
```

#### Option 1 Sample Response: Create a TDO with an Asset
```json
{
  "data": {
    "createTDOWithAsset": {
      "id": "44512341",
      "status": "recorded",
      "assets": {
        "records": [
          {
            "id": "7acfab47-f648-4cfc-9042-74b4cafb1605",
            "type": "media",
            "contentType": "video/mp4",
            "signedUri": "https://inspirent.s3.amazonaws.com/assets/44512341/fd3bd480-2c23-4107-a283-3a2b54d6e512.mp4?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=ASIAJ6G7HKQBDKQ35MFQ%2F20180713%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20180713T155226Z&X-Amz-Expires=3600&X-Amz-Security-Token=FQoDYXdzEF0aDAJG7F%2FR22OpH5fwNCK3A10D6Sx3CYJ6G7kk0ykfjR59f4cLoDNE87BbY2BE0L6ivCD1GN2ZQP4XV8Im6NxQogv5aNqsFnolF5kY9fT%2BSRQwOPNkQA%2FGqPiFJMHkjwvz5zl8SvkW%2FvfAlhIEFEcVKR36jQxNZcrHi10TwMGj3w88bs1lSGz2NOIKZwKOrFSHioxkznZhHLMCzYImVQNmoX2g8qA%2BZVUlsuBh4QvlL3pg7ww%2BXpPaQ1U6vNmpIZEuZIw2TBG%2BmZh9NwNWqFuRtvfxX%2FzoKq71wmI%2F%2F4CGdQ9t%2BUbbWLvT0iVdv7fvoZPbRr6iGvfBsQ2SAD7tXM2iqz2XO%2B8cqKRbeBZ%2BLEzN3K2v2LIGOaHp1%2BDpyYSjo0WcHAVx2KWkq%2BrdvfLH%2Bl%2FANyRMOQdEZrlbkS0ZMrtLIQU4b74ylkslkwhbpP6K%2FxvdDSi7TPQH4xRNdz2OCiELHWKVs%2FHc5Gx1BUrg%2F4C%2BeXTIWFHY2qrqgOhnfW3wZq4p4J%2BZCNbORIOnePb21ZIQtUVQWOEuvmwKKpsz4eDDV%2FvZduq%2Bhkwtwhcr8AimWDbcJyoL9jRwrzqwI9ri12RP7O%2FwYT410jIoxJyi2gU%3D&X-Amz-Signature=e75c68ab486a356da011ba413c88ebb7db3eeacbfa1ebd5b68d469211e736e09&X-Amz-SignedHeaders=host"
          }
        ]
      }
    }
  }
}
```

### Option 2: Pass the file URL 
----------

If the file you want to upload is publicly available online, you can pass the HTTP or HTTPS URL in the request instead of uploading the actual file data. When uploading a file from a URL, include the `uri` parameter in the request with the file's raw URL set as the value. 

#### Option 2 Request Payload: Create a TDO with an Asset
```graphql
mutation {
-----------request fields-----------
  createTDOWithAsset(input:{  => The mutation type and input variable. (required)
    startDateTime: "string"   => The starting date and time of the file to be uploaded in [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) format. (required)
    stopDateTime: "string"    => The ending date and time in [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) format. The value is calculated by adding the length of the file to the startDateTime value. If a value is not provided, a 15-minute default value will be applied. (optional)
    contentType: "string"     => The MIME type of the file being uploaded (e.g., "audio/mp3"). If a value is not provided, the default value "video/mp4" will be applied. (optional)
    assetType: "string"       => A label that classifies the file to be uploaded, such as "transcript," "media," or "text." If a value is not specified, the default value "media" will be applied. (optional)
    addToIndex:               => A Boolean that adds the uploaded file to the search index when set to true. (optional and recommended)
    uri: "string"             => The publicly available URL to the file. (required)
  }){
-----------return fields-----------
    id              => The unique ID associated with the TDO/container, provided by the server. (required)
    status          => The status of the request’s progress. (required)
    assets{         => The Assets object parameter that contains the TDO's assets. (required)
      records {     => The Records object parameter that contains data specific to individual assets. (required)
        id          => The unique ID of the new asset, provided by the server. (required)
        type        => A label that classifies the asset. This is the value specified in the request or the default value "media" if no request value was set. (required)
        contentType => The MIME type of the asset (e.g., "audio/mp3"). (required)
        signedUri   => The secure URI of the asset. (required)
      }
    }    
  }
}
```

#### Option 2 Sample Request: Create a TDO with an Asset
```graphql
mutation {
  createTDOWithAsset(
    input:{
      startDateTime: 1507128535
      stopDateTime: 1507128542
      contentType: "video/mp4"
      assetType: "media"
      addToIndex: true
      uri: "https://www.youtube.com/watch?v=LUzGYV-_qkQ"
    }
  ) {
    id
    status
    assets{
      records {
        id
        type
        contentType
        signedUri
      }
    }
  }
}
```

#### Option 2 Sample Response: Create a TDO with an Asset
```json
{
  "data": {
    "createTDOWithAsset": {
      "id": "44512341",
      "status": "recorded",
      "assets": {
        "records": [
          {
            "id": "7acfab47-f648-4cfc-9042-74b4cafb1605",
            "type": "media",
            "contentType": "video/mp4",
            "signedUri": "https://www.youtube.com/watch?v=LUzGYV-_qkQ"
          }
        ]
      }
    }
  }
}
```
## Create a Job with Cognitive Processing Tasks

Once the file is uploaded and saved as an asset, you’ll create job to perform cognitive processing against it. A job is an object that contains a set of tasks. Each task is an operation (such as transcription or object detection) performed by an engine on an input asset. Within a job tasks can be chained together, where the output asset of one task is passed as the input asset for the next task. This allows any number of tasks to be added to a job and processed in parallel and asynchronously.

A call to the `Create Job` mutation creates a new job with one or more tasks to be processed, according to the Engine ID(s) specified in the request body. Depending on the engine or the type of task, additional payload parameters that define options about how the service is to be performed (e.g., translation language) may be required. Each job request must also include a `Transcoding` task to convert the file to a supported format for processing. In addition, the `Insert Into Index` task is required to ensure output data is indexed and made searchable in Veritone.

*Important Note about Translation:*<br>
Because translation engines use text to translate one language to another, an audio or video file must be transcribed before it can be translated. A transcription task can be included in the same job with translation, or it can occur in a separate job using the same `TDO ID` and the transcription asset as the input file.

#### Engine IDs
Calls to the `createJob` mutation pass the ID of the engine(s) that will carry out task processing and any additional required payload parameters in the body of the request. Depending on your needs, you may choose to offer all cognitive engines in your integration or limit the selection to specific categories. To get a full list of engines available for your organization with Engine IDs and additional required fields, [log into Veritone](https://www.veritone.com/) and [click here](https://api.veritone.com/v3/graphiql?query=%7B%0A%20%20engines%28limit%3A%201000%29%20%7B%0A%20%20%20%20count%0A%20%20%20%20records%20%7B%0A%20%20%20%20%20%20id%0A%20%20%20%20%20%20category%20%7B%0A%20%20%20%20%20%20%20%20id%0A%20%20%20%20%20%20%20%20name%0A%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20fields%20%7B%0A%20%20%20%20%20%20%20%20name%0A%20%20%20%20%20%20%20%20type%0A%20%20%20%20%20%20%20%20max%0A%20%20%20%20%20%20%20%20min%0A%20%20%20%20%20%20%20%20step%0A%20%20%20%20%20%20%20%20info%0A%20%20%20%20%20%20%20%20label%0A%20%20%20%20%20%20%20%20defaultValue%0A%20%20%20%20%20%20%20%20options%20%7B%0A%20%20%20%20%20%20%20%20%20%20key%0A%20%20%20%20%20%20%20%20%20%20value%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%20%20%7D%0A%7D) to run a pre-configured query in GraphiQL. If you will offer a more limited set of engines, you can retrieve engine information by category. First, [run this query](https://api.veritone.com/v3/graphiql?query=query%20%7B%0A%20%20engineCategories%20%7B%0A%20%20%20%20records%20%7B%0A%20%20%20%20%20%20id%20%0A%20%20%20%20%20%20name%0A%20%20%20%20%20%20description%0A%20%20%20%20%7D%0A%20%20%7D%0A%7D%0A%0A%0A) to get a list of available engine categories. Then, enter the selected category name as the value for the `category` field [in this query](https://api.veritone.com/v3/graphiql?query=%7B%0A%20%20engines%28category%3A%20%22transcription%22%2C%20limit%3A%201000%29%20%7B%0A%20%20%20%20count%0A%20%20%20%20records%20%7B%0A%20%20%20%20%20%20id%0A%20%20%20%20%20%20category%20%7B%0A%20%20%20%20%20%20%20%20id%0A%20%20%20%20%20%20%20%20name%0A%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20fields%20%7B%0A%20%20%20%20%20%20%20%20name%0A%20%20%20%20%20%20%20%20type%0A%20%20%20%20%20%20%20%20max%0A%20%20%20%20%20%20%20%20min%0A%20%20%20%20%20%20%20%20step%0A%20%20%20%20%20%20%20%20info%0A%20%20%20%20%20%20%20%20label%0A%20%20%20%20%20%20%20%20defaultValue%0A%20%20%20%20%20%20%20%20options%20%7B%0A%20%20%20%20%20%20%20%20%20%20key%0A%20%20%20%20%20%20%20%20%20%20value%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%20%20%7D%0A%7D) to get back a list of that category’s available engines and details.

If you would like to run the above queries using a client other than GraphiQL, we’ve defined the schemas for you in Appendix 1 of this guide. To see our full API docs, [click here](https://api.veritone.com/v3/graphqldocs/).

#### Request Payload: Create a Job
```graphql
mutation{
-------request fields-----------
  createJob(input: {   => The Create Job mutation type and input variable. (required)
    targetId: "string" => The TDO/Container ID returned in the createAssetWithTDO response. (required)
    tasks: [{ object   => The tasks object parameter with data specific to the job’s tasks. (required)
      engineId: "string"    => The ID of the engine that will process the task. (required)
        payload: {          => An object with required data to upload with an engine task. See the Engine Task Payloads table later in this section for options. (required by some engines)
           target: "string" => The task payload option and value. (required when using the "payload" parameter)    
        },
      {  
      engineId: "fc004413-89f0-132a-60b2-b94522fb7e66" => The task engine ID that transcodes the file into a supported format for processing. Video files are transcoded to MP4 and audio files to MP3. Use the exact value shown. (required)                                             
        payload: {           => The payload object for transcoding. (required)
          setAsPrimary: true => A Boolean set to true to ensure the file is transcoded prior to running any other task in the job. (required)
        },
      {
      engineId: "c2aaa6d7-14fa-f840-f77e-4d2c0b857fa8" => A task engine ID that stores output data in the search index. Use the exact value shown. This engine does not require a payload. (required)     
      }
  }){
-------return fields------------
    id            => The unique ID of the job, provided by the server. (required)
    targetId      => The TDO/Container ID specified in the request. (required)
    status        => The status of the job as a whole. Possible Values are “accepted” (the job is valid and has been accepted for processing), “running” (the job is running), “failed” (at least one task failed and job execution was terminated), and “complete” (all tasks in the job finished normally). (required)
    tasks {       => The "tasks" object parameter with data specific to the job’s tasks. (required)
      records {   => The "records" object parameter with data specific to individual tasks. (required)
        id        => The unique ID of a task, provided by the server. (required)
        engineId  => The unique ID of the engine that processed the task. (required)
        order     => The sequential order that the tasks were processed, starting at 0. (required)
        payload   => The data uploaded with an engine task. (required)
        status    => The status of the task. Possible values are “pending” (the task was received but hasn’t been queued), “queued” (the task has been accepted for processing), “running” (the task is running), “failed” (the task failed and execution was  terminated), and “complete” (the task finished normally). (required)
      }
    }
  }
}
```

#### Sample Request: Create a Job
The example below shows a request for a job with four tasks: transcoding (first engine ID), transcription (second engine ID), translation of the transcript into Spanish (third engine ID), and insert-into-index (fourth engine ID).
```graphql
mutation {
  createJob(input: {
    targetId: "44512341"
    tasks: [
      {
        engineId: "fc004413-89f0-132a-60b2-b94522fb7e66"
        payload: {
          setAsPrimary: true
        },
      },
      {
        engineId: "762bf9a0-fc08-7fbb-4ba0-21a3cee6edaf"
      },
      {
        engineId: "b00fc05e-3502-e8da-4871-7182dc1aa9f2"
        payload: {
          target: "Spanish:es" 
        },
      },
      {
        engineId: "c2aaa6d7-14fa-f840-f77e-4d2c0b857fa8"
      }
    ]
  }) {
    id
    targetId
    tasks {
      records {
        id
        engineId
        order
        payload
        status
      }
    }
  }
}
```
#### Sample Response: Create a Job 
```json
{
  "data": {
    "createJob": {
      "id": "2791851b-0d02-4aaf-97e0-cd77a2e771ad",
      "targetId": "44512341",
      "tasks": {
        "records": [
          {
            "id": "2791851b-0d02-4aaf-97e0-cd77a2e771ad-caaa51ab-e4b4-49e3-b6d9-3291fa4b9836",
            "engineId": "fc004413-89f0-132a-60b2-b94522fb7e66",
            "order": 0,
            "payload": {
              "setAsPrimary": true
            },
            "status": "queued"
          },
          {
            "id": "2791851b-0d02-4aaf-97e0-cd77a2e771ad-fcd5c69b-0f8e-46f0-9aa0-2718529836ad",
            "engineId": "c2aaa6d7-14fa-f840-f77e-4d2c0b857fa8",
            "order": 3,
            "payload": {},
            "status": "pending"
          },
          {
            "id": "2791851b-0d02-4aaf-97e0-cd77a2e771ad-563e8f85-f733-45b7-8eb7-3ff68aed498a",
            "engineId": "b00fc05e-3502-e8da-4871-7182dc1aa9f2",
            "order": 2,
            "payload": {
              "target": "Spanish:es"
            },
            "status": "pending"
          },
          {
            "id": "2791851b-0d02-4aaf-97e0-cd77a2e771ad-a4c4767a-f7a4-4244-8481-a036a286e3a3",
            "engineId": "762bf9a0-fc08-7fbb-4ba0-21a3cee6edaf",
            "order": 1,
            "payload": {},
            "status": "pending"
          }
        ]
      }
    }
  }
}
```

## Check a Job Status

To check the status of the job and its tasks, make a call to the `job` query and provide the job `id` returned in the `createJob` mutation response. Successful calls return an object that contains the status for the job and each task. Unsuccessful calls will result in an error.

#### Request Payload: Check a Job Status
```graphql
query{
-------request fields-----------
  job                => The Job object query type. (required)
    (id: "string") { => The Job ID returned by the Create Job request. (required)
-------return fields------------
    id            => The unique ID of the job, provided by the server. (required)
    status        => The status of the job as a whole. Possible Values are “accepted” (the job is valid and has been accepted for processing), “running” (the job is running), “failed” (at least one task failed and job execution was terminated), and “complete” (all tasks in the job finished normally). (required)
    tasks {       => The tasks object parameter with data specific to the job’s tasks. (required)
      records {   => The records object parameter with data specific to individual tasks. (required)
        id        => The unique ID of a task, provided by the server. (required)
        status    => The status of the task. Possible values are “pending” (the task was received but hasn’t been queued), “queued” (the task has been accepted for processing), “running” (the task is running), “failed” (the task failed and execution was terminated), and “complete” (the task finished normally). (required)
      }
    }
  }
}
```

#### Sample Request: Check a Job Status
```graphql
query{
  job(id: "2791851b-0d02-4aaf-97e0-cd77a2e771ad") {
    id
    status
    tasks {
      records {
        id
        status
        engineId
      }
    }
  }
}
```

#### Sample Response: Check a Job Status
```json
{
  "data": {
    "job": {
      "id": "2791851b-0d02-4aaf-97e0-cd77a2e771ad",
      "status": "running",
      "tasks": {
        "records": [
          {
            "id": "2791851b-0d02-4aaf-97e0-cd77a2e771ad-caaa51ab-e4b4-49e3-b6d9-3291fa4b9836",
            "status": "complete",
            "engineId": "fc004413-89f0-132a-60b2-b94522fb7e66"
          },
          {
            "id": "2791851b-0d02-4aaf-97e0-cd77a2e771ad-fcd5c69b-0f8e-46f0-9aa0-2718529836ad",
            "status": "queued",
            "engineId": "f0acacba-2c33-86c1-0deb-c479906f17c5"
          },
          {
            "id": "2791851b-0d02-4aaf-97e0-cd77a2e771ad-563e8f85-f733-45b7-8eb7-3ff68aed498a ",
            "status": "queued",
            "engineId": "b00fc05e-3502-e8da-4871-7182dc1aa9f2"
          },
          {
            "id": "2791851b-0d02-4aaf-97e0-cd77a2e771ad-a4c4767a-f7a4-4244-8481-a036a286e3a3",
            "status": "pending",
            "engineId": "c2aaa6d7-14fa-f840-f77e-4d2c0b857fa8"
          }
        ]
      }
    }
  }
}
```

## Retrieve Job Outputs

Once a job has completed processing, a request can be made to retrieve an output asset produced by a task, such as a transcript, translation, or object detection results. 

### Retrieve a Transcript
To retrieve a transcript, make a request to the `job` query with the `job ID`. Transcripts are returned in time-correlated text fragments that contain beginning and ending times. A successful call returns the transcript and other details specified in the request. Otherwise, an error is returned.

#### Request Payload: Retrieve a Transcript
```graphql
query{
-------request fields-----------
  job                       => The Job object query type. (required)
    (id: "string") {        => The Job ID returned by the Create Job request. (required)
      assets(               => The assets object parameter with data specific to the job’s assets. (required)
      assetType: "string"){ => A label that classifies an asset. Use the value “transcript”. (required)
-------return fields------------
    records {       => The records object parameter with data specific to individual assets. (required)
      id:           => The unique ID of an asset, provided by the server. (required)
      signedUri:    => The path to the asset data. (required)
      contentType:  => The MIME type of the asset (e.g., audio/mp3). (required)
      transform(transformFunction: "string") => Converts an asset in XML format to JSON. Use the value “XML2JSON”. (required)
       }
      }
    }
    tasks {      => The tasks object parameter with data specific to the job’s tasks. (required)
      records {  => The records object parameter with data specific to individual assets. (required)
        id       => The unique ID of a task, provided by the server. (required)
        output   => The output produced by the task (e.g., transcript). (required)
        status   => The status of the task. Possible values are “pending” (the task was received but hasn’t been queued), “queued” (the task has been accepted for processing), “running” (the task is running), “failed” (the task failed and execution was terminated), and “complete” (the task finished normally). (required)
        engineId => The ID of the engine that processed the task. (optional)
      }
    }
  }
}
```

#### Sample Request: Retrieve a Transcript
```graphql
query:{
  job(id: "2791851b-0d02-4aaf-97e0-cd77a2e771ad") {
    target {
      id
      assets(assetType: "transcript") {
      records {
        id 
        signedUri
        contentType
        transform(transformFunction: XML2JSON)
       }
      }
    }
    tasks {
      records {
        id
        output
        status
        engineId
      }
    }
  }
}
```

#### Sample Response: Retrieve a Transcript
```json
{
  "data": {
    "job": {
      "target": {
        "id": "44512341",
        "assets": {
          "records": [
            {
              "id": "64d40793-119a-4847-980c-f4960e94aa26",
              "signedUri": "https://inspirent.s3.amazonaws.com/assets/39528568/ca40bd53-542f-4f47-8227-9052d71aa03f.ttml",
              "contentType": "application/ttml+xml",
              "transform": null
            },
            {
              "id": "005fa076-ae22-4572-83c9-cc60fc6e14bf",
              "signedUri": "https://inspirent.s3.amazonaws.com/assets/44512341/abcdc58e-bb95-4ab6-ab3f-2c76c2cea6ff.ttml",
              "transform": "{\"tt\":{\"$\":{\"xml:lang\":\"en-us\",\"xmlns\":\"http://www.w3.org/ns/ttml\",\"xmlns:tts\":\"http://www.w3.org/ns/ttml#styling\",\"xmlns:ttm\":\"http://www.w3.org/ns/ttml#metadata\"},\"head\":[{ \"metadata\":[{\"ttm:title\":[\"Media.wvx.aib \"],\"ttm:copyright\":[\"Copyright (c) 2013 Microsoft Corporation.  All rights reserved.\"]}],\"styling\":[{\"style\":[{\"$\":{\"xml:id\":\"Style1\",\"tts:fontFamily\":\"proportional SansSerif\",\"tts:fontSize\":\"0.8c\",\"tts:textAlign\":\"center\",\"tts:color\":\"white\"}}]}],\"layout\":[{\" region\":[{\"$\":{\"style\":\"Style1 \",\"xml:id\":\"CaptionArea\",\"tts:origin\":\"0c12.6c\",\"tts:extent\":\ "32c2.4c\",\"tts:backgroundColor\":\"rgba(0,0,0,160)\",\"tts:displayAlign\":\"center\",\"tts:padding\":\"0.3c0.5c\ "}}]}],\"recognizability\":[\"0.813\"]}] ,\"body\":[{\"$\":{\"region\":\"CaptionArea\"},\"div\":[{\"p\":[{\"_\ ":\"asdf\",\"$\":{\"begin\":\ "00:00:00.250\", \"end\":\"00:00:03.270\"}},{\"_\":\"of people descending  on the hermosa beach to begin a unique\",\"$\":{\"begin\":\"00:00:03.270\",\"end\":\"00:00:08.020\"}},{\"_\":\"...man competition exit includes running public drinkingbeer\",\" $\":{\"begin\":\"00:00:08.020\",\ "end\":\"00:00:12.680 \"}}, {\"_\":\"when they birch in the middle of the action good-morning moi.\",\"$\":{\"begin\":\"00:00:12.680\", \"end\":\"00:00:15.900\"}},{\"_\":\"I want to...like this that I was in the middle of the\",\"$\":{\"begin\": \"00:00:18.380 \",\"end\":\ "00:00:21.270\"}},{\"_\":\"mosh spent a lot of people who are drinking...\",\"$\":{ \"begin\":\"00:00:21.270\",\"end\":\"00:00:25.670\"}},{\"_\":\"You know, no,...come on, my friend\",\"$\":{\" begin\":\"00:00:25.670\",\"end\":\"00:00:29.060\"}},{\"_\":\"...it...get ugly...the...the invariant... let's just recap you\",\"$\":{\"begin\":\"00:00:29.060\",\"end\ ":\"00:00:35.220\"}}, {\"_\":\"...a mile on the beach yet paddled a mile in the ocean and now what do you do "begin\":\"00:00:40.and\",\"$\":{\"begin\":\"00:00:35.220 \",\"end\":\"00:00:40.270\"}},{\"_\":\"...they're.\",\"$\":{\ 270\",\"end\":\" 00:00:41.640\"}},{\"_\":\"Going to be... building you have as.\",\"$\":{\"begin\":\"00:00 :41.640\",\"end\":\"00: 00:48.050 \"}},{\"_\":\"Was line-of-business or six back on the moon...been training for the for...money by...\" ,\"$\":{\"begin\":\"00:00:51.890\" ,\"end\":\"00:01:02.470\"} },{\"_\":\"Is a locker room celebration\",\"$\":{\"begin\":\"00:01:04.720\",\"end\": \"00:01:06.150 \"}},{\"_\":\"...gentleman time to bring your out okay?\",\"$\":{\"begin\":\"00:01:06.150\",\"end\" :\"00:01:09.910\"}},{\"_\":\"I guess we saw everything...that live shot twice yet we\",\"$\":{\"begin\":\"00:01:09 .910\",\"end\":\" 00:01:14.280\"}},{\"_\":\"...one when I...back all right and...when you get out of there\",\"$\ ":{\"begin\":\ "00:01:14.280\",\"end\":\"00:01:19.570\"}},{\"_\":\"...bless you before you get a camera and something other than beer all right?\",\"$\":{\"begin\":\"00:01:19.570\",\"end\": \"00:01:25.710\"}}]}]}]}}"
          }
        ]
      }
    }
  }
}
```

#### Response Attributes: Retrieve a Transcript
<table>
  <tr>
    <td><b>Name</b></td>
    <td><b>Type</b></td>
    <td><b>Description</b></td>
  </tr>
  <tr>
    <td>startTime</td>
    <td>string</td>
    <td>The starting time of the text fragment (in timestamp format: HH&#058;mm:ss.S) from the beginning of the media file.</td>
  </tr>
  <tr>
    <td>stopTime</td>
    <td>string</td>
    <td>The ending time of the text fragment (in timestamp format: HH&#058;mm:ss.S) from the beginning of the media file.</td>
  </tr>
  <tr>
    <td>start</td>
    <td>float</td>
    <td>The number of seconds (in floating point format) from the beginning of the media file that the text fragment begins.</td>
  </tr>
  <tr>
    <td>end</td>
    <td>float</td>
    <td>The number of seconds (in floating point format) from the beginning of the media file that the text fragment ends.</td>
  </tr>
  <tr>
    <td>text</td>
    <td>string</td>
    <td>Text for the indicated start/stop times.</td>
  </tr>
</table>
<br>

### Retrieve Object Detection Results
To retrieve object detection results, make a request to the `job` query with the `job ID`. A successful request returns a JSON object with an array of the detected objects and details. Otherwise, an error is returned.

#### Request Payload: Retrieve Object Detection Results
```graphql
query{
-------request fields-----------
  job                       => The Job object query type. (required)
    (id: "string") {        => The Job ID returned by the Create Job request. (required)
      assets(               => The assets object parameter with data specific to the job’s assets. (required)
      assetType: "string"){ => A label that classifies an asset. Use the value “object”. (required)
-------return fields------------
       records {      => The records object parameter with data specific to individual assets. (required)
         id:          => The unique ID of an asset, provided by the server. (required)
         uri:         => The path to the asset data. (required)
         contentType: => The MIME type of the asset (e.g., audio/mp3). (required)
      tasks {         => The tasks object parameter with data specific to the job’s tasks. (required)
        records {     => The records object parameter with data specific to individual assets. (required)
          id          => The unique ID of a task, provided by the server. (required)
          output      => The output produced by the task (e.g., transcript). (required)
          status      => The status of the task. Possible values are “pending” (the task was received but hasn’t been queued), “queued” (the  task has been accepted for processing), “running” (the task is running), “failed” (the task failed and execution was terminated), and “complete” (the task finished normally). (required)
          engineId    => The ID of the engine that processed the task. (optional)
      }
    }
  }
}
```
#### Sample Request: Retrieve Object Detection Results
```graphql
query{
  job(id: "2791851b-0d02-4aaf-97e0-cd77a2e771ad") {
    target {
      id
      assets(assetType: "object") {
      records {
        id 
        uri
        contentType
       }
      }
    }
    tasks {
      records {
        id
        output
        status
        engineId
      }
    }
  }
}
```

#### Sample Response: Retrieve Object Detection Results
```json
{
  "data": {
    "job": {
      "target": {
        "id": "44512341",
        "assets": {
          "records": []
        }
      },
      "tasks": {
        "records": [
          {
            "id": "2791941b-0d33-4aaf-97e0-cd77a2e771ad-fcd5c69b-0f8e-46f0-9aa0-2718529836ad",
            "output": {
              "series": [
                {
                  "end": 2000,
                  "found": "Cricket",
                  "start": 0,
                  "salience": 88.5096206665039
                },
                {
                  "end": 10000,
                  "found": "Sport",
                  "start": 0,
                  "salience": 88.5096206665039
                },
                {
                  "end": 10000,
                  "found": "American Football",
                  "start": 0,
                  "salience": 58.762550354003906
                },
                {
                  "end": 10000,
                  "found": "Football",
                  "start": 0,
                  "salience": 58.762550354003906
                },
                {
                  "end": 11000,
                  "found": "Team",
                  "start": 0,
                  "salience": 53.086971282958984
                },
                {
                  "end": 8000,
                  "found": "Team Sport",
                  "start": 0,
                  "salience": 53.086971282958984
                },
                {
                  "end": 10000,
                  "found": "Arena",
                  "start": 4000,
                  "salience": 57.37635040283203
                },
                {
                  "end": 15000,
                  "found": "Field",
                  "start": 4000,
                  "salience": 57.37635040283203
                }
           ]
      }
    }
  }
}
```

#### Response Attributes: Retrieve Object Detection Results
<table>
  <tr>
    <td><b>Name</b></td>
    <td><b>Type</b></td>
    <td><b>Description</b></td>
  </tr>
  <tr>
    <td>series</td>
    <td>array</td>
    <td>An array of objects with data that describe a detected object.</td>
  </tr>
  <tr>
    <td>end</td>
    <td>integer</td>
    <td>The time that the object was last identified (in milliseconds) from the beginning of the media file.</td>
  </tr>
  <tr>
    <td>found</td>
    <td>string</td>
    <td>The name of the identified object.</td>
  </tr>
  <tr>
    <td>start</td>
    <td>integer</td>
    <td>The time that the object was first identified (in milliseconds) from the beginning of the media file.</td>
  </tr>
  <tr>
    <td>salience	</td>
    <td>integer</td>
    <td>The estimated probability that the detected object is correctly identified.</td>
  </tr>
</table>
<br>

### Retrieve a Translation
To retrieve a translation, make a request to the `job` query with the `Job ID` and specify `text` as the `assetType` value. Translations are returned as part of the task output. 

#### Request Payload: Retrieve a Translation
```graphql
query{
-------request fields-----------
  job                       => The Job object query type. (required)
    (id: "string") {        => The Job ID returned by the Create Job request. (required)
      assets(               => The assets object parameter with data specific to the job’s assets. (required)
      assetType: "string"){ => A label that classifies an asset. Use the value “text”. (required)
-------return fields------------
       records {      => The records object parameter with data specific to individual assets. (required)
         id:          => The unique ID of an asset, provided by the server. (required)
         uri:         => The path to the asset data. (required)
         contentType: => The MIME type of the asset (e.g., audio/mp3). (required)
      tasks {         => The tasks object parameter with data specific to the job’s tasks. (required)
        records {     => The records object parameter with data specific to individual assets. (required)
          id          => The unique ID of a task, provided by the server. (required)
          output      => The output produced by the task (e.g., transcript). (required)
          status      => The status of the task. Possible values are “pending” (the task was received but hasn’t been queued), “queued” (the task has been accepted for processing), “running” (the task is running), “failed” (the task failed and execution was terminated), and “complete” (the task finished normally). (required)
          engineId    => The ID of the engine that processed the task. (optional)
      }
    }
  }
}
```

#### Sample Request: Retrieve a Translation
```graphql
query{
  job(id: "2791851b-0d02-4aaf-97e0-cd77a2e771ad") {
    target {
      id
      assets(assetType: "text") {
      records {
        id 
        uri
        contentType
       }
      }
    }
    tasks {
      records {
        id
        output
        status
        engineId
      }
    }
  }
}
```
#### Sample Response: Retrieve a Translation
```json
{
  "data": {
    "job": {
      "target": {
        "id": "44512341",
        "assets": {
          "records": [
            {
              "id": "2791851b-0d02-4aaf-97e0-cd77a2e771ad",
              "uri": "https://inspirent.s3.amazonaws.com/assets/44512341/50669e82-4a10-43e6-99f0-314699aa32c2.txt",
              "contentType": "text/plain"
            }
          ]
        }
      },
      "tasks": {
        "records": [
          {
            "id": "2791851b-0d02-4aaf-97e0-cd77a2e771ad-00adce52-6999-46ac-a63f-33109b6e2edc",
            "output": {},
            "status": "complete",
            "engineId": "915bb300-bfa8-4ce6-8498-50d43705a144"
          },
          {
            "id": "2791851b-0d02-4aaf-97e0-cd77a2e771ad-1a15d675-ce3d-44f7-a026-dac508b3bf43",
            "output": {
              "response": "inserted"
            },
            "status": "complete",
            "engineId": "c2aaa6d7-14fa-f840-f77e-4d2c0b857fa8"
          },
          {
            "id": "2791851b-0d02-4aaf-97e0-cd77a2e771ad-beca03a4-32f8-42b7-8607-6370884c4538",
            "output": {
              "response": "inserted"
            },
            "status": "complete",
            "engineId": "c2aaa6d7-14fa-f840-f77e-4d2c0b857fa8"
          },
          {
            "id": "2791851b-0d02-4aaf-97e0-cd77a2e771ad-59d01645-0e8b-4fd1-838c-8597089e4af6",
            "output": {},
            "status": "complete",
            "engineId": "e3c78b0e-5ec9-3d24-67f2-ff2ee10afe10"
          },
          {
            "id": "2791851b-0d02-4aaf-97e0-cd77a2e771ad-8997b3a5-15dc-45f4-8647-8b01a5bf78f7",
            "output": {
              "Spanish": "Bien su sierra aquí vivo casi media hora hace cientos de personas desciende en su mayoría una playa para comenzar un concurso hombre de hierro único sí incluye correr remando y bebiendo cerveza cuando un pájaro tirado en medio de la mañana buena acción. Una vez tuve un sueño como el que me encontraba en medio de la multitud mucha gente estaban bebiendo cerveza cómo son No no no no vamos sobre ti eres mi amigo tuve vamos a obtener. Esto la iglesia ha de la resistencia ahora sólo Recapitulemos chicos corrió una milla en la playa que había remado una milla en el océano y ahora qué hacer para usar pero vas a espera que creo que eres. Creo que esto es. Estaba tratando de terminar para el paquete de 6. Guapura como usted ha estado entrenando para esto durante algún tiempo hasta los 25 años. Y es una celebración de vestuario que sabe usted es un caballero para llevarla cabo OK supongo que todo lo que tiro directo vimos en sí donde empiezas a saber lo que quiero y adónde va cuando usted sale de allí. Antes de que se había descompuesto algo que no sea cerveza. Muy bien."
            },
            "status": "complete",
            "engineId": "388d951e-c90c-f001-9d6b-8bb70b9e6267"
          },
          {
            "id": "2791851b-0d02-4aaf-97e0-cd77a2e771ad-1eef0201-4512-4b24-ad9f-20f99db985b2",
            "output": {
              "status": "finished",
              "mediaId": "b41b81f9-bc95-4e3e-98ab-06713a9f07ce",
              "metadata": {}
            },
            "status": "complete",
            "engineId": "f44aa80e-4650-c55c-58e7-49c965019790"
          },
          {
            "id": "2791851b-0d02-4aaf-97e0-cd77a2e771ad-d09a0885-8326-4333-899e-8e02309240bb",
            "output": {
              "assetId": "5bef94a1-d287-415a-afab-9280c8ba0a81",
              "recordingId": "44626362"
            },
            "status": "complete",
            "engineId": "adf1bbd2-c4b3-328e-5213-84c5b400ba32"
          }
        ]
      }
    }
  }
}
```
<br>

## Delete a TDO and/or Its Content

If a TDO is no longer needed, it can be deleted from an organization’s files to free up storage space or comply with organizational policies. The API provides flexible options that allow you to delete a TDO and all of its assets, or clean up a TDO's content by removing the associated assets so the TDO can be reused and new assets can be created.

### Delete a TDO and All Assets
To delete a TDO and all asset metadata, make a request to the `deleteTDO` mutation and pass the `TDO ID` as an argument. This operation is processed immediately at the time of the request and permanently deletes the specified TDO and its assets from the organization's account. Any subsequent requests against the TDO or assets will return an error.

#### Request Payload: Delete a TDO
```graphql
mutation{
-------request fields-----------
  deleteTDO {      => The Delete TDO mutation type. (required)
    (id: "string") => The unique ID of the TDO/Container to delete. (required)
-------return fields------------
         id:       => The ID of the deleted TDO/Container. (required)
         message:  => A message confirming deletion of the TDO. (required)
      }
    }
 ```

 #### Sample Request: Delete a TDO
 ```graphql
 mutation{
  deleteTDO(id: "44512341") 
     {
      id
      message
      }
    }
```
#### Sample Response: Delete a TDO
```json
{
  "data": {
    "deleteTDO": {
      "id": "44512341",
      "message": "TemporalDataObject 44512341 was deleted."
    }
  }
}
```

### Delete TDO Content
To remove the asset content associated with a TDO while retaining the TDO/container and asset metadata, make a request to the `cleanupTDO` mutation with the `TDO ID`. This mutation uses the `options` parameter along with any combination of the values below to specify the type of data to be deleted.   

* `storage`: Deletes the TDO's assets from storage, including engine results. Asset metadata will remain until the TDO/container is deleted.
* `searchIndex`: Deletes all search index data. The TDO and its assets will no longer be accessible through search.
* `engineResults`: Deletes engine results stored on related task objects. Engine results that are stored as assets will remain unless `storage` is passed as a value in the request.

*Note:* Requests that do not use the `options` parameter will remove the TDO's content from `storage` and the `search index` by default.

#### Request Payload: Delete TDO Content
```graphql
mutation{
-------request fields-----------
  cleanupTDO {            => The Cleanup TDO mutation type. (required)
    (id: "string"         => The unique ID of the TDO/Container of the content to delete. (required)
    options: [enum, enum] => The type of data to be deleted. (optional)
    )}
-------return fields------------
         id      => The ID of the TDO/Container associated with the deleted content. (required)
         message => A message confirming deletion of the TDO content. (required)
      }
    }
 ```

 #### Sample Request: Delete TDO Content
```graphql
mutation{
  cleanupTDO(id: "44512341", options: [storage, engineResults]) {
      id
      message
      }
    }
```

 #### Sample Response: Delete TDO Content
 ```json
 {
  "data": {
    "cleanupTDO": {
      "id": "44512341",
      "message": "Data deleted from 44512341:  storage,engineResults"
    }
  }
}
```
## Appendix 1
This section includes the following sample schemas:
* **Get All Engines**
* **Get All Engine Categories**
* **Get All Engines by Engine Category**

#### Get All Engines
Retrieve a full list of available Veritone engines with engine IDs, categories, and additional input field requirements.

##### Sample Request Payload: Get All Engine IDs
```graphql
query{
  engines => The Get Engines query type.
   (limit: 1000){ => The maximum number of results to return. Set to 1,000 to ensure a full list is retrieved.
    count         => The total number of engines returned in the response.
    records {     => The Records object parameter with data specific to an  individual engine.
      id          => The unique ID of the engine.
      category {  => The Category object parameter with data specific to an engine category.
    	id        => The ID of the engine category.
        name      => The name of the engine category.
    	}
  	{
 	 fields{  => The Fields object parameter with custom input fields used by the engine. 
       name   => The name of the field.
       type   => The field type, such as "Text" or “Picklist.”
       max    => A maximum value for the field. Applies only when “type” field is a “Number.”
       min    => A minimum value for the field. Applies only when “type” field is a “Number.”
       step   => A numeric step (in float format) in a series of incrementing/decrementing actions. Applies only when “type” field is a “number.”
       info   => A general description of the field.
       label  => The human-readable name for the field that displays in the UI.
       defaultValue => The default value assigned to the field if one is not specified by the user.
       options => A set of allowed values for the field. Applies only when “type” field value is “Picklist” or “Multi Picklist.”
         key   => The human-readable label for the option that displays in the UI (e.g., "English-US" for a language selector).
         value => The machine-readable value that will be sent in the engine payload (e.g., "en-us" for a language selector).
        }
      }
    }
  }
}
```

#### Get All Engine Categories

Retrieve a list of available Veritone engine categories.

##### Sample Request Payload: Get All Engine Categories
```graphql
query {
  engineCategories(limit: 1000){ => The Get Engine Categories query type.
   limit: 1000){  => The maximum number of results to return. Set value to 1,000 to ensure a full list is retrieved.       
    records {     => The Records object parameter with data specific to an  individual engine category.
      id          => The unique ID of the engine category.
      name        => The name of the engine category.
      description => The description of the engine category.
    }
  }
}
```

#### Get All Engines by Engine Category

Retrieve a list of available engines for a specified engine category with engine IDs and additional required input fields.

##### Sample Request Payload: Get All Engines by Category
```graphql
query{
  engines(       => The Get Engines query type.
   category: "engine category" => The name of the engine category. Enter a value returned from the Get All Engine Categories query.
   limit: 1000){ => The maximum number of results to return. Set value to 1,000 to ensure a full list is retrieved.
    count        => The total number of engines returned in the response. (optional)
    records {    => The Records object parameter with data specific to an  individual engine.
      id         => The unique ID of the engine.
      category { => The Category object parameter with data specific to an engine category.
    	id       => The ID of the engine category.
        name     => The name of the engine category.
    	}
  	{
 	 fields { => The Fields object parameter with custom input fields used by the engine.                     
       name    => The name of the field.
       type    => The field type, such as "Text" or “Picklist.”
       max     => A maximum value for the field. Applies only when “type” field is a “Number.”
       min     => A minimum value for the field. Applies only when “type” field is a “Number.”
       step    => A numeric step (in float format) in a series of incrementing/decrementing actions. Applies only when “type” field is a “number.”
       info    => A general description of the field.
       label   => The human-readable name for the field that displays in the UI.
       defaultValue => The default value assigned to the field if one is not specified by the user.
       options => A set of allowed values for the field. Applies only when the “type” field value is “Picklist” or “Multi Picklist.”
         key   => The human-readable label for the option that displays in the UI (e.g., "English-US" for a language selector).
         value => The machine-readable value that will be sent in the engine payload (e.g., "en-us" for a language selector).
        }
      }
    }
  }
}
```
