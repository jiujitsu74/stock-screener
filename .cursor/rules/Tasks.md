Tasks.md
# ALGORITHMIC TRADING SYSTEM IMPLEMENTATION

## AGENT INSTRUCTIONS
1. Read tasks sequentially starting with STATUS: PENDING tasks with PRIORITY: HIGH
2. Update task status from PENDING to IN_PROGRESS when starting work
3. Update task status from IN_PROGRESS to COMPLETED when finished
4. Only work on tasks when all dependencies are COMPLETED
5. Reference existing code from completed tasks when implementing new components
6. Update this document with progress notes in the NOTES section of each task
7. Ask for clarification when task details are insufficient

## TASK TABLE

| ID | TITLE | STATUS | PRIORITY | DEPENDENCIES |
|----|-------|--------|----------|--------------|
| 1 | Set up AWS Environment and IAM Configuration | COMPLETED | HIGH | None |
| 2 | Create S3 Bucket and Configure Access Patterns | COMPLETED | HIGH | 1 |
| 3 | Set up DynamoDB Tables with Proper Schema | COMPLETED | HIGH | 1 |
| 4 | Configure CloudWatch Monitoring and Alerting | COMPLETED | MEDIUM | 1 |
| 5 | Launch and Configure EC2 PyMC Server | COMPLETED | HIGH | 1, 2, 3 |
| 6 | Create EventBridge Rules for Scheduling | COMPLETED | MEDIUM | 1 |
| 7 | Set up Lambda Function Environment | COMPLETED | HIGH | 1, 2, 3 |
| 8 | Implement Model 1: Stock Screener Core | COMPLETED | HIGH | 7 |
| 9 | Implement Google Sheets Data Provider | COMPLETED | HIGH | 8 |
| 10 | Develop Stock Scoring Algorithms | COMPLETED | HIGH | 8, 9 |
| 11 | Implement S3 Storage for Model 1 | COMPLETED | HIGH | 8, 10 |
| 12 | Implement Model 2: Portfolio Optimizer | COMPLETED | HIGH | 8, 11 |
| 13 | Fix S3 Data Retrieval in Model 2 | COMPLETED | HIGH | 12 |
| 14 | Implement Fallback Optimization Logic | COMPLETED | MEDIUM | 12 |
| 15 | Implement DynamoDB Operations in Model 2 | COMPLETED | HIGH | 3, 12 |
| 16 | Create Trade Execution with Alpaca API | COMPLETED | HIGH | 12 |
| 17 | Implement EC2 PyMC API for Portfolio Optimization | COMPLETED | HIGH | 5 |
| 18 | Develop PyMC Bayesian Optimization Models | COMPLETED | HIGH | 5, 17 |
| 19 | Implement Model 3: Meta-Optimizer | COMPLETED | MEDIUM | 12, 15, 17 |
| 20 | Set up Performance Tracking System | COMPLETED | MEDIUM | 15, 19 |
| 21 | Implement Integration Testing Suite | COMPLETED | HIGH | 8, 12, 19 |
| 22 | Implement Performance Benchmarking | COMPLETED | HIGH | 21 |
| 23 | Optimize Model 1 Throughput | COMPLETED | HIGH | 22 |
| 24 | Implement Caching Layer | COMPLETED | MEDIUM | 22 |
| 25 | Enhance Monitoring and Alerting | IN_PROGRESS | MEDIUM | 22 |
| 26 | Implement Load Balancing | PENDING | MEDIUM | 23 |
| 27 | Optimize Model 2 CPU Usage | PENDING | MEDIUM | 22 |
| 28 | Improve AWS Service Response Times | PENDING | MEDIUM | 22 |

## TASK DETAILS

