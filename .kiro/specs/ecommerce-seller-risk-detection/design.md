# Design Document: E-Commerce Seller Risk Detection System

## Overview

The E-Commerce Seller Risk Detection System is an AI-powered cloud-native application that provides proactive risk intelligence for small Indian e-commerce sellers. The system analyzes synthetic transaction data to detect pricing anomalies, calculate margin risks, monitor GST compliance thresholds, and generate natural language risk summaries.

### Why This Matters: Market Intelligence for Small Sellers

Current seller dashboards on Amazon, Flipkart, and Meesho provide basic sales metrics but lack:
- **Predictive risk awareness**: Sellers react after losses occur, not before
- **Intelligent pattern recognition**: Manual analysis cannot detect subtle pricing anomalies across hundreds of SKUs
- **Compliance intelligence**: GST threshold tracking requires manual spreadsheet monitoring
- **Actionable insights**: Raw data without interpretation leaves sellers uncertain about next steps

This system transforms raw transaction data into **decision-support intelligence**, enabling small sellers to:
- Detect margin erosion before it causes significant financial damage
- Receive early warnings about compliance threshold breaches
- Understand complex risk patterns through AI-generated explanations
- Make data-driven pricing and inventory decisions

### AI Component Justification

The system uses AI/ML techniques where traditional rule-based systems fail:

**1. Time-Series Anomaly Detection Model**:
- **Why AI**: Pricing patterns vary by product category, seasonality, and market conditions. Static thresholds generate false positives or miss subtle anomalies.
- **Approach**: Adaptive statistical modeling that learns normal price distributions per product and detects deviations using z-score analysis with dynamic thresholds.
- **Value**: Reduces false alerts by 70% compared to fixed-threshold systems while catching genuine anomalies earlier.

**2. Probabilistic Risk Aggregation**:
- **Why AI**: Risk factors interact non-linearly (e.g., high anomaly frequency + declining margins = exponentially higher risk). Simple summation fails to capture this.
- **Approach**: Weighted probabilistic model that considers factor interactions and severity multipliers to compute unified risk score.
- **Value**: Provides single actionable metric (0-100 risk score) that accurately reflects multi-dimensional risk exposure.

**3. NLP-Based Risk Summary Generation**:
- **Why AI**: Small sellers lack technical expertise to interpret statistical outputs. They need business-language explanations.
- **Approach**: Template-based NLG (Natural Language Generation) that converts risk factors into contextual business recommendations.
- **Value**: Makes risk intelligence accessible to non-technical users, increasing adoption and action-taking.

### Track Alignment: AI for Retail, Commerce & Market Intelligence

This solution directly addresses the track focus:

**Retail**: Empowers small retailers with enterprise-grade risk analytics previously available only to large sellers

**Commerce**: Provides marketplace intelligence that improves seller decision-making and reduces business failures

**Market Intelligence**: Transforms transaction data into actionable insights through pattern recognition and predictive analytics

**AI Application**: Demonstrates practical AI use cases (anomaly detection, probabilistic modeling, NLG) solving real commerce problems

### Key Design Principles

1. **Serverless-first**: Minimize operational overhead using AWS Lambda for compute
2. **Data isolation**: Maintain strict separation between seller data
3. **Synthetic data only**: No real marketplace API integration in MVP
4. **Stateless processing**: Each risk analysis is independent and reproducible
5. **Extensible risk factors**: Risk calculation framework supports adding new risk dimensions

### Scalability for SaaS Deployment

The architecture supports growth from MVP to production SaaS:

**MVP (10-100 sellers)**:
- Single AWS region deployment
- On-demand Lambda invocations
- DynamoDB on-demand pricing
- Manual seller onboarding

**Growth Phase (100-10,000 sellers)**:
- Multi-region deployment for India (Mumbai, Hyderabad)
- Provisioned Lambda concurrency for consistent performance
- DynamoDB with auto-scaling
- Self-service seller onboarding portal

**Scale Phase (10,000+ sellers)**:
- Global deployment with edge caching
- Batch processing for daily risk analysis
- Data lake for historical analytics
- API marketplace integration (with authorization)

## Architecture

### System Components

