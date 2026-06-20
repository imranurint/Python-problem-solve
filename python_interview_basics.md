# Basic Python Coding Problems for Backend Engineers
## Bangladesh Tech Interview Prep

---

## PROBLEM 1: Filter and Sort User List
**Scenario:** You have a list of users (dictionaries). Filter users from Bangladesh with age > 25, sort by age descending.

**Problem:**
```python
users = [
    {"name": "Amran", "country": "Bangladesh", "age": 28},
    {"name": "Karim", "country": "India", "age": 32},
    {"name": "Fatima", "country": "Bangladesh", "age": 22},
    {"name": "Sarah", "country": "Bangladesh", "age": 30},
]

# Filter: country == "Bangladesh" AND age > 25
# Sort by age (descending)
# Return list of names
```

**Solution:**
```python
# Method 1: Loop (verbose but clear)
result = []
for user in users:
    if user["country"] == "Bangladesh" and user["age"] > 25:
        result.append(user)

result.sort(key=lambda u: u["age"], reverse=True)
names = [u["name"] for u in result]
print(names)  # Output: ['Sarah', 'Amran']
```

**Better (Pythonic):**
```python
# Method 2: Filter + sort one-liner
filtered = sorted(
    [u for u in users if u["country"] == "Bangladesh" and u["age"] > 25],
    key=lambda u: u["age"],
    reverse=True
)
names = [u["name"] for u in filtered]
print(names)  # Output: ['Sarah', 'Amran']
```

**Key concepts:**
- `filter()` or list comprehension `[...]`
- `sorted()` with `key=` parameter
- Dictionary access with `[]`
- Lambda functions for custom sorting

**Interview talking point:**
"I use list comprehensions because they're Pythonic and readable. For production, I'd check the list size—if it's huge (1M+), I'd do this in SQL instead of Python."

---

## PROBLEM 2: Count Word Frequency
**Scenario:** You have a customer support query. Count how many times each word appears (case-insensitive).

**Problem:**
```python
text = "I want to refund my order. Can you help with refund? I need refund"

# Count word frequency
# Output: {'want': 1, 'refund': 3, 'help': 1, ...}
```

**Solution:**
```python
# Method 1: Dictionary
text = "I want to refund my order. Can you help with refund? I need refund"
words = text.lower().replace("?", "").replace(".", "").split()
freq = {}
for word in words:
    freq[word] = freq.get(word, 0) + 1

print(freq)
# Output: {'i': 2, 'want': 1, 'to': 1, 'refund': 3, 'my': 1, ...}
```

**Better (Python built-in):**
```python
from collections import Counter

text = "I want to refund my order. Can you help with refund? I need refund"
words = text.lower().split()
freq = Counter(words)
print(freq)
print(freq["refund"])  # Output: 3
print(freq.most_common(3))  # Top 3: [('refund', 3), ('i', 2), ('want', 1)]
```

**Key concepts:**
- String methods: `.lower()`, `.split()`, `.replace()`
- Dictionary `.get()` with default
- `Counter` from collections module
- `most_common()` method

**Interview talking point:**
"For spam detection in customer support, I'd use Counter to find high-frequency words. If 'refund' appears 10x in a 50-word message, it's probably a complaint. I'd log this event and escalate to a human agent."

---

## PROBLEM 3: Check if List is Sorted
**Scenario:** You receive a batch of order IDs. Verify they're in ascending order.

**Problem:**
```python
orders = [101, 105, 103, 108]
# Is this sorted? Return True/False

orders2 = [101, 105, 108, 120]
# Is this sorted? Return True/False
```

**Solution:**
```python
# Method 1: Compare with sorted version
def is_sorted(arr):
    return arr == sorted(arr)

print(is_sorted([101, 105, 103, 108]))  # False
print(is_sorted([101, 105, 108, 120]))  # True
```

**Better (efficient):**
```python
# Method 2: One-pass loop (O(n) instead of O(n log n))
def is_sorted(arr):
    for i in range(len(arr) - 1):
        if arr[i] > arr[i + 1]:
            return False
    return True

print(is_sorted([101, 105, 108, 120]))  # True
```

