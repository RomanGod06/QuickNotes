# Computer Graphics: Circle Drawing Algorithms Cheat Sheet

## 1. Core Architecture: 8-Way Symmetry

Both algorithms only calculate Octant 2 (top-right slice, from 0° to 45°) and reflect the coordinates to the other 7 octants.

* **Driving Axis:** X-axis (always increments).
* **Loop Condition:** `while (x <= y)` (Stops exactly at the 45° intersection).

### Shared Helper Function:
```c
// Helper function used by BOTH algorithms
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

## 2. Bresenham's Circle Drawing Algorithm

**Logic:** Evaluates the absolute distance from the true circle boundary to the candidate pixels.

**Mathematical State Machine:**
* **Initial Parameter:** $p_0 = 3 - 2r$
* **If $p < 0$ (Pick East Pixel):** * $y$ stays the same.
    * $p = p + 4x + 6$
* **If $p \ge 0$ (Pick South-East Pixel):**
    * $y = y - 1$
    * $p = p + 4(x - y) + 10$

### Full C Implementation:
```c
void drawBresenhamCircle(int xc, int yc, int r) {
    int x = 0;
    int y = r;
    int p = 3 - 2 * r; 

    while (x <= y) {
        draw8WaySymmetry(xc, yc, x, y);
        
        if (p < 0) {
            p = p + 4 * x + 6;
        } else {
            p = p + 4 * (x - y) + 10;
            y--; 
        }
        x++; 
    }
}
```

---

## 3. Midpoint Circle Drawing Algorithm

**Logic:** Plugs the exact geometric midpoint between the candidate pixels into the implicit circle equation $F(x,y) = x^2 + y^2 - r^2$.

**Mathematical State Machine:**
* **Initial Parameter:** $p_0 = 1 - r$
*(Note: In the code loop below, $x$ is incremented before $p$ is updated, so the formulas use $x_{new}$ and $y_{new}$)*
* **If $p < 0$ (Midpoint is inside, pick East Pixel):** * $y$ stays the same.
    * $p = p + 2x_{new} + 1$
* **If $p \ge 0$ (Midpoint is outside, pick South-East Pixel):**
    * $y_{new} = y_{old} - 1$
    * $p = p + 2x_{new} + 1 - 2y_{new}$

### Full C Implementation:
```c
void drawMidpointCircle(int xc, int yc, int r) {
    int x = 0;
    int y = r;
    int p = 1 - r; 

    while (x <= y) {
        draw8WaySymmetry(xc, yc, x, y);
        
        x++; // x_new is calculated here
        
        if (p < 0) {
            p = p + 2 * x + 1;
        } else {
            y--; // y_new is calculated here
            p = p + 2 * x + 1 - 2 * y; 
        }
    }
}
```
If the examiner asks you for the difference between the two:

* **Similarity:** Both avoid floating-point math entirely, and both use `while (x <= y)` to exploit 8-way symmetry.
* **Difference:** Bresenham's derives its decision parameter by minimizing the absolute error distance to the candidate pixels. Midpoint derives its parameter by passing the coordinate `(x + 1, y - 0.5)` directly into the algebraic circle equation.
