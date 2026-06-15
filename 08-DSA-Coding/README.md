# 💻 DSA & Coding Interview Questions (JavaScript)

All solutions in simple, readable JavaScript with clear explanations.

---

## 🔢 Arrays

### 1. Two Sum
> Given an array of numbers and a target, return indices of two numbers that add up to target.

```javascript
// Brute Force: O(n²)
function twoSumBrute(nums, target) {
  for (let i = 0; i < nums.length; i++) {
    for (let j = i + 1; j < nums.length; j++) {
      if (nums[i] + nums[j] === target) return [i, j];
    }
  }
}

// Optimal: O(n) — HashMap trick
function twoSum(nums, target) {
  const seen = new Map(); // value → index

  for (let i = 0; i < nums.length; i++) {
    const complement = target - nums[i];

    if (seen.has(complement)) {
      return [seen.get(complement), i]; // found it!
    }

    seen.set(nums[i], i); // remember this number
  }
}

// twoSum([2, 7, 11, 15], 9) → [0, 1]  (2 + 7 = 9)
```

### 2. Maximum Subarray (Kadane's Algorithm)
> Find the contiguous subarray with the largest sum.

```javascript
function maxSubArray(nums) {
  let maxSum = nums[0];
  let currentSum = nums[0];

  for (let i = 1; i < nums.length; i++) {
    // Either extend current subarray OR start fresh
    currentSum = Math.max(nums[i], currentSum + nums[i]);
    maxSum = Math.max(maxSum, currentSum);
  }

  return maxSum;
}

// maxSubArray([-2, 1, -3, 4, -1, 2, 1, -5, 4]) → 6  ([4,-1,2,1])
```

### 3. Move Zeroes
> Move all 0s to the end while maintaining order of non-zero elements.

```javascript
function moveZeroes(nums) {
  let insertPos = 0;

  // Place all non-zeros first
  for (let i = 0; i < nums.length; i++) {
    if (nums[i] !== 0) {
      nums[insertPos] = nums[i];
      insertPos++;
    }
  }

  // Fill remaining with zeros
  while (insertPos < nums.length) {
    nums[insertPos] = 0;
    insertPos++;
  }

  return nums;
}

// moveZeroes([0,1,0,3,12]) → [1,3,12,0,0]
```

### 4. Best Time to Buy and Sell Stock
> Find the max profit from one buy/sell.

```javascript
function maxProfit(prices) {
  let minPrice = Infinity;
  let maxProfit = 0;

  for (const price of prices) {
    if (price < minPrice) {
      minPrice = price; // found a cheaper buying point
    } else if (price - minPrice > maxProfit) {
      maxProfit = price - minPrice; // found better profit
    }
  }

  return maxProfit;
}

// maxProfit([7,1,5,3,6,4]) → 5  (buy at 1, sell at 6)
```

---

## 🔤 Strings

### 5. Valid Anagram
> Check if two strings are anagrams of each other.

```javascript
function isAnagram(s, t) {
  if (s.length !== t.length) return false;

  const count = {};

  for (const char of s) count[char] = (count[char] || 0) + 1;
  for (const char of t) {
    if (!count[char]) return false;
    count[char]--;
  }

  return true;
}

// isAnagram("anagram", "nagaram") → true
// isAnagram("rat", "car") → false
```

### 6. Longest Substring Without Repeating Characters

```javascript
function lengthOfLongestSubstring(s) {
  const seen = new Map(); // char → last seen index
  let left = 0;
  let maxLen = 0;

  for (let right = 0; right < s.length; right++) {
    const char = s[right];

    // If we saw this char and it's in our current window
    if (seen.has(char) && seen.get(char) >= left) {
      left = seen.get(char) + 1; // shrink window from left
    }

    seen.set(char, right);
    maxLen = Math.max(maxLen, right - left + 1);
  }

  return maxLen;
}

// lengthOfLongestSubstring("abcabcbb") → 3 ("abc")
```

### 7. Palindrome Check

```javascript
// Simple version
function isPalindrome(s) {
  const clean = s.toLowerCase().replace(/[^a-z0-9]/g, '');
  return clean === clean.split('').reverse().join('');
}

// Two pointer (no extra space)
function isPalindromeOptimal(s) {
  let left = 0, right = s.length - 1;
  while (left < right) {
    if (s[left] !== s[right]) return false;
    left++;
    right--;
  }
  return true;
}
```

### 8. Group Anagrams

```javascript
function groupAnagrams(strs) {
  const map = new Map();

  for (const str of strs) {
    const key = str.split('').sort().join(''); // sorted string = unique key for anagram group
    if (!map.has(key)) map.set(key, []);
    map.get(key).push(str);
  }

  return [...map.values()];
}

// groupAnagrams(["eat","tea","tan","ate","nat","bat"])
// → [["eat","tea","ate"], ["tan","nat"], ["bat"]]
```

---

## 🌲 Trees

### 9. Maximum Depth of Binary Tree

```javascript
// Recursive DFS
function maxDepth(root) {
  if (!root) return 0;
  return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}

// Iterative BFS
function maxDepthBFS(root) {
  if (!root) return 0;
  const queue = [root];
  let depth = 0;

  while (queue.length > 0) {
    depth++;
    const levelSize = queue.length;

    for (let i = 0; i < levelSize; i++) {
      const node = queue.shift();
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
  }

  return depth;
}
```

