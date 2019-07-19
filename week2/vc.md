

```python
%matplotlib inline
import matplotlib
import seaborn as sns
matplotlib.rcParams['savefig.dpi'] = 144
```


```python
from static_grader import grader
```


```python
import expectexception
```

# Object-oriented exercises
## Introduction

The objective of these exercises is to develop your familiarity with Python's `class` syntax and object-oriented programming. By deepening our understanding of Python objects, we will be better prepared to work with complex data structures and machine learning models. We will develop a `Point` class capable of handling some simple linear algebra operations in 2D.

## Exercise 1: `point_repr`

The first step in defining most classes is to define their `__init__` and `__repr__` methods so that we can construct and represent distinct objects of that class. Our `Point` class should accept two arguments, `x` and `y`, and be represented by a string `'Point(x, y)'` with appropriate values for `x` and `y`.

When you've written a `Point` class capable of this, execute the cell with `grader.score` for this question (do not edit that cell; you only need to modify the `Point` class).


```python
class Point(object):

    def __init__(self, x, y):
        self.x = x
        self.y = y
            
    def __repr__(self):
        return "Point({}, {})".format(self.x, self.y)
```


```python
grader.score.vc__point_repr(lambda points: [str(Point(*point)) for point in points])
```

    ==================
    Your score:  1.0
    ==================


## Exercise 2: add_subtract

The most basic vector operations we want our `Point` object to handle are addition and subtraction. For two points $(x_1, y_1) + (x_2, y_2) = (x_1 + x_2, y_1 + y_2)$ and similarly for subtraction. Implement a method within `Point` that allows two `Point` objects to be added together using the `+` operator, and likewise for subtraction. Once this is done, execute the `grader.score` cell for this question (do not edit that cell; you only need to modify the `Point` class.)

(Remember that `__add__` and `__sub__` methods will allow us to use the `+` and `-` operators.)


```python
class Point(object):

    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __repr__(self):
        return 'Point({}, {})'.format(self.x, self.y)

    def __add__(self, p2):
        if isinstance(p2, Point):
            return Point(self.x + p2.x, self.y+p2.y)
        else:
            raise TypeError('Expected a Point type but got {}'.format(type(p2)))
    def __sub__(self, p2):
        if isinstance(p1, Point):
            return Point(self.x - p2.x, self.y - p2.y)

        else:
            raise TypeError('Expected a Point type but got {}'.format(type(p2)))

```


```python
p1 = Point(3, 5)
p2 = Point(5, 6)
print(type(p1))
print(type(p2))
p3 = (p2 + p1)
print(type(p2), 'point2')
print(type(p3), 'point3')

p4 = (p2 - p1)
print(type(p4), 'point4')
```

    <class '__main__.Point'>
    <class '__main__.Point'>
    <class '__main__.Point'> point2
    <class '__main__.Point'> point3
    <class '__main__.Point'> point4



```python
# %%expect_exception TypeError

# p3 = (Point(5,6) - 1)
# print(p3)
```


```python
from functools import reduce
def add_sub_results(points):
    points = [Point(*point) for point in points]
    return [str(reduce(lambda x, y: x + y, points)), 
            str(reduce(lambda x, y: x - y, points))]

grader.score.vc__add_subtract(add_sub_results)
```

    ==================
    Your score:  1.0
    ==================


## Exercise 3: multiplication

Within linear algebra there's many different kinds of multiplication: scalar multiplication, inner product, cross product, and matrix product. We're going to implement scalar multiplication and the inner product.

We can define scalar multiplication given a point $P$ and a scalar $a$ as 
$$aP=a(x,y)=(ax,ay)$$

and we can define the inner product for points $P,Q$ as
$$P\cdot Q=(x_1,y_1)\cdot (x_2, y_2) = x_1x_2 + y_1y_2$$

To test that you've implemented this correctly, compute $2(x, y) \cdot (x, y)$ for a `Point` object. Once this is done, execute the `grader.score` cell for this question (do not edit that cell; you only need to modify the `Point` class.)

