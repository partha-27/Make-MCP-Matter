# ToofanRoute Dispatch

**Team:** CodeForge\
**Track:** T-1\
**Event:** wHACKiest 2026\
**Project:** Weather-Responsive Fleet Dispatch Agent

ToofanRoute Dispatch is a proposed MCP-powered decision-support system
for small delivery fleets. It connects weather information, active fleet
routes, route-impact analysis, and dispatcher-approved notifications in
one workflow.

> **Safety note:** ToofanRoute flags routes that may be affected by
> weather. It does not certify roads as safe, predict flooding, or
> autonomously reroute vehicles. A dispatcher reviews and approves
> operational actions.

## Problem

Small courier and last-mile delivery teams often use separate tools for
weather, vehicle routes, and driver communication. During severe-weather
events, dispatchers may need to compare forecasts with active routes
manually and then contact drivers individually. This fragmented process
can delay decisions and make it difficult to maintain a clear record of
what happened.

## Proposed solution

ToofanRoute monitors weather forecasts and available alerts for a
fleet's operating area, compares affected geographic areas with active
delivery routes, and presents potentially affected vehicles to a
dispatcher. The dispatcher can review the evidence and approve or reject
a proposed notification.

## Core workflow

1.  **Register fleet data** --- store vehicles, drivers, active
    deliveries, and route geometry.
2.  **Monitor weather** --- retrieve forecasts and available weather
    alerts from a weather provider.
3.  **Detect an event** --- identify a new or changed weather condition
    and retain its source and timestamp.
4.  **Orchestrate with MCP** --- the AI agent selects tools from the
    weather, fleet, and messaging MCP servers.
5.  **Assess route impact** --- deterministic spatial logic checks
    whether active route geometry intersects the available
    weather-affected area.
6.  **Review in the dashboard** --- show the event, affected routes,
    evidence, timestamps, and uncertainty.
7.  **Approve or reject** --- a dispatcher decides whether a
    notification should be sent.
8.  **Notify and record** --- send an approved message and record the
    decision and delivery status.

## MCP architecture

The system is organised around three MCP servers coordinated by an AI
agent (MCP host).

### 1. Weather MCP server

Connects to a weather data provider and exposes tools such as:

-   `get_current_weather`
-   `get_rainfall_forecast`
-   `get_weather_alerts`
-   `get_weather_location`

### 2. Fleet database MCP server

Connects to the fleet's SQLite database and exposes tools such as:

-   `get_active_vehicles`
-   `get_vehicle_details`
-   `get_vehicle_route`
-   `find_affected_routes`
-   `record_weather_event`
-   `update_dispatch_status`

### 3. Messaging MCP server

Connects to a notification platform and exposes tools such as:

-   `prepare_alert`
-   `send_dispatch_alert`
-   `notify_driver`
-   `get_message_status`

Tool names above describe the proposed interface; adjust them to match
the actual implementation.

## Technology stack

  -----------------------------------------------------------------------
  Layer                   Proposed technology     Purpose
  ----------------------- ----------------------- -----------------------
  Programming language    Python                  Backend and MCP servers

  Dashboard               Streamlit               Fleet and dispatcher
                                                  interface

  Map                     Folium / GeoJSON        Display routes and
                                                  weather areas

  Agent                   Python + an LLM API     Interpret context and
                                                  coordinate tool calls

  MCP                     Official MCP Python SDK Expose tools to the
                                                  agent

  Weather data            Weather API             Forecasts and available
                                                  alerts

  Database                SQLite                  Vehicles, routes,
                                                  events, and decisions

  Spatial analysis        Shapely                 Route and weather-area
                                                  intersection

  Notifications           Discord or Slack API    Send approved dispatch
                                                  alerts
  -----------------------------------------------------------------------

The AI agent coordinates tools and explains results. Spatial
calculations should be performed by deterministic code rather than
delegated to an LLM.

## Repository structure

A suggested structure for the implementation:

``` text
toofanroute/
├── README.md
├── app/
│   ├── dashboard.py
│   └── components/
├── agent/
│   └── orchestrator.py
├── mcp_servers/
│   ├── weather_server.py
│   ├── fleet_server.py
│   └── messaging_server.py
├── services/
│   ├── weather_service.py
│   ├── route_impact.py
│   └── notification_service.py
├── database/
│   ├── schema.sql
│   └── fleet.db
├── data/
│   └── sample_routes.geojson
├── tests/
├── .env.example
└── requirements.txt
```

