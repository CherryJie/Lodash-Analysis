# lodash源码阅读 —— difference

标签（空格分隔）： 源码阅读

---

### lodash 源码阅读 —— difference
* 该方法是用于过滤，第一个参数是需要过滤的数组，第二个参数是需要被过滤掉（移除）的元素集合，该方法返回的传入的第一个参数的子集或者自身。
* 用法：_.difference(array, [values]);
* [lodash源码仓库](https://github.com/lodash/lodash/blob/master/difference.js)

* 简单分析：如果写为 `_.difference(A, B)` 就相当于求 A-B，即 A 和 B 的差集，也叫做相对补集，就是属于 A 但是不属于 B 的元素。
1. 下面是源码，然后我将依次贴出内部的工具方法
    ```js
    /**
     * Creates an array of `array` values not included in the other given arrays
     * using [`SameValueZero`](http://ecma-international.org/ecma-262/7.0/#sec-samevaluezero)
     * for equality comparisons. The order and references of result values are
     * determined by the first array.
     *
     * **Note:** Unlike `pullAll`, this method returns a new array.
     *
     * @since 0.1.0
     * @category Array
     * @param {Array} array The array to inspect.
     * @param {...Array} [values] The values to exclude.
     * @returns {Array} Returns the new array of filtered values.
     * @see union, unionBy, unionWith, without, xor, xorBy, xorWith,
     * @example
     *
     * difference([2, 1], [2, 3])
     * // => [1]
     */
    function difference(array, ...values) {
      return isArrayLikeObject(array)
        ? baseDifference(array, baseFlatten(values, 1, isArrayLikeObject, true))
        : []
    }
    
    export default difference
    ```

2. `isArrayLikeObject` 方法
    ```js
    /**
     * This method is like `isArrayLike` except that it also checks if `value`
     * is an object.
     *
     * @since 4.0.0
     * @category Lang
     * @param {*} value The value to check.
     * @returns {boolean} Returns `true` if `value` is an array-like object,
     *  else `false`.
     * @example
     *
     * isArrayLikeObject([1, 2, 3])
     * // => true
     *
     * isArrayLikeObject(document.body.children)
     * // => true
     *
     * isArrayLikeObject('abc')
     * // => false
     *
     * isArrayLikeObject(Function)
     * // => false
     */
    function isArrayLikeObject(value) {
      return isObjectLike(value) && isArrayLike(value)
    }
   
    export default isArrayLikeObject
    ```
    *  该方法用于判断传入的值是否是一个数组或者类数组结构，如果是就返回 true，那这里的 `isObjectLike()` 和 `isArrayLike()` 是怎么实现的呢，下面我将贴出源码。

3. `isObjectLike()` 和 `isArrayLike()`
    ```js
    /**
     * Checks if `value` is object-like. A value is object-like if it's not `null`
     * and has a `typeof` result of "object".
     *
     * @since 4.0.0
     * @category Lang
     * @param {*} value The value to check.
     * @returns {boolean} Returns `true` if `value` is object-like, else `false`.
     * @example
     *
     * isObjectLike({})
     * // => true
     *
     * isObjectLike([1, 2, 3])
     * // => true
     *
     * isObjectLike(Function)
     * // => false
     *
     * isObjectLike(null)
     * // => false
     */
    function isObjectLike(value) {
        // 值不为空，并且 typeof 返回值是 object
      return typeof value == 'object' && value !== null
    }
    
    export default isObjectLike
    
    /**
     * Checks if `value` is array-like. A value is considered array-like if it's
     * not a function and has a `value.length` that's an integer greater than or
     * equal to `0` and less than or equal to `Number.MAX_SAFE_INTEGER`.
     *
     * @since 4.0.0
     * @category Lang
     * @param {*} value The value to check.
     * @returns {boolean} Returns `true` if `value` is array-like, else `false`.
     * @example
     *
     * isArrayLike([1, 2, 3])
     * // => true
     *
     * isArrayLike(document.body.children)
     * // => true
     *
     * isArrayLike('abc')
     * // => true
     *
     * isArrayLike(Function)
     * // => false
     */
    function isArrayLike(value) {
      return value != null && typeof value != 'function' && isLength(value.length)
    }
    
    export default isArrayLike
    ```
    * 这里的 `isObjectLike` 判断传入的参数是不是一个对象或者类对象，`isArrayLike` 判断传入的参数是不是一个数组或者类数组，然后你会发现又跑出来一个 `isLength` 这个函数很简单，有兴趣的可以去翻下源码，就是为了判断长度是不是一个有限的数字。
    
4. `baseDifference`： 重头戏部分
    ```js
    /** Used as the size to enable large array optimizations. */
        const LARGE_ARRAY_SIZE = 200
    /**
     * The base implementation of methods like `difference` without support
     * for excluding multiple arrays.
     *
     * @private
     * @param {Array} array The array to inspect.
     * @param {Array} values The values to exclude.
     * @param {Function} [iteratee] The iteratee invoked per element.
     * @param {Function} [comparator] The comparator invoked per element.
     * @returns {Array} Returns the new array of filtered values.
     */
    function baseDifference(array, values, iteratee, comparator) {
      let includes = arrayIncludes
      let isCommon = true
      const result = []
      const valuesLength = values.length
    
      // 判断如果数组的长度是 0，则返回空数组
      if (!array.length) {
        return result
      }
      // 如果传入 iteratee 函数，则遍历处理 values 的每一个值。
      if (iteratee) {
        values = map(values, (value) => iteratee(value))
      }
      if (comparator) {
        includes = arrayIncludesWith
        isCommon = false
      }
      // 当数组的长度大于 200 时执行，这是前面给出的常量。
      else if (values.length >= LARGE_ARRAY_SIZE) {
        // 这里的 cacheHas 是一个函数，有兴趣的可以去查源码，这里就不多余讲了。
        includes = cacheHas
        isCommon = false
        values = new SetCache(values)
      }
      // 标签语法
      outer:
      for (let value of array) {
        // 当 iteratee == null 时返回 value，否则返回 iteratee(value)
        const computed = iteratee == null ? value : iteratee(value)
        // 当 comparator 存在或者 value ！==0 时，value = value
        value = (comparator || value !== 0) ? value : 0
        if (isCommon && computed === computed) {
            // values 的长度
          let valuesIndex = valuesLength 
          while (valuesIndex--) {
            if (values[valuesIndex] === computed) {
             // 如果 values 中有值等于 computed，就退出到最外层的循环，这就是标签语法的作用。
              continue outer
            }
          }
          result.push(value)
        }
        else if (!includes(values, computed, comparator)) {
          result.push(value)
        }
      }
      return result
    }
    
    export default baseDifference
    
    // 下面来举个例子讲一下 baseDifference([1, 2], [2, 3, 4]) 这里 array = [1, 2] , values = [2, 3, 4]
    // 初始值 result = [], valuesLength = 3
    // for 遍历，第一个值 value = 1， computed = 1, valuesIndex = 3;
    // values[2] = 4 !== value, values[1] = 3 !== value, values[0] = 2 !== value
    // result = [1]
    // 接下来，遍历第二个值， value = 2, computed = 2, vlauesIndex = 3;
    // values[2] = 4 !== value, values[1] = 3 !== value, values[0] = 2 == value
    // 当存在值相等的时候，直接跳转到最外层，所以 result = [1]
    // baseDifference([1, 2], [2, 3, 4]) = [1];
    ```
    * 看到 `outer:`是不是一下子懵了，这就是标签语法，形式是 `label: statement`，这里的标签可以是除了保留字以外的任意标识符，当然语句也可以是任意语句。它有什么用呢？这个标签就相当于一个定位符，在 `break, continue` 后面用于指定跳转的位置。

5. `baseFlatten`：关于这部分的源码可以去看我上一篇文章：[Lodash 源码阅读 —— concat](https://zhuanlan.zhihu.com/p/37917828)

* 本人菜鸟，只是一些拙见，有什么错误请大家帮忙指出。
