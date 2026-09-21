# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

Hey there!

Your challenge is ready.
Follow the instructions provided for this challenge and complete the required tasks in this repository.

Make sure your work is committed and pushed to your repository before submission.

Good luck!


---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

# AIOps Monitoring and Event Processing

## Task 1: Project Overview

### Service Being Monitored

The service being monitored is a synthetic `payment-service`. It processes
payment requests and produces operational metrics and log messages for each
observation.

### Operational Problem

The purpose of this assessment is to identify abnormal behaviour in the
payment service. The problems of interest include slow payment requests,
excessive CPU or memory usage, payment service timeouts, and database
connection timeouts.

### Purpose of AIOps

AIOps is used to analyse service operational data, identify anomalies, and
process those anomalies as events. The events move through a simulated
producer, topic, and consumer before reaching the final AIOps pipeline output.

### Repository Components

- `data/service_data.json`: Synthetic operational data for the payment service.
- `src/anomaly_detector.py`: Detects abnormal metric and log behaviour.
- `src/event_producer.py`: Publishes anomaly events.
- `src/event_topic.py`: Provides an in-memory event topic.
- `src/event_consumer.py`: Consumes events from the topic.
- `src/aiops_pipeline.py`: Runs the end-to-end AIOps workflow.
- `tests/`: Contains validation tests for calculations and pipeline components.

## Task 2: Operational Data Analysis

The operational data is stored in `data/service_data.json`. It contains 10
observations for the `payment-service`.

### Metric Fields

The following fields represent service metrics:

- `response_time_ms`: Payment request response time in milliseconds.
- `cpu_percent`: CPU utilization percentage.
- `memory_percent`: Memory utilization percentage.

### Log Fields

The following fields represent log information:

- `log_level`: Log severity, such as `INFO` or `ERROR`.
- `message`: Text describing the service activity or problem.

The `service` field identifies the monitored service.

### Timestamps

The `timestamp` field uses the format `YYYY-MM-DDTHH:MM:SS`, for example
`2026-09-20T10:05:00`.

The records occur once per minute and the timestamps show the order in which
the service observations occurred.

### Normal Behaviour

The observations at the following times represent normal behaviour:

- `10:00`
- `10:01`
- `10:02`
- `10:03`
- `10:04`
- `10:07`
- `10:08`
- `10:09`

These records have:

- Response times between approximately 120 and 150 milliseconds.
- CPU usage between approximately 42% and 50%.
- Memory usage between approximately 51% and 57%.
- `INFO` log levels.
- Successful payment processing messages.

### Unusual Behaviour

The observation at `2026-09-20T10:05:00` is unusual:

- Response time: `610 ms`
- CPU usage: `75%`
- Memory usage: `70%`
- Log level: `ERROR`
- Message: `Payment service timeout`

The response time is significantly higher than the normal values, and the
error message indicates a payment timeout.

The observation at `2026-09-20T10:06:00` is also unusual:

- Response time: `640 ms`
- CPU usage: `94%`
- Memory usage: `91%`
- Log level: `ERROR`
- Message: `Database connection timeout`

This record has a high response time, high CPU usage, high memory usage, and an
error log describing a database connection timeout.
Two workflow problems were identified and corrected.

The first problem was in anomaly_detector.py. The detector checked only for
WARNING logs, but the operational data contained ERROR logs. The condition was
updated to detect both WARNING and ERROR levels. After the correction, the
detector included "Error log detected" in the anomaly reasons.

The second problem was in aiops_pipeline.py. The producer and consumer were
using different in-memory EventTopic objects. The producer published events to
one topic while the consumer read from another empty topic. The pipeline was
corrected so that both components use the same EventTopic instance.

Before the correction, the pipeline detected 2 anomalies but consumed 0 events.
After the correction, it processed 10 records, detected 2 anomalies, and
consumed 2 events successfully.
## Task 3: Anomaly Detection

I ran the provided anomaly detector against all 10 records in
`data/service_data.json`.

The detector classified 8 records as normal and detected 2 anomalies.

The first anomaly occurred at `2026-09-20T10:05:00`. The response time was
`610 ms`, which was above the `500 ms` threshold. The record also contained
an `ERROR` log with the message `Payment service timeout`.

