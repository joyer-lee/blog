---
title: 3D前端开发面试题目解析
tags: []
id: '59'
categories:
  - - 面试
date: 2020-10-23 18:26:22
---

![](../images/uploads/sites/2/2020/10/QQ20201023-145902@2x.png)

### 前言

> 在我之前的业务流程中，有幸接触到了3D前端开发的内容，从layaAir、verge到threejs,最终采用了threejs技术去实现了一个3D服装定制的效果。近期，接到了一个专业3D开发团队的面试邀请，虽然发挥的不尽人意，但仍不失为一个很好的面试经历。特将面试问题记录一下。

### 笔试题

1.编写一种算法，若M\*N矩阵中某个元素为0，则将其所在的行与列清零。

```javascript
//示例：
[
    [1,1,1,1],
    [1,1,0,1],
    [1,1,1,1],
    [1,1,1,1]
]
//输出
[
    [1,1,0,1],
    [0,0,0,0],
    [1,1,0,1],
    [1,1,0,1]
]
```

解读：二维数组矩阵中，含有0的位置，让此位置整行，整列的值都设置为0，返回修改后的数组数列。 思路：先循环矩阵数列，记录需要清零的行和列，再循环修改数组的元素值。 代码实现如下：

```javascript
var matrix=[
    [1,1,1,1],
    [1,1,0,1],
    [1,1,1,1],
    [1,1,1,1]
];
var rowLength=matrix.length,columnLength=matrix[0].length;
//记录需要清零的行数和列数
var tempR=[],tempC=[];

for(var i=0;i<rowLength;i++){
    for(var j=0;j<columnLength;j++){
        if(!matrix[i][j]){
            tempR.push(i);
            tempC.push(j);
        }
    }
}

for(i=0;i<rowLength;i++){
    for(var j=0;j<columnLength;j++){
        if(tempR.includes(i)tempC.includes(j)){
            matrix[i][j]=0;
        }
    }
}

console.log(matrix);
//返回如下
//[
//  [1, 1, 0, 1]
//  [0, 0, 0, 0]
//  [1, 1, 0, 1]
//  [1, 1, 0, 1]
//]

```