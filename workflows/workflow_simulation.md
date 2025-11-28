# AI Agent Workflow Simulation
## AutoParts Inc. Smart Manufacturing System

**Platform**: n8n / make.com  
**Purpose**: Orchestrate three AI agent types for quality control, predictive maintenance, and production optimization

---

## Workflow Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    AUTOPARTS INC. AI AGENT SYSTEM                    │
└─────────────────────────────────────────────────────────────────────┘

┌──────────────────┐      ┌──────────────────┐      ┌──────────────────┐
│  QUALITY CONTROL │      │   PREDICTIVE     │      │   PRODUCTION     │
│      AGENT       │      │  MAINTENANCE     │      │  OPTIMIZATION    │
│                  │      │      AGENT       │      │      AGENT       │
│  Computer Vision │      │  IoT + ML        │      │  Reinforcement   │
│  Defect Detection│      │  Failure Predict │      │  Learning        │
└────────┬─────────┘      └────────┬─────────┘      └────────┬─────────┘
         │                         │                         │
         │                         │                         │
         ▼                         ▼                         ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        n8n WORKFLOW ENGINE                           │
│  • Event Routing                                                     │
│  • Agent Coordination                                                │
│  • Data Transformation                                               │
│  • System Integration                                                │
└─────────────────────────────────────────────────────────────────────┘
         │                         │                         │
         ▼                         ▼                         ▼
┌──────────────────┐      ┌──────────────────┐      ┌──────────────────┐
│       MES        │      │      CMMS        │      │       ERP        │
│  Manufacturing   │      │   Maintenance    │      │   Enterprise     │
│  Execution Sys   │      │   Management     │      │   Resource Plan  │
└──────────────────┘      └──────────────────┘      └──────────────────┘
```

---

## Workflow 1: Quality Control Agent

### Trigger: Image Capture Event

**n8n Workflow Steps**:

```
1. [Webhook] Receive image from production line camera
   ├─ Input: Image URL, Component ID, Production Line, Timestamp
   └─ Trigger: Every 2 seconds per camera (15 cameras total)

2. [HTTP Request] Send image to AI vision API
   ├─ Endpoint: https://api.autoparts.com/vision/inspect
   ├─ Method: POST
   ├─ Body: { "image_url": "...", "component_id": "..." }
   └─ Response: { "defect_detected": true/false, "defect_type": "...", "confidence": 0.98 }

3. [IF Node] Check if defect detected
   ├─ Condition: defect_detected === true
   │
   ├─ TRUE Branch:
   │   ├─ [HTTP Request] Alert production line
   │   │   └─ Endpoint: MES API - /production/alert
   │   │
   │   ├─ [HTTP Request] Route to rework station
   │   │   └─ Endpoint: MES API - /routing/rework
   │   │
   │   ├─ [Database] Log defect
   │   │   └─ Table: defects (component_id, type, timestamp, image_url)
   │   │
   │   └─ [Email] Notify quality manager (if critical defect)
   │       └─ Condition: defect_type === "critical"
   │
   └─ FALSE Branch:
       ├─ [HTTP Request] Continue production
       │   └─ Endpoint: MES API - /production/continue
       │
       └─ [Database] Log pass
           └─ Table: quality_checks (component_id, status: "pass")

4. [Function] Update quality metrics
   ├─ Calculate: Defect rate = defects / total_inspections
   └─ Store in dashboard database

5. [Schedule] Daily defect report (runs at 6 PM)
   ├─ [Database] Query defects from last 24 hours
   ├─ [Function] Generate report (defect types, trends, root causes)
   └─ [Email] Send to production manager
