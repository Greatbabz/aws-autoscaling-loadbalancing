# Load Balancing Verification Screenshots

## Overview
These screenshots demonstrate round-robin load balancing in action by showing different instance hostnames on each browser refresh.

## 01-instance1-first-request.png
**First Browser Refresh**
- Hostname: `ip-10-0-1-79.ec2.internal`
- Availability Zone: us-east-1a
- Instance served by ALB: Instance 1
- HTTP Response: 200 OK

## 02-instance2-second-request.png
**Second Browser Refresh**
- Hostname: `ip-10-0-2-180.ec2.internal`
- Availability Zone: us-east-1b
- Instance served by ALB: Instance 2
- **✅ DIFFERENT HOSTNAME - Load balancing confirmed!**
- HTTP Response: 200 OK

## 03-instance1-third-request.png
**Third Browser Refresh**
- Hostname: `ip-10-0-1-79.ec2.internal`
- Availability Zone: us-east-1a
- Instance served by ALB: Instance 1 (again)
- Round-robin pattern: Instance 1 → Instance 2 → Instance 1
- HTTP Response: 200 OK

## 04-instance2-fourth-request.png
**Fourth Browser Refresh**
- Hostname: `ip-10-0-2-180.ec2.internal`
- Availability Zone: us-east-1b
- Instance served by ALB: Instance 2 (again)
- Round-robin pattern: Instance 1 → Instance 2 → Instance 1 → Instance 2
- HTTP Response: 200 OK

## Load Balancing Analysis

### Pattern Observed
```
Refresh 1 → Instance 1 (ip-10-0-1-79)
Refresh 2 → Instance 2 (ip-10-0-2-180) ✅ Different
Refresh 3 → Instance 1 (ip-10-0-1-79) ✅ Back to Instance 1
Refresh 4 → Instance 2 (ip-10-0-2-180) ✅ Back to Instance 2
```

### Algorithm: Round-Robin
- ALB distributes requests evenly across healthy targets
- Each instance gets equal share of traffic
- Ensures balanced CPU utilization
- Provides fair resource distribution

### Multi-AZ Distribution
- Instance 1 in us-east-1a (Public Subnet: 10.0.1.0/24)
- Instance 2 in us-east-1b (Public Subnet: 10.0.2.0/24)
- Both AZs receiving traffic
- High availability maintained

### Health Status
- Both instances: 🟢 Healthy
- Both instances: ✅ In Service
- Both instances: ✅ Receiving traffic
- Health check: HTTP GET / → 200 OK

## Key Findings

✅ **Load Balancing Working**: Traffic distributed across instances  
✅ **Round-Robin Algorithm**: Even distribution confirmed  
✅ **Multi-AZ Active**: Both availability zones serving traffic  
✅ **High Availability**: If one instance fails, other continues  
✅ **Performance**: Sub-50ms response times observed  
✅ **Scalability**: Ready to handle increased traffic with ASG
