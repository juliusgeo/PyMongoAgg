PyMongoAgg
==========

Minimal POC of a Python-MongoDB Aggregation transpiler.

Have you ever, like me, had to write a MongoDB aggregation expression, and found that you
would rather put your head underneath a car tire and ask the driver to do a burnout?
Well, this is the solution for you. Now you can write your aggregation pipelines using native Python syntax,
and PyMongoAgg will handle conversion by parsing the AST of your function.

This was originally a Skunkworks project (MongoDB's internal hackathon), however I was let go summarily only
2 weeks after my project got to the finals. Luckily, the project has a very permissive license, and I will be continuing
development work by myself.

The version that MongoDB turned into an official repository is not maintained: https://github.com/mongodb-labs/PyMongoAgg

Example:
```python
def basic_func():
    y = a
    a = (a + b) / 2
    b = sqrt(b * y)
    t = t - (x * (y - a) ** 2)
    x = x * 2
    ...
output_dict = transpile_function(basic_func)
print(output_dict)
```
```pycon
[{'$set': {'y': '$a'}}, 
 {'$set': {'a': {'$divide': [{'$add': ['$a', '$b']}, 2]}}},
 {'$set': {'b': {'$sqrt': [{'$multiply': ['$b', '$y']}]}}},
 {'$set': {'t': {'$subtract': ['$t', {'$multiply': ['$x', {'$pow': [{'$subtract': ['$y', '$a']}, 2]}]}]}}},
 {'$set': {'x': {'$multiply': ['$x', 2]}}}]
```
It also supports boolean operations!
```python
def bool_func(y, a, b, c):
    y = 1
    a = (y and 0) and 1
    b = y or 0
    c = not y
    ...
```
```pycon
[{'$set': {'y': 1}}, 
 {'$set': {'a': {'$and': [{'$and': ['$y', 0]}, 1]}}}, 
 {'$set': {'b': {'$or': ['$y', 0]}}},
 {'$set': {'c': {'$not': ['$y']}}}]
```
Changes since original Skunkworks presentation:

  * Added support for builtins like `sqrt`
  * Added support for boolean operations
  * Added tests that run the transpiled functions and checks the result
    of running the functions against the result of executing the transpiled MongoDB aggregation.

To-do:
  * Control flow
  * Vector operations
  * More aggregation operators

