# ETL vs ELT - Complete Guide

> **MUST REMEMBER**: ETL (Extract-Transform-Load) transforms data BEFORE loading into destination (traditional). ELT (Extract-Load-Transform) loads raw data FIRST, transforms IN the destination (modern). ETL: better for sensitive data, complex transformations, limited destination compute. ELT: better for cloud data warehouses (Snowflake, BigQuery), large volumes, iterative analysis. Data warehouses = structured analytics (star schema). Data lakes = raw storage (any format).

---

## How to Explain Like a Senior Developer

"ETL and ELT differ in WHERE transformation happens. Traditional ETL transforms data in a separate processing layer before loading into the warehouse - this made sense when warehouses had limited compute. Modern ELT loads raw data into cloud warehouses first, then transforms using the warehouse's powerful compute (Snowflake, BigQuery, Redshift). ELT is faster for large volumes, more flexible (transform later as needs change), and leverages cheap cloud storage. But ETL is still better when you need to mask sensitive data before it reaches the warehouse, or when the destination can't handle transformation workloads. Data lakes store everything raw (schema-on-read), warehouses store structured data (schema-on-write). Many companies use both: land in lake, load to warehouse, transform there."

---

## Core Implementation

### ETL Pipeline (Transform Before Load)

```typescript
// etl/traditional-etl.ts

import { Pool } from 'pg';
import { S3Client, GetObjectCommand, PutObjectCommand } from '@aws-sdk/client-s3';
import { Readable } from 'stream';
import { createGunzip } from 'zlib';
import * as csv from 'csv-parse';

interface RawOrder {
  order_id: string;
  customer_email: string;
  customer_ssn: string;  // Sensitive!
  amount: string;
  currency: string;
  status: string;
  created_at: string;
}

interface TransformedOrder {
  orderId: string;
  customerEmailHash: string;  // Anonymized
  // SSN removed - never reaches warehouse
  amountUsd: number;  // Converted
  status: string;
  createdAt: Date;
  createdDate: string;  // Partition key
}

/**
 * Traditional ETL: Transform BEFORE loading
 * Use when: sensitive data, complex transformations, limited destination compute
 */
class ETLPipeline {
  private s3: S3Client;
  private sourceDb: Pool;
  private warehouseDb: Pool;
  
  constructor() {
    this.s3 = new S3Client({ region: 'us-east-1' });
    this.sourceDb = new Pool({ connectionString: process.env.SOURCE_DB_URL });
    this.warehouseDb = new Pool({ connectionString: process.env.WAREHOUSE_DB_URL });
  }
  
  /**
   * Extract from source
   */
  async extract(date: string): Promise<RawOrder[]> {
    console.log(`Extracting data for ${date}`);
    
    const result = await this.sourceDb.query<RawOrder>(`
      SELECT 
        order_id,
        customer_email,
        customer_ssn,
        amount,
        currency,
        status,
        created_at
      FROM orders
      WHERE DATE(created_at) = $1
    `, [date]);
    
    console.log(`Extracted ${result.rows.length} records`);
    return result.rows;
  }
  
  /**
   * Transform data BEFORE loading
   * - Remove sensitive data (SSN)
   * - Hash PII (email)
   * - Convert currencies
   * - Add partition keys
   */
  async transform(rawOrders: RawOrder[]): Promise<TransformedOrder[]> {
    console.log('Transforming data...');
    
    const exchangeRates = await this.getExchangeRates();
    
    return rawOrders.map(raw => {
      // Remove SSN - never reaches warehouse
      // Hash email for privacy
      const emailHash = this.hashPII(raw.customer_email);
      
      // Convert to USD
      const amountUsd = this.convertToUsd(
        parseFloat(raw.amount),
        raw.currency,
        exchangeRates
      );
      
      const createdAt = new Date(raw.created_at);
      
      return {
        orderId: raw.order_id,
        customerEmailHash: emailHash,
        amountUsd,
        status: raw.status.toUpperCase(),
        createdAt,
        createdDate: createdAt.toISOString().split('T')[0],
      };
    });
  }
  
  /**
   * Load transformed data to warehouse
   */
  async load(orders: TransformedOrder[]): Promise<void> {
    console.log(`Loading ${orders.length} records to warehouse`);
    
    const client = await this.warehouseDb.connect();
    
    try {
      await client.query('BEGIN');
      
      // Delete existing data for idempotency
      const date = orders[0]?.createdDate;
      if (date) {
        await client.query(
          'DELETE FROM fact_orders WHERE created_date = $1',
          [date]
        );
      }
      
      // Batch insert
      const batchSize = 1000;
      for (let i = 0; i < orders.length; i += batchSize) {
        const batch = orders.slice(i, i + batchSize);
        await this.insertBatch(client, batch);
      }
      
      await client.query('COMMIT');
      console.log('Load completed');
      
    } catch (error) {
      await client.query('ROLLBACK');
      throw error;
    } finally {
      client.release();
    }
  }
  
  private async insertBatch(client: any, orders: TransformedOrder[]): Promise<void> {
    const values: any[] = [];
    const placeholders: string[] = [];
    
    orders.forEach((order, idx) => {
      const offset = idx * 5;
      placeholders.push(`($${offset + 1}, $${offset + 2}, $${offset + 3}, $${offset + 4}, $${offset + 5})`);
      values.push(
        order.orderId,
        order.customerEmailHash,
        order.amountUsd,
        order.status,
        order.createdDate
      );
    });
    
    await client.query(`
      INSERT INTO fact_orders (order_id, customer_hash, amount_usd, status, created_date)
      VALUES ${placeholders.join(', ')}
    `, values);
  }
  
  private hashPII(value: string): string {
    const crypto = require('crypto');
    return crypto.createHash('sha256').update(value).digest('hex').substring(0, 16);
  }
  
  private async getExchangeRates(): Promise<Record<string, number>> {
    // In production, fetch from API or database
    return { USD: 1, EUR: 1.1, GBP: 1.25, JPY: 0.0067 };
  }
  
  private convertToUsd(amount: number, currency: string, rates: Record<string, number>): number {
    const rate = rates[currency] || 1;
    return Math.round(amount * rate * 100) / 100;
  }
  
  /**
   * Run full ETL pipeline
   */
  async run(date: string): Promise<void> {
    const startTime = Date.now();
    
    try {
      // Extract
      const rawData = await this.extract(date);
      
      // Transform
      const transformedData = await this.transform(rawData);
      
      // Load
      await this.load(transformedData);
      
      const duration = (Date.now() - startTime) / 1000;
      console.log(`ETL completed in ${duration}s`);
      
    } catch (error) {
      console.error('ETL failed:', error);
      throw error;
    }
  }
}
```

