
# RDD Transformations 2

## Objectives
* Perform data retrieval from RDDs for exploratory purpose.
* Understand the use of `first()`, `take()` and `top()` for retrieving data from RDDs.
* Use `reduce()` action to reduce elements of an RDD to a single value. 
* Sample data from RDDs using `takeSample()` and `countByValue()`

### Introduction
This lab introduces more RDD transformations for selecting and retrieving data from distributed RDDs. Let's first import the code from our previous lab to re-create the `filteredRDD` and use `collect()` to view its contents. 


```python
# Include all the code from previous lab to create FilteredRDD and view the contents. 

import pyspark
sc = pyspark.SparkContext('local[*]')

def subtract(value):
    return (value - 1)

def lessThanTen(value):
    if (value < 10):
        return True
    else:
        return False
    
intRDD = sc.parallelize(range(1,1001),5)
subRDD = intRDD.map(subtract)
filteredRDD = subRDD.filter(lessThanTen)
filteredRDD.collect()

# [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

#### Bonus 
>You can also join a number of transformations to get the similar output as shown below. All the code from previous lab can be summarized in a neat manner as a one-liner. However, it is imperative that you must understand the function of each transformation and action before you try this, as these statements would be very difficult to debug.


```python
# One-liner for a multiple transformations

sc.parallelize(range(1,1001),5).map(subtract).filter(lessThanTen).collect()
```




    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]



### `first()` vs. `take()` vs. `top()` actions

A very step into exploratory analysis is always to look at first few rows of the data to get an idea about the features of the dataset and datatypes. in pandas this is achoeved by calling `.head()`. A similar approach can be adopted while working with Spark. We have `first()`, `take()`, `top()` and `takeordered()` actions. 

**Remember**, while using `first()` and `take()`, the output depends upon the partitioning of dataset. 

Instead of using `collect()` action, if we are interested in taking only the first element of an RDD, we can use `first()` action. `take(n)` perform similar action where first n elements of an RDD are returned. 

Let's use these actions on the RDDs that we created in the previous lab. 
1. get the first element of filteredRDD
2. Get the first 3 elements of filteredRDD



```python
# Get the first element of filteredRDD
filteredRDD.first()

# 0
```




    0




```python
# Get first three elements of filteredRDD
filteredRDD.take(3)

# [0, 1, 2]
```




    [0, 1, 2]



Let's also see what happenes if we try to take more elements than RDD has. An error maybe! 


```python
# TRy taking more elements than RDD has
filteredRDD.take(20)

# [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```




    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]



So it is okie to take more elements than the length of an RDD. It will return all the elements of RDD without generating an error. 

### `takeOrdered()` action

The `takeOrdered()` action returns the first n elements of the RDD, using either their natural order or some custom criteria. The key advantage of using `takeOrdered()` instead of `first()` or `take()` is that `takeOrdered()` returns a deterministic result, while the other two actions may return differing results, depending on the number of partions or execution environment. `takeOrdered()` returns the list sorted in ascending order. The `top()` action is similar to `takeOrdered()` except that it returns the list in descending order.

Let's use use these to observe the output from the filteredRDD

1. Retrieve three smallest elements of filteredRDD
2. Retrieve three largest elements of filteredRDD
3. Use a lambda function with `takeOrdered()` to reverse the output (i.e. get the top elements) 



```python
# Retrieve three smallest elements of filteredRDD
filteredRDD.takeOrdered(3)

# [0, 1, 2]
```




    [0, 1, 2]




```python
# Retrieve the three largest elements
filteredRDD.top(3)

# [9, 8, 7]
```




    [9, 8, 7]




```python
# Use a lambda function with `takeOrdered()` to reverse the order of output
print (filteredRDD.takeOrdered(3, lambda s: -s)) 

# [9, 8, 7]
```

    [9, 8, 7]
    

We can see that the output from `takeOrdered()`, when reversed, is exactly the same as the `top()` action. 

### `reduce()` action

The `reduce()` action reduces the elements of a RDD to a single value by applying a function that takes two parameters and returns a single value.

> **NOTE:** The function used with `reduce()` action should be **commutative** ( A+B = B+A ) and **associative** ((A+B)+C = A+(B+C)), as `reduce()` is applied at the partition level and then again to aggregate results from partitions.

