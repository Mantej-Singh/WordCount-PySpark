# PySpark- WordCount, under the hood!
Running my first pyspark app in CDH5
___

## PySpark Job in Spark UI
[![screenshot_1509487656.png](https://s19.postimg.org/4sxl9ku9v/screenshot_1509487656.png)](https://postimg.org/image/clo91k08v/)

### Since Spark follows *Lazy Evaluation* it creates a single **action** job in the end. It waits until an action is called.


## Details for 1st Job

[![screenshot_1509488132.png](https://s19.postimg.org/b7wm67yer/screenshot_1509488132.png)](https://postimg.org/image/e1zrjo0kv/)

So this **Job** was further divided into two stages, one for Stage0: reduceByKey and the other for Stage1: collect(our last code).
We used reduceByKey in it,so in future try to minimize the number of shuffles and the amount of data shuffled.
Shuffles are expensive operations; all shuffle data must be written to disk and then transferred over the network.


## DAG for Stage1: Collect

[![screenshot_1509492181.png](https://s19.postimg.org/iy7zoqnfn/screenshot_1509492181.png)](https://postimg.org/image/9qfr81gdb/)


[![screenshot_1509488721.png](https://s19.postimg.org/tq0yqct0j/screenshot_1509488721.png)](https://postimg.org/image/lxaaydn1b/)

This clearly show a PythonRDD object is a **scala** object under the hood. It reads/takes the 2.1 KB of shuffled data, **map** it together and returns a **RDD**.


___
___

# Method 2:
### Using ***.reduceByKey(add)*** instead of *.reduceByKey(lambda w1,w2: w1 + w2)*

```python
from operator import add

count=myfile.flatMap(lambda line: line.split(" ")).map(lambda word: (word,1)).reduceByKey(add)

count.collect()

```


### Later I tried to run it multiple times, with duration reducing in each run:
[![screenshot_1509569301.png](https://s19.postimg.org/ubvg2syfn/screenshot_1509569301.png)](https://postimg.org/image/b6s6t1jrj/)

### What if I skip the reduceByKey() and jump on to collect()
[![screenshot_1509568027.png](https://s19.postimg.org/korxsu5df/screenshot_1509568027.png)](https://postimg.org/image/3o91k5sbz/)

### It will skip the reducing part from DAG and read the shuffled data and displays output
[![screenshot_1509568064.png](https://s19.postimg.org/nvmhclfkj/screenshot_1509568064.png)](https://postimg.org/image/77uza3ksv/)

___
___

# Method 3: reduceByKey VS groupByKey

## reduceByKey()
```python

count_reduceByKey=myfile.flatMap(lambda line: line.split(" ")).map(lambda word: (word,1)).reduceByKey(lambda w1,w2: w1 + w2)

count_reduceByKey.collect()

```
### How it works:
[![screenshot_1509983710.png](https://s19.postimg.org/vb66b4nk3/screenshot_1509983710.png)](https://postimg.org/image/pa8he20xr/)

## VS 


## groupByKey()

```python

count_groupByKey=myfile.flatMap(lambda line: line.split(" ")).map(lambda word: (word,1)).groupByKey().map(lambda (word,count): (word,sum(count)))

count_groupByKey.collect()

```

### How it works:
[![screenshot_1509983773.png](https://s19.postimg.org/mg5c0q937/screenshot_1509983773.png)](https://postimg.org/image/8mgzboghr/)


## Comparing their performance, under the hood: 

[![screenshot_1509988947.png](https://s19.postimg.org/xc5wnqvzn/screenshot_1509988947.png)](https://postimg.org/image/o4do71oxb/)
In Saprk UI under the **Stages** tab you can chekc the read/write performed by our job.
+ This **Job** is divided in to two **Stages**
+ Stage0 for groupByKey()
+ Stage1 for collect()
+ They both shuffled **2.5 KB** ko data, as compared to reduceBYKey() **2.1 KB**


___

# Method 4: Sorting and Filtering:
```python

textbook=sc.textFile('hdfs://devtoolsRnD/data/everyone/work/hive/SparkCookbook.txt')

textbook.flatMap(lambda line: line.split(" ")).map(lambda word: (word,1)).reduceByKey(add).sortBy(lambda x: (x[1]),ascending=False).filter(lambda x:x[1]>600).collect()

```
[(u'the', 2516),
 (u'to', 1157),
 (u'a', 928),
 (u'is', 896),
 (u'of', 895),
 (u'and', 782),
 (u'in', 624)]

___
___
___
## References: 
### 1. A good defination for flatMap I found on [Stackoverflow](https://stackoverflow.com/questions/22350722/can-someone-explain-to-me-the-difference-between-map-and-flatmap-and-what-is-a-g "Stack Overflow page")

### 2. reduceByKey VS groupByKey [Databricks](https://databricks.gitbooks.io/databricks-spark-knowledge-base/content/best_practices/prefer_reducebykey_over_groupbykey.html "Avoid GroupByKey")

### 3. For Kerberos TGT [kinit this!](http://www.roguelynn.com/words/explain-like-im-5-kerberos/ "Nice article, trust me")

### 4. Spark Framework:
[![screenshot_1509568572.png](https://s19.postimg.org/z990npq2b/screenshot_1509568572.png)](https://postimg.org/image/5hby8j38v/)

### 5. Helpful Youtube video:
[![Spark Architecture](https://s19.postimg.org/6wdixj6zn/screenshot_1509568910.png)](https://www.youtube.com/watch?v=wzy0oluoyN8)
