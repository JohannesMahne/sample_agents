# FinOps Analyst for Chargeback

AI agent for Elastic Cloud cost intelligence and chargeback analytics. This specialized agent helps finance teams, engineering managers, and FinOps professionals analyze Elasticsearch Service consumption, implement fair cost allocation, and distribute costs transparently through sophisticated chargeback mechanisms.

## Disclaimer

**This agent has been generated and refined for experimental purposes.** It is designed to help users progress with their AI adoption journey and to enhance the usability of the [Elasticsearch Chargeback](https://github.com/elastic/elasticsearch-chargeback) integration. The tools and agent configuration serve as a practical demonstration of how AI-powered cost analytics can be implemented using Kibana's Agent Builder. Users are encouraged to test, modify, and adapt this agent to their specific organizational needs and provide feedback to improve future iterations.

## Overview

The FinOps Analyst agent provides comprehensive cost intelligence for Elasticsearch Cloud deployments by:
- **Cost Analysis**: Track and analyze ECU (Elastic Compute Unit) consumption across all deployments
- **Chargeback Implementation**: Calculate fair cost allocation using weighted blended cost models with configurable currency conversion
- **Multi-dimensional Attribution**: Distribute costs across deployments, groups, datastreams, and tiers
- **Trend Analysis**: Monitor cost patterns over time with daily, monthly, and custom period breakdowns
- **Transparency**: Provide detailed cost breakdowns showing storage, querying, and indexing components

## Agent ID
`finops_analyst_6261a4`

## Key Capabilities

### 1. **Weighted Blended Cost Calculation**
The agent implements a sophisticated chargeback model that fairly distributes costs based on actual resource consumption:
- **Storage Weight**: 40 (reflects disk space usage)
- **Query Weight**: 20 (reflects search and aggregation activity)
- **Indexing Weight**: 20 (reflects document ingestion activity)
- **Tier-Aware**: Hot/content tiers use all three weights; cold/frozen tiers use storage + query only

### 2. **ECU to Currency Conversion**
All costs are calculated in ECU (Elastic Compute Units) and converted to currency using the configured rate. The rate is configurable and can vary by time period. Use the **chargeback_config_lookup** tool to retrieve the current rate and currency unit.

### 3. **Multi-Level Cost Attribution**
- **Deployment Level**: Total costs per deployment with grouping capabilities
- **Tier Level**: Costs broken down by hot/content, warm, cold, and frozen tiers
- **Datastream Level**: Cost allocation to individual datastreams based on consumption
- **SKU Level**: Costs grouped by cloud service type (e.g., Enterprise_azure)

## Data Sources

The agent operates on 6 specialized Elasticsearch indices designed for FinOps analytics:

### 1. **billing_cluster_cost_lookup**
- **Purpose**: Track actual ECU consumption and costs per cluster
- **Configuration**: 1 shard, lookup mode
- **Key Fields**: 
  - `composite_key` (keyword) - Join key format: "YYYY-MM-DD_deployment_id"
  - `deployment_id` (keyword) - Unique deployment identifier
  - `deployment_name` (keyword) - Human-readable deployment name
  - `deployment_group` (keyword) - Logical grouping (e.g., "unit_2", "production")
  - `cost_type` (keyword) - Cost category (e.g., "Enterprise_azure")
  - `sku` (keyword) - Cloud service type
  - `total_ecu` (numeric) - Total ECU consumption for the period
  - `@timestamp` (date) - Billing timestamp

### 2. **chargeback_conf_lookup**
- **Purpose**: Define cost allocation rules and ECU conversion rates
- **Configuration**: 1 shard, lookup mode
- **Key Fields**:
  - `config_join_key` (keyword) - Configuration identifier
  - `conf_ecu_rate` (numeric) - ECU to currency conversion rate
  - `conf_ecu_rate_unit` (keyword) - Currency unit (query the index for current value)
  - `conf_storage_weight` (numeric) - Weight for storage costs (default: 40)
  - `conf_query_weight` (numeric) - Weight for query costs (default: 20)
  - `conf_indexing_weight` (numeric) - Weight for indexing costs (default: 20)
  - `conf_start_date` (date) - Configuration validity start
  - `conf_end_date` (date) - Configuration validity end
- **Note**: Joined using date predicates: `@timestamp >= conf_start_date AND @timestamp <= conf_end_date`

### 3. **cluster_deployment_contribution_lookup**
- **Purpose**: Track total deployment-level consumption metrics
- **Configuration**: 1 shard, lookup mode
- **Key Fields**:
  - `composite_key` (keyword) - Matches billing data
  - `cluster_name` (keyword) - Deployment ID
  - `deployment_sum_store_size` (numeric) - Total storage in bytes
  - `deployment_sum_data_set_store_size` (numeric) - Dataset storage in bytes
  - `deployment_sum_query_time` (numeric) - Total query time in milliseconds
  - `deployment_sum_indexing_time` (numeric) - Total indexing time in milliseconds
  - `@timestamp` (date) - Measurement timestamp

### 4. **cluster_datastream_contribution_lookup**
- **Purpose**: Track consumption metrics per datastream
- **Configuration**: 1 shard, lookup mode
- **Key Fields**:
  - `composite_key` (keyword) - Matches billing data
  - `cluster_name` (keyword) - Deployment ID
  - `datastream` (keyword) - Datastream name
  - `datastream_sum_store_size` (numeric) - Datastream storage in bytes
  - `datastream_sum_data_set_store_size` (numeric) - Dataset storage in bytes
  - `datastream_sum_query_time` (numeric) - Query time in milliseconds
  - `datastream_sum_indexing_time` (numeric) - Indexing time in milliseconds
  - `@timestamp` (date) - Measurement timestamp

### 5. **cluster_tier_contribution_lookup**
- **Purpose**: Track consumption metrics by tier (hot/content, warm, cold, frozen)
- **Configuration**: 1 shard, lookup mode
- **Key Fields**:
  - `composite_key` (keyword) - Matches billing data
  - `cluster_name` (keyword) - Deployment ID
  - `tier` (keyword) - Storage tier name
  - `tier_sum_store_size` (numeric) - Tier storage in bytes
  - `tier_sum_data_set_store_size` (numeric) - Dataset storage in bytes
  - `tier_sum_query_time` (numeric) - Query time in milliseconds
  - `tier_sum_indexing_time` (numeric) - Indexing time in milliseconds
  - `@timestamp` (date) - Measurement timestamp

### 6. **cluster_tier_and_datastream_contribution_lookup**
- **Purpose**: Track consumption metrics by tier and datastream combination
- **Configuration**: 1 shard, lookup mode
- **Key Fields**:
  - `composite_key` (keyword) - Matches billing data
  - `cluster_name` (keyword) - Deployment ID
  - `datastream` (keyword) - Datastream name
  - `tier` (keyword) - Storage tier name
  - `tier_and_datastream_sum_store_size` (numeric) - Combined storage in bytes
  - `tier_and_datastream_sum_data_set_store_size` (numeric) - Dataset storage in bytes
  - `tier_and_datastream_sum_query_time` (numeric) - Query time in milliseconds
  - `tier_and_datastream_sum_indexing_time` (numeric) - Indexing time in milliseconds
  - `@timestamp` (date) - Measurement timestamp

## Tools (16 Custom + 6 Platform)

### Cost Lookup & Configuration
1. **cluster_cost_details** - Get detailed cost breakdown for a specific deployment by ID
   - Parameters: `deployment_id` (keyword)
   - Returns: ECU consumption, currency-converted costs, deployment details
   
2. **chargeback_config_lookup** - Retrieve current chargeback configuration
   - Parameters: None
   - Returns: ECU rate, weights, date ranges

### Cost Analytics & Rankings
3. **top_cost_clusters** - Identify top 20 clusters by total cost
   - Parameters: None
   - Returns: Ranked list with ECU and currency-converted costs by deployment

4. **cost_by_sku** - Analyze total costs grouped by SKU (cloud service type)
   - Parameters: None
   - Returns: Cost totals per SKU with deployment counts

5. **cost_type_comparison** - Compare costs across different cost types
   - Parameters: None
   - Returns: Cost breakdown by type (e.g., Enterprise_azure)

### Time-Series Analysis
6. **cost_trends_by_deployment** - Analyze daily cost trends for a specific deployment
   - Parameters: `deployment_id` (keyword)
   - Returns: Daily ECU and currency-converted costs over time

7. **daily_cost_trends** - Aggregate daily costs across all deployments
   - Parameters: None
   - Returns: Overall daily cost trends with deployment counts

8. **daily_tier_cost_trends** - Track daily cost trends by tier
   - Parameters: None
   - Returns: Time-series data showing costs per tier per day

### Deployment Group Analysis
9. **deployment_group_costs** - Total costs by deployment group
   - Parameters: None
   - Returns: Aggregated costs per group with deployment counts

### Advanced Chargeback & Cost Allocation
10. **blended_cost_by_tier** - Calculate weighted blended costs by tier
    - Parameters: None
    - Returns: Fair cost allocation per tier using weighted blending

11. **datastream_cost_allocation** - Calculate costs for a specific datastream
    - Parameters: `datastream` (keyword)
    - Returns: Datastream costs across all deployments using blended model

12. **top_datastream_costs** - Identify top 30 most expensive datastreams
    - Parameters: None
    - Returns: Ranked datastreams with blended costs and storage metrics

13. **tier_cost_breakdown_by_type** - Break down costs by component (indexing/querying/storage)
    - Parameters: None
    - Returns: Separate costs for each activity type per tier

14. **datastream_percentage_contrib** - Calculate percentage contribution per datastream
    - Parameters: None
    - Returns: Datastreams ranked by % of total costs

### Tier & Efficiency Analysis
15. **tier_efficiency_analysis** - Analyze cost per GB across tiers
    - Parameters: None
    - Returns: Cost efficiency metrics (cost per GB in configured currency) by tier and deployment

16. **datastream_tier_breakdown** - Analyze storage and activity by datastream and tier
    - Parameters: `cluster_name` (keyword)
    - Returns: Detailed breakdown of storage, queries, and indexing per datastream/tier

### Platform Tools (Available for Ad-Hoc Analysis)
- **platform.core.search** - Search across indices with custom filters
- **platform.core.get_document_by_id** - Retrieve specific documents
- **platform.core.execute_esql** - Run custom ES|QL queries
- **platform.core.get_index_mapping** - Inspect index structures
- **platform.core.list_indices** - List available indices
- **platform.core.generate_esql** - Generate ES|QL from natural language

## Query Examples

### Basic Cost Analysis
```
"What are the total costs for deployment 3cc5a925728f480abee1f5112dd184bf?"
→ Uses: cluster_cost_details

"Which are our top 10 most expensive deployments?"
→ Uses: top_cost_clusters
```

### Chargeback & Cost Allocation
```
"Calculate blended costs by tier for all deployments"
→ Uses: blended_cost_by_tier

"What is the cost allocation for datastream 'logs-system-prod'?"
→ Uses: datastream_cost_allocation

"Which datastreams are driving the most costs?"
→ Uses: top_datastream_costs
```

### Trend Analysis
```
"Show me the daily cost trend for the past month"
→ Uses: daily_cost_trends

"What are the cost trends by tier over time?"
→ Uses: daily_tier_cost_trends
```

### Group & SKU Analysis
```
"What are the total costs for deployment group 'unit_2'?"
→ Uses: deployment_group_costs

"Break down costs by cloud service type"
→ Uses: cost_by_sku
```

### Efficiency & Optimization Insights
```
"What is our cost efficiency per GB by tier?"
→ Uses: tier_efficiency_analysis

"Show me detailed tier and datastream breakdown for deployment X"
→ Uses: datastream_tier_breakdown
```

## How Blended Cost Calculation Works

The agent uses a sophisticated weighted blending model to fairly distribute costs:

### Step 1: Calculate Activity Ratios
For each datastream or tier, calculate its share of total deployment activity:
- **Storage Ratio** = `datastream_storage / deployment_total_storage`
- **Query Ratio** = `datastream_query_time / deployment_total_query_time`
- **Indexing Ratio** = `datastream_indexing_time / deployment_total_indexing_time`

### Step 2: Apply Ratios to Total ECU
Multiply each ratio by total deployment ECU to get activity-based ECU allocation:
- **Storage ECU** = `Storage Ratio × Total ECU`
- **Query ECU** = `Query Ratio × Total ECU`
- **Indexing ECU** = `Indexing Ratio × Total ECU`

### Step 3: Apply Weights
Apply configured weights to each activity:
- **Weighted Storage** = `Storage ECU × storage_weight (40)`
- **Weighted Query** = `Query ECU × query_weight (20)`
- **Weighted Indexing** = `Indexing ECU × indexing_weight (20)`

### Step 4: Calculate Blended ECU
Sum weighted activities and normalize:
- **Blended ECU** = `(Weighted Storage + Weighted Query + Weighted Indexing) / (40 + 20 + 20)`

### Step 5: Convert to Currency
Apply the configured ECU rate to get final cost:
- **Final Cost** = `Blended ECU × Configured ECU Rate`

### Tier-Specific Rules
- **Hot/Content Tier**: Uses all three weights (storage + query + indexing)
- **Cold/Frozen Tiers**: Uses only storage + query weights (indexing typically zero)

## Technical Implementation Details

### Join Strategy
The agent uses ES|QL's `LOOKUP JOIN` with predicate-based joins:
- **Composite Key Join**: Links billing with contribution data via `composite_key`
- **Date Predicate Join**: Links billing with configuration via timestamp range matching
- **Multiple Lookups**: Chains up to 4 lookups for comprehensive cost attribution

### Numeric Precision
All division operations use `TO_DOUBLE()` to ensure visualization compatibility and prevent integer truncation.

### Performance Optimization
- All lookup indices use 1 shard for optimal join performance
- Composite keys enable efficient matching across time periods
- Aggregations use `STATS` for server-side calculation efficiency

## Requirements

### System Requirements
- **Chargeback Integration**: Version 0.2.8 or higher
- **ESS Billing Integration**: Version 1.6.1 or higher
- **Kibana**: Instance with AI Assistant and Agent Builder enabled

## Deployment

### Prerequisites
1. Elasticsearch cluster with the 6 required indices populated
2. Kibana instance with AI Assistant and Agent Builder enabled
3. Configured ELASTIC_URL and ELASTIC_API_KEY in environment

### Deploy the Agent

From the agent directory:
```bash
cd agents/finops_analyst_6261a4
npm run agent:deploy
```

This will:
1. Upload all 16 tool definitions to Kibana
2. Deploy the agent configuration
3. Make the agent available in AI Assistant

### How Deployment Works

The deployment process uses the [Kibana Agent Builder API](https://www.elastic.co/docs/explore-analyze/ai-features/agent-builder/kibana-api) to upload tools and agent configurations. The process follows a simple pattern:

1. **Upload Tools**: Each tool definition (JSON file) is posted to the Agent Builder API
2. **Upload Agent**: The agent configuration references the uploaded tools by ID

**Example: Uploading a Tool**

A tool like `cluster_cost_details` is uploaded via API call:

```bash
POST ${KIBANA_URL}/api/agent_builder/tools
Authorization: ApiKey ${API_KEY}
Content-Type: application/json

{
  "id": "finops_analyst_6261a4_tool_1_cluster_cost_details",
  "name": "cluster_cost_details",
  "description": "Get detailed cost breakdown for a specific deployment",
  "type": "esql",
  "configuration": {
    "query": "FROM billing_cluster_cost_lookup | WHERE deployment_id == ?deployment_id | ...",
    "params": {
      "deployment_id": { "type": "keyword", "description": "Deployment ID" }
    }
  }
}
```

**Example: Uploading the Agent**

After all tools are uploaded, the agent configuration is deployed:

```bash
POST ${KIBANA_URL}/api/agent_builder/agents
Authorization: ApiKey ${API_KEY}
Content-Type: application/json

{
  "id": "finops_analyst_6261a4",
  "name": "FinOps Analyst for Chargeback",
  "description": "AI agent for Elastic Cloud cost intelligence and chargeback analytics",
  "configuration": {
    "instructions": "You are a FinOps analyst specializing in cost allocation...",
    "tools": [
      {
        "tool_ids": [
          "finops_analyst_6261a4_tool_1_cluster_cost_details",
          "finops_analyst_6261a4_tool_2_top_cost_clusters",
          ...
        ]
      }
    ]
  }
}
```

The agent is now available in the AI Assistant interface and can be selected by users.

### Verify Deployment
1. Open Kibana and search for Agents
2. Select "FinOps Analyst for Chargeback" from agent list
3. Try a test query: "What are our top 5 most expensive deployments?"

## Use Cases

### 1. **Monthly Cost Reports**
Generate comprehensive monthly reports showing:
- Total costs by deployment group
- Top cost-driving deployments and datastreams
- Cost trends over the month
- Tier efficiency metrics

### 2. **Chargeback Implementation**
Implement fair cost allocation:
- Calculate datastream-level costs for internal customers
- Provide transparent breakdown of storage, query, and indexing costs
- Support multiple rate cards for different customer tiers

### 3. **Cost Optimization**
Identify optimization opportunities:
- Find inefficient tier usage (high cost per GB)
- Identify unused or underutilized deployments
- Track cost trends to forecast future spending

### 4. **Financial Planning**
Support budget planning:
- Analyze historical cost trends
- Project future costs based on growth patterns
- Compare costs across deployment groups and SKUs

## Troubleshooting

### No data returned
- Verify indices exist: Use `platform.core.list_indices`
- Check date ranges: Ensure `@timestamp` fields align with chargeback configuration dates

## Next Steps

1. **Review Generated Tools**: Examine the 16 tools in `tools/` directory
2. **Test Queries**: Try queries in Kibana Dev Console
3. **Deploy and Test**: Deploy the agent and test with real queries to validate functionality
4. **Refine Agent Prompt**: If needed, adjust the prompt of the agent configuration based on user interactions
5. **Tweak Tools**: Modify ES|QL queries in tools to better match your data patterns and use cases
6. **Train Users**: Provide example queries to finance team and stakeholders
7. **Monitor Usage**: Track which queries are most valuable and which need improvement
8. **Give Feedback**: Share your experience and contribute improvements to the [Elasticsearch Chargeback](https://github.com/elastic/elasticsearch-chargeback) project
9. **Iterate**: Add custom tools for specific organizational needs
