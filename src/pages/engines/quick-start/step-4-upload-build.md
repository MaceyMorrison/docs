---
title: Step 4 - Package and Upload an Engine Build
---

When your engine container is created, your manifest file is built, and your code is set, it&rsquo;s time to upload a build. A build is uploaded as a Docker image, which is a package that includes everything needed to run your software, including the code, libraries, environment variables, config files, and manifest file. Once your build is uploaded, Veritone will run it through compliance testing to ensure it&rsquo;s ready for prime time. Based on the results of the testing, your build will reflect a status that lets you know how to proceed.

In this section, we&rsquo;ll walk you through the steps to upload a build, including creating a Docker image and pushing it to Veritone&rsquo;s Docker registry. We&rsquo;ll also provide details about the compliance testing that your build will undergo and let you how you can download and review the results. Finally we&rsquo;ll dive into the different engine states and provide descriptions and optional actions that can be performed for each.

Remember that our developer support team is here to answer questions and provide assistance. Ping us on our dedicated Veritone Developer [Slack channel](https://chat.veritone.com) at [veritonevda.slack.com](http://veritonevda.slack.com). 

1. Upload a New Build
   * Create a Docker File
   * Package Your Engine
   * Log into Veritone's Docker Registry
   * Tag Your Engine
   * Upload Your Build
2. Veritone's Build Compliance Testing
3. Download a Build Report
4. Reading a Build Report
5. Understanding the Build States


#### Upload a New Build ####

The steps to upload a build are outlined below. While sample commands are provided with the instructions, it&rsquo;s recommended to copy the commands directly from the Developer App UI as they will include your organization ID. 

**Before You Begin**

To help expedite the upload process, be sure to have the following on-hand prior to getting started:

- Your Veritone Developer account login 
- Access to a command line interface on your computer
- Docker installed on your computer. If you don't have Docker, [download it here](https://www.docker.com/). 

If your build instructions are minimized, click on the down arrowhead next to the Copy button to expand the text.

#### Create a Dockerfile ####

Uploading a build in Veritone begins with building a Dockerfile. A Dockerfile is a text file that contains a set of instructions to create your Docker image. These instructions should include any configuration files required by your engine, such as API URLs (Veritone's API, your API, etc), authentication tokens (a Veritone token will be provided in the task payload), etc. If you'd like to get a sense of the different engine components that should be included in your Dockerfile, take a look at our Sample Engines.

**Important Notes Regarding Dockerfile Construction:**  

- Before uploading your build, you will need to install Docker using the [official Docker installation instructions](https://docs.docker.com/engine/installation/).
- The Dockerfile must include your manifest. Be sure it's copied to the correct location. (See sample Dockerfile below.)
- Use 'ENTRYPOINT' in [exec form](https://docs.docker.com/engine/reference/builder/#entrypoint) to launch your engine (not 'CMD').
- We recommend using [Alpine Linux ](https://hub.docker.com/_/alpine/)as a base image and giving your image a descriptive name. 
- To learn more about creating a Dockerfile, visit the [Dockerfile Reference page](https://docs.docker.com/engine/reference/builder/) on Docker&rsquo;s website.

Below is a Dockerfile sample for Go:

```bash
FROM golang:1.9.0-alpine
ADD . /go/src/github.com/your-org/your-engine
RUN apk update && apk add -U build-base git && \
   cd /go/src/github.com/your-org/your-engine && \
   GOPATH=/go go build -o your-engine . && \
   cp manifest.json /var/manifest.json # IMPORTANT! Manifests must be copied to this location.
ENTRYPOINT ["/go/src/github.com/your-org/your-engine/your-engine"]
```


Once your Dockerfile is created, proceed to the next steps package and upload your build.  

|Package and Upload Your Build| |
|--------| ---------- |
|<b>1. Package Your Engine</b><br> Build a Docker image from the Dockerfile, by clicking copy in the <b>Package your engine</b> section and running the Docker build command provided. The Docker build command uses the following syntax: *$ docker build -t my-engine-name. *A *Successfully built <image ID>* message displays when the image is created. <br><br><b>2. Log into Veritone's Docker Registry</b><br> Veritone's Docker registry is a central repository for storing Docker images and distributing containers. After creating your Docker image, you&rsquo;ll push it to your organization's private directory in the repository. As a Veritone developer, you can access the Docker registry using your Veritone Platform credentials. Copy and run the Docker login command provided in the *Log in to the Veritone docker registry* section to access the registry. Then enter your Veritone username and password to log in. The docker login command uses the following syntax: *$ docker login my-engine docker.veritone.com. <br><br><b>3. Tag Your Engine</b><br> Once logged into the Docker registry, copy and run the Docker tag command provided in the <b>Tag your engine</b> section to give your image a custom tag. This tag can be anything you would like to help organize your builds. The docker tag command uses the following syntax: *$ docker tag my-engine docker.veritone.com/organizationId/my-engine-name:custom-tag*. <br><br><b>4. Upload Your Build</b><br> Copy and run the Docker push command provided in the *Upload your build via command line* section to upload your build. The Docker push command uses the following syntax: *$ docker push docker.veritone.com/organizationId/my-engine-name:custom-tag*. Once your build is successfully uploaded to the registry, it displays under the engine&rsquo;s *Builds* list in the *Pending* state, and compliance testing is initiated. | <div style="width: 500px">![](VDA-Upload-an-Engine-1.png)</div> |

#### Veritone's Build Compliance Testing ####

When a build is submitted to Veritone, there are a number of tests run against it to make sure it is ready for prime time on our platform. This testing includes inspecting the packaged manifest, running the engine with a payload appropriate for the engine class, and examining the output and network usage of the run. In addition, a static analysis of vulnerabilities is performed in your Docker container.

An engine is expected to complete testing within 10 minutes, or it will be considered a timeout. When testing is complete, Veritone provides a Build Report with results of the testing that can be downloaded as a JSON file. If your build did not pass testing, this report can offer insight to help determine the cause. 

During testing, the build state displays as Fetching. Once testing is complete, the status changes to Available if the build passed or Invalid if it failed. A build in the Available state is ready to be submitted to Veritone for final review and approval. 

|To download a build report: | |
|--------|--------|
|Under the engine&rsquo;s *Builds*, click the **vertical ellipsis** on the right of the build in the list and select **Download build report** from the drop-down list. The Build Report downloads to your computer.| <div style="width: 500px">![](VDA-Download-Build-Report.png)</div>|

#### Reading a Build Report ####

The build report presents findings from Veritone&rsquo;s engine build testing to give you a clearer picture of satisfied criteria and areas where the build fell short. The table below includes short descriptions of each success criteria included in the build testing.

|Field name|Description|Example value|
|--------|--------|--------|
|engineId|The engine ID that this build belongs to.|2ff9260b-6788-45de-9757-980e5dc653e3|
|build|This build's build ID.|ef9b4702-6092-4b38-a7a6-1f6f8802f8f1|
|uploadTime|Epoch time (in milliseconds) that the build was uploaded.|1504028493942|
|user|Email address belonging to user who pushed this build to Veritone's Docker registry.|you@company.com|
|image|Name of Docker image that was pushed to Veritone's Docker regsitry.|docker.veritone.com/organizationId/my-engine:custom-tag|
|inspect|Output of `docker inspect` command.|See [https://docs.docker.com/engine/reference/commandline/inspect/](https://docs.docker.com/engine/reference/commandline/inspect/)|
|vulnerabilities|Array of vulnerabilities found in the build (if any).|See .|
|vulnerabilityCounts|Counts of vulnerabilities found in the build, by level of severity.|{    "Low": 6,    "Medium": 12}|
|testLog|Array of lines the build wrote to stdout during a test run.|[    "Starting my engines...",    "Processing complete",    "Elapsed time: 200ms"]|
|success|true when a build passes all tests &ndash; build marked `AVAILABLE`.false when a build fails any tests, contains critical vulnerabilities, or does not run properly &ndash; build marked `INVALID`.|true|
|errors|Array of errors encountered when testing the build.|["Error running container: exit code 1"]|

If vulnerabilities are detected in your engine build, they will be included for you to review in the vulnerabilities array of the report (see example below). Engine builds will only be marked invalid if we detect vulnerabilities labeled as "Critical." Anything below this level of severity will not prevent your engine from being made available, but will still be included for your review in the build report.
If you would like to test a container for vulnerabilities on your own, check out Clair's getting started guide.

#### Build States ####

The build state allows you easily identify and track a build&rsquo;s progression through the workflow cycle. Veritone uses six different states to capture the most relevant aspects of a build&rsquo;s lifecycle stages and general operations.

|**Build State**|**Description**|**Available Actions**|
|--------|--------|--------|
|Fetching|The *Fetching* state is system-triggered and takes effect when the build has been successfully uploaded and is undergoing compliance testing. |N/A|
|Available|The *Available* state is system-triggered and takes effect when a build successfully completes and passes the compliance testing process. A build in the *Available* state can be submitted to Veritone for approval and then be deployed into production.|Submit, Delete, Download Build Report|
|Invalid|The *Invalid* state is system-triggered and takes effect when a build fails the compliance testing process. A report can be downloaded that presents testing results.|Delete, Download Build Report|
|Approved|The *Approved* state is system-triggered and takes effect when build is verified as ready for deployment by Veritone. When a build enters the *Approved* state, the engine&rsquo;s state automatically transitions to *Ready*.|Deploy, Delete, Download Build Report|
|Disapproved|The *Disapproved* state is system-triggered and takes effect when build is not approved for deployment.|Delete|
|Deployed|The *Deployed* state takes effect when a user deploys an *Approved* build into production. When a build is *Deployed*, the engine&rsquo;s status moves to *Active*.|Pause, Delete, Download Build Report|
|Paused|The *Paused* state is user-triggered and suspends operation of the build. A build can be manually put in the *Paused* state, or a previously deployed build will enter the *Paused* state when a new build is *Deployed*. |Unpause, Delete, Download Build Report|