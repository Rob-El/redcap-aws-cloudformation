# Technology Stack

## Core AWS Services

### Compute and Application Platform
- **AWS Elastic Beanstalk**: Platform-as-a-Service for REDCap deployment
  - **Platform**: PHP 8.1 running on Amazon Linux 2
  - **Web Server**: Apache HTTP Server with mod_php
  - **Auto Scaling**: Dynamic scaling based on CPU utilization (20-80% thresholds)
  - **Load Balancing**: Application Load Balancer with session stickiness
  - **Health Monitoring**: Enhanced health reporting and monitoring
  - **Rolling Updates**: Zero-downtime deployments with rolling update strategy

- **Amazon EC2**: Virtual servers for application hosting
  - **Instance Types**: t3.micro to r5.24xlarge (configurable)
  - **Operating System**: Amazon Linux 2 with latest security patches
  - **Storage**: GP3 EBS volumes with encryption at rest
  - **Security**: IMDSv2 enforced, SSH access restricted
  - **Monitoring**: CloudWatch agent for detailed metrics

### Database Services
- **Amazon Aurora MySQL**: Managed relational database service
  - **Engine Version**: Aurora MySQL 5.7 (compatible with MySQL 5.7)
  - **Deployment**: Single-AZ or Multi-AZ configuration options
  - **Instance Classes**: db.t3.small to db.r5.24xlarge
  - **Storage**: Auto-scaling storage with encryption at rest
  - **Backup**: 35-day automated backup retention
  - **Performance**: Aurora Backtrack with 24-hour window
  - **Monitoring**: Performance Insights enabled
  - **Security**: VPC isolation, encryption in transit and at rest

### Storage Services
- **Amazon S3**: Object storage for REDCap file repository
  - **Encryption**: Server-side encryption with AES-256
  - **Access Control**: Bucket policies and IAM user permissions
  - **Versioning**: Object versioning for file history
  - **Lifecycle**: Intelligent tiering for cost optimization
  - **Security**: Public access blocked, secure HTTPS access only

- **Amazon EBS**: Block storage for EC2 instances
  - **Volume Type**: GP3 with configurable IOPS and throughput
  - **Encryption**: AWS KMS encryption at rest
  - **Snapshots**: Automated snapshots for backup and recovery
  - **Performance**: Optimized for database and application workloads

### Networking and Security
- **Amazon VPC**: Virtual Private Cloud with isolated network
  - **CIDR Blocks**: Configurable IP ranges (default: 10.105.32.0/20)
  - **Subnets**: Multi-tier architecture (public, application, database)
  - **Availability Zones**: Multi-AZ deployment for high availability
  - **Route Tables**: Separate routing for each subnet tier
  - **Internet Gateway**: Controlled internet access for public subnets
  - **Transit Gateway**: Enterprise connectivity and routing

- **AWS Security Groups**: Virtual firewalls for network access control
  - **Public Security Group**: HTTP/HTTPS access from specified CIDR ranges
  - **Application Security Group**: Internal communication and management
  - **Database Security Group**: Database access restricted to application tier
  - **Stateful Rules**: Automatic return traffic handling

- **Application Load Balancer**: Layer 7 load balancing and SSL termination
  - **SSL/TLS**: AWS Certificate Manager integration
  - **Health Checks**: Application-aware health monitoring
  - **Session Persistence**: Cookie-based session stickiness
  - **Security**: Integration with AWS WAF (optional)

### DNS and Certificate Management
- **Amazon Route 53**: DNS management and health checks
  - **Hosted Zones**: Public DNS zone management
  - **Record Types**: A records pointing to load balancer
  - **Health Checks**: Application availability monitoring
  - **Failover**: DNS-based failover capabilities

- **AWS Certificate Manager (ACM)**: SSL/TLS certificate management
  - **Domain Validation**: Automated certificate provisioning
  - **Auto Renewal**: Automatic certificate renewal
  - **Integration**: Native ALB and CloudFront integration
  - **Wildcard Support**: Subdomain certificate coverage

### Monitoring and Logging
- **Amazon CloudWatch**: Comprehensive monitoring and logging
  - **Metrics**: Custom and built-in performance metrics
  - **Logs**: Centralized log aggregation and analysis
  - **Alarms**: Threshold-based alerting and notifications
  - **Dashboards**: Real-time operational visibility
  - **Retention**: Configurable log retention periods

