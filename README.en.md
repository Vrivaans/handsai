# HandsAI - AI Tool Management Microservice

## 🚀 Overview

HandsAI is a reactive microservice built with Spring Boot 3.2+ and Java 21 that allows Large Language Models (LLMs) to dynamically discover and execute tools through a unified interface following the "Model-facing Controller Protocol" (MCP). The system supports the management of REST APIs with dynamic discovery, parameter validation, and fault-tolerant execution.

## ✨ Important

To enable the full MCP functionality, you also need to download and run the `handsai-bridge` repository. This bridge is required for HandsAI to operate as a full MCP-compliant tool server. The repository will be made public soon.

### 🎯 Key Features

- **Dynamic Discovery**: LLMs can discover available tools at runtime.
- **Unified Interface (MCP)**: Standardized endpoints for LLMs to list and execute tools.
- **Tool Management**: Admin API to create, update, and monitor the lifecycle of tools.
- **Fault Tolerance**: Graceful error handling with comprehensive logging.
- **Smart Caching**: Tool definitions are cached in memory for high performance.
- **Virtual Threads**: Leverages Java 21 for high, non-blocking concurrency.

## 🛠️ Tech Stack

- **Framework**: Spring Boot 3.2+ with Spring WebFlux
- **Java**: Java 21 LTS with Virtual Threads
- **Database**: PostgreSQL with Spring Data JPA
- **Build**: Maven
- **Additional**: Lombok, Spring DevTools

## 📋 Prerequisites

- Java 21 LTS
- PostgreSQL 14+
- Maven 3.8+

## ⚡ Setup and Launch

1.  **Clone the repository**.

2.  **Configure the database**:
    Open the `src/main/resources/application.properties` file and adjust the connection properties for your PostgreSQL database:

    ```properties
    spring.datasource.url=jdbc:postgresql://localhost:5432/handsai_db
    spring.datasource.username=your_user
    spring.datasource.password=your_password
    spring.jpa.hibernate.ddl-auto=update
    ```

3.  **Build and run the application**:
    You can run the application using the Maven wrapper:
    ```bash
    ./mvnw spring-boot:run
    ```
    The service will be available at `http://localhost:8080`.

## 📖 API Endpoints

The API is divided into two sections: the **Admin API** for managing tools and the **LLM API (MCP)** for models to discover and execute them.

---

### Admin API (`/admin/tools`)

These endpoints are used to manage the lifecycle of `ApiTool` entities.

#### 1. Create an API Tool

- **Endpoint**: `POST /admin/tools/api`
- **Description**: Registers a new API tool in the system.
- **Request Body**:

  ```json
  {
    "name": "Get Weather",
    "description": "Provides the current weather for a city.",
    "baseUrl": "https://api.weatherapi.com",
    "endpointPath": "/v1/current.json",
    "httpMethod": "GET",
    "authentication": {
      "type": "API_KEY",
      "apiKeyName": "key",
      "apiKeyLocation": "QUERY"
    },
    "parameters": [
      {
        "name": "q",
        "type": "STRING",
        "description": "City name (e.g., 'London')",
        "required": true
      }
    ]
  }
  ```

#### 2. Update an API Tool

- **Endpoint**: `PUT /admin/tools/api/{id}`
- **Description**: Updates the details of an existing tool. The body can contain any fields to be modified.

- **Request Body (Example)**:
  ```json
  {
    "description": "Provides the current weather and forecast for a city.",
    "enabled": false
  }
  ```

#### 3. Get All API Tools

- **Endpoint**: `GET /admin/tools/api`
- **Description**: Returns a list of all registered tools.

#### 4. Get API Tool by ID

- **Endpoint**: `GET /admin/tools/api/{id}`
- **Description**: Returns the details of a specific tool.

#### 5. Delete an API Tool

- **Endpoint**: `DELETE /admin/tools/api/{id}`
- **Description**: Deletes a tool from the system.

#### 6. Validate Tool Health

- **Endpoint**: `POST /admin/tools/api/{id}/validate`
- **Description**: Forces a health check for a specific tool and returns its updated status.

---

### LLM API (MCP - `/mcp`)

These endpoints are designed to be consumed by LLMs and follow the "Model-facing Controller Protocol".

#### 1. List Available Tools

- **Endpoint**: `GET /mcp/tools/list`
- **Description**: Returns the tools that the LLM can use, in MCP format.
- **Response Body (Example)**:

  ```json
  {
    "jsonrpc": "2.0",
    "result": {
      "tools": [
        {
          "name": "Get Weather",
          "description": "Provides the current weather for a city.",
          "input_schema": {
            "type": "object",
            "properties": {
              "q": {
                "type": "string",
                "description": "City name (e.g., 'London')"
              }
            },
            "required": ["q"]
          }
        }
      ]
    }
  }
  ```

#### 2. Execute a Tool

- **Endpoint**: `POST /mcp/tools/call`
- **Description**: Executes a tool with the provided arguments.
- **Request Body**:

  ```json
  {
    "jsonrpc": "2.0",
    "method": "tools/call",
    "params": {
      "name": "Get Weather",
      "arguments": {
        "q": "Buenos Aires"
      }
    },
    "id": "req-12345"
  }
  ```

- **Response Body (Example)**:
  The `text` field contains the result from the external API as a JSON string.

  ```json
  {
    "jsonrpc": "2.0",
    "result": {
      "content": [
        {
          "type": "text",
          "text": "{'location':{'name':'Buenos Aires', ...},'current':{'temp_c':18.0, ...}}"
        }
      ]
    },
    "id": "req-12345"
  }
  ```