### ELT Pipeline (Load First, Transform in Warehouse)

```typescript
// elt/modern-elt.ts

import { BigQuery } from '@google-cloud/bigquery';
import { Storage } from '@google-cloud/storage';

/**
 * Modern ELT: Load raw, transform in warehouse
 * Use when: cloud warehouse, large volumes, flexible analysis needs
 */
class ELTPipeline {
  private bigquery: BigQuery;
  private storage: Storage;
  private datasetId: string = 'analytics';
  
  constructor() {
    this.bigquery = new BigQuery();
    this.storage = new Storage();
  }
  
  /**
   * Extract: Stream raw data to cloud storage
   */
  async extractToLake(
    sourceQuery: string,
    bucketPath: string
  ): Promise<string> {
    console.log('Extracting to data lake...');
    
    // Export query results to GCS (raw format)
    const [job] = await this.bigquery.createQueryJob({
      query: sourceQuery,
      destination: this.bigquery.dataset('staging').table('raw_export'),
    });
    
    await job.getQueryResults();
    
    // Export to GCS as Parquet
    const destinationUri = `gs://${bucketPath}/raw_*.parquet`;
    const [exportJob] = await this.bigquery
      .dataset('staging')
      .table('raw_export')
      .extract(destinationUri, { format: 'PARQUET' });
    
    await exportJob;
    console.log(`Extracted to ${destinationUri}`);
    
    return destinationUri;
  }
  
  /**
   * Load: Load raw data into warehouse
   */
  async loadRaw(
    sourcePath: string,
    tableName: string
  ): Promise<void> {
    console.log(`Loading raw data to ${tableName}...`);
    
    const table = this.bigquery.dataset(this.datasetId).table(tableName);
    
    // Load from GCS - no transformation yet
    const [job] = await table.load(sourcePath, {
      sourceFormat: 'PARQUET',
      writeDisposition: 'WRITE_APPEND',
      schemaUpdateOptions: ['ALLOW_FIELD_ADDITION'],
    });
    
    await job;
    console.log('Raw data loaded');
  }
  
  /**
   * Transform: SQL transformations IN the warehouse
   */
  async transform(): Promise<void> {
    console.log('Running transformations in warehouse...');
    
    // All transformations happen in BigQuery using SQL
    const transformations = [
      this.createStagingView(),
      this.transformToFact(),
      this.updateDimensions(),
      this.createAggregates(),
    ];
    
    for (const query of transformations) {
      await this.runQuery(query);
    }
    
    console.log('Transformations completed');
  }
  
  private createStagingView(): string {
    // Clean and validate raw data
    return `
      CREATE OR REPLACE VIEW ${this.datasetId}.stg_orders AS
      SELECT
        order_id,
        -- Hash PII in warehouse
        TO_HEX(SHA256(customer_email)) AS customer_hash,
        -- Remove SSN from view
        CAST(amount AS NUMERIC) AS amount,
        UPPER(currency) AS currency,
        UPPER(status) AS status,
        PARSE_TIMESTAMP('%Y-%m-%d %H:%M:%S', created_at) AS created_at,
        DATE(PARSE_TIMESTAMP('%Y-%m-%d %H:%M:%S', created_at)) AS created_date
      FROM ${this.datasetId}.raw_orders
      WHERE order_id IS NOT NULL
        AND amount IS NOT NULL
        AND SAFE_CAST(amount AS NUMERIC) > 0
    `;
  }
  
  private transformToFact(): string {
    // Create fact table with business logic
    return `
      CREATE OR REPLACE TABLE ${this.datasetId}.fact_orders
      PARTITION BY created_date
      CLUSTER BY customer_hash
      AS
      SELECT
        o.order_id,
        o.customer_hash,
        -- Currency conversion in warehouse
        CASE o.currency
          WHEN 'USD' THEN o.amount
          WHEN 'EUR' THEN o.amount * 1.1
          WHEN 'GBP' THEN o.amount * 1.25
          ELSE o.amount
        END AS amount_usd,
        o.status,
        o.created_at,
        o.created_date,
        -- Join with dimensions
        c.segment AS customer_segment,
        c.country AS customer_country
      FROM ${this.datasetId}.stg_orders o
      LEFT JOIN ${this.datasetId}.dim_customers c
        ON o.customer_hash = c.customer_hash
    `;
  }
  
  private updateDimensions(): string {
    // SCD Type 2 for dimensions
    return `
      MERGE ${this.datasetId}.dim_customers AS target
      USING (
        SELECT DISTINCT
          customer_hash,
          -- Derive attributes from orders
          FIRST_VALUE(country) OVER (
            PARTITION BY customer_hash 
            ORDER BY created_at DESC
          ) AS country
        FROM ${this.datasetId}.stg_orders
      ) AS source
      ON target.customer_hash = source.customer_hash
      WHEN MATCHED AND target.country != source.country THEN
        UPDATE SET country = source.country, updated_at = CURRENT_TIMESTAMP()
      WHEN NOT MATCHED THEN
        INSERT (customer_hash, country, created_at)
        VALUES (source.customer_hash, source.country, CURRENT_TIMESTAMP())
    `;
  }
  
  private createAggregates(): string {
    // Pre-computed aggregates for dashboards
    return `
      CREATE OR REPLACE TABLE ${this.datasetId}.agg_daily_sales AS
      SELECT
        created_date,
        customer_segment,
        customer_country,
        COUNT(*) AS total_orders,
        SUM(amount_usd) AS total_revenue,
        AVG(amount_usd) AS avg_order_value,
        COUNT(DISTINCT customer_hash) AS unique_customers
      FROM ${this.datasetId}.fact_orders
      GROUP BY 1, 2, 3
    `;
  }
  
  private async runQuery(query: string): Promise<void> {
    const [job] = await this.bigquery.createQueryJob({ query });
    await job.getQueryResults();
  }
  
  /**
   * Run full ELT pipeline
   */
  async run(sourceQuery: string): Promise<void> {
    const startTime = Date.now();
    
    try {
      // Extract to lake
      const lakePath = await this.extractToLake(
        sourceQuery,
        'my-data-lake/raw/orders'
      );
      
      // Load raw to warehouse
      await this.loadRaw(lakePath, 'raw_orders');
      
      // Transform in warehouse
      await this.transform();
      
      const duration = (Date.now() - startTime) / 1000;
      console.log(`ELT completed in ${duration}s`);
      
    } catch (error) {
      console.error('ELT failed:', error);
      throw error;
    }
  }
}
```

### Data Lake vs Data Warehouse

```typescript
// data-architecture/lake-vs-warehouse.ts

