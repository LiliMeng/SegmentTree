# SegmentTree

Leetcode 850: https://leetcode.com/problems/rectangle-area-ii/description/

The problem of calculating the total area covered by rectangles (where overlaps are counted only once) can be solved using a **sweep line algorithm combined with a segment tree**. The segment tree helps manage the Y-intervals dynamically as we sweep across the X-axis, and it ensures that we correctly account for overlapping regions.

### Approach:

1. **Event Generation**:
   - For each rectangle, generate two events:
     - A **start event** at `x1` when the rectangle starts (with the interval `[y1, y2]`).
     - An **end event** at `x2` when the rectangle ends (with the interval `[y1, y2]`).

2. **Sorting Events**:
   - Sort the events by their X-coordinates. If two events have the same X-coordinate, process the start events first.

3. **Segment Tree**:
   - Use a segment tree to efficiently manage and track the total covered length of Y-intervals.
   - Each node in the segment tree will store two pieces of information:
     - `count[]`: How many rectangles cover the Y-interval.
     - `total[]`: The total length of Y-intervals that are covered by at least one rectangle.

4. **Sweep Line Algorithm**:
   - As the sweep line moves across the X-axis (by processing the sorted events), use the segment tree to keep track of the total length of Y-intervals covered by rectangles at each step. This allows us to compute the area of each vertical slice between consecutive X-coordinates.

5. **Calculating Area**:
   - For each pair of consecutive X-coordinates, the area contribution is the width between the X-coordinates times the total length of Y-intervals that are currently covered by at least one rectangle.

6. **Modulo Operation**:
   - Since the total area may be large, return the result modulo \(10^9 + 7\).

### Python Code Implementation:

```python
MOD = 10**9 + 7

class SegmentTree:
    def __init__(self, ys):
        self.n = len(ys)
        self.count = [0] * (4 * self.n)  # Keeps track of how many rectangles cover each interval
        self.total = [0] * (4 * self.n)  # Total length of Y-intervals covered by at least one rectangle
        self.ys = ys  # Unique Y-coordinates (sorted)
        self.ys_index = {v: i for i, v in enumerate(ys)}  # Map Y-coordinate to its index

    def update(self, node, start, end, l, r, value):
        if l >= end or r <= start:
            return
        if l <= start and end <= r:
            self.count[node] += value
        else:
            mid = (start + end) // 2
            self.update(2 * node + 1, start, mid, l, r, value)
            self.update(2 * node + 2, mid, end, l, r, value)

        if self.count[node] > 0:
            self.total[node] = self.ys[end] - self.ys[start]
        else:
            if start < end - 1:  # Internal node with children
                self.total[node] = self.total[2 * node + 1] + self.total[2 * node + 2]
            else:
                self.total[node] = 0

    def query(self):
        return self.total[0]

def calculateArea(rectangles):
    events = []
    y_coords = set()

    # Step 1: Generate events and collect all unique y-coordinates
    for x1, y1, x2, y2 in rectangles:
        events.append((x1, y1, y2, 1))   # Start of a rectangle
        events.append((x2, y1, y2, -1))  # End of a rectangle
        y_coords.add(y1)
        y_coords.add(y2)

    # Step 2: Sort events by x-coordinate
    events.sort()
    y_coords = sorted(y_coords)

    # Step 3: Initialize the segment tree with unique y-coordinates
    seg_tree = SegmentTree(y_coords)

    prev_x = events[0][0]
    area = 0

    # Step 4: Process each event
    for x, y1, y2, typ in events:
        # Calculate the area covered since the last x-coordinate
        area += seg_tree.query() * (x - prev_x)
        area %= MOD

        # Update the segment tree with the current event
        seg_tree.update(0, 0, len(y_coords) - 1, seg_tree.ys_index[y1], seg_tree.ys_index[y2], typ)

        prev_x = x

    return area

# Example usage:
rectangles = [
    [1, 1, 3, 3],
    [2, 2, 4, 4],
    [2, 0, 3, 1]
]

print(calculateArea(rectangles))  # Output: 8
```

### Explanation of the Code:

1. **Event Generation**:
   - For each rectangle `[x1, y1, x2, y2]`, we generate two events:
     - A start event `(x1, y1, y2, 1)` when the rectangle starts.
     - An end event `(x2, y1, y2, -1)` when the rectangle ends.
   - We also collect all the unique Y-coordinates to build the segment tree.

