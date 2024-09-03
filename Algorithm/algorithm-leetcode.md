<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [关于 Map](#%E5%85%B3%E4%BA%8E-map)
  - [49-字母异位词分组](#49-%E5%AD%97%E6%AF%8D%E5%BC%82%E4%BD%8D%E8%AF%8D%E5%88%86%E7%BB%84)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 关于 Map

## 49-字母异位词分组

```js
/**
 * 给你一个字符串数组，请你将 字母异位词 组合在一起。可以按任意顺序返回结果列表。
 * 字母异位词 是由重新排列源单词的所有字母得到的一个新单词。
 * 示例 1:
 * 输入: strs = ["eat", "tea", "tan", "ate", "nat", "bat"]
 * 输出: [["bat"],["nat","tan"],["ate","eat","tea"]]
 */
const readline = require("readline");

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
});
rl.on("line", (input) => {
  if (input === "exit" || input === "quit") {
    rl.close();
  } else {
    const strs = JSON.parse(input);
    const groupAnagrams = (strs) => {
      let codeToGroup = new Map();
      for (let s of strs) {
        let code = encode(s);
        if (!codeToGroup.has(code)) {
          codeToGroup.set(code, []);
        }
        codeToGroup.get(code).push(s);
      }
      let res = [];
      for (let group of codeToGroup.values()) {
        res.push(group);
      }
      console.log(res);
    };
    // 利用了这个编码技巧
    const encode = (s) => {
      let count = new Array(26).fill(0);
      for (let c of s) {
        let m = c.charCodeAt() - "a".charCodeAt();
        console.log(m);
        count[m]++;
      }
      return count.toString();
    };
    groupAnagrams(strs);
  }
});
rl.on("close", () => {
  console.log("程序结束");
});
```
