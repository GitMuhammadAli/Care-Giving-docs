# Batch Processing - Complete Guide

> **MUST REMEMBER**: Batch processing handles large volumes of data at scheduled intervals (hourly/daily). Tools: Apache Spark (distributed, in-memory), Hadoop MapReduce (disk-based, older). Key concepts: partitioning, shuffling, fault tolerance, checkpointing. Use when: latency tolerance (minutes-hours), full dataset processing, complex transformations. Anti-pattern: forcing batch for real-time needs.

---

## How to Explain Like a Senior Developer

"Batch processing is processing data in chunks at scheduled times rather than as it arrives. Think of it like doing laundry - you wait until you have a full load rather than washing one item at a time. Apache Spark is the modern standard - it's distributed, in-memory, and 10-100x faster than old MapReduce. You define transformations (map, filter, join) that get distributed across a cluster. The key is understanding partitions - data is split across nodes, and operations like joins cause 'shuffles' where data moves between nodes (expensive!). Spark handles failures through lineage - it can recompute lost partitions. Use batch when you can tolerate delays (minutes to hours) and need to process full datasets. For sub-second latency, you need streaming instead."

---

## Core Implementation

### Apache Spark with PySpark

```python
# batch_processing/spark_pipeline.py

from pyspark.sql import SparkSession
from pyspark.sql.functions import (
    col, sum, avg, count, when, lit,
    to_date, year, month, dayofmonth,
    window, explode, array, struct
)
from pyspark.sql.types import (
    StructType, StructField, StringType, 
    IntegerType, DoubleType, TimestampType
)
from datetime import datetime, timedelta

# Initialize Spark Session
spark = SparkSession.builder \
    .appName("BatchProcessingPipeline") \
    .config("spark.sql.shuffle.partitions", "200") \
    .config("spark.executor.memory", "4g") \
    .config("spark.driver.memory", "2g") \
    .config("spark.sql.adaptive.enabled", "true") \
    .getOrCreate()

# Set log level
spark.sparkContext.setLogLevel("WARN")

# Define schema explicitly (better than inferSchema)
order_schema = StructType([
    StructField("order_id", StringType(), False),
    StructField("user_id", StringType(), False),
    StructField("product_id", StringType(), False),
    StructField("quantity", IntegerType(), False),
    StructField("price", DoubleType(), False),
    StructField("order_date", TimestampType(), False),
    StructField("status", StringType(), False)
])

class BatchPipeline:
    def __init__(self, spark_session):
        self.spark = spark_session
    
    def read_data(self, path: str, format: str = "parquet"):
        """Read data from various sources"""
        if format == "parquet":
            return self.spark.read.parquet(path)
        elif format == "csv":
            return self.spark.read \
                .option("header", "true") \
                .schema(order_schema) \
                .csv(path)
        elif format == "json":
            return self.spark.read.json(path)
        elif format == "delta":
            return self.spark.read.format("delta").load(path)
        else:
            raise ValueError(f"Unsupported format: {format}")
    
    def transform_orders(self, df):
        """Apply business transformations"""
        return df \
            .filter(col("status") == "completed") \
            .withColumn("total_amount", col("quantity") * col("price")) \
            .withColumn("order_year", year("order_date")) \
            .withColumn("order_month", month("order_date")) \
            .withColumn("order_day", dayofmonth("order_date"))
    
    def aggregate_daily_sales(self, df):
        """Aggregate sales by day"""
        return df \
            .groupBy("order_year", "order_month", "order_day") \
            .agg(
                count("order_id").alias("total_orders"),
                sum("total_amount").alias("total_revenue"),
                avg("total_amount").alias("avg_order_value"),
                count("user_id").alias("unique_customers")
            ) \
            .orderBy("order_year", "order_month", "order_day")
    
    def aggregate_by_product(self, df):
        """Aggregate sales by product"""
        return df \
            .groupBy("product_id") \
            .agg(
                sum("quantity").alias("total_quantity_sold"),
                sum("total_amount").alias("total_revenue"),
                count("order_id").alias("order_count"),
                avg("price").alias("avg_price")
            ) \
            .orderBy(col("total_revenue").desc())
    
    def write_output(self, df, path: str, format: str = "parquet", 
                     mode: str = "overwrite", partition_by: list = None):
        """Write output with optional partitioning"""
        writer = df.write.mode(mode)
        
        if partition_by:
            writer = writer.partitionBy(*partition_by)
        
        if format == "parquet":
            writer.parquet(path)
        elif format == "delta":
            writer.format("delta").save(path)
        elif format == "csv":
            writer.option("header", "true").csv(path)
    
    def run_pipeline(self, input_path: str, output_path: str):
        """Execute the full pipeline"""
        # Read
        raw_df = self.read_data(input_path)
        raw_df.cache()  # Cache for reuse
        
        print(f"Input records: {raw_df.count()}")
        
        # Transform
        transformed_df = self.transform_orders(raw_df)
        
        # Aggregate
        daily_sales = self.aggregate_daily_sales(transformed_df)
        product_sales = self.aggregate_by_product(transformed_df)
        
        # Write outputs
        self.write_output(
            daily_sales, 
            f"{output_path}/daily_sales",
            partition_by=["order_year", "order_month"]
        )
        
        self.write_output(
            product_sales,
            f"{output_path}/product_sales"
        )
        
        # Cleanup
        raw_df.unpersist()
        
        return {
            "daily_records": daily_sales.count(),
            "product_records": product_sales.count()
        }


# Advanced: Joins and Shuffles
class AdvancedBatchOperations:
    def __init__(self, spark_session):
        self.spark = spark_session
    
    def optimized_join(self, large_df, small_df, join_key: str):
        """
        Broadcast join for small tables (< 10MB default)
        Avoids expensive shuffle operations
        """
        from pyspark.sql.functions import broadcast
        
        return large_df.join(
            broadcast(small_df),  # Broadcast small table to all nodes
            on=join_key,
            how="left"
        )
    
    def handle_skewed_join(self, df1, df2, join_key: str, 
                           skewed_values: list):
        """
        Handle data skew in joins
        Separate processing for skewed keys
        """
        # Split skewed and non-skewed data
        df1_skewed = df1.filter(col(join_key).isin(skewed_values))
        df1_normal = df1.filter(~col(join_key).isin(skewed_values))
        
        df2_skewed = df2.filter(col(join_key).isin(skewed_values))
        df2_normal = df2.filter(~col(join_key).isin(skewed_values))
        
        # Join separately and union
        result_normal = df1_normal.join(df2_normal, on=join_key)
        
        # Broadcast skewed portion (assuming it's small after filtering)
        from pyspark.sql.functions import broadcast
        result_skewed = df1_skewed.join(broadcast(df2_skewed), on=join_key)
        
        return result_normal.union(result_skewed)
    
    def repartition_strategy(self, df, num_partitions: int = None,
                            partition_cols: list = None):
        """
        Repartition for optimal parallelism
        """
        if partition_cols:
            # Hash partitioning by columns
            return df.repartition(*partition_cols)
        elif num_partitions:
            # Round-robin partitioning
            return df.repartition(num_partitions)
        else:
            # Coalesce to reduce partitions (no shuffle)
            return df.coalesce(100)
    
    def incremental_processing(self, new_data_path: str, 
                               existing_data_path: str,
                               output_path: str,
                               key_col: str):
        """
        Process only new/changed data (incremental batch)
        """
        new_df = self.spark.read.parquet(new_data_path)
        existing_df = self.spark.read.parquet(existing_data_path)
        
        # Find truly new records
        new_records = new_df.join(
            existing_df.select(key_col),
            on=key_col,
            how="left_anti"  # Records in new_df not in existing_df
        )
        
        # Append new records
        new_records.write.mode("append").parquet(output_path)
        
        return new_records.count()


# Checkpointing for fault tolerance
class FaultTolerantPipeline:
    def __init__(self, spark_session, checkpoint_dir: str):
        self.spark = spark_session
        self.spark.sparkContext.setCheckpointDir(checkpoint_dir)
    
    def process_with_checkpoint(self, df, complex_transformations):
        """
        Checkpoint after expensive operations to prevent recomputation
        """
        for i, transform in enumerate(complex_transformations):
            df = transform(df)
            
            # Checkpoint every 3 transformations
            if (i + 1) % 3 == 0:
                df = df.checkpoint()
                print(f"Checkpointed after transformation {i + 1}")
        
        return df


# Example usage
if __name__ == "__main__":
    pipeline = BatchPipeline(spark)
    
    result = pipeline.run_pipeline(
        input_path="s3://bucket/raw/orders/",
        output_path="s3://bucket/processed/"
    )
    
    print(f"Pipeline completed: {result}")
    
    spark.stop()
```

