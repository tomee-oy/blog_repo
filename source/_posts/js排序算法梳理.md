---
title: js排序算法梳理
date: 2019-12-19 16:07:29
tags:
categories:
---
排序算法是`js`中最基础的算法，理解各大排序算法的本质，是学习更深层次算法的基础。

### 一、冒泡排序
```javascript
const bubbleSort = arr => {
  const length = arr.length
  for(let i = 0; i < length; i++) {
    for(let j = 0; j < length - i - 1; j++) {
      if (arr[j] > arr[j + 1]) {
        [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]]
      }
    }
  }
  return arr
}
```
每次从每一个数开始，与相邻的数比较，如果它比相邻的数大（以升序排列为例），则交换位置，然后进行下一次比较。所以每次比较完的最后一个数肯定是所有比较过的数当中最大的。这种算法的大O复杂度为O(n*n)。

![冒泡排序动态示意图](js排序算法梳理/bubbleSort.gif)

### 二、选择排序
```javascript
const selectionSort = arr => {
  const length = arr.length
  let minIndex;
  for (let i = 0; i < length - 1; i++) {
    minIndex = i;
    for (let j = i + 1; j < length; j++) {
      if (arr[j] < arr[minIndex]) {
        minIndex = j
      }
    }
    if (i !== minIndex) {
      [arr[i], arr[minIndex]] = [arr[minIndex], arr[i]];
    }
  }
  return arr
}
```
这种排序相当于每次锁定一个位置，然后用这个位置上的值与其后的所有值进行比较，记录下比锁定位置的值小的那个值的索引，本轮比较结束后将记录下位置的值与锁定位置的值进行交换，那么锁定位置就是最小值了。这种算法的大O复杂度为O(n*n)。

![选择排序动态示意图](js排序算法梳理/selectionSort.gif)

### 三、插入排序
```javascript
const insertionSort = arr => {
  const length = arr.length
  let temp;
  for (let i = 1; i < length; i++) {
    temp = arr[i]
    let j = i - 1
    while(j >= 0 && arr[j] > temp) {
      arr[j + 1] = arr[j]
      j--
    }
    arr[j + 1] = temp
  }
  return arr;
};
```
插入排序的思想，就是保证`i`之前的所有值都是有序的。用`i`这个位置的值与前面的值从后往前比较，先把`i`处的值用`temp`存起来，以防在值移动过程中被覆盖。如果`j`处的值比`i`处的值大，那么把`j`处的值往后移一个位置，移到`j+1`处，把`j`这个位置空出来，然后`j--`进行下一次比较。如果此时`i`处的值比`j`处小或者相等了，那么也没必要再往前进行比较了，因为`i`之前的数都是有序的。那么把`i`处的值放在空出来的那个位置，即`j+1`这个位置，即完成插入排序。这种情况的大O复杂度为O(n*n)。

![插入排序动态示意图](js排序算法梳理/insertionSort.gif)

### 四、归并排序
```javascript
// 将数组进行二分拆解
const mergeSort = arr => {
  const length = arr.length
  if (length < 2) return arr
  const middle = Math.floor(length / 2)
  const left = arr.slice(0, middle)
  const right = arr.slice(middle)
  return merge(mergeSort(left), mergeSort(right))
}

// 将两个有序数组合并成一个有序数组
const sort = (left, right) => {
  let result = []
  while(left.length && right.length) {
    if(left[0] > right[0]) {
      result.push(right.shift())
    } else {
      result.push(left.shift())
    }
  }
  result = result.concat(left)
  result = result.concat(right)
  return result
}
```
这种排序算法的核心思想有两点：一、分治法，即将数组进行二分拆解，对左右两部分分别进行排序；二、合并两个有序数组，被拆分后的数组排完序以后需要合并成一个数组。第二个思想的重点在于，两个有序数组如何合并？`sort`方法始终比较两个数组的第一个元素，哪个较小就把哪个取出来放进合并后的数组里，直到其中一个数组为空。然后，将两个数组中剩余元素追加到`result`数组末尾（剩余元素一定比`result`中的所有元素都大），即完成合并。归并排序的大O复杂度为O(n*log2<sup>n</sup>)，相比前三种排序算法操作数有所优化，但它的内存开销比前三种也高出不少，即牺牲内存换效率。

![归并排序动态示意图](js排序算法梳理/mergeSort.gif)

### 五、快速排序
```javascript
const quickSort = arr => {
  const length = arr.length
  if (length < 2) return arr
  const middle = Math.floor(length / 2)
  const base = arr.splice(middle, 1)[0]
  const left = [], right = []
  while(arr.length) {
    const current = arr.shift()
    if (current > base) {
      right.push(current)
    } else {
      left.push(current)
    }
  }
  return quickSort(left).concat([base], quickSort(right))
}
```
快速排序核心思想：找数组的中间数为基准，找出比它大的和比它小的各自放在一个数组，然后通过数组连接把比它小的数全部放在左边，比它大的全部放在右边，组成一个新的数组并返回，达到数组局部有序的效果。同样，对于左右两边的数组，可以采取同样的方式，最终达到全部有序。这种排序算法和归并排序有异曲同工之妙，都采用分治法的思想。但二者的区别也是很明显的，归并排序是先将数组进行一次一次地二分，当达到最小单元后再开始合并，合并的过程中进行有序排列；而快速排序则是在拆分的过程中就先保证局部有序，当拆分到最小单元时就全部有序了，所以合并时采用单纯的`concat`将数组连接起来即可。这种排序的大O复杂度也为O(n*log2<sup>n</sup>)。

![快速排序动态示意图](js排序算法梳理/quickSort.gif)