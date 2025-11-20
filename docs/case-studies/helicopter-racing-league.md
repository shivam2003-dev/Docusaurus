# Helicopter Racing League

## Company Overview
Helicopter Racing League (HRL) is a global sports league for competitive helicopter racing. Each year, HRL holds the world championship and several regional league competitions.

## Solution Concept
HRL wants to migrate their existing service to the cloud to handle the spike in traffic during race events. They also want to provide real-time telemetry data to fans and broadcasters.

## Existing Technical Environment
- **Hosting**: Public cloud provider (VMs).
- **Video**: Live stream using a third-party service.
- **Telemetry**: Custom solution using MQTT.

## Business Requirements
- **Experience**: Low latency for real-time data.
- **Scale**: Handle millions of concurrent viewers during races.
- **Cost**: Minimize costs during off-season.
- **Global**: Serve users worldwide.

## Technical Requirements
- **Compute**: Auto-scaling compute resources.
- **Data**: Real-time ingestion and processing of telemetry data.
- **AI/ML**: Predict race outcomes and analyze pilot performance.
