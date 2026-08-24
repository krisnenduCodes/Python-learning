# Python Objects, References and Memory

> **Learning Note:** Python Basics
> **Topic:** Objects, References, Containers and `sys.getsizeof()`

## 1. What I Learned

While learning Python, I came across an important concept about how Python stores values in memory.

The important idea is:

> **In Python, values are objects, and variable names refer to those objects.**

This became clearer to me when I started learning about `sys.getsizeof()`.

---

## 2. Variables and Objects

Consider:

```python
x = 10
```

A simple mental model is:

```text
x ───────────► 10
               ↑
            int object
```

Here:

* `10` is an **integer object**.
* `x` is a **name/reference** that refers to that object.
* The variable name `x` should not be thought of as a box that directly contains the value `10`.

We can check that `10` is an object using:

```python
x = 10

print(type(x))
```

Output:

```text
<class 'int'>
```

This means that `x` refers to an object whose type is `int`.

---

## 3. What Does `sys.getsizeof()` Do?

Python provides the `sys.getsizeof()` function to find the size of an object.

Example:

```python
import sys

x = 10

print(sys.getsizeof(x))
```

On a typical 64-bit CPython installation, this may print:

```text
28
```

The exact value can depend on the Python implementation and version.

The important thing is not to memorize the number.

The important thing is to understand **what is being measured**.

```python
sys.getsizeof(x)
```

measures the size of the **integer object referred to by `x`**.

It does not measure the size of the variable name `x` or simply the reference itself.

Mental model:

```text
x ───────────► [ 10 ]
                  ↑
          sys.getsizeof(x)
          measures this object
```

---

## 4. What Happens With Containers?

Now consider a list:

```python
lst = [10, 20, 30]
```

A simplified mental model is:

```text
lst
 │
 ▼
┌────────┬────────┬────────┐
│  ref   │  ref   │  ref   │
└───┬────┴───┬────┴───┬────┘
    │        │        │
    ▼        ▼        ▼
   10       20       30
```

The list contains references to the integer objects.

If I run:

```python
sys.getsizeof(lst)
```

Python measures the **list object itself**.

It does not recursively follow every reference and add the size of `10`, `20`, and `30`.

This is an important distinction.

---

## 5. Understanding the Difference

Consider:

```python
lst = [10, 20, 30]
```

### Size of the list

```python
sys.getsizeof(lst)
```

This measures the list object's own memory, including its internal storage for references.

### Size of an element

```python
sys.getsizeof(lst[0])
```

This measures the integer object `10`.

### Size of the list + its directly contained objects

For a simple flat list, I can estimate it with:

```python
total_size = sys.getsizeof(lst)

for element in lst:
    total_size += sys.getsizeof(element)

print(total_size)
```

This adds:

```text
list object
+
integer 10
+
integer 20
+
integer 30
```

---

## 6. A Large Example

I experimented with a list containing one million integers:

```python
import sys

size = 1_000_000
lst1 = list(range(size))

total_size = sys.getsizeof(lst1)

for element in lst1:
    total_size += sys.getsizeof(element)

print(total_size)

print(sys.getsizeof(lst1) * len(lst1))
```

The two calculations represent completely different things.

### First calculation

```python
total_size = sys.getsizeof(lst1)

for element in lst1:
    total_size += sys.getsizeof(element)
```

Conceptually:

```text
size of list
+
size of integer 0
+
size of integer 1
+
size of integer 2
+
...
+
size of integer 999999
```

This gives an estimate of the list plus the sizes of the objects directly contained in it.

### Second calculation

```python
sys.getsizeof(lst1) * len(lst1)
```

This means:

```text
size of list × number of elements
```

It does **not** mean the actual memory used by the list.

For example, if the list itself were approximately 8 MB, multiplying it by 1,000,000 would produce an enormous number. That doesn't represent the actual memory consumption.

The mistake is treating the size of the list container as if every element occupied the same amount of memory as the entire list.

---

## 7. My Initial Confusion

Initially, I thought:

> If a variable contains a reference to an object, then `sys.getsizeof(variable)` should measure the size of the reference/pointer.

But this is not what happens.

For:

```python
x = 10
```

the important mental model is:

```text
x ─────────► [ integer object: 10 ]
```

Therefore:

```python
sys.getsizeof(x)
```

measures the integer object referred to by `x`.

For a container:

```python
lst = [10, 20, 30]
```

the list itself contains references:

```text
lst
 │
 ▼
┌─────────┬─────────┬─────────┐
│   ref   │   ref   │   ref   │
└────┬────┴────┬────┴────┬────┘
     ▼         ▼         ▼
    10        20        30
```

`sys.getsizeof(lst)` measures the list object and its internal storage, but it does not recursively measure the integer objects.

This distinction helped me finally understand how `sys.getsizeof()` works.

---

## 8. Important Limitation of `sys.getsizeof()`

`sys.getsizeof()` is not a complete memory profiler.

For example:

```python
lst = [[1, 2], [3, 4]]
```

Calling:

```python
sys.getsizeof(lst)
```

does not recursively calculate:

```text
outer list
+
inner list [1, 2]
+
inner list [3, 4]
+
integer 1
+
integer 2
+
integer 3
+
integer 4
```

Therefore, for complex nested objects, simply calling `sys.getsizeof()` on the outer object does not tell me the complete memory footprint of everything reachable from it.

---

## 9. Key Takeaways

The main things I want to remember are:

1. **Values in Python are objects.**
2. A variable name refers to an object.
3. `sys.getsizeof(x)` measures the object referred to by `x`.
4. It does not simply measure the reference/pointer.
5. For a container, `sys.getsizeof()` measures the container itself.
6. It does not recursively measure all objects contained inside the container.
7. For a simple flat container, I can add the sizes of the contained objects separately.
8. For complex nested structures, `sys.getsizeof()` alone is not a complete memory profiler.
9. I should remember the **concept**, not memorize exact byte values.

---

## 10. My Mental Model

The simplest way I currently understand it is:

```text
Variable
   │
   │ reference
   ▼
Object
```

For a normal object:

```text
x ─────────► [ object ]
```

For a container:

```text
list ──────► [ reference | reference | reference ]
                   │           │           │
                   ▼           ▼           ▼
                 object      object      object
```

And:

```python
sys.getsizeof(x)
```

means:

> **"How much memory does the object referred to by `x` itself occupy?"**

For a container:

> **"How much memory does this container object itself occupy?"**

It does not automatically mean:

> **"How much memory does everything inside this container occupy?"**

---

## 11. Small Experiment

I can verify these ideas myself:

```python
import sys

x = 10
lst = [10, 20, 30]

print("Integer:", sys.getsizeof(x))
print("List:", sys.getsizeof(lst))
print("First element:", sys.getsizeof(lst[0]))

total = sys.getsizeof(lst)

for element in lst:
    total += sys.getsizeof(element)

print("List + elements:", total)
```

The purpose of this experiment is not to memorize the output.

The purpose is to observe the difference between:

```text
object size
container size
container + contained objects
```

---

## Conclusion

The most important lesson from this topic is that I should distinguish between **an object, a reference to an object, and a container holding references to other objects**.

Once this mental model is clear, `sys.getsizeof()` becomes much easier to understand.

> **Don't memorize the numbers. Understand what object is actually being measured.**
