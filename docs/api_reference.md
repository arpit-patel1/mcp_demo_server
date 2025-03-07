# MCP Server API Reference

This document provides a comprehensive reference for the MCP Server API endpoints, request/response formats, and error handling.

## API Conventions

### Base URL

All API endpoints are prefixed with:

```
https://{server-address}/api/v1/
```

### Authentication

All requests must include authentication using one of the following methods:

**JWT Token (Preferred)**
```
Authorization: Bearer {jwt-token}
```

**API Key**
```
X-API-Key: {api-key}
```

### Response Format

All responses follow a standard format:

**Success Response**
```json
{
  "status": "success",
  "data": {
    // Response data specific to the endpoint
  },
  "metadata": {
    // Optional metadata (pagination, etc.)
  }
}
```

**Error Response**
```json
{
  "status": "error",
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable error message",
    "details": {
      // Optional additional error details
    }
  }
}
```

### Pagination

List endpoints support pagination with the following query parameters:

- `page`: Page number (1-based)
- `limit`: Number of items per page
- `sort`: Field to sort by
- `order`: Sort order (`asc` or `desc`)

Paginated responses include metadata:

```json
{
  "status": "success",
  "data": [...],
  "metadata": {
    "page": 1,
    "limit": 10,
    "total": 42,
    "pages": 5
  }
}
```

### Filtering

List endpoints support filtering with query parameters:

- Simple filters: `?field=value`
- Multiple filters: `?field1=value1&field2=value2`
- Operators: `?field[operator]=value` (where operator can be `eq`, `ne`, `gt`, `lt`, `gte`, `lte`, `in`, `nin`, `regex`)

Example: `?status[in]=running,pending&created_at[gt]=2023-01-01`

## Core API Endpoints

### Authentication

#### POST /auth/login

Authenticate a user and receive a JWT token.

**Request**
```json
{
  "username": "string",
  "password": "string"
}
```

**Response**
```json
{
  "status": "success",
  "data": {
    "access_token": "string",
    "refresh_token": "string",
    "token_type": "Bearer",
    "expires_in": 3600
  }
}
```

#### POST /auth/refresh

Refresh an expired JWT token.

**Request**
```json
{
  "refresh_token": "string"
}
```

**Response**
```json
{
  "status": "success",
  "data": {
    "access_token": "string",
    "refresh_token": "string",
    "token_type": "Bearer",
    "expires_in": 3600
  }
}
```

#### POST /auth/logout

Invalidate the current JWT token.

**Request**
```
No request body required
```

**Response**
```json
{
  "status": "success",
  "data": {
    "message": "Successfully logged out"
  }
}
```

### User Management

#### GET /users

List all users (admin only).

**Response**
```json
{
  "status": "success",
  "data": [
    {
      "id": "string",
      "username": "string",
      "email": "string",
      "role": "string",
      "created_at": "string",
      "last_login": "string"
    }
  ],
  "metadata": {
    "page": 1,
    "limit": 10,
    "total": 42,
    "pages": 5
  }
}
```

#### POST /users

Create a new user (admin only).

**Request**
```json
{
  "username": "string",
  "email": "string",
  "password": "string",
  "role": "string"
}
```

**Response**
```json
{
  "status": "success",
  "data": {
    "id": "string",
    "username": "string",
    "email": "string",
    "role": "string",
    "created_at": "string"
  }
}
```

#### GET /users/{user_id}

Get details for a specific user.

**Response**
```json
{
  "status": "success",
  "data": {
    "id": "string",
    "username": "string",
    "email": "string",
    "role": "string",
    "created_at": "string",
    "last_login": "string"
  }
}
```

#### PUT /users/{user_id}

Update a user.

**Request**
```json
{
  "email": "string",
  "password": "string",
  "role": "string"
}
```

**Response**
```json
{
  "status": "success",
  "data": {
    "id": "string",
    "username": "string",
    "email": "string",
    "role": "string",
    "updated_at": "string"
  }
}
```

#### DELETE /users/{user_id}

Delete a user.

**Response**
```json
{
  "status": "success",
  "data": {
    "message": "User successfully deleted"
  }
}
```

### API Keys

