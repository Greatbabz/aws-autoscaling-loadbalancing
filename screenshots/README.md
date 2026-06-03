# Screenshots - AWS Auto Scaling & Load Balancing Lab

## Overview
This directory contains all verification screenshots from the Week 4 Day 4 AWS Cloud Accelerator lab.

## Screenshot Guide

### 01-load-balancing-demo.png
**Load Balancing in Action - Instance 1**
- Shows browser response from first instance
- Hostname: `ip-10-0-1-79.ec2.internal`
- Deployed in us-east-1a (AZ-A)

### 02-asg-healthy-instances.png
**Auto Scaling Group Dashboard**
- Name: WebApp-ASG
- Status: At desired capacity
- Instances: 2/2 Healthy
- Availability Zones: 2 AZs
- Launch Template: WebServer-LaunchTemplate
- Min: 1 | Desired: 2 | Max: 4

### 03-activity-history.png
**ASG Activity History**
- Shows successful EC2 instance launches
- 2 instances launched successfully
- Date: Wednesday, June 03 2026
- Status: ✅ Successful

### 04-alb-active.png
**Application Load Balancer Status**
- Name: WebApp-ALB
- State: Active ✅
- Type: Application
- Scheme: Internet-facing
- Protocol: HTTP
- Port: 80
- VPC: 2 Availability Zones

### 05-load-balancing-instance1.png
**Round-Robin Distribution - Instance 1**
- First browser refresh
- Hostname: `ip-10-0-1-79.ec2.internal`
- AZ-A instance

### 06-load-balancing-instance2.png
**Round-Robin Distribution - Instance 2**
- Second browser refresh
- Hostname: `ip-10-0-2-180.ec2.internal`
- AZ-B instance
- **✅ Different hostname confirms load balancing!**

### 07-load-balancing-roundrobin1.png
**Continued Round-Robin - Back to Instance 1**
- Third browser refresh
- Hostname: `ip-10-0-1-79.ec2.internal`
- Confirms traffic distribution pattern

### 08-load-balancing-roundrobin2.png
**Continued Round-Robin - Back to Instance 2**
- Fourth browser refresh
- Hostname: `ip-10-0-2-180.ec2.internal`
- Demonstrates consistent load balancing

## Key Findings

✅ **Load Balancing Verified**: Traffic successfully distributed across both instances
✅ **Multi-AZ Deployment**: Instances span two availability zones
✅ **Health Checks Passing**: 2/2 instances healthy and serving traffic
✅ **Auto Scaling Active**: ASG maintaining desired capacity of 2 instances
✅ **Round-Robin Distribution**: ALB alternating traffic between instances

## Portfolio Value

These screenshots demonstrate:
- Successful infrastructure deployment
- Understanding of load balancing concepts
- Ability to verify and troubleshoot AWS infrastructure
- Professional documentation practices
