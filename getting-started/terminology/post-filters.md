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

#### **$\_Math - The `Math` Global Object**

All methods and properties of the `Math` [global object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math) are available under this namespace. An example usage may be:

```text
$_Math.sin(this['x']) < $_Math.cos($_Math.PI) * this.y + z

//this['x']: Bracket notation for accessing property 'x' of 'this'.
//this.y: Dot notation for accessing property 'y' of 'this'.
//z: Implicit notation for accessing property 'z' of 'this'.
```

#### $\_ - The `lodash` utility module

All methods and properties of the `lodash` [utility module](https://lodash.com/docs/) are available under this namespace. In addition to being referred to by the `$_` namespace handle, this is also the default namespace under which methods are searched when they are invoked as standalone functions, without a property dereference. For example:

```text
1. $_.difference(this.x, y).length < 2
2. some(x, partialRight($_.lte, 2))

//partialRight is used in place of an anonymous function
// defintion, since such function defs are not supported by
// jsep.
```

Note that the default namespace is searched **only when a function call is made**, and not when just referencing the function as a member. In that case, the member will be searched for under the `this` context of the current object under iteration of the result array.

#### **$\_RG - The** _**RecallGraph**_ **Namespace for Special Functions**

This namespace is reserved for some special functions defined by RecallGraph. The supported methods are:

{% tabs %}
{% tab title="typeof\(<operand>\)" %}
This method returns the type of the parameter passed to it.  Its behavior is identical to the [JavaScript `typeof` operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof). For example:

```text
1. $_RG.typeof(x)
2. $_RG.typeof(this.y)
3. $_RG.typeof(this[z])

//this[z] (without quotes) is shorthand for this[this.z],
// i.e. an indirect property reference.
```
{% endtab %}

{% tab title="glob\(str, pattern\)" %}
This method performs a [glob match](./#glob-pattern) test on `str` against the provided `pattern`. For example:

```text
$_RG.glob('aaabbbccc', '*b*')

//jsep supports string literals.
```
{% endtab %}

{% tab title="regex\(str, pattern\)" %}
This method performs a [regex match](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/test) test on `str` against the provided `pattern`. For example:

```text
$_RG.regex('aaabbbccc', 'a+b{3}c*')

//jsep supports string literals.
```
{% endtab %}
{% endtabs %}

For more examples, take a look at the test cases defined in [`helpers.test.js`](https://github.com/RecallGraph/RecallGraph/blob/b1d01aaf73bb05037ac68718ab8c661e134ee980/test/unit/lib/operations/helpers.test.js#L31).

