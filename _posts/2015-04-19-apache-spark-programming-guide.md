---
layout: post
title: "apache spark programming guide"
description: ""
category: [apache]
tags: [spark, guide, python]
---
{% include JB/setup %}


### 1. overview

1. every spark app consists of a `driver program`

	* runs the user's `main` function

	* and executes various `parallel operations` on a cluster

1. the main abstraction spark provides is a `resilient distributed dataset (rdd)`

	* rdds are created by starting with a file in the hadoop file system

	* or an existing scala collection in the driver program

	* users may also ask spark to `persist` an rdd in memory

	* rdds automatically recover from node failures

1. a second abstraction is spark is `shared variables` that can be used in `parallel operations` support `two types of shared variables

	1. `broadcast variables`

				cache a value in memory on all nodes

	1. `accumulators`

				are variables that are only "added" to, such as counters and sums

### 2. linking with spark

1. spark 1.3.0 works with python 2.6 or higher (but not python 3)

1. it uses the standard cpython interpreter so c libraries like numpy can be used

1. to run spark app in python

	1. use `bin/spark-submit` to submit apps to a cluster

	1. use `bin/pyspark` to launch an interactive python shell

1. you need to import some spark classes into your program

				from pyspark import SparkContext, SparkConf

### 3. initialing spark

1. the first thing a spark program must do is to create a [SparkContext](https://spark.apache.org/docs/latest/api/python/pyspark.context.SparkContext-class.html) object

				which tells spark how to access a cluster

	* to create a `SparkContext` you first need to build a [SparkConf](https://spark.apache.org/docs/latest/api/python/pyspark.conf.SparkConf-class.html) object

				that contains information about your app

				conf = SparkConf().setAppName(appName).setMaster(master)
				sc   = SparkContext(conf=conf)

	* `appName` is a name for your app to show on the cluster un

	* `master` is a [Spark, Mesos or YARN cluster URL](https://spark.apache.org/docs/latest/submitting-applications.html#master-urls), or a special `local` string to run in local mode

	* in practice when running on a cluster, you will not want to hardcode `master` in the program, but rather [launch the app with spark-sbumit](https://spark.apache.org/docs/latest/submitting-applications.html) and receive it there

	* for local testing and unit tests you can pass "local" to run spark in-process

1. using the shell

	* in the pyspark shell a special interpreter-aware SparkContext is already created for you called `sc`

	* `making you own SparkContext will not work`

	* `--master` argument to set which master the context connects

	* `--py-files` to add python `.zip, .egg or .py` files to the runtime path

	* `--packages` to add dependencies to your shell session

1. examples

	* run `bin/pyspark` on exactly four cores

				$ /usr/local/bin/pyspark --master local[4]

	* to also add `code.py` to the search path (in order to later be able to import code)

				$ /usr/local/bin/pyspark --master local[4] --py-files code.py

	* launch pyspark shell in ipython

				$ PYSPARK_DRIVER_PYTHON=ipython /usr/local/bin/pyspark

				$ PYSPARK_DRIVER_PYTHON=ipython PYSPARK_DRIVER_PYTHON_OPTS="notebook --profile pyspark --notebook-dir=~/Document/hadoop" /usr/local/bin/pyspark

### 4. resilient distributed datasets (rdds)

1. two ways to create rdds

	* parallelizing an existing collection in your driver program

	* referencing a dataset in an external system

		* shared filesystem

		* hdfs

		* hbase

		* any data source offering a hadoop InputFormat

1. parallelized collection

				data = [1, 2, 3, 4, 5]
				distData = sc.parallelize(data)

	* once created the distributed dataset (distData) can be operated on in parallel

				distData.reduce(lambda a, b: a + b)

	* `partitions` to cut the dataset into

		* spark will run one task for each partition of the cluster

		* typically you want 2-4 partitions for each cpu in your cluster

				sc.parallelize(data, 10)

1. external datasets

	* pyspark can create distributed datasets from any storage source

		* supported by hadoop
		
		* including your local file system

		* hdfs

		* cassandra

		* hbase

		* [amazon s3](http://wiki.apache.org/hadoop/AmazonS3)

	* spark supports

		* text files

		* [SequenceFiles](http://hadoop.apache.org/common/docs/current/api/org/apache/hadoop/mapred/SequenceFileInputFormat.html)

		* any other hadoop [InputFormat](http://hadoop.apache.org/docs/stable/api/org/apache/hadoop/mapred/InputFormat.html)

	* rdds can be created using `SparkContext`'s textFile method

				>>> distFile = sc.textFile("data.txt")

	* reading files with spark

		* using local filesystem the file must also be accessible at the same path on worker nodes

				1 copy the file to all workers
				2 use a network-mounted shared file system

		* running on directories, compressed files, wildcards

				textFile("/my/directory")
				textFile("/my/directory/*.txt")
				textFile("/my/directory/*.gz")

		* `textFile` takes an optional second argument for controlling the number of partitions of the file

	* apart from text files, spark's python api also supports several other data fromats

		* `SparkContext.wholeTextFiles` read a directory containing multiple small text files

		* `RDD.saveAsPickleFile` and `SparkContext.pickleFile` support saving an RDD in a simple format consisting of pickled python objects

		* SequenceFile and hadoop Input/Output Formats

1. saving and loading SequenceFiles

				>>> rdd = sc.parallelize(range(1, 4)).map(lambda x: (x, "a" * x ))
				>>> rdd.saveAsSequenceFile("path/to/file")
				>>> sorted(sc.sequenceFile("path/to/file").collect())
				[(1, u'a'), (2, u'aa'), (3, u'aaa')]

### 5. shared variables

### 6. deploying to a cluster

### 7. unit testing

### 8. migrating from pre-1.0 versions of spark

### 9. where to go from here