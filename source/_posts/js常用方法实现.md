---
title: js常用方法实现
date: 2019-12-16 09:47:43
tags: [js]
categories: 技术
---

js中一些常用的方法，在项目开发中都是想到哪儿写到哪儿，难免有不周全之处，所以有必要归纳总结一下。

#### 1、给`number`类型添加千分位符

要实现这个功能有两种方法：
> 1、如果是整数，可以用内置的`Number.prototype.toLocaleString`方法，如：
```javascript
  1234567.toLocaleString() // 1,234,567
```
> 2、如果要给小数位也加千分位符，那么`toLocaleString`就爱莫能助了，该是正则表达式上场了，先贴出具体实现吧：
```javascript
  const localeFun = str => str.replace(/(\d)(?=(?:\d{3})+\.\d+$)|(?<=^\d+\.(?:\d{3})*\d{2})(\d)|(?<!\d*\.\d*)(\d)(?=(?:\d{3})+$)/g, (_, p1, p2, p3) => {
    if (p1) return `${p1},`
    if (p2) return `${p2},`
    if (p3) return `${p3},`
  })
```
这个方法不仅实现为小数添加整数部分和小数部分的千分位符，还实现了为纯整数添加千分位符，也就是涵盖了方法一的功能。这串正则表达式看似复杂，待我慢慢拆解。首先，`g`修饰符不可少，因为整体思路就是要去检测每一个字符所处的位置是否符合加千分位符的条件的，是一个遍历的过程！然后，主体部分分为三个`|`分别实现为整数位加千分位符、为小数位加千分位符、为纯整数加千分位符。第一部分，匹配一个数字`(\d)`，加括号是为了捕获它，以便于在`replace`第二个参数中使用替换，`(?=)`表示该数字的右边必须紧跟的内容，`(?:\d{3})+`表示包含至少一组由三位数字组成的匹配项，而`(?:)`表示非捕获，如果不加这个规则的话，回调中的`p2`就会被它占用而不能表示后续的捕获项。`\.\d+$`表示以小数点后有至少一位的整数字符结尾（如果后面没有数字那么则视为无效的小数，不予以处理）！满足这样规则的则视为匹配小数的整数部分，另外两个规则读者可以试着动手解析一下。

#### 2、获取一个变量的类型
首先要知道，js的数据类型分为基本数据类型和引用类型，基本数据类型有以下几种：`Number、String、Boolean、Null、Undefined、Symbol`，引用类型分为`Object`和`Array`。说到获取变量的类型，首先想到的就是`js`原生操作符`typeof`，遗憾的是，`typeof`对于`null`检测出来的结果是`object`并不是`null`，对于`object`和`array`检测出来的结果都是`object`，所以，要想准确获得变量类型就只能另辟蹊径了。
```javascript
  const getType = param => Object.prototype.toString.call(param).match(/(?<=^\[object\s)[^]+(?=\]$)/).toLowerCase()
```
采用`object`的`toString`方法，会返回`[object xxx]`的形式，而`xxx`恰好就是参数的类型，然后用正则表达式去掉我们不需要的部分，保留类型，把首字母也转成小写即可。这里使用了`Object.prototype.toString.call()`的形式来得到包含类型的字符串，是因为其他基础类型改写了`object`的`toString`方法，比如，`number`的`toString`方法返回数字的字符串形式，`function`的`toString`返回定义方法的字符串形式，只有顶层的`object`的`toString`方法还保留着获取类型的功能。

#### 3、常见排序的js实现

##### 3.1 冒泡排序
```javascript
const bubbleSort = arr => {
  const [length] = arr
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

##### 3.2 选择排序
```javascript
const selectionSort = arr => {
  const [length] = arr;
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
这种排序相当于每次锁定一个位置，然后用这个位置上的值与其后的所有值进行比较，记录下比锁定位置的值小的那个值的索引，本轮比较结束后将记录下位置的值与锁定位置的值进行交换，那么锁定位置就是最小值了。

##### 3.3 插入排序
```javascript
const insertionSort = arr => {
  const [length] = arr;
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
插入排序的思想，就是保证`i`之前的所有值都是有序的。用`i`这个位置的值与前面的值从后往前比较，先把`i`处的值用`temp`存起来，以防在值移动过程中被覆盖。如果`j`处的值比`i`处的值大，那么把`j`处的值往后移一个位置，移到`j+1`处，把`j`这个位置空出来，然后`j--`进行下一次比较。如果此时`i`处的值比`j`处小或者相等了，那么也没必要再往前进行比较了，因为`i`之前的数都是有序的。那么把`i`处的值放在空出来的那个位置，即`j+1`这个位置，即完成插入排序。

未完待续。。。