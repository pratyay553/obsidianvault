
I’ll structure this like a **developer reference map**, not just an API list.

A useful way to think about collections/data structures is:

> Every operation is a question you ask the structure.

Examples:

- "Can I find this quickly?"
    
- "Can I insert here?"
    
- "Can I remove this?"
    
- "Can I maintain order?"
    
- "Can I iterate?"
    
- "Can I transform?"
    

Below is a comprehensive operation map for Java Collections + common DSA structures.

---

# 1. Universal Collection Operations

These exist in most collection types.

|Operation|Meaning|Why it exists|
|---|---|---|
|`add()`|Insert element|Grow the collection|
|`addAll()`|Insert multiple elements|Bulk insertion|
|`remove()`|Delete element|Modify structure|
|`removeAll()`|Remove multiple elements|Bulk deletion|
|`contains()`|Check existence|Membership testing|
|`containsAll()`|Check multiple elements|Set comparison|
|`size()`|Number of elements|Track state|
|`isEmpty()`|Check empty|Avoid unnecessary processing|
|`clear()`|Remove everything|Reset collection|
|`iterator()`|Traverse elements|Generic traversal|
|`toArray()`|Convert to array|Interoperability|
|`equals()`|Compare collections|Value comparison|
|`hashCode()`|Hash representation|Hash-based storage|

---

# 2. List Operations (`ArrayList`, `LinkedList`, `Vector`)

Lists maintain:

- order
    
- duplicates
    
- index positions
    

---

## Index Operations

|Operation|Purpose|
|---|---|
|`get(index)`|Read element|
|`set(index,value)`|Replace element|
|`add(index,value)`|Insert at position|
|`remove(index)`|Delete by position|
|`indexOf(value)`|First occurrence|
|`lastIndexOf(value)`|Last occurrence|

---

## List Specific

|Operation|Purpose|
|---|---|
|`listIterator()`|Bidirectional traversal|
|`subList()`|View a range|

Example:

```java
list.subList(1,5)
```

---

# 3. ArrayList Operations

Backed by:

```text
Object[]
```

---

|Operation|Complexity|
|---|---|
|`add(end)`|O(1) amortized|
|`get(index)`|O(1)|
|`set(index)`|O(1)|
|`remove(index)`|O(n)|
|`contains()`|O(n)|
|`sort()`|O(n log n)|
|`ensureCapacity()`|Preallocate memory|
|`trimToSize()`|Reduce unused memory|

---

# 4. LinkedList Operations

Backed by:

```text
Node <-> Node <-> Node
```

---

## Deque Operations

Because Java LinkedList implements Deque:

|Operation|Meaning|
|---|---|
|`addFirst()`|Insert beginning|
|`addLast()`|Insert end|
|`removeFirst()`|Delete beginning|
|`removeLast()`|Delete end|
|`getFirst()`|Read first|
|`getLast()`|Read last|
|`peekFirst()`|Look first safely|
|`peekLast()`|Look last safely|

---

# 5. Set Operations (`HashSet`, `TreeSet`, `LinkedHashSet`)

Set means:

- unique elements
    
- no duplicates
    

---

## Basic

|Operation|Meaning|
|---|---|
|`add()`|Insert unique value|
|`remove()`|Delete value|
|`contains()`|Check existence|

---

## Mathematical Set Operations

### Union

A ∪ B

```java
addAll()
```

---

### Intersection

A ∩ B

```java
retainAll()
```

---

### Difference

A - B

```java
removeAll()
```

---

# 6. HashSet Operations

Internally:

```text
HashMap
   |
key = element
```

---

|Operation|Complexity|
|---|---|
|add|O(1) average|
|remove|O(1) average|
|contains|O(1) average|

---

# 7. TreeSet Operations

Uses:

```text
Red Black Tree
```

Maintains sorted order.

---

|Operation|Purpose|
|---|---|
|`first()`|Smallest|
|`last()`|Largest|
|`lower(x)`|Less than x|
|`floor(x)`|<= x|
|`ceiling(x)`|>= x|
|`higher(x)`|Greater than x|

---

# 8. Queue Operations

FIFO:

```text
First In First Out
```

---

|Operation|Meaning|
|---|---|
|`offer()`|Add|
|`poll()`|Remove|
|`peek()`|View front|

---

# 9. Deque Operations

Double-ended queue.

---

## Front

```java
addFirst()
removeFirst()
peekFirst()
```

---

## Back

```java
addLast()
removeLast()
peekLast()
```

---

# 10. Stack Operations

LIFO:

```text
Last In First Out
```

---