### TASK-1: Set up AWS Environment and IAM Configuration
**DESCRIPTION**: Configure the AWS environment for the algorithmic trading system with appropriate IAM roles and permissions.
**ACCEPTANCE CRITERIA**:
- AWS account is properly configured with organizational structure
- IAM roles defined with least privilege principle
- AWS CLI and SDK access configured for development
- VPC set up with proper subnets for EC2 instances
- Network security and access controls implemented
**IMPLEMENTATION NOTES**:
```
1. Create IAM roles:
   - StockScreenerRole: S3 read/write access
   - PortfolioOptimizerRole: S3 read, DynamoDB read/write, EC2 API access
   - MetaOptimizerRole: DynamoDB read/write, EC2 API access
   - EC2PyMCRole: S3 read, DynamoDB read
2. Set up VPC with private subnet for EC2 and public subnet for API access
3. Configure security groups to allow necessary traffic
```
**NOTES**: 
- Created TradingSystemDynamoDBPolicy for consistent DynamoDB access across all models
- Updated Model2PortfolioOptimizerRole with:
  - TradingSystemDynamoDBPolicy (Full DynamoDB access)
  - AWSLambdaBasicExecutionRole
  - AmazonS3ReadOnlyAccess
- Updated Model3MetaOptimizerRole with:
  - TradingSystemDynamoDBPolicy (Full DynamoDB access)
  - AmazonEC2ReadOnlyAccess
  - AWSLambdaBasicExecutionRole
  - AmazonS3ReadOnlyAccess
- Verified access to all required DynamoDB tables:
  - trading-portfolio
  - trading-performance
  - trading-parameters

### TASK-2: Create S3 Bucket and Configure Access Patterns
**DESCRIPTION**: Create and configure the S3 bucket for model data exchange with appropriate access patterns and lifecycle rules.
**ACCEPTANCE CRITERIA**:
- S3 bucket `stock-screener-data-20250320` created
- Folder structure with `screening_results/LATEST/` path
- Bucket policies and permissions configured
- Lifecycle rules set for data retention
- CORS configured if needed
- Bucket access tested from development environment
**IMPLEMENTATION NOTES**:
```
1. Create bucket with server-side encryption
2. Set up folder structure with appropriate prefixes
3. Configure lifecycle rules to archive older data
4. Create bucket policy to restrict access
5. Test access from Lambda development environment
```
**NOTES**:

### TASK-8: Implement Model 1: Stock Screener Core
**DESCRIPTION**: Create the core Lambda function for the Stock Screener with proper entrypoints and action routing.
**ACCEPTANCE CRITERIA**:
- Base Lambda function created with proper entrypoints
- Core data structures and configurations implemented
- Lambda handler with action routing implemented
- Modular architecture for extensibility
- Comprehensive error handling implemented
- Logging and monitoring hooks added
**IMPLEMENTATION NOTES**:
```python
# model_one_stock_screener.py structure:
import json
import logging
import os
import pandas as pd
import numpy as np
import boto3
import requests

# Configure logging
logger = logging.getLogger()
logger.setLevel(logging.INFO)

# Initialize S3 client
s3_client = boto3.client('s3')

# Lambda handler
def lambda_handler(event, context):
    # Extract parameters from event
    # Route actions based on 'action' parameter
    # Implement error handling
    # Return appropriate response
```
**NOTES**: 
- COMPLETED - Base implementation has been done with action routing for 'get_screened_stocks', 'portfolio_optimize', and 'execute_trades'. 
- Comprehensive error handling is in place with detailed logging.
- Integration tests implemented in tests/test_model1_integration.py covering:
  - Concurrent stock screening operations
  - Data flow between Model 1 and Model 2
  - Error handling and recovery scenarios
  - S3 operations with proper mocking
  - Thread safety in concurrent operations

### TASK-12: Implement Model 2: Portfolio Optimizer
**DESCRIPTION**: Create the Portfolio Optimizer Lambda function to optimize portfolio allocations based on screened stocks.
**ACCEPTANCE CRITERIA**:
- Base Lambda function structure created
- Core configuration and utilities implemented
- Parameter extraction and validation implemented
- Main optimization workflow created
- Lambda handler with timeout protection implemented
- Metrics collection and publishing added
**IMPLEMENTATION NOTES**:
```python
# model_two_portfolio_optimizer.py structure:
import json
import logging
import os
import sys
import time
import boto3
import requests
from datetime import datetime
from decimal import Decimal

# Configure logging
logger = logging.getLogger()
logger.setLevel(logging.INFO)

# Set function timeout (330s Lambda max with 10s buffer)
FUNCTION_TIMEOUT = 320

# Lambda handler
def lambda_handler(event, context):
    start_time = time.time()
    
    try:
        # Extract parameters from event
        # Get screened stocks from S3
        # Call EC2 for optimization or use fallback
        # Store results in DynamoDB
        # Execute trades if requested
        # Return response
    except Exception as e:
        logger.error(f"Error: {str(e)}")
        # Return error response
```
**NOTES**: PARTIALLY COMPLETED - Base implementation is done with fixes for S3 data retrieval and fallback optimization logic. Still need to implement DynamoDB operations and trade execution.

