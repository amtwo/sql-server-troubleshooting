# AG Send/Redo Overview

Availability Group synchronization works in two separate parts:

1. First, when a transaction happens on Primary, the transaction is written to the Transaction Log on Primary. 
  * If the AG replica is in synchronous commit mode, the log blocks are synchronously copied to the secondary replica before the transaction is considered committed. 
  * If a synchronous secondary is unavailable (offline, network issues, etc), the replica will seamlessly transition to asynchronous to preserve performance. There is no warning or indication of this other than an increased send queue.
  * If the AG replica is in asynchronous commit mode, the transaction is considered committed as soon as the log is hardened locally, and log blocks are asynchronously copied to the secondary replica.
  * Unsent transactions are represented as the "Send Queue" on DMVs & SQL Server instrumentation
2. Second, the transaction log on Secondary is replayed to roll forward or roll back transactions. 
  * Redo of transactions is **ALWAYS** asynchronous.
  * Redo uses the same redo mechanism as crash recovery (`DB STARTUP`), which is similar to a transaction log restore. It reads from the transaction log, and writes the corresponding changes to the data files.
  * Redo is usually single-threaded. SQL Server does support parallel redo, but only for a small number of databases (up to 6). If the AG has many databases, only a few will benefit from parallel redo. Controlling which DBs will benefit from parallel redo is difficult (practically impossible), so generally assume that any given database will only have single-threaded redo.
  * Transaction log needing redo is represented as the "Redo Queue" on DMVs and SQL Server instrumentation.