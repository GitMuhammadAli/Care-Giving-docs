# Data Validation - Complete Guide

> **MUST REMEMBER**: Data validation ensures data quality BEFORE processing. Three levels: schema validation (structure), semantic validation (business rules), statistical validation (anomaly detection). Tools: Great Expectations, Pandera, Deequ, dbt tests. Implement at ingestion AND after transformation. Fail fast on critical errors, log warnings for recoverable issues. Data quality dimensions: accuracy, completeness, consistency, timeliness, uniqueness.

---

## How to Explain Like a Senior Developer

"Data validation is your first line of defense against bad data corrupting your pipelines and reports. It happens at three levels: schema validation checks structure (right columns, types), semantic validation checks business rules (amounts > 0, valid status values), and statistical validation detects anomalies (sudden volume drops, distribution shifts). Use tools like Great Expectations or dbt tests to define expectations declaratively. Validate at ingestion (catch bad source data) and after transformations (catch bugs). The key is deciding what's a hard failure vs a warning - invalid payment amounts should fail, missing optional fields can warn. Track data quality metrics over time because problems often develop gradually."

---

## Core Implementation

### Schema Validation with Zod

```typescript
// data_validation/schema-validation.ts

import { z } from 'zod';

/**
 * Define schemas for validation
 */
const OrderSchema = z.object({
  orderId: z.string().uuid('Invalid order ID format'),
  userId: z.string().min(1, 'User ID required'),
  items: z.array(z.object({
    productId: z.string(),
    quantity: z.number().int().positive('Quantity must be positive'),
    price: z.number().positive('Price must be positive'),
  })).min(1, 'Order must have at least one item'),
  totalAmount: z.number().positive(),
  currency: z.enum(['USD', 'EUR', 'GBP']),
  status: z.enum(['pending', 'confirmed', 'shipped', 'delivered', 'cancelled']),
  createdAt: z.string().datetime(),
  metadata: z.record(z.unknown()).optional(),
});

type Order = z.infer<typeof OrderSchema>;

/**
 * Validation result with detailed errors
 */
interface ValidationResult<T> {
  valid: boolean;
  data?: T;
  errors: ValidationError[];
  warnings: ValidationWarning[];
}

interface ValidationError {
  field: string;
  message: string;
  value: unknown;
}

interface ValidationWarning {
  field: string;
  message: string;
  value: unknown;
}

/**
 * Schema validator with custom rules
 */
class DataValidator<T> {
  private schema: z.ZodSchema<T>;
  private customRules: Array<{
    name: string;
    validate: (data: T) => { valid: boolean; message?: string };
    severity: 'error' | 'warning';
  }> = [];
  
  constructor(schema: z.ZodSchema<T>) {
    this.schema = schema;
  }
  
  /**
   * Add custom validation rule
   */
  addRule(
    name: string,
    validate: (data: T) => { valid: boolean; message?: string },
    severity: 'error' | 'warning' = 'error'
  ): this {
    this.customRules.push({ name, validate, severity });
    return this;
  }
  
  /**
   * Validate single record
   */
  validate(input: unknown): ValidationResult<T> {
    const errors: ValidationError[] = [];
    const warnings: ValidationWarning[] = [];
    
    // Schema validation
    const parseResult = this.schema.safeParse(input);
    
    if (!parseResult.success) {
      for (const issue of parseResult.error.issues) {
        errors.push({
          field: issue.path.join('.'),
          message: issue.message,
          value: this.getNestedValue(input, issue.path),
        });
      }
      
      return { valid: false, errors, warnings };
    }
    
    // Custom rules
    for (const rule of this.customRules) {
      const result = rule.validate(parseResult.data);
      
      if (!result.valid) {
        const item = {
          field: rule.name,
          message: result.message || `Failed rule: ${rule.name}`,
          value: parseResult.data,
        };
        
        if (rule.severity === 'error') {
          errors.push(item);
        } else {
          warnings.push(item);
        }
      }
    }
    
    return {
      valid: errors.length === 0,
      data: errors.length === 0 ? parseResult.data : undefined,
      errors,
      warnings,
    };
  }
  
  /**
   * Validate batch with summary
   */
  validateBatch(inputs: unknown[]): {
    validRecords: T[];
    invalidRecords: Array<{ input: unknown; errors: ValidationError[] }>;
    summary: {
      total: number;
      valid: number;
      invalid: number;
      errorsByField: Record<string, number>;
    };
  } {
    const validRecords: T[] = [];
    const invalidRecords: Array<{ input: unknown; errors: ValidationError[] }> = [];
    const errorsByField: Record<string, number> = {};
    
    for (const input of inputs) {
      const result = this.validate(input);
      
      if (result.valid && result.data) {
        validRecords.push(result.data);
      } else {
        invalidRecords.push({ input, errors: result.errors });
        
        for (const error of result.errors) {
          errorsByField[error.field] = (errorsByField[error.field] || 0) + 1;
        }
      }
    }
    
    return {
      validRecords,
      invalidRecords,
      summary: {
        total: inputs.length,
        valid: validRecords.length,
        invalid: invalidRecords.length,
        errorsByField,
      },
    };
  }
  
  private getNestedValue(obj: unknown, path: (string | number)[]): unknown {
    let current: any = obj;
    for (const key of path) {
      if (current === null || current === undefined) return undefined;
      current = current[key];
    }
    return current;
  }
}

// Usage
const orderValidator = new DataValidator(OrderSchema)
  .addRule(
    'totalMatchesItems',
    (order) => {
      const calculatedTotal = order.items.reduce(
        (sum, item) => sum + item.quantity * item.price,
        0
      );
      const tolerance = 0.01;
      return {
        valid: Math.abs(calculatedTotal - order.totalAmount) < tolerance,
        message: `Total ${order.totalAmount} doesn't match items sum ${calculatedTotal}`,
      };
    },
    'error'
  )
  .addRule(
    'recentOrder',
    (order) => {
      const orderDate = new Date(order.createdAt);
      const dayOld = Date.now() - 24 * 60 * 60 * 1000;
      return {
        valid: orderDate.getTime() > dayOld,
        message: 'Order is more than 24 hours old',
      };
    },
    'warning'
  );
