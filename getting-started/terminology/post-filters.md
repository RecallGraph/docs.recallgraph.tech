---
description: 'Filters applied on a query result AFTER grouping, sorting and slicing.'
---

# Post-Filters

Once a query returns some results, a post-filter can be applied on them to further restrict the number of matching results that are returned. These filters are executed within service code, in a V8 context.

The only type of post-filter supported by _RecallGraph_ is a string containing any JavaScript-like expression that is supported by [jsep](http://jsep.from.so/), with a few extensions defined below.

**Important:** _jsep_ does not support object literals, only array literals. This may be fixed in a fork, maintained by _RecallGraph_, in the future.

## _jsep_ Extensions

### **Operators**

1. `=~` - **Regex Match**. Filters on a regex pattern. The left operand must [match the regex pattern](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/test) given in the right operand for the filter to pass.
2. `=*` - **Glob Match**. Filters on a [glob pattern](./#glob-pattern). The left operand must match the glob pattern given in the right operand for the filter to pass.
3. `in` - **Array Search**. Filters on an array search. The left operand must be present in the array provided in the right operand. **Deep comparison** is performed for each element of the right operand against the left operand.
4. `**` - **Exponentiation**. Returns the left operand raised to the power right operand.

#### **Special Handling for ==, ===, != and !==**

1. `==, ===` - **Deep Equality**. Filters on deep equality. The left operand must be **deeply equal** to the right operand. Both `==` and `===` behave identically.
2. `!=, !==` - **Deep Inequality**. Filters on deep inequality. The left operand must **NOT** be **deeply equal** to the right operand. Both `!=` and `!==` behave identically.

### **Functions**

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

### **Special Handling for Math.\* - EXPERIMENTAL!**

All exposed properties and methods of the built-in [Math](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math) object are correctly recognized and processed.

A consequent **CAVEAT** is that in the intermediate result array \(the DB query results\), any object with field/s \(top-level or nested\) having the name `Math` will **NOT** be processed normally. This behaviour is experimental, and subject to change in future versions.