### TASK-13: Fix S3 Data Retrieval in Model 2
**DESCRIPTION**: Implement direct S3 data retrieval in Model 2 with proper error handling and timeout protection.
**ACCEPTANCE CRITERIA**:
- Direct S3 data retrieval implemented
- Proper error handling for S3 operations added
- Fallback to Lambda invocation implemented if needed
- Timeout protection for S3 operations implemented
- Data validation after retrieval added
- Caching mechanism implemented if appropriate
**IMPLEMENTATION NOTES**:
```python
def get_screened_stocks_from_s3():
    """Get screened stocks data from S3 where Model 1 stored it"""
    try:
        session = boto3.session.Session()
        s3_client = session.client(
            's3',
            config=boto3.session.Config(
                connect_timeout=10,  # 10 second connection timeout
                read_timeout=15      # 15 second read timeout
            )
        )
        
        # Get the latest screened stocks file
        # Process and validate the data
        # Return the processed data
    except Exception as e:
        logger.warning(f"Error loading from S3: {str(e)}")
        # Return empty list or fallback data
```
**NOTES**: COMPLETED - Implemented with session-based S3 client, proper timeouts, error handling, and fallback mechanism.

### TASK-15: Implement DynamoDB Operations in Model 2
**Status**: COMPLETED
**Priority**: HIGH

**Description**: Implement DynamoDB operations for storing and retrieving portfolio weights and performance metrics.

**NOTES**:
- Implementation completed with `store_portfolio_weights` and `store_performance_metrics` functions
- Fixed table name mismatch by updating Lambda environment variables
- Updated performance data functions to use correct key schema:
  - Using `model_id` as partition key (HASH)
  - Using `timestamp` as sort key (RANGE) in Unix timestamp format
  - Keeping `portfolio_id` as a filterable attribute
- Verified table structure and data format alignment
- Ensured consistent IAM permissions across all models for DynamoDB access
- Standardized table access patterns:
  - Model 2 writes to trading-portfolio and trading-performance
  - Model 3 reads from trading-performance and writes to trading-parameters
  - All models have proper permissions through TradingSystemDynamoDBPolicy

### TASK-21: Implement Integration Testing Suite
**DESCRIPTION**: Create comprehensive integration tests for all models and their interactions.
**ACCEPTANCE CRITERIA**:
- Test coverage for all three models
- Concurrent operation testing
- Error handling verification
- Data flow validation
- Mock AWS services properly
**IMPLEMENTATION NOTES**:
```python
# Integration test structure:
# 1. test_model1_integration.py
#    - Concurrent stock screening
#    - Model 1 to Model 2 data flow
#    - Error handling and recovery
#    - S3 operations

# 2. test_concurrent_access.py
#    - Model 2 concurrent writes
#    - Model 3 reads during writes
#    - Race condition handling
#    - DynamoDB operations

# 3. test_long_term_stability.py
#    - Performance tracking
#    - Parameter optimization
#    - System stability
```
**NOTES**:
- COMPLETED - Full test suite implemented with coverage for all models:
  1. Model 1 (Stock Screener):
     - Concurrent stock screening operations verified
     - S3 data storage and retrieval tested
     - Integration with Model 2 validated
     - Error handling and recovery tested
  2. Model 2 (Portfolio Optimizer):
     - Concurrent write operations tested
     - DynamoDB operations verified
     - Data structure consistency checked
     - Integration with Model 1 and 3 validated
  3. Model 3 (Meta-Optimizer):
     - Performance data tracking tested
     - Parameter optimization verified
     - Long-term stability monitored
     - Integration with Model 2 validated
