# Spark for Java: Summary

See this Udemy course: https://www.udemy.com/course/apache-spark-for-java-developers

## 1. Introduction

**Apache Spark** is a powerful **open-source distributed computing system** that provides an interface for programming entire clusters with implicit data parallelism and fault tolerance

It's designed to handle large-scale data processing tasks efficiently

Here are some of the general/main **features of Spark** along with samples in Java:

**Speed**: Spark's main feature is its speed. It can run workloads up to 100 times faster than traditional MapReduce jobs due to its in-memory processing capability

```java
// Sample Java Spark code to calculate the word count from a text file
JavaRDD<String> textFile = sc.textFile("hdfs://...");
JavaRDD<String> words = textFile.flatMap(line -> Arrays.asList(line.split(" ")).iterator());
JavaPairRDD<String, Integer> wordCounts = words.mapToPair(word -> new Tuple2<>(word, 1)).reduceByKey(Integer::sum);
wordCounts.saveAsTextFile("hdfs://...");
```

**Ease of Use**: Spark provides easy-to-use APIs for Java, Scala, Python, and R, making it accessible to a wide range of developers

```java
// Sample Java Spark code to filter even numbers from an RDD
JavaRDD<Integer> numbers = sc.parallelize(Arrays.asList(1, 2, 3, 4, 5));
JavaRDD<Integer> evenNumbers = numbers.filter(num -> num % 2 == 0);
```

**Generality**: Spark supports various workloads like batch processing, interactive queries, streaming analytics, and machine learning

```java
// Sample Java Spark code to perform simple machine learning using MLlib
JavaRDD<LabeledPoint> data = ... // Load labeled data.
LinearRegressionModel model = new LinearRegressionWithSGD().train(data.rdd());
```

**Fault Tolerance**: Spark provides fault tolerance through lineage information, allowing it to recompute lost data using the information about how it was derived

```java
// Sample Java Spark code showing fault tolerance by recomputing lost data
JavaRDD<String> textFile = sc.textFile("hdfs://...");
JavaRDD<String> errors = textFile.filter(line -> line.contains("error"));
errors.persist(StorageLevel.DISK_ONLY()); // Persist RDD to disk
long numErrors = errors.count(); // Compute count
```

**Lazy Evaluation**: Spark uses lazy evaluation, which means transformations on RDDs are not executed until an action is called. This optimizes the execution plan

```java
// Sample Java Spark code showing lazy evaluation
JavaRDD<String> lines = sc.textFile("hdfs://...");
JavaRDD<String> words = lines.flatMap(line -> Arrays.asList(line.split(" ")).iterator());
words.persist(StorageLevel.MEMORY_ONLY()); // Persist RDD to memory
long numWords = words.count(); // Compute count
```

**In-Memory Processing**: Spark keeps intermediate data in memory, which makes it faster than disk-based systems like Hadoop MapReduce

```java
// Sample Java Spark code to cache an RDD in memory
JavaRDD<String> textFile = sc.textFile("hdfs://...");
textFile.persist(StorageLevel.MEMORY_ONLY()); // Cache RDD in memory
```

These are just a few of the main features of Spark demonstrated with sample Java code snippets

Spark's versatility and performance make it a popular choice for big data processing tasks

## 2. Java Samples for Spark High-level APIs 

Let's delve into some more advanced features of Apache Spark for Java:

**Spark SQL**: Spark SQL is a module for working with structured data. It allows you to execute SQL queries as well as manipulate data using DataFrame API, which provides a more user-friendly interface than RDDs

SPARK SQL SAMPLE1:

```java
// Creating a DataFrame from an RDD of JSON strings
JavaRDD<String> jsonStrings = ...
DataFrame df = sqlContext.read().json(jsonStrings);

// Registering DataFrame as a temporary table
df.createOrReplaceTempView("people");

// Executing SQL query on the DataFrame
DataFrame result = sqlContext.sql("SELECT * FROM people WHERE age >= 18");
```

SPARK SQL SAMPLE2:

```java
// Creating a DataFrame from a CSV file
DataFrame df = sqlContext.read().format("csv").option("header", "true").load("path/to/csv/file.csv");

// Registering DataFrame as a temporary table
df.createOrReplaceTempView("sales");

// Executing SQL query on the DataFrame
DataFrame result = sqlContext.sql("SELECT * FROM sales WHERE product='Apple'");
```

**DataFrame API**: DataFrame API provides a more optimized way to work with structured data compared to RDDs. It's available in Java and allows for easier data manipulation and processing

SPARK DATAFRAME API SAMPLE1:

```java
// Creating DataFrame from a list of JavaBeans
List<Person> peopleList = Arrays.asList(
    new Person("Alice", 25),
    new Person("Bob", 30)
);
JavaRDD<Person> peopleRDD = sc.parallelize(peopleList);
DataFrame df = sqlContext.createDataFrame(peopleRDD, Person.class);

// Filtering DataFrame using DataFrame API
DataFrame filteredDF = df.filter(df.col("age").geq(18));
```

