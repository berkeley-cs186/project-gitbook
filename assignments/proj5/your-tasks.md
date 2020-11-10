# Your Tasks

In this project you will implement write-ahead logging and support for savepoints, rollbacks, and ACID compliant restart recovery. If you haven't already, we recommend reading through the [recovery notes](https://cs186berkeley.net/resources/static/notes/n12-Recovery.pdf) and referencing them as needed while you work through this project.

## Understanding the Skeleton Code

This project will be centered around `ARIESRecoveryManager.java`, which implements the `RecoveryManager` interface.

Recall that there are two distinct modes of operation: _forward processing_ where we perform logging and maintain some metadata such as the dirty page table and transaction table during normal operation of the database, and _restart recovery_ \(a.k.a. crash recovery\), which consists of the processes taken when the database starts up again. During normal operation, the rest of the database calls various methods of the recovery manager to indicate that certain operations \(e.g. a page write or flush\) have occurred. During a restart, the `restart` method is called, which should bring the database back to a valid state.

Some files that will be useful to read through are:

* [`RecoveryManager.java`](https://github.com/berkeley-cs186/fa20-moocbase/blob/master/src/main/java/edu/berkeley/cs186/database/recovery/RecoveryManager.java) provides an overview of each of the methods you'll be implementing and when they're called
* [`TransactionTableEntry.java`](https://github.com/berkeley-cs186/fa20-moocbase/blob/master/src/main/java/edu/berkeley/cs186/database/recovery/TransactionTableEntry.java) represents an entry in our transaction table and tracks thing like the lastLSN, touched pages, and active savepoints.
* [`LogManager.java`](https://github.com/berkeley-cs186/fa20-moocbase/blob/master/src/main/java/edu/berkeley/cs186/database/recovery/LogManager.java) contains the implementation for the log manager, which provides an interface for appending, fetching, and flushing logs
* [`LogRecord.java`](https://github.com/berkeley-cs186/fa20-moocbase/blob/master/src/main/java/edu/berkeley/cs186/database/recovery/LogRecord.java) contains the super class for all of the different types of logs that we support. Every log has a type, an LSN, and a prevLSN. Certain subclasses of LogRecord optionally support extra methods.
* The [`records/`](https://github.com/berkeley-cs186/fa20-moocbase/tree/master/src/main/java/edu/berkeley/cs186/database/recovery/records) directory contains all the subclasses of LogRecord. It may seem a bit overwhelming at first but we'll introduce them one-by-one as you progress through the project.

#### Functional objects in Java

The project uses functional objects/interfaces in several places, which you may not be familiar with.

If you are not familiar with these in Java, you should look through [the official Java documentation](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html) - they're how you pass around functions and lambdas in Java, and the list of interfaces in the above link are simply the types of possible functions.

Each interface has a method to call the passed in function \(for example, [the `Consumer` interface has the `accept` method](https://docs.oracle.com/javase/8/docs/api/java/util/function/Consumer.html#accept-T-)\).

#### Disk Space Manager

You will not need to directly use the disk space manager in this project \(the various `LogRecord` subclasses will use it for you as needed\), but it does help to understand how our disk space manager organizes data at a high level.

The disk space manager is responsible for allocating pages, and our disk space manager divides pages into _partitions_. Page 40000000001, for example, is the 1st page \(0-indexed\) in partition 4. Partitions are explicitly allocated and freed \(but can only be freed if there are no pages in them\), and pages are always allocated under a partition.

Partition 0 is reserved for storing the log, which is why in a couple of places, you will see checks comparing the partition number against 0. Every other partition contains either a table or a serialized B+ tree object.

## Part 1: Forward Processing

![](../../.gitbook/assets/proj5-db-happy%20%281%29%20%281%29%20%281%29.png)

When the database is undergoing normal operation - where transactions are running normally, reading and writing data - the job of the recovery manager is to maintain the log, by adding log records and ensuring that the log is properly flushed when necessary, so that we can recover from a crash at any time. A few components of forward processing are already implemented for you. The tests for this part are all located in [`TestForwardProcessing.java`](https://github.com/berkeley-cs186/fa20-moocbase/blob/master/src/test/java/edu/berkeley/cs186/database/recovery/TestForwardProcessing.java).

#### Initialization

At the time the database is first created, before any transactions run, the recovery manager needs to first set the log up, which is done in the `initialize` method in `ARIESRecoveryManager.java`. We store the _master record_ as the very first log record in the log, at LSN 0 \(recall that the master record stores the LSN of the begin checkpoint record of the most recent successful checkpoint\).To simplify things when implementing the analysis phase of restart recovery, we also immediately perform a checkpoint, writing a begin and end checkpoint record in succession, and updating the master record. This has been implemented for you.

### Task 1: Transaction Status

Part of the recovery manager's job during forward processing is to maintain the status of running transactions, and log changes in transaction status. The recovery manager is notified of changes in transaction status through three methods:

* `commit` is called when a transaction attempts to move into the `COMMITTING` state.
* `abort` is called when a transaction attempts to move into the `ABORTING` state.
* `end` is called when a transaction attempts to move into the `COMPLETE` state.

In the three methods \(`commit`, `abort`, `end`\) that you need to implement, you will need to keep the transaction table up-to-date, set the status of the transaction accordingly, and write the appropriate log record to the log. You'll also need to implement:

* In `commit` the commit record needs to be flushed to disk before the commit call returns to ensure durability.
* In `end` if the transaction ends in an abort, all changes must be rolled back before an EndTransaction record is written. Look at the docstring for `rollbackToLSN` for details on how to rollback, and think about what LSN you can pass into this function to completely rollback a transaction. Note that you will need to update the dirty page table for [certain CLRs](https://github.com/berkeley-cs186/fa20-moocbase/blob/master/src/main/java/edu/berkeley/cs186/database/recovery/ARIESRecoveryManager.java#L161-L164) \(if you got your version of the code before 11/07/20 you may not have this docstring in your local copy\).

Some helper functions you may find useful for this task:

* `LogManager#fetchLogRecord`
* `LogManager#appendToLog`
* `LogManager#flushToLSN`
* `Transaction#setStatus`
* `LogRecord#isUndoable`
* `LogRecord#undo`

After completing this task you should pass `testAbort` and `testAbortingEnd`. `testSimpleCommit` requires the next task to work properly.

### Task 2: Logging

During normal operation several methods are called when certain events happen:

* `logAllocPart`, `logFreePart`, `logAllocPage`, `logFreePage`: these are called by the disk space manager whenever someone tries to create or delete a partition or page, and should append the appropriate log record, and have been implemented for you.
* `logPageWrite` is called by the buffer manager whenever someone tries to write to part of a page. The buffer manager guarantees that a single update covers at most`BufferManager.EFFECTIVE_PAGE_SIZE` bytes. We require that all log records are at most `DiskSpaceManager.PAGE_SIZE` bytes when serialized on disk. A page update can hold up to `BufferManager.EFFECTIVE_PAGE_SIZE` bytes of changes, but an update over a `x` byte region requires `2x` bytes in changes \(before and after\). Therefore, whenever a page update comes in over a region larger than `BufferManager.EFFECTIVE_PAGE_SIZE / 2` bytes, we split it up into two records:

  * an undo-only update record \(`after` is null\), then
  * a redo-only update record \(`before` is null\).

  \(note: order is important, in case we crash after writing only one of these\). If a page update is over a smaller region, we just append a single update record with both redo and undo information.

All of these methods should keep the tables maintained by the recovery manager up-to-date \(the dirty page table and transaction table\).

After completing this task the following tests should be passing: `testEnd`, `testSimpleCommit`, `testSimpleLogPageWrite`, `testTwoPartLogPageWrite`.

### Task 3: Savepoints

Recall from lecture that SQL has [savepoints](https://www.postgresql.org/docs/9.6/sql-savepoint.html) to allow for _partial rollback_: `SAVEPOINT pomelo` creates a savepoint named `pomelo` for the current running transaction, allowing a user to rollback all changes made after the savepoint by using `ROLLBACK TO SAVEPOINT pomelo`. The savepoint can be deleted with `RELEASE SAVEPOINT pomelo`.

Write-ahead logging lets us implement savepoints. The recovery manager has three methods related to savepoints, which correspond to the three SQL statements for savepoints, and follow the semantics of the corresponding SQL statements:

* `savepoint` creates a savepoint for the current transaction with the specified name. As with the `SAVEPOINT` statement in SQL, the name of the savepoint is scoped to the transaction: two different transactions may have their own savepoint called `pomelo`.
* `releaseSavepoint` deletes a specific savepoint for the current transaction; it behaves the same as the `RELEASE SAVEPOINT` statement in SQL.
* `rollbackToSavepoint` rolls the transaction back to the specified savepoint. All changes done after the savepoint should be undone, similarly to an aborting transaction, except the status of the transaction does not change; it behaves the same way as the `ROLLBACK TO SAVEPOINT` statement in SQL.

  See Transaction Status for more details on undoing changes.

The skeleton code has provided most of the implementation of savepoints for you - all that is left is to implement the logic for undoing changes in `rollbackToSavepoint`. This is extremely similar to the undo logic in `end()`, so if you already implemented the `rollbackToLSN` method to complete Task 1 you should be able to reuse that helper here.

After completing this task `testSimpleSavepoint`, `testFlushingRollback`, and `testNestedRollback` should be passing.

### Task 4: Checkpoints

Recall from lecture that in ARIES, we periodically perform _fuzzy checkpoints_ which occur even while other transactions run, to minimize recovery time after a crash, without bringing the database to a halt during forward processing.

The approach is outlined below. Note that part of the implementation is already provided to you; you are responsible for writing the end checkpoint records that are not covered by the given code.

First, a begin checkpoint record is added to the log.

Then, we write end checkpoint records, accounting for the fact that we may have to break up end checkpoint records due to too many DPT/Xact table entries. This is similar to how we sometimes had to break apart UpdatePage records into two in order to fit the change on one page; we may also need to write multiple end checkpoint records, if there too many DPT/transaction table entries.

An end checkpoint record should be written even if all tables are empty, and multiple end checkpoint records should only be written if necessary.

This is done as follows:

* iterate through the dirtyPageTable and copy the entries. If at any point, copying the current record would cause the end checkpoint record to be too large, an end checkpoint record with the copied DPT entries should be appended to the log.
* iterate through the transaction table, and copy the status/lastLSN, outputting end checkpoint records only as needed.
* iterate through the transaction table, and copy the touched pages, outputting end checkpoint records only as needed. Transactions without any touched pages should not appear here at all.
* output one final end checkpoint.

Finally, we must rewrite the master record with the LSN of the begin checkpoint record of the new successful checkpoint.

As an example, we might output the following end checkpoint records in the following order:

* EndCheckpoint with 200 DPT entries and 52 transaction table entries
* EndCheckpoint with 240 transaction table entries
* EndCheckpoint with 1 transaction table entry and 507 page numbers dirtied by transaction \#5.
* EndCheckpoint with 10 page numbers dirtied by transaction \#5, and 30 page

  numbers dirtied by transaction \#77.

You may find the `EndCheckpoint.fitsInOneRecord` static method useful for this; it takes in four parameters:

* the number of dirty page table entries stored in the record,
* the number of transaction number/status/lastLSN entries stored in the record,
* the number of transactions represented in the touchedPages map in the record, and
* the number of page numbers in the touchedPages map in the record,

  and returns whether the resulting record would fit in one page.

For example, for the record:

```text
EndCheckpoint{dpt={1 => 30000, 2 => 33000, 3 => 34000},
              txnTable={1 => (RUNNING, 33000), 2 => (RUNNING, 34000)},
              touchedPages={1 => {1, 4}, 2 => {3, 4, 5, 6}}
             }
```

the corresponding call is:

```java
EndCheckpoint.fitsInOneRecord(3,     /* dpt entries */
                             2,     /* txnTable entries */
                             2,     /* # of txns in touchedPages */
                             2 + 4) /* # of page numbers in touched pages */
```

After completing this task everything in `TestForwardProcessing.java` should be passing.

## Part 2: Restart Recovery

![](../../.gitbook/assets/proj5-db-off-the-cliff%20%281%29%20%281%29%20%281%29.png)

When the database starts up again, it enters restart recovery. Recall from lecture that this involves three phases: analysis, redo, and undo. The `RecoveryManager` interface exposes a single method for restart recovery: the `restart` method, which is called when the database starts up.

In order to test each phase in isolation, the skeleton has three package-private helper methods for restart recovery which you will need to implement: `restartAnalysis`, `restartRedo`, and `restartUndo`, which perform the analysis, redo, and undo phases respectively.

In addition to the three phases of recovery, the `restart` method does two things:

* between the redo and undo phases, any page in the dirty page table that isn't actually dirty \(has changes in-memory that have not been flushed\) should be removed from the dirty page table. These pages may be present in the DPT as a result of the analysis phase, if we are uncertain about whether a change has been flushed to disk successfully or not.
* after the undo phase, recovery has finished. To avoid having to abort all the transactions again should we crash, we take a checkpoint.

Finally, we allow for new transactions to start running once the redo phase has completed. In order to do this, `restart` returns a `Runnable` that performs the undo phase and checkpoint, instead of performing those actions immediately. The `Database` object runs the `Runnable` object in the background after `restart` returns. All of the tests for this part of the project are located in [`TestRestartRecovery.java`](https://github.com/berkeley-cs186/fa20-moocbase/blob/master/src/test/java/edu/berkeley/cs186/database/recovery/TestRestartRecovery.java).

### Task 5: Analysis

This section concerns just the `restartAnalysis` method, which performs the analysis pass of restart recovery.

**Master Record**

To begin analysis, the master record needs to be fetched, in order to find the LSN of the checkpoint to start at \(recall that in `initialize`, a checkpoint was written near the start of the log, so there is always a checkpoint to start at\).

**Scanning the Log**

The goal of analysis is to reconstruct the dirty page table and transaction table from the log.

The many types of log records encountered while scanning fall into three categories: log records for operations that a transaction does, checkpoint records, and log records for transaction status changes \(commit/abort/end\). \(There is also the master record, but it should never come up while scanning the log\).

**Log Records for Transaction Operations**

These are the records that involve a transaction, and therefore, we need to update the transaction table whenever we encounter one of these records. The following applies to any function with a non-empty result for `LogRecord#getTransNum()`

* If the transaction is not in the transaction table, it should be added to the table \(the `newTransaction` function object can be used to create a `Transaction` object\).
* The lastLSN of the transaction should be updated.
* If the log record is about a page \(`LogRecord#getPageNum` is non-empty\), the page needs to be added to the touchedPages set in the transaction table entry and the transaction needs to request an X lock on it.

The dirty page table will need to be updated for certain page-related log records:

* UpdatePage/UndoUpdatePage both may dirty a page in memory, without flushing changes to disk.
* AllocPage/FreePage/UndoAllocPage/UndoFreePage all make their changes visible on disk immediately, and can be seen as flushing all changes at the time \(including their own\) to disk.

**Log Records for Transaction Status Changes**

These three types of log records \(CommitTransaction/AbortTransaction/EndTransaction\) all change the status of a transaction.

When one of these records are encountered, the transaction table should be updated as described in the previous section. The status of the transaction should also be set to one of `COMMITTING`, `RECOVERY_ABORTING`, or `COMPLETE`.

If the record is an EndTransaction record, the transaction should also be cleaned up before setting the status, but the entry should **not** be removed from the transaction table yet.

**Checkpoint Records**

When a BeginCheckpoint record is encountered, the transaction counter needs to be updated \(`updateTransactionCounter`\). This ensures that once the database is running again it doesn't reuse a transaction number from before a crash.

When an EndCheckpoint record is encountered, the tables stored in the record should be combined with the tables currently in memory:

* The recLSN of a page in the checkpoint should always be used, even if we have a record in the dirty page table already, since the checkpoint is always more accurate than anything we can infer from just the log.
* The lastLSN of a transaction in the checkpoint should only be used if it is not smaller than the lastLSN of the transaction in the transaction memory \(if present\).
* All pages in the touchedPages table for a transaction should be added to the touchedPages set for the transaction in memory, if the transaction has not finished yet, and an X lock requested for each page.

Additionally, the status of transactions should be updated. Remember that checkpoints are fuzzy, meaning that they capture state from any time between the begin and end records. This means some of the transaction status's stored in the record may be out of date, e.g. the checkpoint may say a transaction is running when we already know that its aborting. Transactions will always advance through states in one of two ways:

* running -&gt; committing -&gt; complete
* running -&gt; aborting -&gt; complete

You should only update a transaction's status if the status in the checkpoint is more "advanced" than the status in memory. Some examples:

* if the checkpoint says a transaction is aborting and our in-memory table says its running, we should update the in-memory status to aborting because its possible to transition from running to complete.
* if the checkpoint says a transaction is running and our in-memory table says its committing, we wouldn't update our in-memory table. There's no way for the status to change from aborting to running in normal operation, and so the checkpoint must be out-of-date.

**Ending Transactions**

The transaction table at this point should have transactions that are in one of the following states: `RUNNING`, `COMMITTING`, or `RECOVERY_ABORTING`or `COMPLETE`.

All transactions in the `COMMITTING` state should be ended \(`cleanup()`, state set to `COMPLETE`, end transaction record written, and removed from the transaction table\).

All transactions in the `RUNNING` state should be moved into the `RECOVERY_ABORTING` state, and an abort transaction record should be written.

All transactions in the `COMPLETE` state should be removed the transaction table.

Nothing needs to be done for transactions in the `RECOVERY_ABORTING` state.

After completing this task you should be passing `testRestartAnalysis` and `testAnalysisCheckpoints`.

### Task 6: Redo

This section concerns just the `restartRedo` method, which performs the redo pass of restart recovery.

Recall from lecture that the redo phase begins at the lowest recLSN in the dirty page table. Scanning from that point on, we redo a record if it is redoable and:

* it is a partition-related record \(AllocPart, UndoAllocPart, FreePart, UndoFreePart\)
* it is a page-related record \(AllocPage, UndoAllocPage, FreePage, UndoFreePage\) where:
  * the page is in the DPT
  * the LSN is greater than or equal to the recLSN of the page
  * the pageLSN on the page itself is strictly less than the LSN of the record.

In order to check the pageLSN of a page, you'll need to fetch it from the buffer manager. We recommend you use the following template:

```text
Page page = bufferManager.fetchPage(
    getPageLockContext(pageNum).parentContext(), pageNum, false);
try {
    // Do anything that requires the page here
} finally {
    page.unpin();
}
```

The buffer manager always returns a pinned page which is why we use a try-finally block to ensure that the page is always unpinned once we're done using it. You may find [this method](https://github.com/berkeley-cs186/fa20-moocbase/blob/master/src/main/java/edu/berkeley/cs186/database/memory/Page.java#L161-L166) of the Page class useful here.

Be sure to account for the case where `restartRedo` is called on an empty log!

After finishing this task you should be passing `testRestartRedo`.

### Task 7: Undo

This section concerns just the `restartUndo` method, which performs the undo pass of restart recovery.

Recall from lecture that during the undo phase, we do not abort and undo the transactions one by one, due to a large number of random I/Os incurred as a result. Instead, we repeatedly undo the log record \(that needs to be undone\) with the highest LSN until we are done, making only one pass through the log.

The undo phase begins with the set of lastLSN of each of the aborting transactions \(in the `RECOVERY_ABORTING` state\).

We repeatedly fetch the log record of the largest of these LSNs and:

* if the record is undoable, we undo it\* and write the CLR out, updating the DPT as necessary;
* replace the LSN in the set with the undoNextLSN of the record if it has one, and the prevLSN otherwise;
* end the transaction if the LSN from the previous step is 0, removing it from the set and the transaction table. Refer to the Ending Transactions subsection of the Analysis task for a more comprehensive overview on what ending a transaction entails.

\* The `undo` method of `LogRecord` does not actually undo changes - it instead returns the compensation log record and a boolean flag indicating whether the log must be flushed after performing the undo. To actually undo changes, you will need to call `redo` on the returned CLR.

After finishing this task you should be passing `testRestartUndo`, `testUndoCLR`, and `testUndoDPTAndFlush`. Additionally, if you've implemented all three of the tasks in this part correctly every test in `TestRestartRecovery.java` should now be passing.

## Tying it all up

There is one TODO\(proj5\) comment in `TransactionImpl#startCommit` in `Database.java`. This is just removing a line of code and replacing it with the commented code, and sets up the database to use recovery.

## Important differences from ARIES as presented in lecture

There are a few important differences between ARIES as presented in the lecture, and the implementation of the recovery manager that you need to do in this project, which are mostly implementation details. **On exams, you should use the simplified version of ARIES as described in lecture whenever this project and lecture diverge.**

#### Forward Processing

|  | Project | Lecture |
| :--- | :--- | :--- |
| 1 | Page Updates may be two records \(Undo & Redo\) | Page Updates are always one record |
| 2 | Log page/partition allocations/frees | No such logging |
| 3 | Begin Checkpoint has transaction number | Begin Checkpoint has no information |
| 4 | End Checkpoint may have many records | End Checkpoint is one record |
| 5 | End Checkpoint and Transaction table have info about "touched" pages | No such information is stored |

* Page updates may be split up into two different records: an undo-only record, followed by a redo-only record, whereas in lecture, a single update corresponds to a single record that is both redoable and undoable.

  **Explanation:** In this implementation, we need a single log record to fit on a page. For very large updates \(more than half of the page\), it is impossible to fit both undo information and redo information in under a page \(we need around 2x the amount of data updated when logging an update\). For this reason, we choose to break up the update into two. The undo-only record must come first, so that we can revert the change should we crash before the redo-only record makes it to disk.

* We log page/partition allocations/frees.

  **Explanation:** This is just a quirk of how our disk space manager works, to ensure that it can be brought back to a consistent state after a crash.

* We keep the transaction number in the begin checkpoint record, whereas in lecture, the begin checkpoint has no information.

  **Explanation:** This is simply to ensure that after restart recovery with a checkpoint, we do not reuse transaction numbers from before the restart.

* A checkpoint may have many end\_checkpoint records, whereas in lecture, only a single end\_checkpoint record is used.

  **Explanation:** This is once more due to the fact that we need a single log record to fit in a page: we may have so many transactions/dirty pages that we cannot fit it all in one page.

* The end\_checkpoint record\(s\) and transaction table also contains information about all the pages that the transaction has "touched", whereas in lecture, this information is not maintained at all.

  **Explanation:** In order to re-acquire locks efficiently during restart recovery, we keep this information and write it in checkpoints - otherwise, the presence of a single transaction that has been running since the beginning of time up to the crash requires us to scan from the start of the log to determine what pages it needs to lock, even if we have a recent checkpoint.

#### Restart Recovery

|  | Project | Lecture |
| :--- | :--- | :--- |
| 1 | Clean up dirty page table after redoing changes | Step does not exist |
| 2 | Checkpoint after undo | Step does not exist |
| 3 | Process checkpoints upon reaching end\_checkpoint record \(single pass\) | Load checkpoints before starting analysis \(2 passes\) |
| 4 | Process page/partition allocation/free records \(may remove entries from dirty page table\) | These entries do not exist |

* We clean out the dirty page table of all pages that are not dirty in the buffer manager, after redoing all changes, whereas in lecture, this step is omitted.

  **Explanation:** We would like the dirty page table to reflect pages that are actually dirty \(because the only time they get removed is when the page is flushed, which may not ever happen if the already-flushed page is never modified again\). We omit this step in lecture and exams out of simplicity.

* We checkpoint after undo, whereas in lecture, this step is omitted.

  **Explanation:** This is a fairly unimportant step \(it is not necessary for correctness - we have completely recovered after undo\), but it is useful for performance reasons and a natural point to perform a checkpoint, and avoid a lot of work the next time we crash.

* We do a single pass through the records, processing checkpoints upon reaching the end-checkpoint record, whereas in lecture we first create the checkpoint's table and then scan through the log

  **Explanation:** The two approaches are equivalent - they will result in the exact same tables after analysis, but it is both simpler and more efficient to process the end\_checkpoint records and add their information to the tables in memory as we reach them, especially since we have multiple end\_checkpoint records.

* We process page/partition allocation/free records, and in some cases, remove the page from the dirty page table while doing so.

  **Explanation:** See explanation about these records under forward processing. In some cases \(free page/undo alloc page\), we remove a page from the dirty page table, and in others \(alloc page/undo free page\), we do not add the page to the dirty page table. This is because these operations all update on-disk data immediately.