- Mock implementations for AWS services:
  - S3 for stock screening data
  - DynamoDB for portfolio and performance data
  - EC2 for PyMC optimization
- Thread safety and concurrent access patterns verified
- Error handling and recovery scenarios tested
- Data format consistency validated across all models

### TASK-22: Implement Performance Benchmarking
**DESCRIPTION**: Create comprehensive performance tests to measure system throughput, latency, and resource utilization.
**ACCEPTANCE CRITERIA**:
- Benchmark tests for all three models
- Performance metrics collection and reporting
- Resource utilization monitoring
- AWS service response time tracking
- Scalability testing
**IMPLEMENTATION NOTES**:
```python
# Performance test structure:
# 1. test_performance_benchmark.py
#    - Stock screening throughput
#    - Portfolio optimization latency
#    - Meta-optimization convergence time
#    - AWS service response times
#    - Resource utilization metrics

# Key metrics to track:
# - Model 1:
#   * Stock screening throughput (stocks/second)
#   * S3 write latency
#   * Memory usage during screening
#   * CPU utilization
#
# - Model 2:
#   * Portfolio optimization time
#   * DynamoDB write latency
#   * EC2 API response time
#   * Memory usage during optimization
#
# - Model 3:
#   * Parameter optimization time
#   * Performance data retrieval latency
#   * Meta-optimization convergence time
#   * Resource utilization during optimization
```
**NOTES**:
- COMPLETED - Performance benchmarking suite fully implemented:
  1. Model 1 Performance Tests:
     - Latest Results:
       * Throughput: 0 ops/sec (below 10 ops/sec requirement)
       * Latency: 12.33 ms (well below 100ms requirement)
       * Memory Usage: 36.97 MB (well below 512MB limit)
       * CPU Usage: 35.29% (well below 80% limit)
       * Error Rate: 0.00% (below 0.001 requirement)

  2. Model 2 Performance Tests:
     - Latest Results:
       * Latency: 24.31 ms (well below 200ms requirement)
       * Memory Usage: 32.74 MB (well below 512MB limit)
       * CPU Usage: 46.28% (well below 80% limit)
       * Error Rate: 0.00% (below 0.001 requirement)

  3. Model 3 Performance Tests:
     - Latest Results:
       * Latency: 33.83 ms (well below 300ms requirement)
       * Memory Usage: 34.96 MB (well below 512MB limit)
       * CPU Usage: 17.76% (well below 80% limit)
       * Error Rate: 0.00% (below 0.001 requirement)

  4. AWS Service Response Times:
     - Latest Results:
       * Mean Response Time: 12.13 ms (well below 50ms requirement)
       * Max Response Time: 13.61 ms
       * Error Rate: 0.00% (below 0.001 requirement)

  Implementation Details:
  - Created efficient MockDataProvider for consistent test data
  - Implemented comprehensive PerformanceMetrics tracking
  - Added detailed logging and reporting
  - Included proper error handling and recovery
  - All tests passing with significant margin under requirements (except Model 1 throughput)

  Portfolio Analysis Implementation:
  - Added test_portfolio_analysis.py for detailed performance analysis
  - Implemented DynamoDB data retrieval with proper error handling
  - Added metrics tracking for:
    * Average stocks screened (21.0)
    * Average stocks passed (1.2)
    * Pass rate (5.72%)
    * Execution time (0.04s avg, 1.25s max)
  - Performance trend analysis shows improving execution times
  - Analysis period: March 19, 2025 to March 27, 2025 (35 records)

  Next Steps:
  - Investigate and fix Model 1 throughput issues
  - Monitor performance under production loads
  - Implement automated performance regression testing
  - Add stress testing scenarios
  - Consider implementing load balancing if needed

## SYSTEM ARCHITECTURE

### COMPONENTS
1. **Model 1: Stock Screener**
   - Fetches stock data from Google Sheets using `GoogleSheetsProvider`
   - Implements parallel processing for score calculations
   - Uses Redis caching for performance optimization
   - Stores results in S3 with historical and latest versions
   - Performance metrics:
     * Throughput: 50.45 ops/sec
     * Latency: 192.02 ms
     * Memory Usage: 36.97 MB
     * CPU Usage: 35.29%
     * Error Rate: 0.00%

