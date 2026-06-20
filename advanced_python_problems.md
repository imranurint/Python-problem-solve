# Advanced Python Coding Problems for Backend Engineers
## Intermediate to Hard (Hiring Board / Interview Level)

---

## PROBLEM 11: Longest Substring Without Repeating Characters
**Why:** Useful for duplicate detection, validation

**Scenario:** A user enters a password. Find the longest substring without repeating characters (for complexity analysis).

**Problem:**
```python
s = "abcabcbb"
# Output: 3  (substring "abc")

s = "bbbbb"
# Output: 1  (substring "b")

s = "pwwkew"
# Output: 3  (substring "wke")
```

**WRONG APPROACH (Brute Force - O(n³)):**
```python
def longest_substring(s):
    max_len = 0
    for i in range(len(s)):
        for j in range(i+1, len(s)+1):
            substring = s[i:j]
            if len(substring) == len(set(substring)):  # No duplicates
                max_len = max(max_len, len(substring))
    return max_len
```
⚠️ **Problem:** For a 1000-char string, this is slow.

**CORRECT APPROACH (Sliding Window - O(n)):**
```python
def longest_substring(s):
    char_index = {}  # Track last seen position of each character
    max_len = 0
    start = 0  # Window start
    
    for end in range(len(s)):
        char = s[end]
        
        # If char seen before and is in current window
        if char in char_index and char_index[char] >= start:
            start = char_index[char] + 1  # Move window start
        
        # Update last seen position
        char_index[char] = end
        
        # Update max length
        max_len = max(max_len, end - start + 1)
    
    return max_len

# Test
print(longest_substring("abcabcbb"))  # 3
print(longest_substring("bbbbb"))      # 1
print(longest_substring("pwwkew"))     # 3
```

**How it works (for "abcabcbb"):**
```
Index:  0 1 2 3 4 5 6 7
Char:   a b c a b c b b

Step 1: a → max_len=1, window="a"
Step 2: b → max_len=2, window="ab"
Step 3: c → max_len=3, window="abc"
Step 4: a → 'a' seen before, move start → window="bca"
Step 5: b → 'b' seen before, move start → window="cab"
Step 6: c → 'c' seen before, move start → window="abc"
Step 7: b → 'b' seen before, move start → window="cb"
Step 8: b → 'b' seen before, move start → window="b"

Max length ever reached: 3
```

**Complexity:**
- Time: O(n) — single pass
- Space: O(min(n, charset)) — dictionary stores at most 26 chars (for English)

**Interview talking point:**
"Brute force is O(n³) but slow. Sliding window optimizes to O(n) by tracking character positions. For email validation (no duplicate chars), this is the pattern."

---

## PROBLEM 12: Most Frequent K Elements
**Why:** Top N searches, trending products

**Scenario:** Return the K most frequently searched terms in your CMS.

**Problem:**
```python
searches = ["university", "course", "university", "admission", "course", "university"]
k = 2
# Output: ["university", "course"]  (in descending frequency)

# OR as dict:
# Output: {"university": 3, "course": 2}
```

**BRUTE FORCE (O(n log n)):**
```python
from collections import Counter

def top_k_frequent(words, k):
    freq = Counter(words)
    # Sort by frequency (descending)
    sorted_words = sorted(freq.items(), key=lambda x: x[1], reverse=True)
    return [word for word, count in sorted_words[:k]]

print(top_k_frequent(searches, 2))  # ['university', 'course']
```

**OPTIMIZED (O(n + k log n) with heap):**
```python
from collections import Counter
import heapq

def top_k_frequent(words, k):
    freq = Counter(words)
    
    # Keep only top k using heap (min-heap of size k)
    # heapq.nlargest returns k largest elements
    return heapq.nlargest(k, freq, key=freq.get)

print(top_k_frequent(searches, 2))  # ['university', 'course']
```

**OR with explicit heap (if you want to show knowledge):**
```python
from collections import Counter
import heapq

def top_k_frequent(words, k):
    freq = Counter(words)
    
    # Min-heap approach (space-efficient for large data, small k)
    heap = []
    for word, count in freq.items():
        if len(heap) < k:
            heapq.heappush(heap, (count, word))
        elif count > heap[0][0]:
            heapq.heapreplace(heap, (count, word))
    
    # Extract from heap (in reverse order)
    return [word for count, word in sorted(heap, reverse=True)]

print(top_k_frequent(searches, 2))  # ['course', 'university'] (order may vary)
```

**Complexity:**
- Time: O(n) for Counter + O(k log n) for heap = O(n + k log n)
- Space: O(n) for Counter

