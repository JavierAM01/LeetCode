### Longest Consecutive Sequence Problem

This markdown explains the progression from a naive `O(n^3)` solution to an optimized `O(n)` solution for the problem of finding the length of the longest consecutive elements sequence in an unsorted array of integers.

---

### **Problem Statement**

Given an unsorted array of integers `nums`, return the length of the longest consecutive elements sequence.

- **Constraints**:
  - Time complexity must be `O(n)`.

#### **Examples**

**Example 1**:
```plaintext
Input: nums = [100, 4, 200, 1, 3, 2]
Output: 4
Explanation: The longest consecutive elements sequence is [1, 2, 3, 4].
```

**Example 2**:
```plaintext
Input: nums = [0,3,7,2,5,8,4,6,0,1]
Output: 9
```

---

### **Initial Naive Solution (O(n^3))**

#### **Idea**
1. Iterate over each number `x` in `nums`.
2. For every `x`, count how many consecutive elements exist by checking if `x+1`, `x+2`, etc., are present in the array.
3. Keep track of the maximum sequence length.

#### **Code**
```cpp
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        int maxLength = 0;

        for (int num : nums) {
            int current = num;
            int length = 1;

            while (find(nums.begin(), nums.end(), current + 1) != nums.end()) {
                current += 1;
                length += 1;
            }

            maxLength = max(maxLength, length);
        }

        return maxLength;
    }
};
```

#### **Time Complexity**
- **Outer loop**: `O(n)` for iterating through `nums`.
- **Inner loop**: For each number, checking for the next element with `find` is `O(n)`.
- Overall complexity: **`O(n^3)`** (for worst-case scenarios).

---

### **Optimized Solution (O(n))**

#### **Key Observations**
- Instead of repeatedly searching for elements, use a data structure like a hash map to store information about sequences.
- Use a two-stage approach:
  1. Check if a number is new or already part of an existing sequence.
  2. Merge or extend sequences dynamically using the hash map.

#### **Core Concepts**
1. **Hash Map Structure**:
   - `data[n]`: Tracks the size of the sequence that includes `n`.
   - Special codes:
     - `-2`: Left endpoint of a sequence.
     - `-1`: Right endpoint of a sequence.
     - `0`: Joining point in the middle of two sequences.
2. **Logic**:
   - If `n` exists in `data`:
     - Update `left`, `right`, or `joining` points accordingly.
   - If `n` is new:
     - Add it to `data` as a single element sequence and update boundaries.

#### **Code**
```cpp
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        int n = nums.size();
        if (n < 2)
            return n;
        
        int maxN = 0;
        unordered_map<int, int> data;

        for (int n : nums) {
            // If the number already exists in data
            if (exists(data, n)) {
                if (data[n] == -2) {
                    int val = data[n+1];
                    data[n] = val + 1;
                    data[n+val] = val + 1;
                    maxN = max(maxN, val + 1);

                    if (exists(data, n-1))
                        data[n-1] = 0;
                    else
                        data[n-1] = -2;

                } else if (data[n] == -1) {
                    int val = data[n-1];
                    data[n] = val + 1;
                    data[n-val] = val + 1;
                    maxN = max(maxN, val + 1);

                    if (exists(data, n+1))
                        data[n+1] = 0;
                    else
                        data[n+1] = -1;

                } else if (data[n] == 0) {
                    int val_left = data[n-1], val_right = data[n+1];
                    int new_val = val_left + 1 + val_right;
                    maxN = max(maxN, new_val);

                    data[n - val_left] = new_val;
                    data[n + val_right] = new_val;
                }
            } else {
                data[n] = 1;
                data[n-1] = exists(data, n-1) ? 0 : -2;
                data[n+1] = exists(data, n+1) ? 0 : -1;
            }
        }
        return max(1, maxN);
    }

    bool exists(unordered_map<int, int> &data, int n) {
        return (data.find(n) != data.end());
    }
};
```

#### **Detailed Steps**
1. **Initialize Variables**:
   - `data`: An unordered map to store sequence details.
   - `maxN`: To track the longest sequence length.

2. **Iterate Through `nums`**:
   - **If `n` exists**:
     - Handle the `left`, `right`, or `joining` point cases.
   - **If `n` is new**:
     - Create a single-point sequence and set boundaries.

3. **Return Result**:
   - Return the maximum sequence length.

#### **Time Complexity**
- **Insertions and Lookups** in `unordered_map`: `O(1)`.
- Iterating through `nums`: `O(n)`.
- **Overall complexity**: **`O(n)`**.

---

### **Comparison Between Solutions**

| **Approach**      | **Time Complexity** | **Space Complexity** | **Scalability** |
|--------------------|---------------------|-----------------------|-----------------|
| Naive (`O(n^3)`)  | `O(n^3)`            | `O(1)`                | Poor            |
| Optimized (`O(n)`) | `O(n)`              | `O(n)`                | Excellent       |

---

### **Key Insights**
- Leveraging hash maps for efficient lookups and updates is crucial for reducing the time complexity.
- Dynamically updating boundaries ensures that sequences are efficiently merged without redundant computations.
