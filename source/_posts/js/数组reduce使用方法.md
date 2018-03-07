
---
title: 数组reduce使用方法
date: 2018-03-06 11:29:44
tags: [js]
categories: js
---

在JS新特性中添加了`reduce()`方法十分好用，它可以将数组中的每一项通过叠加变成一项。
下面从一个简单的例子来说明：
```js
var  arr = [1, 2, 3, 4, 5];
sum = arr.reduce(function(prev, cur, index, arr) {
    console.log(prev, cur, index);
    return prev + cur;
})
console.log(arr, sum);
```

输出结果：
>1 2 1   
3 3 2   
6 4 3    
10 5 4    
[1, 2, 3, 4, 5] 15

我们先重新回顾一下reduce中回调函数的参数，这个回调函数中有4个参数，意思分别为

>prev: 第一项的值或者上一次叠加的结果值    
cur: 当前会参与叠加的项    
index： 当前值的索引    
arr: 数组本身    

首先我们要区分prev与cur这2个参数的区别，刚开始的时候我以为他们是一种类型的，可是后来我发现我理解错了。prev表示每次叠加之后的结果，类型可能与数组中的每一项不同，而cur则表示数组中参与叠加的当前项。在后边我们可以结合实例来理解这个地方。

其次我们看到，上例中其实值遍历了4次，数组有五项。数组中的第一项被当做了prev的初始值，而遍历从第二项开始。

我们看下面一个例子。
某同学的期末成绩如下表示
```js
var result = [
    {
        subject: 'math',
        score: 88
    },
    {
        subject: 'chinese',
        score: 95
    },
    {
        subject: 'english',
        score: 80
    }
];
```

如何求该同学的总成绩？

很显然，利用for循环可以很简单得出结论

```js
var sum = 0;
for(var i=0; i<result.length; i++) {
    sum += result[i].score;
}
```

但是我们的宗旨就是抛弃for循环，因此使用reduce来搞定这个问题

```js
var sum = result.reduce(function(prev, cur) {
    return cur.score + prev;
}, 0);
```

这个时候，我给reduce参数添加了第二个参数。通过打印我发现设置了这个参数之后，reduce遍历便已经从第一项开始了。

这第二个参数就是设置prev的初始类型和初始值，比如为0，就表示prev的初始值为number类型，值为0，因此，reduce的最终结果也会是number类型。

我们来给这个例子增加一点难度。假如该同学的总成绩中，各科所占的比重不同，分别为50%，30%，20%，我们应该如何求出最终的权重结果呢？

解决方案如下：
```js
var dis = {
    math: 0.5,
    chinese: 0.3,
    english: 0.2
}

var sum = result.reduce(function(prev, cur) {
    console.log(prev);
    return cur.score + prev;
}, 0);

var qsum = result.reduce(function(prev, cur) {
    return cur.score * dis[cur.subject] + prev;
}, 0)

console.log(sum, qsum);
```

再看另一个例子，如何知道一串字符串中每个字母出现的次数？

我们可以运用reduce来解决这个问题。

我们在reduce的第二个参数里面初始了回调函数第一个参数的类型和值，将字符串转化为数组，那么迭代的结果将是一个对象，对象的每一项key值就是字符串的字母。运行感受一下吧。
```js
var arrString = 'abcdaabc';

arrString.split('').reduce(function(res, cur) {
    res[cur] ? res[cur] ++ : res[cur] = 1
    return res;
}, {})
```