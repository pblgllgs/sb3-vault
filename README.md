# Using Hashicorp Vault to store secrets in Spring boot

## Dependencies for Spring boot

Vault
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-vault-config</artifactId>
    </dependency>
</dependencies>
```

Bootstrap configs
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-bootstrap</artifactId>
    </dependency>
</dependencies>
```

## Run Vault server

### Docker

```bash
docker run -p 8200:8200 -e 'VAULT_DEV_ROOT_TOKEN_ID=dev-only-token' vault
```

## Basic docker compose

```yml
version: '3.6'
services:

  vault:
    image: hashicorp/vault:1.15
    container_name: vault-server
    ports:
      - "8200:8200" # Puerto por defecto de Vault
    environment:
      VAULT_DEV_ROOT_TOKEN_ID: "root-token" # Token de acceso de desarrollo para pruebas
    command: server -dev -dev-root-token-id=root-token
```

## Run

```bash
docker compose up -d
```

## Set token in environment

```bash
export VAULT_TOKEN="dev-only-token"
```

File postman send-variables-vault-server.postman_collection.json

```json
{
  "info": {
    "_postman_id": "4081c163-69cb-4db0-a82f-f7de8be2625c",
    "name": "send-variables-vault-server",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json",
    "_exporter_id": "17450698",
    "_collection_link": "https://bold-station-345144.postman.co/workspace/VAULT~cc37b850-5207-48e5-b954-8bf872d63fb2/collection/17450698-4081c163-69cb-4db0-a82f-f7de8be2625c?action=share&source=collection_link&creator=17450698"
  },
  "item": [
    {
      "name": "send",
      "request": {
        "method": "POST",
        "header": [
          {
            "key": "X-Vault-Token",
            "value": "root-token",
            "type": "text"
          },
          {
            "key": "Content-Type",
            "value": "application/json",
            "type": "text"
          }
        ],
        "body": {
          "mode": "raw",
          "raw": "{\r\n    \"data\":\r\n    {\r\n        \"spring.datasource.username\": \"root\",\r\n        \"spring.datasource.password\": \"password\"\r\n    }\r\n}",
          "options": {
            "raw": {
              "language": "json"
            }
          }
        },
        "url": {
          "raw": "{{base_url}}/v1/secret/data/{{app}}",
          "host": [
            "{{base_url}}"
          ],
          "path": [
            "v1",
            "secret",
            "data",
            "{{app}}"
          ]
        }
      },
      "response": []
    }
  ]
}
```

Curl request

```bash
curl \
    --header "X-Vault-Token: $VAULT_TOKEN" \
    --header "Content-Type: application/json" \
    --request POST \
    --data '{"data": {"spring.datasource.username": "root","spring-datasource.password":"password"}}' \
    http://127.0.0.1:8200/v1/secret/data/sb3-vault
```