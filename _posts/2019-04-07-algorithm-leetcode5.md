---
layout: post
title: 算法-寻找最长回文子串, Manacher算法
date: 2019-04-07 21:05:00 +0800
categories: Algorithm
description: 给定一个字符串 s，找到 s 中最长的回文子串。你可以假设 s 的最大长度为 1000。
keywords: algorithm,leetcode,javascript,manacher
---
### 题目
给定一个字符串 s，找到 s 中最长的回文子串。你可以假设 s 的最大长度为 1000。
### 解题
首先什么是回文？回文就是对称的字符串，正读反读都一样。每个字符都要左右寻找是否对称。因此时间复杂度为O(n<sup>2</sup>)。
题解中还提到了一种时间复杂度为O(n)的算法，马拉车算法Manacher‘s Algorithm。  
  
首先中心对称算法的实现  
```javascript
/**
 * @param {string} s
 * @return {string}
 */
var longestPalindrome = function(s) {
    let maxRoundStr = "";
    for( let i = 0; i < s.length; i++ ){
        let roundResult = findRoundStr( i, s );
        const resultStr = roundResult.result;
        if( roundResult.offset > 0 ) i+= roundResult.offset;
        if( resultStr.length > maxRoundStr.length ){
            maxRoundStr = resultStr;
            //剩余字符不可能产生比当前更长的回文
            if( (maxRoundStr.length / 2) > s.length - i ) break;
            
        }
    }
    return maxRoundStr;
};

function findRoundStr( position, str ){
    let offsetRight = 1;
    let offsetLeft = 1;
    const startChar = str[position];
    let resultStr = str[position];
    let rightChar = str[position+offsetRight];
    while( rightChar === startChar ){
        resultStr += rightChar;
        offsetRight++;
        rightChar = str[position+offsetRight];
    }
    let offset = offsetRight - 1;
    let preChar = "";
    let nextChar = "";
    while( preChar != undefined && nextChar != undefined && preChar === nextChar ){
        resultStr = preChar + resultStr + nextChar;
        preChar = str[position-offsetLeft];
        nextChar = str[position+offsetRight];
        offsetLeft++;
        offsetRight++;
    }
    return {
        result: resultStr,
        offset: offset  //相同字符跳过
    };
}
```
Manacher算法实现  
```javascript
/**
 * @param {string} s
 * @return {string}
 */
var longestPalindrome = function(s) {
    //插入字符转换成奇数字符串
    const str = "$#"+s.split("").join("#")+"#$";
    //以各个位置的字符为中心的回文字符半径
    let p = [];
    //id位置的回文最右端位置
    let mx = 0;
    //参考位置id
    let id = 0;
    //回文最大半径
    let maxLen=0;
    //回文最大半径字符串中心位置
    let maxPos=0;
    for( let i = 1; i < str.length; i++ ){
        if( mx > i ){
           p[i] = Math.min( p[2*id-i], mx-i );
        }else{
           p[i] = 1;
        }
        while( str[ i-p[i] ] && str[ i + p[i] ] && str[ i-p[i] ] === str[ i + p[i] ] ) p[i]++;
        if( mx < p[i] + id ){
           id = i;
           mx = p[i] + i;
        }
        if( p[i] > maxLen  ){
           maxPos = i;
           maxLen = p[i];
        }
    }
    return s.substring((maxPos-maxLen)/2, (maxPos+maxLen)/2-1);
};

```