```

### Statistical Validation & Anomaly Detection

```typescript
// data_validation/statistical-validation.ts

interface ColumnStats {
  count: number;
  nullCount: number;
  uniqueCount: number;
  min?: number;
  max?: number;
  mean?: number;
  stdDev?: number;
  percentiles?: { p25: number; p50: number; p75: number; p99: number };
}

interface DataProfile {
  rowCount: number;
  columnStats: Record<string, ColumnStats>;
  timestamp: Date;
}

/**
 * Statistical data validator
 * Detects anomalies by comparing against historical profiles
 */
class StatisticalValidator {
  private historicalProfiles: DataProfile[] = [];
  private thresholds: {
    volumeChangePercent: number;
    nullRateChangePercent: number;
    meanChangeStdDevs: number;
  };
  
  constructor(thresholds?: Partial<typeof StatisticalValidator.prototype.thresholds>) {
    this.thresholds = {
      volumeChangePercent: 50,  // Alert if volume changes > 50%
      nullRateChangePercent: 20, // Alert if null rate changes > 20%
      meanChangeStdDevs: 3,      // Alert if mean changes > 3 std devs
      ...thresholds,
    };
  }
  
  /**
   * Profile a dataset
   */
  profile(data: Record<string, unknown>[]): DataProfile {
    const columnStats: Record<string, ColumnStats> = {};
    const columns = data.length > 0 ? Object.keys(data[0]) : [];
    
    for (const column of columns) {
      const values = data.map(row => row[column]);
      columnStats[column] = this.profileColumn(values);
    }
    
    return {
      rowCount: data.length,
      columnStats,
      timestamp: new Date(),
    };
  }
  
