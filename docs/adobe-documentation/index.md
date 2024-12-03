---
title: 'Template Document Generation using Adobe Document Generation API'
---

## Table of Contents

- [1. Generate Token](#1-generate-token)
- [2. Get Upload Presigned URI](#2-get-upload-presigned-uri)
- [3. Upload File](#3-upload-file)
- [4. Document Generation (DOCX or PDF)](#4-document-generation-docx-or-pdf)
- [5. Poll the Document Generation Periodically check the status of your document generation job using Adobe's API until it's completed](#5-poll-the-document-generation-periodically-check-the-status-of-your-document-generation-job-using-adobes-api-until-its-completed)
- [6. Get the Output File](#6-get-the-output-file)

## Steps to Generate Token, Upload File, Generate Document (DOCX or PDF), and Retrieve Output

### 1. Generate Token

Use Adobe's Identity Management Service (IMS) to generate an access token.

**Endpoint:** `https://ims-na1.adobelogin.com/ims/token`

**Request:**

- **Method:** `POST`
- **Headers:**
  - `Content-Type`: `application/x-www-form-urlencoded`
  - `client_id`=`<YOUR_CLIENT_ID>`
  - `client_secret`=`<YOUR_CLIENT_SECRET>`
  - `grant_type`=`client_credentials`

**Response:**

- **Status**: 200 OK

  ```json
  {
    "access_token": "<ACCESS_TOKEN>",
    "token_type": "bearer",
    "expires_in": 86399
  }
  ```

### 2. Get Upload Presigned URI

Get a pre-signed URL for uploading your file to Adobe's cloud storage.

**Endpoint:** `https://pdf-services.adobe.io/assets`

**Request:**

- **Method:** `POST`
- **Headers:**
  - `Authorization`: `Bearer <ACCESS_TOKEN>`
  - `Content-Type`: `application/json`
- **Body for DOCX (Raw JSON)**

  ```json
  {
    "mediaType": "application/vnd.openxmlformats-officedocument.wordprocessingml.document"
  }
  ```

- **Body for PDF (Raw JSON)**

  ```json
  {
    "mediaType": "application/pdf"
  }
  ```

**Response:**

- **Status**: 200 OK

  ```json
  {
    "upload_uri": "<UPLOAD_URI>",
    "asset_id": "<ASSET_ID>"
  }
  ```

### 3. Upload File

Use the pre-signed URL to upload your file to Adobe's cloud storage.

**Request:**

- **Method:** `PUT`
- **Headers:**
  - `Content-Type`: `application/octet-stream`
- **Body:**
  - The body should contain the raw binary content of the file you are uploading.

**Example:**

```http
PUT <UPLOAD_URI> HTTP/1.1
Host: <UPLOAD_HOST>
Content-Type: application/octet-stream

<FILE_CONTENT>
```

**Response:**

- **Status**: 200 OK

  ```json
  {
    "status": "success"
  }
  ```

### 4. Document Generation (DOCX or PDF)

Submit a document generation job to Adobe's API, specifying the desired format (DOCX or PDF) and providing the JSON data for merge.

**Endpoint:** `https://pdf-services.adobe.io/operation/documentgeneration`

**Request:**

- **Method:** `POST`
- **Headers:**
  - `Authorization`: `Bearer <ACCESS_TOKEN>`
  - `Content-Type`: `application/json`
- **Body (Raw JSON):**

  ```json
  {
    "assetID": "{{assetID}}",
    "outputFormat": "docx", // Change to "pdf" if you want a PDF
    "jsonDataForMerge": {
      "author": "Gary Lee",
      "Company": {
        "Name": "Projected",
        "Address": "19718 Mandrake Way",
        "PhoneNumber": "+1-100000098"
      }
    }
  }
  ```

**Response:**

- **Status: 201 Created**
- **Headers:**
  - `x-request-id`: `<JOB_ID>`

### 5. Poll the Document Generation Periodically check the status of your document generation job using Adobe's API until it's completed

**Endpoint:** `https://pdf-services-ue1.adobe.io/operation/documentgeneration/{{jobID}}/status`

**Request:**

- **Method:** `GET`
- **Headers:**
- `Authorization`: `Bearer <ACCESS_TOKEN>`
- `x-api-key`: `<YOUR_CLIENT_ID>`

**Response:**

- **Status**: 200 OK

  ```json
  {
    "status": "done/in progress",
    "asset": {
      "metadata": {
        "type": "application/vnd.openxmlformats-officedocument.  wordprocessingml.document",
        "size": 580360
      },
      "downloadUri": "<DOWNLOAD_URI>",
      "assetID": "urn:aaid:AS:UE1:08b0c24c-90ad-4ab0-aec3-291bfd421abf"
    }
  }
  ```

### 6. Get the Output File

Once the document generation is complete, download the output file from Adobe's cloud storage.

**Endpoint:** `<DOWNLOAD_URI>`

**Request:**

- **Method:** `GET`

**Response:**

- The response will contain the output file content in the specified format (DOCX or PDF).

**Example:**

```http
GET <DOWNLOAD_URI> HTTP/1.1
Host: <DOWNLOAD_URI>
```

**Response:**

- **Status:** 200 OK
- **Body:** The response will contain the output file content in the specified format (DOCX).

Notes:

- Ensure to handle authentication errors and retry logic as per your application needs.
- The key part is the `"format": "docx"` or `"format": "pdf"` line in the request payload. The `output_uri` you receive in Step 5 will point to the generated file in the specified format.
- Refer to Adobe's official documentation for more detailed instructions and error handling.

This completes the detailed steps for integrating with Adobe Document Services API. 😊📄🚀