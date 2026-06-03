# AWS Auto Scaling & Load Balancing Lab - Complete Steps

**Duration**: 80 minutes  
**Level**: Beginner-Friendly  
**Instructor**: AWS Cloud Accelerator

---

## Prerequisites ✅
- Custom VPC from Week 3
- EC2 key pair from Week 4 Day 1

---

## Step 1: Create a Launch Template

### What is a Launch Template?
A Launch Template is a reusable blueprint that defines the configuration for EC2 instances (AMI, instance type, security groups, user data, etc.). This enables consistent, rapid instance provisioning.

### Steps:
1. Navigate to **EC2 → Launch Templates → Create launch template**
2. **Template name**: `WebServer-LaunchTemplate`
3. **AMI**: Select **Amazon Linux 2**
4. **Instance type**: `t2.micro` (eligible for free tier)
5. **Key pair**: Select `web-server-key` (from Week 4 Day 1)
6. **Security Group**: Select `SG-WebLab` (HTTP 80 open)
7. **User Data** (Advanced details):
   ```bash
   #!/bin/bash
   yum install -y httpd
   systemctl start httpd
   echo "<h1>Served from instance: $(hostname -f)</h1>" > /var/www/html/index.html
   ```
   This script installs Apache, starts it, and creates a page displaying the instance hostname.

8. Click **Create launch template**

### Why This Matters:
- **Consistency**: Every instance launched from this template has identical configuration
- **Speed**: No need to manually configure each instance
- **Scalability**: Auto Scaling Groups reference this template

---

## Step 2: Create the Application Load Balancer

### What is an Application Load Balancer?
An ALB distributes incoming traffic across multiple targets (EC2 instances) in different availability zones. It operates at Layer 7 (Application layer) and can route based on hostnames, URLs, and other request properties.

### Steps:
1. Navigate to **EC2 → Load Balancers → Create load balancer → Application Load Balancer**
2. **Name**: `WebApp-ALB`
3. **Scheme**: Select **Internet-facing** (receives traffic from the internet)
4. **IP address type**: IPv4
5. **VPC**: Select `OluTech-Production-VPC`
6. **Mappings** (Select both public subnets for 2 AZs):
   - ☑️ Availability Zone A (public subnet)
   - ☑️ Availability Zone B (public subnet)
7. **Security group**: Select `SG-LoadBalancer` (HTTP 80 open to internet - `0.0.0.0/0`)
8. **Listener** (HTTP on port 80):
   - Protocol: **HTTP**
   - Port: **80**
9. **Target group** (Create new):
   - Name: `WebServers-TG`
   - Protocol: **HTTP**
   - Port: **80**
   - Target type: **Instances**
   - VPC: `OluTech-Production-VPC`
10. **Health check settings**:
    - Protocol: **HTTP**
    - Path: `/`
    - Port: **80**
    - Status code: **200**
11. Click **Create load balancer**

### Why This Matters:
- **High Availability**: Distributes traffic across multiple instances
- **Fault Tolerance**: If one instance fails, traffic routes to healthy ones
- **Scalability**: Can handle increasing traffic by adding instances
- **Health Checks**: Automatically removes unhealthy instances from rotation

---

## Step 3: Create the Auto Scaling Group

### What is an Auto Scaling Group?
An ASG automatically launches or terminates EC2 instances based on demand (CPU utilization). It maintains a desired number of instances and scales up/down based on policies.

### Steps:
1. Navigate to **EC2 → Auto Scaling Groups → Create Auto Scaling group**
2. **Name**: `WebApp-ASG`
3. **Launch template**: Select `WebServer-LaunchTemplate`
4. **VPC**: Select `OluTech-Production-VPC`
5. **Subnets**: Select both public subnets (AZ-A and AZ-B)
6. **Attach to existing load balancer**:
   - ☑️ Select this option
   - Target group: Select `WebServers-TG`
7. **Health checks**:
   - ☑️ Turn on Elastic Load Balancing health checks
   - Health check grace period: **300 seconds**
8. **Group size** (Capacity):
   - Desired capacity: **2**
   - Minimum capacity: **1**
   - Maximum capacity: **4**
9. **Scaling policy**:
   - Policy type: **Target tracking**
   - Metric type: **Average CPU Utilization**
   - Target value: **50%**
10. Click **Create Auto Scaling group**

### Configuration Explanation:
- **Desired: 2**: Start with 2 instances for load balancing
- **Minimum: 1**: Can scale down to 1 instance during low traffic
- **Maximum: 4**: Can scale up to 4 instances during high traffic
- **CPU Target: 50%**: If CPU exceeds 50%, launch new instances