SPARK DATAFRAME API SAMPLE2:

```java
// Creating DataFrame from a list of tuples
List<Tuple2<String, Integer>> data = Arrays.asList(
    new Tuple2<>("Alice", 25),
    new Tuple2<>("Bob", 30)
);
JavaRDD<Row> rowRDD = sc.parallelize(data).map(tuple -> RowFactory.create(tuple._1(), tuple._2()));
StructType schema = DataTypes.createStructType(Arrays.asList(
    DataTypes.createStructField("name", DataTypes.StringType, false),
    DataTypes.createStructField("age", DataTypes.IntegerType, false)
));
DataFrame df = sqlContext.createDataFrame(rowRDD, schema);

// Filtering DataFrame using DataFrame API
DataFrame filteredDF = df.filter(df.col("age").geq(18));
```

**Dataset API**: Dataset API combines the benefits of RDDs and DataFrame API by providing type safety and high-level abstractions. It's available in Scala and Java

SPARK DATASET API SAMPLE1:

```java
// Creating a Dataset from a list of JavaBeans
List<Person> peopleList = Arrays.asList(
    new Person("Alice", 25),
    new Person("Bob", 30)
);
Dataset<Person> peopleDS = spark.createDataset(peopleList, Encoders.bean(Person.class));

// Filtering Dataset using Dataset API
Dataset<Person> filteredDS = peopleDS.filter(person -> person.getAge() >= 18);
```

SPARK DATASET API SAMPLE2:

```java
// Creating a Dataset from a list of tuples
List<Tuple2<String, Integer>> data = Arrays.asList(
    new Tuple2<>("Alice", 25),
    new Tuple2<>("Bob", 30)
);
Dataset<Row> df = spark.createDataset(data, Encoders.tuple(Encoders.STRING(), Encoders.INT())).toDF("name", "age");

// Filtering Dataset using Dataset API
Dataset<Row> filteredDS = df.filter(col("age").geq(18));
```

**Spark Streaming**: Spark Streaming is an extension of the core Spark API that enables scalable, high-throughput, fault-tolerant stream processing of live data streams

SPARK STREAMING SAMPLE1:

```java
// Creating a Spark Streaming context
JavaStreamingContext streamingContext = new JavaStreamingContext(sparkConf, Durations.seconds(1));

// Creating a DStream from a Kafka stream
Map<String, Object> kafkaParams = new HashMap<>();
kafkaParams.put("bootstrap.servers", "localhost:9092");
JavaInputDStream<ConsumerRecord<String, String>> kafkaStream = KafkaUtils.createDirectStream(
    streamingContext,
    LocationStrategies.PreferConsistent(),
    ConsumerStrategies.Subscribe(Arrays.asList("topic"), kafkaParams)
);
```

SPARK STREAMING SAMPLE2:

```java
// Creating a Spark Streaming context
JavaStreamingContext streamingContext = new JavaStreamingContext(sparkConf, Durations.seconds(1));

// Creating a DStream from a TCP socket
JavaReceiverInputDStream<String> lines = streamingContext.socketTextStream("localhost", 9999);

// Counting words in each batch of data
JavaDStream<String> words = lines.flatMap(line -> Arrays.asList(line.split(" ")).iterator());
JavaPairDStream<String, Integer> wordCounts = words.mapToPair(word -> new Tuple2<>(word, 1)).reduceByKey(Integer::sum);

// Printing the word counts
wordCounts.print();
```

**MLlib**: MLlib is Spark's scalable machine learning library. It provides various machine learning algorithms and utilities for data preprocessing, feature engineering, model evaluation, etc

SPARK MLIB SAMPLE1:

```java
// Loading data for training
Dataset<Row> data = spark.read().format("libsvm").load("data/mllib/sample_libsvm_data.txt");

// Splitting the data into training and testing sets
Dataset<Row>[] splits = data.randomSplit(new double[]{0.7, 0.3});
Dataset<Row> trainingData = splits[0];
Dataset<Row> testData = splits[1];

// Training a RandomForestClassifier model
RandomForestClassifier rf = new RandomForestClassifier()
    .setLabelCol("label")
    .setFeaturesCol("features")
    .setNumTrees(10);
RandomForestClassificationModel model = rf.fit(trainingData);
```

SPARK MLIB SAMPLE2:

