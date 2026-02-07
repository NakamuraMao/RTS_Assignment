flowchart LR
  %% ===== Styles =====
  classDef ocs fill:#eef7ff,stroke:#2b6cb0,stroke-width:1px;
  classDef gcs fill:#fff5f5,stroke:#c53030,stroke-width:1px;
  classDef log fill:#f7fafc,stroke:#4a5568,stroke-dasharray: 5 3;

  %% ===== OCS =====
  subgraph OCS["OCS (Onboard Computer System)"]
    direction TB
    O0([Start / System Init]):::ocs
    O1[Initialize sensors + bounded buffer + thresholds]:::ocs
    O2[[Periodic cycle (every T ms)]]:::ocs
    O3[Read 3 onboard sensors]:::ocs
    O4[Timestamp each sample]:::ocs
    O5[Compute latency: now - sampleTimestamp]:::ocs
    O6{Buffer full?}:::ocs
    O7[Drop lowest-priority / oldest sample]:::ocs
    O8[Log dropped sample + timestamp + reason]:::log
    O9[Insert sample by priority into bounded buffer]:::ocs
    O10[Update missing-count for critical data (e.g., Temp)]:::ocs
    O11{Critical data missing >= 3 cycles?}:::ocs
    O12[Raise HEALTH ALERT event]:::ocs
    O13[Package telemetry: sensor stats + buffer stats + latency + drop count + alerts]:::ocs
    O14[Downlink telemetry to GCS]:::ocs
    O15{Ack received from GCS?}:::ocs
    O16[Retry / mark comm-fault + log]:::log
  end

  %% ===== GCS =====
  subgraph GCS["GCS (Ground Control System)"]
    direction TB
    G1([Receive telemetry]):::gcs
    G2[Validate packet + check timestamp order]:::gcs
    G3[Store to ground log / DB]:::gcs
    G4{ALERT included?}:::gcs
    G5[Notify operator + show reason (missing, high latency, drops)]:::gcs
    G6[Send command: adjust priority / buffer size / sampling rate / request resend]:::gcs
    G7[Send ACK]:::gcs
  end

  %% ===== Links =====
  O0 --> O1 --> O2 --> O3 --> O4 --> O5 --> O6
  O6 -- Yes --> O7 --> O8 --> O9
  O6 -- No --> O9
  O9 --> O10 --> O11
  O11 -- Yes --> O12 --> O13
  O11 -- No --> O13
  O13 --> O14 --> G1 --> G2 --> G3 --> G4
  G4 -- Yes --> G5 --> G6 --> G7 --> O15
  G4 -- No --> G7 --> O15
  O15 -- No --> O16 --> O2
  O15 -- Yes --> O2
