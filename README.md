
# SentinelOne SDL Dashboards

This repository contains a curated collection of **SentinelOne Singularity Data Lake (SDL) / Dataset** dashboards, expressed as JSON, for common data sources (for example Fastly CDN, Okta, O365, Fortinet).

Each dashboard follows a consistent **query style**, **layout approach**, and **performance guardrails**, so you can safely reuse and extend them.

---

## Repository Layout

A typical structure looks like:

```text
.
├── README.md                 # This file – global conventions and usage
├── fastly/
│   ├── README.md             # Fastly-specific docs
│   └── fastly-cdn-dashboard.json
├── okta/
│   ├── README.md
│   └── okta-auth-dashboard.json
└── o365/
    ├── README.md
    └── o365-alerts-dashboard.json
```

* The **root README** (this file) documents **shared conventions** and how to work with dashboard JSON in general.
* Each **subfolder README** documents:
  * Data‑source–specific assumptions (fields, normalization, OCSF mapping).
  * What tabs/widgets exist and what questions they answer.
  * Any quirks for that data source.

---

## Requirements

To use dashboards from this repo you need:

* A **SentinelOne Singularity Data Lake / Dataset** environment.
* Relevant data sources ingested and normalized (ideally OCSF‑aligned where appropriate).
* Permissions to **view and manage custom dashboards**.
* For data that should appear in the **XDR** view (and therefore in these dashboards), each event **must** include `dataSource.name`, `dataSource.vendor`, and `dataSource.category`. Dashboards that rely on XDR data are intended to be used with the **XDR** or **ALL** view selected in the console.

For dashboard JSON behavior and syntax, see:

* **Dataset / SDL public docs (Scalyr)**:
  * Help home: https://app.scalyr.com/help  
  * Power Queries: https://app.scalyr.com/help/power-queries  
  * Dashboards & JSON: https://app.scalyr.com/help/dashboards  
  * Regex: https://app.scalyr.com/help/regex  

* **Custom dashboards – quick reference** (official SentinelOne doc):  
  https://community.sentinelone.com/s/article/000006449

* **Custom dashboard panels JSON** – details on `graphStyle` and panel JSON:  
  https://community.sentinelone.com/s/article/000006458

---

## How to Import a Dashboard

For any JSON file in this repo (for example `fastly/fastly-cdn-dashboard.json`):

1. **Open SDL Dashboards**

   * In the SentinelOne console, go to **Singularity Data Lake → Dashboards**.

2. **Create or open a custom dashboard**

   * Click **New Dashboard** to create a fresh one, or open an existing custom dashboard you want to overwrite/extend.

3. **Edit JSON**

   * From the dashboard menu, choose **Edit JSON**.
   * Open the JSON file from this repo.
   * Copy its entire contents and paste into the SDL JSON editor, replacing or merging as needed.

4. **Save and validate**

   * Save the dashboard.
   * Select an appropriate time range (for example **Last 4 hours** or **Last 24 hours**).
   * If needed, adjust filters (for example `dataSource.name = "Fastly"` or similar) to scope to the intended data source.

---

## Shared JSON & Query Conventions

All dashboards in this repo are designed to be:

* **Schema‑correct** for SDL:
  * `version: "1.0.1"`
  * `configType: "TABBED"`
  * `tabs` array with `graphs` per tab (no top‑level `widgets` array).

* **Time‑aware**:
  * **Always use `timestamp`** as the canonical time field in time‑series charts.
  * Use `group ... by timestamp = timebucket("...")` (no `timechart`).

* **Multiplot‑safe**:
  * For multi‑series line/area charts:
    * Group by `timestamp = timebucket("..."), some_dimension`.
    * Sort by `timestamp` (and often `-count()`).
    * **End the query with `transpose`** so each series renders as its own plot, as recommended in the SDL Query guide.

* **Performance‑conscious**:
  * All **table widgets** end their queries with `| limit 100` (or lower if needed) to keep dashboards responsive.
  * Queries prefer explicit `group` aggregates over raw row streams for time‑series graphs.
  * Avoid unnecessarily broad wildcards or regex where exact matches or `contains` will do; see internal “Tips for faster queries and dashboards” for more guidance.

* **Readable PowerQuery**:
  * Use pipe‑chained queries:

    ```powerquery
    search/filter |
    group ... |
    let ... |
    columns ... |
    sort ... [| transpose] [| limit ...]
    ```

  * Math happens **outside** `group` via `let`:

    ```powerquery
    ... |
    group raw_bytes = sum(network.bytes_out) |
    let GB = raw_bytes / 1073741824.0 |
    columns GB, raw_bytes
    ```

  * Conditionals use the **ternary operator**, not `if(...)`:

    ```powerquery
    let cache_hit = http_response.cache_status = "HIT" ? "HIT" : "MISS"
    ```

---

## Security‑Aware Defaults

Where a dashboard targets **security or alert data** (for example OCSF alerts, indicators):

* Queries may use fields like:

  * `event.category = "indicators"`
  * `dataSource.category = "security"`
  * `dataSource.name = "alert"`

* Widgets aim to answer security questions first (alert volumes, severities, indicators, affected entities) rather than generic HTTP‑style metrics.

Each subfolder README will call out any additional security‑specific assumptions for that source.

---

## Adding a New Dashboard

To add a new data‑source dashboard to this repo:

1. **Prototype in SDL**

   * Build panels using **Graphs** / **PowerQueries** until the queries and visuals look right.
   * Use **Save to Dashboard** to create or update a custom dashboard.

2. **Export JSON**

   * From the dashboard, open **Edit JSON** and copy the full JSON.

3. **Create a new subfolder**

   * Add a new folder (for example `nginx/`).
   * Save the JSON as `<source>-dashboard.json`.
   * Add a `README.md` in that folder describing:
     * Required fields.
     * Purpose of each tab.
     * Any data‑source quirks or filters.

4. **Follow repo conventions**

   * Ensure:
     * `version = "1.0.1"`
     * `configType = "TABBED"`
     * Time‑series queries use `timestamp = timebucket("...")`.
     * Multiplot charts end with `transpose`.
     * Tables end with `limit 100` by default.

---

## References

* **Dataset / SDL public docs (Scalyr)**:
  * https://app.scalyr.com/help  
  * https://app.scalyr.com/help/power-queries  
  * https://app.scalyr.com/help/dashboards  
  * https://app.scalyr.com/help/regex  

* **Custom dashboards – quick reference**:  
  https://community.sentinelone.com/s/article/000006449

* **Custom dashboard panels JSON**:  
  https://community.sentinelone.com/s/article/000006458