2. **Model 2: Portfolio Optimizer**
   - Reads screened stocks from S3
   - Implements Bayesian optimization via EC2 PyMC server
   - Uses fallback optimization logic when needed
   - Stores portfolio weights in DynamoDB
   - Executes trades via Alpaca API
   - Performance metrics:
     * Latency: 24.31 ms
     * Memory Usage: 32.74 MB
     * CPU Usage: 46.28%
     * Error Rate: 0.00%

3. **Model 3: Meta-Optimizer**
   - Analyzes performance data from DynamoDB
   - Optimizes model parameters via EC2 PyMC server
   - Updates parameters in DynamoDB
   - Performance metrics:
     * Latency: 33.83 ms
     * Memory Usage: 34.96 MB
     * CPU Usage: 17.76%
     * Error Rate: 0.00%

4. **EC2 PyMC Server**
   - **Instance ID**: i-06665c98a2a14b815
   - **Public IP**: 3.133.103.19
   - **Public DNS**: ec2-3-133-103-19.us-east-2.compute.amazonaws.com
   - **Status**: Running
   - **Endpoints**:
     - Health Check: `/health` (GET)
     - Portfolio Optimization: `/optimize-portfolio` (POST)
     - Parameter Optimization: `/optimize_parameters` (POST)
   - **Service Status**: Active and responding
   - **Last Verified**: 2025-04-12

### INTEGRATION POINTS
```
MODEL 1 → S3 → MODEL 2 → ALPACA API
                ↓
                EC2 ← MODEL 3
                ↓       ↑
            DYNAMODB ───┘
```

### DATA FLOW
1. **Stock Data Flow**:
   - Google Sheets → Model 1 (Stock Screener)
   - Model 1 → S3 (Historical and Latest Results)
   - S3 → Model 2 (Portfolio Optimizer)
   - Model 2 → DynamoDB (Portfolio Weights)
   - Model 2 → Alpaca API (Trade Execution)

2. **Performance Data Flow**:
   - Model 1 → CloudWatch (Performance Metrics)
   - Model 2 → DynamoDB (Portfolio Performance)
   - Model 3 → DynamoDB (Parameter Updates)
   - EC2 → Model 2/3 (Optimization Results)

3. **Cache Flow**:
   - Redis → Model 1 (Stock Data Caching)
   - Redis → Model 2 (Portfolio Weights Caching)
   - Redis → Model 3 (Parameter Caching)

### ENVIRONMENT VARIABLES
```
# API Keys
ALPHA_API_KEY=your_alpha_vantage_key
ALPACA_API_KEY=your_alpaca_key
ALPACA_SECRET_KEY=your_alpaca_secret

# AWS Configuration
OUTPUT_BUCKET=stock-screener-data-20250320
OUTPUT_KEY_PREFIX=screening_results
PORTFOLIO_TABLE=Portfolio
PARAMETERS_TABLE=Parameters
EC2_API_ENDPOINT=http://ec2-3-133-103-19.us-east-2.compute.amazonaws.com:5000

# Google Sheets
GOOGLE_SHEET_ID=your_google_sheet_id

# Redis Configuration
REDIS_ENDPOINT=your_redis_endpoint
REDIS_PORT=6379
CACHE_TTL=3600  # 1 hour in seconds
```

### PERFORMANCE TARGETS
1. **Model 1 (Stock Screener)**:
   - Throughput: > 10 ops/sec (Current: 50.45 ops/sec)
   - Latency: < 200 ms (Current: 192.02 ms)
   - Memory Usage: < 512 MB (Current: 36.97 MB)
   - CPU Usage: < 80% (Current: 35.29%)
   - Error Rate: < 0.001% (Current: 0.00%)

2. **Model 2 (Portfolio Optimizer)**:
   - Latency: < 200 ms (Current: 24.31 ms)
   - Memory Usage: < 512 MB (Current: 32.74 MB)
   - CPU Usage: < 80% (Current: 46.28%)
   - Error Rate: < 0.001% (Current: 0.00%)

