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
*(Note: This implementation is for the 1st Octant where $0 < m < 1$)*
```c
int dx = x2 - x1;
int dy = y2 - y1;
int p = 2 * dy - dx;
int y = y1;

for (int x = x1; x <= x2; x++) {
    putpixel(x, y, WHITE); 

    if (p < 0) {
        p = p + 2 * dy;
    } else {
        y = y + 1;
        p = p + 2 * dy - 2 * dx;
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
