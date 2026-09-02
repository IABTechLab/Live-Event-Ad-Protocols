![IAB Tech Lab](https://drive.google.com/uc?id=10yoBoG5uRETSXRrnJPUDuONujvADrSG1)
# Live Event Ad Playbook (LEAP)
# Multiple API Guidance
# Planning and Real-Time Adjustment

## Table of Contents

- [Purpose](#purpose)
- [Audience](#audience)
- [API Roles](#api-roles)
- [Key Operating Principle](#key-operating-principle)
- [Cross-API Identity Correlation](#cross-api-identity-correlation)
- [Example Scenario](#example-scenario)
- [Pre-Event Forecast Data](#pre-event-forecast-data)
- [Pre-Event Buyer Planning](#pre-event-buyer-planning)
- [Transition at Event Start](#transition-at-event-start)
- [Example Live Audience Curve](#example-live-audience-curve)
- [Concurrent Streams Example Response](#concurrent-streams-example-response)
- [Live Buyer Adjustment Actions](#live-buyer-adjustment-actions)
- [Suggested Polling Guidance](#suggested-polling-guidance)
- [End-to-End Workflow Summary](#end-to-end-workflow-summary)
- [Best Practices](#best-practices)
- [What Not to Do](#what-not-to-do)

## Purpose

This document explains how advertising buyers and other monetization partners can use multiple LEAP APIs together during the live-event monetization lifecycle.

The intended workflow is:

- Forecast API -> Pre-event planning
- Concurrent Streams API -> Live-event adjustment

Forecast API helps participants plan before an event begins.

Concurrent Streams API helps participants adjust while the event is live.

## Audience

This document is intended for:

- advertisers
- DSPs
- SSPs
- exchanges
- publishers
- streamers
- ad servers
- SSAI and SGAI platforms
- planning and forecasting systems

## API Roles

| API | Primary Timeframe | Primary Function |
|---|---|---|
| Forecast API | Before event start | Future event discovery, audience forecasting, inventory planning, deal planning, and infrastructure preparation. |
| Concurrent Streams API | During live event | Near-real-time audience monitoring, QPS scaling, bid adjustment, and pacing optimization. |

## Key Operating Principle

Forecast API data should be treated as an estimate of future supply.

Concurrent Streams API data should be treated as a live operational signal.

Once the event is live, Concurrent Streams API should be used for current concurrency signals. Forecast API may still be useful as a baseline, but it should not be treated as the authoritative signal for current live audience size.

## Cross-API Identity Correlation

To use both APIs together, systems must be able to determine that a forecasted event and a live stream-count record refer to the same event.

Recommended identity hierarchy:

- `content.id`
- `content.data.cids`
- Forecast API event `id`, when aligned with `content.id`
- A deterministic fallback based on title, series, provider, and event timing

### Recommended Pattern

Forecast API response:

```json
{
  "id": "event-content-12345",
  "content": {
    "id": "event-content-12345",
    "data": {
      "name": "id-provider.example",
      "cids": ["shared-content-id-12345"]
    }
  }
}
```

Concurrent Streams API response:

```json
{
  "content": {
    "id": "event-content-12345",
    "data": {
      "name": "id-provider.example",
      "cids": ["shared-content-id-12345"]
    }
  }
}
```

OpenRTB bid requests and reporting systems should use the same identifier where possible.

## Example Scenario

A buyer is preparing a campaign for a generic championship sporting event.

The buyer wants to:

- reserve budget before the event
- estimate available impression opportunities
- prepare QPS capacity
- adjust bids and pacing during live audience spikes
- reduce lost opportunity during high-demand moments

No real companies, leagues, or teams are used in this example.

## Pre-Event Forecast Data

Example Forecast API response:

```json
{
  "version": "1.0.0",
  "timestamp": 1751205600000,
  "events": [
    {
      "id": "event-content-12345",
      "scheduledstart": 1752000000000,
      "scheduledend": 1752014400000,
      "eventstatus": 1,
      "content": {
        "id": "event-content-12345",
        "title": "Championship Event",
        "series": "Generic Sports Series",
        "data": {
          "name": "id-provider.example",
          "cids": ["shared-content-id-12345"]
        }
      },
      "streamsdata": [
        {
          "country": "USA",
          "expectedpeak": 1800000,
          "lowerbound": 1200000
        }
      ],
      "inventoryconfig": {
        "supportedmtype": [2],
        "totaladdurationsec": 1200,
        "expectedpodcount": 10,
        "unplanned": 1
      },
      "lastmodifieddate": 1751202000000
    }
  ]
}
```

## Pre-Event Buyer Planning

Assumptions:

| Input | Value |
|---|---:|
| Expected peak streams | 1,800,000 |
| Lower-bound streams | 1,200,000 |
| Planning average concurrency | 1,300,000 |
| Expected ad pods | 10 |
| Average ads per pod | 4 |
| Estimated event duration | 4 hours |

Estimated impression opportunities:

```text
1,300,000 average concurrent streams x 10 pods x 4 ads = 52,000,000 estimated impression opportunities
```

Buyer use cases before the event:

| Timeframe | Forecast API Use | Buyer Action |
|---|---|---|
| 30+ days | Discover future event and expected peak scale. | Reserve budget and identify planning opportunity. |
| 14 days | Compare lower bound and expected peak. | Set conservative and aggressive delivery plans. |
| 7 days | Review updated forecast and inventory configuration. | Refine pacing assumptions and QPS expectations. |
| 24 hours | Use latest forecast as pre-event baseline. | Warm infrastructure and finalize campaign controls. |

## Transition at Event Start

At event start, the buyer transitions from planning mode to live adjustment mode.

```text
Before event: Forecast API baseline
During event: Concurrent Streams API live signal
```

The original forecast remains useful as a baseline for comparison, but actual live behavior may differ materially.

## Example Live Audience Curve

| Event Phase | Forecast Baseline | Actual Concurrent Streams | Interpretation |
|---|---:|---:|---|
| Pre-event warmup | 500,000 | 400,000 | Below forecast; keep pacing conservative. |
| Event start | 1,000,000 | 1,100,000 | Close to expected; begin normal delivery. |
| Early event | 1,300,000 | 1,400,000 | Slightly above expected; monitor QPS. |
| Intermission | 1,500,000 | 1,900,000 | Spike above expected; increase capacity and pacing. |
| Final segment | 1,800,000 | 2,400,000 | Major upside spike; prioritize high-value inventory. |
| Post-event | 700,000 | 600,000 | Wind down pacing and infrastructure. |

## Concurrent Streams Example Response

```json
{
  "version": "1.0.0",
  "timestamp": 1752007200000,
  "streamsdata": [
    {
      "sdp": "provider-1",
      "mediastreams": [
        {
          "content": {
            "id": "event-content-12345",
            "title": "Championship Event",
            "data": {
              "name": "id-provider.example",
              "cids": ["shared-content-id-12345"]
            }
          },
          "eventstart": 1752000000000,
          "eventend": 1752014400000,
          "streamcount": [
            {
              "region": 1,
              "sstreams": 1500000,
              "cstreams": 400000
            },
            {
              "region": 2,
              "sstreams": 400000,
              "cstreams": 100000
            }
          ]
        }
      ]
    }
  ]
}
```

Total current streams in this example:

- Region 1: 1,500,000 SSAI + 400,000 CSAI = 1,900,000
- Region 2: 400,000 SSAI + 100,000 CSAI = 500,000
- Total: 2,400,000 concurrent streams

## Live Buyer Adjustment Actions

When current streams exceed forecast:

- increase QPS capacity
- raise bid throttles where appropriate
- increase pacing aggressiveness
- prioritize event-specific deals or packages
- avoid blocking valuable bid opportunities due to capacity ceilings

When current streams fall below forecast:

- reduce pacing aggressiveness
- preserve budget for later event phases
- avoid over-allocating QPS capacity
- shift spend to other available inventory if needed

When current streams spike regionally:

- scale regional endpoints or bidding capacity
- update regional throttles
- prioritize latency-sensitive systems

## Suggested Polling Guidance

These intervals are starting points for working group review.

| API | Event State | Possible Polling Frequency |
|---|---|---|
| Forecast API | 30+ days before event | Daily |
| Forecast API | 14 days before event | Every 12 hours |
| Forecast API | 7 days before event | Every 6 hours |
| Forecast API | 24 hours before event | Hourly |
| Concurrent Streams API | Pre-event warmup | Every 5 minutes |
| Concurrent Streams API | Live event | Every 30-60 seconds |
| Concurrent Streams API | Peak moments | Every 15-30 seconds |
| Concurrent Streams API | Post-event wind-down | Every 5 minutes |

Polling should be tuned based on:

- event scale
- provider rate limits
- buyer operational sensitivity
- infrastructure cost
- signal freshness requirements

## End-to-End Workflow Summary

| Stage | API | Primary Decision |
|---|---|---|
| Event discovery | Forecast API | Should the buyer plan around this event? |
| Pre-event planning | Forecast API | What budget, deal, and QPS assumptions are needed? |
| Final readiness | Forecast API | What is the latest expected audience and inventory baseline? |
| Live monitoring | Concurrent Streams API | How many streams are active now? |
| Live optimization | Concurrent Streams API | Should bids, pacing, or infrastructure be adjusted? |
| Post-event review | Both APIs plus reporting | How did actual audience compare to forecast? |

## Best Practices

Participants should:

- align `content.id` and `content.data.cids` across APIs
- treat forecasts as probabilistic estimates
- use Concurrent Streams API for live operational decisions
- define polling expectations before major events
- handle stale snapshots gracefully
- implement exponential backoff and respect rate limits
- avoid synchronized polling by many clients at the same instant
- preserve forecast snapshots for post-event variance analysis

## What Not to Do

Participants should not:

- assume forecasts will match live concurrency exactly
- use Forecast API as the authoritative current-stream signal during the event
- ignore identifier mismatches across APIs
- apply long cache TTLs to live concurrency data
- rely on titles alone for event matching when stable identifiers are available