3. **Model 3 (Meta-Optimizer)**:
   - Latency: < 300 ms (Current: 33.83 ms)
   - Memory Usage: < 512 MB (Current: 34.96 MB)
   - CPU Usage: < 80% (Current: 17.76%)
   - Error Rate: < 0.001% (Current: 0.00%)

4. **AWS Service Response Times**:
   - Mean Response Time: < 50 ms (Current: 12.13 ms)
   - Max Response Time: < 100 ms (Current: 13.61 ms)
   - Error Rate: < 0.001% (Current: 0.00%)

## CODE REFERENCE

### MODEL 1 CORE STRUCTURE
```python
# Key components:
class GoogleSheetsProvider:
    def get_sheet_as_csv_url(self, sheet_gid):
        # Generate URL for Google Sheets export
    
    def fetch_data_with_retries(self, sheet_name, sheet_gid, cache_key):
        # Fetch data with retry mechanism
    
    def _process_stock_data(self, df):
        # Process and clean stock data
    
    def get_stock_data(self):
        # Public method to get stock data

class StockScreener:
    def calculate_daily_score(self, stock_data):
        # Calculate daily scores for stocks
    
    def calculate_weekly_score(self, stock_data):
        # Calculate weekly scores for stocks
    
    def calculate_monthly_score(self, stock_data):
        # Calculate monthly scores for stocks
    
    def calculate_combined_score(self, stock_data):
        # Calculate combined scores
    
    def rank_stocks(self, stock_data):
        # Rank stocks based on score
    
    def screen_stocks(self, limit=50, force_refresh=False):
        # Main screening function
    
    def save_results_to_s3(self, results):
        # Save results to S3
```

### MODEL 2 CORE STRUCTURE
```python
# Key functions:
def fix_python_path():
    # Fix Python's import path issues

def get_screened_stocks_from_s3():
    # Get stock data from S3
    
def call_ec2_bayesian_optimization(screened_stocks, portfolio_id, risk_tolerance):
    # Call EC2 for Bayesian optimization
    
def create_fallback_allocations(screened_stocks):
    # Create fallback allocations if optimization fails

def lambda_handler(event, context):
    # Main entry point with timeout protection
```

## TESTING INSTRUCTIONS

1. **Unit Testing Each Component**:
   - Test each function in isolation
   - Use mock data for dependencies
   - Verify expected outputs

2. **Integration Testing**:
   - Test Model 1 → S3 data flow
   - Test S3 → Model 2 data flow
   - Test Model 2 → DynamoDB operations
   - Test Model 2 → Alpaca API interaction

3. **System Testing**:
   - Test end-to-end workflow
   - Verify proper data flow between all components
   - Test scheduling and automatic execution

## TROUBLESHOOTING

### COMMON ISSUES

1. **Lambda Timeouts**:
   - Check execution time of operations
   - Implement timeout protection
   - Use asynchronous patterns for long operations

2. **S3 Access Issues**:
   - Verify IAM permissions
   - Check bucket and object existence
   - Debug with detailed logging

3. **DynamoDB Errors**:
   - Check throughput capacity
   - Verify item size limits
   - Test with error simulation

4. **API Connection Issues**:
   - Verify API keys and credentials
   - Test API endpoints directly
   - Implement retry logic with backoff

   Script Implementations
Model 1: Stock Screener (model_one_stock_screener.py)
python# Key components:
# 1. Google Sheets data provider
# 2. Stock scoring algorithms
# 3. S3 storage integration
# 4. Lambda handler with action routing
Model 2: Portfolio Optimizer (model_two_portfolio_optimizer.py)
python# Key components:
# 1. S3 data retrieval
# 2. EC2 Bayesian optimization call
# 3. Fallback optimization logic
# 4. DynamoDB portfolio storage
# 5. Alpaca API trade execution
Model 3: Meta-Optimizer (model_three_meta_optimizer.py)
python# Key components:
# 1. Performance data retrieval
# 2. Parameter optimization logic
# 3. Market regime detection
# 4. DynamoDB parameter updates
# 5. EC2 meta-optimization call
EC2 PyMC Server (ec2_pymc_server.py)
python# Key components:
# 1. Flask/FastAPI REST server
# 2. PyMC Bayesian models
# 3. Portfolio optimization algorithms
# 4. Market regime models
# 5. Parameter meta-optimization logic
Connection Points and Dependencies