  private profileColumn(values: unknown[]): ColumnStats {
    const nonNull = values.filter(v => v !== null && v !== undefined);
    const unique = new Set(nonNull);
    
    const stats: ColumnStats = {
      count: values.length,
      nullCount: values.length - nonNull.length,
      uniqueCount: unique.size,
    };
    
    // Numeric stats
    const numbers = nonNull.filter(v => typeof v === 'number') as number[];
    if (numbers.length > 0) {
      numbers.sort((a, b) => a - b);
      
      stats.min = numbers[0];
      stats.max = numbers[numbers.length - 1];
      stats.mean = numbers.reduce((a, b) => a + b, 0) / numbers.length;
      
      const squaredDiffs = numbers.map(n => Math.pow(n - stats.mean!, 2));
      stats.stdDev = Math.sqrt(squaredDiffs.reduce((a, b) => a + b, 0) / numbers.length);
      
      stats.percentiles = {
        p25: this.percentile(numbers, 25),
        p50: this.percentile(numbers, 50),
        p75: this.percentile(numbers, 75),
        p99: this.percentile(numbers, 99),
      };
    }
    
    return stats;
  }
  
  private percentile(sorted: number[], p: number): number {
    const index = (p / 100) * (sorted.length - 1);
    const lower = Math.floor(index);
    const upper = Math.ceil(index);
    
    if (lower === upper) return sorted[lower];
    
    return sorted[lower] * (upper - index) + sorted[upper] * (index - lower);
  }
  
  /**
   * Detect anomalies compared to historical data
   */
  detectAnomalies(currentProfile: DataProfile): AnomalyReport {
    const anomalies: Anomaly[] = [];
    
    if (this.historicalProfiles.length === 0) {
      // First run - no comparison possible
      this.historicalProfiles.push(currentProfile);
      return { anomalies, isHealthy: true };
    }
    
    const baseline = this.getBaseline();
    
    // Volume check
    const volumeChange = Math.abs(
      (currentProfile.rowCount - baseline.avgRowCount) / baseline.avgRowCount * 100
    );
    
    if (volumeChange > this.thresholds.volumeChangePercent) {
      anomalies.push({
        type: 'volume_change',
        severity: volumeChange > 80 ? 'critical' : 'warning',
        message: `Row count changed by ${volumeChange.toFixed(1)}%`,
        details: {
          current: currentProfile.rowCount,
          expected: baseline.avgRowCount,
        },
      });
    }
    
    // Column-level checks
    for (const [column, stats] of Object.entries(currentProfile.columnStats)) {
      const baselineStats = baseline.columnStats[column];
      if (!baselineStats) continue;
      
      // Null rate check
      const currentNullRate = stats.nullCount / stats.count;
      const baselineNullRate = baselineStats.avgNullRate;
      const nullRateChange = Math.abs(currentNullRate - baselineNullRate) * 100;
      
      if (nullRateChange > this.thresholds.nullRateChangePercent) {
        anomalies.push({
          type: 'null_rate_change',
          severity: 'warning',
          message: `Null rate for ${column} changed by ${nullRateChange.toFixed(1)}%`,
          details: {
            current: currentNullRate,
            baseline: baselineNullRate,
          },
        });
      }
      
      // Mean change check (for numeric columns)
      if (stats.mean !== undefined && baselineStats.avgMean !== undefined) {
        const stdDevs = Math.abs(
          (stats.mean - baselineStats.avgMean) / (baselineStats.avgStdDev || 1)
        );
        
        if (stdDevs > this.thresholds.meanChangeStdDevs) {
          anomalies.push({
            type: 'distribution_shift',
            severity: 'warning',
            message: `Mean of ${column} shifted by ${stdDevs.toFixed(1)} std devs`,
            details: {
              currentMean: stats.mean,
              baselineMean: baselineStats.avgMean,
            },
          });
        }
      }
    }
    
    // Store current profile
    this.historicalProfiles.push(currentProfile);
    if (this.historicalProfiles.length > 30) {
      this.historicalProfiles.shift(); // Keep last 30
    }
    
    return {
      anomalies,
      isHealthy: anomalies.filter(a => a.severity === 'critical').length === 0,
    };
  }
  
