# 你可能没有注意到的有用的 API

> 原文：<https://itnext.io/useful-apis-that-you-probably-dont-notice-a5c62c9366e9?source=collection_archive---------2----------------------->

![](img/3058740061a2d6b5576e49a1998aa761.png)

# 日期

# 获取一个月中的天数

下月 0 日是当月的最后一天。

```
function daysInMonth(year, month) {
    let date = new Date(year, month + 1, 0);
    return date.getDate();
}/**
 * Note that JS Date month starts with 0
 * The following computes how many days in March 2017
 */
console.log(daysInMonth(2017, 2)); // 31 
// how many days in Feb 2017
console.log(daysInMonth(2017, 1)); // 28
// how many days in Feb 2016
console.log(daysInMonth(2016, 1)); // 29
```

# [getTimezoneOffset](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/getTimezoneOffset) —获取当前区域设置(主机系统设置)与 UTC 之间的时差，以分钟为单位。

```
let now = new Date(); 
console.log(now.toISOString()); //2018-03-12T01:12:29.566Z
// China is UTC+08:00
console.log(now.getTimezoneOffset()); // -480
// convert to UTC 
let UTCDate = new Date(now.getTime() + now.getTimezoneOffset() * 60 * 1000);
console.log(UTCDate.toISOString()); //2018-03-11T17:12:29.566Z
//convert to UTC+03:00
let eastZone3Date = new Date(UTCDate.getTime() + 3 * 60 * 60 * 1000);
console.log(eastZone3Date.toISOString()); //2018-03-11T20:12:29.566Z
```

# JSON

# [JSON.stringify(值[，替换者[，空间]])](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)

# 当 replacer 是函数时—在字符串化值之前应用 replacer。

```
JSON.stringify({
    a: 4,
    b: [3, 5, 'hello'],
}, (key, val) => {
    if(typeof val === 'number') {
        return val * 2;
    }
    return val;
}); //{"a":8,"b":[6,10,"hello"]}
```

# 当 replacer 是数组时—使用 replacer 作为白名单来过滤关键字

```
JSON.stringify({
    a: 4,
    b: {
        a: 5,
        d: 6
    },
    c: 8
}, ['a', 'b']); //{"a":4,"b":{"a":5}}
```

# 空间可以用来美化输出

```
JSON.stringify({
    a: [3,4,5],
    b: 'hello'
}, null, '|--\t');
/**结果:
{
|--    "a": [
|--    |--    3,
|--    |--    4,
|--    |--    5
|--    ],
|--    "b": "hello"
}
*/
```

# 线

# [string . prototype . split([分隔符[，限制])](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/split)

###

```
''.split('') // []
```

# 分隔符可以是正则表达式！

```
'abc1def2ghi'.split(/\d/);  //["abc", "def", "ghi"]
```

如果分隔符是包含捕获组的正则表达式，则捕获组也会出现在结果中。

```
'abc1def2ghi'.split(/(\d)/);  // ["abc", "1", "def", "2", "ghi"]
```

# [标记字符串文字](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)

```
let person = 'Mike';
let age = 28;function myTag(strings, personExp, ageExp) {
  let str0 = strings[0]; // "that "
  let str1 = strings[1]; // " is a " // There is technically a string after
  // the final expression (in our example),
  // but it is empty (""), so disregard.
  // var str2 = strings[2]; let ageStr;
  if (ageExp > 99){
    ageStr = 'centenarian';
  } else {
    ageStr = 'youngster';
  } return str0 + personExp + str1 + ageStr;
}let output = myTag`that ${ person } is a ${ age }`;
console.log(output);
// that Mike is a youngster
```

# 空 vs 未定义

如果不想区分 null 和 undefined，可以用 *==*

```
undefined == undefined //true
null == undefined // true
0 == undefined // false
'' == undefined // false
false == undefined // false
```

不要简单地使用 *==* 来检查全局变量的存在，因为它会抛出 *ReferenceError* 。使用*类型的*代替。

```
// a is not defiend under global scope
a == null // ReferenceError
typeof a // 'undefined'
```

# 扩展运算符(…)

spread 运算符适用于对象！

```
const point2D = {x: 1, y: 2};
const point3D = {...point2D, z: 3};let obj = {a: 'b', c: 'd', e: 'f'};
let {a, ...other} = obj;
console.log(a); //b
console.log(other); //{c: "d", e: "f"}
```

# 参考

*   [说 Javascript](https://www.amazon.com/Speaking-JavaScript-Depth-Guide-Programmers/dp/1449365035/ref=sr_1_1?s=books&ie=UTF8&qid=1521248539&sr=1-1&keywords=speaking+JavaScript)
*   [MDN](https://developer.mozilla.org/en-US/)

# 通知；注意

*   如果你想了解最新的新闻/文章，请点击[【观察】](https://github.com/n0ruSh/the-art-of-reading)订阅。