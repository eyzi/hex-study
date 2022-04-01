# Alternative Hex Grid Coordinate System

![Eyzi Hex Grid](https://cdn.eyzi.dev/hex-study/capsule.png)

Hexagons are the most fascinating geometrical figure.

I've had the pleasure of dreaming about them lately.
By that, I mean I have been bashing my head with many ideas on how to implement and traverse a hexagonal grid.
Particularly, in a game.

There are a lot of available solutions a google search away.
Some of them I have taken a lot of inspirations from.

- [Catlike Coding](https://catlikecoding.com/unity/tutorials/hex-map/part-1/) explained the Offset Coordinate system quite nicely.
It roughly matches the conventional coordinate system for square grids.
What I find doesn't quite work is determining which direction a "column" goes, which can add complexity in implementation.

- [Red Blob Games](https://www.redblobgames.com/grids/hexagons/) explained the Cube (and Axial) Coordinate system nicely.
This is the most mathematically sound method, and in turn also makes it the hardest concept to completely understand.

- The closest concept to my ideal system is the one that [Hexcell Games](http://www.hexcellgames.com/blog/hexgrid-coordinates/) described.
Conceptually, it is a little similar to the [Tetrahedral Coordinates](https://math.stackexchange.com/questions/1861635/tetrahedral-coordinates-in-space-generalization-of-hexagonal-coordinates) system.
There is still something about this system that is not quite scalable or that requires complex functions.
For the most part, I am building on top of this concept.

My goal is to build a system that feels as intuitive as a square grid coordinate system.
Something that I can build a generic library of to make game development a little easier.

## Axes

Firstly, let us label relevant details on a hexagon. To do that, we will take a congruent, flat-top hexagon and divide it in half vertically.

The line that divides it, we'll call <strong>ORIGIN LINE</strong>.
Where going left is its positive direction.

![Origin Line](https://cdn.eyzi.dev/hex-study/origin-line.png)

Now we have three lines facing away from the origin line.
These are the main axes.
Let's label these as follows:

- Axis Q - To top left
- Axis R - Straight up
- Axis S - To top right

![Q, R, and S Axes](https://cdn.eyzi.dev/hex-study/axes.png)

## Direction

Since the <strong>R AXIS</strong> is perpendicular to the <strong>ORIGIN LINE</strong>, let us give it a bit of a significance over the other two axes.

Where <strong>R AXIS</strong> faces (upwards in the image) is its positive direction.
Meanwhile both <strong>Q AXIS</strong> and <strong>S AXIS</strong> are facing (upwards in the image) their negative directions.

Now that labels are covered, we will rotate it 90 degrees clockwise so that the <strong>ORIGIN LINE</strong> is vertical and the <strong>R AXIS</strong> is horizontal.

![Directions](https://cdn.eyzi.dev/hex-study/directions.png)

## Traversal

Let us take a hex grid and express each hexagon in an axis as its coordinates in `Q, R, S` format where the hexagon at `0, 0, 0` is the <strong>ORIGIN</strong>.

![Labelled Axes](https://cdn.eyzi.dev/hex-study/coordinates.png)

Now let's suppose we want to go to a hexagon that is not in an axis. For instance, the top-left of `0, -2, 0` from the <strong>ORIGIN</strong>:

![Traversal 01](https://cdn.eyzi.dev/hex-study/traversal-01.png)

How do we get there? Well, let's think of it as if we are taking a step on an axis at a time.
In this case, we need to take two negative steps on the <strong>R AXIS</strong> and a postive step on the <strong>S AXIS</strong>.
Another way to express it is:
```
(0, 0, 0) + (0, -1, 0) + (0, -1, 0) + (0, 0, 1)
```
Or simply:
```
(0, 0, 0) + (0, -2, 1)
```
Which, when we add the value on each axis, we get:
```
(0, -2, 1)
```
And that's the coordinates of that hexagon relative to the <strong>ORIGIN</strong>.

![Traversal 02](https://cdn.eyzi.dev/hex-study/traversal-02.png)

Notice how the axis steps can be in any order.
We can choose to do the positive step on the <strong>S AXIS</strong> first, or second, and still arrive at the same coordinate.

> Note that the coordinates are <i>always</i> relative to the <strong>ORIGIN</strong>.

## Normalization

Unlike in square grids, there are two (or more) ways to get to the same spot without negation.
For instance, we can get to the same point `(0, -2, 1)` from the origin by taking two positive steps on the <strong>S AXIS</strong>, a negative step on the <strong>R AXIS</strong>, and a positive step on the <strong>Q AXIS</strong>.

![Traversal 03](https://cdn.eyzi.dev/hex-study/traversal-03.png)

Yes!
That is a valid coordinates for that hexagon.
Though, having two (or more) ways to label a coordinate can be confusing.

To solve this, let us take the concept of coordinate normalization.
Let's say that the most <i>valid</i> coordinate is one with the least amount of steps taken.

In the given case, the coordinates `(0, -2, 1)` takes three steps while the coordinates `(1, -1, 2)` takes four steps, and there is no shorter path to it. Thus, that hexagon will have the coordinates `(0, -2, 1)`.

Still, being able to express the same destination by steps is useful in game development.

Wait.
Does that mean we'll have to <i>count</i> the steps for each coordinate <i>and</i> find a shorter path <i>AND</i> confirm the shortest path?
Isn't that an NP problem?!

Fortunately, there is a crazy easy way to do so.
But first, let's fill the rest of the grid.

![Normalization 01](https://cdn.eyzi.dev/hex-study/normalization-01.png)

Ah, yes.
Emerging patterns.
Don't you just love it?

Let's talk about some of the <i>absolute</i> rules of this coordinate system.

### 1) <strong>Flat Axis</strong>

There MUST <i>always</i> be at least one axis that is zero. The axis that does not contribute to the coordinates of a hexagon is what I call a <strong>FLAT AXIS</strong>.

Only the <strong>ORIGIN</strong> has three <strong>FLAT AXES</strong>.
Along an axis, the coordinates MUST have two <strong>FLAT AXES</strong>.
Otherwise, there MUST be one <strong>FLAT AXIS</strong>.

### 2) <strong>Axis Pair</strong>

If a coordinate has a single <strong>FLAT AXIS</strong>, the signs of the other axes MUST be opposite.

By definition, hexagons along an axis, including the <strong>ORIGIN</strong> are exceptions to this rule.

With these rules, let's discuss how we determine whether the given coordinates is normalized.

<strong>Step 1</strong>: If the coordinates have more than one <strong>FLAT AXIS</strong>, then it is normalized.

<strong>Step 2</strong>: Get the signs of its coefficients.

<strong>Step 3</strong>: Add the signs of its coefficients.

<strong>Step 4</strong>: If the sum of Step 3 is zero, then it is normalized.

<strong>Step 5</strong>: Otherwise, it is not normalized.

Steps 2 and 3 may be confusing so let's go through an example.

For this, let's suppose a function that takes an integer and returns its sign:
```
function getIntegerSign (int n) {
  return (n > 0) - (n < 0)
}
```
If the passed argument is positive, this will return +1.
If it's negative, this will return -1.
If it's zero, this will return zero.

Let's use the coordinates `(2, -2, 1)`

Step 1: Does it have more than one <strong>FLAT AXIS</strong>? No.

Step 2: Let's get the signs of its coefficients using the `getIntegerSign` function above.
The result will be `(1, -1, 1)`

Step 3: Add the signs of its coefficients.
```
1 + (-1) + 1 = 1
```

Step 4: Is the sum zero? No.

Step 5: Then it is not normalized.

So how do we normalize it? Here are the steps.

<strong>Step 1</strong>: Get the median value <strong>D</strong> of all coefficients.

<strong>Step 2</strong>: Subtract every coefficient by <strong>D</strong>.

That's it! Let's try it with our example above `(2, -2, 1)`.

Step 1: The median of `2, -2, 1` is `1`.

Step 2: Subtract every coefficient by `1`.
```
((2 - 1), (-2 - 1), (1 - 1))
=(1, -3, 0)
```

So the normalized coordinates of `(2, -2, 1)` is `(1, -3, 0)`.
That is the shortes path to that hexagon relative to the <strong>ORIGIN</strong>.
That's this one!

![Normalization 02](https://cdn.eyzi.dev/hex-study/normalization-02.png)


## Patterns

### 1) <strong>Magnitude</strong>

If we draw a ring from the <strong>ORIGIN</strong> such that:
- the zeroth ring consists only of the <strong>ORIGIN</strong>;
- the bigger ring contains all hexagons that touch the smaller ring;
- and a hexagon is part of only one ring
Then the magnitude of a hexagon is the index of the ring that it belongs to.

![Magnitude](https://cdn.eyzi.dev/hex-study/magnitude.png)

The magnitude of a hexagon MUST be the sum of the absolute values of the coefficients of its normalized coordinates

### 2) <strong>Circular</strong>

When you take equal number of steps on every axis, you MUST end up back where you started. Thus, the sum of the coefficients should be zero.

```
xQ + xR + xS = 0
```
![Circular](https://cdn.eyzi.dev/hex-study/circular.png)

### 3) <strong>Diagonals</strong>

In case you didn't know, hex grids do have diagonals. Just like in a square grid, you reach a diagonal by taking the same amount of steps on two axes.

The only difference is that, bound by the <strong>Axis Pair</strong> rule, diagonal hexagons are expressed in any of the following:
- xR-xQ
- xR-xS
- xQ-xS

![Diagonals](https://cdn.eyzi.dev/hex-study/diagonals.png)



### 4) <strong>Relative</strong>

Traversal steps may be applied to any cell and the coordinates will still be valid.

Let's revisit our earlier hexagon at coordinates `(0, -2, 1)`.
To get here, we took two negative steps on the <strong>R AXIS</strong> and a positive step on the <strong>S AXIS</strong> from the <strong>ORIGIN</strong>.

```
(0, 0, 0) + (0, -2, 1) = (0, -2, 1)
```

What if we apply the same steps from `(0, 3, -2)`?

```
(0, 3, -2) + (0, -2, 1) = (0, 1, -1)
```

![Relative 01](https://cdn.eyzi.dev/hex-study/relative-01.png)

Looks like it still works. How about the coordinates `(-1, 3, 0)`?

```
(-1, 3, 0) + (0, -2, 1) = (-1, 1, 1)
```

The coordinates `(-1, 1, 1)` is valid, but it will be normalized to `(-2, 0, 0)`.

![Relative 02](https://cdn.eyzi.dev/hex-study/relative-02.png)

### 5) <strong>Transposition</strong>

While the alternating positive and negative axis is confusing, once you understand the basic concept, you should be able to apply any transposition to it as you please.

In this example, we rotate the grid so the <strong>ORIGIN</strong> is on the bottom left and the positive <strong>R AXIS</strong> going upwards.

![Transposition](https://cdn.eyzi.dev/hex-study/transposition.png)

## Conclusion

This doesn't feel complete yet, but I believe it's a good baseline for a system.
