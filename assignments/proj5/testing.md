# Testing

We strongly encourage testing your code yourself, especially after each part \(rather than all at the end\). The given tests for this project \(even more so than previous projects\) are **not** comprehensive tests: it **is** possible to write incorrect code that passes them all.

Things that you might consider testing for include: anything that we specify in the comments or in this document that a method should do that you don't see a test already testing for, and any edge cases that you can think of. Think of what valid inputs might break your code and cause it not to perform as intended, and add a test to make sure things are working.

## Cases not covered in the public tests

Here are a few cases mentioned in the spec but not tested for in the public test set:

* The checkpoint test provided checkpoints with a small number of transaction table and dirty page table entries -- enough to fit within a single page. Make sure your code still works even when there's a large amount of entries. If the entries aren't split up properly and too many entries are inserted into a single EndCheckpointLogRecord, your code will fail to flush the entry for exceeding the log tail size.
* When writing a large update \(large enough that the update must be split in two\), we don't provide a check for the dirty page table. If an entry didn't exist before, you should consider if the redo-only record or the undo-only record's LSN is the one that should be set as the recLSN for the page that was updated.
* For appropriate transactions after analysis/undo, make sure that transactions [have been cleaned up](https://github.com/berkeley-cs186/fa20-rookiedb/blob/master/src/test/java/edu/berkeley/cs186/database/recovery/DummyTransaction.java#L30) \(calling cleanup\(\) on a transaction should set that flag\)

And here are two common cases that your code should be prepared to handle:

* Make sure your redo logic still works without error even if there are no entries in the reconstructed dirty page table after analysis.
* Make sure your undo logic still works without error even if there are no transactions that need to be undone after analysis.

## Writing your own tests

You can use or modify any of the functions we provided in the public test set to write your own tests. Both `TestForwardProcessing.java` and `TestRestartRecovery.java` are structured identically in terms of how objects used in testing are created and used.

### Setup

```java
    @Before
    public void setup() throws Exception {
        testDir = tempFolder.newFolder("test-dir").getAbsolutePath();
        recoveryManager = loadRecoveryManager(testDir);
        DummyTransaction.cleanupTransactions();
        LogRecord.onRedoHandler(t -> {
        });
    }
```

The function above is run before every single test, and sets the value of the `recoveryManager` private variable to a new RecoveryManager object that operates on files in the `"test-dir"` directory \(locally this directory will be generated and likely cleaned up every time you run the test wherever JUnit is configured to create temporary directories\). The recovery manager object created will use a dummy locking system to prevent any dependencies with project 4, and 32 pages of memory in its buffer manager.

### Getting useful objects

The following helper methods are provided so that you can access important member variables of the RecoveryManager for testing purposes:

* `getBufferManager` - Returns the buffer manager used by the database. Useful if you want to manually run updates using records \(see `testFlushingRollback` for an example\)
* `getDiskSpaceManager` - Returns the disk space manager used by the database. Useful if you want to manually run updates using records \(see testFlushingRollbackFor an example\)
* `getLogManager` - Returns the recovery manager's log manager. Useful to directly append and flush logs to see how the recovery manager deals with them when rolling back. See `testAbortingEnd` for an example.
* `getLockRequests` - Returns a list of strings describing locks that were requested \(via `ARIESRecoveryManager#acquireTransactionLock`\) while executing the recovery manager. Useful for making sure that the analysis phase acquires X locks as necessary on touched pages.
* `getDirtyPageTable` - Gets the dirty page table of the recovery manager. Useful to make sure that pages are getting flushed properly and that recLSN's are set correctly or check that its reconstructed properly during analysis.
* `getTransactionTable` - Gets the transaction table of the recovery manager. Useful to make sure that entries are created/removed properly or check that its  reconstructed properly during analysis.

### IO Checks

Calling `getNumIOs` on the buffer manager will return the current number of IOs that have been incurred. By storing this value, performing some action, and the original value to the new value of getNumIOs you can check to see how many IOs were incurred between two points. This is useful in methods where we expect logs to be flushed \(which would incur IOs\). See `testFlushingRollback` for an example.

### Redo checks

You may have noticed calls to `setupRedoChecks` and `finishRedoChecks`. To help with testing, every time redo is called on a LogRecord we make a [call to a provided method](https://github.com/berkeley-cs186/sp21-rookiedb/blob/master/src/main/java/edu/berkeley/cs186/database/recovery/LogRecord.java#L156). During regular operation this will just be a no-op function, but during testing we can set this to be whatever we want using [onRedoHandler](https://github.com/berkeley-cs186/fa20-rookiedb/blob/master/src/main/java/edu/berkeley/cs186/database/recovery/LogRecord.java#L225-L232).

To make it more straight forward to do a series of checks, setupRedoChecks accepts a list of functional objects that take a LogRecord as an argument. Every time redo is called, the first LogRecord in the list is removed and is called using the LogRecord that was redone. For example, in [testAbortingEnd](https://github.com/berkeley-cs186/sp21-rookiedb/blob/master/src/test/java/edu/berkeley/cs186/database/recovery/TestForwardProcessing.java#L159-L163) we use this to check that the expected CLR's are emitted and redone in the order that we anticipated. This is useful when:

* when ending an aborted transaction, rolling back changes should involve calling redo on CLRs as they are generated
* during the undo phase, rolling back changes should involve calling redo on CLRs as they are generated
* during the redo phase

