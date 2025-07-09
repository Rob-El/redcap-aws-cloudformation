# System Architecture

## High-Level Architecture

```mermaid
graph TB
    subgraph "Internet"
        U[👥 Researchers & Users]
    end
    
    subgraph "AWS Cloud - REDCap Environment"
        subgraph "Public Tier"
            ALB[🔄 Application Load Balancer<br/>SSL Termination]
            R53[🌐 Route 53<br/>DNS Management]
            ACM[🔒 Certificate Manager<br/>SSL Certificates]
        end
        
        subgraph "Application Tier - Private Subnets"
            EB[📦 Elastic Beanstalk<br/>Auto Scaling Group]
            EC2A[🖥️ REDCap Instance A<br/>PHP/Apache]
            EC2B[🖥️ REDCap Instance B<br/>PHP/Apache]
        end
        
        subgraph "Data Tier - Private Subnets"
            RDS[🗄️ Aurora MySQL Cluster<br/>Multi-AZ]
            S3[📁 S3 File Repository<br/>Encrypted Storage]
        end
        
        subgraph "Security & Monitoring"
            SG[🛡️ Security Groups]
            CW[📊 CloudWatch<br/>Monitoring & Logs]
            SES[📧 Simple Email Service]
        end
        
        subgraph "Network Infrastructure"
            VPC[🏗️ VPC with Transit Gateway]
            GWLB[⚖️ Gateway Load Balancer<br/>Security Inspection]
        end
    end
    
    U --> R53
    R53 --> ALB
    ACM --> ALB
    ALB --> EB
    EB --> EC2A
    EB --> EC2B
    EC2A --> RDS
    EC2B --> RDS
    EC2A --> S3
    EC2B --> S3
    EC2A --> SES
    EC2B --> SES
    
    SG --> EC2A
    SG --> EC2B
    SG --> RDS
    CW --> EC2A
    CW --> EC2B
    CW --> RDS
    
    ALB --> GWLB
    GWLB --> VPC
    
    style U fill:#ff9999,color:#000
    style ALB fill:#4CAF50,color:#fff
    style R53 fill:#2196F3,color:#fff
    style ACM fill:#FF9800,color:#fff
    style EB fill:#9C27B0,color:#fff
    style EC2A fill:#607D8B,color:#fff
    style EC2B fill:#607D8B,color:#fff
    style RDS fill:#795548,color:#fff
    style S3 fill:#FF5722,color:#fff
    style SG fill:#E91E63,color:#fff
    style CW fill:#00BCD4,color:#fff
    style SES fill:#8BC34A,color:#fff
    style VPC fill:#3F51B5,color:#fff
    style GWLB fill:#FFC107,color:#000
```

## CloudFormation Stack Architecture

```mermaid
graph LR
    subgraph "Master Stack (00-master-rc.yaml)"
        MS[🎯 Master Orchestrator]
    end
    
    subgraph "VPC Stack (01-rc-vpc.yaml)"
        VPC[🏗️ VPC Infrastructure]
        SN[🔗 Subnets & Routing]
        SG[🛡️ Security Groups]
        IAM[👤 IAM Roles & Policies]
        GW[🌉 Transit Gateway]
    end
    
    subgraph "Application Stack (02-rc-elasticbeanstalk.yaml)"
        EB[📦 Elastic Beanstalk]
        RDS[🗄️ Aurora MySQL]
        S3B[📁 S3 Buckets]
        EC2[🖥️ Temp EC2 Builder]
    end
    
    subgraph "DNS Stack (03-rc-route53.yaml)"
        R53[🌐 Route 53 Records]
        DNS[📍 DNS Configuration]
    end
    
    MS --> VPC
    MS --> EB
    MS --> R53
    
    VPC --> SN
    VPC --> SG
    VPC --> IAM
    VPC --> GW
    
    EB --> RDS
    EB --> S3B
    EB --> EC2
    
    R53 --> DNS
    
    style MS fill:#e1f5fe,color:#000
    style VPC fill:#fff3e0,color:#000
    style EB fill:#e8f5e8,color:#000
    style R53 fill:#ffebee,color:#000
```

## Design Principles

### 1. Security by Design
- **Defense in Depth**: Multiple security layers from network to application
- **Zero Trust Architecture**: Verify every connection and transaction
- **Encryption Everywhere**: Data encrypted at rest and in transit
- **Least Privilege Access**: Minimal required permissions for all components
- **Network Isolation**: Private subnets with controlled internet access

### 2. High Availability and Resilience
- **Multi-AZ Deployment**: Resources distributed across availability zones
- **Auto Scaling**: Dynamic scaling based on demand and health checks
- **Load Balancing**: Traffic distribution with health monitoring
- **Database Clustering**: Aurora MySQL with automatic failover
- **Backup and Recovery**: Automated backups with point-in-time recovery