/**
 * Data Lake: Raw storage, schema-on-read
 * - Store anything (JSON, Parquet, images, logs)
 * - Schema applied when reading
 * - Cheap storage (S3, GCS, ADLS)
 * - Good for: exploration, ML, raw archives
 */

interface DataLakeConfig {
  storage: 'S3' | 'GCS' | 'ADLS';
  zones: {
    raw: string;      // Landing zone - unchanged data
    cleansed: string; // Validated, deduplicated
    curated: string;  // Business-ready datasets
  };
  catalog: 'Glue' | 'Hive' | 'Unity';
}

/**
 * Data Warehouse: Structured analytics, schema-on-write
 * - Structured data (tables, columns)
 * - Schema enforced on write
 * - Optimized for queries (columnar, compressed)
 * - Good for: BI, reporting, dashboards
 */

interface DataWarehouseConfig {
  engine: 'Snowflake' | 'BigQuery' | 'Redshift' | 'Databricks';
  schemas: {
    staging: string;  // Landing area
    marts: string;    // Business domains
    analytics: string; // Aggregates, reports
  };
  modeling: 'Star' | 'Snowflake' | 'DataVault';
}

/**
 * Modern Data Stack: Lake + Warehouse (Lakehouse)
 */
class LakehouseArchitecture {
  /**
   * Medallion Architecture (Bronze/Silver/Gold)
   */
  static layers = {
    bronze: {
      description: 'Raw, unchanged data from sources',
      format: 'Delta Lake / Iceberg',
      retention: '7 years',
      access: 'Data Engineers',
    },
    silver: {
      description: 'Cleansed, conformed, deduplicated',
      format: 'Delta Lake / Iceberg',
      retention: '3 years',
      access: 'Data Engineers, Data Scientists',
    },
    gold: {
      description: 'Business-level aggregates, features',
      format: 'Delta Lake / Iceberg',
      retention: '1 year',
      access: 'Analysts, BI Tools, Applications',
    },
  };
  
