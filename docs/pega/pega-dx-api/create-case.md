# Creating New Cases Using Pega DX API

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Steps to Create a Case](#steps-to-create-a-case)
  - [1. Set Up Authentication](#1-set-up-authentication)
  - [2. Generate Microservice Code](#2-generate-microservice-code)
  - [3. Create the Request Body](#3-create-the-request-body)
  - [4. Add Fields to Allow Starting Fields](#4-add-fields-to-allow-starting-fields)
  - [5. Generate Token](#5-generate-token)
  - [6. Make the API Call](#6-make-the-api-call)
  - [7. Verify the Case Creation](#7-verify-the-case-creation)
  - [8. Handling Complex Data Types in **Pega Launchpad**](#8-handling-complex-data-types-in-pega-launchpad)
    - [Explanation of Operations](#explanation-of-operations)
- [Additional Resources](#additional-resources)
- [Troubleshooting](#troubleshooting)
- [Conclusion](#conclusion)

## Overview

The Pega Digital Experience (DX) API provides a set of model-driven REST API endpoints that enable developers to programmatically view, create, and update cases and assignments. This API is particularly useful for web self-service use cases, where it is important to align your user experience with your digital strategy while continuing to enable your business users to define and modify your application cases and user interface within Pega’s low-code model.

## Prerequisites

Before you start, ensure you have:

- Access to a **Pega Infinity** or **Pega Launchpad** environment
- Authentication credentials
- Basic knowledge of REST APIs and JSON

## Steps to Create a Case

### 1. Set Up Authentication

To access the Pega DX API in **Pega Infinity** or **Pega Launchpad**, you need to authenticate your requests. This typically involves setting up an authentication profile and resource path.

### 2. Generate Microservice Code

In **Pega Infinity**, navigate to the case type you want to use. On the **Settings** tab, click **Integration**, and under **Microservice APIs**, click **Generate create case microservice code**. This will generate the necessary code and data transforms.

### 3. Create the Request Body

The request body for creating a case typically includes the case type ID, process ID, parent case ID, and content. The content node should contain the required fields for the case.

**Example Request Body:**

```json
{
  "caseTypeID": "YourCaseTypeID",
  "processID": "",
  "parentCaseID": "",
  "content": {
    "FieldName1": "Value1",
    "FieldName2": "Value2"
  }
}
```

### 4. Add Fields to Allow Starting Fields

After preparing the JSON, add the request body fields to the "Allow Starting Fields" section.

- **Pega Launchpad**: In the case settings, navigate to the **Allow Starting Fields** section and add the necessary fields.
- **Pega Infinity**: In the data transform, navigate to the **Allow Starting Fields** section and add the necessary fields.

### 5. Generate Token

Before making the API call, generate an access token using your client credentials.

**Example Token Request:**

- **Endpoint:** `https://<Your_Pega_Auth_Domain>/oauth/token`
- **Method:** `POST`
- **Headers:**
  - `Content-Type`: `application/x-www-form-urlencoded`
- **Body:**
  - `client_id=<YOUR_CLIENT_ID>`
  - `client_secret=<YOUR_CLIENT_SECRET>`
  - `grant_type=client_credentials`

**Example Token Response:**

```json
{
  "access_token": "YourAccessToken",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

### 6. Make the API Call

Use the generated request URL, HTTP authentication header, and request body to make the API call in **Pega Infinity** or **Pega Launchpad**. Here’s an example of how to make the API call:

**Example API Call:**

- **Infinity**:
  - **Endpoint:** `https://<Application_Url>/api/application/v2/cases`

- **Launchpad**:
  - **Endpoint:** `https://<Application_Name>–<Subscriber_Name>.<Custom_Provider_Domain>/api/application/v2/cases`

**Request:**

- **Method:** `POST`
- **Headers:**
  - `Authorization: Bearer <YourAccessToken>`
  - `Content-Type: application/json`
- **Body:**

  ```json
  {
    "caseTypeID": "YourCaseTypeID",
    "processID": "",
    "parentCaseID": "",
    "content": {
      "FieldName1": "Value1",
      "FieldName2": "Value2"
    }
  }
  ```

### 7. Verify the Case Creation

After making the API call, verify that the case has been created successfully in **Pega Infinity** or **Pega Launchpad**. Retrieve the **`etag`** value from the response headers for future reference.

1. **Check the Response:** Ensure the API response indicates a successful case creation. Look for a status code of `201 Created` and a body containing the case details.
2. **Log in to Pega Infinity or Pega Launchpad:** Go to your Pega instance and search for the newly created case using its ID.
3. **Review Case Details:** Verify that all the fields and data have been correctly populated as per your request.

### 8. Handling Complex Data Types in **Pega Launchpad**

If the request body includes complex data types, you can use a PATCH call with embedded data to handle these scenarios.

**Example Embed Data Call:**

- **Endpoint:** `https://<Application_Name>–<Subscriber_Name>.<Custom_Provider_Domain>/dx/api/application/v2/cases/<CASE_ID>/actions/<VIEW_ID>?viewType=form`
- **Method:** `PATCH`
- **Headers:**
  - `If-Match`: `<etag>`
  - `Content-Type`: `application/json`
- **Body:**

    Certainly! Here's the JSON with comments and explanations of the operations happening:

    ```json
    {
        "content": {}, // Main content of the case
        "pageInstructions": [ // Instructions for updating the page content
            {
                "target": ".Arms", // Targeting the Arms node in the case structure
                "content": {}, // Content to be inserted
                "listIndex": 1, // Position where the content will be inserted in the list
                "instruction": "INSERT" // Operation to be performed: INSERT (adding a new item)
            },
            {
                "target": ".Arms", // Targeting the Arms node
                "content": {
                    "ArmGroupLabel": "Group A" // Label for the arm group
                },
                "listIndex": 1,
                "instruction": "UPDATE" // Operation to be performed: UPDATE (modifying an existing item)
            },
            {
                "target": ".Arms(1).Visits", // Targeting the Visits node within the first Arms element
                "content": {}, // Content to be inserted
                "listIndex": 1,
                "instruction": "INSERT" // Operation to be performed: INSERT (adding a new visit)
            },
            {
                "target": ".Arms(1).Visits", // Targeting the Visits node within the first Arms element
                "content": {
                    "VisitName": "Initial Consultation", // Name of the visit
                    "Category": "Screening", // Category of the visit
                    "Duration": "30", // Duration of the visit
                    "DurationType": "Day", // Type of duration
                    "Notes": "Initial consultation to discuss the procedure." // Additional notes about the visit
                },
                "listIndex": 1,
                "instruction": "UPDATE" // Operation to be performed: UPDATE (modifying the details of the visit)
            },
            {
                "target": ".Arms(1).Visits(1).ProcedureList", // Targeting the ProcedureList node within the first Visit of the first Arms element
                "content": {}, // Content to be inserted
                "listIndex": 1,
                "instruction": "INSERT" // Operation to be performed: INSERT (adding a new procedure)
            },
            {
                "target": ".Arms(1).Visits(1).ProcedureList", // Targeting the ProcedureList node within the first Visit of the first Arms element
                "content": {
                    "ProcedureName": "Blood Test", // Name of the procedure
                    "ProcedureCost": "100", // Cost of the procedure
                    "Notes": "Basic blood test to check overall health." // Additional notes about the procedure
                },
                "listIndex": 1,
                "instruction": "UPDATE" // Operation to be performed: UPDATE (modifying the details of the procedure)
            }
        ]
    }
    ```

#### Explanation of Operations

1. **INSERT Operation**:
   - **Target**: `.Arms`
   - **Description**: This operation is inserting a new item into the "Arms" list at the specified index.
   - **Purpose**: To add a new arm group to the case.

2. **UPDATE Operation**:
   - **Target**: `.Arms`
   - **Description**: This operation updates the existing item in the "Arms" list by setting the `ArmGroupLabel` to "Group A".
   - **Purpose**: To update the label for the existing arm group.

3. **INSERT Operation**:
   - **Target**: `.Arms(1).Visits`
   - **Description**: This operation inserts a new visit into the "Visits" list within the first arm group.
   - **Purpose**: To add a new visit entry to the arm group.

4. **UPDATE Operation**:
   - **Target**: `.Arms(1).Visits`
   - **Description**: This operation updates the existing visit with details such as `VisitName`, `Category`, `Duration`, `DurationType`, and `Notes`.
   - **Purpose**: To update the details of the existing visit.

5. **INSERT Operation**:
   - **Target**: `.Arms(1).Visits(1).ProcedureList`
   - **Description**: This operation inserts a new procedure into the "ProcedureList" within the first visit of the first arm group.
   - **Purpose**: To add a new procedure entry to the visit.

6. **UPDATE Operation**:
   - **Target**: `.Arms(1).Visits(1).ProcedureList`
   - **Description**: This operation updates the existing procedure with details such as `ProcedureName`, `ProcedureCost`, and `Notes`.
   - **Purpose**: To update the details of the existing procedure.

Each of these operations is designed to either add new data (INSERT) or modify existing data (UPDATE) within the case structure, ensuring the case data is accurately maintained and updated as needed.

## Additional Resources

For more detailed information, refer to the [Pega DX API Documentation](https://docs.pega.com/bundle/dx-api/page/platform/dx-api/dx-api-overview.html).

## Troubleshooting

If you encounter any issues, check the following:

- Ensure your authentication credentials are correct.
- Verify that the request body is correctly formatted.
- Check the API endpoint and request method.
- Review the API documentation for required parameters and headers.

## Conclusion

Using the Pega DX API in **Pega Infinity** and **Pega Launchpad**, you can create cases programmatically, aligning your user experience with your digital strategy while leveraging Pega’s powerful case management capabilities.