  private getBaseline(): {
    avgRowCount: number;
    columnStats: Record<string, {
      avgNullRate: number;
      avgMean?: number;
      avgStdDev?: number;
    }>;
  } {
    const profiles = this.historicalProfiles;
    
    const avgRowCount = profiles.reduce((sum, p) => sum + p.rowCount, 0) / profiles.length;
    
    const columnStats: Record<string, any> = {};
    const columns = Object.keys(profiles[0].columnStats);
    
    for (const column of columns) {
      const colProfiles = profiles.map(p => p.columnStats[column]).filter(Boolean);
      
      columnStats[column] = {
        avgNullRate: colProfiles.reduce((sum, s) => sum + s.nullCount / s.count, 0) / colProfiles.length,
        avgMean: colProfiles[0].mean !== undefined
          ? colProfiles.reduce((sum, s) => sum + (s.mean || 0), 0) / colProfiles.length
          : undefined,
        avgStdDev: colProfiles[0].stdDev !== undefined
          ? colProfiles.reduce((sum, s) => sum + (s.stdDev || 0), 0) / colProfiles.length
          : undefined,
      };
    }
    
    return { avgRowCount, columnStats };
  }
}

interface Anomaly {
  type: 'volume_change' | 'null_rate_change' | 'distribution_shift' | 'schema_change';
  severity: 'critical' | 'warning' | 'info';
  message: string;
  details: Record<string, unknown>;
}

interface AnomalyReport {
  anomalies: Anomaly[];
  isHealthy: boolean;
}
```

### Great Expectations Style Validation

```typescript
// data_validation/expectations.ts

type ExpectationType = 
  | 'column_exists'
  | 'column_values_not_null'
  | 'column_values_unique'
  | 'column_values_in_set'
  | 'column_values_between'
  | 'column_mean_between'
  | 'table_row_count_between';

interface Expectation {
  type: ExpectationType;
  column?: string;
  params: Record<string, any>;
}

interface ExpectationResult {
  expectation: Expectation;
  success: boolean;
  observedValue: any;
  details: string;
}

/**
 * Great Expectations-style data validation
 */
class DataExpectations {
  private expectations: Expectation[] = [];
  
  /**
   * Expect column to exist
   */
  expectColumnExists(column: string): this {
    this.expectations.push({
      type: 'column_exists',
      column,
      params: {},
    });
    return this;
  }
  
  /**
   * Expect no null values
   */
  expectColumnValuesNotNull(column: string): this {
    this.expectations.push({
      type: 'column_values_not_null',
      column,
      params: {},
    });
    return this;
  }
  
  /**
   * Expect unique values
   */
  expectColumnValuesUnique(column: string): this {
    this.expectations.push({
      type: 'column_values_unique',
      column,
      params: {},
    });
    return this;
  }
  
  /**
   * Expect values in set
   */
  expectColumnValuesInSet(column: string, valueSet: any[]): this {
    this.expectations.push({
      type: 'column_values_in_set',
      column,
      params: { valueSet },
    });
    return this;
  }
  
  /**
   * Expect numeric values between range
   */
  expectColumnValuesBetween(
    column: string,
    minValue?: number,
    maxValue?: number
  ): this {
    this.expectations.push({
      type: 'column_values_between',
      column,
      params: { minValue, maxValue },
    });
    return this;
  }
  
  /**
   * Expect column mean in range
   */
  expectColumnMeanBetween(
    column: string,
    minValue: number,
    maxValue: number
  ): this {
    this.expectations.push({
      type: 'column_mean_between',
      column,
      params: { minValue, maxValue },
    });
    return this;
  }
  
