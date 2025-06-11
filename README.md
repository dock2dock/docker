# Getting Started with Dock2Dock for Docker

This guide will help you install and run our Docker images using Docker Compose on your local machine.

## Supported third party systems

| Application        | Docker compose file            |
|--------------------|--------------------------------|
| Microsoft Dynamics | compose-nav.yaml               |

## Prerequisites

- Docker Desktop installed (Windows, Mac, or Linux)

> ⚠️ To ensure Docker Desktop restarts after a rebook
> - Open `Docker Desktop`.
> - Go to Settings > General.
> - Enable `Start Docker Desktop when you log in`.

- Docker Compose (included with Docker Desktop)

## Docker images

- api 

  The `api` image is responsible for sending new sales orders and updating existing orders. It acts as the main interface for interacting with dock2dock.

- nav-tasks

  The `nav-tasks` image is periodically polls the NAV API to retrieve newly modified sales orders. It runs on a scheduled cron job.

## NAV Integration

To enable integration with NAV, your API service must meet the following requirements:

#### Endpoint:

The API must expose an endpoint ending with `/dock2dockSalesOrders`. This endpoint will be used by the `nav-tasks` service to receive sales order data.

#### Authentication

The NAV API endpoint must support HTTP Basic Authentication, which is the standard authentication method for on-premise NAV implementations. The `nav-tasks` service will connect to your NAV API using a username and password, which should be provided via environment variables (`NAVAPI_USERNAME` and the password stored in `secrets/navapi_password.txt`). Ensure your API is configured to accept and validate Basic Authentication credentials for all requests to the `/dock2dockSalesOrders` endpoint.

#### Model Schema:

The data exchanged at this endpoint must conform to schema defined. Ensure your API implements this schema for compatibility with the integration.

| Field Name        | Type            | Max Length | Description | 
|-------------------|-----------------|-|------------|
| Id                    |	string  | 50 |Unique identifier for the sales order |
| No                    |	string  | 50 |Sales order number |
| SellToCustomerNo      |	string  | 50 |Customer number |
| SellToCustomerName    |	string	| 200 |Customer name |
| OurAccountNo	        |string	    | 50 |customer store number e.g. 8001 |
| YourReference         | string	| 200 |Reference provided by the customer |
| ShipmentDate          |DateOnly   |  |Date of shipment |
| RequestedDeliveryDate	|DateOnly?	|  |Requested delivery date (optional) |
| ExternalDocumentNo    |	string?	| 50 |External document number (optional) |
| IsCrossdock           |	bool	|  | Indicates if the order is crossdock. Default: false |
| LastModifiedDateTime  |DateTime	|  |Last modified date and time |
| ShippingAgentCode	    | string?	| 50 |Shipping agent code (optional) |

#### Expected JSON Response

The NAV API endpoint should return a JSON response in the following format, which is standard for on-premise NAV implementations:

```json
{
    "@odata.context": "string",
    "@odata.count": number,
    "value": [
        {
            "@odata.etag": "string",
            "id": "string",
            "sellToCustomerNo": "string",
            "no": "string",
            "yourReference": "string",
            "shipmentDate": "YYYY-MM-DD",
            "requestedDeliveryDate": "YYYY-MM-DD",
            "externalDocumentNo": "string",
            "ourAccountNo": "string",
            "sellToCustomerName": "string",
            "isCrossdock": boolean,
            "shippingAgentCode": "string",
            "lastModifiedDateTime": "YYYY-MM-DDTHH:MM:SSZ",
        }
        // ... more sales orders
    ]
}
```

- The root object contains OData metadata fields (`@odata.context`, `@odata.count`) and a `value` array of sales order objects.
- Each sales order object must include all required fields as described in the schema above.
- Dates should be in ISO 8601 format (e.g., `2024-09-05` for dates, `2024-09-08T23:20:04.187Z` for date-times).
- Optional fields may be empty strings or omitted if not applicable.

**Example:**

