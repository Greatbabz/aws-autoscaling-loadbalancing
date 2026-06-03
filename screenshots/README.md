# Screenshots Directory Structure

This directory contains organized verification screenshots from the AWS Auto Scaling & Load Balancing lab.

## Directory Organization

### alb/
Application Load Balancer screenshots showing ALB configuration and DNS responses.
- `01-alb-active-status.png` - ALB dashboard in Active state
- `02-alb-dns-response.png` - First browser response from ALB

### asg/
Auto Scaling Group screenshots showing ASG health and activity.
- `01-asg-healthy-instances.png` - ASG dashboard with 2/2 healthy instances
- `02-asg-activity-history.png` - ASG activity log showing instance launches

### load-balancing/
Load balancing verification screenshots showing round-robin traffic distribution.
- `01-instance1-first-request.png` - First request to Instance 1
- `02-instance2-second-request.png` - Second request to Instance 2 (load balancing confirmed)
- `03-instance1-third-request.png` - Third request back to Instance 1
- `04-instance2-fourth-request.png` - Fourth request back to Instance 2

## How to Use These Screenshots

1. **Portfolio Documentation**: Include these in your portfolio project README
2. **Technical Interviews**: Use as evidence of hands-on AWS experience
3. **LinkedIn Posts**: Share with technical content about infrastructure
4. **Case Studies**: Reference when discussing scalable architecture design

## Screenshot Checklist

- ☐ `alb/01-alb-active-status.png` - ALB is Active
- ☐ `alb/02-alb-dns-response.png` - ALB routing traffic
- ☐ `asg/01-asg-healthy-instances.png` - ASG has 2/2 healthy instances
- ☐ `asg/02-asg-activity-history.png` - ASG launched instances successfully
- ☐ `load-balancing/01-instance1-first-request.png` - Instance 1 response
- ☐ `load-balancing/02-instance2-second-request.png` - Instance 2 response (different hostname)
- ☐ `load-balancing/03-instance1-third-request.png` - Round-robin back to Instance 1
- ☐ `load-balancing/04-instance2-fourth-request.png` - Round-robin back to Instance 2

## What Each Screenshot Demonstrates

### ALB Screenshots
Shows successful load balancer deployment and traffic routing capability.

### ASG Screenshots
Demonstrates auto-scaling group is maintaining desired capacity and instance health.

### Load Balancing Screenshots
Proves traffic is being distributed across multiple instances in different availability zones using round-robin algorithm.

## Technical Value

These screenshots collectively demonstrate:
- Infrastructure deployment and configuration skills
- Understanding of load balancing concepts
- Ability to verify AWS infrastructure health
- Knowledge of multi-AZ architecture
- Hands-on experience with AWS console
