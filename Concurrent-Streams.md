# About the IAB Tech Lab <a name="techlab"></a>
The IAB Technology Laboratory is a nonprofit research and development consortium charged with producing and helping companies implement global industry technical standards and solutions. The goal of the Tech Lab is to reduce friction associated with the digital advertising and marketing supply chain while contributing to the safe growth of an industry. The IAB Tech Lab spearheads the development of technical standards, creates and maintains a code library to assist in rapid, cost-effective implementation of IAB standards, and establishes a test platform for companies to evaluate the compatibility of their technology solutions with IAB standards, which for 18 years have been the foundation for interoperability and profitable growth in the digital advertising supply chain. Further details about the IAB Technology Lab can be found at www.iabtechlab.com
### IAB Tech Lab Lead:

Hillary Slattery, Sr. Director of Product, IAB Tech Lab

### License <a name="license"></a>

This specification is licensed under a Creative Commons Attribution 3.0 License. To view a copy of this license, visit creativecommons.org/licenses/by/3.0/ http://creativecommons.org/licenses/by/3.0/ or write to Creative Commons, 171 Second Street, Suite 300, San Francisco, CA 94105, USA.

### Disclaimer <a name="disclaimer"></a>

THE STANDARDS, THE SPECIFICATIONS, THE MEASUREMENT GUIDELINES, AND ANY OTHER MATERIALS OR SERVICES PROVIDED TO OR USED BY YOU HEREUNDER (THE “PRODUCTS AND SERVICES”) ARE PROVIDED “AS IS” AND “AS AVAILABLE,” AND IAB TECHNOLOGY LABORATORY, INC. (“TECH LAB”) MAKES NO WARRANTY WITH RESPECT TO THE SAME AND HEREBY DISCLAIMS ANY AND ALL EXPRESS, IMPLIED, OR STATUTORY WARRANTIES, INCLUDING, WITHOUT LIMITATION, ANY WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE, AVAILABILITY, ERROR-FREE OR UNINTERRUPTED OPERATION, AND ANY WARRANTIES ARISING FROM A COURSE OF DEALING, COURSE OF PERFORMANCE, OR USAGE OF TRADE. TO THE EXTENT THAT TECH LAB MAY NOT AS A MATTER OF APPLICABLE LAW DISCLAIM ANY IMPLIED WARRANTY, THE SCOPE AND DURATION OF SUCH WARRANTY WILL BE THE MINIMUM PERMITTED UNDER SUCH LAW. THE PRODUCTS AND SERVICES DO NOT CONSTITUTE BUSINESS OR LEGAL ADVICE. TECH LAB DOES NOT WARRANT THAT THE PRODUCTS AND SERVICES PROVIDED TO OR USED BY YOU HEREUNDER SHALL CAUSE YOU AND/OR YOUR PRODUCTS OR SERVICES TO BE IN COMPLIANCE WITH ANY APPLICABLE LAWS, REGULATIONS, OR SELF-REGULATORY FRAMEWORKS, AND YOU ARE SOLELY RESPONSIBLE FOR COMPLIANCE WITH THE SAME, INCLUDING, BUT NOT LIMITED TO, DATA PROTECTION LAWS, SUCH AS THE PERSONAL INFORMATION PROTECTION AND ELECTRONIC DOCUMENTS ACT (CANADA), THE DATA PROTECTION DIRECTIVE (EU), THE E-PRIVACY DIRECTIVE (EU), THE GENERAL DATA PROTECTION REGULATION (EU), AND THE E-PRIVACY REGULATION (EU) AS AND WHEN THEY BECOME EFFECTIVE.

