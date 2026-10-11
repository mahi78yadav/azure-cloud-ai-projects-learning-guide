# Python and PySpark Fundamentals for High School Students

## Beginner material for the Azure Databricks Retail Lakehouse

This guide starts with ordinary Python and builds toward PySpark
DataFrames, Delta Lake, Auto Loader, and the libraries commonly used in
Databricks ETL work. Examples are illustrative; confirm paths, column
names, table names, and write strategies against your project notebooks.

## Learning path

1.  Python basics: variables, types, operators, strings, input
2.  Conditions: `if`, `elif`, `else`
3.  Loops: `for`, `while`, `break`, `continue`
4.  Collections: lists, tuples, sets, dictionaries
5.  Functions and imports
6.  Pandas versus PySpark DataFrames
7.  Common PySpark methods and functions
8.  Databricks libraries and project exercises

# Part 1 --- Basic Python

## 1. Hello world and variables

``` python
print("Hello, world!")

student_name = "Ravi"
student_age = 16
marks = 85.5
is_present = True

print(student_name, student_age, marks, is_present)
print(type(student_name))
print(type(student_age))
```

  Type         Meaning        Example
  ------------ -------------- ----------
  `str`        Text           `"Ravi"`
  `int`        Whole number   `16`
  `float`      Decimal        `85.5`
  `bool`       True/false     `True`
  `NoneType`   No value       `None`

Python is case-sensitive: `age` and `Age` are different names.

## 2. Operators

``` python
a = 10
b = 3

print(a + b)   # 13
print(a - b)   # 7
print(a * b)   # 30
print(a / b)   # 3.333...
print(a // b)  # 3, floor division
print(a % b)   # 1, remainder
print(a ** b)  # 1000, power

print(10 > 5)
print(10 == 10)
print(10 != 3)
print(4 <= 8)
```

`=` assigns a value; `==` compares two values.

Logical operators:

``` python
age = 16
has_id = True

print(age >= 13 and has_id)
print(age < 13 or has_id)
print(not has_id)
```

## 3. Strings and input

``` python
name = "Ravi"
city = "Hyderabad"

print(name.upper())
print(name.lower())
print(name.strip())
print(len(name))
print(f"My name is {name} and I live in {city}.")
```

Useful methods: `.strip()`, `.lower()`, `.upper()`, `.replace()`,
`.split()`. `len(text)` counts characters.

``` python
name = input("Enter your name: ")
age = int(input("Enter your age: "))
print(f"Hello {name}; next year you will be {age + 1}.")
```

`input()` returns text. Use `int()` or `float()` for numeric conversion;
invalid text can cause an error.

# Part 2 --- Conditions

## 4. if, elif, else

``` python
marks = 78

if marks >= 90:
    print("Grade A")
elif marks >= 75:
    print("Grade B")
elif marks >= 50:
    print("Grade C")
else:
    print("Needs more practice")
```

Python uses indentation to define a block. Remember the colon `:` after
each condition.

Example:

``` python
age = 12

if age < 5:
    price = 0
elif age < 18:
    price = 50
else:
    price = 100

print("Ticket price:", price)
```

Common mistakes: missing colons, inconsistent indentation, using `=`
instead of `==`, or checking conditions in the wrong order.

# Part 3 --- Loops

## 5. for loops

``` python
subjects = ["Maths", "Science", "English"]

for subject in subjects:
    print(subject)

for number in range(1, 6):
    print(number)  # 1 through 5; end is excluded
```

Calculate an average:

``` python
marks = [80, 75, 92, 68]
total = 0

for mark in marks:
    total += mark

print("Total:", total)
print("Average:", total / len(marks))
```

## 6. while loops

``` python
count = 1

while count <= 5:
    print(count)
    count += 1
```

Make sure the condition eventually becomes false to avoid an infinite
loop.

## 7. break and continue

``` python
for number in range(1, 8):
    if number == 5:
        break
    print(number)
```

`break` exits the loop.

``` python
for number in range(1, 6):
    if number == 3:
        continue
    print(number)
```

`continue` skips the current iteration.

# Part 4 --- Collections

## 8. Lists

Lists are ordered, changeable, and can contain duplicates.

``` python
fruits = ["apple", "banana", "mango"]
print(fruits[0])   # First item
print(fruits[-1])  # Last item

fruits.append("grapes")
fruits[1] = "orange"
fruits.remove("apple")
print(fruits)
print(len(fruits))
```

  Method            Purpose
  ----------------- -----------------------------
  `append(x)`       Add one item
  `extend(items)`   Add several items
  `insert(i, x)`    Insert at index
  `remove(x)`       Remove first matching value
  `pop()`           Remove and return an item
  `sort()`          Sort in place
  `count(x)`        Count occurrences
  `index(x)`        Find first matching index
  `len(items)`      Count items

