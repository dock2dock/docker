# Getting Started with Dock2Dock for Docker

This guide will help you install and run our Docker images using Docker Compose on your local machine.

## Supported third party systems

| Application        | Docker compose file            |
|--------------------|--------------------------------|
| Microsoft Dynamics | compose-nav.yaml               |

## Prerequisites

- Docker Desktop installed (Windows, Mac, or Linux)
- Docker Compose (included with Docker Desktop)

## Docker images

- api 

  The `api` image is responsible for sending new sales orders and updating existing orders. It acts as the main interface for interacting with dock2dock.

- nav-tasks

  The `nav-tasks` image is periodically polls the NAV API to retrieve newly modified sales orders. It runs on a scheduled cron job (0 0/2 5-18 ? * MON,TUE,WED,THU,FRI,SAT,SUN *). 

## NAV Integration

To enable integration with NAV, your API service must meet the following requirements:

#### Endpoint:

The API must expose an endpoint ending with `/dock2dockSalesOrders`. This endpoint will be used by the `nav-tasks` service to receive sales order data.

#### Model Schema:

The data exchanged at this endpoint must conform to the model defined in NavSalesOrder.cs within the Nav.Api.Client.Models namespace. Ensure your API implements this schema for compatibility with the integration.

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

## Instructions

### 1. Clone the Repository

```bash
git clone <your-repo-url>
cd <your-repo-directory>
```

### 2. Environment Variables

We provide a sample `.env` file containing all the environment variables you can set.

- For local development:

    Copy `.env` to `.env.local` and edit as needed.

- For QA or Production:

    Duplicate the `.env` file and name it accordingly (e.g., `.env.qa`, `.env.prod`).
    Edit the values for each environment.

### 3. Docker Compose File

The cloned repository includes the docker compose file `compose-nav.yaml`

### 4. Build and Run

To start the services with the default `.env` file:

```bash
docker compose -f compose-nav.yaml up -d
```

To use a specific environment file (e.g., for QA or Production):

```bash
docker compose -f compose-nav.yaml --env-file .env.qa up -d
# or
docker compose -f compose-nav.yaml --env-file .env.prod up -d
```

### 5. Stopping the Services

```bash
docker compose down
```

### 6. Environment Variables Reference

| Variable Name     | Description                              |
|-------------------|------------------------------------------|
| DOCK2DOCK_APIKEY | ApiKey to access the Dock2Dock API       |
| DOCK2DOCK_BASEURL | Base API URL for Dock2Dock |
| NAVAPI_URL | The URL of the NAV API. If you are using `localhost` in the URL, replace it with `host.docker.internal`    |
| NAVAPI_USERNAME | The username to connect to the NAV API   | 
| NAVAPI_PASSWORD | The password to connect to the NAV API   | 
| NAVAPI_GET_SALES_ORDERS_FILTER | The GET sales order query filter         |
| NAVAPI_SYNC_SALES_ORDERS_ENABLED | Sync Sales Orders with Dock2Dock enabled |

#### Notes:

The `--env-file` flag lets you specify which environment file to use for different environments.