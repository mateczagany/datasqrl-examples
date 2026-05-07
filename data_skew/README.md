# Data Skew Test Project

Order analytics pipeline designed to produce data skew in Flink subtasks for testing purposes.

## Skew Design

The test data contains 10,000 orders distributed across 4 regions over 7 days (Jan 15-22, 2024):

| Region   | Orders | Stores              | Skew Factor |
|----------|--------|---------------------|-------------|
| us-east  | 4,000  | store-e1, e2, e3    | 1.5x (hot)  |
| us-west  | 2,000  | store-w1, w2        | 1.0x        |
| eu-west  | 2,000  | store-eu1, eu2      | 1.0x        |
| ap-south | 2,000  | store-ap1, ap2      | 1.0x        |

`us-east` has **50% more records** than each other region (4,000 vs 2,000). Any `GROUP BY region` operation will cause the subtask handling `us-east` to process disproportionately more data.

## Where Skew Manifests

1. **RegionStats** - `GROUP BY region` in TUMBLE window: the `us-east` key processes 15 records vs 10 for others.
2. **StoreStats** - `GROUP BY region, store_id`: us-east has 3 stores (more keys) adding to partition imbalance.
3. **CategoryByRegion** - `GROUP BY region, category`: same region-level skew cascading through category breakdown.

## Data Model

**Orders** (source stream):
- `order_id` - Unique order identifier
- `region` - Geographic region (skew key)
- `store_id` - Store within the region
- `customer_id` - Customer identifier
- `category` - Product category (electronics, clothing, food, books)
- `amount` - Order amount in dollars
- `event_time` - Order timestamp

## Running Tests

```bash
cmd.sh test order_analytics-test-package.json
```

## Environments

- **test**: Static JSONL files with skewed distribution (this project's focus)