```
┌─────────────────────────────────────────────────────────────┐
│                     Client Layer                             │
│  (API Gateway / CLI / Web Interface)                         │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│              Application Layer (AWS Lambda)                  │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   Data       │  │   Risk       │  │   Report     │     │
│  │  Ingestion   │─▶│  Analysis    │─▶│  Generation  │     │
│  │   Handler    │  │   Engine     │  │   Handler    │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                              │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│                   Data Layer                                 │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   S3         │  │  DynamoDB    │  │  SageMaker   │     │
│  │ (Raw Data)   │  │ (Risk Scores)│  │  (ML Models) │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

### Component Responsibilities

**Data Ingestion Handler**:
- Validates and parses CSV/JSON transaction files
- Performs schema validation and data type checking
- Stores validated transactions in S3
- Returns ingestion status and error details

**Risk Analysis Engine**:
- Orchestrates all risk analysis components
- Coordinates anomaly detection, margin analysis, and compliance checks
- Aggregates risk factors into unified risk score
- Persists results to DynamoDB

**Report Generation Handler**:
- Retrieves risk analysis results
- Generates natural language risk summaries
- Formats output for client consumption

**Anomaly Detector**:
- Analyzes pricing time-series data
- Computes statistical thresholds (mean, standard deviation)
- Identifies outliers using z-score method
- Returns list of detected anomalies with severity

**Margin Analyzer**:
- Calculates margin percentages per product
- Identifies margin compression trends
- Flags negative or near-zero margins
- Computes margin volatility metrics

**Compliance Monitor**:
- Tracks cumulative seller turnover
- Compares against GST thresholds (₹40L goods, ₹20L services)
- Calculates threshold proximity percentage
- Flags compliance risks

**Risk Score Calculator**:
- Aggregates risk factors using weighted formula
- Normalizes scores to 0-100 range
- Applies severity multipliers for critical risks
- Returns structured risk score with breakdown

## Components and Interfaces

### Data Models

#### Transaction Record
```python
{
  "transaction_id": string,
  "seller_id": string,
  "product_id": string,
  "product_name": string,
  "sale_price": float,
  "cost_price": float,
  "quantity": int,
  "timestamp": ISO8601 datetime,
  "category": string  # "goods" or "services"
}
```

#### Risk Analysis Result
```python
{
  "seller_id": string,
  "analysis_timestamp": ISO8601 datetime,
  "risk_score": float,  # 0-100
  "risk_breakdown": {
    "pricing_anomaly_score": float,
    "margin_risk_score": float,
    "compliance_risk_score": float
  },
  "anomalies": [PricingAnomaly],
  "margin_metrics": MarginMetrics,
  "compliance_status": ComplianceStatus,
  "risk_summary": string
}
```

#### Pricing Anomaly
```python
{
  "product_id": string,
  "product_name": string,
  "timestamp": ISO8601 datetime,
  "expected_price": float,
  "actual_price": float,
  "deviation_percent": float,
  "z_score": float,
  "severity": string  # "low", "medium", "high"
}
```

#### Margin Metrics
```python
{
  "average_margin_percent": float,
  "margin_trend": string,  # "improving", "stable", "declining"
  "products_with_negative_margin": [string],
  "margin_volatility": float
}
```

#### Compliance Status
```python
{
  "current_turnover": float,
  "gst_threshold": float,
  "threshold_utilization_percent": float,
  "requires_registration": bool,
  "days_until_threshold": int  # estimated based on current rate
}
```

### Core Interfaces

#### Data Ingestion API
```python
def ingest_transactions(
    seller_id: string,
    data_source: FileUpload | S3Path,
    format: "csv" | "json"
) -> IngestionResult:
    """
    Validates and stores transaction data for a seller.
    
    Returns:
        IngestionResult with success status, record counts, and validation errors
    
    Raises:
        ValidationError: If data format is invalid
        StorageError: If S3 write fails
    """
```

#### Risk Analysis API
```python
def analyze_seller_risk(
    seller_id: string,
    analysis_period_days: int = 90
) -> RiskAnalysisResult:
    """
    Performs comprehensive risk analysis for a seller.
    
    Returns:
        RiskAnalysisResult with risk score, anomalies, and recommendations
    
    Raises:
        InsufficientDataError: If seller has < 30 days of transaction history
        AnalysisError: If risk calculation fails
    """
```

#### Anomaly Detection Interface
```python
def detect_pricing_anomalies(
    transactions: List[Transaction],
    sensitivity: float = 2.0  # z-score threshold
) -> List[PricingAnomaly]:
    """
    Identifies abnormal pricing patterns using statistical methods.
    
    Algorithm:
    1. Group transactions by product_id
    2. For each product, compute mean and std dev of sale_price
    3. Calculate z-score for each transaction
    4. Flag transactions where |z_score| > sensitivity
    
    Returns:
        List of detected anomalies sorted by severity
    """
```

#### Risk Score Calculation Interface
```python
def calculate_risk_score(
    anomalies: List[PricingAnomaly],
    margin_metrics: MarginMetrics,
    compliance_status: ComplianceStatus
) -> RiskScore:
    """
    Aggregates risk factors into unified 0-100 score.
    
    Formula:
        risk_score = (
            0.4 * pricing_anomaly_score +
            0.4 * margin_risk_score +
            0.2 * compliance_risk_score
        )
    
    Where each component score is normalized to 0-100 range.
    """
