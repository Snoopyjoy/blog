---
layout: post
title: 算法-Z 字形变换
categories: Algorithm
description: 将一个给定字符串根据给定的行数，以从上往下、从左到右进行 Z 字形排列。
keywords: algorithm,leetcode
---
### 题目
将一个给定字符串根据给定的行数，以从上往下、从左到右进行 Z 字形排列。
### 解题
首先Z出现频率是有周期性的。然后就可以根据这个周期规律找到某个字符对应的行。最后把每行的字符串拼接就可以得到最终结果了。时间复杂度为O(n)。
/**
 * @param {string} s
 * @param {number} numRows
 * @return {string}
 */
var convert = function(s, numRows) {
    let gapLen = numRows-2;
    gapLen = gapLen < 0 ? 0:gapLen;
    const roundLen = numRows+gapLen;
    const lines = [];
    for( let i=0; i<s.length; i++ ){
        const charStr = s[i];
        const indexRow = getIndexRow( i, numRows,roundLen,gapLen );
        if( lines[indexRow] ){
            lines[indexRow] += charStr;
        }else{
            lines[indexRow] = charStr;
        }
    }
    return lines.join("");
};

function getIndexRow(index,rows,roundLen,gapLen){
    const roundIndex = index%roundLen;
    if(roundIndex >= rows ){
       return gapLen - ( roundIndex - rows );
    }else{
        return roundIndex;
    }
    
}
```