  /**
   * Expect row count in range
   */
  expectTableRowCountBetween(minRows: number, maxRows?: number): this {
    this.expectations.push({
      type: 'table_row_count_between',
      params: { minRows, maxRows },
    });
    return this;
  }
  
  /**
   * Run all expectations
   */
  validate(data: Record<string, any>[]): {
    success: boolean;
    results: ExpectationResult[];
    successRate: number;
  } {
    const results: ExpectationResult[] = [];
    
    for (const expectation of this.expectations) {
      const result = this.runExpectation(expectation, data);
      results.push(result);
    }
    
    const successCount = results.filter(r => r.success).length;
    
    return {
      success: successCount === results.length,
      results,
      successRate: successCount / results.length,
    };
  }
  
  private runExpectation(
    expectation: Expectation,
    data: Record<string, any>[]
  ): ExpectationResult {
    switch (expectation.type) {
      case 'column_exists':
        return this.checkColumnExists(expectation, data);
      case 'column_values_not_null':
        return this.checkNotNull(expectation, data);
      case 'column_values_unique':
        return this.checkUnique(expectation, data);
      case 'column_values_in_set':
        return this.checkInSet(expectation, data);
      case 'column_values_between':
        return this.checkBetween(expectation, data);
      case 'column_mean_between':
        return this.checkMeanBetween(expectation, data);
      case 'table_row_count_between':
        return this.checkRowCount(expectation, data);
      default:
        throw new Error(`Unknown expectation type: ${expectation.type}`);
    }
  }
  
  private checkColumnExists(
    exp: Expectation,
    data: Record<string, any>[]
  ): ExpectationResult {
    const exists = data.length > 0 && exp.column! in data[0];
    return {
      expectation: exp,
      success: exists,
      observedValue: exists,
      details: exists 
        ? `Column ${exp.column} exists`
        : `Column ${exp.column} not found`,
    };
  }
  
  private checkNotNull(
    exp: Expectation,
    data: Record<string, any>[]
  ): ExpectationResult {
    const nullCount = data.filter(
      row => row[exp.column!] === null || row[exp.column!] === undefined
    ).length;
    
    return {
      expectation: exp,
      success: nullCount === 0,
      observedValue: nullCount,
      details: nullCount === 0
        ? `No null values in ${exp.column}`
        : `Found ${nullCount} null values in ${exp.column}`,
    };
  }
  
  private checkUnique(
    exp: Expectation,
    data: Record<string, any>[]
  ): ExpectationResult {
    const values = data.map(row => row[exp.column!]);
    const uniqueCount = new Set(values).size;
    const isUnique = uniqueCount === values.length;
    
    return {
      expectation: exp,
      success: isUnique,
      observedValue: { total: values.length, unique: uniqueCount },
      details: isUnique
        ? `All values in ${exp.column} are unique`
        : `${values.length - uniqueCount} duplicate values in ${exp.column}`,
    };
  }
  
  private checkInSet(
    exp: Expectation,
    data: Record<string, any>[]
  ): ExpectationResult {
    const { valueSet } = exp.params;
    const setLookup = new Set(valueSet);
    const invalid = data.filter(row => !setLookup.has(row[exp.column!]));
    
    return {
      expectation: exp,
      success: invalid.length === 0,
      observedValue: invalid.length,
      details: invalid.length === 0
        ? `All values in ${exp.column} are in expected set`
        : `${invalid.length} values not in expected set`,
    };
  }
  
  private checkBetween(
    exp: Expectation,
    data: Record<string, any>[]
  ): ExpectationResult {
    const { minValue, maxValue } = exp.params;
    const outOfRange = data.filter(row => {
      const val = row[exp.column!];
      if (typeof val !== 'number') return true;
      if (minValue !== undefined && val < minValue) return true;
      if (maxValue !== undefined && val > maxValue) return true;
      return false;
    });
    
    return {
      expectation: exp,
      success: outOfRange.length === 0,
      observedValue: outOfRange.length,
      details: outOfRange.length === 0
        ? `All values in ${exp.column} within range`
        : `${outOfRange.length} values outside [${minValue}, ${maxValue}]`,
    };
  }
  
