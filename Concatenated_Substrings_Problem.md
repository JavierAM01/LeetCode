### Concatenated Substrings Problem

This document explains how to solve the problem of finding all starting indices of concatenated substrings in a string `s` using an array of words `words`. We will explore the evolution of solutions from a naive approach with poor time complexity to an optimized solution with O(n) complexity.

---

### **Problem Statement**

You are given:
- A string `s`.
- An array of strings `words`, where all strings in `words` have the same length.

#### **Goal**
Find all starting indices of substrings in `s` that are concatenations of all strings in `words` in any order.

#### **Examples**

**Example 1**:
```plaintext
Input: s = "barfoothefoobarman", words = ["foo","bar"]
Output: [0,9]
Explanation: Substrings starting at indices 0 and 9 are "barfoo" and "foobar", respectively.
```

**Example 2**:
```plaintext
Input: s = "wordgoodgoodgoodbestword", words = ["word","good","best","word"]
Output: []
Explanation: No valid concatenation exists.
```

---

### **Initial Naive Solution (O(n^3))**

#### **Idea**
The naive solution involves brute-forcing through the string `s` and:
1. Checking every substring of length `words.size() * word_length`.
2. Verifying if it contains all strings in `words` in any permutation.

#### **Steps**
1. Iterate over all starting indices `i` in `s`.
2. Extract a substring of size `total_length = len(words) * len(words[0])` starting from `i`.
3. Check if this substring is a valid concatenation by comparing permutations.

#### **Code**
```cpp
#include <algorithm>
#include <string>
#include <vector>
using namespace std;

class Solution {
public:
    vector<int> findSubstring(string s, vector<string>& words) {
        vector<int> result;
        int wordLen = words[0].size();
        int totalLen = wordLen * words.size();

        for (int i = 0; i <= (int)s.size() - totalLen; i++) {
            string sub = s.substr(i, totalLen);
            vector<string> tempWords = words;
            
            // Check if sub is a valid concatenation
            for (int j = 0; j < totalLen; j += wordLen) {
                string word = sub.substr(j, wordLen);
                auto it = find(tempWords.begin(), tempWords.end(), word);
                if (it == tempWords.end()) break;
                tempWords.erase(it);
            }
            if (tempWords.empty()) result.push_back(i);
        }
        return result;
    }
};
```

#### **Complexity Analysis**
- **Outer Loop**: `O(n)` for iterating over `s`.
- **Inner Loop**: `O(k)` for checking the substring with all `words`.
- **Finding Word in Words**: `O(k)` for `std::find`.

**Overall Complexity**: **`O(n * k^2)`**, where `n` is the length of `s` and `k` is the number of words.

#### **Why It Is Inefficient**
1. Checking permutations repeatedly is wasteful.
2. Using `std::find` in the inner loop introduces unnecessary overhead.

---

### **Optimized Solution (O(n))**

#### **Key Observations**
1. Every valid substring is exactly `total_length = words.size() * word_length`.
2. Instead of checking permutations, count the frequency of words in `words` and compare it with the frequency of words in the substring.
3. Use a sliding window to efficiently manage substrings of fixed length.

#### **Core Concepts**
- **Hash Map for Word Counts**:
  - Use an unordered map to store the frequency of each word in `words`.
- **Sliding Window**:
  - Move a fixed-size window across `s` and maintain the count of words inside the window.
  - Compare this count with the original word frequency map.

#### **Code**
```cpp
#include <unordered_map>
#include <vector>
#include <string>
using namespace std;

class Solution {
public:
    vector<int> findSubstring(string s, vector<string>& words) {
        vector<int> result;
        int wordLen = words[0].size();
        int totalLen = wordLen * words.size();

        unordered_map<string, int> wordCount;
        for (const string& word : words) wordCount[word]++;

        for (int i = 0; i < wordLen; i++) {
            unordered_map<string, int> window;
            int left = i, right = i, count = 0;

            while (right + wordLen <= s.size()) {
                string word = s.substr(right, wordLen);
                right += wordLen;

                if (wordCount.count(word)) {
                    window[word]++;
                    count++;

                    while (window[word] > wordCount[word]) {
                        string leftWord = s.substr(left, wordLen);
                        window[leftWord]--;
                        left += wordLen;
                        count--;
                    }

                    if (count == words.size()) result.push_back(left);
                } else {
                    window.clear();
                    count = 0;
                    left = right;
                }
            }
        }

        return result;
    }
};
```

#### **Detailed Steps**
1. **Build Word Frequency Map**:
   - Store the count of each word in `words` using `unordered_map`.
2. **Sliding Window**:
   - Traverse the string `s` in `wordLen` chunks to ensure alignment with word boundaries.
   - Maintain a `window` map to track the frequency of words in the current substring.
   - Adjust the window size dynamically to ensure no word appears more frequently than it does in `words`.
3. **Check Valid Substring**:
   - If the `window` map matches the `wordCount` map, add the starting index to the result.

#### **Complexity Analysis**
- **Outer Loop**: `O(wordLen)` to handle alignment.
- **Sliding Window**: `O(n / wordLen)` to traverse `s` in chunks.
- **Word Operations**: `O(1)` per word due to hash map operations.

**Overall Complexity**: **`O(n)`**.

---

### **Comparison Between Solutions**

| **Approach**      | **Time Complexity** | **Space Complexity** | **Scalability** |
|--------------------|---------------------|-----------------------|-----------------|
| Naive (`O(n * k^2)`) | `O(n * k^2)`        | `O(k)`                | Poor            |
| Optimized (`O(n)`) | `O(n)`              | `O(k)`                | Excellent       |

---

### **Key Insights**
1. Avoid redundant operations like permutation generation by using frequency maps.
2. Sliding windows are efficient for fixed-length substring problems.
3. Hash maps provide fast lookups and updates, making them ideal for this use case.
