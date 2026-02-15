# RetailIQ - System Design Document
## AI-Powered Retail Demand Forecasting & Pricing Intelligence Platform

---

## Document Information
- **Version:** 1.0
- **Date:** February 15, 2026
- **Status:** Design Specification
- **Architecture Type:** Cloud-Native Microservices on AWS
- **Target Scale:** 10,000+ concurrent users, 10M+ SKUs, 100M+ daily transactions

---

## Executive Summary

RetailIQ is designed as a scalable, cloud-native SaaS platform leveraging AWS services and advanced AI/ML capabilities to deliver enterprise-grade demand forecasting and pricing intelligence to small and medium retailers. The architecture emphasizes:

- **Scalability:** Horizontal scaling supporting 100,000+ users
- **Reliability:** 99.9% uptime with automated failover and disaster recovery
- **Performance:** Sub-2-second dashboard loads, <5-minute forecast generation
- **Security:** End-to-end encryption, SOC 2 compliance-ready architecture
- **Cost Efficiency:** Serverless-first approach with intelligent resource optimization
- **AI/ML Excellence:** Production-grade ML pipeline with continuous learning

---

## 1. System Architecture Overview

### 1.1 Architecture Principles

The RetailIQ platform follows these core architectural principles:

1. **Microservices Architecture:** Loosely coupled services enabling independent scaling and deployment
2. **Event-Driven Design:** Asynchronous communication using message queues and event streams
3. **Serverless-First:** Leverage AWS Lambda and managed services to minimize operational overhead
4. **API-First:** All functionality exposed through well-documented REST APIs
5. **Multi-Tenancy:** Secure tenant isolation with shared infrastructure
6. **Cloud-Native:** Built specifically for AWS with infrastructure as code
7. **Security by Design:** Defense in depth with encryption, IAM, and network isolation
8. **Observability:** Comprehensive monitoring, logging, and tracing
9. **Resilience:** Fault tolerance with graceful degradation and circuit breakers
10. **Cost Optimization:** Right-sizing resources with auto-scaling and spot instances

### 1.2 Key Architectural Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Cloud Provider | AWS | Comprehensive ML services, global reach, startup credits |
| Container Orchestration | Amazon ECS with Fargate | Serverless containers, lower operational complexity than EKS |
| Primary Database | Amazon RDS PostgreSQL | ACID compliance, strong consistency, JSON support |
| Cache Layer | Amazon ElastiCache Redis | High-performance caching, session management |
| Message Queue | Amazon SQS + SNS | Fully managed, reliable, cost-effective |
| API Gateway | AWS API Gateway | Managed service, built-in throttling, API versioning |
| ML Platform | Amazon SageMaker | End-to-end ML lifecycle, managed infrastructure |
| Data Lake | Amazon S3 | Scalable, durable, cost-effective storage |
| Real-time Streaming | Amazon Kinesis | Real-time data ingestion, integration with analytics |
| Frontend Hosting | CloudFront + S3 | Global CDN, low latency, cost-effective |
| Authentication | AWS Cognito | Managed user pools, MFA, SSO support |
| Infrastructure | AWS CDK (TypeScript) | Type-safe IaC, reusable constructs |

---