```

### n8n Configuration

**Nodes Required**:
- Webhook Trigger
- HTTP Request (×4)
- IF Conditional
- PostgreSQL Database (×2)
- Email (×2)
- Function (×2)
- Schedule Trigger

**Estimated Execution Time**: <500ms per inspection

---

## Workflow 2: Predictive Maintenance Agent

### Trigger: IoT Sensor Data Stream

**n8n Workflow Steps**:

```
1. [MQTT] Subscribe to sensor data topic
   ├─ Topic: sensors/machines/+/telemetry
   ├─ Frequency: Every 10 seconds
   └─ Data: { "machine_id": "CNC-07", "vibration": 2.3, "temp": 65, "current": 12.5 }

2. [Function] Aggregate sensor readings
   ├─ Buffer last 100 readings per machine
   └─ Calculate: Mean, std dev, trend

3. [HTTP Request] Send to ML prediction API
   ├─ Endpoint: https://api.autoparts.com/ml/predict-failure
   ├─ Body: { "machine_id": "...", "sensor_data": [...] }
   └─ Response: { "failure_probability": 0.85, "predicted_failure": "spindle", "time_to_failure_hours": 72 }

4. [IF Node] Check failure probability
   ├─ Condition: failure_probability > 0.7
   │
   ├─ TRUE Branch (High Risk):
   │   ├─ [HTTP Request] Create maintenance work order
   │   │   ├─ Endpoint: CMMS API - /workorders/create
   │   │   └─ Body: { "machine_id": "...", "priority": "high", "description": "Predicted spindle failure in 72h" }
   │   │
   │   ├─ [HTTP Request] Check spare parts inventory
   │   │   └─ Endpoint: ERP API - /inventory/check
   │   │
   │   ├─ [IF Node] Spare parts available?
   │   │   ├─ TRUE: Schedule maintenance
   │   │   └─ FALSE: Order parts + delay maintenance
   │   │
   │   ├─ [HTTP Request] Notify Production Optimization Agent
   │   │   └─ Webhook: /agents/production/machine-downtime
   │   │   └─ Purpose: Re-schedule production around maintenance
   │   │
   │   └─ [Slack] Alert maintenance team
   │       └─ Message: "🚨 CNC-07 predicted failure in 72h - Work Order #1234 created"
   │
   └─ FALSE Branch (Normal):
       └─ [Database] Log sensor data for historical analysis

5. [Schedule] Weekly maintenance report (runs Monday 8 AM)
   ├─ [Database] Query all predictions from last week
   ├─ [Function] Calculate: Prediction accuracy, prevented failures, cost savings
   └─ [Email] Send to operations manager
```

### n8n Configuration

**Nodes Required**:
- MQTT Trigger
- Function (×3)
- HTTP Request (×5)
- IF Conditional (×2)
- PostgreSQL Database (×2)
- Slack Notification
- Email
- Schedule Trigger

**Estimated Execution Time**: <1 second per prediction

---

## Workflow 3: Production Optimization Agent

### Trigger: New Order or Schedule Update

**n8n Workflow Steps**:

```
1. [Webhook] Receive new order or production event
   ├─ Events: New order, rush order, machine downtime, material arrival
   └─ Input: { "event_type": "new_order", "order_id": "...", "quantity": 500, "due_date": "..." }

2. [HTTP Request] Get current production state
   ├─ Endpoint: MES API - /production/current-state
   └─ Response: { "active_jobs": [...], "machine_status": {...}, "inventory": {...} }

3. [HTTP Request] Get maintenance schedule
   ├─ Endpoint: CMMS API - /maintenance/schedule
   └─ Purpose: Avoid scheduling jobs on machines with planned downtime

4. [HTTP Request] Send to optimization API
   ├─ Endpoint: https://api.autoparts.com/ml/optimize-schedule
   ├─ Body: { "orders": [...], "production_state": {...}, "constraints": {...} }
   ├─ AI Model: Reinforcement learning agent
   └─ Response: { "optimized_schedule": [...], "expected_completion": "...", "utilization": 0.87 }

5. [Function] Validate schedule
   ├─ Check: No conflicts, meets deadlines, respects constraints
   └─ If invalid: Re-run optimization with adjusted parameters

