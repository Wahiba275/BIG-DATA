# BIG-DATA
## HDFS Commands
1. ***Version***
   ``
   hadoop version
   ``

2. ***Start HDFS***
   ```
    start-dfs.sh
    start-yarn.sh
    jps
   ```

3. ***Create directory /enset/sdia***
   ```
   hdfs dfs -mkdir /enset
   hdfs dfs -mkdir /enset/sdia
   ```

4. ***Create File***
   ``
   hdfs dfs –touchz /enset/sdia/File1
   ``

5. ***Display the content of directory***
   ``
   hdfs dfs –ls
   ``

6. ***Display the content of a File***
   ``
   hdfs dfs -cat [path_file]
   ``

7. ***Copy Files in HDFS***
   ``
   hdfs dfs -cp [-f] [HDFS src] [HDFS dest]
   ``

8. ***Move Files in HDFS***
   ``
   hdfs dfs -mv [HDFS src] [HDFS dest]
   ``

9. ***Remove Files in HDFS***
   ``
   hdfs dfs -rm [-skipTrash] [Path]
   ``

10. ***Remove Files and Directories Recursively in HDFS***
    ``
    hdfs dfs -rmr [-skipTrash] [path]
    ``

11. ***Empties the trash in HDFS***
    ``
    hdfs dfs -expunge
    ``
12. ***Display Disk Usage of a Path in HDFS***
    ``
    hdfs dfs -du [path]
    ``
13. ***Copy/move from Local to HDFS***
    ```
    hdfs dfs -copyFromLocal [-f] [local src] [HDFS dst]
    hdfs dfs -moveFromLocal [local src] [HDFS dst]
    ````

15. ***Copy from HDFS to Local***
    ``
    hdfs dfs -copyToLocal [HDFS src] [local dst]
    ``
16. ***Put Local Files in HDFS***
    ``
    hdfs dfs -put [local src] [HDFS dst]
    ``
17. ***Get Files from HDFS to Local***
    ``
    hdfs dfs -get [hdfs src] [local dst]
   ``

## Dependencies
- **Spark core**

```java
 <dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-core_2.13</artifactId>
    <version>3.4.1</version>
</dependency>
```
- **Spark SQL**

```java
  <!-- https://mvnrepository.com/artifact/org.apache.spark/spark-sql -->
    <dependency>
        <groupId>org.apache.spark</groupId>
        <artifactId>spark-sql_2.13</artifactId>
        <version>3.4.1</version>
    </dependency>
```
- **Spark MySQL Connector**

```java
    <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.33</version>
    </dependency>
```
- **Spark Streaming**

```java
    <!-- https://mvnrepository.com/artifact/org.apache.spark/spark-streaming -->
    <dependency>
        <groupId>org.apache.spark</groupId>
        <artifactId>spark-streaming_2.13</artifactId>
        <version>3.4.1</version>
    </dependency>
```

- **Mlib**

  ```java
    <dependency>
        <groupId>org.apache.spark</groupId>
        <artifactId>spark-mllib_2.13</artifactId>
        <version>3.5.0</version>
    </dependency>
  ```
## Spark SQL
- **From csv File**
```java
 SparkSession ss = SparkSession.builder().appName("test mysql").master("local[*]").getOrCreate();

        Dataset<Row> data = ss.read().csv("incidents.csv").toDF("id", "titre", "description","service" ,"date" );

        //Afficher le nombre d’incidents par service.
        Dataset<Row> dataset = data.select("service").groupBy("service").count();
        dataset.show();

        //Afficher les deux années où il a y avait plus d’incidents
        Dataset<Row> dataset2=data.groupBy("date").count().orderBy(col("count").desc()).limit(2);
        dataset2.show();

        Dataset<Row> incidentsYear = data.withColumn("year", year(col("date")));

        Dataset<Row> dataset3=incidentsYear.groupBy("year").count().orderBy(col("count").desc()).limit(2);
        dataset3.show();
```
- **From JSON File**
  
```java
 SparkSession ss = SparkSession.builder().appName("test mysql").master("local[*]").getOrCreate();
        // Load JSON data into a DataFrame
        Dataset<Row> jsonData = ss.read().json("Achats.json");

        Dataset<Row> groupedData = jsonData.groupBy("ville").agg(functions.sum("prix").as("revenue"));
        Dataset<Row> top2Villes = groupedData.orderBy(functions.col("revenue").desc()).limit(2);
        top2Villes.show();
        Dataset<Row> dataset = jsonData.filter("ville == 'rabat' ").groupBy("nom").count().filter(functions.col("count").$greater(1));
        dataset.show();
        long dataset1 = jsonData.filter("ville == 'casablanca' ").count();
        System.out.println(dataset1);
