# SeBiDa
Hi, in this repo you find (gradually) the code of SeBiDA. In its core, SeBiDa contains two components: (1) is the Non-Semantic Loading, which in turns conatains a Semantic Lifting component, and (2) Semantic Data Loading. 

As it is the main novelty of SeBiDa, we provide, for now, only the code of Semantic-Loading Loading. The other components may be added in the future.

[Apache Spark](http://spark.apache.org) is used for both Semantic and Non-Semantic Loading, and for the querying.

Semantic Loading
---
It reads an RDF graph as input (currently supporting NT files) and generats tables in Apache Parquet tabular format. SeBiDa suggests an RDF-class-based partitioning scheme by which RDF triples are partioned according to the classes they belong to.
An RDF instance with more than class (rdf:type) is saved into one table corresponding to one type (now chosen using a lexicographical order but class hierarchy will be considered in the future). The choses table will contain "reference columns" (of type boolean) that indicated if a particular tuple of the table (RDF instance) is "also of another type". Here is an example:
<br/>
<img src="https://github.com/EIS-Bonn/SeBiDa/raw/master/SeBiDa_classes_example.png" width="70%"/>

Queries, therefore, need to be rewritten under the hood, so these scattred data can be collected.


Usage (RDF2Parquet)
---
You could either package the code using Maven for example or use the pre-built JAR provided [here](https://sourceforge.net/projects/sebida/files/latest/download?source=files).

We ship this component for now as an undependent tool, named RDF2Parquet, as it undependently converts RDF files into Parquet tables following the partitioning scheme explained above.

We used for our evaluation the [Standalone Mode](http://spark.apache.org/docs/latest/spark-standalone.html). As the docs page above explains, you simply need to start your master (``./sbin/start-master.sh``), which returns you the master-spark-URL (check the logs). Then you start the workers (``./sbin/start-slave.sh <master-spark-URL>``).

Then you submit your JAR to spark like: 

``
bin/spark-submit --class bde.sebida.App --master [master-spark-URL] --conf spark.driver.memory=4g [other conf if needed] SeBiDa.jar [input RDF file] [results location] [masyer-spark-URL]
``

An example is:
``
/home/user/spark-1.6.2-bin-hadoop2.6/bin/spark-submit --class bde.sebida.App --master spark://aaa.bbb.ccc.ddd --executor-memory 200G --conf spark.io.compression.codec=lzf --conf spark.driver.memory=4g SeBiDa.jar hdfs://xxx:ppp/input/dataset200m.nt hdfs://xxx:ppp/output/tables-200m/ spark://aaa.bbb.ccc.ddd
``

The current version has been tested using Spark 1.6.2. Also, please notice that spark-master-url has to be specified twice. This duplication will be omitted in the next version.

Evaluation
---
For benchmarking, we used BSBM benchmark data generator (http://wifo5-03.informatik.uni-mannheim.de/bizer/berlinsparqlbenchmark/spec/BenchmarkRules/#datagenerator) and its 12 queries on SQL. For these latter, we actually didn't use the exact all SQL queries but rather created our own SQL conversion from BSBM SPARQL queries. This is because of the limited syntax of Spark SQL comparing to the standard SQL syntax used in BSBM standard. The rewritten SQL queries will published soon.

For more information, please contact me at: mami@cs.uni-bonn.de, and I'll be happy to assist.