Indexes start at zero.

## 9. Tuples

Tuples are ordered but cannot be changed after creation.

``` python
student = ("Ravi", 16, "Class 10")
print(student[0])
print(student[1])
print(len(student))

coordinates = (17.385, 78.486)
latitude, longitude = coordinates

single_value = (10,)  # Comma makes this a one-item tuple
```

Common operations: indexing, slicing, `len()`, `count()`, and `index()`.

## 10. Sets

Sets contain unique values and do not support positional indexing.

``` python
class_a = {"Ravi", "Anu", "Sara", "Ravi"}
class_b = {"Anu", "John", "Maya"}

print(class_a)  # Ravi appears only once
class_a.add("Kiran")
class_a.discard("Sara")

print(class_a.union(class_b))
print(class_a.intersection(class_b))
print(class_a.difference(class_b))
```

  Method                  Purpose
  ----------------------- --------------------------
  `add(x)`                Add an item
  `discard(x)`            Remove if present
  `remove(x)`             Remove; errors if absent
  `union(other)`          Items in either set
  `intersection(other)`   Common items
  `difference(other)`     Items only in first set
  `issubset(other)`       Test subset relationship

Do not rely on the display order of a set.

## 11. Dictionaries

Dictionaries store key-value pairs.

``` python
student = {"name": "Ravi", "age": 16, "marks": 85}

print(student["name"])
student["city"] = "Hyderabad"
student["marks"] = 90

print(student.keys())
print(student.values())
print(student.items())
print(student.get("phone"))  # None if key is absent
```

Useful methods: `get()`, `keys()`, `values()`, `items()`, `update()`,
`pop()`.

## 12. Collection comparison

  ---------------------------------------------------------------------------
  Collection     Ordered?       Changeable?    Duplicates?    Example
  -------------- -------------- -------------- -------------- ---------------
  List           Yes            Yes            Yes            File names

  Tuple          Yes            No             Yes            Fixed
                                                              coordinates

  Set            No positional  Yes            No             Unique IDs
                 order                                        

  Dictionary     Insertion      Yes            Keys unique    Configuration
                 order                                        
                 preserved                                    
  ---------------------------------------------------------------------------

# Part 5 --- Functions, imports, and standard libraries

## 13. Functions

``` python
def calculate_total(price, quantity):
    return price * quantity

amount = calculate_total(250, 3)
print(amount)
```

`def` defines a function; parameters receive inputs; `return` sends a
result back.

## 14. Standard-library modules

``` python
import math
print(math.sqrt(81))

from datetime import date
print(date.today())
```

  Module          Common use
  --------------- -------------------------------
  `datetime`      Dates and times
  `json`          JSON data
  `os`            Operating-system interactions
  `re`            Pattern matching
  `logging`       Application logs
  `collections`   `Counter`, `defaultdict`
  `pathlib`       Local filesystem paths
  `math`          Mathematical functions

# Part 6 --- Pandas and PySpark DataFrames

## 15. What is a DataFrame?

A DataFrame is table-like data with named columns.

-   **Pandas DataFrame:** usually works in local Python memory on one
    machine.
-   **PySpark DataFrame:** represents data Spark can process in a
    distributed environment.

## 16. Small Pandas example

``` python
import pandas as pd

data = {
    "product": ["Pen", "Book", "Bag"],
    "price": [10, 50, 500]
}

pdf = pd.DataFrame(data)
print(pdf)
print(pdf["price"].mean())
```

## 17. PySpark DataFrame example

Run this in Databricks or a configured Spark environment:

``` python
from pyspark.sql import functions as F

data = [("Pen", 10), ("Book", 50), ("Bag", 500)]
columns = ["product", "price"]

sdf = spark.createDataFrame(data, columns)
sdf.show()
sdf.printSchema()
sdf.filter(F.col("price") >= 50).show()
sdf.groupBy().agg(F.avg("price").alias("average_price")).show()
```

In Databricks, `spark` is generally already available. It is not
automatically defined in an ordinary Python interpreter.

## 18. Common PySpark DataFrame methods

  Method/property                     Typical purpose
  ----------------------------------- -------------------------------
  `spark.read.csv()`                  Read CSV
  `spark.read.parquet()`              Read Parquet
  `spark.table()`                     Read registered table
  `show()`                            Display rows
  `printSchema()`                     Inspect schema
  `select()`                          Select columns
  `filter()` / `where()`              Filter rows
  `withColumn()`                      Add or replace a column
  `withColumnRenamed()`               Rename a column
  `drop()`                            Drop columns
  `dropDuplicates()`                  Remove duplicate rows or keys
  `fillna()` / `dropna()`             Handle null values
  `join()`                            Join DataFrames
  `groupBy()` / `agg()`               Group and aggregate
  `orderBy()`                         Sort rows
  `limit()`                           Limit output
  `count()`                           Count rows
  `distinct()`                        Return distinct rows
  `unionByName()`                     Combine by column names
  `repartition()`                     Change partitioning
  `write.format(...).saveAsTable()`   Write a table
  `write.parquet()`                   Write Parquet