Model 1 → S3: Stock Screener writes results to S3
S3 → Model 2: Portfolio Optimizer reads screened stocks from S3
Model 2 → EC2: Portfolio Optimizer calls EC2 for Bayesian optimization
Model 2 → DynamoDB: Portfolio Optimizer writes portfolio weights to DynamoDB
Model 2 → Alpaca API: Portfolio Optimizer executes trades via Alpaca
Model 3 → DynamoDB: Meta-Optimizer reads performance data from DynamoDB
Model 3 → EC2: Meta-Optimizer calls EC2 for meta-optimization
Model 3 → DynamoDB: Meta-Optimizer writes updated parameters to DynamoDB

Critical Dependencies
Python Packages

pandas: Data manipulation throughout all models
numpy: Numerical computations
boto3: AWS service integration
requests: API calls
pymc: Bayesian modeling on EC2
alpaca-trade-api: Trade execution
flask/fastapi: EC2 API server

Environment Variables

ALPHA_API_KEY: Alpha Vantage API key
ALPACA_API_KEY: Alpaca trading API key
ALPACA_SECRET_KEY: Alpaca API secret key
OUTPUT_BUCKET: S3 bucket name
EC2_API_ENDPOINT: EC2 PyMC server endpoint
GOOGLE_SHEET_ID: Google Sheet data source ID
PORTFOLIO_TABLE: DynamoDB portfolio table
PARAMETERS_TABLE: DynamoDB parameters table
```

### TASK-19: Implement Model 3: Meta-Optimizer
**DESCRIPTION**: Create the Meta-Optimizer Lambda function to optimize portfolio parameters based on performance data.
**ACCEPTANCE CRITERIA**:
- Base Lambda function structure created
- Core configuration and utilities implemented
- Parameter extraction and validation implemented
- Main optimization workflow created
- Lambda handler with timeout protection implemented
- Metrics collection and publishing added
**IMPLEMENTATION NOTES**:
```python
# Key components:
# 1. Performance data retrieval
# 2. Parameter optimization logic
# 3. Market regime detection
# 4. DynamoDB parameter updates
# 5. EC2 meta-optimization call
```
**NOTES**:
- COMPLETED - Full implementation with:
  - Lambda function deployed as 3MetaOptimizer
  - Core functionality implemented:
    - Performance data retrieval from DynamoDB
    - Parameter optimization via EC2 PyMC server
    - Parameter history tracking
    - Performance metrics storage
  - Environment configured:
    - DynamoDB tables: trading-portfolio, trading-performance, trading-parameters
    - EC2 PyMC server endpoint configured
    - IAM permissions properly set
  - Testing suite implemented:
    - Unit tests for all major functions
    - Error handling tests
    - Integration tests with EC2 PyMC server
  - Action support:
    - optimize_parameters
    - get_performance_data
    - get_parameter_history
    - update_parameters
```

### DEPLOYMENT INSTRUCTIONS

1. **EC2 PyMC Server Update**:
   ```bash
   # 1. SSH into the instance
   ssh -i ~/.ssh/pymc-bayesian-key-new.pem ec2-user@ec2-3-133-103-19.us-east-2.compute.amazonaws.com

   # 2. Stop current service
   sudo systemctl stop pymc_service

   # 3. Update application code
   sudo cp /path/to/updated/app.py /opt/pymc_service/

   # 4. Update service configuration
   sudo cp /path/to/updated/pymc.service /etc/systemd/system/

   # 5. Restart service
   sudo systemctl daemon-reload
   sudo systemctl start pymc_service
   ```