This is a recommended layout; rename or remove folders to reflect the
code actually present in the repository.

## Getting started

### Prerequisites

-   Python 3.10 or later (confirm the version supported by the selected
    MCP SDK).
-   `pip` and a virtual environment.
-   API credentials for the selected weather provider.
-   An LLM API key if using a hosted model.
-   A Discord or Slack webhook if enabling real notifications.

### 1. Clone the repository

``` bash
git clone <YOUR_REPOSITORY_URL>
cd toofanroute
```

### 2. Create and activate a virtual environment

**Windows PowerShell**

``` powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

**macOS / Linux**

``` bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies

Once `requirements.txt` has been created:

``` bash
pip install -r requirements.txt
```

### 4. Configure environment variables

Create a `.env` file in the project root. Use only the variables
required by the integrations you implement.

``` dotenv
WEATHER_API_KEY=
LLM_API_KEY=
DISCORD_WEBHOOK_URL=
DATABASE_PATH=database/fleet.db
```

Do not commit `.env` or real credentials to version control. Add `.env`
to `.gitignore`.

### 5. Initialise the database and run the app

Use the actual initialisation and launch commands defined by your
implementation. For a Streamlit dashboard, the launch command will
typically look like:

``` bash
streamlit run app/dashboard.py
```

The exact command depends on the final repository structure.

## Route-impact assessment

The route-impact engine should:

1.  Represent a delivery route as a LineString or equivalent route
    geometry.
2.  Represent the weather-affected area as a polygon or another suitable
    spatial region when the provider supplies one.
3.  Check for a geometric intersection.
4.  Calculate useful overlap information, where the input data supports
    it.
5.  consider event severity, data timestamp, and geographic resolution.
6.  Return an explainable result for dispatcher review.

A city-level weather observation alone is not sufficient evidence that a
specific road segment is affected. If precise weather polygons or
road-condition data are unavailable, the dashboard should clearly label
the assessment as approximate.

## Demo scenario

A controlled demonstration can use a small sample fleet and a clearly
labelled simulated weather event:

1.  Display three active delivery vehicles and their routes.
2.  Trigger a simulated heavy-rain event over part of the map.
3.  Show the weather MCP tool result and timestamp.
4.  Show the fleet MCP retrieving active routes.
5.  Display the route-impact analysis and potentially affected
    vehicle(s).
6.  Open the dispatcher review step.
7.  Approve a notification and show the messaging result.
8.  Show the event and decision in the activity log.

Clearly distinguish live API responses from simulated demo data.

## Safety, reliability, and limitations

-   A weather/route intersection indicates potential exposure; it does
    not prove a road is unsafe.
-   Do not automatically change a driver's route based only on an
    AI-generated recommendation.
-   Require dispatcher approval before sending operational instructions.
-   Show source timestamps and indicate stale or missing data.
-   Handle API errors, rate limits, duplicate alerts, and notification
    failures.
-   Use cached or simulated data only when clearly labelled.
-   Alternative routes should only be presented as options when
    supported by a routing service and suitable road-condition
    information.
-   Do not claim live Doppler radar, flood prediction, cyclone
    telemetry, or guaranteed safe routing unless those capabilities are
    genuinely implemented and validated.

## MVP scope

The initial prototype should focus on:

-   One operating area.
-   A small sample fleet and a few active routes.
-   One weather data source.
-   Three MCP servers: weather, fleet, and messaging.
-   Deterministic route-impact analysis.
-   An interactive dashboard.
-   Dispatcher approval and one notification channel.
-   An activity/audit log.

Advanced forecasting, global cyclone tracking, enterprise fleet
integrations, and autonomous routing are future extensions rather than
MVP requirements.

## Team

  Name             USN            Responsibility
  ---------------- -------------- ----------------
  Skanda           `1DT25EC120`   To be assigned
  Partha Sarathi   `1DT25CI045`   To be assigned
  Lalan            `1DT25CI051`   To be assigned
  B Sanjan         `1DT25CD013`   To be assigned

Update the responsibility column once the team finalises task ownership.

## Future enhancements

-   Additional weather providers and geographic regions.
-   Verified road-closure and traffic information.
-   More fleet-management integrations.
-   Improved alert deduplication and event history.
-   Configurable fleet-specific alert thresholds.
-   Analytics on alert response and dispatch coordination.

## Project status

This README documents the proposed architecture and MVP workflow.
Replace proposed components, tool names, commands, and environment
variables with the exact details of the working implementation before
publishing the repository.

## License

Add the license selected by the team before public distribution.
