 prevent-undefined
=====================

`prevent-undefined` throws an error to let you know that you accidentally refer
an undefined propery on your object.

It also displays dump of the refered object for your convenience. See below :

```javascript
const  { preventUndefined } = require("prevent-undefined" );
// or
import { preventUndefined } from "prevent-undefined";

// foo.js
  const someObj = {
    hello : {
      world : {
        foo : {
          bar : {
            baz : "HELLO",
          },
        }
      }
    }
  };

  preventUndefined( someObj ).hello.world.foo.bar.VAZ;
  //                                 SOME TYPO    ^^^
```

then, you'll get:

```javascript
> node foo.js
ReferenceError: obj.hello.world.foo.bar.VAZ is not defined in {
  hello: {
    world: { foo: { bar: { baz: 'HELLO' } } }
  }
}
```


 The Major Usecase of `preventUndefined`
--------------------------------------------------------------------------------
The major usecase of `preventUndefined` is checking validity of named parameter
values:

```javascript
function someFunc(args){
  const {foo,bar} = preventUndefined(args);
  console.error("foo:",foo);
}

someFunc({wrongArg1:"foo", wrongArg2:"bar";});

> ReferenceError: obj.foo is not defined in {
>   "wrongArg1": "foo",
>   "wrongArg2": "bar"
> }
```


 unprevent()
--------------------------------------------------------------------------------

Sometimes, you will want to unprevent your objects especially when you want to
use [optional chaining][]:

```javascript
const { preventUndefined } = require('prevent-undefined');
const someObj = {
  hello : {
    world : {
      foo : {
        bar : {
          baz : {
            message : "HELLO",
          }
        },
      }
    }
  }
};
const o = preventUndefined( someObj );

function proc(o){
  return o.hello?.world?.foo?.bar?.baz?.value ?? "hello world";
}
proc(o);
```
you'll get

```javascript
ReferenceError: obj.hello.world.foo.bar.baz.value is not defined in {
  hello: {
    world: {
      foo: { bar: { baz: { message: 'HELLO' } } }
    }
  }
}
    at Object.get (/.../common.js:91:17)
```

which you definitely don't want.

Use `unprevent()`

```javascript
const  { unprevent } = require("prevent-undefined");
function proc(o){
  o=unprevent(o);
  return o.hello?.world?.foo?.bar?.baz?.value ?? "hello world";
}
proc(o); // you'll get "hello world" as expected.
```

[optional chaining]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining


 recursivelyUnprevent()
--------------------------------------------------------------------------------
`recursivelyUnprevent()` recursively unprevent the specified object, literally.
Currently `recursivelyUnprevent()` is in the beta state; therefore, please use
this with care. Currently, this does not check any circular references.

`recursivelyUnprevent()` was **ADDED v0.2.19**.


 preventUnusedProperties()
--------------------------------------------------------------------------------
The basic idea is :
```javascript
// There is a func as :
function someFunc({foo,bar}){
  console.error("foo:",foo);
  console.error("bar:",bar);
  // note that `baz` is not used at all.
}

var foo = 'foo';
var bar = 'bar';
var baz = 'baz';
// `baz` a property which the caller thinks it is valid but actually isn't.
someFunc({foo,bar,baz});

> foo: foo
> bar: bar

// You'll get `foo` `bar` without getting any error that sucks.
```

In order to prevent such tiredness, use `preventUnusedProperties()`.

```javascript
const  { preventUnusedProperties } = require("prevent-undefined");

function someFunc(args){
  // Prevent undefined on the `args` object before destructuring it.
  const {foo,bar} = preventUndefined(args);

  // After destructured the `args` object, call `preventUnusedProperties()`
  preventUnusedProperties(args);

  console.error("foo:",foo);
  console.error("bar:",bar);
}

var foo = 'foo';
var bar = 'bar';
var baz = 'baz';
someFunc({foo,bar,baz});


> ReferenceError: the field [baz] was not referred in
> {
>   "foo": "foo",
>   "bar": "bar",
>   "baz": "baz"
> }
```

`preventUnusedProperties()` has been **added v0.2.20**.


 isUndefinedPrevented()
--------------------------------------------------------------------------------
`isUndefinedPrevented()` function returns true if a specified object is
protected by `preventUndefined()` function.

`isUndefinedPrevented()` has been added on ***v0.2.31***



 maxRecursiveLevel
