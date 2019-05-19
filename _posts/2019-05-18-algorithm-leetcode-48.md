---
layout: post
title: 算法-旋转图像
categories: Algorithm
description: 定一个 n × n 的二维矩阵表示一个图像。将图像顺时针旋转 90 度。
keywords: algorithm,leetcode
---
### 题目
给定一个 n × n 的二维矩阵表示一个图像。

将图像顺时针旋转 90 度。

说明：

你必须在原地旋转图像，这意味着你需要直接修改输入的二维矩阵。请不要使用另一个矩阵来旋转图像。
### 解题
```javascript
/**
 * @param {number[][]} matrix
 * @return {void} Do not return anything, modify matrix in-place instead.
 */
var rotate = function(matrix) {
  const temp = [];
    const mtxLen = matrix.length;
    for( let i = mtxLen - 1;i >= 0; i-- ){
        const orgRow = matrix[i];
        const covI = mtxLen - i - 1;
        for( let j = 0; j < orgRow.length; j++){
            temp[i] = temp[i] || [];
            let orgVal = temp[i][j] || temp[i][j] === 0? temp[i][j] :
            matrix[i][j];
            temp[j] = temp[j] || [];
            temp[j][covI] = matrix[j][covI];
            matrix[j][covI] = orgVal;
        }
    }
    return matrix;
};
```