#### GET /api-keys

List all API keys for the current user.

**Response**
```json
{
  "status": "success",
  "data": [
    {
      "id": "string",
      "name": "string",
      "prefix": "string",
      "created_at": "string",
      "expires_at": "string",
      "last_used": "string"
    }
  ]
}
```

#### POST /api-keys

Create a new API key.

**Request**
```json
{
  "name": "string",
  "expires_in": 0  // Optional, seconds until expiration
}
```

**Response**
```json
{
  "status": "success",
  "data": {
    "id": "string",
    "name": "string",
    "key": "string",  // Full API key, only shown once
    "prefix": "string",
    "created_at": "string",
    "expires_at": "string"
  }
}
```

#### DELETE /api-keys/{key_id}

Delete an API key.

**Response**
```json
{
  "status": "success",
  "data": {
    "message": "API key successfully deleted"
  }
}
```

## Kubernetes Module API

### Clusters

#### GET /kubernetes/clusters

List all Kubernetes clusters.

**Response**
```json
{
  "status": "success",
  "data": [
    {
      "id": "string",
      "name": "string",
      "description": "string",
      "status": "connected|disconnected|error",
      "version": "string",
      "nodes_count": 0,
      "created_at": "string",
      "last_connected": "string"
    }
  ],
  "metadata": {
    "page": 1,
    "limit": 10,
    "total": 42,
    "pages": 5
  }
}
```

#### POST /kubernetes/clusters

Add a new Kubernetes cluster.

**Request**
```json
{
  "name": "string",
  "description": "string",
  "config": {
    "kubeconfig": "string",  // Base64 encoded kubeconfig
    // OR
    "host": "string",
    "token": "string",
    "ca_cert": "string"  // Base64 encoded CA certificate
  }
}
```

**Response**
```json
{
  "status": "success",
  "data": {
    "id": "string",
    "name": "string",
    "description": "string",
    "status": "connected",
    "version": "string",
    "nodes_count": 0,
    "created_at": "string"
  }
}
```

#### GET /kubernetes/clusters/{cluster_id}

Get details for a specific Kubernetes cluster.

**Response**
```json
{
  "status": "success",
  "data": {
    "id": "string",
    "name": "string",
    "description": "string",
    "status": "connected|disconnected|error",
    "version": "string",
    "nodes_count": 0,
    "created_at": "string",
    "last_connected": "string",
    "error": "string",  // Only present if status is "error"
    "nodes": [
      {
        "name": "string",
        "status": "string",
        "roles": ["string"],
        "version": "string",
        "cpu": "string",
        "memory": "string"
      }
    ]
  }
}
```

#### PUT /kubernetes/clusters/{cluster_id}

Update a Kubernetes cluster.

**Request**
```json
{
  "name": "string",
  "description": "string",
  "config": {
    "kubeconfig": "string",  // Base64 encoded kubeconfig
    // OR
    "host": "string",
    "token": "string",
    "ca_cert": "string"  // Base64 encoded CA certificate
  }
}
```

**Response**
```json
{
  "status": "success",
  "data": {
    "id": "string",
    "name": "string",
    "description": "string",
    "status": "connected",
    "updated_at": "string"
  }
}
```

#### DELETE /kubernetes/clusters/{cluster_id}

Remove a Kubernetes cluster.

**Response**
```json
{
  "status": "success",
  "data": {
    "message": "Cluster successfully removed"
  }
}
```

### Namespaces

#### GET /kubernetes/{cluster_id}/namespaces

List all namespaces in a cluster.

**Response**
```json
{
  "status": "success",
  "data": [
    {
      "name": "string",
      "status": "string",
      "created_at": "string",
      "labels": {
        "key": "value"
      },
      "annotations": {
        "key": "value"
      }
    }
  ]
}
```

#### POST /kubernetes/{cluster_id}/namespaces

Create a new namespace.

**Request**
```json
{
  "name": "string",
  "labels": {
    "key": "value"
  },
  "annotations": {
    "key": "value"
  }
}
```

**Response**
```json
{
  "status": "success",
  "data": {
    "name": "string",
    "status": "string",
    "created_at": "string",
    "labels": {
      "key": "value"
    },
    "annotations": {
      "key": "value"
    }
  }
}
```

