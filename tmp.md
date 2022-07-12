https://github.com/apache/parquet-mr/blob/e990eb3f14c39273e46a9fce07ec85d2edf7fccb/parquet-hadoop/src/main/java/org/apache/parquet/hadoop/mapred/DeprecatedParquetOutputFormat.java#L67 https://github.com/apache/parquet-mr/blob/master/parquet-hadoop/src/main/java/org/apache/parquet/hadoop/ParquetOutputFormat.java

https://www.geeksforgeeks.org/extending-a-class-in-scala/

private val recordWriter: RecordWriter[Void, InternalRow] = {
  val outputFormat = {
    new ParquetOutputFormat[InternalRow]() {
      // ...
      override def getDefaultWorkFile(context: TaskAttemptContext, extension: String): Path = {
        // ..
        //  prefix is hard-coded here:
        new Path(path, f"part-r-$split%05d-$uniqueWriteJobId$bucketString$extension")
    }
  }
}
extends ParquetOutputFormat