--------------------------------------------------------------------------------

`preventUndefined()` protects the specified object and all of its properties as
well. As default, it protects all of descendants recursively. You can limit its
maximum recursive level.

```javascript
const obj = {
  foo: {
    bar : {
      baz :{
        data:1
      },
    },
  },
};

test( 'maxRecursiveLevel ... base ', ()=>{
  const po = preventUndefined( obj );
  expect( isUndefinedPrevented( po.foo         )).toBe( true );
  expect( isUndefinedPrevented( po.foo.bar     )).toBe( true );
  expect( isUndefinedPrevented( po.foo.bar.baz )).toBe( true );
});

test( 'maxRecursiveLevel ...0 ', ()=>{
  const po = preventUndefined( obj, {maxRecursiveLevel:0 }  );
  expect( isUndefinedPrevented( po             )).toBe( false );
  expect( isUndefinedPrevented( po.foo         )).toBe( false );
  expect( isUndefinedPrevented( po.foo.bar     )).toBe( false );
  expect( isUndefinedPrevented( po.foo.bar.baz )).toBe( false );
});

test( 'maxRecursiveLevel ...1 ', ()=>{
  const po = preventUndefined( obj, {maxRecursiveLevel:1 }  );
  expect( isUndefinedPrevented( po             )).toBe( true  );
  expect( isUndefinedPrevented( po.foo         )).toBe( false );
  expect( isUndefinedPrevented( po.foo.bar     )).toBe( false );
  expect( isUndefinedPrevented( po.foo.bar.baz )).toBe( false );
});

test( 'maxRecursiveLevel ...2 ', ()=>{
  const po = preventUndefined( obj, {maxRecursiveLevel:2 }  );
  expect( isUndefinedPrevented( po             )).toBe( true  );
  expect( isUndefinedPrevented( po.foo         )).toBe( true  );
  expect( isUndefinedPrevented( po.foo.bar     )).toBe( false );
  expect( isUndefinedPrevented( po.foo.bar.baz )).toBe( false );
});

test( 'maxRecursiveLevel ...3 ', ()=>{
  const po = preventUndefined( obj, {maxRecursiveLevel:3 }  );
  expect( isUndefinedPrevented( po             )).toBe( true  );
  expect( isUndefinedPrevented( po.foo         )).toBe( true  );
  expect( isUndefinedPrevented( po.foo.bar     )).toBe( true  );
  expect( isUndefinedPrevented( po.foo.bar.baz )).toBe( false );
});
```

***ADDED v0.2.33 (Sat, 07 Jan 2023 17:51:53 +0900)***



 Validators
--------------------------------------------------------------------------------
If you set a validator to the second parameter of `preventUndefined()`, it monitors
every modification on the object to keep consistency of the object.

```javascript
const validator = (o)=>typeof o?.foo?.bar?.value === 'number';
const obj = preventUndefined({
  foo : {
    bar : {
      value : 100,
    },
  }
}, validator);

obj.foo.bar.value = 'BUMMER! NOT A NUMBER';

> ReferenceError: detected defining an invalid property value to obj.foo.bar.value on
> {
>   "foo": {
>     "bar": {
>       "value": "BUMMER! NOT A NUMBER"
>     }
>   }
> }
```

`validator` is a function which receives an argument and returns a boolean
value.

The only argument is the value to be validated. Return `true` if the target
object is valid; otherwise return false.

A `validator` function should not throw an error; if the validator throws an
error, that causes the process that invokes the validation to throw the error.

( You might expect that throwing any error is treated as invalid state of the
target object since if the validation process interrupts the main process, it
could interrupt entire process; but this behavior is intentional because it is
important to let the user know the validator encounters something that the user
did not expect. )

I recommend you to use a non-opinionated validator [Vanilla Schema Validator][];
though it is not mandatory.

See [Vanilla Schema Validator][]

[Vanilla Schema Validator]: https://www.npmjs.com/package/vanilla-schema-validator

Validators have been added in **v0.2.23**.


  `onError`
--------------------------------------------------------------------------------
Started from **v0.2.27**, you can specify an event handler when you call
`preventUndefined()`:
```javascript
  const obj = preventUndefined({
    foo: 'foo',
    bar: 'bar',
  }, {
    onError: (info)=>{
      console.error( 'called ' + info.propPath  );
    }
  });

  console.error( "peekaboo! ",  obj.wrongProp );

> "called wrongProp"
>
> ReferenceError: [prevent-undefined] obj.wrongProp is not defined in {
>   "foo": "foo",
>   "bar": "bar"
> }
> [stacktrace]
> ...
```