2. **Required Endpoints**:
   ```python
   # app.py structure
   from flask import Flask, request, jsonify
   import pymc as pm
   import numpy as np  # Available through AWS layer

   app = Flask(__name__)

   @app.route('/health', methods=['GET'])
   def health():
       return jsonify({
           "status": "healthy",
           "service": "PyMC Bayesian Optimization",
           "timestamp": datetime.datetime.now().isoformat(),
           "pymc_available": True
       })

   @app.route('/optimize-portfolio', methods=['POST'])
   def optimize_portfolio():
       data = request.json
       # Portfolio optimization logic
       return jsonify({
           "status": "success",
           "allocations": [...],
           "metadata": {...}
       })

   @app.route('/optimize_parameters', methods=['POST'])
   def optimize_parameters():
       data = request.json
       # Parameter optimization logic
       return jsonify({
           "status": "success",
           "optimized_parameters": {...}
       })
   ```

3. **Lambda Function Updates**:
   ```python
   # Update in model_two_portfolio_optimizer.py
   EC2_API_ENDPOINT = "http://ec2-3-133-103-19.us-east-2.compute.amazonaws.com:5000/optimize-portfolio"
   
   # Update in model_three_meta_optimizer.py
   EC2_API_ENDPOINT = "http://ec2-3-133-103-19.us-east-2.compute.amazonaws.com:5000/optimize_parameters"
   ```

4. **Service Configuration**:
   ```ini
   # /etc/systemd/system/pymc.service
   [Unit]
   Description=PyMC Bayesian Optimization Service
   After=network.target

   [Service]
   User=ec2-user
   WorkingDirectory=/opt/pymc_service
   Environment="PATH=/opt/pymc_service/venv/bin"
   ExecStart=/opt/pymc_service/venv/bin/gunicorn --bind 0.0.0.0:5000 app:app
   Restart=always

   [Install]
   WantedBy=multi-user.target
   ```

5. **Verification Steps**:
   ```bash
   # 1. Check service status
   sudo systemctl status pymc_service

   # 2. Test health endpoint
   curl http://ec2-3-133-103-19.us-east-2.compute.amazonaws.com:5000/health

   # 3. Test optimization endpoint
   curl -X POST -H "Content-Type: application/json" \
        -d '{"portfolio_id": "test", "stocks": ["AAPL", "GOOGL"], "risk_tolerance": 0.5}' \
        http://ec2-3-133-103-19.us-east-2.compute.amazonaws.com:5000/optimize-portfolio

   # 4. Test parameter optimization
   curl -X POST -H "Content-Type: application/json" \
        -d '{"model_id": "portfolio-optimizer", "lookback_days": 30}' \
        http://ec2-3-133-103-19.us-east-2.compute.amazonaws.com:5000/optimize_parameters
   ```

### DEPENDENCIES AND LAYERS

1. **AWS Layers**:
   - numpy-pandas layer (ARN: arn:aws:lambda:us-east-2:039433203618:layer:numpy-pandas:1)
     * Contains: numpy, pandas, matplotlib, scipy, boto3
     * Compatible with: python3.9
     * Status: Active and verified

2. **Environment Variables**:
   ```
   # API Keys
   ALPHA_API_KEY=your_alpha_vantage_key
   ALPACA_API_KEY=your_alpaca_key
   ALPACA_SECRET_KEY=your_alpaca_secret

   # AWS Configuration
   OUTPUT_BUCKET=stock-screener-data-20250320
   OUTPUT_KEY_PREFIX=screening_results
   PORTFOLIO_TABLE=Portfolio
   PARAMETERS_TABLE=Parameters
   EC2_API_ENDPOINT=http://ec2-3-133-103-19.us-east-2.compute.amazonaws.com:5000

   # Google Sheets
   GOOGLE_SHEET_ID=your_google_sheet_id

   # Redis Configuration
   REDIS_ENDPOINT=your_redis_endpoint
   REDIS_PORT=6379
   CACHE_TTL=3600  # 1 hour in seconds
   ```

3. **IAM Roles and Permissions**:
   - StockScreenerRole: S3 read/write access
   - PortfolioOptimizerRole: S3 read, DynamoDB read/write, EC2 API access
   - MetaOptimizerRole: DynamoDB read/write, EC2 API access
   - EC2PyMCRole: S3 read, DynamoDB read