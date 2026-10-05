---
inclusion: auto
name: net-security-baseline
description: VPC, security group, and TLS baseline for AWS networks. Use when designing or changing VPC, subnet, security group, NACL, or load balancer configuration.
tags:
  - type/steering
  - aws/security
  - aws/networking
  - tool/terraform
  - tool/cloudformation
---

# Network security baseline

Segment the network, expose as little as possible, log traffic, and encrypt in transit. Apply this to VPC, subnet, security group, NACL, and load balancer definitions.

## Rules

- No `0.0.0.0/0` or `::/0` ingress to SSH (22), RDP (3389), or data-store ports (3306, 5432, 1433, 6379, 27017, 9200).
- Public subnets hold internet-facing load balancers and NAT gateways. Application and data tiers stay in private subnets with no direct route to an internet gateway.
- East-west rules reference another security group instead of a wide CIDR when you can.
- Scope egress. A data tier does not need all-ports egress to the internet.
- Turn on VPC Flow Logs (CloudWatch Logs or S3) for every VPC you care about.
- Send AWS API traffic through VPC endpoints (gateway endpoints for S3 and DynamoDB, interface endpoints for other services) when the workload should not depend on the public path.
- Terminate TLS at the edge (ALB or CloudFront, with AWS WAF if the app is on the internet) and use TLS 1.2 or newer to the target.
- Use SSM Session Manager for admin access instead of opening SSH or RDP.
- Treat NACLs as a coarse second layer. Keep the rules few and reviewed.
- NAT gateways cost money per hour and per gigabyte. One per AZ is a production choice, not the default for a sandbox.

## Do not open SSH to the world

```hcl
ingress {
  from_port   = 22
  to_port     = 22
  protocol    = "tcp"
  cidr_blocks = ["0.0.0.0/0"] # SSH open to the world
}
```

## Security group to security group

```hcl
resource "aws_security_group" "db" {
  name   = "db-tier"
  vpc_id = aws_vpc.main.id
}

resource "aws_security_group_rule" "db_from_app" {
  type                     = "ingress"
  from_port                = 5432
  to_port                  = 5432
  protocol                 = "tcp"
  security_group_id        = aws_security_group.db.id
  source_security_group_id = aws_security_group.app.id
  description              = "Postgres from the app tier only"
}
```

## VPC Flow Logs

```hcl
resource "aws_flow_log" "main" {
  vpc_id          = aws_vpc.main.id
  traffic_type    = "ALL"
  log_destination = aws_cloudwatch_log_group.flow.arn
  iam_role_arn    = aws_iam_role.flow_logs.arn
}
```

## Gateway and interface endpoints

```hcl
resource "aws_vpc_endpoint" "s3" {
  vpc_id            = aws_vpc.main.id
  service_name      = "com.amazonaws.${var.region}.s3"
  vpc_endpoint_type = "Gateway"
  route_table_ids   = aws_route_table.private[*].id
}

resource "aws_vpc_endpoint" "ssm" {
  vpc_id              = aws_vpc.main.id
  service_name        = "com.amazonaws.${var.region}.ssm"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = aws_subnet.private[*].id
  security_group_ids  = [aws_security_group.endpoints.id]
  private_dns_enabled = true
}
```

## HTTPS listener

```yaml
Resources:
  HttpsListener:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      Protocol: HTTPS
      Port: 443
      SslPolicy: ELBSecurityPolicy-TLS13-1-2-2021-06
      Certificates: [{ CertificateArn: !Ref Cert }]
      DefaultActions: [{ Type: forward, TargetGroupArn: !Ref Tg }]
```

## Related

- [AWS security standards](../cloudformation/cfn-security-standards.md)
- [Terraform standards](../terraform/tf-standards.md)