6. [HTTP Request] Update production schedule in MES
   ├─ Endpoint: MES API - /production/update-schedule
   └─ Body: { "schedule": [...] }

7. [Parallel Execution]
   ├─ [HTTP Request] Update ERP with delivery dates
   │   └─ Endpoint: ERP API - /orders/update-delivery
   │
   ├─ [Email] Notify production supervisors
   │   └─ Subject: "Production schedule updated - New priorities"
   │
   └─ [Slack] Alert floor managers
       └─ Message: "📅 Schedule optimized - Check MES for updates"

8. [IF Node] Rush order?
   ├─ Condition: order_priority === "rush"
   └─ TRUE: 
       ├─ [HTTP Request] Notify Quality Control Agent
       │   └─ Increase inspection frequency for rush order components
       └─ [SMS] Alert shift supervisor
           └─ Message: "Rush order #... prioritized - Due in 48h"

9. [Schedule] Hourly schedule re-optimization (runs every hour)
   ├─ [HTTP Request] Get latest production state
   ├─ Re-run optimization if significant changes
   └─ Update schedule dynamically
```

### n8n Configuration

**Nodes Required**:
- Webhook Trigger
- HTTP Request (×7)
- Function (×2)
- IF Conditional (×2)
- Email
- Slack Notification
- SMS Notification
- Schedule Trigger
- Merge/Split nodes for parallel execution

**Estimated Execution Time**: 2-5 seconds per optimization

---

## Multi-Agent Coordination Workflow

### Scenario: Machine Predicted to Fail During Rush Order

**Workflow Steps**:

```
1. [Predictive Maintenance Agent] Detects high failure probability
   └─ Triggers webhook: /coordination/machine-risk

2. [n8n Coordinator] Receives alert
   ├─ Machine: CNC-07
   ├─ Failure Probability: 85%
   ├─ Time to Failure: 48 hours
   └─ Current Job: Rush order #5678 (due in 60 hours)

3. [Decision Logic]
   ├─ [HTTP Request] Query Production Optimization Agent
   │   └─ Question: "Can we complete rush order on alternative machine?"
   │   └─ Response: "Yes, CNC-12 available, adds 6 hours"
   │
   ├─ [HTTP Request] Query Quality Control Agent
   │   └─ Question: "Is CNC-12 producing acceptable quality?"
   │   └─ Response: "Yes, 98.5% pass rate (vs. 98.7% for CNC-07)"
   │
   └─ [Function] Calculate best action
       ├─ Option A: Continue on CNC-07, risk failure mid-job
       ├─ Option B: Switch to CNC-12, add 6 hours but ensure completion
       └─ Decision: Option B (minimize risk)

4. [Execute Decision]
   ├─ [HTTP Request] Production Optimization Agent - Re-schedule
   │   └─ Move rush order to CNC-12
   │
   ├─ [HTTP Request] Predictive Maintenance Agent - Schedule maintenance
   │   └─ Maintenance window: After rush order completion
   │
   └─ [HTTP Request] Quality Control Agent - Monitor CNC-12
       └─ Increase inspection frequency during rush order

5. [Notifications]
   ├─ [Email] Operations manager - Decision summary
   ├─ [Slack] Production team - Schedule change
   └─ [SMS] Maintenance team - Confirmed maintenance window

6. [Feedback Loop]
   └─ [Database] Log decision and outcome for future learning
```

---

## Implementation Guide for n8n

### Step 1: Setup n8n Instance

```bash
# Option 1: Cloud (Recommended for quick start)
# Visit: https://n8n.io/
# Sign up for free account