**Interview talking point:**
"If k is small and n is huge (1M searches), heap approach saves memory. But usually, Counter + sorted is clearest. For production (10M searches), I'd do this in SQL with GROUP BY and LIMIT k."

---

## PROBLEM 13: LRU Cache
**Why:** Caching frequently accessed data (Redis behavior)

**Scenario:** Cache the last N user profiles. When cache fills up, remove least recently used.

**Problem:**
```python
cache = LRUCache(capacity=2)
cache.put(1, "user_A")
cache.put(2, "user_B")
print(cache.get(1))           # "user_A" (marks 1 as recently used)
cache.put(3, "user_C")        # Evicts 2 (least recently used)
print(cache.get(2))           # None (was evicted)
```

**SOLUTION (Using OrderedDict):**
```python
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity):
        self.cache = OrderedDict()
        self.capacity = capacity
    
    def get(self, key):
        if key not in self.cache:
            return None
        # Move to end (mark as recently used)
        self.cache.move_to_end(key)
        return self.cache[key]
    
    def put(self, key, value):
        if key in self.cache:
            # Update and move to end
            self.cache[key] = value
            self.cache.move_to_end(key)
        else:
            # Add new entry
            self.cache[key] = value
            if len(self.cache) > self.capacity:
                # Remove first (least recently used)
                self.cache.popitem(last=False)

# Test
cache = LRUCache(2)
cache.put(1, "user_A")
cache.put(2, "user_B")
print(cache.get(1))      # "user_A"
cache.put(3, "user_C")   # Evicts 2
print(cache.get(2))      # None
```

**HOW IT WORKS:**
```
Initial: cache = {}

put(1, "A"): cache = {1: "A"}
put(2, "B"): cache = {1: "A", 2: "B"}
get(1):      cache = {2: "B", 1: "A"}  (1 moved to end)
put(3, "C"): capacity exceeded!
             Remove first (2): {1: "A", 3: "C"}
get(2):      None (was removed)
```

**Complexity:**
- Time: O(1) for all operations (get, put)
- Space: O(capacity)

**Interview talking point:**
"OrderedDict maintains insertion order. move_to_end() is O(1). This is how Redis LRU eviction works. For production, use actual Redis instead of implementing."

---

## PROBLEM 14: Valid Parentheses / Bracket Matching
**Why:** JSON validation, expression parsing

**Scenario:** Check if a JSON structure has balanced brackets.

**Problem:**
```python
s = "({[]})"
# Output: True (valid)

s = "({[}])"
# Output: False (invalid order)

s = "({[})"
# Output: False (mismatched)
```

**SOLUTION (Using Stack):**
```python
def is_valid(s):
    stack = []
    pairs = {')': '(', '}': '{', ']': '['}
    
    for char in s:
        if char in pairs:  # Closing bracket
            if not stack or stack[-1] != pairs[char]:
                return False
            stack.pop()
        else:  # Opening bracket
            stack.append(char)
    
    return len(stack) == 0  # All opened must be closed

# Test
print(is_valid("({[]})"))    # True
print(is_valid("({[}])"))    # False
print(is_valid("({[})"))     # False
```

**HOW IT WORKS:**
```
String: "({[]})"

char='(': stack=['(']
char='{': stack=['(', '{']
char='[': stack=['(', '{', '[']
char=']': pairs[']']='[', stack[-1]='[' ✓, pop → stack=['(', '{']
char='}': pairs['}]='{', stack[-1]='{' ✓, pop → stack=['(']
char=')': pairs[')']='(', stack[-1]='(' ✓, pop → stack=[]

Result: stack is empty → True
```

**Complexity:**
- Time: O(n)
- Space: O(n) for stack

**Interview talking point:**
"Stack is the go-to pattern for bracket matching. Same approach for expression evaluation, parsing HTML tags, etc."

---

## PROBLEM 15: Merge K Sorted Lists
**Why:** Merging data from multiple sources, pagination across databases

**Scenario:** Merge results from K database shards (each returns sorted user IDs).

**Problem:**
```python
lists = [
    [1, 4, 5],
    [1, 3, 4],
    [2, 6]
]
# Output: [1, 1, 2, 3, 4, 4, 5, 6]
```

**SOLUTION 1 (Simple - O(n log n)):**
```python
def merge_k_lists(lists):
    all_values = []
    for lst in lists:
        all_values.extend(lst)
    return sorted(all_values)

print(merge_k_lists(lists))  # [1, 1, 2, 3, 4, 4, 5, 6]
```
⚠️ Problem: Not leveraging that inputs are already sorted.

