# Document Creation Using Adobe SDK

## Table of Contents

- [1. Get Adobe Credentials](#1-get-adobe-credentials)
- [2. Setup Adobe SDK](#2-setup-adobe-sdk)
- [3. Initialize SDK](#3-initialize-sdk)
- [4. Create Document](#4-create-document)
- [5. Save Document](#5-save-document)
- [6. Setting Up Environment Variables](#6-setting-up-environment-variables)
- [7. Setting Up a Node.js Environment](#7-setting-up-a-node-js-environment)
- [8. Running the Samples](#8-running-the-samples)
- [9. Verifying Download Authenticity](#9-verifying-download-authenticity)
- [10. Logging](#10-logging)
- [11. Custom Projects](#11-custom-projects)

## Steps to Setup Adobe SDK and Create Documents

### 1. Get Adobe Credentials

To use the Adobe SDK, you need to obtain your credentials from the Adobe Admin Console. Follow these steps to download your credentials:

1. **Log in to the Adobe Admin Console**: Go to the Adobe Admin Console and sign in with your Adobe ID.
2. **Navigate to the Service Account (JWT)**: Once logged in, click on the **Console** button in the top right corner. Scroll down and click on the associated project.
3. **Generate Key Pair**: On the left side, click on **Service Account (JWT)**. Scroll down and click on **Generate public/private keypair**. A `config.zip` file will be downloaded.
4. **Extract the Credentials**: Open the `config.zip` file and extract the contents. You will find a `pdfservices-api-credentials.json` file containing your `client_id`, `client_secret`, and `organization_id`.

Here is an example `pdfservices-api-credentials.json` file:

```json
{
  "client_credentials": {
    "client_id": "e5d18eb1ca3e439e8c92bbab6878a5c0",
    "client_secret": "p8e-8rZSPCqvPLJ3fIISF2Fz_AwQPFNmsFAr"
  },
  "service_principal_credentials": {
    "organization_id": "10990B8D672DD3BF0A495C4E@AdobeOrg"
  }
}
```

### 2. Setup Adobe SDK

Install the Adobe SDK and set up your project.

**Installation:**

```bash
npm install --save @adobe/pdfservices-node-sdk
```

### 3. Initialize SDK

Initialize the Adobe SDK with your credentials.

**Example:**

```javascript
const {
    ServicePrincipalCredentials,
    PDFServices,
    MimeType,
    DocumentMergeParams,
    OutputFormat,
    DocumentMergeJob,
    DocumentMergeResult,
    SDKError,
    ServiceUsageError,
    ServiceApiError 
    } = require("@adobe/pdfservices-node-sdk");

const fs = require("fs");

// Initial setup, create credentials instance
const credentials = new ServicePrincipalCredentials({
    clientId: process.env.PDF_SERVICES_CLIENT_ID,
    clientSecret: process.env.PDF_SERVICES_CLIENT_SECRET
});

// Creates a PDF Services instance
const pdfServices = new PDFServices({credentials});
```

### 4. Create Document

Use the SDK to create a document.

**Example:**

```javascript
// Create Document 
(async () => {
        try {
            // Upload Template
            const readStream = fs.createReadStream("resources/documentMergeTemplate.docx");
            const inputAsset = await pdfServices.upload({readStream, mimeType: MimeType.DOCX});

            // Document Merge Data 
            const jsonDataForMerge = {
                customerName: 'Kane Miller', 
                customerVisits: 100, 
                itemsBought: [
                    { 
                        description: 'Sprays', quantity: 50, amount: 100 
                    },
                    { 
                        description: 'Chemicals', quantity: 100, amount: 200
                    } 
                ],
                totalAmount: 300, 
                previousBalance: 50, 
                lastThreeBillings: [
                    100, 200, 300
                ], 
                photograph: 'data:image/png;base64,...'
            };

            // Job Parameters 
            const params = new DocumentMergeParams({
                 jsonDataForMerge, 
                 outputFormat: OutputFormat.DOCX 
            });

            // Submit Job and Get Result 
            const job = new DocumentMergeJob({
                inputAsset, 
                params
            }); 
            const pollingURL = await pdfServices.submit({job}); 
            const pdfServicesResponse = await pdfServices.getJobResult({pollingURL, resultType: DocumentMergeResult}); 

            // Save Result
            const resultAsset = pdfServicesResponse.result.asset; 
            const streamAsset = await pdfServices.getContent({asset: resultAsset}); 
            const outputFilePath = "output/mergedDocument.docx"; 
            const writeStream = fs.createWriteStream(outputFilePath); 
            streamAsset.readStream.pipe(writeStream); 
        } catch (error) {
            console.error("Error:", error); 
        }
})();
```

### 5. Save Document

Save the created document to your desired location.

**Example:**

```javascript
// Generates a string containing a directory structure and file name for the output file
function createOutputFilePath() {
    const filePath = "output/MergeDocumentToDOCX/";
    const date = new Date();
    const dateString = date.getFullYear() + "-" + ("0" + (date.getMonth() + 1)).slice(-2) + "-" +
        ("0" + date.getDate()).slice(-2) + "T" + ("0" + date.getHours()).slice(-2) + "-" +
        ("0" + date.getMinutes()).slice(-2) + "-" + ("0" + date.getSeconds()).slice(-2);
    fs.mkdirSync(filePath, { recursive: true });
    return (`${filePath}merge${dateString}.docx`);
}

// Save the document
(async () => {
    let readStream;
    try {
        // Your existing code for creating the document...

        // Create parameters for the job
        const params = new DocumentMergeParams({
            jsonDataForMerge,
            outputFormat: OutputFormat.DOCX
        });

        // Creates a new job instance
        const job = new DocumentMergeJob({ inputAsset, params });

        // Submit the job and get the job result
        const pollingURL = await pdfServices.submit({ job });
        const pdfServicesResponse = await pdfServices.getJobResult({
            pollingURL,
            resultType: DocumentMergeResult
        });

        // Get content from the resulting asset(s)
        const resultAsset = pdfServicesResponse.result.asset;
        const streamAsset = await pdfServices.getContent({ asset: resultAsset });

        // Creates a write stream and copy stream asset's content to it
        const outputFilePath = createOutputFilePath();
        console.log(`Saving asset at ${outputFilePath}`);

        const writeStream = fs.createWriteStream(outputFilePath);
        streamAsset.readStream.pipe(writeStream);
    } catch (err) {
        if (err instanceof SDKError || err instanceof ServiceUsageError || err instanceof ServiceApiError) {
            console.log("Exception encountered while executing operation", err);
        } else {
            console.log("Exception encountered while executing operation", err);
        }
    } finally {
        readStream?.destroy();
    }
})();
```

### 6. Setting Up Environment Variables

Create an .env file in your project root directory and add your Adobe credentials.

**.env File:**

```Plaintext
PDF_SERVICES_CLIENT_ID=<YOUR CLIENT ID>
PDF_SERVICES_CLIENT_SECRET=<YOUR CLIENT SECRET>
```

**Example pdfservices-api-credentials.json File**

```json
{
  "client_credentials": {
    "client_id": "<YOUR_CLIENT_ID>",
    "client_secret": "<YOUR_CLIENT_SECRET>"
  },
  "service_principal_credentials": {
    "organization_id": "<YOUR_ORGANIZATION_ID>"
  }
}
```

### 7. Setting Up a Node.js Environment

Running any sample or custom code requires the following steps:

1. Install Node.js 18.0 or higher.
2. The `@adobe/pdfservices-node-sdk` npm package automatically downloads when you build the sample project.

**Installation:**

```bash
npm install --save @adobe/pdfservices-node-sdk
```

### 8. Running the Samples

The quickest way to get up and running is to download the code samples during the Getting Credentials workflow. These samples provide everything from ready-to-run sample code, an embedded credential json file, and pre-configured connections to dependencies.

1. Download the Node.js sample project.
2. From the samples root directory, run `npm install`.
3. Set the environment variables `PDF_SERVICES_CLIENT_ID` and `PDF_SERVICES_CLIENT_SECRET` by running the following commands:

    **Windows:**

    ```bash
    set PDF_SERVICES_CLIENT_ID=<YOUR CLIENT ID>
    set PDF_SERVICES_CLIENT_SECRET=<YOUR CLIENT SECRET>
    ```

    **MacOS/Linux:**

    ```bash
    export PDF_SERVICES_CLIENT_ID=<YOUR CLIENT ID>
    export PDF_SERVICES_CLIENT_SECRET=<YOUR CLIENT SECRET>
    ```

4. Test the sample code on the command line:

    ```bash
    node yourSampleFile.js
    ```

Replace yourSampleFile.js with the actual name of your JavaScript file.

### 9. Verifying Download Authenticity

For security reasons, you may wish to confirm the installer's authenticity. To do so:

1. After installing the package, find and open package.json.
2. Find the _integrity key.
3. Verify the hash in the downloaded file matches the value published here: sha512-ZvwGfMlGa0mGK5HpPAdTTlsaeKKPLEbVfAeLsHr7YAQ6NO7cVkbxaPqV3Ng8dpq4mNj5Ah0Y1Q80uTjLb0Qf+A==.

### 10. Logging

Refer to the API docs for error and exception details.

The SDK uses the log4js API for logging. During execution, the SDK searches for config/pdfservices-sdk-log4js-config.json in the working directory and reads the logging properties from there. If you do not provide a configuration file, the default logging logs INFO to the console. Customize the logging settings as needed.

//log4js.properties file

```json
{
  "appenders": {
    "consoleAppender": {
      "_comment": "A sample console appender configuration, Clients can change as per their logging implementation",
      "type": "console",
      "layout": {
        "type": "pattern",
        "pattern": "%d:[%p]: %m"
      }
    }
  },
  "categories": {
    "default": {
      "appenders": ["consoleAppender"],
      "_comment": "Change the logging levels as per need. info is recommended for pdfservices-node-sdk",
      "level": "info"
    }
  }
}
```

### 11. Custom Projects

While building the sample project automatically downloads the Node package, you can do it manually if you wish to use your own tools and process.

Go to <https://www.npmjs.com/package/@adobe/pdfservices-node-sdk>

Download the latest package.

Follow the installation instructions provided in the package documentation.

This should provide a clear and comprehensive guide for setting up and using the Adobe SDK to create documents 😊📚🚀.
