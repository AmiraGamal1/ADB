# Database Recovery Techniques

**Objective:**

* To understand database recovery techniques based on logging mechanisms.
* To learn how MySQL (specifically the InnoDB storage engine) performs crash recovery.
* To gain a better understanding of how a DBMS handles non-catastrophic crashes (such as system failures or power outages, excluding physical hardware damage).

--

## Basic Concepts Used in the Lab

**Crash Recovery:** Crash recovery involves restoring the database to its most recent consistent state before a system failure occurred. The recovery process ensures that the **atomicity** and **durability** properties of transactions are preserved.

**Log Sequence Number (LSN)** Is a unique incremental value which is assigned whenever changes occur in the InnoDB storage engine.

**Write Ahead Log (WAL)/ Redo Log** The Write-Ahead Log (WAL) called the redo log in **InnoDB**, is a sequential file that records all changes made to the database by ongoing transactions. These changes are written to the log before they are applied to the actual data files on disk.

**Undo Log** This log stores information required to roll back uncommitted transactions, It is also used to provide read consistency in multi-version concurrency control (MVCC) by preserving older versions of data for ongoing reads.

**Binary Log (binlog)** Is a set of files that record all changes made to the database. This includes both data modifications (such as **INSERT**, **UPDATE**, and **DELETE**) and schema changes (such as creating or altering tables).

**Buffer Pool** InnoDB uses a buffer pool in memory to cache frequently accessed data pages. 

**Dirty Pages** When data is modified, the corresponding pages in the buffer pool become "dirty" (modified but not yet written to disk).

**Fuzzy Checkpointing** Instead of flushing all dirty pages at once, InnoDB performs **fuzzy checkpointing**, a process where dirty pages are flushed to disk incrementally. This reduces I/O overhead and helps maintain system performance while ensuring recoverability.

**Purge** Is the process of deleting delete-marked records that are no longer visible to active transactions.

## InnoDB Crash Recovery Elastrated by example:

## Assignment

