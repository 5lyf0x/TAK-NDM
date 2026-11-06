# TAK Rig Navigation Data Manager (NDM)

**Self-hosted offline navigation, elevation, and cellular coverage data management for TAK Rig and TAK Map.**

TAK Rig Navigation Data Manager (NDM) builds, organizes, maintains, and distributes regional datasets that TAK Rig can use when internet connectivity is limited or unavailable. It combines road and trail source data, Valhalla-compatible routing graphs, elevation data, and cellular coverage datasets behind a browser-based management interface and local HTTP API.

> **Project status:** Active development. The latest project revision documented here is **v0.2.0-r16-r29**. The navigation/routing pipeline remains based on **v0.2.0-r16**.

## Features

### Navigation data and offline routing

- Build regional **Valhalla-compatible routing graphs** for use in offline navigation workflows.
- Integrate multiple geographic and road data sources, including **OpenStreetMap (OSM)** and applicable public-land or agency road/trail datasets.
- Incorporate road and track information from sources such as **USFS Motor Vehicle Use Maps (MVUM)**, **BLM**, **National Park Service**, and relevant state agencies where supported.
- Support selectable source configurations and regional build/rebuild workflows.
- Generate accompanying **navigation coverage** data describing the roads and tracks represented in a completed routing graph.
- Keep road and off-road track coverage distinct and apply access and vehicle-suitability filtering when processing source data.
- Package completed navigation data for local distribution and TAK Rig import.

### Terrain and elevation

- Manage elevation datasets independently of navigation graphs.
- Prepare regional terrain/elevation packages for offline applications.
- Provide elevation data to compatible TAK Rig terrain functions, including terrain visualization and analysis workflows.
- Support build, check, update/rebuild, and removal operations as provided by the relevant dataset workflow.
- Make completed elevation packages available through the local package catalog and download interface.

### Cellular coverage

- Manage **cellular coverage as a separate dataset**, independent of navigation and elevation.
- Support source acquisition and processing workflows based on **FCC Broadband Data Collection (BDC)** information.
- Organize carrier coverage for **AT&T, Verizon, and T-Mobile**, subject to source availability.
- Represent service classes including **Voice/3G, LTE, and 5G**.
- Prepare compact regional coverage data for compatible TAK Rig offline overlays.
- Display source/vintage information, acquisition progress, and package readiness in per-state **CELL** cards.
- Provide carrier/source details and supported import, acquisition, and package lifecycle controls.

### Web management dashboard

- **Executive Slate** dark interface, designed for desktop and smaller screens.
- State-oriented package management with dedicated **NAV**, **ELEV**, and **CELL** sections.
- State imagery, live status indicators, and package readiness information.
- Activity, package, and storage status panels.
- Source configuration and maintenance operations available through the WebUI.
- Progress visibility for acquisition and build operations, including supported pause/resume workflows.
- Locally served state artwork rather than requiring remote image hosting.

### Downloads and package management

- Regional packages with associated metadata and readiness states.
- HTTP downloads through the NDM service and catalog API.
- Resumable transfer support in applicable acquisition/download workflows.
- Parallel elevation acquisition with retry/fallback handling for transient upstream errors.
- Existing READY package preservation during supported update and recovery scenarios.
- Source and package integrity checks as part of supported build workflows.
- Upgrade-oriented installation designed to retain configuration, caches, and completed datasets.

### Source handling

NDM can combine supported public geographic sources into a routing-oriented dataset. Source support and completeness vary by state, provider, licensing terms, and availability. Depending on the region, inputs can include:

| Data source | Purpose |
| --- | --- |
| OpenStreetMap | General road and track geometry |
| USFS MVUM | Designated motor-vehicle routes on National Forest lands |
| Bureau of Land Management | Applicable public-land road and route information |
| National Park Service | Applicable park road and access information |
| State transportation / OHV agencies | Regional road and vehicle-access information |
| USGS / elevation providers | Elevation and terrain inputs |
| FCC BDC | Mobile broadband coverage inputs |

**Important:** A route appearing in a dataset does **not** establish that it is currently open, legal, safe, or passable. Always verify road restrictions, seasonal closures, land access, and conditions with the responsible authority.

## How NDM works

```text
Public geographic and coverage sources
                  |
                  v
       Acquire and validate
                  |
                  v
      Normalize / process data
                  |
       +----------+-----------+
       |          |           |
       v          v           v
  Navigation   Elevation   Cellular
  + Valhalla   packages    packages
       |          |           |
       +----------+-----------+
                  |
                  v
     Regional READY packages
                  |
                  v
        Local catalog / API
                  |
                  v
              TAK Rig
```

