# HandsAI - IA como cerebro, HandsAI como sus manos.

## ✨ Importante
Para funcionar se necesita también descargar handsai-bridge: https://github.com/Vrivaans/handsai-bridge

## 🚀 Descripción

HandsAI es un microservicio reactivo construido con Spring Boot 3.2+ y Java 21 que permite a los Modelos de Lenguaje Grande (LLMs) descubrir y ejecutar herramientas dinámicamente a través de una interfaz unificada que sigue el "Model-facing Controller Protocol" (MCP). El sistema soporta la gestión de APIs REST con descubrimiento dinámico, validación de parámetros y ejecución tolerante a fallos.

### 🎯 Características Principales

- **Descubrimiento Dinámico**: Los LLMs pueden descubrir herramientas disponibles en tiempo de ejecución.
- **Interfaz Unificada (MCP)**: Endpoints estandarizados para que los LLMs listen y ejecuten herramientas.
- **Gestión de Herramientas**: API de administración para crear, actualizar y monitorear el ciclo de vida de las herramientas.
- **Tolerancia a Fallos**: Manejo elegante de errores con logging completo.
- **Caché Inteligente**: Definiciones de herramientas cacheadas en memoria para alta performance.
- **Hilos Virtuales**: Aprovecha Java 21 para alta concurrencia sin bloqueo.

## 🛠️ Stack Tecnológico

- **Framework**: Spring Boot 3.2+ con Spring WebFlux
- **Java**: Java 21 LTS con Virtual Threads
- **Base de Datos**: PostgreSQL con Spring Data JPA
- **Build**: Maven
- **Adicionales**: Lombok, Spring DevTools

## 📋 Requisitos Previos

- Java 21 LTS
- PostgreSQL 14+
- Maven 3.8+

## ⚡ Configuración y Arranque

1.  **Clonar el repositorio**.

2.  **Configurar la base de datos**:
    Abre el archivo `src/main/resources/application.properties` y ajusta las propiedades de conexión a tu base de datos PostgreSQL:

    ```properties
    spring.datasource.url=jdbc:postgresql://localhost:5432/handsai_db
    spring.datasource.username=tu_usuario
    spring.datasource.password=tu_contraseña
    spring.jpa.hibernate.ddl-auto=update
    ```

3.  **Construir y ejecutar la aplicación**:
    Puedes ejecutar la aplicación usando el wrapper de Maven:
    ```bash
    ./mvnw spring-boot:run
    ```
    El servicio estará disponible en `http://localhost:8080`.

## 📖 API Endpoints

La API se divide en dos secciones: la **API de Administración** para gestionar las herramientas y la **API para LLMs (MCP)** para que los modelos las descubran y ejecuten.

---

### API de Administración (`/admin/tools`)

Estos endpoints se utilizan para gestionar el ciclo de vida de las `ApiTool`.

#### 1. Crear una Herramienta API

- **Endpoint**: `POST /admin/tools/api`
- **Descripción**: Registra una nueva herramienta de API en el sistema.
- **Request Body**:

  ```json
  {
    "name": "Obtener Clima",
    "description": "Proporciona el clima actual para una ciudad.",
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
        "description": "Nombre de la ciudad (ej. 'London')",
        "required": true
      }
    ]
  }
  ```

#### 2. Actualizar una Herramienta API

- **Endpoint**: `PUT /admin/tools/api/{id}`
- **Descripción**: Actualiza los detalles de una herramienta existente. El body puede contener cualquier campo que se desee modificar.

- **Request Body (Ejemplo)**:
  ```json
  {
    "description": "Proporciona el clima actual y el pronóstico para una ciudad.",
    "enabled": false
  }
  ```

#### 3. Obtener todas las Herramientas API

- **Endpoint**: `GET /admin/tools/api`
- **Descripción**: Devuelve una lista de todas las herramientas registradas.

#### 4. Obtener una Herramienta API por ID

- **Endpoint**: `GET /admin/tools/api/{id}`
- **Descripción**: Devuelve los detalles de una herramienta específica.

#### 5. Eliminar una Herramienta API

- **Endpoint**: `DELETE /admin/tools/api/{id}`
- **Descripción**: Elimina una herramienta del sistema.

#### 6. Validar Salud de una Herramienta

- **Endpoint**: `POST /admin/tools/api/{id}/validate`
- **Descripción**: Fuerza una comprobación de salud para una herramienta específica y devuelve su estado actualizado.

---

### API para LLMs (MCP - `/mcp`)

Estos endpoints están diseñados para ser consumidos por LLMs y siguen el "Model-facing Controller Protocol".

#### 1. Listar Herramientas Disponibles

- **Endpoint**: `GET /mcp/tools/list`
- **Descripción**: Devuelve las herramientas que el LLM puede usar, en formato MCP.
- **Response Body (Ejemplo)**:

  ```json
  {
    "jsonrpc": "2.0",
    "result": {
      "tools": [
        {
          "name": "Obtener Clima",
          "description": "Proporciona el clima actual para una ciudad.",
          "input_schema": {
            "type": "object",
            "properties": {
              "q": {
                "type": "string",
                "description": "Nombre de la ciudad (ej. 'London')"
              }
            },
            "required": ["q"]
          }
        }
      ]
    }
  }
  ```

#### 2. Ejecutar una Herramienta

- **Endpoint**: `POST /mcp/tools/call`
- **Descripción**: Ejecuta una herramienta con los argumentos proporcionados.
- **Request Body**:

  ```json
  {
    "jsonrpc": "2.0",
    "method": "tools/call",
    "params": {
      "name": "Obtener Clima",
      "arguments": {
        "q": "Buenos Aires"
      }
    },
    "id": "req-12345"
  }
  ```

- **Response Body (Ejemplo)**:
  El campo `text` contiene el resultado de la API externa como una cadena de texto JSON.

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