# 🖥️ Computer Graphics: Line Drawing Algorithms Cheat Sheet

## 1. Digital Differential Analyzer (DDA)
DDA is an incremental method of scan conversion that relies on floating-point increments. It prevents aliasing (gaps) by finding the dominant axis of movement.

### Mathematical Foundation
* **Identify Dominant Axis:** $steps = \max(|\Delta x|, |\Delta y|)$
* **Calculate Increments:** * $x_{inc} = \frac{\Delta x}{steps}$
    * $y_{inc} = \frac{\Delta y}{steps}$
* **Rounding Constraint:** Because monitors are discrete grids, continuous float coordinates must be rounded: `putpixel(round(x), round(y))`.

### Production C Code (CSPC-614P Lab 2)
```c
int dx = x2 - x1;
int dy = y2 - y1;
int steps = abs(dx) > abs(dy) ? abs(dx) : abs(dy);

float x_inc = (float)dx / steps;
float y_inc = (float)dy / steps;

float current_x = x1;
float current_y = y1;

for (int i = 0; i <= steps; i++) {
    putpixel(round(current_x), round(current_y), WHITE);
    current_x += x_inc;
    current_y += y_inc;
}
```

---

## 2. Bresenham's Line Algorithm
The industry standard for graphics hardware. It is fundamentally faster because it completely eliminates floating-point arithmetic and division, using a state machine driven by an integer decision parameter.

### Mathematical Foundation
* **Initial Decision Parameter:** $p_0 = 2\Delta y - \Delta x$
* **State Machine Logic:**
    * **Case 1 ($p < 0$):** Choose lower pixel. Y stays the same.
        * $p_{next} = p + 2\Delta y$
    * **Case 2 ($p \ge 0$):** Choose upper pixel. Y increments.
        * $p_{next} = p + 2\Delta y - 2\Delta x$

### Production C Code (CSPC-614P Lab 3)
```c
void drawBresenhamLine(int x1, int y1, int x2, int y2) {
    // 1. Force strictly positive distances
    int dx = abs(x2 - x1);
    int dy = abs(y2 - y1);
    
    // 2. Determine the step directions (+1 or -1)
    int step_x = (x1 < x2) ? 1 : -1;
    int step_y = (y1 < y2) ? 1 : -1;
    
    int x = x1;
    int y = y1;
    
    // 3. Branch based on the driving axis (The Octant Fix)
    if (dx > dy) {
        // X is the driving axis (Octants 1, 4, 5, 8)
        int p = 2 * dy - dx;
        for (int i = 0; i <= dx; i++) {
            putpixel(x, y, WHITE);
            if (p < 0) {
                p = p + 2 * dy;
            } else {
                y = y + step_y;
                p = p + 2 * dy - 2 * dx;
            }
            x = x + step_x; // X steps every iteration
        }
    } else {
        // Y is the driving axis (Octants 2, 3, 6, 7)
        int p = 2 * dx - dy;
        for (int i = 0; i <= dy; i++) {
            putpixel(x, y, WHITE);
            if (p < 0) {
                p = p + 2 * dx;
            } else {
                x = x + step_x;
                p = p + 2 * dx - 2 * dy;
            }
            y = y + step_y; // Y steps every iteration
        }
    }
}
```

---

## 3. Architectural Comparison

| Feature | DDA Algorithm | Bresenham's Algorithm |
| :--- | :--- | :--- |
| **Arithmetic Type** | Floating-point (Slow) | Integer only (Fast) |
| **Operations Used** | Division, Addition, Rounding | Addition, Subtraction, Bit-shifting |
| **Hardware Efficiency** | Low (Heavy ALU load) | High (Easily hardwired into GPUs) |
| **Accuracy** | Prone to slight floating-point drift | Mathematically exact pixel selection |

***