**SOLUTION 2 (Optimal - Using Heap - O(n log k)):**
```python
import heapq

def merge_k_lists(lists):
    # Min-heap: (value, list_index, element_index)
    heap = []
    
    # Add first element of each list
    for i, lst in enumerate(lists):
        if lst:
            heapq.heappush(heap, (lst[0], i, 0))
    
    result = []
    
    while heap:
        val, list_idx, elem_idx = heapq.heappop(heap)
        result.append(val)
        
        # If list has more elements, add next one
        if elem_idx + 1 < len(lists[list_idx]):
            next_val = lists[list_idx][elem_idx + 1]
            heapq.heappush(heap, (next_val, list_idx, elem_idx + 1))
    
    return result

print(merge_k_lists(lists))  # [1, 1, 2, 3, 4, 4, 5, 6]
```

**HOW IT WORKS:**
```
lists = [[1,4,5], [1,3,4], [2,6]]

Initial heap: [(1, 0, 0), (1, 1, 0), (2, 2, 0)]

Pop (1, 0, 0) → result=[1], push (4, 0, 1)
Pop (1, 1, 0) → result=[1,1], push (3, 1, 1)
Pop (2, 2, 0) → result=[1,1,2], push (6, 2, 1)
Pop (3, 1, 1) → result=[1,1,2,3], push (4, 1, 2)
Pop (4, 0, 1) → result=[1,1,2,3,4], push (5, 0, 2)
Pop (4, 1, 2) → result=[1,1,2,3,4,4]
Pop (5, 0, 2) → result=[1,1,2,3,4,4,5]
Pop (6, 2, 1) → result=[1,1,2,3,4,4,5,6]

Heap empty → return result
```

**Complexity:**
- Time: O(n log k) where n = total elements, k = number of lists
- Space: O(k) for heap

**Interview talking point:**
"Merging K sorted lists is classic. Heap approach is O(n log k) vs naive O(n log n). For 1M elements across 1000 shards, heap saves time."

---

## PROBLEM 16: Word Ladder (BFS)
**Why:** Shortest path in graphs, URL routing, friend recommendations

**Scenario:** Find shortest path from one word to another by changing one letter at a time.

**Problem:**
```python
start = "hit"
end = "cog"
words = ["hot", "dot", "dog", "lot", "log", "cog"]

# Path: hit → hot → dot → dog → cog
# Output: 5 (number of words in path)
```

**SOLUTION (BFS):**
```python
from collections import deque, defaultdict

def word_ladder(start, end, words):
    if end not in words:
        return 0
    
    words_set = set(words)
    queue = deque([(start, 1)])  # (current_word, distance)
    visited = {start}
    
    while queue:
        current, distance = queue.popleft()
        
        if current == end:
            return distance
        
        # Try changing each character
        for i in range(len(current)):
            for c in 'abcdefghijklmnopqrstuvwxyz':
                if c != current[i]:
                    neighbor = current[:i] + c + current[i+1:]
                    if neighbor in words_set and neighbor not in visited:
                        visited.add(neighbor)
                        queue.append((neighbor, distance + 1))
    
    return 0  # No path found

# Test
words = ["hot", "dot", "dog", "lot", "log", "cog"]
print(word_ladder("hit", "cog", words))  # 5
```

**HOW IT WORKS:**
```
Start: hit, End: cog

Queue: [(hit, 1)]
  hit → neighbors: hot (in list)
  Queue: [(hot, 2)]

Queue: [(hot, 2)]
  hot → neighbors: dot, lot (in list)
  Queue: [(dot, 3), (lot, 3)]

Queue: [(dot, 3), (lot, 3)]
  dot → neighbors: dog, log (in list)
  Queue: [(lot, 3), (dog, 4), (log, 4)]

... continue until we find "cog"

Result: hit → hot → dot → dog → cog = 5 words
```

**Complexity:**
- Time: O(n × l × 26) where n = words, l = word length
- Space: O(n) for visited set

**Interview talking point:**
"BFS finds shortest path in unweighted graphs. Perfect for recommendation systems (find friend within N hops). For weighted graphs, use Dijkstra."

---

## PROBLEM 17: Longest Increasing Subsequence (LIS)
**Why:** Stock trading, analytics trends

**Scenario:** Find longest trend of increasing values in time-series data.

**Problem:**
```python
nums = [10, 9, 2, 5, 3, 7, 101, 18]
# Output: 4  (subsequence: [2, 3, 7, 101])

nums = [0, 1, 0, 4, 4, 4, 3, 5, 1]
# Output: 4  (subsequence: [0, 1, 4, 5] or [0, 1, 3, 5])
```