### 3. HIPAA Compliance Architecture
- **Data Encryption**: All PHI encrypted using AWS KMS
- **Access Logging**: Comprehensive audit trails for all data access
- **Network Security**: VPC isolation with security group controls
- **Administrative Safeguards**: IAM roles and policies for access control
- **Physical Safeguards**: AWS data center security and compliance

### 4. Cost Optimization
- **Right-Sizing**: Appropriate instance types for workload requirements
- **Auto Scaling**: Pay only for resources actually needed
- **Reserved Capacity**: Cost savings for predictable workloads
- **Storage Optimization**: Intelligent tiering and lifecycle policies

## Key Components

### VPC Infrastructure (`01-rc-vpc.yaml`)

#### Network Architecture
```mermaid
graph TB
    subgraph "VPC (10.105.32.0/20)"
        subgraph "Public Subnets"
            PUB1[📡 Public Subnet A<br/>10.105.34.0/24]
            PUB2[📡 Public Subnet B<br/>10.105.35.0/24]
        end
        
        subgraph "Application Subnets"
            APP1[🖥️ App Subnet A<br/>10.105.36.0/24]
            APP2[🖥️ App Subnet B<br/>10.105.37.0/24]
        end
        
        subgraph "Database Subnets"
            DB1[🗄️ Data Subnet A<br/>10.105.38.0/24]
            DB2[🗄️ Data Subnet B<br/>10.105.39.0/24]
        end
        
        subgraph "GWLB Subnets"
            GWLB1[⚖️ GWLB Subnet A<br/>10.105.32.0/24]
            GWLB2[⚖️ GWLB Subnet B<br/>10.105.33.0/24]
        end
        
        IGW[🌐 Internet Gateway]
        TGW[🔄 Transit Gateway]
        
        IGW --> PUB1
        IGW --> PUB2
        PUB1 --> APP1
        PUB2 --> APP2
        APP1 --> DB1
        APP2 --> DB2
        
        TGW --> DB1
        TGW --> DB2
        
        GWLB1 --> IGW
        GWLB2 --> IGW
    end
    
    style PUB1 fill:#4CAF50,color:#fff
    style PUB2 fill:#4CAF50,color:#fff
    style APP1 fill:#FF9800,color:#fff
    style APP2 fill:#FF9800,color:#fff
    style DB1 fill:#F44336,color:#fff
    style DB2 fill:#F44336,color:#fff
    style GWLB1 fill:#9C27B0,color:#fff
    style GWLB2 fill:#9C27B0,color:#fff
    style IGW fill:#2196F3,color:#fff
    style TGW fill:#607D8B,color:#fff
```

#### Security Groups
- **Public Security Group**: Controls ALB access (HTTP/HTTPS)
- **Application Security Group**: Controls EC2 instance access
- **Database Security Group**: Restricts database access to application tier only
- **Default Security Group**: Cleaned up and secured per AWS best practices

#### IAM Roles and Policies
- **Elastic Beanstalk Service Role**: Platform management permissions
- **EC2 Instance Profile**: Application-specific permissions
- **Temporary EC2 Role**: REDCap installation and configuration
- **S3 Access User**: File repository access for REDCap

### Application Platform (`02-rc-elasticbeanstalk.yaml`)

#### Elastic Beanstalk Configuration
```mermaid
graph TB
    subgraph "Elastic Beanstalk Environment"
        ALB[🔄 Application Load Balancer]
        ASG[📈 Auto Scaling Group]
        
        subgraph "EC2 Instances"
            EC2A[🖥️ REDCap Instance A<br/>PHP 8.1 + Apache]
            EC2B[🖥️ REDCap Instance B<br/>PHP 8.1 + Apache]
        end
        
        subgraph "Configuration"
            SSL[🔒 SSL/TLS Configuration]
            LOGS[📋 CloudWatch Logs]
            HEALTH[❤️ Health Monitoring]
        end
    end
    
    ALB --> EC2A
    ALB --> EC2B
    ASG --> EC2A
    ASG --> EC2B
    
    SSL --> ALB
    LOGS --> EC2A
    LOGS --> EC2B
    HEALTH --> EC2A
    HEALTH --> EC2B
    
    style ALB fill:#4CAF50,color:#fff
    style ASG fill:#FF9800,color:#fff
    style EC2A fill:#2196F3,color:#fff
    style EC2B fill:#2196F3,color:#fff
    style SSL fill:#E91E63,color:#fff
    style LOGS fill:#9C27B0,color:#fff
    style HEALTH fill:#00BCD4,color:#fff
```

