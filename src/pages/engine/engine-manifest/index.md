---
title: Engine Manifest
---

## Overview
Every container uploaded to Veritone for an engine should include a manifest.json file,
which contains important information about your engine and build. Veritone relies on the
information in the manifest to correctly operate the engine on our platform, so please be
as accurate and comprehensive as you can. Omitting the manifest.json file or neglecting
to use the proper engineId will result in your build not being registered with the system.
If you have pushed an engine to our Docker registry and it is not showing up, check that 
your manifest is properly formatted and stored in the correct location (/var/manifest.json).

## Format
Manifest files should be written in JSON format. We expect the manifest to be stored in your 
container as  /var/manifest.json - this should be done in your Dockerfile
 ([example](https://github.com/veritone/veritone-sample-engine-py/blob/master/Dockerfile#L6)).

## Fields
The fields that should be included in manifest.json are listed in the table below. Please 
include all fields indicated as being required. If fields are missing, we may invalidate 
your container or in some cases, assume the default values. Fields that are not marked as 
being required are optional; they should be included in the manifest file if you have any 
values to declare but may be omitted if you don't.

| Field | Format | Required  | Description | Example |
| --- |:---:|:---:|---|---| 
| engineId|string|Yes|The ID of your engine. You can find your engine ID at the top of the Engines section pages in the Developer Portal.|{"engineId":"f06e3ecb-cb30-3d0f-3268-c08428dc72be"}|
| category| string|Yes|The category of the engine that you are providing. The available options are Refer to Cognitive Engine Classes for more information about each engine category.|"category": "transcription"|
| preferredInputFormat |string|Yes|Identify the MIME type of the input media format that is preferred by your engine. Choose one format only.|"preferredInputFormat": "audio/wav"|
| supportedInputFormats |string|No|List the MIME types of all of the input media formats that your engine can support. Include your preferred Input format here as well.|"supportedInputFormats": ["application/ttml+xml", "audio/wav"]|
| outputFormats|list of string|Yes|List all of the MIME types of the media formats that your engine will output. |"outputFormats": ["application/ttml+xml", "audio/wav"]|
| clusterSize|string|Yes|The cluster size on which your engine should run: small, medium, large (defined below)|"clusterSize": "small"|
| initialConcurrency|integer|No|The initial number of instances of your engine that can run at the same time.<br>If omitted, we will use a value of 50.|"initialConcurrency": 50|
| maxConcurrency|integer|No|The maximum number of instances of your engine that can run at the same time.<br>If omitted, we will use a value of 50|"maxConcurrency": 50|
| url|string|No|The URL of the website where the user can get more information about your engine.|"url": "https://www.veritone.com/aiware/engines/transcription-engine/"|
| externalCalls|string |No|The domains of any external calls that are required by your code. This should include all calls that require internet access.|"externalCalls": ["http://s3.amazonaws.com", "http://github.com"]|
| libraries|string|No|List any dependent libraries required by your engine.	|"libraries": ["tensorflow", "apache mahout"]|
| maxFileMb|float|No|The maximum file size that your engine can process, in megabytes. Omit this field if you engine can process any size of file.|"maxFileMb": 1200.0|
| minMediaLengthMs|integer|No|The minimum duration of the media file that your engine can process, expressed in milliseconds. Omit this field if your engine can process any length of media.	|"minMediaLengthMs": 1000|
| maxMediaLengthMs|integer|No|The maximum duration of the media file that your engine can process, expressed in milliseconds. Omit this field if your engine can process any length of media.	|"maxMediaLengthMs": 900000|
| trainableViaApi|boolean|No|Describes as to whether an API's available for training|"trainableViaApi": true|
| supportedLanguages|list of string|No|List of supported Languages in ISO 639-1 Codes|"supportedLanguages": ["en", "ko", "fr"]|
| gpuSupported|string|Yes|List of supported GPU engines<br> See the Supported GPU section below<br> Examples include: "G2", "G3", "P2"|"gpuSupported": "P2"|
| minMemoryRequired|integer|No|Minimum amount of RAM needed to run in MB.|"minMemoryRequired": 1024|
| releaseNotes|string|No|Tell users what has changed in this version of your code base. Enter unformatted, plain text in this field only.|"releaseNotes": "This version integrates a new algorithm that is better at detecting accented speech, specifically targeting Southern US accents. In addition to the improved accuracy, the algorithm runs 20% faster now. The version also fixes some minor bugs with dictionary files and permissions."|

## Supported Cluster Sizes
|Code|Ram|
| --- |---|
|small|512MB|
|medium|2GB|
|large|6GB|

## Supported Categories
* audioDetection
* facialDetection
* fingerprint
* geolocation
* logoRecognition
* objectDetection
* ocr
* sentiment
* transcription
* transcode
* translate

## Supported Mime Types
* application/json
* application/pdf
* application/smil+xml
* application/ttml+xml
* application/x-flv
* application/xml
* audio/aac
* audio/flac
* audio/midi
* audio/mp4
* audio/mpeg
* audio/wav
* audio/webm
* image/gif
* image/jpeg
* image/tiff
* text/csv
* text/html
* text/plain
* video/3gpp
* video/mp4
* video/mpeg
* video/ogg
* video/quicktime
* video/webm
* video/x-m4v
* video/x-ms-wmv
* video/x-msvideo

Contact us if you have a MIME type that is not currently supported by Veritone.


## Supported GPU Engines
| GPU | Supported | Code  |
| --- |:---:|:---:|
NVIDIA GRID K520 GPUs|Yes|G2|
NVIDIA Tesla M60 GPUs|Yes|G3|
NVIDIA Tesla K80 GPUs|Yes|P2|
NVIDIA Tesla V100 GPUs|	Coming soon!|	P3
Xilinx UltraScale+ VU9P FPGAs|	Coming soon!|	F1
NVIDIA Tesla M2050 GPUs|Coming soon!|	CG1

Example
Putting it all together, an example of a manifest.json submission is provided below.
```json
{
  "engineId": "f06e3ecb-cb30-3d0f-3268-c08428dc72be",
  "category": "transcription",
  "url": "https://www.veritone.com/aiware/engines/transcription-engine/",
  "externalCalls": ["http://s3.amazonaws.com", "http://github.com"],
  "preferredInputFormat": "audio/wav",
  "supportedInputFormats":
  [
    "audio/wav",
    "audio/mpeg",
    "audio/flac",
    "video/mp4",
    "application/json"
  ],
  "inputOptions":
  [
    {
        "name": "target",
        "label": "Output Language",
        "type": "picklist",
        "defaultValue": "en-us",
        "info":"The language that you would like to translate to",
        "options":
        {
            "en-us":"English",
            "es-mx":"Spanish",
            "fr-fr":"French"
         }
    },
    {
        "name": "priority",
        "label": "Priority",
        "type": "picklist",
        "defaultValue": "cost",
        "info":"The primary criteria for optimizing processing",
        "options":
        {
            "cost":"Minimize Cost",
            "speed":"Maximize Speed"
        }
    }
  ],
  "outputFormats":
  [
    "application/ttml+xml",
    "audio/wav"
  ],
  "initialConcurrency": 50,
  "maxConcurrency": 50,
  "clusterSize": "small",
  "maxFileMb": 1200.0,
  "minMediaLengthMs": 1000,
  "maxMediaLengthMs":  900000,
  "supportedLanguages": ["en", "fr"]
  "releaseNotes": "This version integrates a new algorithm that is better at detecting accented speech, specifically targeting Southern US accents. In addition to the improved accuracy, the algorithm runs 20% faster now. The version also fixes some minor bugs with dictionary files and permissions."
}
```