The second anomaly occurred at `2026-09-20T10:06:00`. The response time was
`640 ms`, CPU usage was `94%`, and memory usage was `91%`. These values were
above the configured thresholds. The record also contained an `ERROR` log with
the message `Database connection timeout`.

The detector correctly identified the abnormal metric values and did not flag
any of the 8 normal observations.

The initial detector checked only for `WARNING` logs. Because the data contains
`ERROR` logs, the error messages were not included as separate anomaly reasons.
This issue will be corrected during the workflow troubleshooting task.

One limitation is that the detector uses fixed thresholds. If normal service
behaviour changes, fixed thresholds may miss anomalies or create false alarms.
Dynamic thresholds based on historical data would be an improvement.

## Task 4: AIOps Event Flow

The anomaly event-processing flow is:

Operational data -> Anomaly detection -> Event -> Producer -> Topic -> Consumer -> AIOps output

The anomaly detector reads each operational record. When abnormal behaviour is
found, it creates an event containing the timestamp, service name, event type,
reasons, and original source record.

The producer receives the anomaly event and publishes it to the in-memory event
topic.

The topic acts as a temporary message channel. It stores the events published
by the producer.

The consumer reads the events from the topic and returns them for downstream
processing.

The AIOps pipeline combines these components and displays the final result,
including the affected service, timestamp, event type, and anomaly reasons.

The event-flow validation confirmed that an anomaly event was created,
published by the producer, stored in the `anomaly-events` topic, and received
by the consumer. The producer and consumer must use the same topic instance
for the consumer to receive the published event.

## Task 6: End-to-End Pipeline Execution

The complete AIOps workflow was executed successfully.

The pipeline processed 10 operational records and detected 2 anomalies. The
anomaly events were passed to the producer and published to the shared
`anomaly-events` topic.

The consumer received and processed both events. The final output identified
the following operational issues:

- A payment service timeout at `2026-09-20T10:05:00`.
- A database connection timeout at `2026-09-20T10:06:00`.
- High response time in both anomalous records.
- High CPU and memory usage at `10:06`.

The final execution result was:


Records processed: 10
Anomalies detected: 2
Events consumed: 2

## AIOps Assessment Summary

### Scenario

This project monitors a synthetic `payment-service`. The aim is to detect slow
payment requests, high resource usage, and service or database timeouts.

AIOps is used to analyse service data, detect unusual behaviour, and move
anomaly events through a simple event-processing workflow.

### Operational Data

The data is stored in `data/service_data.json`. Each record contains a
timestamp, service name, response time, CPU usage, memory usage, log level, and
message.

The metric fields are:

- `response_time_ms`
- `cpu_percent`
- `memory_percent`

The log fields are:

- `log_level`
- `message`

The records are timestamped once per minute. Most records show normal
behaviour: response times are around 120 to 150 ms, CPU and memory usage are
moderate, and the log level is `INFO`.

Two records are unusual:

- At `10:05`, the response time was `610 ms` and the service reported a payment
  timeout.
- At `10:06`, the response time was `640 ms`, CPU was `94%`, memory was `91%`,
  and the service reported a database connection timeout.

### Anomaly Detection

The detector processed 10 records and identified 2 anomalies. It detected the
high response time in both unusual records, and also detected high CPU and
memory usage at `10:06`.

The detector initially checked only for `WARNING` logs, even though the data
contained `ERROR` logs. This was corrected so both `WARNING` and `ERROR` logs
are detected. No normal records were incorrectly flagged.

### Event Processing Flow

The workflow is:

```text
Operational data -> Anomaly detector -> Event -> Producer -> Topic -> Consumer -> AIOps output
Limitation:
The detector uses fixed thresholds. If normal service behaviour changes, these
thresholds may create false alarms or miss anomalies. Dynamic thresholds based
on historical data would improve the approach.
## Files Changed

I kept the original project structure and changed only the files needed for the
assessment:

- `README.md`: Documents the AIOps scenario, findings, corrections, and results.
- `requirements.txt`: Includes `pytest-cov` so the coverage command can run.
- `src/anomaly_detector.py`: Detects both `WARNING` and `ERROR` log events.
- `src/aiops_pipeline.py`: Uses one shared topic for the producer and consumer.