  /**
   * Choose ETL vs ELT based on requirements
   */
  static chooseApproach(requirements: {
    sensitiveData: boolean;
    volumeGB: number;
    transformComplexity: 'simple' | 'complex';
    destinationCompute: 'limited' | 'powerful';
    latencyRequirement: 'batch' | 'near-realtime';
  }): 'ETL' | 'ELT' {
    // ETL if sensitive data must be masked before warehouse
    if (requirements.sensitiveData) {
      return 'ETL';
    }
    
    // ETL if destination can't handle transform workload
    if (requirements.destinationCompute === 'limited') {
      return 'ETL';
    }
    
    // ELT for large volumes with powerful warehouse
    if (requirements.volumeGB > 100 && requirements.destinationCompute === 'powerful') {
      return 'ELT';
    }
    
    // ELT for flexibility and iteration
    return 'ELT';
  }
}
```

---

## Real-World Scenarios

### Scenario 1: dbt for ELT Transformations

```sql
-- dbt/models/staging/stg_orders.sql

-- dbt model: staging layer transformation
-- Run with: dbt run --select stg_orders

{{
  config(
    materialized='view',
    schema='staging'
  )
}}

WITH source AS (
    SELECT * FROM {{ source('raw', 'orders') }}
),

cleaned AS (
    SELECT
        order_id,
        {{ dbt_utils.generate_surrogate_key(['customer_id']) }} AS customer_key,
        
        -- Clean amount
        CASE 
            WHEN amount ~ '^[0-9]+\.?[0-9]*$' 
            THEN CAST(amount AS NUMERIC)
            ELSE NULL 
        END AS amount,
        
        -- Standardize status
        UPPER(TRIM(status)) AS status,
        
        -- Parse timestamp
        CAST(created_at AS TIMESTAMP) AS created_at,
        
        -- Derived fields
        DATE(CAST(created_at AS TIMESTAMP)) AS created_date,
        
        -- Metadata
        CURRENT_TIMESTAMP AS _loaded_at
        
    FROM source
    WHERE order_id IS NOT NULL
)

