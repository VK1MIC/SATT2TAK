# SATT2TAK

A Node-RED flow that tracks satellites in real time and publishes their position, coverage footprint, and 60-minute ground track as Cursor-on-Target (CoT) events to a TAK server.

## What it does

- Fetches live satellite positions from N2YO API every 60 seconds (ISS, SONATE-2)
- Fetches TLE data from SatNOGS every hour for four satellites (ISS, SONATE-2, LILIUM-4, PARUS-6U1)
- Uses SGP4 propagation (satellite.js) to compute position every 10 seconds for TLE-tracked satellites
- Builds three CoT events per satellite:
  - Point marker at current position (CoT type `a-f-P-S`)
  - Coverage footprint polygon (closed polyline, cyan fill)
  - 60-minute ground track (open polyline, red)
- Handles antimeridian crossing by splitting geometries into segments
- Transmits all events to a TAK server over TLS TCP

## Prerequisites

- Node-RED with the following nodes installed:
  - Built-in: `xml`, `tcp out`
  - npm module available to Node-RED: `satellite.js`
- A running TAK server accessible over TCP with TLS
- A valid TAK client certificate and key
- A free N2YO API key from https://www.n2yo.com/api/

## Configuration

Edit the following values in the `N2YO Build Requests` function node before importing:

| Variable | Description |
|---|---|
| `YOUR_N2YO_API_KEY` | Your N2YO API key |
| `YOUR_OBS_LAT` | Observer latitude in decimal degrees (e.g. `-35.28`) |
| `YOUR_OBS_LON` | Observer longitude in decimal degrees (e.g. `149.13`) |
| `YOUR_OBS_ALT` | Observer altitude in metres above sea level (e.g. `600`) |

And in the `tcp out` and `tls-config` nodes:

| Node | Field | Description |
|---|---|---|
| `tcp out` (TAK Server) | Host | Your TAK server hostname or IP |
| `tcp out` (TAK Server) | Port | Your TAK server TCP port (typically 8089) |
| `tls-config` | Certificate name | Your TAK client certificate filename (.pem) |
| `tls-config` | Key name | Your TAK client key filename (.key) |

## Satellite sources

| Satellite | NORAD ID | Position source |
|---|---|---|
| ISS | 25544 | N2YO (live) + SatNOGS TLE (SGP4) |
| SONATE-2 | 59112 | N2YO (live) + SatNOGS TLE (SGP4) |
| LILIUM-4 | 98391 | SatNOGS TLE (SGP4) only |
| PARUS-6U1 | 98392 | SatNOGS TLE (SGP4) only |

## Installing satellite.js in Node-RED

From your Node-RED user directory (typically `~/.node-red`):

```bash
npm install satellite.js
```

Then restart Node-RED.