```java
// Loading data for training
Dataset<Row> data = spark.read().format("libsvm").load("data/mllib/sample_libsvm_data.txt");

// Splitting the data into training and testing sets
Dataset<Row>[] splits = data.randomSplit(new double[]{0.7, 0.3});
Dataset<Row> trainingData = splits[0];
Dataset<Row> testData = splits[1];

// Training a RandomForestClassifier model
RandomForestClassifier rf = new RandomForestClassifier()
    .setLabelCol("label")
    .setFeaturesCol("features")
    .setNumTrees(10);
RandomForestClassificationModel model = rf.fit(trainingData);
```

These are just a few of the advanced features and APIs available in Apache Spark for Java

Spark provides a wide range of tools and libraries for various use cases, making it a powerful framework for big data processing, machine learning, and stream processing

## 3. More advance samples

**Custom Aggregation in Spark SQL**:

```java
// Registering a UDAF (User Defined Aggregate Function) for calculating median
sqlContext.udf().register("median", new UserDefinedAggregateFunction() {
    @Override
    public StructType inputSchema() {
        return new StructType().add("value", DataTypes.DoubleType);
    }

    @Override
    public StructType bufferSchema() {
        return new StructType().add("values", DataTypes.createArrayType(DataTypes.DoubleType));
    }

    @Override
    public DataType dataType() {
        return DataTypes.DoubleType;
    }

    @Override
    public boolean deterministic() {
        return true;
    }

    @Override
    public void initialize(MutableAggregationBuffer buffer) {
        buffer.update(0, new ArrayList<Double>());
    }

    @Override
    public void update(MutableAggregationBuffer buffer, Row input) {
        double value = input.getDouble(0);
        ArrayList<Double> values = new ArrayList<>(buffer.getList(0));
        values.add(value);
        buffer.update(0, values);
    }

    @Override
    public void merge(MutableAggregationBuffer buffer1, Row buffer2) {
        ArrayList<Double> values1 = new ArrayList<>(buffer1.getList(0));
        ArrayList<Double> values2 = new ArrayList<>(buffer2.getList(0));
        values1.addAll(values2);
        buffer1.update(0, values1);
    }

    @Override
    public Double evaluate(Row buffer) {
        ArrayList<Double> values = new ArrayList<>(buffer.getList(0));
        Collections.sort(values);
        int size = values.size();
        if (size % 2 == 0) {
            return (values.get(size / 2) + values.get(size / 2 - 1)) / 2;
        } else {
            return values.get(size / 2);
        }
    }
});

// Using the UDAF to calculate median
DataFrame df = sqlContext.read().format("csv").option("header", "true").load("path/to/csv/file.csv");
df.createOrReplaceTempView("data");
DataFrame result = sqlContext.sql("SELECT median(value) FROM data");
result.show();
```

**Handling Stateful Operations in Spark Streaming**:

```java
// Stateful word count using updateStateByKey
JavaPairDStream<String, Integer> wordCounts = lines.flatMapToPair(line -> Arrays.asList(line.split(" ")).stream()
        .map(word -> new Tuple2<>(word, 1)).iterator())
        .updateStateByKey((values, state) -> {
            int sum = state.orElse(0);
            for (Integer value : values) {
                sum += value;
            }
            return Optional.of(sum);
        });

wordCounts.print();
```

**Feature Engineering and Pipeline in MLlib**:

```java
// Feature Engineering and Pipeline
// Load data
Dataset<Row> data = spark.read().format("libsvm").load("data/mllib/sample_libsvm_data.txt");

// Split data into training and testing sets
Dataset<Row>[] splits = data.randomSplit(new double[]{0.7, 0.3});
Dataset<Row> trainingData = splits[0];
Dataset<Row> testData = splits[1];

// Configure an ML pipeline, which consists of three stages: tokenizer, hashingTF, and logistic regression
Tokenizer tokenizer = new Tokenizer().setInputCol("text").setOutputCol("words");
HashingTF hashingTF = new HashingTF().setNumFeatures(1000).setInputCol(tokenizer.getOutputCol()).setOutputCol("features");
LogisticRegression lr = new LogisticRegression().setMaxIter(10).setRegParam(0.01);
Pipeline pipeline = new Pipeline().setStages(new PipelineStage[]{tokenizer, hashingTF, lr});

// Fit the pipeline to training data
PipelineModel model = pipeline.fit(trainingData);

// Make predictions on test data
Dataset<Row> predictions = model.transform(testData);

// Evaluate the predictions
MulticlassClassificationEvaluator evaluator = new MulticlassClassificationEvaluator()
        .setLabelCol("label")
        .setPredictionCol("prediction")
        .setMetricName("accuracy");
double accuracy = evaluator.evaluate(predictions);
System.out.println("Test set accuracy = " + accuracy);
```

These examples demonstrate more complex and advanced scenarios in Apache Spark for Java, including custom aggregation functions in Spark SQL, stateful operations in Spark Streaming, and feature engineering with MLlib

They showcase the flexibility and power of Apache Spark for handling diverse data processing tasks in a distributed environment.