SELECT * FROM cleaned
```

```sql
-- dbt/models/marts/fct_orders.sql

-- dbt model: fact table
{{
  config(
    materialized='incremental',
    unique_key='order_id',
    schema='marts',
    partition_by={
      "field": "created_date",
      "data_type": "date"
    }
  )
}}

WITH orders AS (
    SELECT * FROM {{ ref('stg_orders') }}
    {% if is_incremental() %}
    WHERE created_at > (SELECT MAX(created_at) FROM {{ this }})
    {% endif %}
),

customers AS (
    SELECT * FROM {{ ref('dim_customers') }}
),

final AS (
    SELECT
        o.order_id,
        o.customer_key,
        c.customer_segment,
        c.customer_country,
        
        -- Convert to USD using macro
        {{ convert_currency('o.amount', 'o.currency', 'USD') }} AS amount_usd,
        
        o.status,
        o.created_at,
        o.created_date,
        o._loaded_at
        
    FROM orders o
    LEFT JOIN customers c ON o.customer_key = c.customer_key
)

SELECT * FROM final
```

### Scenario 2: Hybrid ETL/ELT

```typescript
// hybrid/hybrid-pipeline.ts

/**
 * Hybrid approach: ETL for sensitive data, ELT for the rest
 */
class HybridPipeline {
  /**
   * Phase 1: ETL for sensitive data (runs on secure compute)
   */
  async etlSensitiveData(date: string): Promise<void> {
    // Extract from source
    const rawData = await this.extractFromSource(date);
    
    // Transform: Remove/mask sensitive fields
    const maskedData = rawData.map(record => ({
      ...record,
      // Remove sensitive fields
      ssn: undefined,
      credit_card: undefined,
      // Mask PII
      email: this.hashEmail(record.email),
      phone: this.maskPhone(record.phone),
      // Keep business data as-is
      amount: record.amount,
      product_id: record.product_id,
    }));
    
    // Load to staging (sensitive data never reaches warehouse raw)
    await this.loadToStaging(maskedData, 'stg_orders_masked');
  }
  
  /**
   * Phase 2: ELT for everything else (transforms in warehouse)
   */
  async eltTransformations(): Promise<void> {
    // All further transformations in warehouse SQL
    const transformations = [
      // Join with dimensions
      `CREATE OR REPLACE TABLE analytics.fct_orders AS
       SELECT m.*, d.category, d.brand
       FROM staging.stg_orders_masked m
       LEFT JOIN dimensions.dim_products d ON m.product_id = d.product_id`,
      
      // Create aggregates
      `CREATE OR REPLACE TABLE analytics.agg_daily_sales AS
       SELECT DATE(created_at) as date, SUM(amount) as revenue
       FROM analytics.fct_orders
       GROUP BY 1`,
    ];
    
    for (const sql of transformations) {
      await this.runWarehouseQuery(sql);
    }
  }
  
  private hashEmail(email: string): string {
    const crypto = require('crypto');
    return crypto.createHash('sha256').update(email).digest('hex').substring(0, 16);
  }
  
  private maskPhone(phone: string): string {
    return phone.replace(/\d(?=\d{4})/g, '*');
  }
  
  private async extractFromSource(date: string): Promise<any[]> {
    // Implementation
    return [];
  }
  
  private async loadToStaging(data: any[], table: string): Promise<void> {
    // Implementation
  }
  
