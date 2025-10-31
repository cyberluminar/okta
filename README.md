# Cognyte Luminar Connector Documentation

## Overview

The Cognyte Luminar connector integrates with the Cognyte Luminar API to fetch compromised account records for security workflows in Okta Workflows. The connector provides a single action, Get Leaked Records, to retrieve leaked user data based on a specified fetch date, enabling automated breach detection and response.

## Setup

### Prerequisites

- Access to the Cognyte Luminar API at `https://www.cyberluminar.com/externalApi`.
- Valid API credentials: `luminar_client_id`, `luminar_client_secret`, and `luminar_account_id`.
- An Okta Workflows organization with Connector Builder enabled.

### Authentication Configuration

The Cognyte Luminar connector uses custom authentication with API key-based credentials. Follow these steps to configure the connection:

1. In Okta Workflows, navigate to **Connector Builder** and select the Cognyte Luminar connector.
2. Create a new connection:
   - **Luminar Client ID**: Enter your `luminar_client_id` (e.g., `moviesoft-v2`).
   - **Luminar Client Secret**: Enter your `luminar_client_secret` (e.g., `hjfgudsfaa`). This field is hidden for security.
   - **Luminar Account ID**: Enter your `luminar_account_id`.
3. Save the connection to authenticate API requests.

The connector sends the `luminar_client_id` and `luminar_client_secret` in the `Authorization` header as an API key (e.g., `Authorization: Bearer <client_id>:<client_secret>`) and includes the `luminar_account_id` as a query parameter or header, depending on API requirements.

## Action Card: Get Leaked Records

### Description

Fetches compromised account records from the Cognyte Luminar API based on a specified fetch date.

### Inputs

| Input Name | Type | Description | Required |
|------------|------|-------------|----------|
| fetch_date | Text | Date and time to filter leaked records in ISO 8601 format (e.g., `2025-09-23T13:51:00.000000`). | Yes |

### Outputs

| Output Name | Type | Description |
|-------------|------|-------------|
| output_obj | Object | Contains a `results` key with a list of objects representing compromised account records, incidents, malware, relationships, or IP addresses. |

#### Results List Details

The `results` key in `output_obj` contains a list of STIX 2.1-compliant objects with the following types and fields:

| Fields | Description | Type |
|--------|-------------|------|
| type | The type of STIX object (e.g., `user-account`, `incident`, `malware`, `relationship`, `ipv4-addr`). | Text |
| spec_version | The STIX specification version (e.g., `2.1`). | Text |
| id | Unique identifier for the object (e.g., `user-account--d10e0104-f10e-535e-b572-65894843605f`). | Text |
| extensions | Custom properties specific to Cognyte Luminar. | Object |
| extensions.{extension-definition-id}.extension_type | Type of extension (e.g., `property-extension`). | Text |
| extensions.{extension-definition-id}.luminar_tenant_id | Tenant ID for the Cognyte Luminar account (e.g., `00bed954-4b1a-4d52-97f7-2a2c51b824ff`). | Text |
| extensions.{extension-definition-id}.url | URL associated with the compromised credential (e.g., `www.7-eleven.com/login`). Only in `user-account`. | Text |
| extensions.{extension-definition-id}.source | Source of the compromised credential (e.g., `telegram`). Only in `user-account`. | Text |
| extensions.{extension-definition-id}.monitoring_plan_terms | List of terms monitored (e.g., `["7-eleven.com"]`). Only in `user-account`. | List of Text |
| extensions.{extension-definition-id}.credential_is_fresh | Indicates if the credential is recently compromised (e.g., `true`). Only in `user-account`. | Boolean |
| extensions.{extension-definition-id}.luminar_threat_score | Threat score for the incident (e.g., `121`). Only in `incident`. | Number |
| extensions.{extension-definition-id}.collection_date | Date the incident data was collected (e.g., `2025-06-16T19:00:28.832Z`). Only in `incident`. | Text (ISO 8601) |
| credential | Compromised credential (e.g., `Alexandria11`). Only in `user-account`. | Text |
| account_login | Login username or email (e.g., `mfuray81185@gmail.com`). Only in `user-account`. | Text |
| display_name | Display name for the account (e.g., `mfuray81185@gmail.com`). Only in `user-account`. | Text |
| created | Creation timestamp of the object (e.g., `2025-06-16T00:00:00.000Z`). In `incident`, `malware`, `relationship`. | Text (ISO 8601) |
| modified | Last modified timestamp (e.g., `2025-06-17T02:02:50.876Z`). In `incident`, `malware`, `relationship`. | Text (ISO 8601) |
| created_by_ref | Reference to the identity that created the incident (e.g., `identity--5bf1ac35-8d08-509e-b31a-044cb09b4199`). Only in `incident`. | Text |
| name | Name of the incident or malware (e.g., `@RATCLOUDS_-_775626_LINES_06.16.2025.txt` for incident, `REDLINE` for malware). In `incident`, `malware`. | Text |
| description | Description of the incident (e.g., credentials shared on Telegram). Only in `incident`. | Text |
| relationship_type | Type of relationship (e.g., `related-to`). Only in `relationship`. | Text |
| source_ref | Source object ID (e.g., `user-account--d10e0104-f10e-535e-b572-65894843605f`). Only in `relationship`. | Text |
| target_ref | Target object ID (e.g., `incident--9940dbc8-facd-5e08-8326-7af7875e961c`). Only in `relationship`. | Text |
| capabilities | Malware capabilities (e.g., `["steals-authentication-credentials"]`). Only in `malware`. | List of Text |
| malware_types | Types of malware (e.g., `["spyware"]`). Only in `malware`. | List of Text |
| is_family | Indicates if the malware is a family (e.g., `false`). Only in `malware`. | Boolean |
| value | IP address (e.g., `192.0.137.203`). Only in `ipv4-addr`. | Text |

### Configuration Steps

1. Add the **Get Leaked Records** action card to your flow.
2. Select the configured Cognyte Luminar connection.
3. Specify `fetch_date` to filter records for a specific date and time (e.g., `2025-09-23T13:51:00.000000`).
4. Map the `output_obj.results` output to subsequent cards for processing (e.g., store in a table or match with Okta users).

### Example Usage

To fetch and process leaked records:
1. Use **Get Leaked Records** with a specific `fetch_date` to retrieve records (e.g., `output_obj: {"results": [{"type": "user-account", "account_login": "mfuray81185@gmail.com", ...}, ...]}`).
2. Store the `output_obj.results` list in a Workflows table (e.g., `LeakedUsers`) for further processing.
3. Match `user-account` objects’ `account_login` against Okta users using **Okta - List Users** and trigger notifications or password resets for matched users.

### Notes
- The connector can handle large datasets (up to 9,999 records). Use batching in your flow (e.g., 50 records per iteration) to process `output_obj.results` efficiently.
- Ensure the `fetch_date` is in a valid ISO 8601 format (e.g., `YYYY-MM-DDTHH:MM:SS.ssssss`) to prevent API errors.
- Verify that `luminar_account_id` is correct to avoid authentication failures.

## Support

For issues with the Cognyte Luminar connector, contact your Cognyte Luminar account representative or Okta Support.