### Why This Matters:
- **Cost Efficiency**: Only pay for instances you need
- **Automatic Scaling**: Responds to traffic without manual intervention
- **High Availability**: Maintains minimum instance count for resilience
- **Performance**: Scales up to maintain target performance metrics

---

## Step 4: Test Load Balancing

### Verification Process:

1. **Wait for instances to launch** (3-4 minutes)
   - The ASG launches instances from the Launch Template
   - Instances must pass health checks before receiving traffic

2. **Access the ALB**:
   - Go to **EC2 → Load Balancers → WebApp-ALB**
   - Copy the **DNS name** (e.g., `WebApp-ALB-123456789.us-east-1.elb.amazonaws.com`)

3. **Test in browser**:
   - Paste the DNS name in your browser: `http://WebApp-ALB-123456789.us-east-1.elb.amazonaws.com`
   - Page displays: "Served from instance: ip-10-0-X-XXX.ec2.internal"

4. **Verify load balancing** (Refresh 5-10 times):
   - **Refresh 1**: Shows `ip-10-0-1-79.ec2.internal` (Instance 1 - AZ-A)
   - **Refresh 2**: Shows `ip-10-0-2-180.ec2.internal` (Instance 2 - AZ-B)
   - **Refresh 3**: Shows `ip-10-0-1-79.ec2.internal` (Instance 1 again)
   - **Refresh 4**: Shows `ip-10-0-2-180.ec2.internal` (Instance 2 again)
   - **Pattern**: Traffic alternates between instances (round-robin)

5. **Monitor ASG Activity**:
   - Go to **Auto Scaling Groups → WebApp-ASG → Activity tab**
   - Observe instance launch events and timestamps
   - Verify both instances launched successfully

### Expected Results:
✅ Different hostnames on each refresh  
✅ Both instances receiving traffic  
✅ Instances in different AZs (1a and 1b)  
✅ ALB showing as Active  
✅ Health checks: 2/2 Healthy

---

## Step 5: Clean Up

### ⚠️ Important!
ALBs incur hourly charges (~$0.008/hr in us-east-1). Delete resources when done to avoid unexpected costs.

### Cleanup Steps:

1. **Delete the Auto Scaling Group**:
   - Go to **EC2 → Auto Scaling Groups → WebApp-ASG**
   - Click **Delete**
   - This automatically terminates all instances
   - Wait for deletion to complete

2. **Delete the Load Balancer**:
   - Go to **EC2 → Load Balancers → WebApp-ALB**
   - Click **Delete load balancer**
   - Confirm deletion

3. **Delete the Target Group**:
   - Go to **EC2 → Target Groups → WebServers-TG**
   - Click **Delete**
   - Confirm deletion

### Verification:
- No running EC2 instances
- No load balancers listed
- No target groups listed
- No charges will be incurred

---

## 📸 Screenshot Checklist

Capture these for your portfolio:

- ☑️ Auto Scaling Group showing 2 running instances
- ☑️ Auto Scaling Group health check: 2/2 Healthy
- ☑️ Load Balancer showing Active state
- ☑️ Activity history showing instance launch events
- ☑️ Browser showing page from Instance 1 (one hostname)
- ☑️ Browser showing page from Instance 2 (different hostname)
- ☑️ Browser showing Instance 1 again (proves load balancing)
- ☑️ Browser showing Instance 2 again (proves round-robin)

---

## 🎓 Learning Outcomes

After completing this lab, you understand:

1. **Launch Templates**: Reusable blueprints for EC2 instance configuration
2. **Load Balancing**: Distributing traffic across multiple instances
3. **Auto Scaling**: Automatic capacity management based on demand
4. **High Availability**: Spreading resources across availability zones
5. **Health Checks**: Ensuring only healthy instances receive traffic
6. **Infrastructure Monitoring**: Tracking instance status and events
7. **Cost Management**: Properly terminating resources to avoid charges

---

## 💡 Real-World Applications

- **E-commerce**: Handle Black Friday traffic spikes
- **SaaS Applications**: Scale API servers based on demand
- **Content Delivery**: Distribute user requests efficiently
- **Microservices**: Auto-scale individual service containers
- **Gaming**: Handle player load fluctuations

---

## 📚 Additional Resources

- [AWS Auto Scaling Documentation](https://docs.aws.amazon.com/autoscaling/)
- [Application Load Balancer Guide](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/)
- [EC2 Launch Templates](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-templates.html)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