## 2. High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           CLIENT LAYER                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │   Web App    │  │  Mobile App  │  │  POS Systems │  │  External    │   │
│  │  (React SPA) │  │ (React Native│  │  (REST API)  │  │  Integrations│   │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘   │
│         │                  │                  │                  │           │
└─────────┼──────────────────┼──────────────────┼──────────────────┼───────────┘
          │                  │                  │                  │
          └──────────────────┴──────────────────┴──────────────────┘
                                      │
┌─────────────────────────────────────┼─────────────────────────────────────────┐
│                    EDGE & SECURITY LAYER                                      │
├─────────────────────────────────────┼─────────────────────────────────────────┤
│                                     │                                         │
│  ┌──────────────────────────────────▼──────────────────────────────────┐    │
│  │              Amazon CloudFront (CDN)                                 │    │
│  │  - Global content delivery  - SSL/TLS termination                   │    │
│  │  - DDoS protection (Shield) - WAF rules                             │    │
│  └──────────────────────────────────┬──────────────────────────────────┘    │
│                                     │                                         │
│  ┌──────────────────────────────────▼──────────────────────────────────┐    │
│  │              AWS API Gateway (REST + WebSocket)                      │    │
│  │  - Rate limiting  - Request validation  - API versioning            │    │
│  │  - Authentication - Throttling          - Request/Response transform│    │
│  └──────────────────────────────────┬──────────────────────────────────┘    │
│                                     │                                         │
└─────────────────────────────────────┼─────────────────────────────────────────┘
                                      │
┌─────────────────────────────────────┼─────────────────────────────────────────┐
│                    APPLICATION LAYER (VPC)                                    │
├─────────────────────────────────────┼─────────────────────────────────────────┤
│                                     │                                         │
│  ┌──────────────────────────────────▼──────────────────────────────────┐    │
│  │           Application Load Balancer (ALB)                            │    │
│  │  - Health checks  - SSL termination  - Path-based routing           │    │
│  └──────────────────────────────────┬──────────────────────────────────┘    │
│                                     │                                         │
│         ┌───────────────────────────┼───────────────────────────┐           │
│         │                           │                           │           │
│  ┌──────▼────────┐  ┌───────────────▼──────────┐  ┌───────────▼────────┐  │
│  │  Auth Service │  │  API Services (ECS)      │  │  Background Jobs   │  │
│  │  (Lambda)     │  │  ┌────────────────────┐  │  │  (Lambda + Batch)  │  │
│  │               │  │  │ User Management    │  │  │                    │  │
│  │ - JWT tokens  │  │  │ Data Ingestion     │  │  │ - ETL pipelines    │  │
│  │ - MFA         │  │  │ Forecast Service   │  │  │ - Report generation│  │
│  │ - SSO         │  │  │ Pricing Service    │  │  │ - Data cleanup     │  │
│  └───────────────┘  │  │ Inventory Service  │  │  └────────────────────┘  │
│                     │  │ Analytics Service  │  │                           │
│                     │  │ Notification Svc   │  │                           │
│                     │  └────────────────────┘  │                           │
│                     └─────────────────────────┘                           │
│                                     │                                         │
└─────────────────────────────────────┼─────────────────────────────────────────┘
                                      │
┌─────────────────────────────────────┼─────────────────────────────────────────┐
│                    DATA & ML LAYER                                            │
├─────────────────────────────────────┼─────────────────────────────────────────┤
│                                     │                                         │
│  ┌──────────────────────────────────▼──────────────────────────────────┐    │
│  │              Data Ingestion Pipeline                                 │    │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐   │    │
│  │  │  Kinesis   │→ │  Lambda    │→ │  Glue ETL  │→ │  S3 Data   │   │    │
│  │  │  Streams   │  │  Transform │  │  Jobs      │  │  Lake      │   │    │
│  │  └────────────┘  └────────────┘  └────────────┘  └────────────┘   │    │
│  └──────────────────────────────────────────────────────────────────────┘    │
│                                                                               │
│  ┌──────────────────────────────────────────────────────────────────────┐    │
│  │              ML Pipeline (Amazon SageMaker)                          │    │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐   │    │
│  │  │  Feature   │→ │  Training  │→ │  Model     │→ │  Inference │   │    │
│  │  │  Store     │  │  Pipeline  │  │  Registry  │  │  Endpoints │   │    │
│  │  └────────────┘  └────────────┘  └────────────┘  └────────────┘   │    │
│  │                                                                      │    │
│  │  ┌────────────┐  ┌────────────┐                                     │    │
│  │  │  Model     │  │  Experiment│                                     │    │
│  │  │  Monitor   │  │  Tracking  │                                     │    │
│  │  └────────────┘  └────────────┘                                     │    │
│  └──────────────────────────────────────────────────────────────────────┘    │
│                                                                               │
└───────────────────────────────────────────────────────────────────────────────┘
                                      │
┌─────────────────────────────────────┼─────────────────────────────────────────┐
│                    DATA STORAGE LAYER                                         │
├─────────────────────────────────────┼─────────────────────────────────────────┤
│                                     │                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │  RDS         │  │  DynamoDB    │  │  ElastiCache │  │  S3 Buckets  │   │
│  │  PostgreSQL  │  │              │  │  Redis       │  │              │   │
│  │              │  │  - Sessions  │  │              │  │  - Raw data  │   │
│  │  - Users     │  │  - Events    │  │  - Cache     │  │  - Models    │   │
│  │  - Products  │  │  - Metrics   │  │  - Sessions  │  │  - Reports   │   │
│  │  - Forecasts │  │  - Logs      │  │  - Rate      │  │  - Backups   │   │
│  │  - Pricing   │  │              │  │    limiting  │  │  - Archives  │   │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘   │
│                                                                               │
└───────────────────────────────────────────────────────────────────────────────┘
                                      │
┌─────────────────────────────────────┼─────────────────────────────────────────┐
│                    INTEGRATION & MESSAGING LAYER                              │
├─────────────────────────────────────┼─────────────────────────────────────────┤
│                                     │                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │  SQS Queues  │  │  SNS Topics  │  │  EventBridge │  │  Step        │   │
│  │              │  │              │  │              │  │  Functions   │   │
│  │  - Jobs      │  │  - Alerts    │  │  - Events    │  │              │   │
│  │  - Tasks     │  │  - Webhooks  │  │  - Schedules │  │  - Workflows │   │
│  │  - DLQ       │  │  - Email/SMS │  │  - Rules     │  │  - Orchestr. │   │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘   │
│                                                                               │
└───────────────────────────────────────────────────────────────────────────────┘
                                      │
┌─────────────────────────────────────┼─────────────────────────────────────────┐
│                    OBSERVABILITY & OPERATIONS LAYER                           │
├─────────────────────────────────────┼─────────────────────────────────────────┤
│                                     │                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │  CloudWatch  │  │  X-Ray       │  │  CloudTrail  │  │  Config      │   │
│  │              │  │              │  │              │  │              │   │
│  │  - Metrics   │  │  - Tracing   │  │  - Audit     │  │  - Compliance│   │
│  │  - Logs      │  │  - Service   │  │  - Security  │  │  - Changes   │   │
│  │  - Alarms    │  │    Map       │  │  - API calls │  │  - Inventory │   │
│  │  - Dashboards│  │  - Latency   │  │              │  │              │   │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘   │
│                                                                               │
└───────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Data Flow Explanation

### 3.1 User Authentication Flow

```
1. User → CloudFront → API Gateway → Cognito
2. Cognito validates credentials, issues JWT tokens
3. JWT stored in ElastiCache for session management
4. Subsequent requests validated via API Gateway authorizer
5. Token refresh handled automatically by Cognito
```

### 3.2 Data Ingestion Flow

```
Real-time Ingestion:
POS/E-commerce → API Gateway → Lambda → Kinesis Stream → Lambda Processor
→ Data Validation → S3 Raw → Glue ETL → S3 Processed → RDS/DynamoDB

Batch Ingestion:
CSV Upload → S3 Landing → EventBridge → Lambda → Validation
→ Glue ETL → S3 Processed → RDS → SNS Notification
```

### 3.3 Forecast Generation Flow

```
1. EventBridge triggers daily forecast job (Step Functions)
2. Step Functions orchestrates:
   a. Fetch data from RDS/S3 (Lambda)
   b. Prepare features (SageMaker Processing Job)
   c. Load features to Feature Store
   d. Invoke SageMaker endpoint for inference
   e. Post-process predictions (Lambda)
   f. Store forecasts in RDS
   g. Update cache (ElastiCache)
   h. Send notifications (SNS)
3. Results available via API within 5 minutes
```

### 3.4 Pricing Intelligence Flow

```
1. Competitor price scraping (Lambda on schedule)
2. Store raw prices in S3
3. Price analysis ML model (SageMaker)
4. Generate pricing recommendations
5. Store in RDS with confidence scores
6. Alert users via SNS if significant changes
7. Dashboard displays recommendations via API
```

### 3.5 Dashboard Query Flow

```
User Request → CloudFront → API Gateway → ALB → ECS Service
→ Check ElastiCache (cache hit: return)
→ Cache miss: Query RDS/DynamoDB
→ Aggregate data (Lambda if complex)
→ Store in ElastiCache (TTL: 5 min)
→ Return to user (< 2 seconds)
```

---

## 4. AI/ML Pipeline Design

### 4.1 ML Architecture Overview

The ML pipeline follows MLOps best practices with automated training, deployment, and monitoring.

```
┌─────────────────────────────────────────────────────────────────┐
│                    ML PIPELINE ARCHITECTURE                      │
└─────────────────────────────────────────────────────────────────┘

┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Data       │────▶│   Feature    │────▶│   Training   │
│   Sources    │     │   Engineering│     │   Pipeline   │
└──────────────┘     └──────────────┘     └──────────────┘
                                                  │
                                                  ▼
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Model      │◀────│   Model      │◀────│   Model      │
│   Deployment │     │   Validation │     │   Registry   │
└──────────────┘     └──────────────┘     └──────────────┘
       │
       ▼
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Inference  │────▶│   Model      │────▶│   Feedback   │
│   Endpoints  │     │   Monitoring │     │   Loop       │
└──────────────┘     └──────────────┘     └──────────────┘
```

### 4.2 Feature Engineering Pipeline

**Feature Store (SageMaker Feature Store):**
- Centralized repository for ML features
- Online store (low-latency) for real-time inference
- Offline store (S3) for training
- Feature versioning and lineage tracking

**Feature Categories:**

1. **Time-based features:** Day of week, month, season, holidays, promotions
2. **Product features:** Category, price, brand, attributes, lifecycle stage
3. **Historical features:** Sales trends, moving averages, lag features
4. **External features:** Weather, local events, economic indicators
5. **Derived features:** Price elasticity, seasonality index, trend strength

**Feature Pipeline (AWS Glue + Lambda):**
```python
# Pseudo-code for feature engineering
def create_features(raw_data):
    features = {
        'lag_features': calculate_lags(raw_data, [7, 14, 30, 90]),
        'rolling_stats': calculate_rolling_stats(raw_data, [7, 30]),
        'seasonality': extract_seasonality(raw_data),
        'trend': calculate_trend(raw_data),
        'price_features': calculate_price_metrics(raw_data),
        'external': enrich_external_data(raw_data)
    }
    return features
```

### 4.3 Model Training Pipeline

**SageMaker Pipelines Workflow:**

```
Step 1: Data Validation
- Check data quality, completeness, schema
- Detect anomalies and outliers
- Generate data quality report

Step 2: Data Preprocessing
- Handle missing values
- Normalize/standardize features
- Split train/validation/test sets (70/15/15)

Step 3: Feature Engineering
- Create time-series features
- Generate interaction features
- Feature selection (top 50 features)

Step 4: Model Training (Parallel)
├─ ARIMA/SARIMA (statistical baseline)
├─ Prophet (Facebook's time-series model)
├─ XGBoost (gradient boosting)
├─ LSTM (deep learning for sequences)
└─ Ensemble (weighted combination)

Step 5: Model Evaluation
- Calculate MAPE, RMSE, MAE
- Validate on holdout set
- Compare against baseline
- Generate evaluation report

Step 6: Model Registration
- Register best model in Model Registry
- Tag with metadata (accuracy, version, date)
- Approve for production deployment

Step 7: Model Deployment
- Deploy to SageMaker endpoint
- Configure auto-scaling
- Enable data capture for monitoring
```

### 4.4 Forecasting Models

**Model Ensemble Strategy:**

| Model | Use Case | Weight | Accuracy Target |
|-------|----------|--------|-----------------|
| ARIMA/SARIMA | Stable products with clear seasonality | 20% | 80-85% |
| Prophet | Products with holidays/events impact | 25% | 82-87% |
| XGBoost | Products with many features/drivers | 30% | 85-90% |
| LSTM | Complex patterns, long sequences | 25% | 83-88% |

**Model Selection Logic:**
```python
def select_model(product_data):
    if product_data.history_length < 90:
        return "Prophet"  # Better for limited data
    elif product_data.seasonality_strength > 0.7:
        return "SARIMA"  # Strong seasonality
    elif product_data.feature_count > 20:
        return "XGBoost"  # Many features
    else:
        return "Ensemble"  # Default to ensemble
```

### 4.5 Pricing Intelligence Models

**Price Elasticity Model (XGBoost):**
- Predict demand change based on price changes
- Features: historical prices, competitor prices, seasonality
- Output: Elasticity coefficient per product

**Dynamic Pricing Model (Reinforcement Learning):**
- Agent learns optimal pricing policy
- State: inventory level, demand forecast, competitor prices
- Action: price adjustment (-10% to +10%)
- Reward: profit margin × sales volume

**Competitor Price Prediction (LSTM):**
- Predict competitor price movements
- Features: historical competitor prices, market trends
- Output: Expected competitor prices (7-day horizon)

### 4.6 Model Monitoring & Retraining

**Monitoring Metrics:**
- Forecast accuracy (MAPE) tracked daily
- Model drift detection (data distribution changes)
- Prediction latency and throughput
- Feature importance changes

**Automated Retraining Triggers:**
1. Accuracy drops below 85% for 3 consecutive days
2. Significant data drift detected (KL divergence > 0.1)
3. Weekly scheduled retraining (every Sunday)
4. Manual trigger by data science team

**A/B Testing Framework:**
- Shadow mode: New model runs alongside production
- Compare predictions for 7 days
- Gradual rollout: 10% → 50% → 100% traffic
- Automatic rollback if accuracy degrades

---

## 5. AWS Services Used

### 5.1 Compute Services

**Amazon ECS with Fargate:**
- **Purpose:** Run containerized microservices
- **Services:** API services, background workers
- **Configuration:** Auto-scaling (2-20 tasks), health checks
- **Cost:** ~$150-500/month (startup scale)

**AWS Lambda:**
- **Purpose:** Serverless functions for event processing
- **Use Cases:** Data transformation, API authorizers, scheduled jobs
- **Configuration:** 1GB memory, 5-minute timeout, provisioned concurrency
- **Cost:** ~$50-200/month (within free tier initially)

**Amazon SageMaker:**
- **Purpose:** End-to-end ML platform
- **Components:** Training jobs, endpoints, pipelines, feature store
- **Instance Types:** ml.m5.xlarge (training), ml.t2.medium (inference)
- **Cost:** ~$300-1000/month (depends on training frequency)

**AWS Batch:**
- **Purpose:** Large-scale batch processing
- **Use Cases:** ETL jobs, report generation, data backfill
- **Configuration:** Spot instances for cost savings
- **Cost:** ~$50-150/month

### 5.2 Storage Services

**Amazon S3:**
- **Buckets:**
  - `retailiq-data-lake-raw`: Raw ingested data
  - `retailiq-data-lake-processed`: Cleaned data
  - `retailiq-ml-models`: Model artifacts
  - `retailiq-reports`: Generated reports
  - `retailiq-backups`: Database backups
- **Lifecycle Policies:** Move to Glacier after 90 days
- **Cost:** ~$100-300/month (1-5 TB)

**Amazon RDS PostgreSQL:**
- **Purpose:** Primary transactional database
- **Configuration:** db.r5.large, Multi-AZ, 500GB storage
- **Backup:** Automated daily backups, 30-day retention
- **Cost:** ~$400-600/month

**Amazon DynamoDB:**
- **Purpose:** High-velocity operational data
- **Tables:** Sessions, events, metrics, real-time inventory
- **Configuration:** On-demand pricing, global tables
- **Cost:** ~$100-300/month

**Amazon ElastiCache Redis:**
- **Purpose:** Caching layer, session store
- **Configuration:** cache.r5.large, cluster mode enabled
- **Cost:** ~$150-250/month

### 5.3 Networking & Content Delivery

**Amazon CloudFront:**
- **Purpose:** Global CDN for static assets and API acceleration
- **Configuration:** 50+ edge locations, custom SSL certificate
- **Cost:** ~$50-150/month

**AWS API Gateway:**
- **Purpose:** API management and routing
- **Configuration:** REST API, WebSocket API, usage plans
- **Cost:** ~$35-100/month (1M requests)

**Application Load Balancer:**
- **Purpose:** Distribute traffic to ECS services
- **Configuration:** Cross-zone load balancing, health checks
- **Cost:** ~$25-50/month

**Amazon VPC:**
- **Configuration:** 
  - Public subnets (2 AZs): ALB, NAT Gateway
  - Private subnets (2 AZs): ECS, RDS, ElastiCache
  - Isolated subnets (2 AZs): Data processing
- **Cost:** ~$50-100/month (NAT Gateway)

### 5.4 Data & Analytics

**Amazon Kinesis:**
- **Kinesis Data Streams:** Real-time data ingestion
- **Configuration:** 2 shards initially, auto-scaling enabled
- **Cost:** ~$50-150/month

**AWS Glue:**
- **Purpose:** ETL jobs, data catalog
- **Jobs:** Data transformation, feature engineering
- **Cost:** ~$100-300/month

**Amazon Athena:**
- **Purpose:** Ad-hoc SQL queries on S3
- **Use Cases:** Data exploration, reporting
- **Cost:** ~$20-50/month ($5 per TB scanned)

### 5.5 Security & Identity

**AWS Cognito:**
- **Purpose:** User authentication and authorization
- **Configuration:** User pools, identity pools, MFA
- **Cost:** ~$50-150/month (50K MAU free tier)

**AWS KMS:**
- **Purpose:** Encryption key management
- **Keys:** Database encryption, S3 encryption, application secrets
- **Cost:** ~$10-30/month

**AWS Secrets Manager:**
- **Purpose:** Store API keys, database credentials
- **Cost:** ~$5-15/month

**AWS WAF:**
- **Purpose:** Web application firewall
- **Rules:** SQL injection, XSS, rate limiting
- **Cost:** ~$20-50/month

**AWS Shield Standard:**
- **Purpose:** DDoS protection (included free)

### 5.6 Monitoring & Operations

**Amazon CloudWatch:**
- **Logs:** Centralized logging (30-day retention)
- **Metrics:** Custom metrics, dashboards
- **Alarms:** 50+ alarms for critical metrics
- **Cost:** ~$50-150/month

**AWS X-Ray:**
- **Purpose:** Distributed tracing
- **Cost:** ~$10-30/month

**AWS CloudTrail:**
- **Purpose:** Audit logging
- **Cost:** ~$10-20/month

### 5.7 DevOps & Deployment

**AWS CodePipeline:**
- **Pipelines:** Frontend, backend, ML models
- **Cost:** ~$10-30/month

**AWS CodeBuild:**
- **Purpose:** Build and test automation
- **Cost:** ~$20-50/month

**AWS CloudFormation / CDK:**
- **Purpose:** Infrastructure as Code
- **Cost:** Free (pay for resources created)

### 5.8 Messaging & Integration

**Amazon SQS:**
- **Queues:** Job queue, DLQ, notification queue
- **Cost:** ~$10-30/month (within free tier)

**Amazon SNS:**
- **Topics:** Alerts, webhooks, email/SMS
- **Cost:** ~$10-30/month

**Amazon EventBridge:**
- **Purpose:** Event bus for application events
- **Cost:** ~$5-15/month

**AWS Step Functions:**
- **Purpose:** Workflow orchestration
- **Workflows:** ML pipeline, ETL jobs
- **Cost:** ~$25-75/month

---

## 6. Database Design Overview

### 6.1 Database Architecture

**Multi-Database Strategy:**
- **RDS PostgreSQL:** ACID transactions, complex queries, relational data
- **DynamoDB:** High-velocity writes, session data, time-series metrics
- **ElastiCache Redis:** Caching, rate limiting, real-time data
- **S3:** Data lake, archives, ML artifacts

### 6.2 RDS PostgreSQL Schema

**Core Tables:**

```sql
-- Users and Organizations
CREATE TABLE organizations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    subscription_tier VARCHAR(50),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID REFERENCES organizations(id),
    email VARCHAR(255) UNIQUE NOT NULL,
    role VARCHAR(50) NOT NULL,
    cognito_sub VARCHAR(255) UNIQUE,
    created_at TIMESTAMP DEFAULT NOW(),
    last_login TIMESTAMP
);

-- Product Catalog
CREATE TABLE products (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID REFERENCES organizations(id),
    sku VARCHAR(100) NOT NULL,
    name VARCHAR(255) NOT NULL,
    category VARCHAR(100),
    brand VARCHAR(100),
    unit_cost DECIMAL(10,2),
    current_price DECIMAL(10,2),
    attributes JSONB,
    created_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(organization_id, sku)
);

CREATE INDEX idx_products_org_category ON products(organization_id, category);
CREATE INDEX idx_products_sku ON products(sku);

-- Sales Transactions
CREATE TABLE sales_transactions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID REFERENCES organizations(id),
    product_id UUID REFERENCES products(id),
    store_id UUID,
    transaction_date DATE NOT NULL,
    quantity INTEGER NOT NULL,
    unit_price DECIMAL(10,2),
    total_amount DECIMAL(10,2),
    channel VARCHAR(50),
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_sales_org_date ON sales_transactions(organization_id, transaction_date DESC);
CREATE INDEX idx_sales_product_date ON sales_transactions(product_id, transaction_date DESC);

-- Demand Forecasts
CREATE TABLE forecasts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID REFERENCES organizations(id),
    product_id UUID REFERENCES products(id),
    forecast_date DATE NOT NULL,
    forecast_horizon INTEGER,
    predicted_quantity DECIMAL(10,2),
    confidence_lower DECIMAL(10,2),
    confidence_upper DECIMAL(10,2),
    model_version VARCHAR(50),
    accuracy_score DECIMAL(5,4),
    created_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(product_id, forecast_date, forecast_horizon)
);

CREATE INDEX idx_forecasts_product_date ON forecasts(product_id, forecast_date);

-- Pricing Recommendations
CREATE TABLE pricing_recommendations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID REFERENCES organizations(id),
    product_id UUID REFERENCES products(id),
    current_price DECIMAL(10,2),
    recommended_price DECIMAL(10,2),
    expected_demand_change DECIMAL(5,2),
    expected_revenue_impact DECIMAL(10,2),
    confidence_score DECIMAL(5,4),
    reasoning TEXT,
    valid_from TIMESTAMP,
    valid_until TIMESTAMP,
    status VARCHAR(50) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT NOW()
);

-- Inventory
CREATE TABLE inventory (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID REFERENCES organizations(id),
    product_id UUID REFERENCES products(id),
    store_id UUID,
    quantity_on_hand INTEGER,
    reorder_point INTEGER,
    reorder_quantity INTEGER,
    last_updated TIMESTAMP DEFAULT NOW(),
    UNIQUE(product_id, store_id)
);

-- Competitor Pricing
CREATE TABLE competitor_prices (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID REFERENCES organizations(id),
    product_id UUID REFERENCES products(id),
    competitor_name VARCHAR(255),
    competitor_price DECIMAL(10,2),
    source_url TEXT,
    scraped_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_competitor_prices_product ON competitor_prices(product_id, scraped_at DESC);
```

**Partitioning Strategy:**
- `sales_transactions`: Partitioned by month (range partitioning)
- `forecasts`: Partitioned by quarter
- Automatic partition creation via pg_partman extension

**Sharding Strategy (Future):**
- Shard by `organization_id` when reaching 1000+ tenants
- Use Citus extension for distributed PostgreSQL

### 6.3 DynamoDB Tables

**Sessions Table:**
```
Partition Key: session_id (String)
Sort Key: -
Attributes: user_id, expires_at, data (Map)
TTL: expires_at
GSI: user_id-index
```

**Events Table:**
```
Partition Key: organization_id (String)
Sort Key: timestamp (Number)
Attributes: event_type, user_id, data (Map)
TTL: 90 days
GSI: event_type-timestamp-index
```

**Real-time Metrics:**
```
Partition Key: metric_key (String)
Sort Key: timestamp (Number)
Attributes: value, dimensions (Map)
TTL: 7 days
```

### 6.4 Data Retention Policies

| Data Type | Hot Storage | Warm Storage | Cold Storage | Deletion |
|-----------|-------------|--------------|--------------|----------|
| Transactions | 90 days (RDS) | 1 year (S3) | 7 years (Glacier) | Never |
| Forecasts | 180 days (RDS) | 2 years (S3) | - | After 2 years |
| Logs | 30 days (CloudWatch) | 1 year (S3) | - | After 1 year |
| ML Models | Current (SageMaker) | 6 months (S3) | - | After 6 months |
| User Sessions | 24 hours (DynamoDB) | - | - | Auto (TTL) |

---

## 7. API Design Overview

### 7.1 API Architecture

**API Gateway Configuration:**
- Base URL: `https://api.retailiq.com/v1`
- Authentication: JWT tokens (Cognito)
- Rate Limiting: 1000 requests/minute per user
- Versioning: URL-based (/v1, /v2)

### 7.2 Core API Endpoints

**Authentication APIs:**
```
POST   /auth/register          - Register new user
POST   /auth/login             - User login
POST   /auth/logout            - User logout
POST   /auth/refresh           - Refresh access token
POST   /auth/forgot-password   - Password reset request
POST   /auth/mfa/enable        - Enable MFA
```

**User Management APIs:**
```
GET    /users                  - List users (admin)
GET    /users/{id}             - Get user details
PUT    /users/{id}             - Update user
DELETE /users/{id}             - Delete user
POST   /users/{id}/roles       - Assign role
```

**Product APIs:**
```
GET    /products               - List products (paginated)
POST   /products               - Create product
GET    /products/{id}          - Get product details
PUT    /products/{id}          - Update product
DELETE /products/{id}          - Delete product
POST   /products/bulk          - Bulk import products
GET    /products/{id}/history  - Sales history
```

**Forecast APIs:**
```
GET    /forecasts              - List forecasts
POST   /forecasts/generate     - Trigger forecast generation
GET    /forecasts/{id}         - Get forecast details
GET    /forecasts/product/{id} - Get product forecasts
GET    /forecasts/accuracy     - Get accuracy metrics
POST   /forecasts/scenarios    - What-if scenario analysis
```

**Pricing APIs:**
```
GET    /pricing/recommendations        - List pricing recommendations
GET    /pricing/recommendations/{id}   - Get recommendation details
POST   /pricing/recommendations/accept - Accept recommendation
GET    /pricing/competitors            - Competitor pricing data
GET    /pricing/elasticity/{product_id}- Price elasticity analysis
POST   /pricing/optimize               - Optimize pricing strategy
```

**Inventory APIs:**
```
GET    /inventory                      - List inventory
GET    /inventory/{product_id}         - Get product inventory
PUT    /inventory/{product_id}         - Update inventory
GET    /inventory/reorder-points       - Get reorder recommendations
POST   /inventory/optimize             - Optimize inventory levels
GET    /inventory/alerts               - Get stockout alerts
```

**Analytics APIs:**
```
GET    /analytics/dashboard            - Dashboard KPIs
GET    /analytics/sales                - Sales analytics
GET    /analytics/forecast-accuracy    - Forecast accuracy trends
GET    /analytics/inventory-turnover   - Inventory metrics
POST   /analytics/reports              - Generate custom report
GET    /analytics/reports/{id}         - Download report
```

**Integration APIs:**
```
POST   /integrations/data              - Ingest data
GET    /integrations/status            - Integration status
POST   /integrations/webhooks          - Register webhook
DELETE /integrations/webhooks/{id}     - Delete webhook
```

### 7.3 API Request/Response Examples

**Generate Forecast Request:**
```json
POST /v1/forecasts/generate
Authorization: Bearer <jwt_token>
Content-Type: application/json

{
  "product_ids": ["uuid1", "uuid2"],
  "horizon_days": 30,
  "include_confidence_intervals": true,
  "scenario": {
    "promotion": true,
    "discount_percentage": 15
  }
}
```

**Generate Forecast Response:**
```json
{
  "job_id": "job-uuid",
  "status": "processing",
  "estimated_completion": "2026-02-15T10:30:00Z",
  "products_count": 2,
  "message": "Forecast generation started"
}
```

**Get Forecast Response:**
```json
{
  "id": "forecast-uuid",
  "product_id": "product-uuid",
  "product_name": "Product A",
  "forecasts": [
    {
      "date": "2026-02-16",
      "predicted_quantity": 125.5,
      "confidence_lower": 110.2,
      "confidence_upper": 140.8,
      "confidence_score": 0.92
    }
  ],
  "model_version": "v2.3.1",
  "accuracy_mape": 12.5,
  "generated_at": "2026-02-15T10:00:00Z"
}
```

### 7.4 WebSocket API

**Real-time Updates:**
```
wss://api.retailiq.com/v1/ws

Events:
- forecast.completed
- pricing.updated
- inventory.alert
- competitor.price_change
```

### 7.5 API Security

**Authentication Flow:**
1. User authenticates with Cognito
2. Receives JWT access token (1 hour expiry)
3. Includes token in Authorization header
4. API Gateway validates token via Cognito authorizer
5. Request forwarded to backend services

**Rate Limiting:**
- Per-user: 1000 requests/minute
- Per-organization: 10,000 requests/minute
- Burst: 2000 requests
- Throttling: 429 Too Many Requests

**API Keys (for integrations):**
- Long-lived API keys for POS/e-commerce integrations
- Stored in Secrets Manager
- Rotated every 90 days

---

## 8. Scalability Strategy

### 8.1 Horizontal Scaling

**Application Layer:**
- ECS services auto-scale based on CPU/memory (target: 70%)
- Lambda concurrent executions: 1000 (can request increase)
- API Gateway: Unlimited scaling (managed by AWS)
- Scale-out: Add more ECS tasks/Lambda invocations

**Database Layer:**
- RDS: Read replicas (up to 5) for read-heavy workloads
- DynamoDB: On-demand scaling (auto-scales to demand)
- ElastiCache: Cluster mode with 3-6 shards
- Future: Implement database sharding by organization_id

**ML Layer:**
- SageMaker endpoints: Auto-scaling (1-10 instances)
- Batch inference for bulk forecasts
- Model serving: Multi-model endpoints for efficiency

### 8.2 Vertical Scaling

**Scaling Triggers:**
- RDS: Upgrade instance class when CPU > 80% for 1 hour
- ElastiCache: Upgrade when memory > 85%
- ECS: Increase task CPU/memory allocation

**Instance Sizing Strategy:**
```
Startup (0-100 users):
- RDS: db.t3.large
- ElastiCache: cache.t3.medium
- ECS: 2 tasks × 1 vCPU, 2GB RAM

Growth (100-1000 users):
- RDS: db.r5.xlarge + 2 read replicas
- ElastiCache: cache.r5.large (cluster mode)
- ECS: 5-10 tasks × 2 vCPU, 4GB RAM

Scale (1000-10000 users):
- RDS: db.r5.2xlarge + 5 read replicas
- ElastiCache: cache.r5.xlarge (6 shards)
- ECS: 10-20 tasks × 4 vCPU, 8GB RAM
```

### 8.3 Caching Strategy

**Multi-Layer Caching:**

```
Layer 1: Browser Cache (CloudFront)
- Static assets: 1 year
- API responses: 5 minutes (with ETag)

Layer 2: API Gateway Cache
- Frequently accessed endpoints: 5 minutes
- User-specific data: Disabled

Layer 3: Application Cache (ElastiCache)
- Dashboard data: 5 minutes
- Forecast results: 1 hour
- Product catalog: 30 minutes
- User sessions: 24 hours

Layer 4: Database Query Cache
- PostgreSQL query cache
- Materialized views for complex aggregations
```

**Cache Invalidation:**
- Event-driven: Invalidate on data updates (SNS → Lambda → Redis)
- Time-based: TTL expiration
- Manual: Admin API for cache clearing

### 8.4 Database Optimization

**Query Optimization:**
- Indexes on frequently queried columns
- Composite indexes for multi-column queries
- Partial indexes for filtered queries
- Query plan analysis and optimization

**Connection Pooling:**
- RDS Proxy for connection management
- Pool size: 100 connections per service
- Connection reuse and multiplexing

**Read/Write Splitting:**
- Write operations → Primary RDS instance
- Read operations → Read replicas (round-robin)
- Application-level routing via connection strings

**Partitioning:**
- Time-based partitioning for sales_transactions
- Automatic partition management
- Partition pruning for query optimization

### 8.5 Asynchronous Processing

**Job Queue Architecture:**
```
Synchronous (< 5 seconds):
- User queries → API Gateway → Lambda/ECS → Response

Asynchronous (5 seconds - 5 minutes):
- Heavy operations → SQS → Lambda/ECS Worker → SNS notification

Batch (> 5 minutes):
- Large-scale processing → Step Functions → AWS Batch → S3 results
```

**Queue Configuration:**
- Standard queue for non-critical jobs
- FIFO queue for ordered processing
- Dead Letter Queue (DLQ) for failed jobs
- Visibility timeout: 5 minutes
- Message retention: 14 days

### 8.6 Content Delivery

**CloudFront Configuration:**
- 50+ edge locations globally
- Origin: S3 (static) + ALB (dynamic)
- Cache behaviors: /api/* (no cache), /assets/* (1 year)
- Compression: Gzip/Brotli enabled
- HTTP/2 and HTTP/3 support

### 8.7 Load Testing Strategy

**Performance Targets:**
- 10,000 concurrent users
- 100,000 requests per minute
- < 2 second response time (95th percentile)

**Load Testing Tools:**
- AWS Distributed Load Testing solution
- Apache JMeter for API testing
- Locust for user behavior simulation

**Testing Scenarios:**
1. Baseline: Normal traffic patterns
2. Peak: 3x normal traffic (holiday season)
3. Spike: 10x sudden traffic increase
4. Soak: Sustained load for 24 hours

---

## 9. Security Architecture

### 9.1 Defense in Depth

**Security Layers:**

```
Layer 1: Edge Security
├─ CloudFront: DDoS protection (Shield)
├─ WAF: SQL injection, XSS, rate limiting
└─ SSL/TLS: Certificate management (ACM)

Layer 2: Network Security
├─ VPC: Network isolation
├─ Security Groups: Stateful firewall
├─ NACLs: Stateless firewall
└─ PrivateLink: Secure service access

Layer 3: Application Security
├─ API Gateway: Request validation, throttling
├─ Cognito: Authentication, MFA
├─ IAM: Authorization, least privilege
└─ Secrets Manager: Credential management

Layer 4: Data Security
├─ KMS: Encryption key management
├─ S3: Encryption at rest (SSE-KMS)
├─ RDS: Encryption at rest (KMS)
└─ TLS: Encryption in transit

Layer 5: Monitoring & Response
├─ GuardDuty: Threat detection
├─ CloudTrail: Audit logging
├─ Config: Compliance monitoring
└─ Security Hub: Centralized security view
```

### 9.2 Identity & Access Management

**IAM Strategy:**
- Principle of least privilege
- Role-based access control (RBAC)
- Service roles for AWS services
- Cross-account roles for partners

**User Roles:**
```
Admin:
- Full system access
- User management
- Configuration changes

Manager:
- View all data
- Generate forecasts
- Accept pricing recommendations
- Export reports

Analyst:
- View data
- Generate forecasts
- View recommendations (no accept)

Viewer:
- Read-only access
- Dashboard viewing
- Report viewing
```

**Service-to-Service Authentication:**
- IAM roles for ECS tasks
- Lambda execution roles
- SageMaker execution roles
- Cross-service trust policies

### 9.3 Data Encryption

**Encryption at Rest:**
- S3: SSE-KMS with customer-managed keys
- RDS: Transparent Data Encryption (TDE) with KMS
- DynamoDB: Encryption enabled by default
- EBS volumes: Encrypted with KMS
- Backups: Encrypted with separate keys

**Encryption in Transit:**
- TLS 1.3 for all API communications
- VPC endpoints for AWS service communication
- Certificate pinning for mobile apps
- mTLS for service-to-service communication

**Key Management:**
- AWS KMS for key storage and rotation
- Automatic key rotation (annual)
- Separate keys per environment (dev/staging/prod)
- Customer-managed keys (BYOK) for enterprise tier

### 9.4 Network Security

**VPC Architecture:**
```
VPC: 10.0.0.0/16

Public Subnets (10.0.1.0/24, 10.0.2.0/24):
- ALB
- NAT Gateway
- Bastion host (if needed)

Private Subnets (10.0.10.0/24, 10.0.11.0/24):
- ECS tasks
- Lambda functions (VPC-attached)
- Application servers

Data Subnets (10.0.20.0/24, 10.0.21.0/24):
- RDS instances
- ElastiCache clusters
- No internet access
```

**Security Groups:**
```
ALB Security Group:
- Inbound: 443 from 0.0.0.0/0
- Outbound: 8080 to ECS security group

ECS Security Group:
- Inbound: 8080 from ALB security group
- Outbound: 5432 to RDS security group, 6379 to Redis

RDS Security Group:
- Inbound: 5432 from ECS security group
- Outbound: None

Redis Security Group:
- Inbound: 6379 from ECS security group
- Outbound: None
```

**Network Access Control:**
- NACLs for subnet-level filtering
- VPC Flow Logs for traffic analysis
- AWS PrivateLink for secure AWS service access
- No direct internet access for data layer

### 9.5 Application Security

**OWASP Top 10 Mitigations:**

1. **Injection:** Parameterized queries, input validation
2. **Broken Authentication:** Cognito, MFA, session management
3. **Sensitive Data Exposure:** Encryption, data masking
4. **XML External Entities:** Disable XML parsing
5. **Broken Access Control:** RBAC, API authorization
6. **Security Misconfiguration:** Automated security scanning
7. **XSS:** Content Security Policy, input sanitization
8. **Insecure Deserialization:** Validate serialized data
9. **Using Components with Known Vulnerabilities:** Dependency scanning
10. **Insufficient Logging:** Comprehensive audit logs

**Security Headers:**
```
Strict-Transport-Security: max-age=31536000; includeSubDomains
Content-Security-Policy: default-src 'self'
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Referrer-Policy: strict-origin-when-cross-origin
```

**Input Validation:**
- API Gateway request validation
- JSON schema validation
- SQL injection prevention (parameterized queries)
- XSS prevention (output encoding)
- File upload validation (type, size, content)

### 9.6 Compliance & Auditing

**Compliance Frameworks:**
- SOC 2 Type II (target: 12 months)
- GDPR (EU data protection)
- CCPA (California privacy)
- PCI DSS (if handling payments)

**Audit Logging:**
- CloudTrail: All API calls logged
- Application logs: User actions, data access
- Database audit logs: Query logging
- Retention: 7 years for compliance

**Data Privacy:**
- Data minimization: Collect only necessary data
- Purpose limitation: Use data only for stated purposes
- Data portability: Export user data on request
- Right to deletion: Delete user data on request
- Consent management: Explicit opt-in for data collection

### 9.7 Incident Response

**Incident Response Plan:**

```
1. Detection
   - Automated alerts (GuardDuty, CloudWatch)
   - User reports
   - Security scanning

2. Containment
   - Isolate affected resources
   - Revoke compromised credentials
   - Block malicious IPs (WAF)

3. Investigation
   - Analyze logs (CloudTrail, application logs)
   - Identify root cause
   - Assess impact

4. Remediation
   - Patch vulnerabilities
   - Restore from backups if needed
   - Update security controls

5. Recovery
   - Restore normal operations
   - Monitor for recurrence
   - Validate security posture

6. Post-Incident
   - Document incident
   - Update runbooks
   - Conduct lessons learned
```

**Security Contacts:**
- On-call rotation for security incidents
- Escalation path defined
- External security firm on retainer

---

## 10. Deployment Strategy

### 10.1 Environment Architecture

**Environments:**

```
Development (dev):
- Purpose: Active development and testing
- Infrastructure: Minimal (single AZ, smaller instances)
- Data: Synthetic/anonymized data
- Cost: ~$500/month

Staging (staging):
- Purpose: Pre-production testing, QA
- Infrastructure: Production-like (multi-AZ)
- Data: Anonymized production data
- Cost: ~$1,500/month

Production (prod):
- Purpose: Live customer environment
- Infrastructure: Full redundancy (multi-AZ, multi-region future)
- Data: Real customer data
- Cost: ~$3,000-5,000/month (startup scale)
```

**Environment Isolation:**
- Separate AWS accounts per environment
- AWS Organizations for centralized management
- Cross-account roles for deployment
- Separate VPCs, no network connectivity between environments

### 10.2 CI/CD Pipeline

**Pipeline Architecture:**

```
┌─────────────┐
│   GitHub    │
│  Repository │
└──────┬──────┘
       │
       │ (git push)
       ▼
┌─────────────────────────────────────────────────────────┐
│              AWS CodePipeline                            │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐           │
│  │  Source  │──▶│  Build   │──▶│   Test   │           │
│  │ (GitHub) │   │(CodeBuild│   │(CodeBuild│           │
│  └──────────┘   └──────────┘   └──────────┘           │
│                                                          │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐           │
│  │ Security │──▶│  Deploy  │──▶│  Verify  │           │
│  │  Scan    │   │  (Dev)   │   │          │           │
│  └──────────┘   └──────────┘   └──────────┘           │
│                                                          │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐           │
│  │ Manual   │──▶│  Deploy  │──▶│  Smoke   │           │
│  │ Approval │   │(Staging) │   │  Tests   │           │
│  └──────────┘   └──────────┘   └──────────┘           │
│                                                          │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐           │
│  │ Manual   │──▶│  Deploy  │──▶│ Monitor  │           │
│  │ Approval │   │  (Prod)  │   │          │           │
│  └──────────┘   └──────────┘   └──────────┘           │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

**Build Stage (CodeBuild):**
```yaml
# buildspec.yml
version: 0.2
phases:
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - aws ecr get-login-password | docker login --username AWS --password-stdin $ECR_REGISTRY
      - COMMIT_HASH=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)
      - IMAGE_TAG=${COMMIT_HASH:=latest}
  build:
    commands:
      - echo Build started on `date`
      - docker build -t $ECR_REPOSITORY:$IMAGE_TAG .
      - docker tag $ECR_REPOSITORY:$IMAGE_TAG $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
  post_build:
    commands:
      - echo Pushing Docker image...
      - docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
      - echo Writing image definitions file...
      - printf '[{"name":"app","imageUri":"%s"}]' $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG > imagedefinitions.json
artifacts:
  files: imagedefinitions.json
```

**Test Stage:**
- Unit tests (Jest, pytest)
- Integration tests
- API contract tests
- Code coverage (target: 80%)
- Linting and code quality (SonarQube)

**Security Scan Stage:**
- SAST: Static code analysis (Snyk, SonarQube)
- DAST: Dynamic application security testing
- Dependency scanning (npm audit, pip-audit)
- Container image scanning (ECR image scanning)
- Infrastructure scanning (Checkov for IaC)

**Deployment Stage:**
- Blue/green deployment for zero downtime
- Automated rollback on failure
- Database migrations (Flyway/Liquibase)
- Configuration updates (Parameter Store)

### 10.3 Deployment Strategies

**Blue/Green Deployment (ECS):**
```
1. Deploy new version (Green) alongside current (Blue)
2. Run smoke tests on Green environment
3. Shift 10% traffic to Green (canary)
4. Monitor metrics for 10 minutes
5. Shift 50% traffic to Green
6. Monitor metrics for 10 minutes
7. Shift 100% traffic to Green
8. Keep Blue running for 1 hour (quick rollback)
9. Terminate Blue environment
```

**Lambda Deployment:**
- Alias-based deployment (prod alias)
- Weighted traffic shifting (10% → 50% → 100%)
- Automatic rollback on CloudWatch alarms

**Database Migrations:**
- Backward-compatible migrations only
- Run migrations before code deployment
- Separate rollback scripts
- Test migrations in staging first

### 10.4 Infrastructure as Code

**AWS CDK Structure:**
```
infrastructure/
├── bin/
│   └── app.ts                 # CDK app entry point
├── lib/
│   ├── stacks/
│   │   ├── network-stack.ts   # VPC, subnets, security groups
│   │   ├── database-stack.ts  # RDS, DynamoDB, ElastiCache
│   │   ├── compute-stack.ts   # ECS, Lambda, API Gateway
│   │   ├── ml-stack.ts        # SageMaker resources
│   │   ├── storage-stack.ts   # S3 buckets
│   │   └── monitoring-stack.ts# CloudWatch, X-Ray
│   └── constructs/
│       ├── ecs-service.ts     # Reusable ECS service
│       ├── lambda-function.ts # Reusable Lambda
│       └── api-endpoint.ts    # Reusable API Gateway
├── config/
│   ├── dev.ts                 # Dev environment config
│   ├── staging.ts             # Staging environment config
│   └── prod.ts                # Prod environment config
└── package.json
```

**Deployment Commands:**
```bash
# Deploy to dev
cdk deploy --all --context env=dev

# Deploy to staging
cdk deploy --all --context env=staging

# Deploy to production (requires approval)
cdk deploy --all --context env=prod --require-approval always
```

### 10.5 Configuration Management

**AWS Systems Manager Parameter Store:**
```
/retailiq/dev/database/host
/retailiq/dev/database/port
/retailiq/dev/api/rate-limit
/retailiq/dev/ml/model-version
/retailiq/dev/features/pricing-enabled
```

**AWS Secrets Manager:**
```
/retailiq/dev/database/credentials
/retailiq/dev/api-keys/stripe
/retailiq/dev/api-keys/sendgrid
```

**Environment Variables (ECS Task Definition):**
```json
{
  "environment": [
    {"name": "ENV", "value": "production"},
    {"name": "LOG_LEVEL", "value": "info"},
    {"name": "DB_HOST", "valueFrom": "arn:aws:ssm:...:/database/host"}
  ],
  "secrets": [
    {"name": "DB_PASSWORD", "valueFrom": "arn:aws:secretsmanager:..."}
  ]
}
```

### 10.6 Rollback Strategy

**Automated Rollback Triggers:**
- Error rate > 5% for 5 minutes
- Response time > 5 seconds (95th percentile)
- Health check failures > 50%
- Custom CloudWatch alarms

**Manual Rollback:**
```bash
# ECS: Update service to previous task definition
aws ecs update-service --service api-service --task-definition api:42

# Lambda: Update alias to previous version
aws lambda update-alias --function-name forecast --name prod --function-version 42

# Database: Run rollback migration
flyway migrate -target=42
```

**Rollback Testing:**
- Quarterly rollback drills
- Automated rollback in staging
- Document rollback procedures

### 10.7 Feature Flags

**Feature Flag Service (AWS AppConfig):**
```json
{
  "features": {
    "pricing_intelligence": {
      "enabled": true,
      "rollout_percentage": 100
    },
    "new_forecast_model": {
      "enabled": true,
      "rollout_percentage": 10,
      "whitelist": ["org-uuid-1", "org-uuid-2"]
    },
    "competitor_scraping": {
      "enabled": false
    }
  }
}
```

**Usage in Code:**
```python
from aws_appconfig import get_feature_flag

if get_feature_flag('new_forecast_model', organization_id):
    forecast = new_model.predict(data)
else:
    forecast = legacy_model.predict(data)
```

---

## 11. Monitoring & Logging

### 11.1 Observability Architecture

**Three Pillars of Observability:**

```
┌─────────────────────────────────────────────────────────┐
│                    METRICS                               │
│  CloudWatch Metrics, Custom Metrics, Business KPIs      │
└─────────────────────────────────────────────────────────┘
                          │
┌─────────────────────────────────────────────────────────┐
│                     LOGS                                 │
│  CloudWatch Logs, Application Logs, Audit Logs          │
└─────────────────────────────────────────────────────────┘
                          │
┌─────────────────────────────────────────────────────────┐
│                    TRACES                                │
│  X-Ray, Distributed Tracing, Service Map                │
└─────────────────────────────────────────────────────────┘
```

### 11.2 Metrics & Monitoring

**Infrastructure Metrics:**
- ECS: CPU, memory, task count, deployment status
- Lambda: Invocations, duration, errors, throttles
- RDS: CPU, connections, IOPS, replication lag
- ElastiCache: CPU, memory, cache hit rate, evictions
- API Gateway: Request count, latency, 4xx/5xx errors

**Application Metrics:**
- Request rate (requests/minute)
- Response time (p50, p95, p99)
- Error rate (%)
- Availability (%)
- Active users (concurrent)

**Business Metrics:**
- Forecasts generated (per day)
- Forecast accuracy (MAPE)
- Pricing recommendations accepted (%)
- Data ingestion volume (records/day)
- API usage per customer

**Custom Metrics (CloudWatch):**
```python
import boto3

cloudwatch = boto3.client('cloudwatch')

# Publish custom metric
cloudwatch.put_metric_data(
    Namespace='RetailIQ/Application',
    MetricData=[
        {
            'MetricName': 'ForecastAccuracy',
            'Value': 87.5,
            'Unit': 'Percent',
            'Dimensions': [
                {'Name': 'OrganizationId', 'Value': 'org-123'},
                {'Name': 'ProductCategory', 'Value': 'Electronics'}
            ]
        }
    ]
)
```

### 11.3 Logging Strategy

**Log Levels:**
- ERROR: Application errors, exceptions
- WARN: Potential issues, degraded performance
- INFO: Important business events, API calls
- DEBUG: Detailed diagnostic information (dev/staging only)

**Structured Logging (JSON):**
```json
{
  "timestamp": "2026-02-15T10:30:00Z",
  "level": "INFO",
  "service": "forecast-service",
  "trace_id": "abc123",
  "user_id": "user-uuid",
  "organization_id": "org-uuid",
  "message": "Forecast generated successfully",
  "duration_ms": 4523,
  "product_count": 150,
  "model_version": "v2.3.1"
}
```

**Log Aggregation:**
- All logs sent to CloudWatch Logs
- Log groups per service
- Retention: 30 days (hot), 1 year (S3 archive)
- Log insights for querying and analysis

**Log Queries (CloudWatch Insights):**
```
# Find errors in last hour
fields @timestamp, @message
| filter level = "ERROR"
| sort @timestamp desc
| limit 100

# Calculate average response time
fields @timestamp, duration_ms
| stats avg(duration_ms) by bin(5m)

# Find slow API calls
fields @timestamp, endpoint, duration_ms
| filter duration_ms > 2000
| sort duration_ms desc
```

### 11.4 Distributed Tracing

**AWS X-Ray Integration:**
- Trace requests across services
- Identify bottlenecks and latency issues
- Service map visualization
- Error analysis

**Trace Example:**
```
API Gateway (50ms)
  └─ Lambda Authorizer (20ms)
  └─ ALB (10ms)
      └─ ECS Service (200ms)
          ├─ ElastiCache (5ms) [cache miss]
          └─ RDS Query (180ms)
              └─ Complex JOIN query
```

**X-Ray Annotations:**
```python
from aws_xray_sdk.core import xray_recorder

@xray_recorder.capture('generate_forecast')
def generate_forecast(product_id):
    xray_recorder.put_annotation('product_id', product_id)
    xray_recorder.put_metadata('model_version', 'v2.3.1')
    
    # Business logic
    forecast = model.predict(data)
    
    xray_recorder.put_annotation('forecast_accuracy', forecast.accuracy)
    return forecast
```

### 11.5 Alerting & Notifications

**Alert Categories:**

**Critical (PagerDuty):**
- Service down (health check failures)
- Database connection failures
- Error rate > 10%
- Response time > 10 seconds

**High (Slack + Email):**
- Error rate > 5%
- Response time > 5 seconds
- Forecast accuracy < 70%
- Disk usage > 85%

**Medium (Email):**
- Unusual traffic patterns
- Forecast accuracy degradation
- High cache miss rate

**Low (Dashboard only):**
- Minor performance degradation
- Non-critical warnings

**CloudWatch Alarms:**
```python
# Example: High error rate alarm
alarm = cloudwatch.Alarm(
    self, "HighErrorRate",
    metric=api_errors_metric,
    threshold=5,
    evaluation_periods=2,
    datapoints_to_alarm=2,
    comparison_operator=cloudwatch.ComparisonOperator.GREATER_THAN_THRESHOLD,
    treat_missing_data=cloudwatch.TreatMissingData.NOT_BREACHING,
    actions_enabled=True
)
alarm.add_alarm_action(sns_action)
```

### 11.6 Dashboards

**CloudWatch Dashboards:**

**Executive Dashboard:**
- System health (green/yellow/red)
- Active users (real-time)
- API request rate
- Error rate
- Forecast accuracy trend

**Operations Dashboard:**
- Infrastructure metrics (CPU, memory, disk)
- Service health checks
- Deployment status
- Recent alarms

**Business Dashboard:**
- Forecasts generated (daily)
- Pricing recommendations (daily)
- Customer usage statistics
- Revenue metrics

**ML Dashboard:**
- Model accuracy trends
- Training job status
- Inference latency
- Feature drift detection

### 11.7 Audit Logging

**CloudTrail Configuration:**
- All API calls logged
- Multi-region trail
- Log file validation enabled
- S3 bucket with MFA delete
- SNS notification on log delivery

**Application Audit Logs:**
```json
{
  "event_type": "pricing_recommendation_accepted",
  "timestamp": "2026-02-15T10:30:00Z",
  "user_id": "user-uuid",
  "organization_id": "org-uuid",
  "resource_id": "recommendation-uuid",
  "action": "accept",
  "ip_address": "203.0.113.42",
  "user_agent": "Mozilla/5.0...",
  "changes": {
    "old_price": 99.99,
    "new_price": 89.99
  }
}
```

---

## 12. Cost Optimization Considerations

### 12.1 Cost Breakdown (Startup Scale: 100-500 users)

**Monthly Cost Estimate:**

| Service Category | Service | Configuration | Monthly Cost |
|------------------|---------|---------------|--------------|
| **Compute** | ECS Fargate | 5 tasks × 2 vCPU, 4GB | $150 |
| | Lambda | 10M invocations, 1GB | $50 |
| | SageMaker Training | 20 hours/month ml.m5.xlarge | $100 |
| | SageMaker Inference | 1 endpoint ml.t2.medium | $50 |
| **Storage** | S3 | 2TB storage, 1M requests | $50 |
| | RDS PostgreSQL | db.r5.large, Multi-AZ | $450 |
| | DynamoDB | 10GB, on-demand | $50 |
| | ElastiCache Redis | cache.r5.large | $150 |
| **Networking** | CloudFront | 1TB transfer | $85 |
| | API Gateway | 10M requests | $35 |
| | ALB | 1 ALB, 1GB/hour | $25 |
| | Data Transfer | 500GB out | $45 |
| **Data & Analytics** | Kinesis | 2 shards | $50 |
| | Glue | 50 DPU-hours | $50 |
| **Security** | Cognito | 10K MAU | $50 |
| | WAF | 1 web ACL, 10 rules | $20 |
| | KMS | 10 keys, 100K requests | $15 |
| **Monitoring** | CloudWatch | Logs, metrics, alarms | $75 |
| | X-Ray | 1M traces | $10 |
| **Other** | Secrets Manager | 20 secrets | $10 |
| | SNS/SQS | 10M messages | $10 |
| **Total** | | | **~$1,530/month** |

**Cost Scaling:**
- 100 users: ~$1,500/month ($15/user)
- 500 users: ~$3,000/month ($6/user)
- 1,000 users: ~$5,000/month ($5/user)
- 5,000 users: ~$15,000/month ($3/user)

### 12.2 Cost Optimization Strategies

**1. Compute Optimization**

**Right-Sizing:**
- Start with smaller instances, scale up based on metrics
- Use AWS Compute Optimizer recommendations
- Review instance utilization monthly

**Spot Instances:**
- Use Spot for batch processing (70% cost savings)
- AWS Batch with Spot fleet
- ML training on Spot instances (SageMaker)

**Savings Plans:**
- Compute Savings Plans (1-year): 17% savings
- Compute Savings Plans (3-year): 54% savings
- Apply after stable baseline established (6 months)

**Lambda Optimization:**
- Right-size memory allocation (affects CPU)
- Use provisioned concurrency only when needed
- Optimize cold start times

**2. Storage Optimization**

**S3 Intelligent-Tiering:**
- Automatically moves data to cost-effective tiers
- Frequent access: Standard
- Infrequent access (30 days): IA (40% savings)
- Archive (90 days): Glacier (80% savings)

**S3 Lifecycle Policies:**
```json
{
  "Rules": [
    {
      "Id": "ArchiveOldData",
      "Status": "Enabled",
      "Transitions": [
        {
          "Days": 90,
          "StorageClass": "STANDARD_IA"
        },
        {
          "Days": 365,
          "StorageClass": "GLACIER"
        }
      ]
    }
  ]
}
```

**Database Optimization:**
- Use read replicas only when needed
- Schedule non-production databases (stop nights/weekends)
- Use Aurora Serverless for dev/staging (pay per second)
- Optimize queries to reduce IOPS

**3. Data Transfer Optimization**

**Reduce Data Transfer Costs:**
- Use CloudFront for static content (cheaper than S3 direct)
- Keep data in same region when possible
- Compress responses (gzip/brotli)
- Use VPC endpoints for AWS service communication (free)

**4. Monitoring & Logging Optimization**

**CloudWatch Logs:**
- Set appropriate retention periods (30 days vs. 1 year)
- Use log sampling for high-volume logs
- Archive old logs to S3 (90% cheaper)

**Metrics:**
- Use metric filters instead of custom metrics when possible
- Aggregate metrics before publishing
- Use standard resolution (1 minute) vs. high resolution

**5. ML Cost Optimization**

**Training:**
- Use Spot instances (70% savings)
- Optimize hyperparameter tuning (fewer trials)
- Use incremental training when possible
- Schedule training during off-peak hours

**Inference:**
- Use multi-model endpoints (share infrastructure)
- Auto-scale endpoints based on traffic
- Use batch inference for non-real-time predictions
- Consider serverless inference for low traffic

**6. Reserved Capacity**

**When to Use Reserved Instances:**
- After 6 months of stable usage
- For baseline capacity (not peak)
- 1-year term initially (flexibility)

**Reserved Capacity Candidates:**
- RDS instances (40-60% savings)
- ElastiCache instances (30-50% savings)
- ECS/EC2 baseline capacity (30-50% savings)

### 12.3 Cost Monitoring & Governance

**AWS Cost Explorer:**
- Daily cost monitoring
- Cost allocation tags (environment, service, team)
- Forecast future costs
- Identify cost anomalies

**AWS Budgets:**
```
Budget 1: Monthly Total
- Amount: $2,000
- Alert at 80% ($1,600)
- Alert at 100% ($2,000)

Budget 2: Per-Service
- RDS: $500
- SageMaker: $200
- S3: $100

Budget 3: Per-Environment
- Production: $1,500
- Staging: $300
- Development: $200
```

**Cost Allocation Tags:**
```
Environment: production | staging | development
Service: api | ml | data | frontend
Team: engineering | data-science
CostCenter: product | infrastructure
```

**FinOps Practices:**
- Weekly cost review meetings
- Monthly cost optimization sprints
- Quarterly reserved capacity planning
- Annual cost optimization goals (10% reduction)

### 12.4 Cost Optimization Roadmap

**Phase 1: Foundation (Months 1-3)**
- Implement cost allocation tags
- Set up AWS Budgets and alerts
- Right-size initial resources
- Enable S3 Intelligent-Tiering

**Phase 2: Optimization (Months 4-6)**
- Analyze usage patterns
- Implement auto-scaling policies
- Use Spot instances for batch jobs
- Optimize database queries

**Phase 3: Commitment (Months 7-12)**
- Purchase Savings Plans (1-year)
- Reserved capacity for stable workloads
- Implement advanced caching strategies
- Optimize ML training schedules

**Phase 4: Continuous Improvement (Ongoing)**
- Monthly cost reviews
- Quarterly optimization initiatives
- Annual reserved capacity renewal
- Continuous right-sizing

### 12.5 Cost Per Customer Metrics

**Target Unit Economics:**
- Customer Acquisition Cost (CAC): $500
- Monthly Revenue Per User (ARPU): $200
- Monthly Infrastructure Cost Per User: $5
- Gross Margin: 97.5%
- LTV:CAC Ratio: 10:1 (target)

**Cost Efficiency KPIs:**
- Infrastructure cost as % of revenue: < 5%
- Cost per API request: < $0.0001
- Cost per forecast generated: < $0.01
- Cost per GB stored: < $0.02
- Cost per ML inference: < $0.001

---

## 13. Disaster Recovery & Business Continuity

### 13.1 Backup Strategy

**RDS Automated Backups:**
- Daily automated snapshots
- 30-day retention period
- Cross-region backup replication
- Point-in-time recovery (5-minute granularity)

**S3 Versioning & Replication:**
- Versioning enabled on all buckets
- Cross-region replication to secondary region
- MFA delete for critical buckets
- Lifecycle policies for old versions

**DynamoDB Backups:**
- Point-in-time recovery enabled
- Daily automated backups
- 35-day retention
- On-demand backups before major changes

**Application Configuration:**
- Parameter Store values backed up to S3
- Secrets Manager automatic rotation
- Infrastructure as Code in Git (version controlled)

### 13.2 Recovery Objectives

**RTO/RPO Targets:**

| System Component | RPO | RTO | Strategy |
|------------------|-----|-----|----------|
| Database (RDS) | 5 minutes | 1 hour | Automated snapshots, Multi-AZ |
| Application (ECS) | 0 (stateless) | 15 minutes | Auto-scaling, health checks |
| ML Models | 1 day | 2 hours | S3 versioning, model registry |
| Data Lake (S3) | 0 | 1 hour | Cross-region replication |
| Configuration | 1 hour | 30 minutes | Parameter Store, IaC |

### 13.3 Disaster Recovery Plan

**DR Scenarios:**

**Scenario 1: Single AZ Failure**
- Impact: Minimal (Multi-AZ architecture)
- Response: Automatic failover to secondary AZ
- RTO: < 5 minutes
- Action: Monitor and investigate root cause

**Scenario 2: Regional Service Outage**
- Impact: Partial service degradation
- Response: Failover to secondary region (manual)
- RTO: 2-4 hours
- Action: Activate DR runbook, notify customers

**Scenario 3: Complete Region Failure**
- Impact: Full service outage
- Response: Failover to DR region
- RTO: 4-8 hours
- Action: Execute full DR plan, restore from backups

**Scenario 4: Data Corruption**
- Impact: Data integrity issues
- Response: Restore from point-in-time backup
- RTO: 2-4 hours
- Action: Identify corruption point, restore, validate

**DR Runbook:**
```
1. Incident Detection
   - Automated alerts trigger
   - On-call engineer notified
   - Assess impact and severity

2. Incident Declaration
   - Declare disaster if RTO/RPO at risk
   - Notify stakeholders
   - Activate DR team

3. Failover Execution
   - Update Route53 to point to DR region
   - Restore database from latest backup
   - Deploy application to DR region
   - Validate functionality

4. Communication
   - Update status page
   - Notify customers via email
   - Post updates every 30 minutes

5. Recovery Validation
   - Run smoke tests
   - Verify data integrity
   - Monitor error rates

6. Post-Incident
   - Document timeline
   - Conduct post-mortem
   - Update DR procedures
```

### 13.4 Testing & Validation

**DR Testing Schedule:**
- Quarterly: Backup restoration test
- Semi-annually: Partial failover test
- Annually: Full DR drill

**Test Checklist:**
- [ ] Restore RDS from backup
- [ ] Deploy application to DR region
- [ ] Validate data integrity
- [ ] Test API functionality
- [ ] Verify ML model availability
- [ ] Check monitoring and alerting
- [ ] Measure actual RTO/RPO
- [ ] Document lessons learned

---

## 14. Future Enhancements

### 14.1 Multi-Region Architecture (Phase 2)

**Global Deployment:**
```
Primary Region: us-east-1 (N. Virginia)
Secondary Region: eu-west-1 (Ireland)
Tertiary Region: ap-southeast-1 (Singapore)

Benefits:
- Lower latency for global customers
- Improved disaster recovery
- Compliance with data residency requirements
```

**Multi-Region Components:**
- Route53 latency-based routing
- DynamoDB Global Tables
- S3 Cross-Region Replication
- Aurora Global Database
- CloudFront with multiple origins

### 14.2 Advanced ML Capabilities

**AutoML:**
- Automated model selection
- Hyperparameter optimization
- Feature engineering automation
- SageMaker Autopilot integration

**Real-Time Forecasting:**
- Streaming ML with Kinesis Data Analytics
- Online learning and model updates
- Sub-second inference latency

**Explainable AI:**
- SHAP values for feature importance
- Counterfactual explanations
- Model interpretability dashboard

### 14.3 Enhanced Analytics

**Real-Time Analytics:**
- Kinesis Data Analytics for streaming
- Real-time dashboards with WebSocket updates
- Anomaly detection in real-time

**Advanced Visualizations:**
- Interactive charts with drill-down
- Geospatial analysis
- Network graphs for product relationships

**Embedded Analytics:**
- QuickSight embedded dashboards
- White-label analytics for customers
- Custom report builder

### 14.4 Platform Expansion

**Marketplace:**
- Third-party integrations
- Plugin architecture
- API marketplace

**Mobile Apps:**
- Native iOS/Android apps
- Offline mode support
- Push notifications

**Voice Interface:**
- Alexa skill for voice queries
- Natural language query interface
- Voice-activated alerts

---

## 15. Conclusion

The RetailIQ platform architecture is designed to deliver enterprise-grade demand forecasting and pricing intelligence to small and medium retailers through a scalable, secure, and cost-effective cloud-native solution. Key architectural highlights include:

**Scalability:** Horizontal scaling supporting 100,000+ users with auto-scaling across all layers

**Reliability:** 99.9% uptime SLA with Multi-AZ deployment, automated failover, and comprehensive disaster recovery

**Performance:** Sub-2-second dashboard loads, <5-minute forecast generation, and <500ms API response times

**Security:** Defense-in-depth approach with encryption, IAM, network isolation, and SOC 2 compliance readiness

**Cost Efficiency:** Serverless-first architecture with intelligent resource optimization, targeting <$5 infrastructure cost per user

**AI/ML Excellence:** Production-grade ML pipeline with ensemble models, automated retraining, and continuous monitoring

**Operational Excellence:** Comprehensive observability, automated deployments, and infrastructure as code

This architecture provides a solid foundation for rapid growth while maintaining the flexibility to adapt to changing business requirements and scale globally. The modular design enables independent evolution of components, and the cloud-native approach ensures operational efficiency and cost optimization.

---

## Appendix A: Technology Stack Summary

**Frontend:**
- React 18 with TypeScript
- Material-UI component library
- Redux for state management
- Recharts for data visualization
- Axios for API communication

**Backend:**
- Node.js 20 with Express (API services)
- Python 3.11 (ML services)
- TypeScript for type safety
- Docker containers on ECS Fargate

**Databases:**
- PostgreSQL 15 (RDS)
- DynamoDB (NoSQL)
- Redis 7 (ElastiCache)
- S3 (Data Lake)

**ML/AI:**
- Python: scikit-learn, pandas, numpy
- Time-series: Prophet, statsmodels (ARIMA)
- ML: XGBoost, LightGBM
- Deep Learning: TensorFlow, PyTorch
- SageMaker: Training, inference, pipelines

**Infrastructure:**
- AWS CDK with TypeScript
- Docker for containerization
- GitHub for version control
- GitHub Actions / AWS CodePipeline for CI/CD

**Monitoring:**
- CloudWatch (metrics, logs, alarms)
- X-Ray (distributed tracing)
- CloudTrail (audit logs)
- Custom dashboards

---

## Appendix B: Glossary

- **ARIMA:** AutoRegressive Integrated Moving Average - statistical forecasting model
- **CDN:** Content Delivery Network
- **DLQ:** Dead Letter Queue - for failed message processing
- **ECS:** Elastic Container Service
- **ETL:** Extract, Transform, Load - data processing pipeline
- **IAM:** Identity and Access Management
- **KMS:** Key Management Service
- **MAPE:** Mean Absolute Percentage Error - forecast accuracy metric
- **MFA:** Multi-Factor Authentication
- **RPO:** Recovery Point Objective - maximum acceptable data loss
- **RTO:** Recovery Time Objective - maximum acceptable downtime
- **SKU:** Stock Keeping Unit - unique product identifier
- **TLS:** Transport Layer Security - encryption protocol
- **VPC:** Virtual Private Cloud - isolated network

---

## Appendix C: References

- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [AWS SageMaker Best Practices](https://docs.aws.amazon.com/sagemaker/latest/dg/best-practices.html)
- [Microservices Architecture Patterns](https://microservices.io/patterns/)
- [OWASP Security Guidelines](https://owasp.org/www-project-top-ten/)
- [The Twelve-Factor App](https://12factor.net/)
- [Site Reliability Engineering (Google)](https://sre.google/books/)

---

**Document Version:** 1.0  
**Last Updated:** February 15, 2026  
**Status:** Final Design Specification  
**Next Review:** March 15, 2026
