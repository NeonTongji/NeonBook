---
description: 字典树、LRU
---

# 设计类

## 十四、字典树

### 构建字典树

1. 建立TrieNode 类。包含两个域，① boolean isEnd ② TrieNode\[] next
2. 初始化TrieNode。① isEnd = false ② next = new TrieNode\[26]
3. 初始化Trie。包含一个域，root。 root = new TrieNode(
4. 实现三个API。① startwith ② search ③ insert
5. insert实现：不为空，指针向下；为new TrieNode
6. search实现：为空，false；指针向下；遍历结束判断isEnd?
7. startWith实现：为空，false；指向向下；遍历结束返回true。

```java
class Trie {
    
    private class TrieNode{
        private boolean isEnd;
        private TrieNode[] next;

        public TrieNode() {
            this.isEnd = false;
            this.next = new TrieNode[26];
        }
    }

    private TrieNode root;

    public Trie() {
        this.root = new TrieNode();
    }
    
    public void insert(String word) {
        TrieNode cur = root;

        char[] arr = word.toCharArray();

        for(char c : arr) {
            if(cur.next[c - 'a'] == null) {
                cur.next[c - 'a'] = new TrieNode();
            }
            cur = cur.next[c - 'a'];
        }
        cur.isEnd = true;
    }
    
    public boolean search(String word) {
        TrieNode cur = root;

        char[] arr = word.toCharArray();

        for(char c : arr) {
            if(cur.next[c - 'a'] == null) return false;
            cur = cur.next[c - 'a'];
        }
        return cur.isEnd;
    }
    
    public boolean startsWith(String prefix) {
        TrieNode cur = root;

        char[] arr = prefix.toCharArray();

        for(char c : arr) {
            if(cur.next[c - 'a'] == null) return false;
            cur = cur.next[c - 'a'];
        }
        return true;
    }
}
```

### LeetCode472 连接字符串

Given an array of strings words (without duplicates), return all the concatenated words in the given list of words.

A concatenated word is defined as a string that is comprised entirely of at least two shorter words in the given array.

Example 1:

Input: words = \["cat","cats","catsdogcats","dog","dogcatsdog","hippopotamuses","rat","ratcatdogcat"] Output: \["catsdogcats","dogcatsdog","ratcatdogcat"] Explanation: "catsdogcats" can be concatenated by "cats", "dog" and "cats"; "dogcatsdog" can be concatenated by "dog", "cats" and "dog"; "ratcatdogcat" can be concatenated by "rat", "cat", "dog" and "cat"

### 分析

遇见字符串数组，可以考虑采用字典树Trie。

先把words排序，一个个查找是否能连接成，如果是元单词，加入字典树。 如果有一个单词search到了一部分，则把这个单词 剪掉search到的那部分，剩下继续search，如果最后剪成空，就可以连接成，否则加入字典树

```java
class Solution {
    Trie trie = new Trie();

    public List<String> findAllConcatenatedWordsInADict(String[] words) {
 
      	List<String> ans = new LinkedList<>();
 
        Arrays.sort(words, (a, b) -> a.length()  - b.length()); //字符串按长度升序

        for(String word : words) {
            // System.out.println(word);
            if(word.length() == 0) continue; // 空格跳过
            if(dfs(word, 0)) { // 找到了word可以被合成
                ans.add(word);
            } else { //不可以被合成
                trie.insert(word); //插入字典树
            }
        }

        return ans;
        
    }

    /** @ return: 有没有在字典树中找到word
        @ word: 要查找的单词
        @ index: 查找开始的索引
     */
    private boolean dfs(String word, int index) {
        
        if(index == word.length()) return true;
        
        // 从index开始查找word
        Trie cur = trie;
        
        char[] c = word.toCharArray();

        for(int i = index; i < c.length; i++) {
            cur = cur.next[c[i] - 'a'];
            if(cur == null) return false;
            if(cur.isEnd && dfs(word, i + 1)) {
                return true;
            }
        }

        return false; // 分支比单词还长
    }

   
    class Trie {

        private boolean isEnd;
        private Trie[] next;

        public Trie() {
            isEnd = false;
            next = new Trie[26];
        }
       

        public void insert(String word) {
            Trie cur = this;

            char[] arr = word.toCharArray();

            for(char c : arr) {
                if(cur.next[c - 'a'] == null) {
                    cur.next[c - 'a'] = new Trie();
                }
                cur = cur.next[c - 'a'];
            }
            cur.isEnd = true;
        }
    }
}
```

##