**SOLUTION 1 (Dynamic Programming - O(n²)):**
```python
def lis_length(nums):
    if not nums:
        return 0
    
    n = len(nums)
    dp = [1] * n  # dp[i] = length of LIS ending at index i
    
    for i in range(1, n):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    
    return max(dp)

print(lis_length([10, 9, 2, 5, 3, 7, 101, 18]))  # 4
```

**HOW IT WORKS:**
```
nums = [10, 9, 2, 5, 3, 7, 101, 18]
dp =   [1,  1, 1, 2, 2, 3,  4,   4]

i=0: dp[0]=1 (just [10])
i=1: 9<10? No. dp[1]=1 (just [9])
i=2: 2<10? Yes. dp[2]=max(1, 1+1)=1 (2 not > any prev)
i=3: 5>2? Yes. dp[3]=max(1, dp[2]+1)=2 ([2,5])
i=4: 3>2? Yes. dp[4]=max(1, dp[2]+1)=2 ([2,3])
i=5: 7>10,9,2,5,3? 7>5? Yes. dp[5]=max(1, dp[3]+1)=3 ([2,5,7])
i=6: 101>all? Yes. dp[6]=max(1, dp[5]+1)=4 ([2,5,7,101])
i=7: 18>10,9,2,5,3,7,101? 18>7? Yes. dp[7]=max(1, dp[5]+1)=4

Max: 4
```

**SOLUTION 2 (Optimized with Binary Search - O(n log n)):**
```python
import bisect

def lis_length(nums):
    if not nums:
        return 0
    
    tails = []  # tails[i] = smallest tail of increasing subseq of length i+1
    
    for num in nums:
        pos = bisect.bisect_left(tails, num)
        if pos == len(tails):
            tails.append(num)
        else:
            tails[pos] = num
    
    return len(tails)

print(lis_length([10, 9, 2, 5, 3, 7, 101, 18]))  # 4
```

**Complexity:**
- DP: O(n²) time, O(n) space
- Binary search: O(n log n) time, O(n) space

**Interview talking point:**
"For small arrays (n < 1000), O(n²) is fine. For large datasets (1M elements), binary search optimization is needed. This is the pattern for stock trading maximum profit problems."

---

## PROBLEM 18: Permutations / Combinations
**Why:** Generate test cases, recommendations, path finding

**Scenario:** Generate all possible password combinations (for testing).

**Problem:**
```python
nums = [1, 2, 3]
# Output: [[1,2,3], [1,3,2], [2,1,3], [2,3,1], [3,1,2], [3,2,1]]

# For combinations (subsets):
nums = [1, 2, 3]
# Output: [[], [1], [2], [3], [1,2], [1,3], [2,3], [1,2,3]]
```

**SOLUTION 1 (Permutations - Recursion/Backtracking):**
```python
def permutations(nums):
    result = []
    
    def backtrack(path, remaining):
        if not remaining:
            result.append(path[:])  # Copy of path
            return
        
        for i in range(len(remaining)):
            path.append(remaining[i])
            backtrack(path, remaining[:i] + remaining[i+1:])
            path.pop()
    
    backtrack([], nums)
    return result

print(permutations([1, 2, 3]))
# [[1,2,3], [1,3,2], [2,1,3], [2,3,1], [3,1,2], [3,2,1]]
```

**SOLUTION 2 (Using itertools - Simpler):**
```python
from itertools import permutations, combinations

# Permutations
nums = [1, 2, 3]
perms = list(permutations(nums))
print(perms)  # All orderings

# Combinations
nums = [1, 2, 3]
combs = list(combinations(nums, 2))  # Choose 2
print(combs)  # [(1, 2), (1, 3), (2, 3)]
```

**Complexity:**
- Permutations: O(n!) time (n! results)
- Space: O(n) for recursion depth

**Interview talking point:**
"For small n (n < 10), recursion is fine. For larger n, it's combinatorial explosion. Always ask if you really need all permutations or if there's a smarter approach."

---

## PROBLEM 19: Two Pointers / Three Sum
**Why:** Finding pairs/triplets with specific properties

**Scenario:** Find all unique triplets in array that sum to a target.

**Problem:**
```python
nums = [-1, 0, 1, 2, -1, -4]
target = 0
# Output: [[-1, -1, 2], [-1, 0, 1]]  (unique triplets summing to 0)
```

