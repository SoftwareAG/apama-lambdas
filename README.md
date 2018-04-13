# Lambdas
## Contents
* [Installation](#install)
* [Introduction to Lambdas](#intro) 
* [When to Use Lambdas?](#usage)
* [Different Types of Lambdas](#lambda-types) 
* [Language Features](#language) 

## <a id="install"></a>Installation
### 1. Installing files
Copy the folders contained inside `CopyContentsToApamaInstallDir` into the Apama install directory (Usually `C:\SoftwareAG\Apama` on Windows or `/opt/softwareag/apama` on Unix), merging them with what is already there.
### 2. Adding to Designer
1. From designer right click on your project in `Project Explorer`
2. Select `Apama` from the drop down menu;
3. Select `Add Bundle`
4. Scroll down to `Standard bundles` and select `Lambdas`
5. Click `Ok`

## <a id="intro"></a>Introduction to Lambdas
Lambdas in EPL are closely based on Arrow Functions in JavaScript. They are inline actions that manipulate one or more provided values, implicitly returning the result.
```javascript
action<any> returns any multiplyBy10:= Lambda.function1("x => x * 10");

multiplyBy10(1.5) = <any> 15.0;
```
## <a id="usage"></a>When to Use Lambdas?
Ever find yourself writing this:
```javascript
aFunctionThatTakesACallback(callback);
... 
// Much further down
...
action callback(float arg1, float arg2) {
	// A simple callback
	return arg1 + arg2 + 30.0;
}
```
You can simplify the code by doing this:
```javascript
aFunctionThatTakesACallback(Lambda.function2("arg1, arg2 => arg1 + arg2 + 30"));
```
This is particularly useful when used with functional programming
```javascript
Observable.fromValues([1,2,3,4,5])
	.filter(Lambda.predicate("x => x % 2 = 0"))
	.map(Lambda.function1("x => x * 30"))
	.reduce(Lamda.function2("sum, x => sum + x"))
	.subscribe(Subscriber.create().onNext(printValue));
```

## <a id="lambda-types"></a>Different Type of Lambdas
**Functions** - Take a varying number of arguments and return a value  
```javascript
action<> returns any ex0 := Lambda.function0("10 + 5 / 2 + ' = 12.5'");
action<any> returns any ex1 := Lambda.function1("x => x * 10");
action<any, any> returns any ex2 := Lambda.function2("x, y => x + y");
action<sequence<any> > returns any ex3 := Lambda.function("x, y, z => x + y / z");
```
**Predicate** - Take a single argument and returns a boolean  
```javascript
action<any> returns boolean ex := Lambda.predicate("x => x >= 10 and x < 20");
```
**Call (Awaiting Apama 10.2)** - Run a lambda just for side-effects (Not yet possible)
```javascript
action<any> returns boolean ex := Lambda.call("x => x.doSomething(10 + 3)");
```
## <a id="language"></a>Language Features
Lambdas attempt to provide a language very close to EPL but without the need for casting. You will never have to write  `<sequence<float, boolean> >` within a lambda!

**Numbers**
All numbers are treated the same and can be operated on without conversion. 
```javascript
Lambda.function1("x => x + 10 + 11.1 + 22.2d")(1.1) = <any> 44.4
```
This follows the rules of [coercion](#coercion).

**Strings**
Strings can be defined with either `\"` or `'`. Generally `'` is easier because there is no need for the backslash escape character.
```javascript
Lambda.function1("x => \"hello\" + 'world'")
```

**Brackets**
The usual bracketing syntax applies
```javascript
Lambda.function1("x => (x + 5) / 12")
```

**Numeric Operators**
All of the usual epl operators (`%`, `/`, `*`, `-`, `+`).
_Note: Type [coercion](#coercion) may happen._

**Logical Operators**
All of the usual epl operators (`=`, `!=`, `>=`, `>`, `<=`, `<`, `and`, `xor`, `or`,`not`). 
_Note: Type [coercion](#coercion) may happen._

Some non-epl operators:
`!` - Same as `not`

**Ternary Operator**
```javascript
Lambda.function1("x => x > 10 ? 'Big' : 'Small'")
```

**Field, Dictionary, and Sequence Access**
```javascript
Lambda.function1("event0 => event0.fieldName")
// or
Lambda.function1("event0 => event0['fieldName']")
```
```javascript
Lambda.function1("dict => dict[key]")
// or (If the dictionary has string keys)
Lambda.function1("dict => dict.key")
```
```javascript
Lambda.function1("seq0 => seq0[0]")
```
**Special Values**
`currentTime` - The same as in EPL - Gets the current time in seconds (usually rounded to nearest 100ms) 

**Action Calling**
Calling actions in a generic way is not possible in Apama 10.1, so only a handful of particularly useful actions are supported. This will improve in 10.2.

|Action          |Description                                                   |
|---------------:|--------------------------------------------------------------|
|    `.toFloat()`|Call on any numeric type to convert to float                  |
|  `.toDecimal()`|Call on any numeric type to convert to decimal                |
|      `.round()`|Call on any numeric type to round to an integer               |
|       `.ceil()`|Call on any numeric type to round upwards to an integer       |
|      `.floor()`|Call on any numeric type to round downwards to an integer     |
|        `.abs()`|Call on any numeric type to provide the non-negative value    |
|       `.pow(n)`|Call on any numeric type to raise it to the `n`^th^ power     |
|       `.sqrt()`|Call on any numeric type to square root the value             |
|   `.toString()`|Call on any type to convert it to a string (has no effect on strings)|

**Sequence Literals**
Sequences can be constructed in lambdas in much the same way that they can in EPL, except that they are always `sequence<any>`.
```javascript
Lambda.function1("x => [x, x + 1, x + 2]")(0) = [<any>0, 1, 2]
```
**Spread Operator**
```javascript
Lambda.function1("x => [...x, ...x])([0, 1, 2]) = [<any>0, 1, 2, 0, 1, 2]
```
**Array Destructuring**
When using lambdas (particularly with Observables) you may find that a lambda is provided with a sequence as the argument. Rather than accessing each value using `seq[index]` it is easier to assign a name to each item in the sequence:
```javascript
Lambda.function1("[sum, count] => sum / count")([56, 7]) = <any> 8.0
```
<a id="coercion"></a>**Type Coercion**
Numbers and sometimes values are coerced to sensible types, using the following rules:
* Operations on two values of the same type _mostly_ result in the same type
* Operations on `integers` which may result in fractions result in `floats`
* Operations involving `integer` and `decimal` result in `decimal`
* Operations involving `integer` and `float` result in `float`
* Operations involving `float` and `decimal` result in `float`
* Operations on any type and `string` will call `.valueToString` on the any type

If you need the result to be a particular type (and that isn't possible through explicit typing eg. `22.0d`) then use the `.toFloat()`, `.toDecimal()`, `.round()`, `ceil()`, `.floor()`, `.toString()` actions.