### Node.js Batch Processing with Streams

```typescript
// batch_processing/node-batch.ts

import { createReadStream, createWriteStream } from 'fs';
import { pipeline, Transform } from 'stream';
import { promisify } from 'util';
import * as readline from 'readline';

const pipelineAsync = promisify(pipeline);

interface Order {
  orderId: string;
  userId: string;
  productId: string;
  quantity: number;
  price: number;
  status: string;
  orderDate: string;
}

interface AggregatedResult {
  date: string;
  totalOrders: number;
  totalRevenue: number;
  avgOrderValue: number;
}

/**
 * Process large files in batches using Node.js streams
 * Memory efficient - doesn't load entire file
 */
class NodeBatchProcessor {
  private batchSize: number;
  private buffer: Order[] = [];
  private aggregations: Map<string, AggregatedResult> = new Map();
  
  constructor(batchSize: number = 10000) {
    this.batchSize = batchSize;
  }
  
  /**
   * Process large JSON lines file
   */
  async processFile(inputPath: string, outputPath: string): Promise<void> {
    const fileStream = createReadStream(inputPath);
    const rl = readline.createInterface({
      input: fileStream,
      crlfDelay: Infinity,
    });
    
    let lineCount = 0;
    
    for await (const line of rl) {
      try {
        const order: Order = JSON.parse(line);
        this.buffer.push(order);
        lineCount++;
        
        // Process batch when buffer is full
        if (this.buffer.length >= this.batchSize) {
          await this.processBatch();
          console.log(`Processed ${lineCount} records...`);
        }
      } catch (e) {
        // Skip malformed lines
        console.error(`Skipping malformed line: ${line.substring(0, 50)}`);
      }
    }
    
    // Process remaining records
    if (this.buffer.length > 0) {
      await this.processBatch();
    }
    
    // Write final aggregations
    await this.writeResults(outputPath);
    
    console.log(`Completed processing ${lineCount} records`);
  }
  
  private async processBatch(): Promise<void> {
    for (const order of this.buffer) {
      if (order.status !== 'completed') continue;
      
      const date = order.orderDate.split('T')[0];
      const revenue = order.quantity * order.price;
      
      const existing = this.aggregations.get(date) || {
        date,
        totalOrders: 0,
        totalRevenue: 0,
        avgOrderValue: 0,
      };
      
      existing.totalOrders++;
      existing.totalRevenue += revenue;
      existing.avgOrderValue = existing.totalRevenue / existing.totalOrders;
      
      this.aggregations.set(date, existing);
    }
    
    // Clear buffer
    this.buffer = [];
  }
  
  private async writeResults(outputPath: string): Promise<void> {
    const results = Array.from(this.aggregations.values())
      .sort((a, b) => a.date.localeCompare(b.date));
    
    const writeStream = createWriteStream(outputPath);
    
    for (const result of results) {
      writeStream.write(JSON.stringify(result) + '\n');
    }
    
    writeStream.end();
  }
}

/**
 * Transform stream for batch processing
 */
class BatchTransformStream extends Transform {
  private buffer: any[] = [];
  private batchSize: number;
  private processCallback: (batch: any[]) => Promise<any[]>;
  
  constructor(batchSize: number, processCallback: (batch: any[]) => Promise<any[]>) {
    super({ objectMode: true });
    this.batchSize = batchSize;
    this.processCallback = processCallback;
  }
  
  async _transform(chunk: any, encoding: string, callback: Function): Promise<void> {
    this.buffer.push(chunk);
    
    if (this.buffer.length >= this.batchSize) {
      try {
        const results = await this.processCallback(this.buffer);
        for (const result of results) {
          this.push(result);
        }
        this.buffer = [];
        callback();
      } catch (error) {
        callback(error);
      }
    } else {
      callback();
    }
  }
  
  async _flush(callback: Function): Promise<void> {
    if (this.buffer.length > 0) {
      try {
        const results = await this.processCallback(this.buffer);
        for (const result of results) {
          this.push(result);
        }
        callback();
      } catch (error) {
        callback(error);
      }
    } else {
      callback();
    }
  }
}

// Parallel batch processing
class ParallelBatchProcessor {
  private concurrency: number;
  
  constructor(concurrency: number = 4) {
    this.concurrency = concurrency;
  }
  
  async processInParallel<T, R>(
    items: T[],
    batchSize: number,
    processor: (batch: T[]) => Promise<R[]>
  ): Promise<R[]> {
    const batches: T[][] = [];
    
    // Split into batches
    for (let i = 0; i < items.length; i += batchSize) {
      batches.push(items.slice(i, i + batchSize));
    }
    
    // Process batches with concurrency limit
    const results: R[] = [];
    
    for (let i = 0; i < batches.length; i += this.concurrency) {
      const batchGroup = batches.slice(i, i + this.concurrency);
      const groupResults = await Promise.all(
        batchGroup.map(batch => processor(batch))
      );
      results.push(...groupResults.flat());
    }
    
    return results;
  }
}

// Usage
async function main() {
  const processor = new NodeBatchProcessor(10000);
  await processor.processFile(
    './data/orders.jsonl',
    './output/daily_aggregations.jsonl'
  );
}
```