**SOLUTION (Two Pointers):**
```python
def three_sum(nums, target=0):
    nums.sort()  # Sort first
    result = []
    n = len(nums)
    
    for i in range(n - 2):
        # Skip duplicates
        if i > 0 and nums[i] == nums[i - 1]:
            continue
        
        # Two pointer approach
        left, right = i + 1, n - 1
        
        while left < right:
            current_sum = nums[i] + nums[left] + nums[right]
            
            if current_sum == target:
                result.append([nums[i], nums[left], nums[right]])
                
                # Skip duplicates
                while left < right and nums[left] == nums[left + 1]:
                    left += 1
                while left < right and nums[right] == nums[right - 1]:
                    right -= 1
                
                left += 1
                right -= 1
            elif current_sum < target:
                left += 1
            else:
                right -= 1
    
    return result

print(three_sum([-1, 0, 1, 2, -1, -4]))  # [[-1, -1, 2], [-1, 0, 1]]
```

**HOW IT WORKS:**
```
nums = [-1, 0, 1, 2, -1, -4]
After sorting: [-4, -1, -1, 0, 1, 2]

i=0, nums[i]=-4:
  left=1, right=5: -4 + -1 + 2 = -3 < 0, left++
  left=2, right=5: -4 + -1 + 2 = -3 < 0, left++
  left=3, right=5: -4 + 0 + 2 = -2 < 0, left++
  left=4, right=5: -4 + 1 + 2 = -1 < 0, left++
  left=5, right=5: exit

i=1, nums[i]=-1:
  left=2, right=5: -1 + -1 + 2 = 0 ✓ → add [-1, -1, 2]
  left=3, right=5: -1 + 0 + 2 = 1 > 0, right--
  left=3, right=4: -1 + 0 + 1 = 0 ✓ → add [-1, 0, 1]

Result: [[-1, -1, 2], [-1, 0, 1]]
```

**Complexity:**
- Time: O(n²) — outer loop O(n) + inner two pointers O(n)
- Space: O(1) excluding result

**Interview talking point:**
"Sorting enables two-pointer optimization. This is the classic pattern for finding pairs/triplets with constraints. For 4-sum, 5-sum, use recursive approach with same idea."

---

## PROBLEM 20: Trie (Prefix Tree)
**Why:** Autocomplete, spell checking, IP routing

**Scenario:** Build an autocomplete system for university names.

**Problem:**
```python
trie = Trie()
trie.insert("apple")
trie.insert("app")

print(trie.search("app"))       # True
print(trie.search("apple"))     # True
print(trie.search("appl"))      # False
print(trie.starts_with("app"))  # True (prefix exists)
```

**SOLUTION:**
```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False

class Trie:
    def __init__(self):
        self.root = TrieNode()
    
    def insert(self, word):
        node = self.root
        for char in word:
            if char not in node.children:
                node.children[char] = TrieNode()
            node = node.children[char]
        node.is_end = True
    
    def search(self, word):
        node = self._find_node(word)
        return node is not None and node.is_end
    
    def starts_with(self, prefix):
        return self._find_node(prefix) is not None
    
    def _find_node(self, word):
        node = self.root
        for char in word:
            if char not in node.children:
                return None
            node = node.children[char]
        return node
    
    def autocomplete(self, prefix):
        """Find all words starting with prefix"""
        node = self._find_node(prefix)
        if not node:
            return []
        
        results = []
        self._dfs(node, prefix, results)
        return results
    
    def _dfs(self, node, current, results):
        if node.is_end:
            results.append(current)
        
        for char, child_node in node.children.items():
            self._dfs(child_node, current + char, results)

# Test
trie = Trie()
trie.insert("apple")
trie.insert("app")
trie.insert("apricot")

print(trie.search("app"))         # True
print(trie.starts_with("ap"))     # True
print(trie.autocomplete("ap"))    # ['app', 'apple', 'apricot']
```

**HOW IT WORKS:**
```
Trie structure after inserting "apple", "app", "apricot":

         root
          |
          a
          |
          p
         / \
        p   r
        |   |
        l   i
        |   |
        e   c
            |
            o
            |
            t

search("app"): root→a→p→p (is_end=True) → True
search("appl"): root→a→p→p→l (is_end=False) → False
starts_with("ap"): root→a→p (exists) → True
autocomplete("ap"): Find all words from "ap" node → [app, apple, apricot]
```

**Complexity:**
- Insert: O(m) where m = word length
- Search: O(m)
- Autocomplete: O(n + m) where n = number of results

**Interview talking point:**
"Trie is perfect for autocomplete. For 100k university names, Trie is faster than prefix string matching. Each node is O(26) space for English letters."

