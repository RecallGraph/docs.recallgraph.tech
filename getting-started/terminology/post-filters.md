---
description: 'Filters applied on a query result AFTER grouping, sorting and slicing.'
---

# Post-Filters

Once a query returns some results, a post-filter can be applied on them to further restrict the number of matching results that are returned.

Post-filter expressions are processed in the V8 engine and **NOT the query engine**, using a [jsep](http://jsep.from.so/) expression parser that has been augmented with a few extensions.

The filter expression is applied individually to each item in the result array \(these items may be entire groups\). Only those items for which the expression evaluates to a _truthy_ value are passed through.

**Important:** _jsep_ does not support object literals, only array literals. This may be fixed in a fork, maintained by _RecallGraph_, in the future.

## Core JS Operators

The following \(non-mutating\) standard operators are understood by _jsep_ out of the box, and are processed exactly as they would be processed in a normal JavaScript runtime:

* [All comparison operators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators#Comparison)
* [All **non-mutating** arithmetic operators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators#Arithmetic) \(everything except increment and decrement\)
* [All bitwise operators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators#Bitwise)
* [All logical operators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators#Logical)
* [String concatenation](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators#String)
* [The ternary conditional operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators#Conditional)
* [The `in` relational operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators#in)
* [The `this` keyword](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators#this)
* [The grouping operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators#Grouping_operator)
* Property accessors \(including method calls\)
  * [Explicit](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Property_Accessors) \(dot and bracket notation; using `this` keyword to refer to current item in result array\)
  * Implicit \(using implicit `this` context per item in a result array; **method calls on the `this` object not accessible with this style**\)
* Literals
  * [All primitive types](https://developer.mozilla.org/en-US/docs/Glossary/Primitive) \(except `Symbol`\)
  * [Array expressions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array) \(both literal and dynamic\)

## _jsep_ Extensions

Some of the core JS operators listed above are actually implemented using extensions in _jsep_. Additionally, some useful operators that are **not part of JS syntax** are also supported for convenience.

### **Operators**

1. `=~` - **Regex Match**. Filters on a regex pattern. The left operand must [match the regex pattern](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/test) given in the right operand for the filter to pass.
2. `=*` - **Glob Match**. Filters on a [glob pattern](./#glob-pattern). The left operand must match the glob pattern given in the right operand for the filter to pass.

### **Namespaced Helpers**

Apart from the regular operators and member functions that the built-in data types support, it is often handy to be able to invoke `Math` functions or methods from the `lodash` utility. _jsep_ has been extended to provide access to these functions and some more, using named object literals that represent these modules. The available namespaces are listed below, along with special shortcuts, if any, to access them.

#### **Binary Functions**

1. `eq(left, right)` - **Deep Equality**. Filters on deep equality. See [https://lodash.com/docs/4.17.15\#isEqual](https://lodash.com/docs/4.17.15#isEqual).
2. `lt(left, right)` - **'Less Than' Comparison**. Filters on strict inequality. The `left` operand must be strictly less than the `right` operand. See [https://lodash.com/docs/4.17.15\#lt](https://lodash.com/docs/4.17.15#lt).
3. `gt(left, right)` - **'Greater Than' Comparison**. Filters on strict inequality. The `left` operand must be strictly greater than the `right` operand. See [https://lodash.com/docs/4.17.15\#gt](https://lodash.com/docs/4.17.15#gt).
4. `lte(left, right)` - **'Less Than or Equals' Comparison**. Filters on non-strict inequality. The `left` operand must be less than or equal to the `right` operand. See [https://lodash.com/docs/4.17.15\#lte](https://lodash.com/docs/4.17.15#lte).
5. `gte(left, right)` - **'Greater Than or Equals' Comparison**. Filters on non-strict inequality. The `left` operand must be greater than or equal to the `right` operand. See [https://lodash.com/docs/4.17.15\#gte](https://lodash.com/docs/4.17.15#gte).
6. `typeof(val)` - **Type Identifier**. Returns the type of `val` as per the specification defined at [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof).
7. `in(needle, haystack)` - **Array Search**. Filters on an array search. The `needle` operand must be present in the array provided in the `haystack` operand. **Deep comparison** is performed for each element of `haystack` against `needle`.
8. `glob(string, pattern)` - **Glob Match**. Filters on a [glob pattern](). The `string` operand must match the glob pattern given in the `pattern` operand for the filter to pass.
9. `regx(string, pattern)` - **Regex Match**. Filters on a regex pattern. The `string` operand must [match the regex pattern](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/test) given in the `pattern` operand for the filter to pass.

#### **Ternary Functions**

1. `all(op, array, value)` - **Array 'All Match'**. Filters on an array 'all match'. Every element `el` in `array` must satisfy the `op(el, value)` function filter, where `op` is one of the above [binary function filters]().
2. `any(op, array, value)` - **Array 'Any Match'**. Filters on an array 'any match'. At least one element `el` in `array` must satisfy the `op(el, value)` function filter, where `op` is one of the above [binary function filters]().

