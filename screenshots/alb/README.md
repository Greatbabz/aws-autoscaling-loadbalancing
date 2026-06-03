# ALB Screenshots

## 01-alb-active-status.png
Application Load Balancer dashboard showing:
- Name: WebApp-ALB
- State: Active ✅
- Type: Application
- Scheme: Internet-facing
- Protocol: HTTP
- Port: 80
- VPC: 2 Availability Zones
- Security Group: SG-LoadBalancer

## 02-alb-dns-response.png
First ALB DNS response from browser showing:
- Initial request to ALB DNS name
- Response from first instance (ip-10-0-1-79.ec2.internal)
- Confirms ALB is routing traffic successfully