```  
- **From DataBase**
  
```java
SparkSession ss = SparkSession.builder().appName("test mysql").master("local[*]").getOrCreate();
        Map<String , String > options = new HashMap< >( ) ;
        options.put( "driver" ,  "com.mysql.cj.jdbc.Driver" );
        options.put( "url" , "jdbc:mysql://localhost:3306/db_aeroport1") ;
        options.put( "user" , "root") ;
        options.put( "password" , "" ) ;

        Dataset<Row> dataset = ss.read().format("jdbc")
                .options(options)
                .option("query" , "SELECT R.id_vol AS ID_VOLS, V.DATEDEPART AS DATEDEPART, COUNT(P.id) AS NOMBRE FROM reservations R \n" +
                        "JOIN vols V ON R.id_vol = V.id JOIN passagers P ON R.id_passager = P.id GROUP BY R.id_vol, V.DATEDEPART ORDER BY V.id")
                .load();
        dataset.show();

        Dataset<Row> dataset1 = ss.read().format("jdbc")
                .options(options)
                .option("query" , " SELECT id AS ID_VOL , DATEDEPART AS DATEDEPART , DATEARRIVEE AS DATEARRIVE\n"+
                        "FROM VOLS \n" +
                        "WHERE DATEDEPART <= CURDATE() AND DATEARRIVEE >= CURDATE()")
                .load();
        dataset1.show();
```
## Spark Streaming
This command sets up a simple network listener on the specified port and waits for incoming connections:
   ``
     nc -lk localhost (port) 
   ``

- **Sockets as a source**

```java
public class App2 {
    public static void main(String[] args) throws InterruptedException {
        SparkConf conf=new SparkConf().setAppName("TP Streaming").setMaster("local[*]");
        JavaStreamingContext sc=new JavaStreamingContext(conf, Durations.seconds(8));
        JavaReceiverInputDStream<String> dStream=sc.socketTextStream("localhost",64256);
        JavaDStream<String> dStreamWord=dStream.flatMap(line-> Arrays.asList(line.split(" ")).iterator());
        //RDDs
        JavaPairDStream<String,Integer> dPairStream=dStreamWord.mapToPair(m-> new Tuple2<>(m,1));
        JavaPairDStream<String,Integer> dStreamWordsCount=dPairStream.reduceByKey((a,b)->a+b);
        dStreamWordsCount.print();
        sc.start();
        sc.awaitTermination();
    }
}
```

- **HDFS as a source**
  
```java
public static void main(String[] args) throws InterruptedException {
        //API DStram
        SparkConf conf=new SparkConf().setAppName("TP Streaming").setMaster("local[*]");
        JavaStreamingContext sc=new JavaStreamingContext(conf, Durations.seconds(8));
        //listening to rep1 if we add a file in rep1 this app will trait it
        JavaDStream<String> dStream=sc.textFileStream("hdfs://localhost:9000/rep1");
        JavaDStream<String> dStreamWord=dStream.flatMap(line-> Arrays.asList(line.split(" ")).iterator());
        //RDDs
        JavaPairDStream<String,Integer> dPairStream=dStreamWord.mapToPair(m-> new Tuple2<>(m,1));
        JavaPairDStream<String,Integer> dStreamWordsCount=dPairStream.reduceByKey((a,b)->a+b);
        dStreamWordsCount.print();
        sc.start();
        sc.awaitTermination();
    }
```
 ## Structured Streaming
 ```java
public static void main(String[] args) throws Exception {
    SparkSession ss=SparkSession.builder().appName("Structured Streaming").master("local[*]").getOrCreate();
    Dataset<Row> inputTable=ss.readStream()
            .format("socket")
            .option("host","localhost")
            .option("port",64256)
            .load();
    Dataset<String> words=inputTable.as(Encoders.STRING())
            .flatMap((FlatMapFunction<String,String>)line-> Arrays.asList(line.split(" ")).iterator(),Encoders.STRING() );
    Dataset<Row> resultTable=words.groupBy("value").count();
    StreamingQuery query=resultTable.writeStream()
            .format("console")
            .outputMode("complete") //pour utiliser complete if faut utiliser une aggregation dans query
            //append, complete, update
            .start();
    query.awaitTermination();
    }