# Option 2: Self-hosted (Docker)
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```

### Step 2: Install Required Nodes

**Core Nodes** (pre-installed):
- Webhook
- HTTP Request
- IF Conditional
- Function
- Email
- Schedule Trigger

**Additional Nodes** (install via n8n UI):
- PostgreSQL
- MQTT
- Slack
- Twilio (SMS)

### Step 3: Create Workflows

1. **Import Workflow Templates**:
   - Download JSON templates from `/workflows` directory
   - In n8n: Settings → Import from File

2. **Configure Credentials**:
   - MES API: Add HTTP Request credentials
   - CMMS API: Add HTTP Request credentials
   - ERP API: Add HTTP Request credentials
   - Email: Add SMTP credentials
   - Slack: Add Slack OAuth token
   - Database: Add PostgreSQL connection

3. **Set Environment Variables**:
   ```
   MES_API_URL=https://mes.autoparts.com/api
   CMMS_API_URL=https://cmms.autoparts.com/api
   ERP_API_URL=https://erp.autoparts.com/api
   ML_API_URL=https://ml.autoparts.com/api
   ```

### Step 4: Test Workflows

1. **Quality Control Test**:
   - Send test webhook with sample image URL
   - Verify defect detection and routing

2. **Predictive Maintenance Test**:
   - Publish test MQTT message with sensor data
   - Verify work order creation

3. **Production Optimization Test**:
   - Send test order via webhook
   - Verify schedule optimization

### Step 5: Deploy to Production

1. Enable workflows (toggle switch in n8n)
2. Monitor execution logs
3. Set up error notifications
4. Configure backup/redundancy

---

## Alternative: make.com Implementation

### Workflow Structure

**Scenario Builder**:
```
1. Trigger: Webhook (Image Capture)
2. HTTP Module: Vision API
3. Router:
   ├─ Path 1 (Defect): MES Alert + Database Log + Email
   └─ Path 2 (Pass): Continue Production
4. Data Store: Update Metrics
```

**Advantages of make.com**:
- Visual drag-and-drop interface
- Built-in error handling
- Easier for non-technical users

**Disadvantages**:
- More expensive at scale
- Less flexibility than n8n

---

## Monitoring and Maintenance

### Key Metrics to Track

```
┌──────────────────────────────────────────────────────┐
│         AI AGENT WORKFLOW DASHBOARD                  │
├──────────────────────────────────────────────────────┤
│ Quality Control Agent:                               │
│   • Inspections/day: 43,200 (15 cameras × 2/sec)    │
│   • Defects detected: 1,296 (3% rate)               │
│   • False positives: 26 (2% of detections)          │
│   • Workflow execution time: Avg 320ms              │
│                                                       │
│ Predictive Maintenance Agent:                        │
│   • Machines monitored: 50                           │
│   • Predictions/day: 720 (every 10 sec)             │
│   • High-risk alerts: 3                              │
│   • Prevented failures: 12 this month                │
│   • Workflow execution time: Avg 850ms              │
│                                                       │
│ Production Optimization Agent:                       │
│   • Schedule updates/day: 24 (hourly)               │
│   • Orders optimized: 156                            │
│   • Utilization improvement: +28%                    │
│   • Workflow execution time: Avg 3.2s               │
└──────────────────────────────────────────────────────┘
```

### Alerting Rules

- Workflow execution failure → Slack alert + Email
- Execution time >5 seconds → Performance warning
- Error rate >1% → System health check
- Agent coordination conflict → Manual review

---

## Simulation Link

**n8n Cloud Workflow**:
```
[To be created by user at: https://n8n.io/]

Steps to create:
1. Sign up for n8n cloud account
2. Import workflow templates from this repository
3. Configure API credentials
4. Test with sample data
5. Share public workflow link
```

**Expected Workflow URL Format**:
```
https://[your-instance].app.n8n.cloud/workflow/[workflow-id]
```

**Add this link to GitHub README after creation**

---

## Conclusion

This workflow simulation demonstrates how three AI agent types coordinate through n8n to transform AutoParts Inc.'s manufacturing operations. The system handles 50,000+ events per day, orchestrates complex multi-agent decisions, and integrates seamlessly with existing MES/CMMS/ERP systems. The modular design allows for easy expansion and continuous improvement as the AI agents learn from production data.

---

**Document Version**: 1.0  
**Last Updated**: November 28, 2025
