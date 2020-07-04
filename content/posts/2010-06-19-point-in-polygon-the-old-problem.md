---
title: "Point in Polygon: The Old Problem"
date: 2010-06-19
thumb: "/images/2010/06/pip-thumbnail.png"
hero: "/images/2010/06/pip-hero.jpg"
summary: "Solving a drawing problem without a GPU."
tags: [programming]
---
Calculating whether a point on a 2D plane is inside a polygon on the same plane is a computer science problem from the beginnings of time!

If you are building a desktop based program, it can be simple - set up a drawing canvas off screen and colour it white, draw the polygon and colour it black, then get the colour of the point - black for inside, white for outside.

{{< figure src="/images/2010/06/pip-problem.png" title="The Problem" class="float-left mr-6" >}}

But what about those of us without the luxury of a GPU to work it out for us?

Let's define our problem, a complex polygon, with three points - which are inside it.

For now, we will deal with the case of solid polygons.  Later we'll discuss polygons with holes, and look at methods of dealing with them.

We also want to do this as computationally simply as possible - we don't want to be waiting for answers, even for extremely complex polygons.

A solution to this problem involves drawing a horizontal line from outside the polygon to the point.

{{< figure src="/images/2010/06/pip-solution.png" title="The solution, part 1" class="float-right mr-6" >}}

We count the number of times the line intersects the polygon - if it is odd the point is inside the polygon, even and it is outside.

This will give us the right answer every time, but is not as simple as it could be.  Luckily, we can do a pre-test to exclude some points which is simple - is the point inside the bounding box?

Finding the bounding box is relatively simple, and checking whether the point is inside the bounding box is also very simple.

If our point is inside our bounding box, we continue and work out the line intersections, but if it is outside it definitely can't be in polygon!

{{< figure src="/images/2010/06/pip-intersection.png" title="Line intersection" class="float-right mr-6" >}}

So how do we work out the intersections? As we know the corners of the polygon (an array of points defines the polygon) we can iterate clockwise over the points and look at each side - do the line segments intersect?

If these line segments were in fact lines, we could simply work out the equations and solve them simultaneously.  Actually, it turns out we can do this anyway, with certain additional conditions.

Let's work out the equations first, and we could look at the conditions later.  The corner we are looking at is *i*, the corner before is *j* and the point is *p*.  So we define the co-ordinates as:

{{< figure src="/images/2010/06/pip-coords.png" alt="Coordinates" class="text-center mb-4" >}}

The horizontal line is easy: ![Y Line](/images/2010/06/pip-line-y.png)

For the polygon segment, we can use the general equation of a line, and substitute our variables:

{{< figure src="/images/2010/06/pip-general-line.png" alt="Line equation" class="text-center mb-4" >}}

We know the *y* co-ordinate where our lines intersect, so we just need to work out the *x* co-ordinate.  Let's solve the equations simultaneously and re-arrange to find *x*:

{{< figure src="/images/2010/06/pip-solve.png" alt="Simultaneous equations" class="text-center mb-4" >}}
We now know the co-ordinates of where the two lines intersect, but we really want to know *if* the line segments intersect.

{{< figure src="/images/2010/06/pip-out-of-bounds.png" alt="Out of bounds" class="text-center mb-4" >}}

We do this by limiting the co-ordinates on the end points of the line.  The *y* co-ordinate of the point must lie between the y co-ordinates of the polygon side - this test is the easiest, so we do this first.  As you can see from the diagram, if the *y* co-ordinate of the intersect is outside these boundaries, it is not on the polygon side.

Finally, the most complex calculation - we use the equation we worked out earlier and solve it for *x*.  If this is above 0 and below the *x* co-ordinate of the point, it lies on the horizontal line segment.

If our point has passed all these tests, it intersects with this side of the polygon.  After we have iterated over each side, we add up all the intersects and see if it is odd or even - we have our answer!

So what about the code?  Let's look at an implementation in JavaScript.  Assuming that we have an array of corners for our polygon:

```javascript {linenos=table}
var aPolygon = new Array();
aPolygon[0] = new Object;
aPolygon[0].iX = 73;
aPolygon[0].iY = 81;
aPolygon[1] = new Object;
aPolygon[1].iX = 111;
aPolygon[1].iY = 48;    
```

And so on...

We can define a general function that takes a point and the array that defines the polygon:

```javascript {linenos=table}
function inPolygon(oPoint, aPolygon) {
    // bounding box first
    var iXMin = aPolygon[0].iX;
    var iXMax = aPolygon[0].iX;
    var iYMin = aPolygon[0].iY;
    var iYMax = aPolygon[0].iY;
    for (var i = 1; i < aPolygon.length; i++) {
        if (aPolygon[i].iX > iXMax) {
            iXMax = aPolygon[i].iX;
        }
        else if (aPolygon[i].iX < iXMin) {
            iXMin = aPolygon[i].iX
        }

        if (aPolygon[i].iY > iYMax) {
            iYMax = aPolygon[i].iY;
        }
        else if (aPolygon[i].iY < iYMin) {
            iYMin = aPolygon[i].iY;
        }
    }
    if ((oPoint.iX > iXMin & oPoint.iX < iXMax) &
        (oPoint.iY > iYMin & oPoint.iY < iYMax)) {
        /* line segments
           take each side of the polygon and see if line
           from 0,pY to pX,pY goes through it
           to simplify, we can work out if pY is within the
           lines vertical bounds, then
           see if the point on the line at pY is less than pX
        */

        // loop through the polygon, looking at this point, and the one before it
        j = aPolygon.length - 1;
        var iIntersects = 0;
        for (var i = 0; i < aPolygon.length; i++) {
            if ((oPoint.iY > aPolygon[i].iY & oPoint.iY <= aPolygon[j].iY) ||
                (oPoint.iY <= aPolygon[i].iY & oPoint.iY > aPolygon[j].iY)) {
                if ((((oPoint.iY - aPolygon[i].iY) * 
                    ((aPolygon[j].iX - aPolygon[i].iX) /
                    (aPolygon[j].iY - aPolygon[i].iY))) + aPolygon[i].iX) <
                    oPoint.iX) {
                    iIntersects++;
                }
            }
            j = i;
        }
        if (iIntersects &amp; 1 == 1) {
            return true;
        }
        else {
            return false;
        }
    }
    else {
        return false;
    }
}
```

Feel free to use this function is any of your projects.

By the way, the equations above were created using the excellent [LaTeX equation editor](http://www.codecogs.com/latex/eqneditor.php). My thanks go out to Will Bateman and Steve Mayer!