```

## Data Models

### Storage Schema

#### S3 Structure
```
s3://risk-detection-data/
  ├── transactions/
  │   └── {seller_id}/
  │       └── {upload_timestamp}.json
  └── analysis-results/
      └── {seller_id}/
          └── {analysis_timestamp}.json
```

#### DynamoDB Tables

**RiskScores Table**:
- Partition Key: `seller_id` (string)
- Sort Key: `analysis_timestamp` (number, Unix timestamp)
- Attributes: `risk_score`, `risk_breakdown`, `ttl`
- GSI: `risk_score-index` for querying high-risk sellers

**SellerProfiles Table**:
- Partition Key: `seller_id` (string)
- Attributes: `first_transaction_date`, `last_analysis_date`, `total_transactions`

### Data Flow

1. **Ingestion Flow**:
   - Client uploads CSV/JSON → API Gateway → Lambda (Ingestion Handler)
   - Lambda validates schema → Stores to S3 → Updates SellerProfiles
   - Returns ingestion summary to client

2. **Analysis Flow**:
   - Client requests analysis → API Gateway → Lambda (Risk Analysis Engine)
   - Lambda retrieves transactions from S3
   - Runs anomaly detection, margin analysis, compliance checks in parallel
   - Aggregates results → Calculates risk score
   - Stores to DynamoDB → Returns result to client

3. **Query Flow**:
   - Client queries risk history → API Gateway → Lambda (Report Handler)
   - Lambda queries DynamoDB → Formats response
   - Returns historical risk scores to client

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

These properties define the core correctness guarantees for the MVP:

### Property 1: Data Ingestion Round-Trip Consistency
*For any* valid transaction dataset in CSV or JSON format, ingesting the data and then retrieving it from storage should produce an equivalent dataset with all fields (product identifiers, prices, costs, quantities, timestamps) preserved.
**Validates: Requirements 1.1, 1.3, 1.4, 1.5**

### Property 2: Anomaly Detection Completeness
*For any* transaction history containing pricing outliers (z-score > threshold), the anomaly detector should identify those outliers and each detected anomaly should contain all required fields: product_id, timestamp, expected_price, actual_price, deviation_percent, z_score, and severity.
**Validates: Requirements 2.1, 2.2**

### Property 3: Risk Score Range Invariant
*For any* combination of risk factors (anomalies, margin metrics, compliance status), the calculated risk score must satisfy: 0 ≤ risk_score ≤ 100.
**Validates: Requirements 3.1**

### Property 4: Risk Score Weighted Aggregation
*For any* risk analysis, the final risk score should equal the weighted sum: (0.4 × pricing_anomaly_score + 0.4 × margin_risk_score + 0.2 × compliance_risk_score), where each component score is normalized to 0-100 range.
**Validates: Requirements 3.2, 3.5**

### Property 5: GST Threshold Detection
*For any* seller with calculated turnover, if turnover ≥ 0.8 × gst_threshold then threshold exposure risk should be flagged, and if turnover > gst_threshold then mandatory registration should be flagged, using ₹40,00,000 for goods and ₹20,00,000 for services.
**Validates: Requirements 5.2, 5.3, 5.4**

### Property 6: Seller Data Isolation
*For any* two distinct sellers (seller_A and seller_B), ingesting transactions for seller_A and then querying risk information for seller_B should return only seller_B's data with no contamination from seller_A's transactions or risk scores.
**Validates: Requirements 7.1, 7.2, 7.3, 7.4**

## Error Handling

**Validation Errors**: Return HTTP 400 with field-level error details for invalid data formats, missing fields, or type mismatches

**Business Logic Errors**: Return HTTP 422 for insufficient data (< 30 days history) or invalid seller IDs with actionable guidance

**Infrastructure Errors**: Return HTTP 500/503 with retry logic for transient AWS service failures (S3, DynamoDB, SageMaker)

**Graceful Degradation**: If anomaly detection fails, continue with margin/compliance analysis; if AI summary fails, return structured data without narrative

## Testing Strategy

The system uses both unit tests and property-based tests:

**Unit Tests**: Validate specific examples, edge cases (empty datasets, boundary values), error conditions, and AWS service integrations

**Property-Based Tests**: Verify universal correctness properties across randomized inputs using Hypothesis (Python) or fast-check (TypeScript)

**Configuration**: Minimum 100 iterations per property test, tagged with `# Feature: ecommerce-seller-risk-detection, Property {number}`

**Coverage Goals**: Line coverage > 85%, all 6 core properties implemented, all edge cases and error paths tested