**Most Pythonic:**
```python
# Method 3: zip magic (compare each pair)
def is_sorted(arr):
    return all(arr[i] <= arr[i+1] for i in range(len(arr)-1))

# Or even simpler:
def is_sorted(arr):
    return all(a <= b for a, b in zip(arr, arr[1:]))
```

**Key concepts:**
- List comparison with `==`
- `range()` and indexing
- `all()` function
- `zip()` for pairing elements

**Interview talking point:**
"The first solution is readable but slow (O(n log n) sorting). The loop version is O(n) which matters for 1M orders. I'd use the efficient version in production."

---

## PROBLEM 4: Flatten Nested List
**Scenario:** API returns nested category structure. Flatten it into a single list.

**Problem:**
```python
categories = [
    ["Electronics", "Mobile", "Laptop"],
    ["Clothing"],
    ["Food", "Groceries", "Dairy"]
]
# Output: ["Electronics", "Mobile", "Laptop", "Clothing", "Food", "Groceries", "Dairy"]
```

**Solution:**
```python
# Method 1: Loop
result = []
for sublist in categories:
    for item in sublist:
        result.append(item)
print(result)
```

**Better (list comprehension):**
```python
# Method 2: Nested list comprehension
result = [item for sublist in categories for item in sublist]
print(result)
```

**Most Pythonic:**
```python
# Method 3: itertools
from itertools import chain
result = list(chain(*categories))
print(result)

# Or:
from itertools import chain
result = list(chain.from_iterable(categories))
```

**Key concepts:**
- Nested loops in list comprehension
- `itertools.chain()`
- `*unpacking` operator

**Interview talking point:**
"For deeply nested data (like JSON responses), I'd use recursion. But for 1-2 levels, list comprehension is fastest and clearest."

---

## PROBLEM 5: Remove Duplicates (Preserve Order)
**Scenario:** User submitted form multiple times. Remove duplicate entries but keep first occurrence.

**Problem:**
```python
emails = ["amran@gmail.com", "karim@gmail.com", "amran@gmail.com", "fatima@gmail.com"]
# Remove duplicates, keep order
# Output: ["amran@gmail.com", "karim@gmail.com", "fatima@gmail.com"]
```

**Solution:**
```python
# Method 1: Loop with set tracking
seen = set()
result = []
for email in emails:
    if email not in seen:
        seen.add(email)
        result.append(email)
print(result)
```

**Shorter (dict preserves insertion order in Python 3.7+):**
```python
# Method 2: dict.fromkeys()
result = list(dict.fromkeys(emails))
print(result)
```

**If order doesn't matter:**
```python
# Method 3: Set (loses order, but fastest)
result = list(set(emails))
print(result)  # Order not guaranteed
```

**Key concepts:**
- Set for O(1) membership check
- `set.add()`
- `dict.fromkeys()`
- Order preservation

**Interview talking point:**
"I use Method 1 in production because it's explicit and efficient O(n). The dict.fromkeys() trick works but less readable. For order-agnostic dedup, set() is fastest."

---

## PROBLEM 6: Group By Category
**Scenario:** You have a list of orders. Group by customer ID.

**Problem:**
```python
orders = [
    {"customer_id": 1, "amount": 100},
    {"customer_id": 2, "amount": 50},
    {"customer_id": 1, "amount": 75},
    {"customer_id": 3, "amount": 200},
    {"customer_id": 1, "amount": 120},
]
# Output: {
#     1: [100, 75, 120],
#     2: [50],
#     3: [200],
# }
```

**Solution:**
```python
# Method 1: Loop with defaultdict
from collections import defaultdict

grouped = defaultdict(list)
for order in orders:
    grouped[order["customer_id"]].append(order["amount"])

print(dict(grouped))
```

**Method 2: Manual dictionary:**
```python
grouped = {}
for order in orders:
    cid = order["customer_id"]
    if cid not in grouped:
        grouped[cid] = []
    grouped[cid].append(order["amount"])

print(grouped)
```