```

```java
 SparkSession ss = SparkSession.builder().appName("Hospital incidents streaming app").master("local[*]").getOrCreate();
        StructType schema = new StructType(
                new StructField[]{
                        new StructField("id", DataTypes.LongType, false, Metadata.empty()),
                        new StructField("titre", DataTypes.StringType, false, Metadata.empty()),
                        new StructField("description", DataTypes.StringType, false, Metadata.empty()),
                        new StructField("service", DataTypes.StringType, false, Metadata.empty()),
                        new StructField("date", DataTypes.StringType, false, Metadata.empty()),
                }
        );
        Dataset<Row> incidents = ss
                .readStream()
                .option("header", "true")
                .schema(schema)
                .csv("src/main/resources");

        // compter le nombre d'incidents par service
        Dataset<Row> incidentsCountByService = incidents.groupBy("service").count();
       StreamingQuery query = incidentsCountByService
                .writeStream()
                .outputMode("update")
                .format("console")
                .trigger(Trigger.ProcessingTime("5000 milliseconds"))
                .start();
     //la fin du streaming
        query.awaitTermination();

```
## pyspqrk
- **From json File**
  
  ```python
   from pyspark.sql import SparkSession
   from pyspark.sql import functions as F
   spark=SparkSession.builder.appName("TP PySpark").master("local[*]").getOrCreate()
   dfAchats=spark.read.json("achats.json")
   dfAchats=spark.read.option("multiline",True).json("achats.json")
   total_revenues_per_city = dfAchats.groupBy("ville").agg(F.sum("prix"))
   total_revenues_per_city.show()
   ```

- **From DataBase**
  - ***Start pyspark***
    
    ``bash
    pyspark --jars "/home/bouzyan/mysql-connector-java-8.0.13.jar "
    ``
    
    ```python
       from pyspark.sql import SparkSession
       df1-spark.read.format("jdbc").option("driver","com.mysql.jdbc.Driver").option("url","jdbc:mysql://localhost:3306/DB_spark").option("user",
      "root").option("password"," ").option("query","select from projets WHERE DATE_FIN > = CURRENT_DATE").load()
       df1.show ( )
    ```
## Machine Learning with Spark

- **KMeans**
  
  ```java
      KMeans kMeans=new KMeans().setK(3)
            .setSeed(123)
            .setFeaturesCol("normalizedFeatures")
            .setPredictionCol("Cluster");
    KMeansModel model=kMeans.fit(normalizedDF);
    Dataset<Row> predictions=model.transform(normalizedDF);
    predictions.show(100);
    ClusteringEvaluator evaluator=new ClusteringEvaluator();
    double score=evaluator.evaluate(predictions);
    System.out.println("Score "+score);

    ```
  
- **Linear Regression**
  
```java
  SparkSession ss= SparkSession.builder().appName("Tp spark ml").master("local[*]").getOrCreate();
        Dataset<Row> dataset =ss.read().option("inferSchema",true).option("header",true).csv("advertising.csv");
        VectorAssembler assembler=new VectorAssembler().setInputCols(new String[]{"TV","Radio","Newspaper"}
        ).setOutputCol("Features");
        Dataset<Row> assembledDS=assembler.transform(dataset);
        Dataset<Row> splits[]=assembledDS.randomSplit(new double[]{0.8,0.2},123);
        Dataset<Row> train=splits[0];
        Dataset<Row> test=splits[1];

        LinearRegression lr=new LinearRegression().setLabelCol("Sales").setFeaturesCol("Features");
        LinearRegressionModel model=lr.fit(train);
        Dataset<Row> prediction=model.transform(test);
        prediction.show();
        System.out.println("Intercept: "+model.intercept()+ " coef: "+model.coefficients());
    
```
## Spark with Docker
- **Commande to generate an image**
     ``
     docker pull bitnami/spark
  ``
- **Command to run the jar**
  ``
docker exec -it spark-master spark-submit --class org.example.App2 /bitnami/Spark_Docker-1.0-SNAPSHOT.jar
  ``
  
## SQOOP
- **Start Xampp**
  
``
  :/opt/lampp$ sudo ./manager-linux-x64.run
``

- **Import**
  
```
  sqoop import --connect jdbc:mysql://localhost/spark_db --username "root" --password "" --table employees --target-dir /sqoop {sqoop file in hdfs}
```

- **Export**
  
```
 sqoop export --connect jdbc:mysql://localhost/spark_db --username "root" --password "" --table "employees" --export-dir "/sqoop_data" --input-fields-terminated-by ',' --input-lines-terminated-by '\n'
```












   