# Table of Contents
- [About the IAB Tech Lab](#techlab)
- [License](#license)
- [Disclaimer](#disclaimer)
- [Introduction](#intro)
- [Concurrent Streams Request](#streamsrequest)
- [Concurrent Streams Request](#streamsresponse)
  - [Object: StreamsData](#streamsdata)
  - [Object: MediaStreams](#mediastreams)
  - [Object: StreamCount](#streamcount)
  - [Object: Content](#content)
- [Implementation Guidance](#impguide)
  - [General Information](#geninfo)
  - [Content Information](#contentinfo)
  - [Base URI](#base-uri)
  - [Endpoints](#endpoints)
  - [Query Parameters](#query-parameters)
  - [Path Parameters](#path-parameters)
  - [Request Examples](#request-examples)
  - [Response Examples](#response-examples)
  - [Usage Guidance](#usage-guidance)
  - [Role-Based Access](#role-based-access)
  - [Integration with Event Forecasts API](#integration)


# Introduction <a name="intro"></a>
This is version 1.0 of the Concurrent Streams specification and every attempt will be made to make future versions backward compatible if possible.

## Executive Summary
Due to challenges of forecasting live inventory, and dynamic viewership changes that occur during the live event playout, advertising systems may be inappropriately scaled to process ad decision transactions, causing poor UX and lost revenue opportunities for DSPs, SSPs, pub ad servers, and publishers. 

This API aims to give a standard way for subscribers to ask for information on how many viewers are in a stream when a live event is occurring at the time of ping.

## Business Problem
SSAIs: Need a standard way to provide viewership signals to subscriber systems, so that they can receive ad decisions back in a timely, low-latency manner to improve user experience and minimize slate during a live event. 

Subscribers to the API on the sell side such as Publishers, Publisher Ad Servers, and/or SSPs need a way to understand near real time viewership data to ensure optimal monetization, avoid slate, and scale their capacity to be able to handle spikes in traffic.

Subscribers on the buy side such as DSPs and advertisers need to adjust their infrastructure to manage QPS spikes during an event to ensure valuable inventory doesn’t get blocked due to hitting upper limits on QPS, ensure effective campaign pacing and adjust their bids appropriately. All of this will help advertisers increase advertising opportunities during live events they’re interested in.


## Concurrent Streams Request <a name="streamsrequest"></a>

| Attribute | Type   | Required    | Description                                                                                                                                                                                                                                      |
| --------- | ------ | ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| version   | string | yes         | E.g., 1.0.0                                                                                                                                                                                                                                      |
| requestor | string | recommended | Authorized party requesting information from the endpoint. Not required if the endpoint allows anonymous requests.                                                                                                                               |
| sdp       | string | optional    | Streams Data Provider. This is the content owner or distributor for which live viewership data is being requested. Recommended only when the endpoint provides data from multiple providers, and the requestor needs to filter to specific Streams Data Providerss. |

## Concurrent Streams Response <a name="streamsresponse"></a>

| Attribute    | Type          | Required | Description                                                |
| ------------ | ------------- | -------- | ---------------------------------------------------------- |
| version      | string        | yes      | E.g., 1.0.0                                                |
| timestamp    | integer       | yes      | Date/time of the snapshot (Unix timestamp in milliseconds) |
| streamsdata | object, array | yes      | Streams data requested by the caller                       |

### Object: StreamsData <a name="streamsdata"></a>

| Attribute    | Type          | Required | Description                                                                          |
| ------------ | ------------- | -------- | ------------------------------------------------------------------------------------ |
| sdp          | string        |          | Streams Data Provider                                                                |
| mediastreams | object, array | yes      | Snapshot of viewership data for live events. Each mediastream includes stream count. |

### Object: MediaStreams <a name="mediastreams"></a>

| Attribute  | Type          | Required | Description                             |
| ---------- | ------------- | -------- | --------------------------------------- |
| content    | object        |          | See: [Object: Content from AdCOM 1.0](https://github.com/InteractiveAdvertisingBureau/AdCOM/blob/main/AdCOM%20v1.0%20FINAL.md#object--content-) |
| eventstart | integer       | yes      | Event start time (Unix timestamp in ms) |
| eventend   | integer       | yes      | Event end time (Unix timestamp in ms)   |
| streamcount    | object, array |          | Contains separate SSAI and CSAI metrics |

### Object: StreamCount <a name="streamcount"></a>

| Attribute | Type    | Required    | Description                                                                    |
| --------- | ------- | ----------- | ------------------------------------------------------------------------------ |
| region    | integer |             | Region codes: 1 - N\_America\_East, 2 - N\_America\_West, etc.                 |
| sstreams  | integer |  | Concurrent SSAI streams. At least one of sstreams or cstreams must be present. |
| cstreams  | integer |          | Concurrent CSAI streams. At least one of sstreams or cstreams must be present. |

### Object: Content <a name="content"></a>
Refer to [Object: Content from AdCOM 1.0](https://github.com/InteractiveAdvertisingBureau/AdCOM/blob/main/AdCOM%20v1.0%20FINAL.md#object--content-) for specific values.


# Implementation Guidance <a name="impguide"></a>

## General Information <a name="geninfo"></a>
It is expected that some security model or authentication mechanism is in place prior to the API being called to ensure only authorized subscribers are able to request or receive the data.

This API is decoupled from the Request/Response in OpenRTB.

The region is expected to represent the endpoint traffic distribution across different geographic areas. Coarse granularity acceptable for MVP and is expected to drive system scaling in regional datacenters. E.g. US West Coast.

SSAI includes SGAI. Streamers leveraging Server Guided Ad Insertion models should leverage ‘SSAI’ workflows. 

As a best practice, it is recommended that viewership values be representative of users at the live edge/timepoint.

Requests will be coming from an API consumer to SSAI vendors on behalf of the Content Owner. 

When API is called, metadata will be returned for all live events the Subscriber has access to.

Event name and/or ID is sharable with API subscribers at run time and/or event name is known to subscribers of the API <i>a priori</i> to the request for concurent streams 
App information is known <i>a priori</i> to the request 

Providers are expected to return information for all Live Events occurring at time of API request.

## Content Information <a name="contentinfo"></a>
The attributes under the <code>mediastream</code> object provide various options for providers to declare a live event, and so that consumers of this data can identify that event in other contexts 
A provider can sufficiently identify a specific live event with one or more of the optional attributes from the [<code>content</code> object](#content), each of which is paired with the required attributes of <code>eventstart</code> and <code>eventend</code>:
- Content ID (E.g., global/persistent ID ABC123, or synthetic ID XYZ-456, either of which points to a specific NBA game)
- Title (E.g., NBA Basketball: Lakers vs. Celtics)
- Series (E.g., NBA Basketball)
- Channel: (E.g., ESPN)

It is strongly recommended that some identifier for the content in question is passed. This could correspond to the content owners Content Management System (CMS), and/or some other commercially available content identifier. Whatever content ID is provided, should be the same as the ID that is provided on the Bid Request. Additional information about the Content can be provided in the Extended Content IDs array. 

## Base URI <a name="base-uri"></a>

```
https://api.example.com/v1/concurrentstreams

```

----------

## Endpoints <a name="endpoints"></a>

### Collection Endpoints

**`GET /concurrentstreams`**

-   Retrieves current concurrent stream data for all active live events
-   Returns near real-time viewership signals
-   Supports filtering by requestor and Streams Data Provider (SDP)

**`GET /concurrentstreams/{contentId}`**

-   Retrieves concurrent stream data for specific content
-   Returns detailed viewership breakdown by stream type and region
-   Includes SSAI and CSAI stream counts

----------

## Query Parameters <a name="query-parameters"></a>

### Filtering Parameters

#### Table

| Parameter | Type | Description |
|-----------|------|-------------|
| `requestor` | String | Identifies the requesting entity (SSAI, publisher, DSP, advertiser) |
| `sdp` | String | Optional Streams Data Provider identifier for multi-provider scenarios |
| `region` | Integer | Filters stream data by region code (1=N_America_East, 2=N_America_West, 3=Europe, etc.) |

### Pagination Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `limit` | Integer | Maximum results per page (default: 50, max: 100) |
| `offset` | Integer | Starting position (default: 0) |

----------

## Path Parameters <a name="path-parameters"></a>

| Parameter | Type | Description |
|-----------|------|-------------|
| `{contentId}` | String | Unique content identifier (CMS ID or persistent identifier) |

----------

## Request Examples <a name="request-examples"></a>

### Get All Concurrent Streams

```http
GET /v1/concurrentstreams?requestor=example-ssai
Authorization: Bearer {access_token}
```

### Get Concurrent Streams for Specific Content

```http
GET /v1/concurrentstreams/CMS123?requestor=example-ssai
Authorization: Bearer {access_token}
```

### Filter by Region

```http
GET /v1/concurrentstreams?requestor=example-ssai&region=1
Authorization: Bearer {access_token}
```

----------
## Response Examples <a name="response-examples"></a>

### Single Content Response

```json
{
  "version": "1.0.0",
  "timestamp": "1738785540000",
  "streamsdata": [
    {
      "sdp": "TV Network A",
      "mediastreams": [
        {
          "content": {
            "id": "CMS123",
            "genres": [646],
            "gtax": 9,
            "contentrating": "PG-13",
            "channel": {"name": "Comedy Channel"}
          },
          "eventstart": "1738785000000",
          "eventend": "1738796800000",
          "streamcount": [
            {"region": 1, "sstreams": 140000, "cstreams": 400000},
            {"region": 2, "sstreams": 100000, "cstreams": 20000},
            {"region": 3, "sstreams": 500000, "cstreams": 10000}
          ]
        }
      ]
    }
  ]
}
```

### Multi-Event Response

```json
{
  "version": "1.0.0",
  "timestamp": "1713366138",
  "streamsdata": [
    {
      "sdp": "TV Network A",
      "mediastreams": [
        {
          "content": {
            "id": "CMS123",
            "genres": [646],
            "gtax": 9,
            "contentrating": "PG-13",
            "channel": {"name": "Comedy Channel"},
            "data": {
              "name": "vendorA.com",
              "cids": ["alpha", "beta", "gamma", "delta", "epsilon"]
            }
          },
          "eventstart": "1713366132",
          "eventend": "1713378132",
          "streamcount": [
            {"region": "1", "sstreams": "140000", "cstreams": "400000"},
            {"region": "2", "sstreams": "100000", "cstreams": "20000"},
            {"region": "3", "sstreams": "500000", "cstreams": "10000"}
          ]
        },
        {
          "content": {
            "id": "CMS456",
            "genres": [162],
            "gtax": 9,
            "contentrating": "PG-13",
            "channel": {"name": "WXYZ Channel"},
            "data": {
              "name": "vendorA.com",
              "cids": ["ayyy", "beee", "ceee", "deee", "eeee"]
            }
          },
          "eventstart": "1713366132",
          "eventend": "1713378132",
          "streamcount": [
            {"region": "1", "sstreams": "150000", "cstreams": "500000"},
            {"region": "2", "sstreams": "130000", "cstreams": "23000"},
            {"region": "3", "sstreams": "520000", "cstreams": "12000"}
          ]
        }
      ]
    },
    {
      "sdp": "TV Network B",
      "mediastreams": [
        {
          "content": {
            "id": "CMS789",
            "genres": [483],
            "gtax": 9,
            "contentrating": "PG",
            "channel": {"name": "Sports Now!"},
            "data": {
              "name": "vendorA.com",
              "cids": ["zeta", "eta", "theta", "iota", "kappa"]
            }
          },
          "eventstart": "1713366132",
          "eventend": "1713378132",
          "streamcount": [
            {"region": "1", "sstreams": "140050", "cstreams": "400050"},
            {"region": "2", "sstreams": "100060", "cstreams": "20060"},
            {"region": "3", "sstreams": "500070", "cstreams": "10070"}
          ]
        },
        {
          "content": {
            "id": "CMS101112",
            "genres": [483],
            "gtax": 9,
            "contentrating": "PG",
            "channel": {"name": "Sports 8 (The Ocho)"},
            "data": {
              "name": "vendorA.com",
              "cids": ["efff", "geee", "aych", "aiii", "jayy"]
            }
          },
          "eventstart": "1713366132",
          "eventend": "1713378132",
          "streamcount": [
            {"region": "1", "sstreams": "150050", "cstreams": "500050"},
            {"region": "2", "sstreams": "130060", "cstreams": "23060"},
            {"region": "3", "sstreams": "520070", "cstreams": "12070"}
          ]
        }
      ]
    }
  ]
}
```
----------

## Usage Guidance <a name="usage-guidance"></a>

### Data Freshness

-   API provides  **near real-time**  viewership data
-   Typical latency: 30-60 seconds from actual viewership
-   Subscribers should poll at appropriate intervals (recommended: 1-5 minutes)
-   Avoid excessive polling to respect rate limits

### Content Identification

-   Use  **CMS IDs**  or other persistent identifiers for content
-   Content identifiers should remain consistent across API calls
-   Map content IDs to internal systems for correlation
-   Content ID should match the ID provided in bid requests

### Viewership Values

-   Stream counts represent users at the  **live edge**
-   Excludes DVR/time-shifted viewing
-   Counts are point-in-time snapshots, not cumulative
-   Regional breakdowns enable geographic scaling decisions

### Region Codes

-   Region codes are integers representing geographic areas
-   Example codes: 1=N_America_East, 2=N_America_West, 3=Europe
-   Coarse granularity acceptable for MVP
-   Drives system scaling in regional datacenters


### Scaling Recommendations

**Infrastructure Scaling:**

-   Monitor stream count trends (increasing/decreasing)
-   Scale proactively when counts exceed 80% of capacity
-   Implement auto-scaling based on stream count thresholds

**Bidding Strategy:**

-   Higher viewership = higher competition = adjust bids accordingly
-   Use historical patterns to predict viewership curves

**QPS Management:**

-   Calculate expected QPS: (concurrent streams × ad requests per stream)
-   Scale ad serving infrastructure before reaching capacity
-   Implement circuit breakers for overload protection

----------

## Role-Based Access <a name="role-based-access"></a>

**SSAI Provider (Read Access)**

-   Read concurrent stream data
-   Query by content and region
-   Monitor viewership for infrastructure scaling

**Publisher (Read Access)**

-   Read concurrent stream data for owned content
-   Monitor viewership for monetization optimization
-   Access historical viewership patterns

**DSP/Advertiser (Read Access)**

-   Read concurrent stream data
-   Adjust bidding strategies based on viewership
-   Optimize campaign pacing

**Streams Data Provider (Write + Read Access)**

-   Publish concurrent stream data (if applicable)
-   Read all stream data
-   Manage data quality and accuracy

----------

## Integration with Event Forecasts API <a name="integration"></a>

The Concurrent Streams API complements the Event Forecasts API (upcoming):

**Event Forecasts API**  - Provides advance notice of upcoming events with forecasted viewership

**Concurrent Streams API**  - Provides real-time viewership during live events

**Recommended Integration Pattern:**

1.  Use Event Forecasts API to prepare infrastructure for upcoming events
2.  Switch to Concurrent Streams API once event goes live
3.  Compare actual viewership (Concurrent Streams) vs. forecast (Event Forecasts) for accuracy tuning
4.  Use historical concurrent stream data to improve future forecasts