  private checkMeanBetween(
    exp: Expectation,
    data: Record<string, any>[]
  ): ExpectationResult {
    const { minValue, maxValue } = exp.params;
    const values = data
      .map(row => row[exp.column!])
      .filter(v => typeof v === 'number') as number[];
    
    const mean = values.reduce((a, b) => a + b, 0) / values.length;
    const success = mean >= minValue && mean <= maxValue;
    
    return {
      expectation: exp,
      success,
      observedValue: mean,
      details: success
        ? `Mean of ${exp.column} (${mean.toFixed(2)}) is within range`
        : `Mean of ${exp.column} (${mean.toFixed(2)}) outside [${minValue}, ${maxValue}]`,
    };
  }
  
  private checkRowCount(
    exp: Expectation,
    data: Record<string, any>[]
  ): ExpectationResult {
    const { minRows, maxRows } = exp.params;
    const count = data.length;
    const success = count >= minRows && (maxRows === undefined || count <= maxRows);
    
    return {
      expectation: exp,
      success,
      observedValue: count,
      details: success
        ? `Row count ${count} within expected range`
        : `Row count ${count} outside [${minRows}, ${maxRows ?? '∞'}]`,
    };
  }
}

// Usage
const expectations = new DataExpectations()
  .expectColumnExists('order_id')
  .expectColumnValuesNotNull('order_id')
  .expectColumnValuesUnique('order_id')
  .expectColumnValuesInSet('status', ['pending', 'confirmed', 'shipped', 'delivered'])
  .expectColumnValuesBetween('amount', 0, 100000)
  .expectColumnMeanBetween('amount', 50, 500)
  .expectTableRowCountBetween(1000, 100000);

const validationResult = expectations.validate(data);
console.log(`Validation ${validationResult.success ? 'PASSED' : 'FAILED'}`);
console.log(`Success rate: ${(validationResult.successRate * 100).toFixed(1)}%`);
```

---

## Real-World Scenarios

### Scenario 1: Pipeline with Validation Gates

```typescript
// data_validation/pipeline-gates.ts

interface PipelineStage {
  name: string;
  execute: () => Promise<any>;
  validate: () => Promise<ValidationGateResult>;
}

interface ValidationGateResult {
  passed: boolean;
  criticalErrors: string[];
  warnings: string[];
}

class ValidatedPipeline {
  private stages: PipelineStage[] = [];
  
  addStage(stage: PipelineStage): this {
    this.stages.push(stage);
    return this;
  }
  
  async run(): Promise<{
    success: boolean;
    stageResults: Array<{
      stage: string;
      success: boolean;
      validation: ValidationGateResult;
    }>;
  }> {
    const stageResults = [];
    
    for (const stage of this.stages) {
      console.log(`Running stage: ${stage.name}`);
      
      // Execute stage
      await stage.execute();
      
      // Validate output
      const validation = await stage.validate();
      
      stageResults.push({
        stage: stage.name,
        success: validation.passed,
        validation,
      });
      
      // Log warnings
      for (const warning of validation.warnings) {
        console.warn(`[${stage.name}] Warning: ${warning}`);
      }
      
      // Stop on critical errors
      if (!validation.passed) {
        console.error(`[${stage.name}] Validation failed:`);
        for (const error of validation.criticalErrors) {
          console.error(`  - ${error}`);
        }
        
        return { success: false, stageResults };
      }
      
      console.log(`[${stage.name}] Validation passed ✓`);
    }
    
    return { success: true, stageResults };
  }
}