---

## Real-World Scenarios

### Scenario 1: Daily ETL Pipeline

```python
# batch_processing/daily_etl.py

from pyspark.sql import SparkSession
from pyspark.sql.functions import col, current_date, date_sub
from datetime import datetime
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class DailyETLPipeline:
    """
    Production-grade daily ETL pipeline
    Runs at 2 AM, processes previous day's data
    """
    
    def __init__(self, spark: SparkSession, config: dict):
        self.spark = spark
        self.config = config
        self.run_date = datetime.now().strftime('%Y-%m-%d')
    
    def extract(self) -> dict:
        """Extract from multiple sources"""
        logger.info(f"Starting extraction for {self.run_date}")
        
        # Previous day's data
        process_date = date_sub(current_date(), 1)
        
        dataframes = {}
        
        # Orders from transactional DB
        dataframes['orders'] = self.spark.read \
            .format("jdbc") \
            .option("url", self.config['db_url']) \
            .option("dbtable", f"""
                (SELECT * FROM orders 
                 WHERE DATE(created_at) = '{process_date}') as orders
            """) \
            .option("user", self.config['db_user']) \
            .option("password", self.config['db_password']) \
            .load()
        
        # User data from data lake
        dataframes['users'] = self.spark.read.parquet(
            f"{self.config['data_lake']}/users/"
        )
        
        # Product catalog
        dataframes['products'] = self.spark.read.parquet(
            f"{self.config['data_lake']}/products/"
        )
        
        for name, df in dataframes.items():
            logger.info(f"Extracted {name}: {df.count()} records")
        
        return dataframes
    
    def transform(self, dataframes: dict) -> dict:
        """Apply transformations"""
        logger.info("Starting transformations")
        
        orders = dataframes['orders']
        users = dataframes['users']
        products = dataframes['products']
        
        # Enrich orders with user and product data
        enriched_orders = orders \
            .join(users.select('user_id', 'segment', 'country'), 
                  on='user_id', how='left') \
            .join(products.select('product_id', 'category', 'brand'),
                  on='product_id', how='left')
        
        # Calculate metrics
        from pyspark.sql.functions import broadcast
        
        results = {
            'fact_orders': enriched_orders,
            'daily_summary': self._create_daily_summary(enriched_orders),
            'segment_metrics': self._create_segment_metrics(enriched_orders)
        }
        
        return results
    
    def _create_daily_summary(self, df):
        from pyspark.sql.functions import sum, count, avg, countDistinct
        
        return df.agg(
            count('order_id').alias('total_orders'),
            sum('amount').alias('total_revenue'),
            avg('amount').alias('avg_order_value'),
            countDistinct('user_id').alias('unique_customers')
        )
    
    def _create_segment_metrics(self, df):
        from pyspark.sql.functions import sum, count
        
        return df.groupBy('segment', 'country').agg(
            count('order_id').alias('orders'),
            sum('amount').alias('revenue')
        )
    
    def load(self, results: dict):
        """Load to destination"""
        logger.info("Starting load phase")
        
        output_base = f"{self.config['warehouse']}/date={self.run_date}"
        
        # Fact table - append mode
        results['fact_orders'].write \
            .mode('append') \
            .partitionBy('country') \
            .parquet(f"{self.config['warehouse']}/fact_orders/")
        
        # Summary tables - overwrite daily partition
        results['daily_summary'].write \
            .mode('overwrite') \
            .parquet(f"{output_base}/daily_summary/")
        
        results['segment_metrics'].write \
            .mode('overwrite') \
            .parquet(f"{output_base}/segment_metrics/")
        
        logger.info("Load completed successfully")
    
    def run(self):
        """Execute full ETL"""
        try:
            dataframes = self.extract()
            results = self.transform(dataframes)
            self.load(results)
            logger.info(f"ETL completed for {self.run_date}")
            return True
        except Exception as e:
            logger.error(f"ETL failed: {e}")
            raise
```

