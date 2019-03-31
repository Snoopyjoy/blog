---
layout: post
title: 算法-寻找两个有序数组的中位数
date: 2019-03-26 15:38:55 +0800
categories: Algorithm
description: 给定两个大小为m和n的有序数组 nums1 和 nums2。 请你找出这两个有序数组的中位数，并且要求算法的时间复杂度为O(log(m+n))。
keywords: algorithm,leetcode
---
### 题目
给定两个大小为m和n的有序数组 nums1 和 nums2。 请你找出这两个有序数组的中位数，并且要求算法的时间复杂度为O(log(m+n))。你可以假设nums1和nums2不会同时为空。
### 解题
第一印象就是把两个数组合并排序就可以找出中位数了。但是题目中有要求时间复杂度为O(log(m+n))，如果用使用快速排序的时间复杂度也要O((m+n)log(m+n))，所以排除直接排序的做法。
首先为了得出结果，需要达成的目标就是得出两个长度一致的数组，且左边数组的最大值要小于右边数组的最小值。具体做法是把A数组分成A1（左），A2（右）两个部分，B数组分成B1（左），B2（右）两个部分，其中A1的长度加上B1的长度等于A2的长度加上B2的长度。如果A1、B1中的最大值小于A2、B2中的最小值那么就找到答案了。代码如下：
```javascript
/**
 * @param {number[]} nums1
 * @param {number[]} nums2
 * @return {number}
 */
var findMedianSortedArrays = function(nums1, nums2) {
    const swap = nums1.length>nums2.length;
    const arrA = swap?nums2:nums1;
    const arrB = swap?nums1:nums2;

    const totalLen = nums1.length + nums2.length;
    const halfLen = ((totalLen+1)/2)>>0;

    if( totalLen === 0 ){
       return 0;
    }

    if( totalLen === 1 ){
       return arrB[0];
    }

    let posA = (arrA.length/2)>>0;
    let posB = halfLen - posA;

    while( posA >=0 && posA <= arrA.length ){

        let leftAMax = arrA[posA-1];
        let leftBMax = arrB[posB-1];
        let leftMax;
        if(leftAMax === undefined ){
            leftMax = leftBMax;
        }else if( leftBMax === undefined ){
            leftMax = leftAMax
        }else{
            leftMax = Math.max(leftAMax, leftBMax);
        }

        let rightAMin = arrA[posA];
        let rightBMin = arrB[posB];
        let rightMin;
        if( rightAMin === undefined ){
            rightMin = rightBMin;
        }else if( rightBMin === undefined ){
            rightMin = rightAMin;
        }else{
            rightMin = Math.min(rightAMin,rightBMin);
        }

        if(rightMin >= leftMax){
            if( totalLen % 2 === 1 ) return leftMax;
            return (leftMax + rightMin) / 2;
        }else if( leftAMax > rightBMin ){
            posA--;
            posB++;
        }else{
            posA++;
            posB--;
        }
    }
    return 0;

};
```