**For more complex grouping:**
```python
# Method 3: Using itertools.groupby (requires sorted input)
from itertools import groupby

sorted_orders = sorted(orders, key=lambda x: x["customer_id"])
grouped = {k: [o["amount"] for o in g] for k, g in groupby(sorted_orders, key=lambda x: x["customer_id"])}
print(grouped)
```

**Key concepts:**
- `defaultdict(list)` to avoid KeyError
- Dictionary creation
- Sorting + `groupby()`

**Interview talking point:**
"In production, I'd do this in SQL with GROUP BY. In Python, defaultdict is cleanest. If the dataset is huge (1M+ orders), I'd use Pandas or PySpark instead."

---

## PROBLEM 7: Two Sum (Find Pair)
**Scenario:** Find two prices that sum to a target.

**Problem:**
```python
prices = [2, 7, 11, 15, 3]
target = 9
# Find two numbers that sum to 9
# Output: (2, 7) or [2, 7]
```

**Solution:**
```python
# Method 1: Brute force (slow, but simple)
def two_sum(prices, target):
    for i in range(len(prices)):
        for j in range(i + 1, len(prices)):
            if prices[i] + prices[j] == target:
                return [prices[i], prices[j]]
    return None

print(two_sum(prices, 9))  # [2, 7]
```

**Better (O(n) instead of O(n²)):**
```python
# Method 2: Hash set
def two_sum(prices, target):
    seen = set()
    for price in prices:
        complement = target - price
        if complement in seen:
            return [complement, price]
        seen.add(price)
    return None

print(two_sum(prices, 9))  # [2, 7]
```

**Key concepts:**
- Brute force (nested loops)
- Hash set for O(1) lookup
- Complement trick (target - current = complement)

**Interview talking point:**
"Brute force is O(n²)—fine for 1000 items, fails for 1M. Hash set is O(n) which scales. In production, I'd do this in SQL with a self-join on the price table."

---

## PROBLEM 8: Count Characters in String
**Scenario:** Analyze customer feedback. Count each character frequency.

**Problem:**
```python
text = "hello world"
# Output: {'h': 1, 'e': 1, 'l': 3, 'o': 2, ' ': 1, 'w': 1, 'r': 1, 'd': 1}
```

**Solution:**
```python
# Method 1: Dictionary
text = "hello world"
freq = {}
for char in text:
    freq[char] = freq.get(char, 0) + 1

print(freq)
```

**Method 2: Counter:**
```python
from collections import Counter

freq = Counter(text)
print(freq)
print(freq['l'])  # Output: 3
print(freq.most_common(3))  # Top 3: [('l', 3), ('o', 2), ('h', 1)]
```

**Key concepts:**
- Character iteration
- Dictionary `.get()`
- `Counter` class
- `most_common()`

---

## PROBLEM 9: Find Missing Number
**Scenario:** User IDs should be 1-100. Find which one is missing from a list.

**Problem:**
```python
user_ids = [1, 2, 3, 5, 6, 7, 8, 9, 10]
# Missing from 1-10: 4
# Output: 4
```

**Solution:**
```python
# Method 1: Set difference
def find_missing(arr, n):
    expected = set(range(1, n + 1))
    actual = set(arr)
    missing = expected - actual
    return missing.pop() if missing else None

print(find_missing(user_ids, 10))  # 4
```

**Method 2: Sum formula:**
```python
# Mathematical approach (fast)
def find_missing(arr, n):
    expected_sum = n * (n + 1) // 2
    actual_sum = sum(arr)
    return expected_sum - actual_sum

print(find_missing(user_ids, 10))  # 4
```

**Key concepts:**
- Set operations (difference `-`)
- Sum formula for arithmetic series
- Integer division `//`

---

## PROBLEM 10: Reverse a String / List
**Scenario:** Reverse order of comments or logs.

**Problem:**
```python
text = "hello"
# Output: "olleh"

arr = [1, 2, 3, 4, 5]
# Output: [5, 4, 3, 2, 1]
```