Some information about the error which has just occured is available as the
properties on the first argument of the event handler.

#### `propPath` ####
`propPath` property is an array object which consists the property names.

```javascript
  const onError(info)=>{
    console.error(info.propPath); //
  };
  const someObj = {
    hello : {
      world : {
        foo : {
          bar : {
            baz : "HELLO",
          },
        }
      }
    }
  };

  preventUndefined( someObj,{onError}).hello.world.foo.bar.VAZ;
  //                                      SOME TYPO AGAIN  ^^^
```

then, you'll get

```javascript
> hello.world.foo.bar.VAZ
```

#### `target` ####
`target` property is the target object which you specify as the first argument
when you call `preventUndefined()`.

#### `message` ####
`message` property is the string value on the message property on the error
object which is thrown.

#### `stackOnCreated` ####
`stackOnCreated` is an array object which consists the stacktrace when
`preventUndefined()` is called.

#### `stackOnOccured` ####
`stackOnOccured` is an array object which consists the stacktrace when the
illegal access on the target object is occured.

`event handler` started from **v0.2.27**


 errorIfUndefined()
--------------------------------------------------------------------------------
`errorIfUndefined()` is a perfect fancy mascot to implement
''prevent-undefined way'' of named parameters. When you call `errorIfUndefined()`,
it simply throws ReferenceError(). Its only use case is as following :

```javascript
function strictFunc({foo=errorIfUndefined('foo')}) {
  console.error(foo);
}
strictFunc({ foo_WITH_TYPO: 'foo' });

> ReferenceError: the parameter value of foo was undefined; any reference to an undefined value is strictly prohibited on this object.
```

`errorIfUndefined` takes only one parameter : `name` which is to specify the name of the parameter.

`errorIfUndefined()` has been added in **v0.2.19**.


***DEPRECATED v0.2.33*** This feature will be removed in near future.


 The Basic Idea of `prevent-undefined`
--------------------------------------------------------------------------------
IMHO, `undefined` is always evil:

```javascript
  // you defined some procedures :
  function showName(name){
    if (name.length<10) {
      console.error( "a short name", name );
    } else {
      console.error( "a long name", name );
    }
  }
  function foo(guy) {
    showName(guy.name);
  }

  // then, you have some data:
  const guy1 = getYourDataFromResource('john');
  const guy2 = getYourDataFromResource('paul');

  // then, you want to exec the procedures with your data:
  foo(guy1)
  foo(guy2);

  // and if you misteriously get ReferenceError as

  > TypeError: Cannot read properties of undefined (reading 'length')
  > at showName (..) file.js:17:14
  > ...
```

At this point, you have no clue where the problem came from. At this point, you
will never notice that some specific unenthusiastic programmer made a function
as following:

```javascript
const JOHN ={
  name:'John',
  age: 18,
};
const PAUL ={
  name:'Paul',
  age: 23,
};

function getYourDataFromResource(id) {
  if ( id === 'john' ) {
    return {
      name : JOHN.nameeeeeeeeeeeeeeeeee;
    }
  } else if ( id ==='paul' ) {
    return {
      name : PAUL.naaaaaaaaaaaame;
    }
  } else {
    // wat
  }
}
```

Even though the problem is obviously occured inside the function
`getYourDataFromResource()` you will only get the error from a totaly
irrelevant function `showName()` which is very confusing and annoying.

The language should actually throw an Error where the `undefined` comes out.

IMHO, `undefined` is always poisoneous; the language system should have never
allow the existence of `undefined`. In spite of the difficulty, the current
specification of ECMA Script cannot detect undefined properties even if you
enable 'use strict'.

This is where `prevent-undefined` come in.

```javascript
const JOHN = preventUndefined({
  NAME:'John',
  AGE: 18,
});

const PAUL = preventUndefined({
  NAME:'Paul',
  AGE: 23,
});

function getYourDataFromResource(id) {
  if ( id === 'john' ) {
    return {
      name : JOHN.namee; // this throws error.
    }
  } else if ( id ==='PaUL' ) {
    return {
      name : PAUL.naame; // this throws error, too.
    }
  }
}
```

When you applied `prevent-undefined` in this way, you can easily indicate where
the problematic `undefined` comes from.

