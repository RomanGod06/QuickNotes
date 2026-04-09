# Circle Rasterization Algorithms: Study Notes

## 1. The Core Architecture: 8-Way Symmetry

Both Bresenham's and the Midpoint algorithms rely on the geometric properties of a circle to save ALU clock cycles.

* **The Concept:** We only calculate the rasterization path for a single 45° slice (Octant 2, from 90° to 45°). For every `(x, y)` coordinate generated, we instantly mirror it to the other 7 octants.
* **The Driving Axis:** The X-axis is the driving axis because, in the top-right slice, the curve is relatively flat (`|Δx| > |Δy|`). Therefore, X increments unconditionally every loop.
* **The Loop Condition:** The boundary of this 45° slice is exactly where the X and Y coordinates cross. Therefore, our zero-math loop condition is always: `while (x <= y)`.

### Shared Helper Function (C Code):
```c
#include <graphics.h>

// Placed above your algorithm functions to handle the rendering
void draw8WaySymmetry(int xc, int yc, int x, int y) {
    putpixel(xc + x, yc + y, WHITE); // Octant 1
    putpixel(xc - x, yc + y, WHITE); // Octant 4
    putpixel(xc + x, yc - y, WHITE); // Octant 8
    putpixel(xc - x, yc - y, WHITE); // Octant 5
    putpixel(xc + y, yc + x, WHITE); // Octant 2
    putpixel(xc - y, yc + x, WHITE); // Octant 3
    putpixel(xc + y, yc - x, WHITE); // Octant 7
    putpixel(xc - y, yc - x, WHITE); // Octant 6
}
```

---

## 2. Bresenham’s Circle Drawing Algorithm

**Theory:** This algorithm calculates the true geometric distance from the center of the two candidate pixels to the actual theoretical circle boundary. It selects the pixel with the smaller absolute distance.

**Mathematical State Machine:**
* **Initial Decision Parameter:** `p_0 = 3 - 2r`
* **Case 1 (p < 0):** We pick the East pixel. Y stays the same.
  * **Update:** `p_{k+1} = p_k + 4x_k + 6`
* **Case 2 (p >= 0):** We pick the South-East pixel. Y drops by 1.
  * **Update:** `p_{k+1} = p_k + 4(x_k - y_k) + 10`

### Full C Implementation:
```c
void drawBresenhamCircle(int xc, int yc, int r) {
    int x = 0;
    int y = r;
    int p = 3 - 2 * r; // Initial decision parameter

    while (x <= y) {
        draw8WaySymmetry(xc, yc, x, y);
        
        if (p < 0) {
            // Pick upper pixel (Y stays same)
            p = p + 4 * x + 6;
        } else {
            // Pick lower pixel (Y decrements)
            p = p + 4 * (x - y) + 10;
            y--; 
        }
        x++; // X unconditionally steps forward
    }
}
```

---

## 3. Midpoint Circle Drawing Algorithm

**Theory:** Instead of measuring distances to the pixels, this algorithm evaluates the implicit equation of a circle `F(x, y) = x^2 + y^2 - r^2` exactly at the geometric midpoint between the East and South-East candidate pixels.

**Mathematical State Machine:**
* **Initial Decision Parameter:** We evaluate the midpoint at `(1, r - 0.5)`. Dropping the fractional remainder to keep integer arithmetic, we get: `p_0 = 1 - r`
* **Case 1 (p < 0):** Midpoint is inside the circle. The curve is closer to the East pixel. Y stays the same.
  * **Update (assuming X is already incremented):** `p_{k+1} = p_k + 2x_new + 1`
* **Case 2 (p >= 0):** Midpoint is outside the circle. The curve is closer to the South-East pixel. Y drops by 1.
  * **Update (assuming X is already incremented):** `p_{k+1} = p_k + 2x_new + 1 - 2y_new`

### Full C Implementation:
```c
void drawMidpointCircle(int xc, int yc, int r) {
    int x = 0;
    int y = r;
    int p = 1 - r; // Initial Midpoint decision parameter

    while (x <= y) {
        draw8WaySymmetry(xc, yc, x, y);
        
        x++; // X unconditionally steps forward
        
        if (p < 0) {
            // Curve is closer to top pixel
            p = p + 2 * x + 1;
        } else {
            // Curve is closer to bottom pixel
            y--;
            p = p + 2 * x + 1 - 2 * y; 
        }
    }
}
```

---

## Key Takeaways for Your Viva

If the examiner asks you for the difference between the two:

* **Similarity:** Both avoid floating-point math entirely, and both use `while (x <= y)` to exploit 8-way symmetry.
* **Difference:** Bresenham's derives its decision parameter by minimizing the absolute error distance to the candidate pixels. Midpoint derives its parameter by passing the coordinate `(x + 1, y - 0.5)` directly into the algebraic circle equation.
