## 对象的浅拷贝

如果是数组，我们可以利用数组的一些方法比如：`slice`、`concat` 返回一个新数组的特性来实现拷贝。

比如：
```js
var arr = ['old', 1, true, null, undefined];
var new_arr = arr.concat();
new_arr[0] = 'new';
console.log(arr) // ["old", 1, true, null, undefined]
console.log(new_arr) // ["new", 1, true, null, undefined]
```
用 `slice` 可以这样做：

`var new_arr = arr.slice();`

但是如果数组嵌套了对象或者数组的话，比如：
```js
var arr = [{old: 'old'}, ['old']];

var new_arr = arr.concat();

arr[0].old = 'new';
arr[1][0] = 'new';

console.log(arr) // [{old: 'new'}, ['new']]
console.log(new_arr) // [{old: 'new'}, ['new']]
```
我们会发现，无论是新数组还是旧数组都发生了变化，也就是说使用 concat 方法，克隆的并不彻底。
如果数组元素是基本类型，就会拷贝一份，互不影响，而如果是对象或者数组，就会只拷贝对象和数组的引用，这样我们无论在新旧数组进行了修改，两者都会发生变化。
我们把这种复制引用的拷贝方法称之为浅拷贝，与之对应的就是深拷贝，深拷贝就是指完全的拷贝一个对象，即使嵌套了对象，两者也相互分离，修改一个对象的属性，也不会影响另一个。
所以我们可以看出使用 `concat` 和 `slice` 是一种浅拷贝。

另外对象 `Object.assign({})` ，扩展运算符也是浅拷贝

### 浅拷贝实现

```js
var shallowCopy = function(obj) {
    // 只拷贝对象
    if (typeof obj !== 'object') return;
    // 根据obj的类型判断是新建一个数组还是对象
    var newObj = obj instanceof Array ? [] : {};
    // 遍历obj，并且判断是obj的属性才拷贝
    for (var key in obj) {
        if (obj.hasOwnProperty(key)) {
            newObj[key] = obj[key];
        }
    }
    return newObj;
}
```

### 正则对象拷贝

```js

function isRegExp (pattern) {
  var flags = ''
  flags += pattern.global ? 'g' : ''
  flags += pattern.ignoreCase ? 'i' : ''
  flags += pattern.multiline ? 'm' : ''
  return new RegExp(pattern.source, flags)
}

var reg=new RegExp('/111/');
var newReg=isRegExp(reg);
```
### 其他对象拷贝

```js
function getRawType(value) {
    return Object.prototype.toString.call(value).slice(8, -1)
}
 let type = getRawType(value)
    
    switch(type){
        case 'Date':
        case 'RegExp': 
        case 'Error': value = new window[type](value); break;
    }
    return value
```

### 函数拷贝
复制函数似乎没有太大意义，`JS` 里函数的身份比较特殊，由于闭包的关系，每个函数在声明时处于独立的执行环境。所以你简单复制一个函数，比如 `funcA = new Function(funcB.toString)`，无法进入原先的执行环境，运行结果肯定是不同的。
```js
var fn=function(){}
`eval('(' + fn.toString() + ')');`
```

## 深拷贝的实现

第一种就是 `JSON.stringify` 这个对于，函数就不适用了。

那如何实现一个深拷贝呢？说起来也好简单，我们在拷贝的时候判断一下属性值的类型，如果是对象，我们递归调用深拷贝函数不就好了~
```js
var deepCopy = function(obj) {
    if (typeof obj !== 'object') return;
    var newObj = obj instanceof Array ? [] : {};
    for (var key in obj) {
        if (obj.hasOwnProperty(key)) {
            newObj[key] = typeof obj[key] === 'object' ? deepCopy(obj[key]) : obj[key];
        }
    }
    return newObj;
}
```

## 循环引用拷贝

循环引用。

例如：
```js
var man = {
    name: 'aepkill',
    sex: 'male'
};
man['deefRef'] = man;

```

此时如果调用刚才的`deepCopy`函数的话，会陷入一个循环的递归过程，从而导致爆栈：


解决这个问题也非常简单，只需要判断一个对象的字段是否引用了这个对象或这个对象的任意父级即可，修改一下代码：

```js
var deepCopy = function(obj,parent) {
    if (typeof obj !== 'object') return;
    var newObj = obj instanceof Array ? [] : {};
	var _parent = parent;
	   _parent = parent;
            // 该字段有父级则需要追溯该字段的父级
            while (_parent) {
                // 如果该字段引用了它的父级则为循环引用
                if (_parent.originalParent === obj) {
                    // 循环引用直接返回同级的新对象
                    return _parent.currentParent;
                }
                _parent = _parent.parent;
            }
    for (var key in obj) {
        if (obj.hasOwnProperty(key)) {
            newObj[key] = typeof obj[key] === 'object' ? deepCopy(obj[key], {
                        originalParent: obj,
                        currentParent: newObj,
                        parent: _parent
                    }) : obj[key];
        }
    }
    return newObj;
}
```

## 性能问题
尽管使用深拷贝会完全的克隆一个新对象，不会产生副作用，但是深拷贝因为使用递归，性能会不如浅拷贝，在开发中，还是要根据实际情况进行选择。

## 参考

- https://github.com/mqyqingfeng/Blog/issues/32
- https://blog.csdn.net/liwusen/article/details/78759373
- https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp
- https://segmentfault.com/q/1010000019806573?_ea=15209539

## 今日图 - 灵魂一米八
![16b4040ec7c834f6.gif](../../images/16b4040ec7c834f6.gif)