  private async runWarehouseQuery(sql: string): Promise<void> {
    // Implementation
  }
}
```

---

## Common Pitfalls

### 1. Loading Sensitive Data Raw

```typescript
// ❌ BAD: Loading PII to warehouse raw
await loadRawToWarehouse({
  email: 'user@example.com',  // PII in warehouse!
  ssn: '123-45-6789',         // Very bad!
  amount: 100,
});

// ✅ GOOD: ETL sensitive data before warehouse
const maskedData = {
  email_hash: hashPII(data.email),
  // SSN never loaded
  amount: data.amount,
};
await loadToWarehouse(maskedData);
```

### 2. Over-Transforming in ETL

```typescript
// ❌ BAD: Complex business logic in ETL code
const transformed = rawData.map(d => ({
  ...d,
  // Tight coupling to business rules
  customerTier: calculateComplexTier(d),
  riskScore: runMLModel(d),
  recommendations: generateRecommendations(d),
}));

// ✅ GOOD: Load raw, transform in warehouse
// ETL: Just clean and mask
const cleaned = rawData.map(d => ({
  ...d,
  email: hash(d.email),
}));
await loadRaw(cleaned);

// ELT: Business logic in SQL (easy to change)
await runSQL(`
  SELECT *, 
    calculate_tier(total_spent) as customer_tier
  FROM raw_customers
`);
```

### 3. Ignoring Schema Evolution

```typescript
// ❌ BAD: Rigid schema breaks on changes
const schema = z.object({
  orderId: z.string(),
  amount: z.number(),
  // New field added to source - pipeline breaks!
});

// ✅ GOOD: Handle schema evolution
const schema = z.object({
  orderId: z.string(),
  amount: z.number(),
}).passthrough();  // Allow extra fields

// Or use schema registry
await loadWithSchemaEvolution(data, {
  allowNewFields: true,
  allowFieldRemoval: false,
  allowTypeChanges: false,
});
```

---

## Interview Questions

### Q1: When would you choose ETL over ELT?

**A:** Choose **ETL** when: 1) Sensitive data must be masked/removed before reaching warehouse. 2) Destination has limited compute (can't handle transform workload). 3) Complex transformations need specialized tools (Spark, Python). 4) Regulatory compliance requires data processing in specific locations. Choose **ELT** when: powerful cloud warehouse (Snowflake, BigQuery), large volumes, need flexibility to re-transform later.

### Q2: What is the difference between a data lake and data warehouse?

**A:** **Data Lake**: Raw storage, schema-on-read, any format (JSON, Parquet, images), cheap storage (S3), good for exploration and ML. **Data Warehouse**: Structured storage, schema-on-write, optimized for queries (columnar), good for BI and reporting. Modern **Lakehouse** combines both: store raw in lake, query with warehouse-like performance (Delta Lake, Iceberg).

### Q3: Explain the medallion architecture.

**A:** Three-layer architecture for data lakes: **Bronze** (raw, unchanged from source), **Silver** (cleansed, conformed, deduplicated), **Gold** (business-level aggregates, feature store). Each layer increases quality and decreases volume. Enables: traceability (can always go back to raw), iterative refinement, different access levels per layer.

### Q4: How do you handle schema changes in pipelines?

**A:** 1) **Schema registry**: Central source of truth for schemas (Confluent Schema Registry). 2) **Schema evolution policies**: Define what's allowed (add columns OK, remove columns fail). 3) **Versioning**: Load to versioned tables, transform to latest schema. 4) **Backward/forward compatibility**: Design schemas that work with old and new data. 5) **Monitoring**: Alert on schema drift.

---

## Quick Reference Checklist

### ETL vs ELT Decision
- [ ] Sensitive data → ETL (mask before load)
- [ ] Large volume + cloud DW → ELT
- [ ] Complex transforms → ETL (Spark)
- [ ] Flexible analysis → ELT

### Data Architecture
- [ ] Define lake zones (raw/cleansed/curated)
- [ ] Choose modeling approach (star/snowflake)
- [ ] Plan for schema evolution
- [ ] Set up data catalog

### Pipeline Quality
- [ ] Implement idempotency
- [ ] Add data validation
- [ ] Handle late/duplicate data
- [ ] Monitor lineage

---

*Last updated: February 2026*

