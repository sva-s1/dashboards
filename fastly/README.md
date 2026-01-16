
# Fastly CDN SDL Dashboard

This folder contains a custom **Fastly CDN** dashboard for **SentinelOne Singularity Data Lake (SDL)** / Dataset.

It reuses all of the **foundational conventions** described in the repository root [SDL Dashboards README](../README.md) (schema versioning, `timestamp`/`timebucket` usage, `transpose` for multiplot charts, `limit 100` on tables, PowerQuery style, etc.) and adds only **Fastly‑specific details** here.

If you have not read the parent README yet, start there:

> 📘 See: [`../README.md`](../README.md) for:
> * SDL dashboard JSON schema (`version`, `configType`, `tabs`, `graphs`)
> * Time‑series and multiplot patterns (including required `transpose`)
> * Table performance rules (`limit 100`)
> * General PowerQuery and security‑aware defaults

---

## Files

* `fastly-cdn-dashboard.json`  
  Main dashboard JSON for Fastly CDN logs (schema `version: "1.0.1"`, `configType: "TABBED"`).

* `README.md`  
  This file – Fastly‑specific context and navigation.

---

## Data Assumptions (Fastly)

This dashboard assumes Fastly logs are ingested into SDL/Dataset and can be scoped with fields like:

* **Source identification**

  * `dataSource.name = "Fastly"`  
  * `dataSource.name = "Fastly"`, `dataSource.vendor = "Fastly"`, and `dataSource.category = "security"` **must** be present on all Fastly events so they appear in the **XDR** view; as a result, this dashboard is intended to be used with the **XDR** or **ALL** view selected.

* **Time**

  * `timestamp` – canonical event time.  
    (The dashboard follows the parent README’s rule that **all** time‑series use `timestamp` + `timebucket(...)`.)

* **Request**

  * `http_request.http_method`  
  * `http_request.url.hostname`  
  * `http_request.url.path`  
  * `http_request.user_agent`

* **Response**

  * `http_response.status_code` (or `http_response.code`)  
  * `http_response.cache_status` (for example `"HIT"`, `"MISS"`, `"PASS"`)

* **Traffic volume**

  * `network.bytes_in`  
  * `network.bytes_out`

* **Optional security context**

  * `event.category` (for example `"indicators"`, `"file"`, `"network"`)  
  * `dataSource.name = "alert"` when Fastly‑related alerts are ingested.

If your field names differ, adjust the queries in `fastly-cdn-dashboard.json` to match your schema, but keep the general patterns from the parent README.

---

## Importing the Fastly Dashboard

1. Open **Singularity Data Lake → Dashboards** in the SentinelOne console.

2. Create or select a **custom dashboard**:
   * Click **New Dashboard**, or open an existing one you want to overwrite/extend.

3. From the dashboard menu, choose **Edit JSON**.

4. Open `fastly-cdn-dashboard.json` from this folder and paste its full contents into the SDL JSON editor.

5. Save the dashboard, set an appropriate time range (for example **Last 4 hours** / **Last 24 hours**), and, if needed, apply a filter such as:

   ```text
   dataSource.name = "Fastly"
   ```

---

## Tab Layout and Intent

The exact tab names may vary slightly, but the Fastly dashboard is organized around these main views.

### 1. Overview

High‑level Fastly health and traffic:

* **Total requests** – billboard (number) summarizing request volume.

* **Requests over time** – line chart of Fastly events over `timestamp = timebucket("...")`.

* **Top hostnames/domains** – table or donut showing top `http_request.url.hostname`.

Use this tab to answer “Is Fastly up and doing roughly what I expect?” before drilling into specifics.

---

### 2. Performance & Errors

HTTP behavior and reliability:

* **Status code distribution** – donut/pie chart by `http_response.status_code`.

* **Error rate over time** – multiplot line chart broken down by status code / code class.  
  (Implements the parent README’s multiplot rule: `group ... by timestamp = timebucket(...), status` then `transpose`.)

* **Top error URLs** – table of noisy paths/hosts with non‑2xx status codes.

Use this tab to spot spikes in 4xx/5xx responses and quickly identify which URLs or hosts are contributing.

---

### 3. Cache & Content

Cache behavior and content mix:

* **Cache status over time** – multiplot line chart broken down by `http_response.cache_status` (HIT/MISS/PASS). Uses `transpose` per the global convention.

* **Top cached vs non‑cached objects** – tables/donuts comparing HIT vs MISS by hostname/path.

* **Bandwidth usage** – numbers or time series for `sum(network.bytes_out)` and `sum(network.bytes_in)`, with unit conversions done via `let` expressions (as recommended in the parent README).

Use this tab to tune cache efficiency and understand where bandwidth is going.

---

### 4. Security Signals (Optional)

Only meaningful if your Fastly deployment is tied into **security or alert data**.

Examples of widgets in this tab:

* **Indicators over time** – where `event.category = "indicators"`, broken down by `indicator.category` or similar.

* **Fastly‑related alerts** – where `dataSource.name = "alert"` and metadata indicates Fastly as the product/vendor, broken down by severity or category.

* **Top suspicious domains/paths** – tables highlighting domains/paths that appear frequently in security/alert events.

If you don’t ingest Fastly‑related security data, you can remove this tab or leave it hidden/unused.

---

## Fastly‑Specific Customization Ideas

Once imported, you can safely customize the dashboard while preserving the shared conventions from the parent README:

* **Per‑customer or per‑domain views**

  * Clone the Overview or Performance tab and add filters like:
    * `http_request.url.hostname = "customer1.example.com"`

* **Different time resolutions**

  * Adjust `timebucket("5m")` to `"1m"` for burst analysis or `"15m"/"1h"` for higher‑level overviews.

* **Specialized content views**

  * Add panels focused on specific content groups (for example `http_request.url.path starts_with "/api/"` vs static assets).

As you extend, keep the global rules from `../README.md` in mind (time handling, `transpose` for multiplots, `limit 100` for tables) so this dashboard remains consistent with the rest of the repo.

---

## References

For details on how the queries and JSON for this dashboard are structured, see:

* Parent repo: [`../README.md`](../README.md)

* Dataset / SDL docs:
  * Power Queries: https://app.scalyr.com/help/power-queries  
  * Dashboards: https://app.scalyr.com/help/dashboards  
  * Regex: https://app.scalyr.com/help/regex