### Scenario 2: Data Quality Checks

```python
# batch_processing/data_quality.py

from pyspark.sql import DataFrame
from pyspark.sql.functions import col, count, when, isnan, isnull
from typing import List, Dict
from dataclasses import dataclass

@dataclass
class QualityCheckResult:
    check_name: str
    passed: bool
    metric: float
    threshold: float
    details: str

class DataQualityChecker:
    """
    Run data quality checks before proceeding with pipeline
    """
    
    def __init__(self, df: DataFrame):
        self.df = df
        self.results: List[QualityCheckResult] = []
    
    def check_null_rate(self, column: str, max_rate: float = 0.01):
        """Check null rate doesn't exceed threshold"""
        total = self.df.count()
        nulls = self.df.filter(col(column).isNull()).count()
        null_rate = nulls / total if total > 0 else 0
        
        self.results.append(QualityCheckResult(
            check_name=f"null_rate_{column}",
            passed=null_rate <= max_rate,
            metric=null_rate,
            threshold=max_rate,
            details=f"{nulls} nulls out of {total} records"
        ))
        
        return self
    
    def check_unique(self, column: str):
        """Check column has unique values"""
        total = self.df.count()
        unique = self.df.select(column).distinct().count()
        
        self.results.append(QualityCheckResult(
            check_name=f"uniqueness_{column}",
            passed=total == unique,
            metric=unique / total if total > 0 else 0,
            threshold=1.0,
            details=f"{unique} unique values out of {total} records"
        ))
        
        return self
    
    def check_range(self, column: str, min_val: float, max_val: float):
        """Check values are within expected range"""
        out_of_range = self.df.filter(
            (col(column) < min_val) | (col(column) > max_val)
        ).count()
        
        total = self.df.count()
        in_range_rate = 1 - (out_of_range / total) if total > 0 else 0
        
        self.results.append(QualityCheckResult(
            check_name=f"range_{column}",
            passed=out_of_range == 0,
            metric=in_range_rate,
            threshold=1.0,
            details=f"{out_of_range} values outside [{min_val}, {max_val}]"
        ))
        
        return self
    
    def check_row_count(self, min_rows: int, max_rows: int = None):
        """Check row count is within expected range"""
        count = self.df.count()
        max_rows = max_rows or float('inf')
        
        self.results.append(QualityCheckResult(
            check_name="row_count",
            passed=min_rows <= count <= max_rows,
            metric=count,
            threshold=min_rows,
            details=f"{count} rows (expected {min_rows}-{max_rows})"
        ))
        
        return self
    
    def validate(self, fail_on_error: bool = True) -> bool:
        """Run all checks and return overall result"""
        all_passed = all(r.passed for r in self.results)
        
        # Log results
        for result in self.results:
            status = "✅ PASSED" if result.passed else "❌ FAILED"
            print(f"{status}: {result.check_name} - {result.details}")
        
        if not all_passed and fail_on_error:
            failed = [r for r in self.results if not r.passed]
            raise ValueError(f"Data quality checks failed: {failed}")
        
        return all_passed
```