IMHO, you should always avoid referencing undefined properties for any form.

Write not like this:

```javascript
if ( o.foo ) {
  console.error( 'foo exists' );
}
```

Write like this:

```javascript
if ( 'foo' in o  ) {
  console.error( 'foo exists' );
}
```

Because the conditional check `o.foo` implicitly references `undefined` which
makes detecting `undefined` very difficult.

```javascript

// if the o was get preventUndefined()ed somewhere before:
if ( o.foo ) { // this throws ReferenceError()
  console.error( 'foo exists' );
}
```

Unfortunately some functions from the standard JavaScript library use implicit
referencing `undefined`; for example,

```javascript
const o = preventUndefined( {hello:1})

// The following code will end up ReferenceError which led our `preventUndefined()` to
// ignore `then` with some hard-coding.
Promise.resolve( o );
```

That is, if you have found some standard functions that implicitly refers
`undefined` it is necessary to add the name of the property to the ignore list
which is hard-coded in `prevent-undefined` module.



 History
--------------------------------------------------------------------------------

#### v0.2.6 ####
This didn't work; it didn't support common.js.

#### v0.2.7 ####
(may) supports both common.js and es6 module.

Special thanks goes to [How to Create a Hybrid NPM Module for ESM and CommonJS.](https://www.sensedeep.com/blog/posts/2021/how-to-create-single-source-npm-module.html)
This site helped a lot. Thank you very much.

#### v0.2.8 ####
(Fri, 14 Oct 2022 19:53:18 +0900)

Supported unpreventing null
```
unprevent( null ) == null
```

#### v0.2.9 ####
Fixed an issue that it throws TypeError when a symbol property is accessed.

#### v0.2.10 ####
Added keywords to ignore such as toJSON, `Symbol.toStringTag`,
`Symbol.toPrimitive` , etc.

#### v0.2.11 ####
Added `$$typeof` to the ignore keyword list to let `prevent-undefined` work
cooperatively  with `React.js`.

This very informative article [Why Do React Elements Have a $$typeof Property?][$$typeof]
helped me to understand the problem. Thank you very much.

[$$typeof]: https://overreacted.io/why-do-react-elements-have-typeof-property/

#### v0.2.12 ####
`v0.2.11` does not work as an ES6 module due to my carelessness.  The problem
was fixed n this version.

#### v0.2.13 ####
Skipped the version `v0.2.13` via fear of Friday the 13th.  This version have
never been released.

#### v0.2.14 ####
In the version `v0.2.11`, I documented, commented, and tested it without
modifying the actual code.  Believe me, this time I actually modified.

#### v0.2.15 ####
- Added standard well-known symbols to ignore list; they are to added since
  they are likely to be checked as
```
const i = a[Symbol.iterator];
if (i === undefined ) {
  // ...
}
```

  which is not a preferable behavior for this module.

- Applied automated code generation for tests.


#### v0.2.16 ####
Changed the way to check if the object is a symbol or not.

#### v0.2.17 ####
(Mon, 24 Oct 2022 17:40:53 +0900)

Added '@@iterator' to the ignore list in order to avoid an issue that will
occur when use this module with `React.js`.

#### v0.2.18 ####
(Thu, 03 Nov 2022 17:11:29 +0900)
Removed a unnecessarily left unremoved debug logging output.

#### v0.2.19 ####
(Mon, 07 Nov 2022 16:17:06 +0900)
Added `recursivelyUnprevent()` and `errorIfUndefined()`


#### v0.2.20 ####
(Wed, 09 Nov 2022 15:52:32 +0900)
Added `preventUnusedProperties()`
Updated README.md; added a section `The Basic Idea of preventUndefined()`.

#### v0.2.21 ####
(Wed, 09 Nov 2022 16:01:54 +0900)
Updated README.md.

#### v0.2.22 ####
(Wed, 09 Nov 2022 16:11:21 +0900)
Updated README.md.

#### v0.2.23 ####
(Mon, 14 Nov 2022 17:25:30 +0900)
Supported validators.

#### v0.2.24 ####
(Tue, 15 Nov 2022 16:03:18 +0900)
Supported `rtti`.

#### v0.2.25 ####
(Tue, 15 Nov 2022 16:08:53 +0900)
`rtti` is renamed to `typesafe`.

#### v0.2.26 ####
(Wed, 23 Nov 2022 19:15:59 +0900)
Now `preventUndefined` displays the stacktrace not only where the illegal
access is occured but where the object is protected by `preventUndefined()`,
too. Hopefully this will let you know where your problematic object comes from.

#### v0.2.27 ####
Added the error handler `onError()`.

#### v0.2.28 ####
(Fri, 30 Dec 2022 11:44:22 +0900)
Removed unnecessary files from the package.

#### v0.2.29 ####
(Fri, 30 Dec 2022 14:02:41 +0900)
Deprecated `typesafe`.

#### v0.2.30 ####
- Fixed `README.md`; changed the link to the recommended library for validation
  from `rtti.js` to `vanilla-schema-validator` due to deprecation of `rtti.js`.
- Use Symbol.for() for all iternal properties of `prevent-undefined`.
  (Thu, 05 Jan 2023 11:45:45 +0900)
- Now it protects function objects.
- Now it honors Proxy invariants.
  See the reference :
  https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy/get

#### v0.2.31 ####
- If a specified validator has `script` propety, show its content on validation
  fail.
- Fixed unexpected behavior when the parent class of a class is protected by
  preventUndefined().
- Added ***isUndefinedPrevented()***

#### v0.2.32 ####
-  A Workaround issue that using spread syntax causes following errors :

1.
```
'ownKeys' on proxy: trap returned duplicate entries`
```

2.
```
This is caused by either a bug in Node.js or incorrect usage of Node.js internals.
Please open an issue with this stack trace at https://github.com/nodejs/node/issues`
```

Intercepting `getOwnPropertyDescriptor()` and ownKeys() by using `Proxy`
sometimes makes Node.js crash; I am not sure what exactly causes the problem
but it surely causes it to crash.


#### v0.2.33 ###
(Sat, 07 Jan 2023 17:51:53 +0900)
- Added `maxRecursiveLevel` feature

#### v0.2.34 ###
(Fri, 20 Jan 2023 19:40:35 +0900)
- improved more informative error messages
- removed unnecessary instances of `debugger` directive

#### v0.2.35 ###
(Fri, 20 Jan 2023 19:58:25 +0900)
fixed the issue that the package could not be loaded; v0.2.34 was broken.

#### v0.2.36 ####
(Wed, 25 Jan 2023 14:42:54 +0900)
- Check the specified validator if it returns a function. If so, throw an error
  to let the user know that it could be a validator factory, not a validator
  itself.

#### v0.2.37 ####
(Wed, 25 Jan 2023 16:31:47 +0900)
- validators should be called with explicitly specified `thisArg` via `call()`
  / `apply()`.

#### v3.0.0 ####
(Tue, 21 Feb 2023 10:35:08 +0900)
- Added a new error message system.
  - Quit replacing stacktrace of throwing errors.
  - Set an error object which is created when prevent-undefined is configured
    on every throwing error object.
  - Output the trace of validators when a validation session was failed.

#### v3.0.1 ####
(Tue, 21 Feb 2023 17:27:43 +0900)
- Updated the document.

#### v3.0.2 ####
(Mon, 08 May 2023 13:50:28 +0900)
(Though, actual modification was done on May 2 2023)
- Convert symbol properties without throwing errors.

#### v3.0.3 ####
(Mon, 31 Jul 2023 19:15:26 +0900)
- Added Node.js `Buffer` to the excluding class list.
- Applied `peerDependencies`.

#### v3.0.4 ####
(Fri, 24 May 2024 20:17:33 +0900)
- Distinguish `! ( 'propertyName' in obj )` and 
 `typeof obj.propertyName === 'undefined'`; only throw
  an error when the specified property does not exist.
  (not released)

(Thu, 13 Jun 2024 14:50:09 +0900)
- The internal `inspect` function returns a string value "undefined" when it
  inspects `undefined`; note that JSON.stringify returns a `undefined` value not
  a string value when it receives `undefined` value.  (not released)

(Thu, 05 Sep 2024 18:17:04 +0900)
- The latest version of `prevent-undefined` was released as v3.0.4 on Sep 5 2024.


 Conclusion
--------------------------------------------------------------------------------
That's all. Thank you very much.

- [Author : Atsushi Oka][author]
- [Repository : https://github.com/apupu-framework/prevent-undefined][repository]

[author]: https://github.com/apupu-framework
[repository]: https://github.com/apupu-framework/prevent-undefined