(Remember that `__mul__` method will allow us to use the `*` operator. Also don't forget that the ordering of operands matters when implementing these operators.)


```python
class Point(object):

    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __repr__(self):
        return 'Point({}, {})'.format(self.x, self.y)
    
    def __mul__(self, multipler):
        if isinstance(multipler, int):
            return Point(self.x*multipler, self.y*multipler)
        elif isinstance(multipler, Point):
            return (self.x * multipler.x) + (self.y*multipler.y)
        else:
            raise TypeError('Expect value of type Int or Poin, got type %' % type(multipler))
```


```python
p1 = Point(943, 177)
print(p1 * 2)
print(p1 * p1)
```

    Point(1886, 354)
    920578



```python
def mult_result(points):
    points = [Point(*point) for point in points]
    return [point*point*2 for point in points]

grader.score.vc__multiplication(mult_result)
```

    ==================
    Your score:  1.0
    ==================


## Exercise 4: Distance

Another quantity we might want to compute is the distance between two points.  This is generally given for points $P_1=(x_1,y_1)$ and $P_2=(x_2,y_2)$ as 
$$D = |P_2 - P_1| = \sqrt{(x_1-x_2)^2 + (y_1-y_2)^2}.$$

Implement a method called `distance` which finds the distance from a point to another point. 

Once this is done, execute the `grader.score` cell for this question (do not edit that cell; you only need to modify the `Point` class.)

### Hint
* *You can use the `sqrt` function from the math package*.


```python
from math import sqrt

class Point(object):

    def __init__(self, x, y):
        self.x = x
        self.y = y

    def distance(self, other):
        if isinstance(self, Point) and isinstance(other, Point):
            xdiff = (self.x - other.x)**2
            ydiff = (self.y - other.y)**2
            return sqrt(xdiff + ydiff)
        raise TypeError('Argument 1 and argument 2 must be instance of Point')
    
    def __repr__(self):
        return 'Point({},{})'.format(self.x, self.y)
```


```python
def dist_result(points):
    points = [Point(*point) for point in points]
    return [points[0].distance(point) for point in points]

grader.score.vc__distance(dist_result)
```

    ==================
    Your score:  1.0
    ==================


## Exercise 5: Algorithm

Now we will use these points to solve a real world problem!  We can use our Point objects to represent measurements of two different quantities (e.g. a company's stock price and volume).  One thing we might want to do with a data set is to separate the points into groups of similar points.  Here we will implement an iterative algorithm to do this which will be a specific case of the very general $k$-means clustering algorithm.  The algorithm will require us to keep track of two clusters, each of which have a list of points and a center (which is another point, not necessarily one of the points we are clustering).  After making an initial guess at the center of the two clusters, $C_1$ and $C_2$, the steps proceed as follows

1. Assign each point to $C_1$ or $C_2$ based on whether the point is closer to the center of $C_1$ or $C_2$.
2. Recalculate the center of $C_1$ and $C_2$ based on the contained points. 

See [reference](https://en.wikipedia.org/wiki/K-means_clustering#Standard_algorithm) for more information.

This algorithm will terminate in general when the assignments no longer change.  For this question, we would like you to initialize one cluster at `(1, 0)` and the other at `(-1, 0)`.  

The returned values should be the two centers of the clusters ordered by greatest `x` value.  Please return these as a list of numeric tuples $[(x_1, y_1), (x_2, y_2)]$

In order to accomplish this we will create a class called cluster which has two methods besides `__init__` which you will need to write.  The first method `update` will update the center of the Cluster given the points contained in the attribute `points`.  Remember, you after updating the center of the cluster, you will want to reassign the points and thus remove previous assignments.  The other method `add_point` will add a point to the `points` attribute.

Once this is done, execute the `grader.score` cell for this question (do not edit that cell; you only need to modify the `Cluster` class and `compute_result` function.)

Test dat points 
[[-107, 630], [-790, -305], [-564, -387], [-181, -68], [330, -474], [-295, -803], [407, -920], [-640, 20], [943, 177], [428, -391], [-62, -335], [964, -98], [-306, -540], [-103, -979], [393, 208], [-94, -689], [497, -273], [201, 903], [965, 416], [-204, -928], [-809, -521], [116, -442], [56, 292], [-1, -604], [-241, 54], [-473, -996], [-61, -70], [-496, -354], [443, 539], [-786, 905], [620, 581], [-547, 588], [320, 102], [643, 964], [-696, 219], [-449, -941], [685, 640], [-763, -178], [120, 638], [-419, -894], [826, -216], [-583, -731], [-909, 170], [848, -749], [156, 946], [595, -172], [436, 93], [561, 48], [535, -868], [-507, 424]] Points


```python
class Cluster(object):
    def __init__(self, x, y):
        self.center = Point(x, y)
        self.points = []
    
    def update(self):
        points_size = len(self.points)
        sum_X = 0
        sum_Y = 0
        for i in range(points_size):
            sum_X = sum_X + self.points[i].x
            sum_Y = sum_Y + self.points[i].y
        x_mean = sum_X/points_size
        y_mean = sum_Y/points_size
        self.center.x = x_mean
        self.center.y = y_mean

    def add_point(self, point):
        self.points.append(point)
```


```python
def compute_result(points):
    points = [Point(*point) for point in points]
    a = Cluster(1,0)
    b = Cluster(-1,0)
    count = 0
    a_old = []
    b_old = []
    for _ in range(10000): # max iterations
        for point in points:
            if point.distance(a.center) < point.distance(b.center):
                # add the right point
                a.add_point(point)
            else:
                # add the right point
                b.add_point(point)
        if a_old == a.points:
            print('Time to break')
            break
        a_old = a.points
        b_old = b.points
        a.update()
        b.update()
        a.points = []
        b.points = []
    return [(a.center.x, a.center.y),(b.center.x, b.center.y)]

```


```python
grader.score.vc__k_means(compute_result)
```

    Time to break
    ==================
    Your score:  1.0
    ==================


*Copyright &copy; 2017 The Data Incubator.  All rights reserved.*
