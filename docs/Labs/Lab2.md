# Concurrency Control Techniques

**Objective:**

* To understand concrrency control techniques based on Locking.
* To understand concurrency control techniques based Multiversion concept.
* To understand How MySQL deal with concrrent transations

## Isolation Level on MySQL

Isolation level ^instruct^ the database engine on how to manage multiple transactions being performed concurrently, and what violations are possible.

1. `READ UNCOMMITTED` is the lowest isolation level. It allows transactions to read the most recent version of a row, even if the change has not been committed by other transactions. This leads to the dirty read anomaly, as explained in [Lab 1](docs/Labs/Lab1.md).

2. `READ COMMITTED`  is one step above `READ UNCOMMITTED` and prevents dirty reads. In this mode, each `SELECT `operation retrieves the latest committed version of the row at the time of the query. However, this can result in non-repeatable reads.

3. `REPEATABLE READ` is a step above `READ COMMITTED` and prevents non-repeatable reads. In this mode, shared locks are placed on all rows read during the transaction and are held until the transaction ends. This prevents other transactions from modifying any of the data that was read.

4. `SERIALIZABLE`  is the highest isolation level. It prevents all types of concurrency anomalies. However, it is often impractical due to the extensive locking it requires, which increases the risk of deadlocks and can significantly reduce performance.

**Syntax for Setting the Isolation Level in MySQL**

    ```SQL
    SET TRANSACTION ISOLATION LEVEL [ISOLATION LEVEL];
    ```

## InnoDB Storage Engine


## Concurrency control techniques based on locking

## Concurrency control techniques based on timestamp ordering

## Advanced work

## Assignment (Simulate Deadlock Scienario)