Note: `DataFrame.coalesce(n)` changes the number of partitions.
`pyspark.sql.functions.coalesce(...)` returns the first non-null
expression; they are different operations.

## 19. Common PySpark functions

Import:

``` python
from pyspark.sql import functions as F
```

  Function                            Use
  ----------------------------------- ----------------------------------
  `F.col()`                           Reference column
  `F.lit()`                           Literal value
  `F.when()`                          Conditional expression
  `F.coalesce()`                      First non-null expression
  `F.trim()`                          Trim text
  `F.lower()` / `F.upper()`           Change text case
  `F.concat()`                        Combine expressions
  `F.split()`                         Split text
  `F.regexp_replace()`                Regex text replacement
  `F.to_date()`                       Convert to date
  `F.to_timestamp()`                  Convert to timestamp
  `F.current_date()`                  Current date
  `F.current_timestamp()`             Current timestamp
  `F.sum()` / `F.avg()`               Sum / average
  `F.count()` / `F.countDistinct()`   Count values
  `F.min()` / `F.max()`               Minimum / maximum
  `F.round()`                         Round numeric values
  `F.isnull()` / `F.isnan()`          Test null / NaN where applicable

PySpark transformations return new DataFrames rather than modifying the
original DataFrame in place.

# Part 7 --- Libraries and classes commonly used in Databricks

There is no universal list that represents exactly 80% of every
Databricks codebase. The list below is a practical priority list for
common PySpark ETL and lakehouse projects.

## 20. Core: `pyspark.sql`

Imports:

``` python
from pyspark.sql import SparkSession, DataFrame
from pyspark.sql import functions as F
from pyspark.sql.types import (
    StructType, StructField, StringType, IntegerType,
    LongType, DoubleType, DecimalType, DateType,
    TimestampType, BooleanType
)
```

  -----------------------------------------------------------------------
  Class/module                        Important APIs
  ----------------------------------- -----------------------------------
  `SparkSession`                      `read`, `readStream`, `table`,
                                      `sql`, `createDataFrame`, `catalog`

  `DataFrame`                         `select`, `filter`, `withColumn`,
                                      `join`, `groupBy`, `agg`, `show`,
                                      `count`, `write`, `writeStream`

  `functions` as `F`                  `col`, `lit`, `when`, `sum`, `avg`,
                                      `count`, `trim`, `to_date`

  `StructType`                        Build schema

  `StructField`                       Define field name, type, and
                                      nullability

  Data types                          `StringType`, `IntegerType`,
                                      `LongType`, `DoubleType`,
                                      `DecimalType`, `DateType`,
                                      `TimestampType`, `BooleanType`
  -----------------------------------------------------------------------

## 21. Window functions

Import:

``` python
from pyspark.sql.window import Window
from pyspark.sql import functions as F

window_spec = Window.partitionBy("customer_id").orderBy(
    F.col("updated_at").desc()
)
```

Common functions: `row_number()`, `rank()`, `dense_rank()`, `lag()`, and
`lead()`. They help with latest-record selection, ranking, and running
calculations.

## 22. Delta Lake

Advanced Delta APIs may be available through:

``` python
from delta.tables import DeltaTable
```

Common APIs and concepts:

-   `DeltaTable.forName(...)`
-   `DeltaTable.forPath(...)`
-   `history()`
-   `detail()`
-   `alias()`
-   `merge(...)`
-   `whenMatchedUpdateAll()`
-   `whenNotMatchedInsertAll()`
-   `execute()`

Availability depends on the installed Delta Lake/Databricks Runtime
version. SQL `MERGE INTO` is also common.

## 23. Supporting libraries

  Library                Typical use                                 Priority
  ---------------------- ------------------------------------------- -----------------------
  `datetime`             Dates and times                             Core Python
  `json`                 JSON configuration/payloads                 Common
  `logging`              Application logs                            Common
  `re`                   Pattern matching                            Useful
  `collections`          `Counter`, `defaultdict`                    Useful
  `pandas`               Small local datasets and interoperability   Situational
  `numpy`                Numeric arrays                              Situational
  `pyarrow`              Columnar data interoperability              Situational
  `requests` / `httpx`   HTTP API calls                              API-dependent
  `pytest`               Automated tests                             Testing
  `pydantic`             Data/config validation                      Application-dependent

Do not install every library listed. Follow the dependencies and
policies of your actual project.

## 24. Auto Loader and Databricks utilities

