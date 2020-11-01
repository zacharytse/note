[TOC]
# 题目
给定一个非空字符串 s 和一个包含非空单词列表的字典 wordDict，在字符串中增加空格来构建一个句子，使得句子中所有的单词都在词典中。返回所有这些可能的句子。

说明：

分隔时可以重复使用字典中的单词。
你可以假设字典中没有重复的单词。

# 思路
采用自顶向下的记忆化搜索
记忆化的实现是通过一个HashMap存储不同下标可以组成的不同语句，如果在回溯过程中遇到已经计算过的下标，则直接返回结果
```java{.line-numbers}
class Solution {
    public List<String> wordBreak(String s, List<String> wordDict) {
        Map<Integer,List<List<String>>> map = new HashMap<>();
        List<List<String>> wordBreaks = backtrack(s,s.length(),new HashSet<String>(wordDict),0,map);
        List<String> breakList = new LinkedList<>();
        for(List<String> wordBreak : wordBreaks){
            breakList.add(String.join(" ",wordBreak));
        }
        return breakList;
    }

    List<List<String>> backtrack(String s,int length,Set<String> dict,int index,
    Map<Integer,List<List<String>>> map){
        if(!map.containsKey(index)){
            List<List<String>> wordBreaks = new LinkedList<>();
            if(index == length){
                wordBreaks.add(new LinkedList<String>());
            }
            for(int i = index + 1; i <= length; ++i){
                String word = s.substring(index,i);
                if(dict.contains(word)){
                    List<List<String>> nextWordBreaks = backtrack(s,length,dict,i,map);
                    for(List<String> nextWordBreak : nextWordBreaks){
                        LinkedList<String> wordBreak = new LinkedList<String>(nextWordBreak);
                        wordBreak.offerFirst(word);
                        wordBreaks.add(wordBreak);
                    }
                }
            }
            map.put(index,wordBreaks);
        }
        return map.get(index);
    }
}
```