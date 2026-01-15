
<h1>Azure-pipeline</h1>
<h1>Real-Time Streaming Data Pipeline on Azure</h1>

<h2>Problem Statement</h2>
<p>
Modern data platforms are required to ingest continuous event streams, process them with low latency,
and expose the results for both operational and analytical use cases. This project addresses a common
enterprise problem:
</p>

<p>
<strong>How do you design a real-time streaming pipeline that is scalable, fault-tolerant, replayable,
and analytics-ready, while keeping ingestion, processing, and consumption layers loosely coupled?</strong>
</p>

<p>The pipeline must support:</p>
<ul>
  <li>Continuous high-volume event ingestion</li>
  <li>Late-arriving and out-of-order data</li>
  <li>Reliable recovery from failures</li>
  <li>Clear separation of raw, refined, and aggregated data</li>
  <li>Operational serving and analytics without pipeline duplication</li>
</ul>

<p>
This repository documents a production-oriented solution using Azure-native services and a lakehouse-based architecture.
</p>

<h2>High-Level Architecture</h2>

<pre>
CSV Source
   ↓
Python Event Producer
   ↓
Azure Event Hub
   ↓
Event Hub Capture
   ↓
Azure Data Lake Storage Gen2 (Avro)
   ↓
Azure Databricks (Structured Streaming)
   ↓
Bronze → Silver → Gold (Delta Lake)
   ↓
Azure Cosmos DB (Serving Layer)
   ↓
Microsoft Fabric (Analytics & BI)
</pre>

<p>
<strong>Design principle:</strong> each layer has a single responsibility. No component assumes downstream
availability, and failures are isolated to their respective layers.
</p>

<h2>Architecture Components and Design Rationale</h2>

<h3>Ingestion Layer: Azure Event Hub</h3>
<p><strong>Why Event Hub</strong></p>
<ul>
  <li>Designed for high-throughput, low-latency ingestion</li>
  <li>Native integration with Azure services</li>
  <li>Decouples producers from consumers</li>
  <li>Handles burst traffic and back-pressure</li>
</ul>

<p><strong>Tradeoffs</strong></p>
<ul>
  <li>Not suitable for long-term storage</li>
  <li>Requires an external persistence layer for replay</li>
</ul>

<h3>Durable Raw Storage: Event Hub Capture and ADLS Gen2</h3>
<p><strong>Why Event Hub Capture</strong></p>
<ul>
  <li>Automatic persistence of all events</li>
  <li>Durability independent of downstream availability</li>
  <li>Supports replay without re-ingestion</li>
</ul>

<p><strong>Why ADLS Gen2</strong></p>
<ul>
  <li>Cost-effective and scalable object storage</li>
  <li>Hierarchical namespace support</li>
  <li>Native integration with Databricks and Delta Lake</li>
</ul>

<p>
Captured data is stored in Avro format and partitioned by time. This layer serves as the immutable source of truth.
</p>

<h3>Processing Layer: Azure Databricks (Structured Streaming)</h3>
<p>
Databricks is used exclusively for transformation and stateful processing.
</p>

<ul>
  <li>Unified batch and streaming semantics</li>
  <li>Event-time processing and watermarking</li>
  <li>Exactly-once guarantees with Delta Lake sinks</li>
</ul>

<p>
<strong>Design decision:</strong> Streaming jobs read from ADLS rather than directly from Event Hub to enable replay,
backfills, and loose coupling.
</p>

<h2>Lakehouse Design</h2>

<h3>Bronze Layer</h3>
<table>
  <tr>
    <th>Aspect</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>Input</td>
    <td>Avro files from ADLS</td>
  </tr>
  <tr>
    <td>Transformations</td>
    <td>None</td>
  </tr>
  <tr>
    <td>Purpose</td>
    <td>Replay, auditing, debugging</td>
  </tr>
</table>

<h3>Silver Layer</h3>
<table>
  <tr>
    <th>Aspect</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>Transformations</td>
    <td>Parsing, flattening, type casting</td>
  </tr>
  <tr>
    <td>Event Time</td>
    <td>Explicit event_time column</td>
  </tr>
  <tr>
    <td>Purpose</td>
    <td>Clean and queryable dataset</td>
  </tr>
</table>

<h3>Gold Layer</h3>
<table>
  <tr>
    <th>Aspect</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>Processing</td>
    <td>Window-based aggregations</td>
  </tr>
  <tr>
    <td>Late Data</td>
    <td>Handled using watermarking</td>
  </tr>
  <tr>
    <td>Output</td>
    <td>Business-ready Delta tables</td>
  </tr>
</table>

<h2>Event-Time Processing Strategy</h2>
<p>
Windowing groups unbounded streams into finite intervals, while watermarking defines how long the system
waits for late-arriving events.
</p>

<ul>
  <li>Prevents unbounded state growth</li>
  <li>Enables append-mode writes</li>
  <li>Ensures correctness with out-of-order data</li>
</ul>

<h2>Fault Tolerance and Checkpointing</h2>
<p>
Each streaming query maintains a dedicated checkpoint directory storing offsets, watermark progress,
and aggregation state.
</p>

<p>
<strong>Rule:</strong> any logic change requires a new checkpoint path to avoid inconsistent state.
</p>

<h2>Serving Layer: Azure Cosmos DB</h2>
<ul>
  <li>Low-latency reads</li>
  <li>Horizontally scalable</li>
  <li>Optimized for application-facing workloads</li>
</ul>

<p>
Cosmos DB is intentionally not used for analytics to avoid operational impact.
</p>

<h2>Analytics Layer: Microsoft Fabric</h2>
<p>
Cosmos DB data is mirrored into OneLake using Fabric-native mirroring.
</p>

<ul>
  <li>No custom ingestion pipelines</li>
  <li>Near real-time synchronization</li>
  <li>Unified analytics environment</li>
</ul>

<h2>Screenshots and Diagrams</h2>

<div class="placeholder">Architecture diagram placeholder</div>
<div class="placeholder">Databricks streaming job overview placeholder</div>
<div class="placeholder">Cosmos DB container view placeholder</div>
<div class="placeholder">Microsoft Fabric mirrored table placeholder</div>

<h2>Key Outcomes</h2>
<ul>
  <li>End-to-end real-time streaming pipeline</li>
  <li>Replayable and fault-tolerant design</li>
  <li>Correct event-time processing</li>
  <li>Clear separation of concerns</li>
</ul>

<h2>Conclusion</h2>
<p>
This project demonstrates a production-grade streaming data architecture on Azure, emphasizing correctness,
recoverability, and pragmatic tradeoffs under real platform constraints.
</p>

</body>
</html>