|Operation|Meaning|
|---|---|
|`push()`|Add|
|`pop()`|Remove|
|`peek()`|View top|

Modern Java prefers:

```java
ArrayDeque
```

instead of Stack.

---

# 11. PriorityQueue Operations

Heap structure.

Used for:

- Top K
    
- scheduling
    
- shortest path
    

---

|Operation|Meaning|
|---|---|
|`offer()`|Insert|
|`poll()`|Remove highest priority|
|`peek()`|View highest priority|

---

Complexity:

|Operation|Cost|
|---|---|
|Insert|O(log n)|
|Remove|O(log n)|
|Peek|O(1)|

---

# 12. Map Operations

Maps store:

```text
Key -> Value
```

---

## Basic

|Operation|Meaning|
|---|---|
|`put()`|Insert/update|
|`get()`|Retrieve|
|`remove()`|Delete|
|`containsKey()`|Check key|
|`containsValue()`|Check value|
|`size()`|Count|
|`clear()`|Remove all|

---

## Safer Access

|Operation|Purpose|
|---|---|
|`getOrDefault()`|Avoid null|
|`putIfAbsent()`|Insert only missing|
|`replace()`|Update existing|

---

## Functional Map Operations

Java 8+

### Compute

```java
compute()
```

Recalculate value.

---

### Compute If Missing

```java
computeIfAbsent()
```

Common in graphs.

Example:

```java
map.computeIfAbsent(node,k->new ArrayList<>());
```

---

### Merge

```java
merge()
```

Frequency counting.

Example:

```java
map.merge(x,1,Integer::sum);
```

---

# 13. HashMap Operations

Internally:

```text
Array
 |
Buckets
 |
LinkedList / Tree
```

---

|Operation|Average|
|---|---|
|put|O(1)|
|get|O(1)|
|remove|O(1)|

---

# 14. TreeMap Operations

Sorted map.

---

|Operation|Meaning|
|---|---|
|firstKey()|Smallest key|
|lastKey()|Largest key|
|floorKey()|<= key|
|ceilingKey()|>= key|
|higherKey()|> key|
|lowerKey()|< key|

---

# 15. Iteration Operations

Every collection eventually needs traversal.

---

## Iterator

```java
iterator()
```

Operations:

```java
hasNext()

next()

remove()
```

---

## ListIterator

Extra:

```java
hasPrevious()

previous()

add()

set()
```

---

# 16. Stream Operations

Streams are not storage.

They process data.

---

## Intermediate

Returns another stream.

|Operation|
|---|
|filter|
|map|
|flatMap|
|distinct|
|sorted|
|limit|
|skip|
|peek|

---

## Terminal

Consumes stream.

|Operation|
|---|
|collect|
|reduce|
|count|
|min|
|max|
|findFirst|
|findAny|
|anyMatch|
|allMatch|
|noneMatch|
|forEach|

---

# 17. Common DSA Data Structure Operations

---

# Array

|Operation|Cost|
|---|---|
|Access|O(1)|
|Search|O(n)|
|Insert|O(n)|
|Delete|O(n)|

---

# Linked List

|Operation|Cost|
|---|---|
|Access|O(n)|
|Insert head|O(1)|
|Delete node|O(1) if reference exists|
|Search|O(n)|

---

# Stack

Operations:

```text
push
pop
peek
```

---

# Queue

Operations:

```text
enqueue
dequeue
front
```

---

# Heap

Operations:

```text
insert
extractMin
extractMax
heapify
```

---

# Graph

Operations:

```text
addVertex()

addEdge()

removeVertex()

removeEdge()

BFS()

DFS()

shortestPath()
```

---

# Trie

Operations:

```text
insert(word)

search(word)

startsWith(prefix)

delete(word)
```

---

# Union Find (Disjoint Set)

Operations:

```text
find()

union()

connected()
```

---

# The Mental Map

When solving DSA:

Ask:

| Need                       | Think      |     |
| -------------------------- | ---------- | --- |
| Fast lookup                | HashMap    |     |
| Unique values              | HashSet    |     |
| Sorted values              | TreeSet    |     |
| Sorted key-value           | TreeMap    |     |
| Index access               | ArrayList  |     |
| Frequent insertion/removal | LinkedList |     |
| FIFO                       | Queue      |     |
| LIFO                       | Stack      |     |
| Highest priority           | Heap       |     |
| Relationships              | Graph      |     |
| Prefix search              | Trie       |     |

---

# The deepest rule:

**A data structure is not defined by its methods.  
It is defined by which operations it makes cheap.**

Your DSA skill is recognizing:

> "What operation dominates this problem?"

Then choose the structure that makes that operation efficient.