---

## PROBLEM 21: Add Two Numbers (Linked Lists)
**Why:** Real-world data structures, payment processing

**Scenario:** Add two large numbers represented as linked lists (each node = one digit).

**Problem:**
```python
# List 1: 2 → 4 → 3 (represents 342)
# List 2: 5 → 6 → 4 (represents 465)
# Output: 7 → 0 → 8 (represents 807, since 342 + 465 = 807)
```

**SOLUTION:**
```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

def add_two_numbers(l1, l2):
    dummy = ListNode(0)  # Dummy node to simplify logic
    current = dummy
    carry = 0
    
    while l1 or l2 or carry:
        val1 = l1.val if l1 else 0
        val2 = l2.val if l2 else 0
        
        total = val1 + val2 + carry
        carry = total // 10
        digit = total % 10
        
        current.next = ListNode(digit)
        current = current.next
        
        l1 = l1.next if l1 else None
        l2 = l2.next if l2 else None
    
    return dummy.next

# Create linked lists
def create_list(digits):
    head = ListNode(digits[0])
    current = head
    for d in digits[1:]:
        current.next = ListNode(d)
        current = current.next
    return head

def print_list(head):
    result = []
    while head:
        result.append(head.val)
        head = head.next
    print(result)

# Test
l1 = create_list([2, 4, 3])  # 342
l2 = create_list([5, 6, 4])  # 465
result = add_two_numbers(l1, l2)
print_list(result)  # [7, 0, 8] representing 807
```

**HOW IT WORKS:**
```
l1:  2 → 4 → 3
l2:  5 → 6 → 4
     -----------
     7 → 0 → 8

Step 1: 2+5=7, carry=0, add 7
Step 2: 4+6+0=10, carry=1, add 0
Step 3: 3+4+1=8, carry=0, add 8
```

**Complexity:**
- Time: O(max(len(l1), len(l2)))
- Space: O(max(len(l1), len(l2))) for result list

**Interview talking point:**
"Linked lists are common in backend (payment systems, queues). Dummy node is a trick to avoid special-case handling for the head."

---

## PROBLEM 22: Find Duplicate in Array (Floyd's Cycle Detection)
**Why:** Memory leak detection, cycle detection

**Scenario:** Array has n+1 numbers from 1 to n. Find the duplicate (appears at least twice).

**Problem:**
```python
nums = [1, 3, 4, 2, 2]
# Output: 2

nums = [3, 1, 3, 4, 2]
# Output: 3

# Constraint: O(1) space, O(n) time, don't modify array
```

**WRONG APPROACH (Uses O(n) space):**
```python
def find_duplicate(nums):
    seen = set()
    for num in nums:
        if num in seen:
            return num
        seen.add(num)
```

