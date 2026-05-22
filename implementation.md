# API Conventions Reference

<a name="toc"></a>

## Table of Contents
- [Overview](#overview)
- [Naming Conventions](#naming-conventions)
- [HTTP Methods](#http-methods)
- [Authentication](#authentication)
- [Request and Response Formats](#request-response-formats)
- [Status Codes](#status-codes)
- [Error Handling](#error-handling)
- [Pagination](#pagination)
- [Versioning](#versioning)
- [Rate Limiting](#rate-limiting)
- [Sorting](#sorting)


----------

<a name="overview"></a>

## Overview

This document establishes common conventions and standards applicable to all APIs. API-specific details (endpoints, query parameters, examples) are documented in separate API specification files.

----------

<a name="naming-conventions"></a>

## Naming Conventions

Establishes foundational naming rules for resources, fields, and parameters across all APIs.

**Resource Naming**

-   Use plural nouns for collections (e.g.,  `/eventforecasts`,  `/concurrentstreams`)
-   Use lowercase for resource paths
-   Avoid verbs in resource names (actions expressed through HTTP methods)

**Field Naming**

-   Use camelCase for JSON fields (e.g.,  `eventId`,  `contentId`,  `expectedPeak`)
-   Use lowercase for query parameters (e.g.,  `scheduledstart`,  `scheduledend`,  `lastmodifieddate`)
-   Boolean fields use  `is`,  `has`, or descriptive names (e.g.,  `tentative`,  `unplanned`)

----------

<a name="http-methods"></a>

## HTTP Methods

Defines how HTTP verbs map to operations on API resources.

**GET**  - Retrieve resources (idempotent, safe)

-   Collection:  `GET /resources`  with filtering/sorting/pagination
-   Individual:  `GET /resources/{id}`

**POST**  - Create resources (non-idempotent)

-   Returns 201 Created with Location header pointing to new resource

**PUT**  - Replace entire resource (idempotent)

-   Returns 200 OK or 204 No Content

**PATCH**  - Partial update (idempotent)

-   Modifies specified fields only

**DELETE**  - Remove resource (idempotent)

-   Returns 204 No Content

----------

<a name="authentication"></a>

## Authentication

Specifies security mechanisms for API access and authorization.

Authentication protocol is left to the discretion of the event creator, and should be discussed with API users a priori. The suggested protocol to utilize is OAuth 2.0 for secure communication. Include the access token in the Authorization header: Authorization: Bearer {token}.

**Mechanism:**  OAuth 2.0 (Client Credentials or Authorization Code grant)

**Required Headers:**

```
Authorization: Bearer {access_token}
Content-Type: application/json
Accept: application/json

```

**Optional Headers:**

```
Idempotency-Key: {uuid}
X-Request-ID: {uuid}

```

**Security Requirements:**

-   HTTPS only (TLS 1.2+)
-   Token expiration: 1 hour (access), 30 days (refresh)
-   Role-based access control as defined per API

**Error Responses:**

-   `401 Unauthorized`  - Missing/invalid credentials
-   `403 Forbidden`  - Insufficient permissions

----------

<a name="request-response-formats"></a>

## Request and Response Formats

Defines data structure, encoding, and format conventions for API communication.

**Content Type:**  `application/json; charset=utf-8`

**Date/Time Format:**  ISO 8601 UTC (`YYYY-MM-DDTHH:MM:SSZ`)

**Field Conventions:**

-   camelCase for JSON fields
-   Optional fields may be omitted
-   `null`  indicates explicit removal (PATCH operations)
-   Omitted fields in PATCH are not modified

**Response Headers:**

```
Content-Type: application/json; charset=utf-8
X-Request-ID: {uuid}
X-RateLimit-Limit: {number}
X-RateLimit-Remaining: {number}
X-RateLimit-Reset: {timestamp}

```

**Collection Response Structure:**

json

```json
{
  "data": [ ... ],
  "pagination": {
    "limit": 50,
    "offset": 0,
    "total": 1234,
    "hasMore": true
  }
}
```

----------

<a name="status-codes"></a>

## Status Codes

Follow standard HTTP status codes per RFC 7231.

**Success (2xx):**

-   `200 OK`  - Successful GET, PUT, PATCH
-   `201 Created`  - Successful POST with Location header
-   `204 No Content`  - Successful DELETE

**Client Errors (4xx):**

-   `400 Bad Request`  - Invalid JSON or missing required fields
-   `401 Unauthorized`  - Missing/invalid authentication
-   `403 Forbidden`  - Insufficient permissions
-   `404 Not Found`  - Resource does not exist
-   `409 Conflict`  - Duplicate resource identifier
-   `422 Unprocessable Entity`  - Invalid field values
-   `429 Too Many Requests`  - Rate limit exceeded

**Server Errors (5xx):**

-   `500 Internal Server Error`  - Unexpected error
-   `503 Service Unavailable`  - Temporary outage

----------

<a name="error-handling"></a>

## Error Handling

Defines standardized error response structure and common error codes.

**Standard Error Response:**

json

```json
{
  "error": {
    "code": "string",
    "message": "string",
    "details": [
      {
        "field": "string",
        "issue": "string"
      }
    ],
    "requestId": "string"
  }
}
```

**Common Error Codes:**

-   `INVALID_JSON`,  `MISSING_REQUIRED_FIELD`,  `INVALID_FIELD_VALUE`,  `INVALID_DATE_FORMAT`  (400)
-   `MISSING_CREDENTIALS`,  `INVALID_TOKEN`,  `EXPIRED_TOKEN`  (401)
-   `INSUFFICIENT_PERMISSIONS`  (403)
-   `RESOURCE_NOT_FOUND`  (404)
-   `DUPLICATE_RESOURCE`,  `RESOURCE_CONFLICT`  (409)
-   `RATE_LIMIT_EXCEEDED`  (429)
-   `INTERNAL_ERROR`,  `SERVICE_UNAVAILABLE`  (500/503)

**Best Practices:**

-   Include X-Request-ID for tracing
-   Provide actionable error messages
-   Include field-level details for validation errors
-   Never expose internal system details

----------

<a name="pagination"></a>

## Pagination

Defines standard approach for handling large result sets.

**Query Parameters:**

-   `limit={number}`  - Maximum results per page (default: 50, max: 100)
-   `offset={number}`  - Starting position (default: 0)

**Response Structure:**

json

```json
{
  "data": [ ... ],
  "pagination": {
    "limit": 50,
    "offset": 0,
    "total": 1234,
    "hasMore": true
  }
}
```

**Pagination Fields:**

-   `limit`  - Number of results in current page
-   `offset`  - Starting position of current page
-   `total`  - Total resources matching query
-   `hasMore`  - Boolean indicating more results exist

----------

<a name="versioning"></a>

## Versioning

Defines API evolution and backward compatibility strategy.

**Strategy:**  URI-based versioning with major version in base path (e.g.,  `/v1/`,  `/v2/`)

**Version Policy:**

-   Major version only (v1, v2)
-   Increment for breaking changes (field removal, type changes, behavior changes)
-   Maintain backward compatibility within same major version
-   Support previous versions for 12 months minimum
-   Provide 6-month deprecation notice

----------

<a name="rate-limiting"></a>

## Rate Limiting

Defines request throttling policies to protect API resources and ensure fair usage.

**Default Thresholds:**

-   1,000 requests/hour per client
-   100 requests/minute burst limit

**Response Headers:**

```
X-RateLimit-Limit: {number}
X-RateLimit-Remaining: {number}
X-RateLimit-Reset: {timestamp}

```

**429 Response:**

json

```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded. Retry after {seconds} seconds.",
    "requestId": "..."
  }
}
```

Includes  `Retry-After: {seconds}`  header.

**Best Practices:**

-   Monitor X-RateLimit-Remaining header
-   Implement exponential backoff for 429 responses
-   Cache responses to reduce API calls

----------

<a name="sorting"></a>

## Sorting

Defines standard approach for ordering result sets.

**Query Parameters:**

-   `sort={field}`  - Field to sort by
-   `order=asc|desc`  - Sort direction (default: asc)

**Example:**

```
GET /resources?sort=createdDate&order=desc
```
