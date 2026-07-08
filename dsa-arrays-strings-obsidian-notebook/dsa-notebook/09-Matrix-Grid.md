---
tags: [dsa, matrix, grid, bfs, dfs]
---

# 9. Matrix & Grid

⬅ [[00-Home]] | Prev: [[08-Arrays-Advanced-DS]] | Next: [[10-Strings-Foundations]]

---

## Matrix Traversal

> [!tip] When to use
> Any row/column scan, searching a sorted 2D matrix, general grid problems.

**Template (search in row+col sorted matrix, staircase search):**
```java
int row = 0, col = matrix[0].length - 1;
while (row < matrix.length && col >= 0) {
    if (matrix[row][col] == target) return true;
    else if (matrix[row][col] > target) col--;
    else row++;
}
return false;
```

**Complexity:** Time O(n + m) · Space O(1)

**Common Bugs:** starting from top-left (wrong corner — must start top-right or bottom-left for this trick to work).

**Related:** [[04-Arrays-Windows-Search#Binary Search|Binary Search]] (if fully sorted like a flattened 1D array)

**Must Solve:** Medium: Search a 2D Matrix (74), Search a 2D Matrix II (240)

---

## Matrix Rotation

> [!tip] When to use
> "Rotate image 90° clockwise/counter-clockwise in place."

**Intuition:** Transpose the matrix, then reverse each row (clockwise) or reverse each column first then transpose (counter-clockwise).

**Visual:**
```
Original       Transpose      Reverse rows (= rotate 90° CW)
1 2 3          1 4 7          7 4 1
4 5 6    ->    2 5 8    ->    8 5 2
7 8 9          3 6 9          9 6 3
```

**Complexity:** Time O(n²) · Space O(1) in-place

**Template:**
```java
// Transpose
for (int i = 0; i < n; i++)
    for (int j = i + 1; j < n; j++) {
        int t = matrix[i][j]; matrix[i][j] = matrix[j][i]; matrix[j][i] = t;
    }
// Reverse each row (clockwise 90°)
for (int[] row : matrix) {
    for (int l = 0, r = row.length - 1; l < r; l++, r--) {
        int t = row[l]; row[l] = row[r]; row[r] = t;
    }
}
```

**Common Bugs:** starting the inner transpose loop at `j = 0` instead of `j = i + 1` (double-swaps and undoes the transpose).

**Related:** [[09-Matrix-Grid#Transpose|Transpose]]

**Must Solve:** Medium: Rotate Image (48)

---

## Spiral Matrix

> [!tip] When to use
> "Print/traverse matrix in spiral order."

**Intuition:** Maintain 4 boundaries (top, bottom, left, right); traverse right along top, down along right, left along bottom, up along left; shrink boundaries after each side.

**Visual:**
```
1 2 3
4 5 6   -> spiral: 1,2,3,6,9,8,7,4,5
7 8 9
```

**Complexity:** Time O(n·m) · Space O(1) extra (excluding output)

**Template:**
```java
int top = 0, bottom = rows - 1, left = 0, right = cols - 1;
List<Integer> result = new ArrayList<>();
while (top <= bottom && left <= right) {
    for (int j = left; j <= right; j++) result.add(matrix[top][j]);
    top++;
    for (int i = top; i <= bottom; i++) result.add(matrix[i][right]);
    right--;
    if (top <= bottom) {
        for (int j = right; j >= left; j--) result.add(matrix[bottom][j]);
        bottom--;
    }
    if (left <= right) {
        for (int i = bottom; i >= top; i--) result.add(matrix[i][left]);
        left++;
    }
}
```

**Common Bugs:** forgetting the extra `top <= bottom` / `left <= right` guards for the last two sides (double-counts elements on single-row/column matrices).

**Edge Cases:** single row, single column, 1×1 matrix, non-square matrix.

**Must Solve:** Medium: Spiral Matrix (54), Spiral Matrix II (59)

---

## Diagonal Traversal

> [!tip] When to use
> "Print matrix diagonally", zig-zag diagonal order.

**Intuition:** Elements on the same diagonal share `i + j` (top-left to bottom-right diagonals) or `i - j` (anti-diagonals). Group by that value.

**Template:**
```java
Map<Integer, List<Integer>> diagonals = new TreeMap<>();
for (int i = 0; i < rows; i++)
    for (int j = 0; j < cols; j++)
        diagonals.computeIfAbsent(i + j, k -> new ArrayList<>()).add(matrix[i][j]);
// reverse alternate diagonals for the zig-zag LeetCode variant
```

**Complexity:** Time O(n·m) · Space O(n·m)

**Common Bugs:** forgetting to reverse every other diagonal for the classic zig-zag output problem.

**Must Solve:** Medium: Diagonal Traverse (498)

---

## Transpose

> [!tip] When to use
> Swap rows and columns — building block for rotation, checking matrix symmetry.

**Template:**
```java
int[][] transpose(int[][] matrix) {
    int rows = matrix.length, cols = matrix[0].length;
    int[][] result = new int[cols][rows];
    for (int i = 0; i < rows; i++)
        for (int j = 0; j < cols; j++)
            result[j][i] = matrix[i][j];
    return result;
}
```

**Complexity:** Time O(n·m) · Space O(n·m) (or O(1) if square and in-place)

**Must Solve:** Easy: Transpose Matrix (867)

---

## Flood Fill

> [!tip] When to use
> "Paint bucket", connected region coloring starting from a single cell.

**Intuition:** DFS/BFS from the starting cell, spreading to 4-directionally adjacent cells with the same original value/color.

**Template (DFS):**
```java
void floodFill(int[][] image, int r, int c, int oldColor, int newColor) {
    if (r < 0 || r >= image.length || c < 0 || c >= image[0].length) return;
    if (image[r][c] != oldColor || image[r][c] == newColor) return;
    image[r][c] = newColor;
    floodFill(image, r+1, c, oldColor, newColor);
    floodFill(image, r-1, c, oldColor, newColor);
    floodFill(image, r, c+1, oldColor, newColor);
    floodFill(image, r, c-1, oldColor, newColor);
}
```

**Common Bugs:** forgetting the `oldColor == newColor` guard (infinite recursion if colors match); not bounds-checking before array access.

**Related:** [[09-Matrix-Grid#BFS Grid|BFS Grid]], [[09-Matrix-Grid#DFS Grid|DFS Grid]]

**Must Solve:** Easy: Flood Fill (733) · Medium: Number of Islands (200)

---

## BFS Grid

> [!tip] When to use
> Shortest path in an unweighted grid, "minimum steps to reach", multi-source spreading (rotting oranges).

**Template:**
```java
int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};
Queue<int[]> queue = new LinkedList<>();
boolean[][] visited = new boolean[rows][cols];
queue.offer(new int[]{startR, startC});
visited[startR][startC] = true;
int steps = 0;
while (!queue.isEmpty()) {
    int size = queue.size();
    for (int i = 0; i < size; i++) {
        int[] cur = queue.poll();
        if (cur[0] == targetR && cur[1] == targetC) return steps;
        for (int[] d : dirs) {
            int nr = cur[0] + d[0], nc = cur[1] + d[1];
            if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && !visited[nr][nc] && grid[nr][nc] != obstacle) {
                visited[nr][nc] = true;
                queue.offer(new int[]{nr, nc});
            }
        }
    }
    steps++;
}
return -1;
```

**Common Bugs:** marking visited on `poll` instead of `offer` (causes duplicate enqueues, exponential blowup); forgetting the level-by-level `size` snapshot when step-counting is needed.

**Related:** [[09-Matrix-Grid#DFS Grid|DFS Grid]] (better for existence/counting, not shortest path)

**Must Solve:** Medium: Rotting Oranges (994), 01 Matrix (542) · Hard: Shortest Path in Binary Matrix (1091)

---

## DFS Grid

> [!tip] When to use
> "Count connected components/islands", exhaustive path exploration, backtracking on a grid.

**Template:**
```java
void dfs(int[][] grid, int r, int c, boolean[][] visited) {
    if (r < 0 || r >= grid.length || c < 0 || c >= grid[0].length) return;
    if (visited[r][c] || grid[r][c] == 0) return;
    visited[r][c] = true;
    for (int[] d : new int[][]{{0,1},{0,-1},{1,0},{-1,0}}) {
        dfs(grid, r + d[0], c + d[1], visited);
    }
}
```

**Common Bugs:** stack overflow on very large grids with recursive DFS (convert to iterative with an explicit stack if grid is huge); forgetting to mark visited before recursing (infinite loop).

**Related:** [[08-Arrays-Advanced-DS#Union Find (Disjoint Set Union — Overview)|Union Find]] (alternative for counting components)

**Must Solve:** Medium: Number of Islands (200), Max Area of Island (695) · Hard: Making A Large Island (827)

Next: [[10-Strings-Foundations]]
