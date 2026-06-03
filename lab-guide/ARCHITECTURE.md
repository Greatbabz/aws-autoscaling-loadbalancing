# AWS Auto Scaling & Load Balancing Architecture

## High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                          INTERNET                               │
│                        (Users/Clients)                          │
└────────────────────────────────┬────────────────────────────────┘
                                 │
                    HTTP Request (Port 80)
                                 │
                                 ▼
          ┌──────────────────────────────────────────┐
          │   Application Load Balancer (ALB)        │
          │         WebApp-ALB                        │
          │                                          │
          │  Type: application                       │
          │  Scheme: internet-facing                 │
          │  VPC: OluTech-Production-VPC             │
          │  Listeners: HTTP :80                     │
          │  Target Group: WebServers-TG             │
          │  Health Check: / (HTTP 200)              │
          │  Availability Zones: us-east-1a, 1b      │
          └────────────┬─────────────────────────────┘
                       │
                       │ Load Balancing (Round-Robin)
                       │
        ┌──────────────┴──────────────┐
        │                             │
        ▼                             ▼
┌──────────────────────┐      ┌──────────────────────┐
│   Availability       │      │   Availability       │
│     Zone A           │      │     Zone B           │
│   (us-east-1a)       │      │   (us-east-1b)       │
│                      │      │                      │
│  ┌────────────────┐  │      │  ┌────────────────┐  │
│  │  EC2 Instance  │  │      │  │  EC2 Instance  │  │
│  │  (t2.micro)    │  │      │  │  (t2.micro)    │  │
│  │                │  │      │  │                │  │
│  │ IP: 10.0.1.79  │  │      │  │ IP: 10.0.2.180 │  │
│  │ Hostname:      │  │      │  │ Hostname:      │  │
│  │ ip-10-0-1-79   │  │      │  │ ip-10-0-2-180  │  │
│  │                │  │      │  │                │  │
│  │ Apache HTTP    │  │      │  │ Apache HTTP    │  │
│  │ Server Running │  │      │  │ Server Running │  │
│  │                │  │      │  │                │  │
│  │ User Data:     │  │      │  │ User Data:     │  │
│  │ - Install httpd│  │      │  │ - Install httpd│  │
│  │ - Start httpd  │  │      │  │ - Start httpd  │  │
│  │ - Show hostname│  │      │  │ - Show hostname│  │
│  └────────────────┘  │      │  └────────────────┘  │
│         ▲            │      │         ▲             │
│         │            │      │         │             │
│    Health Check      │      │    Health Check       │
│    (HTTP /)          │      │    (HTTP /)           │
│         │            │      │         │             │
│    Every 30 sec      │      │    Every 30 sec       │
│         │            │      │         │             │
│      Status: ✅       │      │      Status: ✅       │
│                      │      │                      │
│  Public Subnet       │      │  Public Subnet       │
│  (10.0.1.0/24)       │      │  (10.0.2.0/24)       │
│  + Nat Gateway       │      │  + Nat Gateway       │
└──────────────────────┘      └──────────────────────┘
        ▲                             ▲
        │                             │
        └────────┬────────────────────┘
                 │
              VPC: OluTech-Production-VPC (10.0.0.0/16)
