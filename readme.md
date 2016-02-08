# Overview

This project is an interactive simulation of the [art gallery problem](https://en.wikipedia.org/wiki/Art_gallery_problem), which asks how many guards it takes so that the entire polygon is visible. Although an upper bound of floor(n/3) is known, determining the optimal guard placement for an arbitrary polygon is NP-hard. This simulation allows you to create an arbitrary polygon and investigate guard visibility.

You can try it out [here](https://rawgit.com/hexahedria/math_198o_gallery/master/index.html).

# Algorithms

## Determining Insideness

To check if a guard is actually inside the polygon (which we check to make sure our visibility algorithm works), we cast a single ray downward and count how many times it intersects an edge. If this number is odd, the guard must be inside the polygon. If the guard is directly above a vertex, we nudge the guard slightly to avoid errors due to floating-point precision.

## Calculating Visibility

To compute the region visible to a specific guard, we need to determine what obstructs the guard's view. Since vision follows straight lines, and the polygon edges are straight, we know that the only place an obstacle would appear is at one of the polygon vertices. Thus, we follow the following algorithm:

1. Sort the vertices by angle around the guard (O(n log n)).
2. Iterate through the vertices in increasing angle. For each vertex, (overall O(n^2))
    + Cast a ray from the guard to that vertex and record the closest  intersection (by checking the ray against every edge). (O(n))
    + Determine how far the guard can see on either side of the vertex (O(1))
        - If the ray hits an edge before hitting the vertex, add the edge intersection point to the visibility polygon
        - If the ray hits the vertex first, but the vertex's adjacent edges lie on one side of the ray only, add two points to the polygon: the edge intersection as well as the vertex, on the appropriate side.
        - If the ray hits the vertex first, and the vertex's adjacent edges block both sides, add the vertex only to the polygon

Thus this algorithm is O(n^2)

## Triangulation and 3 Coloring

Since we know every simple polygon can be triangulated, and every triangulation must have at least 2 ears, we can simply iterate over all candidate ears, and if we find an ear, add it to our triangulation and repeat until we reduce the polygon to a triangle. This has worst-case performance of O(n^3), but is still very fast, since there are often many ears to the polygon and we remove a point at each iteration.

As we remove ears, we maintain a stack of the ears we find. Once we have just a triangle remaining, we can color it arbitrarily, then walk back up the stack and color each ear as we go. This results in a 3-coloring of our vertices.

Summary of the algorithm:

1. Iterate over all vertices. For each vertex, check if it is an ear tip:
    - Loop through every other vertex polygon to see if it is in the ear (O(n))
    - Ensure the ear is pointed in the right direction (O(1))
   When we find an ear, add it to our stack. (O(n^2))
2. Repeat 1 until we are left with a single triangle (O(n^3))
3. Assign colors to remaining triangle (O(1))
4. Pop off each ear, coloring it as we go (O(n))

## Placing Guards

Once we have a 3-coloring, we can loop through each color to choose the color that is used the least. Then we can place a guard at each of the vertices with that color. One caveat to this is that guards need to be strictly inside the polygon, so we bump them inward (away from the vertex, into the polygon) by 1 pixel.
