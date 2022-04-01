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
Let us use variables `Q`, `R`, and `S` as labels for the axes:

- Axis Q - Straight up
- Axis R - To top right
- Axis S - To top left

![Q, R, and S Axes](https://cdn.eyzi.dev/hex-study/axes.png)

Since the axis perpendicular to the <strong>ORIGIN LINE</strong> is <strong>AXIS Q</strong>, we'll give it a bit more significance over the other two axes.

## Direction

Where <strong>Q AXIS</strong> faces is its positive direction.
Meanwhile it is the negative direction for the other two axes.

Now that labels are covered, we will rotate it 90 degrees clockwise so that the <strong>ORIGIN LINE</strong> is vertical and the <strong>Q AXIS</strong> is horizontal.

![Directions](https://cdn.eyzi.dev/hex-study/directions.png)

Let us take a hex grid and express each hexagon in an axis as its coordinates in `(Q, R, S)` format where the hexagon at `(0, 0, 0)` is the <strong>ORIGIN</strong>.

![Labelled Axes](https://cdn.eyzi.dev/hex-study/coordinates.png)

## Intuitiveness

As we go further right, the <strong>Q AXIS</strong> gets bigger, which is the first value in our coordinates.
Since this is a straight axis (i.e., completely horizontal or vertical), this would be the axis we use first when traversing the grid.

As we go further up towards the positive side of the <strong>ORIGIN LINE</strong>, the <strong>R AXIS</strong> gets bigger, which is the second value in our coordinates.
Together with the <strong>Q AXIS</strong>, this offers a familiar spatial configuration as in square grids.

There's nothing intuitive about the <strong>S AXIS</strong> getting smaller as we move right or up.
Still, there's a good reason for this decision.

To explain that, let's assume that we take equal step, say `2`, on every axis. That is, 2 positive steps on the <strong>Q AXIS</strong>, 2 positive steps on the <strong>R AXIS</strong>, and 2 positive steps on the <strong>S AXIS</strong>. Where does that lead us?

With the current configuration, those steps will lead us back to where we started.

![Circular](https://cdn.eyzi.dev/hex-study/circular.png)

Which makes for a nice function:
```
xQ + xR + xS = 0
```

Obviously, flipping directions would just mean changing some signs in that function for other calculations to still fit.
However, I believe this particular equation adds to its intuitiveness.

## Traversal

Let's suppose we want to go to a hexagon that is not in an axis. For instance, the top-left of `(-2, 0, 0)` from the <strong>ORIGIN</strong>:

![Traversal 01](https://cdn.eyzi.dev/hex-study/traversal-01.png)

How do we get there?
Well, let's think of it as if we are taking a step on an axis at a time.
In this case, we need to take two negative steps on the <strong>Q AXIS</strong> and a postive step on the <strong>R AXIS</strong>.

Visually, we can represent it as
```
(0, 0, 0) - Q - Q + R
```
Which in numerical form would be
```
(0, 0, 0) + (-1, 0, 0) + (-1, 0, 0) + (0, 1, 0)
```
Or simply
```
(0, 0, 0) + (-2, 1, 0)
```
Which, when we add the value on each axis, we get
```
(-2, 1, 0)
```
And that's the coordinates of that hexagon <strong>ORIGIN</strong>.

![Traversal 02](https://cdn.eyzi.dev/hex-study/traversal-02.png)

Notice how the axis steps can be in any order.
`-Q-Q+R`, or `-Q+R-Q`, or `+R-Q-Q` all lead to the same point and are expressed in the same numerical form.
For our purposes, any groups of steps that can be expressed in the same numerical form is considered a single <strong>PATH</strong>.

## Normalized Coordinates

However, unlike in square grids, there is more than one <strong>PATH</strong> to get to the same spot.
For instance, we can get to the same point `(-2, 1, 0)` from <strong>ORIGIN</strong> by taking 2 positive steps on the <strong>R AXIS</strong>, a negative step on the <strong>Q AXIS</strong>, and a positive step on the <strong>S AXIS</strong>.

![Traversal 03](https://cdn.eyzi.dev/hex-study/traversal-03.png)

Does that make `(-1, 2, 1)` the coordinates of this hexagon?
It can have <i>two (or more)</i> coordinates?!

Yes, it can.
Those coordinates are valid for that hexagon.
Though, it does make it confusing not having unique coordinates for a specific cell.

We can solve this by normalizing the coordinates.
That is, formulating an equation such that it takes any valid coordinates of a particular cell and always gives back the same unique value which would be considered the <strong>NORMALIZED</strong> coordinates of that cell.

We can say that the <strong>NORMALIZED</strong> coordinates of a cell is the <strong>PATH</strong> with the shortest steps from the <strong>ORIGIN</strong>.

In the given case, the coordinates `(0, -2, 1)` has three steps while `(-1, 2, 1)` has four. There is no <strong>PATH</strong> shorter than `(0, -2, 1)`.
Therefore, the <strong>NORMALIZED</strong> coordinates of this cell is `(0, -2, 1)`.

Wait!
Does that mean we'll have to <i>count</i> the steps for each coordinate <i>and</i> find a shorter path <i>AND</i> confirm the shortest path?
Isn't that an NP problem?!

Fortunately, there is a crazy easy way to do it.
But first, let's fill the rest of the grid.

![Normalization 01](https://cdn.eyzi.dev/hex-study/normalization-01.png)

And talk about some <i>absolute</i> rules in this system.

## Rules

### <strong>Flat Axis</strong>

There MUST <i>always</i> be at least one axis that is zero. The axis that does not contribute to the <strong>NORMALIZED</strong> coordinates of a hexagon is what we'll call a <strong>FLAT AXIS</strong>.

Only the <strong>ORIGIN</strong> has three <strong>FLAT AXES</strong>.
Along an axis, the <strong>NORMALIZED</strong> coordinates MUST have two <strong>FLAT AXES</strong>.
Otherwise, there MUST be one <strong>FLAT AXIS</strong>.

When a given coordinates does not have a <strong>FLAT AXIS</strong>, then it is not <strong>NORMALIZED</strong>.

### <strong>Axis Coordinates</strong>

Coordinates with at least two <strong>FLAT AXES</strong> is already considered <strong>NORMALIZED</strong>. This includes the <strong>ORIGIN</strong>.

### <strong>Inverse Pair</strong>

>Note: This rule only applies to coordinates with a single <strong>FLAT AXIS</strong> (i.e., non-axis coordinates)

When we format a given coordinates such that:
- a zero is represented as `0`
- a negative value is represented as `-1`
- and a positive value is represented as `+1`

(or what we'll refer to as the given coordinates' <strong>SIGN FORMAT</strong>)

Then the sum of the values of a coordinates' <strong>SIGN FORMAT</strong> MUST be zero.

As an example, let's suppose a coordinates `(3, 0, 1)`.

This will have a <strong>SIGN FORMAT</strong> of `(1, 0, 1)`.

When we add the values
```
1 + 0 + 1
```
We get
```
2
```
Which is not zero. Therefore, the coordinates `(3, 0, 1)` is not <strong>NORMALIZED</strong>.

## Normalization Function

So how <i>do</i> we normalize a given coordinates?

Just subtract each value by the median value. Easy, right?

Let's try it with our example above `(3, 0, 1)`.

As mentioned, we need to subtract each value by the median value. In this case, the median value is `1`:
```
= ((3 - 1), (0 - 1), (1 - 1))
= (2, -1, 0)
```

So the normalized coordinates of `(3, 0, 1)` is `(2, -1, 0)`.

That is the shortest path to that hexagon relative to the <strong>ORIGIN</strong>.
That's this one!

![Normalization 02](https://cdn.eyzi.dev/hex-study/normalization-02.png)


## Patterns

### <strong>Magnitude</strong>

If we draw a ring from the <strong>ORIGIN</strong> such that:
- the zeroth ring consists only of the <strong>ORIGIN</strong>;
- the bigger ring contains all hexagons that touch the smaller ring;
- and a hexagon is part of only one ring

Then the magnitude of a hexagon is the index of the ring that it belongs to.

![Magnitude](https://cdn.eyzi.dev/hex-study/magnitude.png)

The magnitude of a hexagon MUST be the sum of the absolute values of the coefficients of its normalized coordinates

### <strong>Diagonals</strong>

In case you didn't know, hex grids do have diagonals. Just like in a square grid, you reach a diagonal by taking the same amount of steps on two axes.

The only difference is that, bound by the <strong>Axis Pair</strong> rule, diagonal hexagons are expressed in any of the following:
- xR-xQ
- xR-xS
- xQ-xS

![Diagonals](https://cdn.eyzi.dev/hex-study/diagonals.png)

### <strong>Relative</strong>

Traversal steps may be applied to any cell and the coordinates will still be valid.

Let's revisit our earlier hexagon at coordinates `(-2, 1, 0)`.
To get here, we took two negative steps on the <strong>Q AXIS</strong> and a positive step on the <strong>R AXIS</strong> from the <strong>ORIGIN</strong>.

```
(0, 0, 0) + (-2, 1, 0) = (-2, 1, 0)
```

What if we apply the same steps from `(3, -2, 0)`?

```
(3, -2, 0) + (-2, 1, 0) = (1, -1, 0)
```

![Relative 01](https://cdn.eyzi.dev/hex-study/relative-01.png)

Looks like it still works. How about the coordinates `(3, 0, -1)`?

```
(3, 0, -1) + (-2, 1, 0) = (1, 1, -1)
```

The coordinates `(1, 1, -1)` is valid, but it will be normalized to `(0, 0, -2)`.

![Relative 02](https://cdn.eyzi.dev/hex-study/relative-02.png)

Similarly, if we want to get the <strong>PATH</strong> from one coordinate to another, we simply get the difference of the destination coordinates and the source coordinates.

If we want to go from `(-2, 0, 0)` to `(0, 0, -2)`
```
= (0, 0, -2) - (-2, 0, 0)
= ((0 - -2), (0 - 0), (-2 - 0))
= (2, 0, -2)
```

The shortest path to get from `(-2, 0, 0)` to `(0, 0, -2)` is `(2, 0, -2)`.

![Relative 03](https://cdn.eyzi.dev/hex-study/relative-03.png)

### <strong>Transposition</strong>

While the alternating positive and negative axis is confusing, once you understand the basic concept, you should be able to apply any transposition to it as you please.

In this example, we rotated the grid so the <strong>ORIGIN</strong> is on the bottom left with the positive <strong>Q AXIS</strong> going upwards.

![Transposition](https://cdn.eyzi.dev/hex-study/transposition.png)

## Conclusion

This doesn't feel complete yet, but I believe it's a good baseline for a system.
