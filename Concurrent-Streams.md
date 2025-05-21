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
  - [json Examples](#json)

# Introduction <a name="intro"></a>
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
| sdp       | string | optional    | Streams Data Provider. This is the content owner or distributor for which live viewership data is being requested. Recommended only when the endpoint provides data from multiple providers, and the requestor needs to filter to specific VDPs. |

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
| content    | object        |          | See: Object: Content in ORTB            |
| eventstart | integer       | yes      | Event start time (Unix timestamp in ms) |
| eventend   | integer       | yes      | Event end time (Unix timestamp in ms)   |
| streamsdata    | object, array |          | Contains separate SSAI and CSAI metrics |

### Object: StreamCount <a name="streamcount"></a>

| Attribute | Type    | Required    | Description                                                                    |
| --------- | ------- | ----------- | ------------------------------------------------------------------------------ |
| region    | integer |             | Region codes: 1 - N\_America\_East, 2 - N\_America\_West, etc.                 |
| sstreams  | integer | recommended | Concurrent SSAI streams. At least one of sstreams or cstreams must be present. |
| cstreams  | integer | yes         | Concurrent CSAI streams. At least one of sstreams or cstreams must be present. |

### Object: Content <a name="content"></a>

| Attribute          | Type               | Description                         |
| ------------------ | ------------------ | ----------------------------------- |
| id                 | string             | Unique content ID                   |
| episode            | integer            | Episode number                      |
| title              | string             | Content title                       |
| series             | string             | Content series                      |
| season             | string             | Content season                      |
| artist             | string             | Artist credited                     |
| gtax               | int; default 9     | Taxonomy in use                     |
| genres             | int array          | Genre categories per taxonomy       |
| album              | string             | Album name (for audio)              |
| isrc               | string             | ISO-3901 code                       |
| producer           | object             | Content producer details            |
| url                | string             | Content URL                         |
| cattax             | integer; default 1 | Taxonomy in use for cat field       |
| cat                | string array       | Content categories                  |
| prodq              | integer            | Production quality                  |
| context            | integer            | Type of content (game, video, etc.) |
| contentrating      | string             | Content rating (e.g., MPAA)         |
| userrating         | string             | User rating                         |
| qagmediarating     | integer            | Media rating per IQG                |
| keywords           | string             | Comma-separated keywords            |
| kwarray            | string array       | Array of keywords                   |
| livestream         | integer            | 0 = not live, 1 = live content      |
| sourcerelationship | integer            | 0 = indirect, 1 = direct            |
| len                | integer            | Content length in seconds           |
| language           | string             | ISO-639-1-alpha-2 code              |
| langb              | string             | IETF BCP 47 language code           |
| embeddable         | integer            | 0 = no, 1 = yes                     |
| data               | object array       | Additional content data             |
| network            | object             | Network details                     |
| channel            | object             | Channel details                     |
| ext                | object             | Exchange-specific extensions        |


# Implementation Guidance <a name="impguide"></a>

## General Information <a name="geninfo"></a>
It is expected that some security model or authentication mechanism is in place prior to the API being called to ensure only authorized subscribers are able to request or receive the data.

This API is decoupled from the Request/Response in OpenRTB.

The region is expected to represent the endpoint traffic distribution across different geographic areas. Coarse granularity acceptable for MVP and is expected to drive system scaling in regional datacenters. E.g. US West Coast.

SSAI includes SGAI

As a best practice, it is recommended that viewership values be representative of users at the live edge/timepoint.

## Content Information <a name="contentinfo"></a>
The attributes under the <code>mediastream</code> object provide various options for providers to declare a live event, and so that consumers of this data can identify that event in other contexts 
A provider can sufficiently identify a specific live event with one or more of the optional attributes from the [<code>content</code> object](#content), each of which is paired with the required attributes of <code>eventstart</code> and <code>eventend</code>:
- Content ID (E.g., global/persistent ID ABC123, or synthetic ID XYZ-456, either of which points to a specific NBA game)
- Title (E.g., NBA Basketball: Lakers vs. Celtics)
- Series (E.g., NBA Basketball)
- Channel: (E.g., ESPN)

It is strongly recommended that some identifier for the content in question is passed. This could correspond to the content owners Content Management System (CMS), and/or some other commercially available content identifier. Whatever content ID is provided, should be the same as the ID that is provided on the Bid Request. Additional information about the Content can be provided in the Extended Content IDs array. 


## json Examples <a name="json"></a>

```
{
  "version": "1.0.0",
  "timestamp": "1713366138",
  "viewership-data": [
    {
      "vdp": "TV Network A",
      "mediastreams": [
        {
          "content": {
            "id": "CMS123",
            "genres": [646],
            "gtax": 9,
            "contentrating": "PG-13",
            "channe": {"name": "Comedy Channel"},
            "data": {
              "name": "vendorA.com",
              "cids": ["alpha", "beta", "gamma", "delta", "epsilon"]
            }
          },
          "eventstart": "1713366132",
          "eventend": "1713378132",
          "viewership": [
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
          "viewership": [
            {"region": "1", "sstreams": "150000", "cstreams": "500000"},
            {"region": "2", "sstreams": "130000", "cstreams": "23000"},
            {"region": "3", "sstreams": "520000", "cstreams": "12000"}
          ]
        }
      ]
    },
    {
      "vdp": "TV Network B",
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
          "viewership": [
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
          "viewership": [
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