---

## Common Pitfalls

### 1. Not Managing Partitions

```python
# ❌ BAD: Default partitions causing issues
df = spark.read.parquet("huge_dataset/")
result = df.groupBy("category").count()  # 200 default shuffle partitions

# ✅ GOOD: Configure partitions appropriately
spark.conf.set("spark.sql.shuffle.partitions", "auto")  # Spark 3.0+
# Or explicitly
spark.conf.set("spark.sql.shuffle.partitions", "50")  # For smaller datasets

# Repartition before expensive operations
df = df.repartition(100, "category")  # Partition by join/group key
```

### 2. Forgetting to Cache

```python
# ❌ BAD: Recomputing expensive operations
df = spark.read.parquet("data/").filter(...).join(...)
count1 = df.count()  # Computes entire lineage
count2 = df.groupBy("x").count().count()  # Recomputes again!

# ✅ GOOD: Cache when reusing DataFrames
df = spark.read.parquet("data/").filter(...).join(...)
df.cache()  # Or df.persist(StorageLevel.MEMORY_AND_DISK)
count1 = df.count()
count2 = df.groupBy("x").count().count()
df.unpersist()  # Clean up when done
```

### 3. Small Files Problem

```python
# ❌ BAD: Writing many small files
df.write.partitionBy("date", "hour", "minute").parquet("output/")
# Results in thousands of tiny files

# ✅ GOOD: Coalesce before writing
df.coalesce(10).write.parquet("output/")  # Fewer, larger files

# Or repartition to control output size
df.repartition(100).write.partitionBy("date").parquet("output/")
```

