---
title: Solving Length of Longest Substring Without Repeating Characters
tags: competitive-programming leetcode
key: longest-substring-no-repeat
---

Finding the length of the longest substring with no repeating characters is a straightforward problem. Given a string `s`, find the length of the longest substring that has no repeating characters. It is not difficult to come up with a naive solution; a one-pass solution however, requires a bit of creative thinking.

<!--more-->

The full problem description, with examples, can be found on [LeetCode](https://leetcode.com/problems/longest-substring-without-repeating-characters/).

# Naive Solution
A naive approach is to form substrings that start with each character of `s` - this is done by continuously appending the characters next to them until a repeated character is reached. Then, we note the length of the substring, and repeat the process again using the character next to the starting character. 

Let's use `abcabcabc` as an example. We start with `a`, and append `b` then `c` to form the substring `abc`. The next character is `a` which already exists in the substring - so we do not append `a`. The length of this substring is hence `3`. We repeat the process using `b` as the starting character: we get `bca` before encountering `b` as the next character. This process repeats until all characters had a chance to be the starting token of a substring.

The code for the naive approach can be written as such:
```javascript
function lengthOfLongestSubstring(s) {
    let maxLength = 0;    

    for (let i = 0; i < s.length; i++) {
        let substring = s[i];
        for (let j = i + 1; j < s.length; j++) {
            if (substring.includes(s[j])) {
                break; // form new substring with different starting character
            }
            substring += s[j];
        }
        maxLength = Math.max(maxLength, substring.length);
    }

    return maxLength;
};
```

This approach has time complexity of O(n<sup>2</sup>). In the worst case, every iteration of the outer loop fully iterates through the inner loop, that is, the inner loop iterates from `j = i + 1` to `j = s.length - 1`. 

While this is a working solution, there is evidently a flaw to the algorithm. Every time we want to form a new substring, we start all over with another starting character. If a character repeats in the next few characters, it is likely that any substring we form in between the two repeating characters will not be longer than the one we just formed. Thus, we can avoid forming certain substrings and cut down on time complexity. This can be done through the use of a sliding window.
 
# Sliding Window 
The sliding window technique is a popular approach to solving iteration problems involving arrays or similar in a single pass, i.e., they have O(n) time complexity. The core idea is to form a subarray of a particular section of the original array (the 'window'), then adjust the subarray to remove earlier elements or include future elements of the original array into the subarray (the 'sliding'). 

As an illustration, suppose we have an array `['a', 'b', 'c', 'd', 'e']`. A sliding window of size 3 will form the following subarrays:

```
['a', 'b', 'c']
     ['b', 'c', 'd']
          ['c', 'd', 'e']
```

It is not a requirement for the sliding window to have a fixed size - as will be shown in our solution. We can also form the following subarrays, but what is key is the subarrays must be a slice of the original array - it must maintain the order of the array and we cannot join two different sections of the array together.

```javascript
['a', 'b', 'c']
     ['b', 'c']
          ['c', 'd']
               ['d']
                   ['e']
```

Sliding windows are typically implemented through the use of two pointers (e.g. `i` and `j`, or `left` and `right`, etc), which tracks the starting and ending positions of the current window. The end pointer (`j` or `right`) will be incremented every iteration, while the start pointer (`i` or `left`) will be updated only if certain conditions are true. Note that again it is not a requirement for a sliding window to be implemented using pointers - we can use an array instead, as will be shown in our solution.

Even though the sliding window is commonly used with arrays, it is also applicable for strings-related questions, since strings are an array of characters. 

## Solution
For our problem, we use the sliding window to form substrings. Suppose `s = "leetcode"`. When our substring (the window) is `le`, the next character is `e`. Rather than attempting to form a substring starting with the first `e`, we can instead skip it and start to form the new substring with the second `e`. Specifically, whenever we encounter a repeated character, we move the left side of the window forward until the repeated character in the window is gone. Whenever we encounter a new unique character, we move the right side of the window forward.

```javascript
['l', 'e', 'e', 't', 'c', 'o', 'd', 'e'] 
[]          // start
['l']
['l', 'e']  // next character is a repeat - move left side forward
     ['e']  // window still has repeated character - move left side forward again
        []  // window is empty - no repeated characters. move right side forward
          ['e']
          ['e', 't']
          ...
```

As another example, suppose `s = "banana"`. The window will slide as such:

```javascript
['b', 'a', 'n', 'a', 'n', 'a']
[]
['b']
['b', 'a']
['b', 'a', 'n']
     ['a', 'n']
          ['n']
          ['n', 'a']
          ...
```

The code for implementing the sliding window is as follows:

```javascript
function lengthOfLongestSubstring(s) {
    let window = [];
    let maxLength = 0;
    
    for (let c of s) {
        while (window.includes(c)) {
            window.shift();    // removes an element from the front
        }
        window.push(c);    // adds an element from the back
        maxLength = Math.max(maxLength, window.length);
    }
    
    return maxLength;
}
```

Since we only iterate through the string `s` just once, the algorithm has a time complexity of O(n).

## Improvement
In the above sliding window approach, we are modifying the window one character at a time. It is possible to instead slide the window multiple characters at a time. This can be done by tracking the index of the last occurring repeated character. 

Consider the above `s = "leetcode"` example again. The moment we encounter the first repeated `'e'`, we can immediately shift our window towards it, as opposed to shrinking the left side one character at a time before incrementing the right side by one. 

```javascript
['l', 'e', 'e', 't', 'c', 'o', 'd', 'e'] 
[]          
['l']
['l', 'e']  
          <can be removed> 
          ['e']
          ['e', 't']
          ...
```

We can use a hash map `seen` to track each character's last occurrence. For example, in the above example, the hash map would be `seen = {'l': 0, 'e': 1}` just before we encounter the second `'e'`. Then, we are able to quickly deduce the left side should start at `seen['e'] + 1 = 2`.

The code for this improved approach will look something like the following:

```javascript
function lengthOfLongestSubstring(s) {
    let maxLength = 0;
    let seen = {};
    
    let left = 0, right = 0;
    while (right < s.length) {
        let curr = s[right];  // current right most character of window
        if (curr in seen) {   // curr was seen before
            left = seen[curr] + 1;
        }
        seen[curr] = right;   // update last occurrence of character
        maxLength = Math.max(maxLength, right - left + 1);
        right++;
    }
    return maxLength;
}
```

There is an issue, however, with our current code, specifically with the line `left = seen[curr] + 1;`. By simply updating the left side of the window to be the right of the last occurrence of the current character, it is possible that we are expanding our window backwards. 

This sounds complicated, so let's use an example `s = "abba"` to illustrate. When `right = 3`, `curr = 'a'` and `seen['a'] = 0`. Then, `left = 1`, and `right - left + 1 = 3`. Our `maxLength` is hence `3`, which is obviously wrong - our window is `bba` when it should be `ba`. This happens because the last occurrence of `a`, i.e., with index `0`, is outside of our current window. Only when the last occurrence is within our current window then should we move the left side of the window past it. We can modify the above line to only update `left` if `left` is smaller than the index of the last occurrence of a character, i.e., the line will now be `left = Math.max(left, seen[curr]+1);`

```javascript
function lengthOfLongestSubstring(s) {
    let maxLength = 0;
    let seen = {};
    
    let left = 0, right = 0;
    while (right < s.length) {
        let curr = s[right];  // current right most character of window
        if (curr in seen) {   // curr was seen before
            left = Math.max(left, seen[curr]+1);  // updated line
        }
        seen[curr] = right;   // update last occurrence of character
        maxLength = Math.max(maxLength, right - left + 1);
        right++;
    }
    return maxLength;
}
```

Similar to the first sliding window approach, this solution has O(n) time complexity.