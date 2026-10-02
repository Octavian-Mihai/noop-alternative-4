# Architecture

A local-first WHOOP 4.0 client: an iOS app collects, decodes, stores and analyses strap data on-device, and optionally syncs to a self-hosted server. A single protocol schema drives decoding on both sides.

```mermaid
flowchart LR
    Strap[(WHOOP 4.0 strap)] <-->|BLE| BLE

    subgraph iOS["ios/OpenWhoop (SwiftUI + CoreBluetooth)"]
        BLE["BLE/<br/>BLEManager · FrameRouter"]
        Coll["Collect/<br/>Collector · Backfiller · clock policy"]
        Ana["Analysis/ + Metrics/<br/>HRV · recovery · sleep · strain"]
        UI["Tabs · Charts · Live · Alerts · Alarm"]
        Up["Upload/ + Sync/"]
        BLE --> Coll
        Coll --> Ana --> UI
    end

    Schema["protocol/whoop_protocol.json<br/>canonical decode schema"]
    PkgP["Packages/WhoopProtocol<br/>Swift decoder"]
    PkgS[("Packages/WhoopStore<br/>on-device GRDB/SQLite")]
    Coll --> PkgP
    Schema -.drives.-> PkgP
    Coll --> PkgS
    Ana --> PkgS
    PkgS --> UI
    PkgS --> Up

    subgraph Server["server/ (optional, self-hosted)"]
        Ingest["FastAPI ingest<br/>app/ingest.py · read.py"]
        PyP[whoop-protocol Python pkg]
        Analysis[analysis/ HRV · sleep · strain]
        TS[(TimescaleDB)]
        Ingest --> PyP --> TS
        Ingest --> Analysis --> TS
    end
    Schema -.drives.-> PyP
    Up -->|HTTPS + API key| Ingest

    Dash["dashboard/ (Mac BLE inspection tool)"] <-->|BLE| Strap
    RE["re/ · FINDINGS.md<br/>reverse-engineering notes"] -.informs.-> Schema
```