NDM is the **data preparation and distribution service**. TAK Rig is the client that downloads and consumes compatible packages. NDM does not require an internet routing API for navigation calculation once the appropriate offline graph is installed in a compatible client.

## Installation

NDM is intended for a Linux host with the required project dependencies and enough disk space for source data, working files, and completed packages. Install using the script included in the matching release archive.

1. Download the NDM source/release ZIP from this repository.
2. Extract it into a fresh, versioned directory.
3. Enter that extracted directory and run:

```bash
sudo ./install.sh
```

The installer is designed to install or update NDM while retaining persistent data and configuration. **Back up important datasets and configuration before upgrading**, particularly when moving between substantially different revisions.

### Service and WebUI

The NDM WebUI and local API use port **8765** by default. On the NDM host, open:

```text
http://127.0.0.1:8765
```

For a separate device on the same network, use the NDM host's reachable address and the configured listener/firewall settings. Do not expose an unauthenticated management interface directly to the public internet.

On systemd-based installations, the main service is:

```bash
sudo systemctl status takrig-nav-manager.service
```

Additional timers or helper services may be installed depending on the release.

## Using NDM

1. **Select a state or region.** Review the available NAV, ELEV, and CELL datasets.
2. **Review sources and settings.** Configure or inspect the supported navigation, elevation, or cellular data sources for the chosen region.
3. **Acquire or build.** Use the applicable build/download controls and follow progress in the dashboard.
4. **Verify readiness.** Check the package status and metadata before distribution.
5. **Download in TAK Rig.** Point TAK Rig at the NDM service, refresh its catalog, and download compatible regional packages.
6. **Maintain data.** Check, update, rebuild, or remove packages as needed without assuming that every dataset must be rebuilt together.

## TAK Rig integration

TAK Rig supports NDM discovery or a manually configured NDM server URL, catalog checks, and download of compatible regional data. The three dataset families serve different purposes:

| Package | TAK Rig use |
| --- | --- |
| **NAV** | Regional offline routing/navigation data |
| **ELEV** | Elevation, terrain display, and compatible terrain analysis |
| **CELL** | Offline mobile-network coverage visualization |

These packages have separate lifecycles so that users can obtain only the regional data they need.

## API and package structure

NDM provides a local HTTP API for status, catalog access, and supported package operations. For example:

```text
GET /api/v1/catalog
GET /api/v1/packages/CA/download
```

Actual routes, request parameters, and package schemas may differ by dataset family and software revision. Inspect the source and versioned documentation before building a third-party client.

Navigation packages are built around a Valhalla routing graph and associated metadata/coverage; elevation and cellular packages have their own formats. A downloaded package is **not interchangeable** with a general-purpose map tile archive.

## Project layout and storage

Typical Linux installations use persistent NDM data under:

```text
/var/lib/takrig-routing/
```

Completed navigation packages are typically staged under:

```text
/var/lib/takrig-routing/ready/
```

Elevation and other dataset families use their corresponding managed storage areas. File locations can change between releases; consult your installed configuration before scripting against the filesystem.

## Limitations and safety

- **Development status:** Features may change, and source-provider compatibility must be confirmed on the deployment host.
- **Coverage is not a guarantee:** Data availability, road classifications, cellular signal models, and elevation resolution vary by source and region.
- **Offline use requires preparation:** Download and verify all required packages *before* leaving connectivity.
- **Road legality and conditions:** Dataset content cannot replace current official access restrictions, closures, or on-the-ground judgment.
- **Data licensing:** Follow the attribution and redistribution requirements of OSM, agencies, FCC datasets, and any other source used.
- **Security:** Keep the management/API service on a trusted network or behind appropriate access controls.

## Development

NDM is actively developed alongside TAK Rig. Contributions and issue reports should include the NDM revision, Linux distribution, region/dataset, relevant service logs, and steps to reproduce the problem. Do not include secrets, credentials, or privately licensed datasets in reports.

## License and acknowledgments

**License:** Refer to the repository's `LICENSE` file, if provided. No open-source license is implied solely by publication of the source code.

NDM uses or processes data and technology from their respective maintainers and providers, including **OpenStreetMap**, **Valhalla**, public land-management agencies, **USGS**, and the **FCC**. Those projects and datasets retain their own licenses, terms, and attribution requirements.