```

---

## Component Details

### 1. Application Load Balancer (ALB)
**Role**: Distributes incoming HTTP traffic across EC2 instances

**Configuration**:
- **Type**: Application (Layer 7 - Application layer)
- **Scheme**: Internet-facing (accessible from public internet)
- **Protocol**: HTTP
- **Port**: 80
- **VPC**: OluTech-Production-VPC
- **Subnets**: Public subnets in AZ-A and AZ-B
- **Security Group**: SG-LoadBalancer (allows inbound HTTP from 0.0.0.0/0)

**Functions**:
- Listens for incoming HTTP requests on port 80
- Performs health checks on target instances every 30 seconds
- Routes traffic to healthy instances using round-robin algorithm
- Maintains connection pooling for efficient request handling
- Removes unhealthy instances from rotation automatically

### 2. Target Group (WebServers-TG)
**Role**: Defines and monitors the group of EC2 instances receiving traffic

**Configuration**:
- **Name**: WebServers-TG
- **Protocol**: HTTP
- **Port**: 80
- **Target Type**: Instances
- **VPC**: OluTech-Production-VPC

**Health Check Settings**:
- **Protocol**: HTTP
- **Path**: `/` (root path)
- **Port**: 80
- **Status Code**: 200 (HTTP OK)
- **Interval**: 30 seconds
- **Timeout**: 5 seconds
- **Healthy Threshold**: 2 consecutive checks
- **Unhealthy Threshold**: 2 consecutive failures

### 3. Auto Scaling Group (WebApp-ASG)
**Role**: Automatically manages the number of EC2 instances based on demand

**Configuration**:
- **Name**: WebApp-ASG
- **Launch Template**: WebServer-LaunchTemplate
- **VPC**: OluTech-Production-VPC
- **Subnets**: Public subnets in AZ-A and AZ-B (for multi-AZ deployment)
- **Target Group**: WebServers-TG

**Capacity Settings**:
- **Desired Capacity**: 2 instances (initial state)
- **Minimum Capacity**: 1 instance (floor)
- **Maximum Capacity**: 4 instances (ceiling)

**Scaling Policy**:
- **Type**: Target Tracking
- **Metric**: Average CPU Utilization
- **Target Value**: 50%
- **Scale-Out Behavior**: If CPU > 50%, launch new instances
- **Scale-In Behavior**: If CPU < 50%, terminate instances (with cooldown)

**Health Check Configuration**:
- **Type**: ELB (Elastic Load Balancing)
- **Grace Period**: 300 seconds (5 minutes)

### 4. EC2 Instances
**Role**: Host the web application (Apache HTTP Server)

**Configuration per Instance**:
- **AMI**: Amazon Linux 2 (ami-*)
- **Instance Type**: t2.micro (1 vCPU, 1 GB RAM)
- **Key Pair**: web-server-key
- **Security Group**: SG-WebLab (allows inbound HTTP port 80)
- **Tenancy**: Default
- **Public IP**: Assigned via NAT Gateway in public subnet

**User Data Script** (Runs on startup):
```bash
#!/bin/bash
yum install -y httpd                  # Install Apache
systemctl start httpd                 # Start Apache service
echo "<h1>Served from instance: $(hostname -f)</h1>" > /var/www/html/index.html
```

**Instance Details**:
- **Instance 1**: ip-10-0-1-79.ec2.internal (AZ-A)
- **Instance 2**: ip-10-0-2-180.ec2.internal (AZ-B)
- **Status**: 2/2 Healthy
- **Health Check State**: Healthy ✅

### 5. VPC and Networking
**VPC**: OluTech-Production-VPC (10.0.0.0/16)

**Subnets**:
- **Public Subnet AZ-A**: 10.0.1.0/24
  - Contains: EC2 Instance 1
  - Route Table: Internet Gateway route (0.0.0.0/0 → IGW)
  - NAT Gateway for outbound traffic

- **Public Subnet AZ-B**: 10.0.2.0/24
  - Contains: EC2 Instance 2
  - Route Table: Internet Gateway route (0.0.0.0/0 → IGW)
  - NAT Gateway for outbound traffic

**Internet Gateway**: Enables communication between instances and internet

**Security Groups**:
- **SG-LoadBalancer**: 
  - Inbound: HTTP (80) from 0.0.0.0/0
  - Outbound: All traffic allowed

- **SG-WebLab**:
  - Inbound: HTTP (80) from SG-LoadBalancer
  - Inbound: SSH (22) from your IP (for management)
  - Outbound: All traffic allowed

---

## Traffic Flow

### Request Journey (Step-by-Step)

1. **User sends HTTP request**
   - User opens browser and navigates to ALB DNS name
   - Request: `http://WebApp-ALB-123456789.us-east-1.elb.amazonaws.com/`

2. **DNS Resolution**
   - ALB DNS name resolves to ALB public IP

3. **ALB Listener receives request**
   - ALB listener on port 80 receives HTTP request
   - Examines request headers and path

4. **Target Group routing**
   - ALB checks Target Group (WebServers-TG) for healthy targets
   - Queries which instances are in "Healthy" state
   - Available: Instance 1 ✅ and Instance 2 ✅

5. **Load Balancing algorithm**
   - ALB uses **Round-Robin** algorithm
   - First request → Instance 1
   - Second request → Instance 2
   - Third request → Instance 1 (cycle repeats)

6. **Request forwarded to instance**
   - ALB forwards request to selected instance (e.g., Instance 1)
   - Maintains persistent connection to instance

7. **Instance processes request**
   - Apache HTTP Server on Instance 1 receives request
   - Serves `/var/www/html/index.html`
   - Page contains: `<h1>Served from instance: ip-10-0-1-79.ec2.internal</h1>`

8. **Response returned to user**
   - Instance sends HTML response through ALB
   - ALB forwards response to user
   - Browser displays page with instance hostname

9. **Connection closed**
   - ALB closes connection after response sent
   - User sees page from specific instance

---

## Scaling Behavior

### Scenario 1: Traffic Increases
```
CPU Utilization climbs to 60% (above 50% target)
          ↓
Auto Scaling detects threshold breach
          ↓
ASG launches new EC2 instance (Instance 3)
          ↓
Instance 3 passes health checks (takes ~2 minutes)
          ↓
ALB adds Instance 3 to Target Group
          ↓
Traffic now distributed across 3 instances
          ↓
CPU utilization decreases back to ~50%
```