### 10. Invert Binary Tree

```javascript
function invertTree(root) {
  if (!root) return null;

  // Swap left and right
  [root.left, root.right] = [root.right, root.left];

  // Recursively invert both subtrees
  invertTree(root.left);
  invertTree(root.right);

  return root;
}
```

### 11. Validate Binary Search Tree

```javascript
// A BST: left < node < right (for ALL nodes, not just immediate children)
function isValidBST(root, min = -Infinity, max = Infinity) {
  if (!root) return true;

  if (root.val <= min || root.val >= max) return false;

  return isValidBST(root.left, min, root.val) &&
         isValidBST(root.right, root.val, max);
}
```

---

## 🔗 Linked Lists

### 12. Reverse a Linked List

```javascript
function reverseList(head) {
  let prev = null;
  let curr = head;

  while (curr) {
    const next = curr.next; // save next before overwriting
    curr.next = prev;       // reverse the link
    prev = curr;            // move prev forward
    curr = next;            // move curr forward
  }

  return prev; // new head
}
```

### 13. Detect Cycle (Floyd's Algorithm)

```javascript
function hasCycle(head) {
  let slow = head;
  let fast = head;

  while (fast && fast.next) {
    slow = slow.next;       // moves 1 step
    fast = fast.next.next;  // moves 2 steps

    if (slow === fast) return true; // they meet → cycle!
  }

  return false;
}
```

---

## 📊 Sorting & Searching

### 14. Binary Search

```javascript
function binarySearch(nums, target) {
  let left = 0;
  let right = nums.length - 1;

  while (left <= right) {
    const mid = Math.floor((left + right) / 2);

    if (nums[mid] === target) return mid;
    else if (nums[mid] < target) left = mid + 1;
    else right = mid - 1;
  }

  return -1; // not found
}

// O(log n) — cuts search space in half each time
```

### 15. Merge Sort

```javascript
function mergeSort(arr) {
  if (arr.length <= 1) return arr;

  const mid = Math.floor(arr.length / 2);
  const left = mergeSort(arr.slice(0, mid));
  const right = mergeSort(arr.slice(mid));

  return merge(left, right);
}

function merge(left, right) {
  const result = [];
  let i = 0, j = 0;

  while (i < left.length && j < right.length) {
    if (left[i] <= right[j]) result.push(left[i++]);
    else result.push(right[j++]);
  }

  return [...result, ...left.slice(i), ...right.slice(j)];
}

// O(n log n) time, O(n) space
```

---

## 🧮 Dynamic Programming

### 16. Fibonacci with Memoization

```javascript
// Naive recursive: O(2^n)
function fibNaive(n) {
  if (n <= 1) return n;
  return fibNaive(n - 1) + fibNaive(n - 2);
}

// Memoized: O(n)
function fib(n, memo = {}) {
  if (n in memo) return memo[n];
  if (n <= 1) return n;
  memo[n] = fib(n - 1, memo) + fib(n - 2, memo);
  return memo[n];
}

// Bottom-up DP: O(n) time, O(1) space
function fibOptimal(n) {
  if (n <= 1) return n;
  let prev = 0, curr = 1;
  for (let i = 2; i <= n; i++) {
    [prev, curr] = [curr, prev + curr];
  }
  return curr;
}
```

### 17. Coin Change

```javascript
// Given coins and amount, find minimum number of coins
function coinChange(coins, amount) {
  const dp = new Array(amount + 1).fill(Infinity);
  dp[0] = 0; // 0 coins needed to make amount 0

  for (let i = 1; i <= amount; i++) {
    for (const coin of coins) {
      if (coin <= i) {
        dp[i] = Math.min(dp[i], dp[i - coin] + 1);
      }
    }
  }

  return dp[amount] === Infinity ? -1 : dp[amount];
}

// coinChange([1, 5, 6, 9], 11) → 2  (5+6)
```

---

## 🗂️ Hash Maps / Sets

### 18. Find Duplicates

```javascript
function findDuplicates(arr) {
  const seen = new Set();
  const duplicates = [];

  for (const item of arr) {
    if (seen.has(item)) duplicates.push(item);
    else seen.add(item);
  }

  return duplicates;
}
```

### 19. First Non-Repeating Character

```javascript
function firstUniqChar(s) {
  const count = {};
  for (const char of s) count[char] = (count[char] || 0) + 1;
  for (let i = 0; i < s.length; i++) {
    if (count[s[i]] === 1) return i;
  }
  return -1;
}
```

---

## 🔑 Big O Quick Reference

| Operation | Array | Hash Map | BST |
|-----------|-------|----------|-----|
| Access | O(1) | O(1) | O(log n) |
| Search | O(n) | O(1) | O(log n) |
| Insert | O(n) | O(1) | O(log n) |
| Delete | O(n) | O(1) | O(log n) |

| Algorithm | Time | Space |
|-----------|------|-------|
| Binary Search | O(log n) | O(1) |
| Merge Sort | O(n log n) | O(n) |
| Quick Sort | O(n log n) avg | O(log n) |
| Bubble Sort | O(n²) | O(1) |
| BFS / DFS | O(V + E) | O(V) |