Auto Loader is a Databricks feature accessed through Spark streaming
APIs; it is not a standard Python class to import.

Typical interfaces:

-   `spark.readStream`
-   `.format("cloudFiles")`
-   `.option("cloudFiles.format", "...")`
-   `.option("cloudFiles.schemaLocation", "...")`
-   `.writeStream`
-   `.option("checkpointLocation", "...")`
-   `.toTable(...)`

In supported Databricks notebooks, `dbutils` is often available as a
utility object:

  -----------------------------------------------------------------------
  Namespace                           Use
  ----------------------------------- -----------------------------------
  `dbutils.widgets`                   Notebook parameters

  `dbutils.fs`                        Supported filesystem operations

  `dbutils.secrets`                   Authorized secret access

  `dbutils.notebook`                  Notebook workflow utilities in
                                      supported contexts
  -----------------------------------------------------------------------

Availability varies by compute type and runtime. For production
orchestration, Databricks Workflows is generally preferred over complex
orchestration inside notebooks.

Azure SDK libraries are only needed when code directly calls Azure
services through those SDKs. Accessing tables through an already
configured Unity Catalog external location does not automatically
require a separate SDK.

# Part 8 --- Python versus PySpark

## 25. List versus DataFrame

``` python
prices = [10, 50, 500]
print(sum(prices))
```

PySpark version:

``` python
price_df = spark.createDataFrame(
    [(10,), (50,), (500,)],
    ["price"]
)

price_df.agg(F.sum("price").alias("total")).show()
```

Use lists for small collections and normal Python logic. Use PySpark
DataFrames for distributed table transformations.

## 26. Loops versus DataFrame operations

A Python loop is suitable for a small list:

``` python
for name in ["customers", "products", "orders"]:
    print(name)
```

For distributed data, prefer Spark expressions:

``` python
orders_df.filter(F.col("order_amount") > 100).show()
```

Avoid collecting a large dataset to Python and looping over every row.
It can be slow and memory-intensive.

# Part 9 --- Practice exercises

Use `customers.csv`, `products.csv`, `orders.csv`, and `stores.csv`.
Replace placeholder paths and field names with the actual values.

1.  Read customers; display five rows, print schema, and count records.
2.  Trim customer names; find null customer IDs and duplicate keys.
3.  Read orders; calculate total amount and sales by product.
4.  Join orders with customers; compare counts before and after the
    join.
5.  Find orders without a matching customer.
6.  Write a test DataFrame to a separate Delta table and compare append
    with overwrite safely.
7.  Inspect Delta history.
8.  Identify the source path, schema location, checkpoint, and
    destination in the Auto Loader notebook.
9.  Validate counts, mandatory fields, duplicates, and entity
    relationships.
10. Explain which concepts are used in each of the five project
    notebooks.

# Part 10 --- Beginner and interview questions

1.  What does `print()` do?
2.  What is a variable?
3.  Difference between `=` and `==`?
4.  What does an `if` statement do?
5.  Difference between `for` and `while`?
6.  Difference between lists and tuples?
7.  Why do sets remove duplicates?
8.  What is a dictionary?
9.  What is a function?
10. What is a Python library?
11. What is a DataFrame?
12. Difference between Pandas and PySpark DataFrames?
13. What does `filter()` do?
14. What is a join?
15. What does `groupBy()` do?
16. Difference between a transformation and an action?
17. What is lazy evaluation?
18. What is Delta Lake?
19. What is Auto Loader?
20. Why do streaming jobs use checkpoints?

# Suggested study sequence

1.  Python variables, types, operators, strings, conditions, loops,
    collections, and functions.
2.  DataFrames, schemas, reading files, selecting, filtering, and
    creating columns.
3.  Null handling, duplicate detection, joins, and aggregations.
4.  Delta tables and write modes.
5.  Bronze/Silver/Gold notebook walkthrough.
6.  Auto Loader, streaming, and checkpoints.
7.  Data-quality checks and troubleshooting.

Advanced RDDs, custom UDFs, complex Spark internals, and advanced tuning
can wait until they are needed.

## Official documentation

-   Python tutorial: https://docs.python.org/3/tutorial/
-   PySpark API:
    https://spark.apache.org/docs/latest/api/python/index.html
-   Databricks PySpark: https://docs.databricks.com/aws/en/pyspark/
-   Auto Loader:
    https://docs.databricks.com/aws/en/ingestion/cloud-object-storage/auto-loader/

The Databricks links above are in the AWS documentation section; consult
equivalent Azure Databricks documentation for cloud-specific
configuration.

## Completion goal

You are ready to read the project notebooks when you can write a small
Python program using variables, conditions, loops, collections, and
functions; create and transform a PySpark DataFrame; join two
DataFrames; write a Delta table; and validate the result.