### Scenario 2: Traffic Decreases
```
CPU Utilization drops to 30% (below 50% target)
          ↓
Auto Scaling detects underutilization
          ↓
ASG triggers scale-down (with cooldown period)
          ↓
ASG terminates Instance 3 (respects Minimum: 1)
          ↓
ALB removes Instance 3 from Target Group
          ↓
Traffic concentrated on fewer instances
          ↓
Cost reduced, but still maintains minimum availability
```

---

## Availability and Resilience

### Multi-AZ Benefits

1. **Zone Outage Protection**
   - If AZ-A fails: Instance 1 unavailable, but Instance 2 (AZ-B) still serves traffic
   - ASG automatically replaces failed instance in different AZ

2. **Network Redundancy**
   - Each AZ has independent NAT Gateway
   - Each AZ has independent Internet Gateway connection

3. **Application Continuity**
   - Minimum 1 instance ensures service never fully down
   - If 1 instance fails, other immediately takes 100% traffic

### Health Check Resilience

1. **Health Check Failures**
   - Instance fails health check
   - ALB marks as "Unhealthy"
   - ALB stops routing traffic to unhealthy instance
   - ASG detects unhealthy instance after grace period
   - ASG terminates unhealthy instance
   - ASG launches replacement instance (maintains desired capacity)

2. **Recovery Time**
   - Health check interval: 30 seconds
   - Health check failures before unhealthy: 2 consecutive
   - Grace period: 300 seconds
   - Total time to replacement: ~5-7 minutes

---

## Performance Characteristics

### Latency
- **Network latency**: <10ms (same region)
- **ALB processing**: 1-5ms
- **Instance processing**: Variable (depends on application)
- **Total**: Typically <50ms for static content

### Throughput
- **ALB capacity**: Can handle thousands of requests per second
- **t2.micro capacity**: ~100-500 requests per second (depends on application)
- **Total capacity**: Scales horizontally as instances added

### Concurrent Connections
- **ALB**: Can maintain 1M+ concurrent connections
- **t2.micro**: Can maintain ~1000-5000 concurrent connections

---

## Cost Implications

### Hourly Charges
- **ALB**: ~$0.0225/hour (us-east-1)
- **EC2 t2.micro**: ~$0.0116/hour each (if not free tier eligible)
- **Data Transfer**: ~$0.01/GB (inter-AZ traffic)

### Cost Optimization Tips
1. **Use AWS Free Tier**: t2.micro is free for 12 months
2. **Consolidate traffic**: Use fewer instances during off-peak
3. **Reserved Instances**: Pre-purchase capacity for predictable workloads
4. **Spot Instances**: Use for non-critical workloads (up to 90% savings)
5. **Cleanup**: Always delete resources not in use

---

## Monitoring and Observability

### CloudWatch Metrics
- **ALB Metrics**:
  - Active Connection Count
  - New Connection Count
  - Processed Bytes
  - Request Count
  - Target Response Time
  - HTTP 5XX errors

- **Auto Scaling Metrics**:
  - GroupDesiredCapacity
  - GroupInServiceInstances
  - GroupMinSize
  - GroupMaxSize
  - GroupPendingInstances
  - GroupTerminatingInstances

- **EC2 Metrics**:
  - CPU Utilization (triggers scaling policy)
  - Network In
  - Network Out
  - Disk Read Bytes
  - Disk Write Bytes

### Activity Logs
- ASG Activity History shows:
  - Instance launch/termination events
  - Timestamps
  - Status (Success/Failed)
  - Cause (Scaling policy, manual, health check, etc.)

---

## Security Considerations

### Network Security
- ALB in public subnets with strict security group
- EC2 instances in private application security group
- Only ALB can reach EC2 instances on port 80
- SSH access restricted to known IPs

### Data Security
- HTTP (unencrypted) suitable for demo
- Production should use HTTPS/TLS
- Consider AWS Certificate Manager for SSL certificates
- Enable ALB access logs for auditing

### Identity & Access
- EC2 instances don't need internet access (NAT Gateway provides outbound)
- Use IAM roles for any AWS API calls from instances
- Restrict SSH access to specific IP ranges

---

## Summary

This architecture demonstrates:
✅ **Load Balancing**: Distribution of traffic across instances
✅ **Auto Scaling**: Dynamic capacity management
✅ **High Availability**: Multi-AZ deployment
✅ **Health Management**: Automatic failure detection and recovery
✅ **Cost Efficiency**: Pay only for capacity needed
✅ **Production-Ready**: Foundation for real-world applications