2. **Segment Tree Initialization**:
   - We initialize a segment tree that manages the Y-axis using the unique Y-coordinates.
   - The segment tree is used to track the total covered length of Y-intervals as rectangles are added or removed during the sweep.

3. **Processing Events**:
   - The events are processed in order of their X-coordinates.
   - For each event, we first calculate the area of the vertical strip between the current and the previous X-coordinate using the total covered Y-length stored in the segment tree (`query()` function).
   - Then, we update the segment tree to reflect the addition or removal of the Y-interval `[y1, y2)` (depending on whether it's a start or end event).

4. **Updating the Segment Tree**:
   - The `update()` function is responsible for updating the `count[]` array (which tracks how many rectangles cover each Y-interval) and recalculating the `total[]` array (which tracks the total covered length of Y-intervals).
   - If `count[node] > 0`, the segment is fully covered, and the total covered length is the length of the interval. If `count[node] == 0`, the total covered length is the sum of the covered lengths of the left and right children.

5. **Returning the Result**:
   - After processing all the events, the total area is returned modulo \(10^9 + 7\).

### Example:

For the input `rectangles = [[1, 1, 3, 3], [2, 2, 4, 4], [2, 0, 3, 1]]`:

1. Events:
   - Start: (1, [1, 3]), (2, [2, 4]), (2, [0, 1])
   - End: (3, [1, 3]), (4, [2, 4]), (3, [0, 1])

2. Sorting Events:
   - The events are sorted by X-coordinate.

3. Processing:
   - At each step, the area of the vertical strip between consecutive X-coordinates is calculated based on the total covered length of Y-intervals.

The final total area covered by the rectangles is `8`.

### Time and Space Complexity:

- **Time Complexity**: 
  - Sorting the events takes \(O(n \log n)\), where \(n\) is the number of rectangles.
  - Updating and querying the segment tree takes \(O(\log m)\) for each event, where \(m\) is the number of unique Y-coordinates.
  - Overall time complexity is \(O(n \log n + k \log m)\), where \(k\) is the number of events (twice the number of rectangles).

- **Space Complexity**:
  - The space complexity is \(O(m)\) to store the segment tree arrays `count[]` and `total[]`, where \(m\) is the number of unique Y-coordinates.

### Conclusion:

This solution uses a **sweep line algorithm with a segment tree** to efficiently compute the total area covered by rectangles, ensuring that overlapping areas are counted only once. The segment tree helps manage the dynamic intervals in the Y-axis as the sweep line moves across the X-axis.


The lines:

```python
self.count = [0] * (4 * self.n)  # Keeps track of how many rectangles cover each interval
self.total = [0] * (4 * self.n)  # Total length of Y-intervals covered by at least one rectangle
```

are part of the **segment tree** initialization in your code for calculating the area covered by rectangles. Here's why these arrays are initialized in this way:

### 1. Why `4 * self.n`?

A segment tree is usually stored in an array, and for a segment tree with `n` leaf nodes (representing individual Y-intervals), we need a total of `4 * n` nodes in the array to represent the entire tree structure. This is because:

- The segment tree is a **complete binary tree**, and the number of nodes required to represent a segment tree is proportional to `4 * n` to ensure enough space for all internal nodes, especially when `n` is not a power of 2.
- **Leaf nodes** represent individual Y-intervals, and **internal nodes** represent merged intervals (ranges that cover more than one leaf).
  
For example, if there are 5 unique Y-coordinates, `n = 5`, but the full binary tree structure needs space for approximately `4 * 5 = 20` nodes.

### 2. Purpose of `self.count`

- **`self.count`** is an array where each element tracks **how many rectangles currently cover the Y-interval** represented by the corresponding node in the segment tree.
  
#### Why do we need `self.count`?

- To keep track of how many rectangles are "active" (covering the interval) at a given time during the sweep line algorithm.
- **When a rectangle starts**: We increment the `count` for the corresponding Y-interval.
- **When a rectangle ends**: We decrement the `count` for the corresponding Y-interval.
- This helps the segment tree determine whether a given Y-interval is fully or partially covered by one or more rectangles.

### 3. Purpose of `self.total`

- **`self.total`** is an array where each element stores the **total length of Y-intervals covered by at least one rectangle** at that point in the tree.

#### Why do we need `self.total`?

- `self.total[node]` represents the total length of Y-intervals covered by at least one rectangle in the interval represented by `node`.
- For each segment of the Y-axis (represented by nodes in the segment tree), `self.total[node]` stores the total covered length based on the `count[]` array.
  
#### How is `self.total` used?

- After updating the `count[]` array (i.e., when a rectangle starts or ends), the `total[]` array is recalculated to reflect how much of the Y-interval is actually covered by one or more rectangles.
- The `total[0]` (the root node) will give the total covered length of Y-intervals across the entire Y-axis at any given moment.
  
### How They Work Together:

- **`self.count[]`** is used to track the number of rectangles currently covering a Y-interval. This helps determine whether the interval is fully or partially covered.
- **`self.total[]`** stores the actual length of the Y-intervals that are covered, which is updated based on the values in `self.count[]`.

### Example:

Let's say we are working with the following Y-intervals: `[0, 1, 2, 3, 4]`. The segment tree manages intervals like `[0, 5)` (the entire Y-range), `[0, 2)`, `[2, 5)`, etc. Here's how the arrays work:

- `self.count`: This array keeps track of how many rectangles overlap with each segment of the Y-axis at any given time.
  - If `count[node] > 0`, it means that the entire segment (Y-interval) covered by this `node` is fully covered by one or more rectangles.
  - If `count[node] == 0`, it means that no rectangle covers this segment, and the total length of the Y-interval covered by rectangles is derived from the `total[]` values of its child nodes.

- `self.total`: This array stores the actual length of Y-intervals covered by rectangles, based on the values of `self.count`. 
  - For each node in the segment tree, `total[node]` is either the full length of the Y-interval (if `count[node] > 0`), or the sum of the covered lengths from its children (if `count[node] == 0`).

### Recap of Segment Tree Structure:
- The segment tree operates over the Y-axis, managing which Y-intervals are covered by rectangles as we sweep over the X-axis.
- **`self.count[]`**: Tracks how many rectangles cover each segment of the Y-interval.
- **`self.total[]`**: Stores the total length of Y-intervals covered by rectangles based on `self.count[]`.

This combination of `count[]` and `total[]` allows you to efficiently compute the total covered area in the sweep line algorithm, ensuring that overlapping regions are only counted once.


In a typical **segment tree** implementation, we often initialize the tree with an array of size `2 * n`, where `n` is the number of leaves or intervals we want to track. This size is sufficient when we are building a **full binary tree** where `n` is a power of 2, or when `n` is not too large and we can still efficiently fit the intervals using `2 * n` space.

However, when dealing with more complex use cases, such as managing dynamic intervals for the **sweep line algorithm** with a segment tree (as in the case of calculating the area covered by rectangles), we often use a tree size of **`4 * n`**. Here's why we use `4 * n` instead of `2 * n`:

### 1. **Handling Arbitrary Interval Lengths**:

In the sweep line algorithm with a segment tree, the segment tree needs to efficiently manage **dynamic ranges** (Y-intervals) for which the number of intervals is not necessarily a power of 2. 

If `n` is not a power of 2 (which is the case for most real-world problems), then a segment tree will need additional space to account for the uneven division of intervals across its nodes. For example:
- In a complete binary tree, where the number of leaves is a power of 2, `2 * n` nodes would suffice to represent the entire tree.
- However, if `n` is not a power of 2, the segment tree structure becomes "incomplete" or unbalanced in some parts, requiring extra internal nodes to handle all segments correctly. This is why `4 * n` is used to ensure there is enough space for all the nodes, including internal nodes, to represent any possible division of the intervals.

### 2. **Dynamic Updates in a 2D Plane**:

In this specific problem of calculating the area covered by rectangles, the segment tree is used to manage **Y-intervals** dynamically as rectangles are added or removed during the sweep over the X-axis. The tree needs to efficiently handle:
- **Partial overlaps**: A segment of the Y-axis may be partially covered by multiple rectangles, and the segment tree needs to keep track of these overlaps at different levels in the tree. This requires more nodes to represent the internal structure and intermediate computations.
- **Full coverage**: For each interval, the segment tree needs to calculate whether the interval is fully or partially covered by one or more rectangles. This requires the tree to split intervals into finer segments as needed, which necessitates a larger tree size.

### 3. **Efficient Interval Management**:

When the segment tree is used in the context of managing dynamic intervals (like Y-intervals for overlapping rectangles), the number of internal nodes can grow quickly. A size of `4 * n` ensures that:
- **Full and partial overlaps** are handled correctly by the segment tree. This is important because rectangles may overlap on the Y-axis, and we need to account for areas covered by one or more rectangles while avoiding double-counting overlaps.
- **Propagation of updates**: Updates in a segment tree involve recursively propagating changes to parent nodes. If the tree size is too small (e.g., `2 * n`), the tree might not have enough space to properly propagate these updates for arbitrary interval lengths.

### 4. **`4 * n` Guarantees Enough Space for All Nodes**:

- The choice of `4 * n` ensures that the tree can handle any number of leaf nodes (`n`) and their corresponding internal nodes, even if `n` is not a power of 2.
- It guarantees that the tree can efficiently store and manage the necessary information (like `count[]` and `total[]`) for each interval, avoiding issues where the tree becomes unbalanced or incomplete.
- Even if `n` is relatively small, the extra space provided by `4 * n` ensures that the tree is flexible enough to handle the recursive updates and merges for dynamic intervals.

### Comparison Between `2 * n` and `4 * n`:

- **`2 * n`**: This is commonly used for simpler segment tree problems where the number of leaves is fixed, and there are no dynamic updates. This works well when `n` is a power of 2 and the tree structure is balanced.
- **`4 * n`**: This is used for more complex problems, like managing dynamic ranges or intervals, where the segment tree needs to handle arbitrary-sized inputs, partial overlaps, and dynamic updates. The extra space ensures that the tree can split and merge intervals efficiently without running out of space.

### Example to Illustrate:

Imagine you have 5 unique Y-coordinates (Y-values `[1, 2, 3, 4, 5]`). The segment tree must represent intervals like `[1, 2)`, `[2, 3)`, `[3, 4)`, etc. Using `2 * n` might not provide enough space to handle all the internal nodes when these intervals are divided further due to overlaps.

By using `4 * n`, you ensure that there is always enough space for all internal nodes, even when intervals overlap or are divided unevenly.

### Conclusion:

- Using **`4 * n`** instead of `2 * n` ensures that the segment tree can handle dynamic intervals, especially in problems involving **sweep line algorithms** or complex interval management (like the rectangle area problem).
- The additional space ensures that the segment tree can handle updates, merges, and splits of intervals correctly, without running into issues caused by an incomplete or unbalanced tree structure.


## Example
### Problem Recap

We are given a list of rectangles, where each rectangle is defined by its bottom-left corner \((x1, y1)\) and top-right corner \((x2, y2)\). We need to compute the total area covered by these rectangles, ensuring that overlapping regions are counted only once.

### Example Input

We'll use the following three rectangles as the example:
```plaintext
rectangles = [
    [1, 1, 4, 4],  # Rectangle 1: from (1, 1) to (4, 4)
    [2, 2, 5, 6],  # Rectangle 2: from (2, 2) to (5, 6)
    [3, 1, 6, 3]   # Rectangle 3: from (3, 1) to (6, 3)
]
```

### Step 1: Event Generation and Collecting Unique Y-Coordinates

For each rectangle, we generate two types of **events**:
- A **start event** when a rectangle begins at `x1`.
- An **end event** when a rectangle ends at `x2`.

Each event stores the X-coordinate of the event, the Y-interval \([y1, y2]\), and whether the event is starting (`+1`) or ending (`-1`).

**Generated Events**:
```plaintext
(1, 1, 4, 1)  # Start of Rectangle 1
(4, 1, 4, -1)  # End of Rectangle 1
(2, 2, 6, 1)  # Start of Rectangle 2
(5, 2, 6, -1)  # End of Rectangle 2
(3, 1, 3, 1)  # Start of Rectangle 3
(6, 1, 3, -1)  # End of Rectangle 3
```

We also collect all unique Y-coordinates from the rectangles: 
```plaintext
Unique Y-coordinates = [1, 2, 3, 4, 6]
```

### Step 2: Sorting the Events

Next, we **sort the events** based on their X-coordinate. If two events have the same X-coordinate, start events are processed before end events to ensure proper updates in the segment tree.

**Sorted Events**:
```plaintext
(1, 1, 4, 1)  # Start of Rectangle 1
(2, 2, 6, 1)  # Start of Rectangle 2
(3, 1, 3, 1)  # Start of Rectangle 3
(4, 1, 4, -1)  # End of Rectangle 1
(5, 2, 6, -1)  # End of Rectangle 2
(6, 1, 3, -1)  # End of Rectangle 3
```

### Step 3: Initializing the Segment Tree

We initialize a **segment tree** that operates over the **unique Y-coordinates**:
```plaintext
Y-coordinates = [1, 2, 3, 4, 6]
```

We need to map the Y-coordinates to indices that can be used by the segment tree. The **mapping** of Y-values to their indices:
```plaintext
ys_index = {1: 0, 2: 1, 3: 2, 4: 3, 6: 4}
```

### Step 4: Processing the Events Using the Sweep Line Algorithm

We now process each event, sweeping from left to right along the X-axis. We will update the segment tree and compute the area contribution from the current X-coordinate to the next one. 

- We initialize `prev_x = 1` (the X-coordinate of the first event).
- We initialize `area = 0` to keep track of the total area covered.

#### Event 1: (1, 1, 4, 1) (Start of Rectangle 1)
- **Current X = 1**
- Y-interval = \([1, 4)\)
- We update the segment tree to indicate that this Y-interval is now covered by a rectangle.
- **No area is calculated** yet because we haven't moved along the X-axis.

#### Event 2: (2, 2, 6, 1) (Start of Rectangle 2)
- **Current X = 2**
- Calculate the **covered Y-length** using the segment tree. 
- The Y-length covered by Rectangle 1 from \([1, 4)\) is `3`. 
- Calculate the **area**: \((2 - 1) \times 3 = 3\).
- Add `3` to `area`: `area = 3`.
- Y-interval for this rectangle is \([2, 6)\). Update the segment tree to reflect this new interval.

#### Event 3: (3, 1, 3, 1) (Start of Rectangle 3)
- **Current X = 3**
- Calculate the **covered Y-length** using the segment tree.
- The Y-length covered by Rectangle 1 and Rectangle 2 is `5` (from Y = 1 to Y = 4 for Rectangle 1 and Y = 2 to Y = 6 for Rectangle 2).
- Calculate the **area**: \((3 - 2) \times 5 = 5\).
- Add `5` to `area`: `area = 8`.
- Y-interval for this rectangle is \([1, 3)\). Update the segment tree.

#### Event 4: (4, 1, 4, -1) (End of Rectangle 1)
- **Current X = 4**
- Calculate the **covered Y-length** using the segment tree.
- The Y-length covered by Rectangle 2 and Rectangle 3 is `5` (from Y = 1 to Y = 3 for Rectangle 3 and Y = 2 to Y = 6 for Rectangle 2).
- Calculate the **area**: \((4 - 3) \times 5 = 5\).
- Add `5` to `area`: `area = 13`.
- Remove the Y-interval \([1, 4)\) from the segment tree, as Rectangle 1 ends.

#### Event 5: (5, 2, 6, -1) (End of Rectangle 2)
- **Current X = 5**
- Calculate the **covered Y-length** using the segment tree.
- The Y-length covered by Rectangle 3 is `5` (from Y = 1 to Y = 3).
- Calculate the **area**: \((5 - 4) \times 5 = 5\).
- Add `5` to `area`: `area = 18`.
- Remove the Y-interval \([2, 6)\) from the segment tree, as Rectangle 2 ends.

#### Event 6: (6, 1, 3, -1) (End of Rectangle 3)
- **Current X = 6**
- Calculate the **covered Y-length** using the segment tree.
- Now there are no covered intervals, so the Y-length is `2` (from Y = 1 to Y = 3).
- Calculate the **area**: \((6 - 5) \times 2 = 2\).
- Add `2` to `area`: `area = 20`.
- Remove the Y-interval \([1, 3)\) from the segment tree, as Rectangle 3 ends.

### Final Area Calculation

After processing all events, the total area covered by the rectangles is `20`. Since this is less than \(10^9 + 7\), we return `20` as the final answer.

### Summary of Area Calculations:
1. Between X = 1 and X = 2: Area = \(3\)
2. Between X = 2 and X = 3: Area = \(5\)
3. Between X = 3 and X = 4: Area = \(5\)
4. Between X = 4 and X = 5: Area = \(5\)
5. Between X = 5 and X = 6: Area = \(2\)

### Conclusion:
The **total area covered** by the rectangles is **20**, which is the correct result.