#### Database Architecture
```mermaid
graph LR
    subgraph "Aurora MySQL Cluster"
        WRITER[🖊️ Writer Instance<br/>Primary Database]
        READER[📖 Reader Instance<br/>Read Replica]
        
        subgraph "Configuration"
            ENCRYPT[🔒 Encryption at Rest]
            BACKUP[💾 Automated Backups]
            MONITOR[📊 Performance Insights]
        end
    end
    
    WRITER --> READER
    ENCRYPT --> WRITER
    ENCRYPT --> READER
    BACKUP --> WRITER
    MONITOR --> WRITER
    MONITOR --> READER
    
    style WRITER fill:#4CAF50,color:#fff
    style READER fill:#2196F3,color:#fff
    style ENCRYPT fill:#E91E63,color:#fff
    style BACKUP fill:#FF9800,color:#fff
    style MONITOR fill:#9C27B0,color:#fff
```

#### REDCap Installation Process
```mermaid
sequenceDiagram
    participant CFN as CloudFormation
    participant EC2 as Temp EC2 Instance
    participant S3 as S3 Bucket
    participant RDS as Aurora MySQL
    participant EB as Elastic Beanstalk
    
    CFN->>EC2: Launch temporary instance
    EC2->>EC2: Download REDCap source
    EC2->>EC2: Configure EB extensions
    EC2->>S3: Upload configured package
    EC2->>RDS: Initialize database schema
    EC2->>RDS: Create REDCap users
    EC2->>CFN: Signal completion
    CFN->>EB: Deploy REDCap application
    EB->>S3: Download application package
    EB->>RDS: Connect to database
    EC2->>EC2: Terminate instance
```

### DNS and SSL Management (`03-rc-route53.yaml`)

#### DNS Architecture
- **Route 53 Hosted Zone**: Manages domain DNS records
- **A Record**: Points domain to ALB endpoint
- **Health Checks**: Monitors application availability
- **SSL Certificate**: ACM-managed certificate with auto-renewal

## Security Architecture

### Network Security
```mermaid
graph TB
    subgraph "Internet"
        USERS[👥 Users]
        THREATS[⚠️ Threats]
    end
    
    subgraph "Security Perimeter"
        WAF[🛡️ Web Application Firewall]
        GWLB[⚖️ Gateway Load Balancer<br/>Security Inspection]
        ALB[🔄 Application Load Balancer<br/>SSL Termination]
    end
    
    subgraph "Application Security"
        SG[🔒 Security Groups]
        NACL[🚧 Network ACLs]
        IAM[👤 IAM Roles]
    end
    
    subgraph "Data Security"
        KMS[🔑 AWS KMS<br/>Encryption Keys]
        ENCRYPT[🔐 Encryption at Rest]
        TLS[🔒 TLS in Transit]
    end
    
    USERS --> WAF
    THREATS --> WAF
    WAF --> GWLB
    GWLB --> ALB
    ALB --> SG
    SG --> NACL
    NACL --> IAM
    
    KMS --> ENCRYPT
    KMS --> TLS
    
    style USERS fill:#4CAF50,color:#fff
    style THREATS fill:#F44336,color:#fff
    style WAF fill:#FF9800,color:#fff
    style GWLB fill:#9C27B0,color:#fff
    style ALB fill:#2196F3,color:#fff
    style SG fill:#E91E63,color:#fff
    style NACL fill:#795548,color:#fff
    style IAM fill:#607D8B,color:#fff
    style KMS fill:#FF5722,color:#fff
    style ENCRYPT fill:#8BC34A,color:#fff
    style TLS fill:#00BCD4,color:#fff
```

### Data Flow Security
```mermaid
sequenceDiagram
    participant U as User
    participant ALB as Load Balancer
    participant EC2 as REDCap Instance
    participant RDS as Database
    participant S3 as File Storage
    participant SES as Email Service
    
    Note over U,SES: All communications encrypted with TLS
    U->>ALB: HTTPS Request (TLS 1.2+)
    ALB->>EC2: HTTPS Forward (Internal TLS)
    EC2->>RDS: Encrypted Connection (TLS)
    EC2->>S3: Encrypted API Calls (TLS)
    EC2->>SES: Encrypted SMTP (TLS)
    
    Note over RDS: Data encrypted at rest (AES-256)
    Note over S3: Files encrypted at rest (AES-256)
    
    S3->>EC2: Encrypted Response
    RDS->>EC2: Encrypted Response
    EC2->>ALB: HTTPS Response
    ALB->>U: HTTPS Response
```

## Scalability and Performance

