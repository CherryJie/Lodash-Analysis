# Lodash 源码阅读 —— chunk

标签（空格分隔）： 源码阅读

---

# Lodash 源码阅读 —— chunk

### 这里我先给出源码：
```js
 /**
 * Creates an array of elements split into groups the length of `size`.
 * If `array` can't be split evenly, the final chunk will be the remaining
 * elements.
 *
 * @static
 * @memberOf _
 * @since 3.0.0
 * @category Array
 * @param {Array} array The array to process.
 * @param {number} [size=1] The length of each chunk
 * @param- {Object} [guard] Enables use as an iteratee for methods like `_.map`.
 * @returns {Array} Returns the new array of chunks.
 * @example
 *
 * _.chunk(['a', 'b', 'c', 'd'], 2);
 * // => [['a', 'b'], ['c', 'd']]
 *
 * _.chunk(['a', 'b', 'c', 'd'], 3);
 * // => [['a', 'b', 'c'], ['d']]
 */
 
function chunk(array, size, guard) {
  if ((guard ? isIterateeCall(array, size, guard) : size === undefined)) {
    size = 1;
  } else {
    size = nativeMax(toInteger(size), 0);
  }
  var length = array == null ? 0 : array.length;
  if (!length || size < 1) {
    return [];
  }
  var index = 0,
      resIndex = 0,
      result = Array(nativeCeil(length / size));

  while (index < length) {
    result[resIndex++] = baseSlice(array, index, (index += size));
  }
  return result;
}

```

### 下面是分片段来解读：

```js
if ((guard ? isIterateeCall(array, size, guard) : size === undefined)) {
    size = 1;
} else {
    size = nativeMax(toInteger(size), 0);
}
```
```js
@param- {Object} [guard] Enables use as an iteratee for methods like `_.map`.
```
* 这一段代码的意思是说，如果传入 guard，判断 `isIterateeCall(array, size, guard)`，如果为 true，size = 1，如果为 false，则取 size 和 0 中的较大值。如果没有传入 guard，判断 size 是否为 undefined，如果是，size = 1，如果不是，取 size 和 0 中的较大值。
* 划重点！！！ 我并不知道 guard 怎么用 TAT

```js
var length = array == null ? 0 : array.length;
// 判断 array 是不是为空，如果是，length = 0；如果不是， length = array.length；
if (!length || size < 1) {
    return [];
}
// 这里判断如果 length = 0 或者 传入的 size 为负数或者0；就返回空数组。
```

* 下面是核心的代码：
```js
var index = 0,
  resIndex = 0,
  result = Array(nativeCeil(length / size));
  // 这里的 nativeCeil，就是 Math.ceil(); 
  while (index < length) {
    result[resIndex++] = baseSlice(array, index, (index += size));
  }
  return result;
}
```

* Math.ceil()：该函数并不是四舍五入，而是向上取整，即 `Math.ceil(1.333) = 2`
* baseSlice(): 由原生的 `Array.prototype.slice()`而来。
```js
[1, 2, 3, 4, 5, 6].slice(0, 2); // [1, 2]
// array.slice(begin, end); 包含 begin，但是不包含 end
```
```js
  while (index < length) {
    result[resIndex++] = baseSlice(array, index, (index += size));
  }
  return result;
  
 // _.chunk(['a', 'b', 'c', 'd'], 3);
 // => [['a', 'b', 'c'], ['d']]
 // 用上面这个举例来讲：
 // result = new Array(Math.ceil(4/3));
 // result = [undefined, undefined]
 // 第一轮循环：
 // index = 0; length = 4; size = 3; resIndex = 0; baseSlice(array, 0, 3);
 // result = [['a', 'b', 'c'], undefined];
 // 第二轮循环：
 // index = 3; length = 4; size = 3; resIndex = 1; baseSlice(array, 3, 6);
 // result = [['a', 'b', 'c'], ['d']] 
 // index = 6, 大于 length， 进不去第三轮的循环，到此结束。
```