---

## Interview Questions

### Q1: What is the difference between Spark transformations and actions?

**A:** **Transformations** are lazy operations that define a computation but don't execute it (map, filter, join, groupBy). They build up a DAG (Directed Acyclic Graph). **Actions** trigger actual computation (count, collect, write, show). Spark optimizes the entire DAG before executing. This lazy evaluation enables optimizations like predicate pushdown and column pruning.

### Q2: How do you handle data skew in Spark?

**A:** Data skew (uneven distribution) causes some tasks to take much longer. Solutions: 1) **Salting**: Add random prefix to skewed keys, join, then remove. 2) **Broadcast joins**: If one side is small enough. 3) **Adaptive Query Execution** (Spark 3.0+): Automatically handles skew. 4) **Separate processing**: Process skewed keys separately. 5) **Repartition**: Use more partitions to spread data.

### Q3: When would you choose batch over stream processing?

**A:** Choose **batch** when: 1) Latency tolerance is minutes/hours. 2) Processing complete datasets (full recompute). 3) Complex transformations needing multiple passes. 4) Cost is a concern (batch is cheaper). 5) Data arrives in batches naturally. Choose **streaming** when: real-time insights needed, continuous data flow, sub-second latency required.

### Q4: Explain Spark's fault tolerance mechanism.

**A:** Spark achieves fault tolerance through **lineage**. Each RDD/DataFrame knows how it was derived from parent data (the transformations applied). If a partition is lost (node failure), Spark recomputes only that partition using the lineage graph. For expensive computations, **checkpointing** saves data to reliable storage, truncating the lineage. Shuffle data is written to disk, so shuffle outputs don't need recomputation.

---

## Quick Reference Checklist

### Spark Configuration
- [ ] Set appropriate shuffle partitions
- [ ] Configure executor/driver memory
- [ ] Enable adaptive query execution
- [ ] Set checkpoint directory

### Performance
- [ ] Cache reused DataFrames
- [ ] Use broadcast joins for small tables
- [ ] Handle data skew
- [ ] Coalesce before writing

### Data Quality
- [ ] Validate input data
- [ ] Check null rates
- [ ] Verify row counts
- [ ] Monitor for schema changes

### Operations
- [ ] Set up monitoring (Spark UI)
- [ ] Configure logging
- [ ] Handle failures gracefully
- [ ] Clean up resources

---

*Last updated: February 2026*

