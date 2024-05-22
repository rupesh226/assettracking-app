Here's a step-by-step guide and documentation for setting up and running the `assettracking-app` using the provided `docker-compose.yml` file. This guide also includes instructions to access the Swagger UI for the application.

## Asset Tracking Application Setup and Configuration

### Prerequisites

1. **Docker**: Ensure Docker is installed on your local machine. You can download and install Docker from [here](https://www.docker.com/products/docker-desktop).
2. **Docker Compose**: Ensure Docker Compose is installed. Docker Desktop includes Docker Compose, but you can also install it separately from [here](https://docs.docker.com/compose/install/).

### Step-by-Step Guide

1. **Clone the Repository** (if applicable):

   ```bash
   git clone https://github.com/rupesh226/assettracking-app.git
   cd assettracking-app
   ```

2. **Double Check Necessary Directories**:
   Ensure the directories for DynamoDB local data and configuration files exist:

   **_dynamodblocal/data_** - For database, we already have all dataset with indexing
   **_config_** - for configuration, springboot application.propeties share from this files

3. **Create Docker Compose File**:
   Ensure you have the following `docker-compose.yml` file in the root directory of your project:

   ```yaml
   version: "3.8"

   networks:
     assettracking-network:

   services:
     dynamodb:
       image: amazon/dynamodb-local:2.3.0
       container_name: dynamodb
       ports:
         - "8000:8000"
       volumes:
         - ./dynamodblocal/data:/home/dynamodblocal/data
       command: -jar DynamoDBLocal.jar -sharedDb -dbPath /home/dynamodblocal/data
       networks:
         - assettracking-network

     redis:
       image: redis:latest
       container_name: redis
       ports:
         - "6379:6379"
       networks:
         - assettracking-network
       healthcheck:
         test: ["CMD", "redis-cli", "ping"]
         interval: 10s
         timeout: 5s
         retries: 5

     assettracking:
       image: rupesh226/assettracking:1.0.1
       container_name: assettracking
       ports:
         - "8080:8080"
       volumes:
         - ./config:/etc/assettracking/config/
       environment:
         SPRING_DATA_DYNAMODB_ENDPOINT: http://dynamodb:8000
         SPRING_DATA_DYNAMODB_REGION: us-west-2
         SPRING_REDIS_HOST: redis
         SPRING_REDIS_PORT: 6379
       depends_on:
         redis:
           condition: service_healthy
         dynamodb:
           condition: service_started
       networks:
         - assettracking-network
   ```

4. **Run Docker Compose**:
   In the root directory of your project where the `docker-compose.yml` file is located, run the following command to start the services:

   ```bash
   docker-compose up --build
   ```

5. **Verify Services**:

   - **DynamoDB**: Ensure DynamoDB local is running on `http://localhost:8000`.
   - **Redis**: Ensure Redis is running and healthy.
   - **Asset Tracking Application**: Ensure the application is running on `http://localhost:8080`.

6. **Access the Application**:
   Open a web browser and go to the Swagger UI to access the application documentation and test the endpoints:
   ```
   http://localhost:8080/swagger-ui/index.html
   ```

### Docker Compose File Breakdown

- **Networks**:

  - `assettracking-network`: A custom network that allows the services to communicate with each other.

- **Services**:

  - **dynamodb**:

    - Uses the `amazon/dynamodb-local` image to run a local instance of DynamoDB.
    - Maps port `8000` on the host to port `8000` in the container.
    - Mounts a volume for persistent data storage.
    - Configures the command to use a shared database and specify the data path.

  - **redis**:

    - Uses the `redis:latest` image to run Redis.
    - Maps port `6379` on the host to port `6379` in the container.
    - Includes a health check to ensure Redis is running and healthy.

  - **assettracking**:
    - Uses the `rupesh226/assettracking:1.0.1` image to run the asset tracking application.
    - Maps port `8080` on the host to port `8080` in the container.
    - Mounts a volume for configuration files.
    - Sets environment variables to configure the application.
    - Depends on `redis` and `dynamodb` services to be healthy and started before running.

### Additional Notes

- **Persistent Data**: The DynamoDB data is persisted in the `./dynamodblocal/data` directory on your local machine. Ensure this directory exists before running Docker Compose.
- **Configuration Files**: The application mounts the `./config` directory. Place any necessary configuration files in this directory before running the application.

By following these steps, you can set up and run the `assettracking-app` on your local machine and access the Swagger UI for API documentation and testing.

http://localhost:8080/swagger-ui/index.html

![alt text](http://url/to/img.png)