### Pods

#### GET /kubernetes/{cluster_id}/namespaces/{namespace}/pods

List all pods in a namespace.

**Response**
```json
{
  "status": "success",
  "data": [
    {
      "name": "string",
      "status": "string",
      "phase": "string",
      "ip": "string",
      "node": "string",
      "created_at": "string",
      "containers": [
        {
          "name": "string",
          "image": "string",
          "ready": true,
          "restart_count": 0,
          "state": "string"
        }
      ]
    }
  ],
  "metadata": {
    "page": 1,
    "limit": 10,
    "total": 42,
    "pages": 5
  }
}
```

#### GET /kubernetes/{cluster_id}/namespaces/{namespace}/pods/{pod_name}

Get details for a specific pod.

**Response**
```json
{
  "status": "success",
  "data": {
    "name": "string",
    "status": "string",
    "phase": "string",
    "ip": "string",
    "node": "string",
    "created_at": "string",
    "containers": [
      {
        "name": "string",
        "image": "string",
        "ready": true,
        "restart_count": 0,
        "state": "string",
        "ports": [
          {
            "container_port": 0,
            "protocol": "string"
          }
        ],
        "resources": {
          "requests": {
            "cpu": "string",
            "memory": "string"
          },
          "limits": {
            "cpu": "string",
            "memory": "string"
          }
        }
      }
    ],
    "labels": {
      "key": "value"
    },
    "annotations": {
      "key": "value"
    },
    "volumes": [
      {
        "name": "string",
        "type": "string",
        "details": {}
      }
    ]
  }
}
```

#### GET /kubernetes/{cluster_id}/namespaces/{namespace}/pods/{pod_name}/logs

Get logs for a pod.

**Query Parameters**
- `container`: Container name (required if pod has multiple containers)
- `tail_lines`: Number of lines to return from the end
- `since_seconds`: Return logs newer than a relative duration in seconds
- `timestamps`: Include timestamps on each line
- `follow`: Stream logs (returns chunked response)

**Response**
```json
{
  "status": "success",
  "data": {
    "logs": "string"
  }
}
```

#### POST /kubernetes/{cluster_id}/namespaces/{namespace}/pods/{pod_name}/exec

Execute a command in a pod.

**Request**
```json
{
  "container": "string",  // Optional if pod has only one container
  "command": ["string"],
  "stdin": "string",      // Optional
  "tty": true            // Optional
}
```

**Response**
```json
{
  "status": "success",
  "data": {
    "stdout": "string",
    "stderr": "string",
    "exit_code": 0
  }
}
```

### Deployments

#### GET /kubernetes/{cluster_id}/namespaces/{namespace}/deployments

List all deployments in a namespace.

**Response**
```json
{
  "status": "success",
  "data": [
    {
      "name": "string",
      "replicas": 0,
      "available_replicas": 0,
      "strategy": "string",
      "created_at": "string",
      "labels": {
        "key": "value"
      }
    }
  ],
  "metadata": {
    "page": 1,
    "limit": 10,
    "total": 42,
    "pages": 5
  }
}
```

#### POST /kubernetes/{cluster_id}/namespaces/{namespace}/deployments

Create a new deployment.

**Request**
```json
{
  "name": "string",
  "replicas": 0,
  "selector": {
    "match_labels": {
      "key": "value"
    }
  },
  "template": {
    "metadata": {
      "labels": {
        "key": "value"
      }
    },
    "spec": {
      "containers": [
        {
          "name": "string",
          "image": "string",
          "ports": [
            {
              "container_port": 0,
              "protocol": "string"
            }
          ],
          "resources": {
            "requests": {
              "cpu": "string",
              "memory": "string"
            },
            "limits": {
              "cpu": "string",
              "memory": "string"
            }
          },
          "env": [
            {
              "name": "string",
              "value": "string"
            }
          ],
          "volume_mounts": [
            {
              "name": "string",
              "mount_path": "string",
              "read_only": true
            }
          ]
        }
      ],
      "volumes": [
        {
          "name": "string",
          "config_map": {
            "name": "string"
          }
          // Other volume types supported
        }
      ]
    }
  }
}
```