**Solution:**
```python
# String
text = "hello"
reversed_text = text[::-1]  # Slice with step -1
print(reversed_text)  # "olleh"

# List
arr = [1, 2, 3, 4, 5]
reversed_arr = arr[::-1]
print(reversed_arr)  # [5, 4, 3, 2, 1]

# Or in-place reverse
arr.reverse()
print(arr)  # [5, 4, 3, 2, 1]
```

**Key concepts:**
- Slicing with `[::-1]`
- `.reverse()` method
- String immutability vs list mutability

---

## 🎯 MOST IMPORTANT PATTERNS FOR INTERVIEWS

### Pattern 1: List Comprehension
```python
# Filter + map in one line
squared = [x**2 for x in range(5) if x % 2 == 0]  # [0, 4, 16]
```

### Pattern 2: Dictionary Comprehension
```python
# Create dict from list
prices = [10, 20, 30]
price_dict = {i: price for i, price in enumerate(prices)}  # {0: 10, 1: 20, 2: 30}
```

### Pattern 3: Lambda + Map/Filter
```python
# Transform list
numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x**2, numbers))  # [1, 4, 9, 16, 25]

# Filter list
evens = list(filter(lambda x: x % 2 == 0, numbers))  # [2, 4]
```

### Pattern 4: Sorting Custom Objects
```python
users = [{"name": "A", "age": 30}, {"name": "B", "age": 25}]
sorted_users = sorted(users, key=lambda u: u["age"])  # Sort by age
```

### Pattern 5: Dictionary Get with Default
```python
user = {"name": "Amran"}
age = user.get("age", 0)  # Returns 0 if "age" doesn't exist
```

---

## 📌 QUICK CHEAT SHEET

| Task | Code |
|------|------|
| Filter list | `[x for x in lst if condition]` |
| Map/transform | `[transform(x) for x in lst]` |
| Sort | `sorted(lst, key=lambda x: x.attr)` |
| Count frequency | `Counter(lst).most_common(3)` |
| Group by | `defaultdict(list)` |
| Flatten | `[item for sublist in lst for item in sublist]` |
| Remove duplicates | `list(dict.fromkeys(lst))` |
| String reverse | `string[::-1]` |
| Find in O(1) | `x in set_var` |
| Default dict access | `d.get(key, default)` |

---

## 🚨 THINGS TO AVOID IN INTERVIEWS

**DON'T:**
```python
# ❌ Don't use unclear variable names
d = [1, 2, 3]  # What is 'd'?

# ❌ Don't nest too deep
result = [[x*2 for x in row] for row in matrix if any(x > 5 for x in row)]  # Unreadable

# ❌ Don't write code without explaining
# Just write code and wait — WRONG!

# ✅ DO Explain as you write
# "I'll use a set to track seen items because lookup is O(1)"
# [Then write code]
```

**DO:**
```python
# ✅ Clear variable names
filtered_users = [u for u in users if u["age"] > 25]

# ✅ Break into steps
seen = set()
for email in emails:
    if email not in seen:
        seen.add(email)
        # Process email

# ✅ Explain before coding
# "I'll use a hash set to avoid O(n²) complexity"
```

---

## 💡 HOW TO APPROACH CODING PROBLEMS IN INTERVIEWS

1. **Read the problem** (1 min)
2. **Ask clarifying questions** (1 min)
   - "Can the input be empty? Can we have duplicates?"
3. **Explain your approach** (1 min)
   - "I'll use a set to track seen items, then loop through"
4. **Write code** (3 min)
   - Start simple, then optimize
5. **Walk through example** (1 min)
   - "So for input [1,2,1], we'd output [1,2]"
6. **Discuss complexity** (1 min)
   - "Time: O(n), Space: O(n)"

---

## 🎯 TOP 3 THINGS TO MEMORIZE

### 1. List Comprehension
```python
[x**2 for x in range(10) if x % 2 == 0]
```

### 2. Dictionary/Counter Operations
```python
from collections import Counter, defaultdict
freq = Counter(lst).most_common(3)
grouped = defaultdict(list)
```

### 3. Lambda + Sorted
```python
sorted(data, key=lambda x: x["age"], reverse=True)
```

These three patterns solve 70% of backend coding problems.