// Usage
const pipeline = new ValidatedPipeline()
  .addStage({
    name: 'extract',
    execute: async () => { /* extract logic */ },
    validate: async () => {
      // Validate extracted data
      const rowCount = 1000; // actual count
      return {
        passed: rowCount > 0,
        criticalErrors: rowCount === 0 ? ['No data extracted'] : [],
        warnings: rowCount < 100 ? ['Low volume extracted'] : [],
      };
    },
  })
  .addStage({
    name: 'transform',
    execute: async () => { /* transform logic */ },
    validate: async () => {
      // Validate transformed data
      return {
        passed: true,
        criticalErrors: [],
        warnings: [],
      };
    },
  });

await pipeline.run();
```

---

## Common Pitfalls

### 1. Validating Only at Ingestion

```typescript
// ❌ BAD: Only validate input
const rawData = await extract();
validate(rawData);  // ✓ Validated
const transformed = transform(rawData);  // Bug introduced here
load(transformed);  // Bad data loaded!

// ✅ GOOD: Validate at each stage
const rawData = await extract();
validate(rawData, 'extraction');

const transformed = transform(rawData);
validate(transformed, 'transformation');  // Catches bugs

load(transformed);
validate(loadedData, 'post-load');  // Verify load succeeded
```

### 2. Hard-Failing on Warnings

```typescript
// ❌ BAD: Fail on any issue
if (nullRate > 0) {
  throw new Error('Found null values');  // Too strict!
}

// ✅ GOOD: Distinguish errors from warnings
if (nullRate > 0.5) {
  throw new Error('Critical: >50% null values');
} else if (nullRate > 0.1) {
  logger.warn('Warning: >10% null values');
  metrics.increment('data_quality.warnings');
}
// Continue processing
```

### 3. Not Tracking Quality Over Time

```typescript
// ❌ BAD: Point-in-time validation only
if (isValid(data)) {
  process(data);
}

// ✅ GOOD: Track quality metrics over time
const qualityMetrics = {
  nullRate: calculateNullRate(data),
  duplicateRate: calculateDuplicateRate(data),
  volumeChange: calculateVolumeChange(data, previousVolume),
};

// Store for trending
await metricsStore.record('data_quality', qualityMetrics);

// Alert on degradation trends
if (isQualityDegrading(qualityMetrics)) {
  alertOps('Data quality degrading - review required');
}
```

---

## Interview Questions

### Q1: What are the main dimensions of data quality?

**A:** Six key dimensions: 1) **Accuracy**: Data correctly represents reality. 2) **Completeness**: Required fields are present. 3) **Consistency**: Data agrees across systems. 4) **Timeliness**: Data is current/fresh enough. 5) **Uniqueness**: No unintended duplicates. 6) **Validity**: Data conforms to rules/formats.

### Q2: How do you handle data validation failures in production pipelines?

**A:** 1) **Classify severity**: Critical (stop pipeline) vs warning (continue with alerts). 2) **Quarantine bad data**: Move invalid records to error queue for investigation. 3) **Alert and monitor**: Notify team, track quality metrics. 4) **Graceful degradation**: Process valid records, retry invalid later. 5) **Root cause analysis**: Investigate source of bad data.

### Q3: What is schema drift and how do you handle it?

**A:** Schema drift is when source data structure changes unexpectedly (new columns, type changes, removed fields). Handle with: 1) **Schema registry**: Central schema definitions with versioning. 2) **Schema evolution policies**: Define allowed changes. 3) **Flexible parsing**: Handle unknown fields gracefully. 4) **Monitoring**: Alert on schema differences. 5) **Testing**: Validate against expected schema.

---

## Quick Reference Checklist

### Validation Levels
- [ ] Schema validation (structure)
- [ ] Semantic validation (business rules)
- [ ] Statistical validation (anomalies)

### Pipeline Integration
- [ ] Validate at ingestion
- [ ] Validate after transformation
- [ ] Validate after load

### Monitoring
- [ ] Track quality metrics over time
- [ ] Alert on degradation
- [ ] Dashboard for visibility

---

*Last updated: February 2026*