**Response**
```json
{
  "status": "success",
  "data": {
    "name": "string",
    "replicas": 0,
    "available_replicas": 0,
    "strategy": "string",
    "created_at": "string",
    "labels": {
      "key": "value"
    }
  }
}
```

#### PUT /kubernetes/{cluster_id}/namespaces/{namespace}/deployments/{deployment_name}/scale

Scale a deployment.

**Request**
```json
{
  "replicas": 0
}
```

**Response**
```json
{
  "status": "success",
  "data": {
    "name": "string",
    "replicas": 0,
    "available_replicas": 0,
    "updated_at": "string"
  }
}
```

## Network Module API

### Devices

#### GET /network/devices

List all network devices.

**Response**
```json
{
  "status": "success",
  "data": [
    {
      "id": "string",
      "name": "string",
      "type": "cisco_ios|cisco_nxos|juniper_junos",
      "hostname": "string",
      "status": "connected|disconnected|error",
      "created_at": "string",
      "last_connected": "string"
    }
  ],
  "metadata": {
    "page": 1,
    "limit": 10,
    "total": 42,
    "pages": 5
  }
}
```

#### POST /network/devices

Add a new network device.

**Request**
```json
{
  "name": "string",
  "type": "cisco_ios|cisco_nxos|juniper_junos",
  "hostname": "string",
  "port": 22,
  "credentials": {
    "username": "string",
    "password": "string",
    // OR
    "ssh_key": "string"  // Base64 encoded private key
  },
  "connection_options": {
    "timeout": 0,
    "keepalive": 0,
    "auto_connect": true
  }
}
```

**Response**
```json
{
  "status": "success",
  "data": {
    "id": "string",
    "name": "string",
    "type": "cisco_ios|cisco_nxos|juniper_junos",
    "hostname": "string",
    "status": "connected|disconnected",
    "created_at": "string"
  }
}
```

#### GET /network/devices/{device_id}

Get details for a specific network device.

**Response**
```json
{
  "status": "success",
  "data": {
    "id": "string",
    "name": "string",
    "type": "cisco_ios|cisco_nxos|juniper_junos",
    "hostname": "string",
    "port": 22,
    "status": "connected|disconnected|error",
    "created_at": "string",
    "last_connected": "string",
    "error": "string",  // Only present if status is "error"
    "system_info": {
      "model": "string",
      "serial": "string",
      "os_version": "string",
      "uptime": "string"
    }
  }
}
```

#### PUT /network/devices/{device_id}

Update a network device.

**Request**
```json
{
  "name": "string",
  "hostname": "string",
  "port": 22,
  "credentials": {
    "username": "string",
    "password": "string",
    // OR
    "ssh_key": "string"  // Base64 encoded private key
  },
  "connection_options": {
    "timeout": 0,
    "keepalive": 0,
    "auto_connect": true
  }
}
```

**Response**
```json
{
  "status": "success",
  "data": {
    "id": "string",
    "name": "string",
    "hostname": "string",
    "status": "connected|disconnected",
    "updated_at": "string"
  }
}
```

#### DELETE /network/devices/{device_id}

Remove a network device.

**Response**
```json
{
  "status": "success",
  "data": {
    "message": "Device successfully removed"
  }
}
```

### Device Operations

#### POST /network/devices/{device_id}/connect

Connect to a network device.

**Response**
```json
{
  "status": "success",
  "data": {
    "id": "string",
    "name": "string",
    "status": "connected",
    "connected_at": "string"
  }
}
```

#### POST /network/devices/{device_id}/disconnect

Disconnect from a network device.

**Response**
```json
{
  "status": "success",
  "data": {
    "id": "string",
    "name": "string",
    "status": "disconnected",
    "disconnected_at": "string"
  }
}
```

#### POST /network/devices/{device_id}/command

Execute a command on a network device.

**Request**
```json
{
  "command": "string",
  "timeout": 0  // Optional
}
```

**Response**
```json
{
  "status": "success",
  "data": {
    "command": "string",
    "output": "string",
    "executed_at": "string"
  }
}
```

#### POST /network/devices/{device_id}/commands