- **AWS CloudTrail**: API activity logging and auditing
  - **Event Logging**: All AWS API calls recorded
  - **Compliance**: Audit trail for security and compliance
  - **Integration**: CloudWatch Logs integration
  - **Encryption**: Log file encryption at rest

### Email Services
- **Amazon SES**: Simple Email Service for REDCap notifications
  - **SMTP Interface**: Standard SMTP integration
  - **Authentication**: SMTP credentials for secure access
  - **Delivery**: High deliverability rates
  - **Monitoring**: Bounce and complaint handling
  - **Compliance**: HIPAA-eligible service

## REDCap Application Stack

### REDCap Platform
- **Version Compatibility**: REDCap 8.9.3 or higher required
- **Source Distribution**: Vanderbilt University REDCap Consortium
- **Installation Method**: Automated download via REDCap API or S3 upload
- **Configuration**: Database-driven configuration management
- **Customization**: Hook functions and external modules support

### Web Application Technologies
- **PHP Runtime**: PHP 8.1 with optimized configuration
  - **Extensions**: MySQL, LDAP, OpenSSL, cURL, GD, mbstring
  - **Configuration**: Optimized for REDCap requirements
  - **Security**: Secure session handling and input validation
  - **Performance**: OPcache enabled for improved performance

- **Apache HTTP Server**: Web server with SSL/TLS support
  - **Modules**: mod_php, mod_ssl, mod_rewrite, mod_headers
  - **Configuration**: Optimized for PHP applications
  - **Security**: Security headers and SSL configuration
  - **Logging**: Access and error logging to CloudWatch

### Database Configuration
- **MySQL Compatibility**: Aurora MySQL 5.7 engine
- **Character Set**: UTF-8 (utf8mb4) for international character support
- **Storage Engine**: InnoDB with optimized parameters
- **Connection Pooling**: Persistent connections for performance
- **Query Optimization**: Optimized MySQL parameters for REDCap workloads

## Infrastructure as Code

### AWS CloudFormation
- **Template Format**: YAML-based CloudFormation templates
- **Nested Stacks**: Modular architecture with separate stack files
- **Parameters**: Configurable deployment options
- **Conditions**: Conditional resource creation based on parameters
- **Outputs**: Cross-stack references and resource sharing
- **Validation**: Template validation and drift detection

### Template Structure
```
redcap-aws-cloudformation/
├── 00-master-rc.yaml          # Master orchestration template
├── 01-rc-vpc.yaml             # VPC and networking infrastructure
├── 02-rc-elasticbeanstalk.yaml # Application platform and database
├── 03-rc-route53.yaml         # DNS and certificate management
├── eb-extensions/             # Elastic Beanstalk configuration
└── AmazonLinux2upgradescripts/ # Platform upgrade utilities
```

### Configuration Management
- **Elastic Beanstalk Extensions**: Application-specific configuration
- **Environment Variables**: Runtime configuration parameters
- **Configuration Files**: Automated file deployment and management
- **Hooks**: Pre and post-deployment script execution
- **Platform Hooks**: Amazon Linux 2 platform-specific configurations

## Development and Deployment Tools

### Source Code Management
- **Git**: Version control for infrastructure templates
- **Bitbucket**: Repository hosting and collaboration
- **Branching Strategy**: Feature branches with pull request workflow
- **Code Review**: Peer review process for all changes

### Deployment Pipeline
- **AWS CloudFormation**: Infrastructure deployment automation
- **Parameter Files**: Environment-specific configuration
- **Stack Policies**: Protection against accidental resource deletion
- **Change Sets**: Preview infrastructure changes before deployment
- **Rollback**: Automatic rollback on deployment failures

### Testing and Validation
- **CloudFormation Linting**: Template syntax and best practice validation
- **Security Scanning**: Infrastructure security assessment
- **Cost Analysis**: Deployment cost estimation and optimization
- **Performance Testing**: Load testing and performance validation

## Security and Compliance

### Encryption and Key Management
- **AWS KMS**: Customer-managed encryption keys
- **Encryption at Rest**: EBS, RDS, and S3 encryption
- **Encryption in Transit**: TLS 1.2+ for all communications
- **Key Rotation**: Automatic key rotation policies
- **Access Control**: IAM policies for key usage

