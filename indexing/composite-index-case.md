# Composite Index Case Study

## 1. Problem
Slow query on orders table.

```sql
SELECT *
FROM orders
WHERE user_id = 1001
AND status = 'PAID';


Execution time: ~120ms
Rows scanned: 250k

EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE user_id = 1001
AND status = 'PAID';

Result:

Seq Scan

No index used


CREATE INDEX idx_orders_user_status
ON orders(user_id, status)


Column order matters

High selectivity improves performance