```json
{
    "@odata.context": "http://navURL/$metadata#companies(companyID)/dock2dockSalesOrders",
    "@odata.count": 1,
    "value": [
        {
            "@odata.etag": "W/\"JzQ0Ozl5aHRsbjZvNGhvYjFPR3p6clBmTUlZVDJWc3hGMzlKUEJuczh3bXEwVG89MTswMDsn\"",
            "id": "f96e63b4-b7fb-4cc5-9618-007db6a46aa2",
            "sellToCustomerNo": "C1135",
            "no": "42385",
            "yourReference": "",
            "shipmentDate": "2024-09-05",
            "requestedDeliveryDate": "0001-01-01",
            "externalDocumentNo": "",
            "ourAccountNo": "8097",
            "sellToCustomerName": "New World Blenheim",
            "isCrossdock": false,
            "shippingAgentCode": "",
            "lastModifiedDateTime": "2024-09-08T23:20:04.187Z",
        }
    ]
}
```


## Instructions

### 1. Clone the Repository

```bash
cd <your-repo-directory>
git clone https://github.com/dock2dock/docker Dock2dock-docker
cd Dock2dock-docker
```

### 2. Environment Variables

We provide a sample `.env` file containing all the environment variables you can set.

- For local development:

    Copy `.env` to `.env-local` and edit as needed.

- For QA or Production:

    Duplicate the `.env` file and name it accordingly (e.g., `.env-qa`, `.env-prod`).
    Edit the values for each environment.

### 3. Secrets

For improved security, Dock2Dock uses a `secrets` folder to store sensitive values such as API keys and passwords. This folder should already exist in your project.

#### **Populate your secrets:**

- For the Dock2Dock API key, add your key to `secrets/dock2dock_apikey.txt`:

```sh
echo "your-dock2dock-apikey" > secrets/dock2dock_apikey.txt
```

- For the NAV API password, add your password to `secrets/navapi_password.txt`:

```sh
echo "your-navapi-password" > secrets/navapi_password.txt
```

> **Important:**  
After deploying your services, we recommend deleting the API key and password from your `secrets` folder to minimise the risk of leaking sensitive information.

### 4. Docker Compose File

The cloned repository includes the docker compose file `compose-nav.yaml`

### 5. Build and Run

To start the services with the default `.env` file:

```bash
docker compose -f compose-nav.yaml up -d
```

To use a specific environment file (e.g., for QA or Production):

```bash
docker compose -f compose-nav.yaml -p dock2dock --env-file .env-qa up -d 
# or
docker compose -f compose-nav.yaml -p dock2dock --env-file .env-prod up -d
```

To update Dock2Dock docker images:

```bash
 docker-compose -f compose-nav.yaml pull
```

### 6. Stopping the Services

```bash
docker compose -f compose-nav.yaml down
```

### 7. Environment Variables Reference

| Variable Name     | Description                              |
|-------------------|------------------------------------------|
| DOCK2DOCK_BASEURL | Base API URL for Dock2Dock |
| NAVAPI_URL | The URL of the NAV API. If you are using `localhost` in the URL, replace it with `host.docker.internal`    |
| NAVAPI_USERNAME | The username to connect to the NAV API   | 
| NAVAPI_GET_SALES_ORDERS_FILTER | The GET sales order query filter. Please ensure your environment variable includes `lastModifiedDateTime ge {{LastSalesOrderSyncDate}}`. For example: `lastModifiedDateTime ge {{LastSalesOrderSyncDate}} and stage eq 'CONFIRM'`.       |
| NAVAPI_SYNC_SALES_ORDERS_ENABLED | Sync Sales Orders with Dock2Dock enabled |
| NAVAPI_SYNC_SALES_ORDERS_SCHEDULE | Scheduled cron schedule to fetch newly updated sales orders |
| CRON_TZ | Timezone for cron job. Default: New Zealand Standard Time |

#### Notes:

The `--env-file` flag lets you specify which environment file to use for different environments.