Execute multiple commands on a network device.

**Request**
```json
{
  "commands": ["string"],
  "timeout": 0  // Optional
}
```

**Response**
```json
{
  "status": "success",
  "data": {
    "results": [
      {
        "command": "string",
        "output": "string",
        "success": true,
        "error": "string"  // Only present if success is false
      }
    ],
    "executed_at": "string"
  }
}
```

### Configuration Management

#### GET /network/devices/{device_id}/config

Get the configuration of a network device.

**Query Parameters**
- `format`: Output format (`text`, `json`, `xml`)
- `section`: Configuration section to retrieve

**Response**
```json
{
  "status": "success",
  "data": {
    "config": "string",
    "format": "text|json|xml",
    "retrieved_at": "string"
  }
}
```

#### PUT /network/devices/{device_id}/config

Update the configuration of a network device.

**Request**
```json
{
  "config": "string",
  "format": "text|json|xml",
  "replace": false,  // Whether to replace the entire config or merge
  "commit": true     // Whether to commit changes immediately
}
```

**Response**
```json
{
  "status": "success",
  "data": {
    "message": "Configuration updated successfully",
    "updated_at": "string"
  }
}
```

#### POST /network/devices/{device_id}/config/commit

Commit configuration changes (for devices that support transaction model).

**Response**
```json
{
  "status": "success",
  "data": {
    "message": "Configuration committed successfully",
    "committed_at": "string"
  }
}
```

#### POST /network/devices/{device_id}/config/rollback

Rollback configuration changes (for devices that support transaction model).

**Response**
```json
{
  "status": "success",
  "data": {
    "message": "Configuration rolled back successfully",
    "rolled_back_at": "string"
  }
}
```

#### POST /network/devices/{device_id}/config/backup

Create a backup of the device configuration.

**Request**
```json
{
  "description": "string"  // Optional
}
```

**Response**
```json
{
  "status": "success",
  "data": {
    "backup_id": "string",
    "description": "string",
    "created_at": "string",
    "size": 0
  }
}
```

#### GET /network/devices/{device_id}/config/backups

List configuration backups for a device.

**Response**
```json
{
  "status": "success",
  "data": [
    {
      "backup_id": "string",
      "description": "string",
      "created_at": "string",
      "size": 0
    }
  ]
}
```

#### POST /network/devices/{device_id}/config/restore

Restore a configuration backup.

**Request**
```json
{
  "backup_id": "string",
  "commit": true  // Whether to commit changes immediately
}
```

**Response**
```json
{
  "status": "success",
  "data": {
    "message": "Configuration restored successfully",
    "restored_at": "string"
  }
}
```

## Error Codes

The MCP Server uses standardized error codes:

| Code | Description |
|------|-------------|
| `AUTH_INVALID_CREDENTIALS` | Invalid username or password |
| `AUTH_TOKEN_EXPIRED` | JWT token has expired |
| `AUTH_INSUFFICIENT_PERMISSIONS` | User lacks required permissions |
| `RESOURCE_NOT_FOUND` | Requested resource does not exist |
| `RESOURCE_ALREADY_EXISTS` | Resource already exists |
| `VALIDATION_ERROR` | Request validation failed |
| `K8S_CONNECTION_ERROR` | Failed to connect to Kubernetes cluster |
| `K8S_OPERATION_ERROR` | Kubernetes operation failed |
| `NETWORK_CONNECTION_ERROR` | Failed to connect to network device |
| `NETWORK_COMMAND_ERROR` | Network command execution failed |
| `NETWORK_CONFIG_ERROR` | Network configuration operation failed |
| `INTERNAL_SERVER_ERROR` | Unexpected server error |

## Rate Limiting

The MCP Server implements rate limiting to prevent abuse:

- Default limit: 100 requests per minute per user
- Kubernetes operations: 50 requests per minute per user
- Network device operations: 30 requests per minute per user

Rate limit headers are included in all responses:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1620000000
```

When rate limit is exceeded, a 429 Too Many Requests response is returned:

```json
{
  "status": "error",
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded. Try again in 30 seconds.",
    "details": {
      "retry_after": 30
    }
  }
}
``` 