### Auto Scaling Architecture
```mermaid
graph TB
    subgraph "Auto Scaling Triggers"
        CPU[📊 CPU Utilization > 80%]
        MEM[💾 Memory Utilization]
        CONN[🔗 Connection Count]
        CUSTOM[⚙️ Custom Metrics]
    end
    
    subgraph "Scaling Actions"
        SCALE_OUT[📈 Scale Out<br/>Add Instances]
        SCALE_IN[📉 Scale In<br/>Remove Instances]
        HEALTH[❤️ Health Checks]
    end
    
    subgraph "Load Distribution"
        ALB[🔄 Application Load Balancer]
        STICKY[🍪 Session Stickiness]
        HEALTH_CHECK[🏥 Health Monitoring]
    end
    
    CPU --> SCALE_OUT
    MEM --> SCALE_OUT
    CONN --> SCALE_OUT
    CUSTOM --> SCALE_OUT
    
    SCALE_OUT --> ALB
    SCALE_IN --> ALB
    HEALTH --> ALB
    
    ALB --> STICKY
    ALB --> HEALTH_CHECK
    
    style CPU fill:#FF5722,color:#fff
    style MEM fill:#FF9800,color:#fff
    style CONN fill:#4CAF50,color:#fff
    style CUSTOM fill:#9C27B0,color:#fff
    style SCALE_OUT fill:#2196F3,color:#fff
    style SCALE_IN fill:#607D8B,color:#fff
    style HEALTH fill:#E91E63,color:#fff
    style ALB fill:#00BCD4,color:#fff
    style STICKY fill:#795548,color:#fff
    style HEALTH_CHECK fill:#8BC34A,color:#fff
```

### Performance Optimization
- **Database Performance**: Aurora MySQL with optimized parameters
- **Application Caching**: PHP OPcache and session management
- **Static Content**: S3 for file storage with optional CloudFront CDN
- **Connection Pooling**: Optimized database connections
- **Resource Right-Sizing**: Instance types matched to workload requirements

## Disaster Recovery and Backup

### Backup Strategy
```mermaid
graph TB
    subgraph "Automated Backups"
        RDS_BACKUP[🗄️ RDS Automated Backups<br/>35-day retention]
        S3_VERSIONING[📁 S3 Object Versioning<br/>File history]
        SNAPSHOT[📸 EBS Snapshots<br/>Instance backups]
    end
    
    subgraph "Point-in-Time Recovery"
        PIT[⏰ Point-in-Time Recovery<br/>Database restoration]
        BACKTRACK[⏪ Aurora Backtrack<br/>24-hour window]
    end
    
    subgraph "Cross-Region Replication"
        CROSS_REGION[🌍 Cross-Region Backup<br/>Disaster recovery]
        S3_REPLICATION[🔄 S3 Cross-Region<br/>File replication]
    end
    
    RDS_BACKUP --> PIT
    RDS_BACKUP --> BACKTRACK
    S3_VERSIONING --> S3_REPLICATION
    SNAPSHOT --> CROSS_REGION
    
    style RDS_BACKUP fill:#4CAF50,color:#fff
    style S3_VERSIONING fill:#FF9800,color:#fff
    style SNAPSHOT fill:#2196F3,color:#fff
    style PIT fill:#9C27B0,color:#fff
    style BACKTRACK fill:#E91E63,color:#fff
    style CROSS_REGION fill:#607D8B,color:#fff
    style S3_REPLICATION fill:#795548,color:#fff
```

## Monitoring and Observability

### Monitoring Architecture
```mermaid
graph TB
    subgraph "Application Monitoring"
        CW_LOGS[📋 CloudWatch Logs<br/>Application logs]
        CW_METRICS[📊 CloudWatch Metrics<br/>Performance data]
        HEALTH[❤️ Health Checks<br/>Availability monitoring]
    end
    
    subgraph "Infrastructure Monitoring"
        EC2_METRICS[🖥️ EC2 Metrics<br/>Instance performance]
        RDS_METRICS[🗄️ RDS Metrics<br/>Database performance]
        ALB_METRICS[🔄 ALB Metrics<br/>Load balancer stats]
    end
    
    subgraph "Alerting"
        ALARMS[🚨 CloudWatch Alarms<br/>Threshold alerts]
        SNS[📧 SNS Notifications<br/>Alert delivery]
        DASHBOARD[📈 CloudWatch Dashboard<br/>Visual monitoring]
    end
    
    CW_LOGS --> ALARMS
    CW_METRICS --> ALARMS
    EC2_METRICS --> ALARMS
    RDS_METRICS --> ALARMS
    ALB_METRICS --> ALARMS
    
    ALARMS --> SNS
    ALARMS --> DASHBOARD
    HEALTH --> DASHBOARD
    
    style CW_LOGS fill:#4CAF50,color:#fff
    style CW_METRICS fill:#FF9800,color:#fff
    style HEALTH fill:#2196F3,color:#fff
    style EC2_METRICS fill:#9C27B0,color:#fff
    style RDS_METRICS fill:#E91E63,color:#fff
    style ALB_METRICS fill:#607D8B,color:#fff
    style ALARMS fill:#F44336,color:#fff
    style SNS fill:#795548,color:#fff
    style DASHBOARD fill:#00BCD4,color:#fff
```