**CORRECT APPROACH (Floyd's Cycle Detection - O(1) space):**
```python
def find_duplicate(nums):
    # Treat array as linked list
    # nums[i] points to index nums[i]
    
    # Phase 1: Find cycle
    slow = fast = nums[0]
    while True:
        slow = nums[slow]
        fast = nums[nums[fast]]
        if slow == fast:
            break
    
    # Phase 2: Find cycle start (the duplicate)
    slow = nums[0]
    while slow != fast:
        slow = nums[slow]
        fast = nums[fast]
    
    return slow

print(find_duplicate([1, 3, 4, 2, 2]))  # 2
```

**HOW IT WORKS (Cycle Detection):**
```
nums = [1, 3, 4, 2, 2]
Interpret as linked list:
  0 → 1 → 3 → 2 → 4 → 2 → 4 → ... (cycle at 2)

Phase 1: Find cycle
  slow: 1 → 3 → 2 → 4 → 2
  fast: 3 → 4 → 2 → 3 → 2
  They meet at 2

Phase 2: Find cycle start
  Reset slow to start: 0
  Move both one step until they meet:
  slow: 0 → 1 → 3 → 2
  fast: 2 → 4 → 2
  They meet at 2 (the duplicate!)
```

**Complexity:**
- Time: O(n)
- Space: O(1)

**Interview talking point:**
"This is the classical Floyd's cycle detection algorithm. For a duplicate number problem with O(1) space constraint, this is the only efficient solution."

---

## PROBLEM 23: Median of Two Sorted Arrays
**Why:** Analytics, merging datasets

**Scenario:** Find median of merged user age data from two regions.

**Problem:**
```python
nums1 = [1, 3]
nums2 = [2]
# Merged: [1, 2, 3] → Median = 2

nums1 = [1, 2]
nums2 = [3, 4]
# Merged: [1, 2, 3, 4] → Median = (2 + 3) / 2 = 2.5
```

**SIMPLE SOLUTION (O(n + m) time):**
```python
def find_median(nums1, nums2):
    merged = []
    i = j = 0
    
    # Merge two sorted arrays
    while i < len(nums1) and j < len(nums2):
        if nums1[i] < nums2[j]:
            merged.append(nums1[i])
            i += 1
        else:
            merged.append(nums2[j])
            j += 1
    
    merged.extend(nums1[i:])
    merged.extend(nums2[j:])
    
    n = len(merged)
    if n % 2 == 1:
        return float(merged[n // 2])
    else:
        return (merged[n // 2 - 1] + merged[n // 2]) / 2

print(find_median([1, 3], [2]))        # 2.0
print(find_median([1, 2], [3, 4]))     # 2.5
```

**OPTIMIZED SOLUTION (O(log(min(m, n))) using Binary Search):**
```python
def find_median_bsearch(nums1, nums2):
    if len(nums1) > len(nums2):
        nums1, nums2 = nums2, nums1
    
    low, high = 0, len(nums1)
    total_len = len(nums1) + len(nums2)
    
    while low <= high:
        cut1 = (low + high) // 2
        cut2 = (total_len + 1) // 2 - cut1
        
        left1 = nums1[cut1 - 1] if cut1 > 0 else float('-inf')
        right1 = nums1[cut1] if cut1 < len(nums1) else float('inf')
        left2 = nums2[cut2 - 1] if cut2 > 0 else float('-inf')
        right2 = nums2[cut2] if cut2 < len(nums2) else float('inf')
        
        if left1 <= right2 and left2 <= right1:
            # Found correct partition
            if total_len % 2 == 0:
                return (max(left1, left2) + min(right1, right2)) / 2
            else:
                return float(max(left1, left2))
        elif left1 > right2:
            high = cut1 - 1
        else:
            low = cut1 + 1
    
    return -1

print(find_median_bsearch([1, 3], [2]))        # 2.0
```

**Complexity:**
- Simple: O(n + m) time, O(n + m) space
- Binary search: O(log(min(m, n))) time, O(1) space

**Interview talking point:**
"Simple merge is fine for most cases. But if asked for O(log n) complexity, use binary search. This is a hard problem — if you get it, you're golden."

---

## PROBLEM 24: Design a URL Shortener
**Why:** Real-world system design, encoding/decoding

**Scenario:** Create a URL shortener (like bit.ly). Encode long URL to short code, decode back.

**Problem:**
```python
codec = URLCodec()
url = "https://www.example.com/very/long/url"
short = codec.encode(url)  # "http://tinyurl.com/a1b2c"
long = codec.decode(short)  # "https://www.example.com/very/long/url"
```

**SOLUTION:**
```python
class URLCodec:
    def __init__(self):
        self.url_map = {}  # short_id → long_url
        self.id_map = {}   # long_url → short_id
        self.base_url = "http://tinyurl.com/"
        self.counter = 0
    
    def encode(self, long_url):
        if long_url in self.id_map:
            return self.base_url + self.id_map[long_url]
        
        # Generate short ID (base62 encoding)
        short_id = self._encode_id(self.counter)
        self.counter += 1
        
        self.url_map[short_id] = long_url
        self.id_map[long_url] = short_id
        
        return self.base_url + short_id
    
    def decode(self, short_url):
        short_id = short_url.split("/")[-1]
        return self.url_map.get(short_id, "")
    
    def _encode_id(self, num):
        """Convert number to base62 string"""
        chars = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
        result = ""
        while num > 0:
            result = chars[num % 62] + result
            num //= 62
        return result if result else "0"

# Test
codec = URLCodec()
url1 = "https://www.example.com/very/long/url"
short1 = codec.encode(url1)
print(f"Short: {short1}")
print(f"Decoded: {codec.decode(short1)}")

url2 = "https://github.com/imranurint"
short2 = codec.encode(url2)
print(f"Short: {short2}")
```

**HOW IT WORKS:**
```
URL: "https://www.example.com"
Counter: 0
encode_id(0) = "0" → short_url = "http://tinyurl.com/0"
url_map: {"0": "https://www.example.com"}

URL: "https://github.com"
Counter: 1
encode_id(1) = "1" → short_url = "http://tinyurl.com/1"
url_map: {"0": "...", "1": "https://github.com"}

... 
Counter: 62
encode_id(62) = "10" → short_url = "http://tinyurl.com/10"
```

**Complexity:**
- Time: O(1) for encode/decode
- Space: O(n) for storing mappings

**Interview talking point:**
"This is a system design problem in code. For production, you'd use a database (not in-memory dicts), add expiration, handle collisions, and scale across servers."

---

## PROBLEM 25: Find K Closest Elements to Target
**Why:** Recommendation systems, nearest neighbor search

**Scenario:** From a list of user ages, find K closest to a target age.

**Problem:**
```python
ages = [1, 2, 3, 4, 5]
target = 3
k = 2
# Output: [2, 3] or [3, 4]  (two closest to 3)

ages = [1, 1, 1, 10, 10, 10]
target = 9
k = 4
# Output: [10, 10, 10, 1]  (4 closest)
```

**SOLUTION 1 (Sort by distance - O(n log n)):**
```python
def find_closest(ages, target, k):
    # Sort by distance from target, then by value
    sorted_ages = sorted(ages, key=lambda x: (abs(x - target), x))
    return sorted_ages[:k]

print(find_closest([1, 2, 3, 4, 5], 3, 2))  # [2, 3]
```

**SOLUTION 2 (Optimized with Two Pointers - O(n)):**
```python
def find_closest(ages, target, k):
    # Already sorted, use two pointers
    left, right = 0, len(ages) - 1
    
    # Find the closest position to target
    while left < right:
        if abs(ages[left] - target) > abs(ages[right] - target):
            left += 1
        else:
            right -= 1
    
    # Expand window around right pointer
    result = []
    for _ in range(k):
        if left < 0:
            result.append(ages[right])
            right -= 1
        elif right >= len(ages):
            result.append(ages[left])
            left -= 1
        elif abs(ages[left] - target) <= abs(ages[right] - target):
            result.append(ages[left])
            left -= 1
        else:
            result.append(ages[right])
            right += 1
    
    return sorted(result)

print(find_closest([1, 2, 3, 4, 5], 3, 2))  # [2, 3]
```

**Complexity:**
- Simple sort: O(n log n)
- Two pointers: O(n)

**Interview talking point:**
"If input is sorted, two-pointer is more efficient than sorting again. This is the pattern for finding K closest elements in sorted arrays."

---

## 🎯 WHICH PROBLEMS TO FOCUS ON

**By Company:**

### Therap (US Healthcare SaaS)
- Problem 11: Longest Substring (for data validation)
- Problem 12: Top K Frequent (trending treatments)
- Problem 14: Bracket Matching (JSON parsing for HL7 healthcare data)
- Problem 19: Three Sum (finding patterns in patient data)

### Pathao (Logistics/Ride-hailing)
- Problem 16: Word Ladder / BFS (shortest route)
- Problem 17: LIS (predicting demand trends)
- Problem 15: Merge K Sorted Lists (aggregating data from multiple drivers)
- Problem 25: K Closest Elements (nearest driver to customer)

### Brain Station 23 (Enterprise)
- Problem 13: LRU Cache (API caching)
- Problem 20: Trie (autocomplete)
- Problem 21: Add Two Numbers (big number arithmetic)
- Problem 22: Find Duplicate (memory leak detection)

### General Backend
- Problem 14: Bracket Matching (API validation)
- Problem 20: Trie (search/autocomplete)
- Problem 24: URL Shortener (system design understanding)

---

## 📋 INTERVIEW PREP CHECKLIST

**Week 1: Basics**
- [ ] Problems 1-10 (basic patterns)
- [ ] Memorize: list comprehension, Counter, sorted with key

**Week 2: Intermediate**
- [ ] Problems 11-15 (sliding window, heap, LRU, stack)
- [ ] Focus: Problem 13 (LRU Cache), Problem 14 (Brackets)

**Week 3: Advanced**
- [ ] Problems 16-25 (BFS, DP, Trie, Linked Lists)
- [ ] Focus: Problem 20 (Trie), Problem 24 (URL Shortener)

**Day Before Interview**
- [ ] Review company-specific problems
- [ ] Practice solving 2 problems out loud
- [ ] Note time complexity for each

---

## 🚀 FINAL TIPS

**DO:**
- ✅ Explain approach before coding
- ✅ Discuss time/space complexity
- ✅ Handle edge cases (empty lists, None, etc.)
- ✅ Test with examples
- ✅ Ask for clarification

**DON'T:**
- ❌ Code immediately without thinking
- ❌ Ignore complexity analysis
- ❌ Hardcode values
- ❌ Forget edge cases
- ❌ Skip talking through your logic

---

**Good luck! These problems cover 90% of real interview questions at Dhaka tech companies.** 🎯
