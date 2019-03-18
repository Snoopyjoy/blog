---
layout: post
title: 快速排序
category: algorithm
description: 快速排序算法是将数组按序不断分解，直到所有子数组不能再分，那么所有子数组拼接起来就是一个完整的有序数组。
keywords: 函数式,快速排序,Javascript
---
&#160; &#160; &#160; &#160; 

<!--description--> 
### 快速排序的原理
&#160; &#160; &#160; &#160;  快速排序算法是将数组按序不断分解，直到所有子数组不能再分，那么所有子数组拼接起来就是一个完整的有序数组。  
&#160; &#160; &#160; &#160;  首先，从数组中选取一个数作为基准值，比这个基准值小的元素放在左边，大的放在右边。然后对左右两个新数组继续进
行上述步骤，直到所有的子序列只剩下一个元素。如下图（图中取数组第一个元素作为基准值）:  

![]({{site.baseurl}}/assets/img/quick_sort.png)
    
### 快速排序的实现
&#160; &#160; &#160; &#160;  根据快速排序的原理使用Javascript的实现:  
```javascript
const quickSort = (array) => {
  const sort = (arr, left = 0, right = arr.length - 1) => {
    if (left >= right) {//左边的索引大于等于右边的索引则表示不能继续分解
      return;
    }
    let i = left;
    let j = right;
    const pivot = arr[left]; // 取无序数组第一个数为基准值
    while (i < j) { //把所有比基准值小的数放在左边 大的数放在右边
      while (j > i && arr[j] >= pivot) { //从右向左找一个小于基准值的元素
        j--;
      }
      arr[i] = arr[j]; //将较小的值放在当前i的位置 没找到的情况下i 等于 j
      while (i < j && arr[i] <= pivot) { //从左向右找大于基准值的元素
        i++;
      }
      arr[j] = arr[i]; // 将较大的值放在当前j的位置 没找到的情况下i 等于 j
    }
    arr[i] = pivot; // 将基准值放至i位置完成一次循环（这时候 j 等于 i ）
    sort(arr, left, j - 1); // 将左边的无序数组重复上面的操作
    sort(arr, j + 1, right); // 将右边的无序数组重复上面的操作
  };
  sort(array);
  return array;
}
```
&#160; &#160; &#160; &#160;  -_-!上面那段代码是不是感觉很难读懂，尤其是在将数组按基准值将元素放到左右两边的时候。更好理解的版本:  
```javascript
const quickSort1 = function(arr) {
  if (arr.length <= 1) { //如果数组长度小于等于1无需继续分解
    return arr;
  }
  const pivot = arr.shift(); //取基准值的值
  const left = []; //存放比基准值小的数组
  const right = []; //存放比基准值大的数组
  for (let i = 0; i < arr.length; i++){ //遍历数组，进行判断分配
    if (arr[i] < pivot) {
      left.push(arr[i]); //比基准值小的放在左边数组
    } else {
      right.push(arr[i]); //比基准值大的放在右边数组
    }
  }
  //递归执行以上操作,对左右两个数组进行操作，直到数组长度为<=1
  return quickSort1(left).concat([pivot], quickSort1(right));
};
```
&#160; &#160; &#160; &#160; 上述代码就清晰多了，这段代码很容易可以很清晰的展示快速排序的过程，不过也是有代价的，因为每次递归都创建了新的
数组，所以运行时内存的需求也就更大。  
&#160; &#160; &#160; &#160; 下面是使用函数式编程工具库[Ramda](https://github.com/ramda/ramda)实现的快速排序：
```javascript
const R = require('ramda');
const quickSort3 = R.unless(        //直到返回空数组长度为1停止递归
  arr=>R.length(arr)<=1,
  ([pivot, ...rest]) => [
    ...R.compose(quickSort3, R.filter(R.lt(R.__, pivot)))(rest),    //比基准值小的放到左边
    pivot,
    ...R.compose(quickSort3, R.filter(R.gte(R.__, pivot)))(rest),   //大于等于基准值的放到右边
  ]
);
```

&#160; &#160; &#160; &#160; 以上对快速排序算法的研究更多的是关注代码的实现，通过代码的实现来理解快速排序的原理。  
### 时间复杂度
&#160; &#160; &#160; &#160; 平均状况下时间复杂度为O(n log(n)), 最坏情况下时间复杂度为O(n<sup>2</sup>)

### 参考
[https://zhuanlan.zhihu.com/p/30936617](https://zhuanlan.zhihu.com/p/30936617)
[http://www.ruanyifeng.com/blog/2011/04/quicksort_in_javascript.html](http://www.ruanyifeng.com/blog/2011/04/quicksort_in_javascript.html)