If commutative and associative propoerties of the function don't hold, the results from `reduce()` will be inconsistent. Reducing locally at partitions makes `reduce()` very effective. 

Let's try to apply `reduce()` action with a simple `add` operator from Python's `operator` module. 



```python
# Obtain Python's add function
from operator import add
```

1. Apply `add` function with `reduce()` action to sum all the elements of filteredRDD
2. Add numbers using a simple lambda function (a+b) with `reduce()` to add all elements sequentially. Confirm that a+b = b+a
3. Repeat 2 by changing addition to subtraction within lambda function and comment on the results. 


```python
# Sum the RDD using reduce() with imported add method
filteredRDD.reduce(add)

# 45
```




    45




```python
# Sum using reduce with a lambda function and check the output of a+b vs. b+a
filteredRDD.reduce(lambda a, b: a + b), filteredRDD.reduce(lambda a, b: b + a)

# (45, 45)
```




    (45, 45)



Due to commutative and associative property of addition, we will observe the same result however we add the elements of RDD. But this assumption may not hold true for subtraction as it is not both commutative and associative. Let's try above lambda function with subtraction to view the effect of `reduce()` on a - b vs. b - a


```python
# Note that subtraction is not both associative and commutative
filteredRDD.reduce(lambda a, b: a - b), filteredRDD.reduce(lambda a, b: b - a)
```




    (-45, 5)



So special care must be taken while using the `reduce()` function. There are a number of workaround for non-commutative, non-associative functions while using `reduce()`. [Here is a good explanation](https://cloudxlab.com/assessment/slide/17/apache-spark-basics/387/apache-spark-reduce-commutative-associative)

### `takeSample()` and `counByValue()` 

These two methods can also be used for retrieving contents of an RDD. 

The `takeSample()` action outputs an array while performing random sampling on the RDD. It also accepts a `withReplacement = True` argument to identify if an item can be sampled multiple times. It uses a `seed` parameter for generating a seed to control the randomness during experimentation, producing *reproducible* results. 

We can apply `takeSample()` with replacement set to True and False to observe the effect. Remember, without setting up a seed, you would get random output.

> **`RDD.takeSample(withReplacement, num, seed)`**


```python
# takeSample reusing elements and set number of samples to 5

filteredRDD.takeSample(withReplacement=True, num=5)

# [4, 3, 7, 9, 8]
```




    [4, 3, 7, 9, 8]




```python
# takeSample not reusing elements and set number of samples to 5

filteredRDD.takeSample(withReplacement=False, num=5)

# [5, 4, 2, 9, 7]
```




    [5, 4, 2, 9, 7]



If we run above cells repeatedly, we will see that the output changes everytime due to uncontrolled randomness. We can control this behaviour by setting a seed value. 


```python
# Set seed for predictability
# Try reruning this cell and the cell above -- the results from this cell will remain constant

filteredRDD.takeSample(withReplacement=False, num=5, seed=200)

# [8, 6, 5, 7, 2]
```




    [8, 6, 5, 7, 2]



Finally, The `countByValue()` action returns the count of each unique value in the RDD as a dictionary that maps values to counts, using `RDD.countByValue()`. 

Using following list, create a new RDD and apply the countByValue() action to get frequency of unique elements. 
[1, 2, 3, 1, 2, 3, 1, 2, 1, 2, 3, 3, 3, 4, 5, 4, 6]



```python
# Create new base RDD to show frequency of values using countByValue and show contents 
testRDD = sc.parallelize([1, 2, 3, 1, 2, 3, 1, 2, 1, 2, 3, 3, 3, 4, 5, 4, 6])
print(testRDD.countByValue())

# defaultdict(<class 'int'>, {1: 4, 2: 4, 3: 5, 4: 2, 5: 1, 6: 1})
```

    defaultdict(<class 'int'>, {1: 4, 2: 4, 3: 5, 4: 2, 5: 1, 6: 1})
    

## Summary

In this lab we saw some more transformations for processing data in RDDs. We looked at actions for exploring data stored in an RDD and for for sampling RDD data. `reduce()` action was also introduced while focusing on its commutative and associative assumptions. In the next lab, we shall discover some advanced transformations before moving on to developing some real applications using all these transformations and actions. 