### Identity and Access Management
- **IAM Roles**: Service-specific roles with least privilege
- **Instance Profiles**: EC2 instance access to AWS services
- **Cross-Account Access**: Controlled access between AWS accounts
- **MFA Requirements**: Multi-factor authentication for administrative access
- **Access Logging**: Comprehensive access audit trails

### Network Security
- **VPC Isolation**: Private network with controlled access
- **Security Groups**: Stateful firewall rules
- **Network ACLs**: Additional network-level security
- **VPC Flow Logs**: Network traffic monitoring and analysis
- **DDoS Protection**: AWS Shield Standard protection

### HIPAA Compliance Features
- **Business Associate Agreement**: AWS BAA coverage
- **Audit Logging**: Comprehensive activity logging
- **Data Encryption**: PHI encryption at rest and in transit
- **Access Controls**: Role-based access to PHI
- **Backup and Recovery**: Secure backup procedures
- **Incident Response**: Security incident handling procedures

## Performance and Scalability

### Auto Scaling Configuration
- **Scaling Metrics**: CPU utilization, memory usage, connection count
- **Scaling Policies**: Target tracking and step scaling
- **Health Checks**: Application and infrastructure health monitoring
- **Cooldown Periods**: Scaling stabilization and cost optimization
- **Instance Warm-up**: Gradual traffic routing to new instances

### Database Performance
- **Connection Pooling**: Optimized database connections
- **Query Optimization**: MySQL parameter tuning for REDCap
- **Read Replicas**: Optional read scaling for reporting workloads
- **Performance Insights**: Database performance monitoring
- **Automated Backups**: Point-in-time recovery capabilities

### Caching and Optimization
- **PHP OPcache**: Bytecode caching for improved performance
- **Session Management**: Optimized session storage and handling
- **Static Content**: S3 for file storage with optional CDN
- **Database Caching**: Query result caching where appropriate
- **Resource Optimization**: Right-sized instances and storage

## Operational Excellence

### Monitoring and Alerting
- **CloudWatch Metrics**: Comprehensive performance monitoring
- **Custom Metrics**: Application-specific monitoring
- **Log Aggregation**: Centralized logging and analysis
- **Alerting**: Proactive issue detection and notification
- **Dashboards**: Real-time operational visibility

### Backup and Recovery
- **Automated Backups**: Daily database and file system backups
- **Point-in-Time Recovery**: Database restoration to specific timestamps
- **Cross-Region Replication**: Disaster recovery capabilities
- **Backup Testing**: Regular backup restoration validation
- **Recovery Procedures**: Documented recovery processes

### Maintenance and Updates
- **Managed Platform Updates**: Automated OS and platform patching
- **Application Updates**: REDCap version upgrade procedures
- **Security Updates**: Rapid security patch deployment
- **Maintenance Windows**: Scheduled maintenance with minimal downtime
- **Change Management**: Controlled change deployment process

## Cost Optimization

### Resource Right-Sizing
- **Instance Types**: Appropriate sizing for workload requirements
- **Storage Optimization**: GP3 volumes with optimized IOPS
- **Database Sizing**: Right-sized RDS instances
- **Auto Scaling**: Dynamic scaling to match demand
- **Reserved Capacity**: Cost savings for predictable workloads

### Cost Monitoring
- **AWS Cost Explorer**: Detailed cost analysis and forecasting
- **Resource Tagging**: Cost allocation and tracking
- **Budget Alerts**: Proactive cost monitoring and alerting
- **Usage Reports**: Regular cost and usage analysis
- **Optimization Recommendations**: AWS Trusted Advisor insights

## Future Enhancements

### Planned Improvements
- **Container Migration**: ECS/Fargate deployment option
- **Multi-Region**: Cross-region deployment for disaster recovery
- **Advanced Monitoring**: AWS X-Ray distributed tracing
- **Security Enhancements**: AWS Config compliance monitoring
- **Performance**: ElastiCache for session and data caching

### Integration Opportunities
- **AWS Directory Service**: LDAP/Active Directory integration
- **AWS Secrets Manager**: Enhanced credential management
- **AWS Systems Manager**: Advanced configuration management
- **AWS Lambda**: Serverless functions for automation
- **Amazon API Gateway**: RESTful API management and throttling