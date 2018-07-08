# Lodash 源码阅读 —— concat

标签（空格分隔）： 源码阅读

---

### Lodash 源码阅读 —— concat

* 该方法的作用是：创建一个新的数组，将 array 与任何数组或值连接在一起。

### 这里我先给出源码：
```js
/**
 * Creates a new array concatenating `array` with any additional arrays
 * and/or values.
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Array
 * @param {Array} array The array to concatenate.
 * @param {...*} [values] The values to concatenate.
 * @returns {Array} Returns the new concatenated array.
 * @example
 *
 * var array = [1];
 * var other = _.concat(array, 2, [3], [[4]]);
 *
 * console.log(other);
 * // => [1, 2, 3, [4]]
 *
 * console.log(array);
 * // => [1]
 */
function concat() {
  var length = arguments.length;
  if (!length) {
    return [];
  }
  var args = Array(length - 1),
      array = arguments[0],
      index = length;

  while (index--) {
    args[index - 1] = arguments[index];
  }
  return arrayPush(isArray(array) ? copyArray(array) : [array], baseFlatten(args, 1));
}
```
### 这里分片段来分析，下面我会把相关工具方法的源码会贴出来：
* 这里是 `arrayPush` 的源码：
```js
/**
* Appends the elements of `values` to `array`.
*
* @private
* @param {Array} array The array to modify.
* @param {Array} values The values to append.
* @returns {Array} Returns `array`.
*/
function arrayPush(array, values) {
    var index = -1,
        length = values.length,
        offset = array.length;
    
    while (++index < length) {
      array[offset + index] = values[index];
    }
    return array;
}
// 这里举例来进行分析：
// arrayPush([1, 2, 3], [4, [5]]);
// 那么这里 index = -1, length = 2, offset = 3,
// 进入循环
// 第一轮： index = 0; offset + index = 3; array[3] = values[0] = 4;
// 第二轮： index = 1; offset + index = 4; array[4] = values[1] = [5];
// index = 2; 大于 length 不成立，故进入不了第三轮循环。
// result = [1, 2, 3, 4, [5]]
```
* `arrayPush`方法是在传入的参数`array`基础上进行拼接的。

* 这里是 `copyArray` 的源码：
```js
/**
 * Copies the values of `source` to `array`.
 *
 * @private
 * @param {Array} source The array to copy values from.
 * @param {Array} [array=[]] The array to copy values to.
 * @returns {Array} Returns `array`.
 */
function copyArray(source, array) {
  var index = -1,
      length = source.length;

  array || (array = Array(length));
  // 这里判断 array 是否为 undefined，如果是就定义一个与 source 长度相等的数组。
  while (++index < length) {
    array[index] = source[index];
  }
  return array;
}
```
* 这里是数组的一个深拷贝，拷贝的数组存放在 `array` 中。

* 这里是 `baseFlatten` 的源码：
```js

/**
 * Checks if `value` is a flattenable `arguments` object or array.
 *
 * @private
 * @param {*} value The value to check.
 * @returns {boolean} Returns `true` if `value` is flattenable, else `false`.
 */
function isFlattenable(value) {
  return isArray(value) || isArguments(value) ||
    !!(spreadableSymbol && value && value[spreadableSymbol]);
}
// 这个方法是在判断 value 是否是可以被打平的 arguments 对象或者数组。如果是，返回 true，若不是，则返回 false
/**
 * The base implementation of `_.flatten` with support for restricting flattening.
 *
 * @private
 * @param {Array} array The array to flatten.
 * @param {number} depth The maximum recursion depth.
 * @param {boolean} [predicate=isFlattenable] The function invoked per iteration.
 * @param {boolean} [isStrict] Restrict to values that pass `predicate` checks.
 * @param {Array} [result=[]] The initial result value.
 * @returns {Array} Returns the new flattened array.
 */
function baseFlatten(array, depth, predicate, isStrict, result) {
  var index = -1,
      length = array.length;

  predicate || (predicate = isFlattenable);
  // 判断是否传入 predicate, 若没有，将方法 isFlattenable 赋值给 predicate
  result || (result = []);
  // 判断是否传入 result, 若没有，将空数组赋值给 result

  while (++index < length) {
    var value = array[index];
    if (depth > 0 && predicate(value)) {
      if (depth > 1) {
        // Recursively flatten arrays (susceptible to call stack limits).
        baseFlatten(value, depth - 1, predicate, isStrict, result);
      } else {
        arrayPush(result, value);
      }
    } else if (!isStrict) {
      result[result.length] = value;
    }
  }
  return result;
}

// baseFlatten([[1,2,3,4],[5, 6, [7]]],1,false);
// 这里的 depth 等于1，也就是说最大的递归深度为 1
// 进入 while 循环后，
// 第一轮： 
// length = 2; index = 0; value = array[0] = [1, 2, 3, 4]; 
// 这里 (depth > 0 && predicate(value)) 为 true，但是 depth = 1 
// 所以执行 arrayPush(result, value)；
// 得到结果：result = [1, 2, 3, 4]
// 第二轮： 
// length = 2; index = 1; value = array[1] = [5, 6, [7]]
// 这里 (depth > 0 && predicate(value)) 为false, 但是 depth = 1；
// 所以执行 arrayPush(result, value);
// 得到结果：result = [1, 2, 3, 4, 5, 6, [7]]
// 到此循环结束，得到结果 result = [1, 2, 3, 4, 5, 6, [7]];
```
* `baseFlatten` 方法是实现 `_.flatten` 的基础方法。这个方法是为了减少数组的嵌套层数。 
* 这里 depth > 1 的情况将会在 `_.flattenDepth` 方法中进行分析。

* 下面是重头戏 `concat` ：
```js
var length = arguments.length;
if(!length) {
    return [];
}
```
* 在函数内部，有两个特殊的对象：`arguments` 和 `this`，`arguments`是一个类数组的对象，包含着传入函数的所有参数。可以使用方括号语法来访问它的每一个元素（即第一个元素是 `arguments[0]`，第二个元素是`arguments[1]`），可以使用`length`属性来确定传递进来多少个参数。

```js
var args = Array(length - 1),
    array = arguments[0],
    index = length;
while (index--) {
    args[index - 1] = arguments[index];
}
return arrayPush(isArray(array) ? copyArray(array) : [array], baseFlatten(args, 1));

// var array = [1];
// var other = _.concat(array, 2, [3], [[4]]);
// length = arguments.length = 4; index = 4;
// args = [undefined, undefined, undefined]; array = [1]; index = 4
// args[2] = arguments[3] = [[4]]; args[1] = arguments[2] = [3]; args[0] = arguments[1] = 2; 
// args = [2, [3], [[4]]]
// baseFlatten(args, 1) 会得到 [2, 3, [4]];
// arrayPush(array, baseFlatten(args, 1)) = [1, 2, 3, [4]];
// other = [1, 2, 3, [4]];
``` 

* 本人菜鸟，只是一些拙见，有什么错误请大家帮忙指出。