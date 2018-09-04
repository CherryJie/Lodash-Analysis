# lodash源码阅读 —— differenceBy

标签（空格分隔）： 源码阅读

---
### lodash 源码阅读 —— differenceBy
* 该方法和 `_.difference`方法的区别就是它可以多接受一个`iteratee`参数，可以称之为迭代器。
* 该方法的用法为 `_.differenceBy(array, values, iteratee)`。

1. 下面将给出`_.differenceBy`的源码
```js
    /**
    * This method is like `difference` except that it accepts `iteratee` which
    * is invoked for each element of `array` and `values` to generate the criterion
    * by which they're compared. The order and references of result values are
    * determined by the first array. The iteratee is invoked with one argument:
    * (value).
    *
    * **Note:** Unlike `pullAllBy`, this method returns a new array.
    *
    * @since 4.0.0
    * @category Array
    * @param {Array} array The array to inspect.
    * @param {...Array} [values] The values to exclude.
    * @param {Function} iteratee The iteratee invoked per element.
    * @returns {Array} Returns the new array of filtered values.
    * @example
    *
    * differenceBy([2.1, 1.2], [2.3, 3.4], Math.floor)
    * // => [1.2]
    */
    function differenceBy(array, ...values) {
    // 这里的 iteratee 取了最后一个值
    let iteratee = last(values)
    if (isArrayLikeObject(iteratee)) {
        iteratee = undefined
    }
    return isArrayLikeObject(array) ? baseDifference(array, baseFlatten(values, 1, isArrayLikeObject, true), iteratee) : []
    }
    
    export default differenceBy
```
    
* 我们可以看出这里核心代码就在`baseDifference`，关于这部分代码在我上一篇文章写`_.difference()`时有很详细的解释，这里需要注意的就是：
```js
    if (iteratee) {
        // 这里就是让每一个 value 都经过 iteratee 的处理，返回的 values 其实是参数经过处理的结果
        values = map(values, (value) => iteratee(value))
    }
    
    outer:
      for (let value of array) {
      // 这里是用 iteratee 处理 array 中的每一个值
        const computed = iteratee == null ? value : iteratee(value)
    
        value = (comparator || value !== 0) ? value : 0
        // 这里 computed === computed 是为了排除 NaN 的情况。
        if (isCommon && computed === computed) {
          let valuesIndex = valuesLength
          while (valuesIndex--) {
            if (values[valuesIndex] === computed) {
              continue outer
            }
          }
          result.push(value)
        }
        else if (!includes(values, computed, comparator)) {
          result.push(value)
        }
      }
```

* 这时会发现文档中给出了一个很奇怪的例子`_.differenceBy([{ 'x': 2 }, { 'x': 1 }], [{ 'x': 1 }], 'x')`，那关于这个 `iteratee`到底是什么呢，下面是它的源码：
```js
    /**
     * The base implementation of `_.iteratee`.
     *
     * @private
     * @param {*} [value=_.identity] The value to convert to an iteratee.
     * @returns {Function} Returns the iteratee.
     */
    function baseIteratee(value) {
      // Don't store the `typeof` result in a variable to avoid a JIT bug in Safari 9.
      // See https://bugs.webkit.org/show_bug.cgi?id=156034 for more details.
      if (typeof value == 'function') {
        return value;
      }
      if (value == null) {
        return identity;
      }
      if (typeof value == 'object') {
        return isArray(value)
          ? baseMatchesProperty(value[0], value[1])
          : baseMatches(value);
      }
      return property(value);
    }
```
    
* 这里可以看出如果传入的 `iteratee` 是不同的类型，会得到不同的结果。如果是 `function` 不用处理直接返回，如果是 `null`, 返回 `identity`, 如果是 `object`, 会去判断是不是数组，在进行判断，如果以上都不是，就会返回 `property(value)`。所以此处就相当于:
```js
_.differenceBy([{ 'x': 2 }, { 'x': 1 }], [{ 'x': 1 }], _.isProperty('x'));
```
