# Javascript proxy - Querying arrays with more readable methods

## javascript proxy
The proxy object is used to define custom behavior for fundamental operations(e.g.property lookup, assignment, enumeration, function invocation, etc)

## the basics.
there are 3 key terms we need to define before we proceed:
- handler - the placeholder object which contains trap(s)
- trasp - the method(s) that provide property acces
- target - object which the proxy virtualizes

Here is a list of available traps.
- apply
- construct
- defineProperty
- deleteProperty
- get
- getOwnPropertyDescriptor
- getPrototypeOf
- has
- isExtensible
- ownKeys
- preventExtensoins
- set
- setPrototypeOf

## 语法
````
 let proxy = new Proxy( target, trapObject ); 
````

## 例子
在开始创建我们数据查询方法库之前我们先来看一个快速实例。

```` javascript
class Student {
    constructor(first, last, scores) {
        this.firstName = first
        this.lastName = last
        this.testScores = scores
    }

    get average() {
        let averate = this.testScores.reduce((a, b) => a + b, 0) / this.testScores.length

        return average
    }
}

// instantiate the student class
let john = new Student('john', 'Doe', [70, 80, 90])

let johnProxy = new Proxy(john, {
    get: function(target, key, context) {
        console.log(`john[${key}] was accessed.`)
        
        return john[key]
    }
})

johnProxy.firstName
// output
// > john[firstName] was accessed
// > <property content>
````

在这个例子中，我们只拦截了属性调用，例如`johnProxy,firstName`，并在控制台打印出来了。这种拦截可用于在开发环境排除错误。当我们为一直值设置属性时也可以做到同样的事情：我们也可以拦截`set`并且在控制台打印一个消息。

## 开始创建我们的查询库吧
有点像写断言库，你可以利用你想查询的数组域名来通过方法查询，而不是把它作为一个参数来传递。

### 首先我们先创建一个断言对象
在这里我们将要定义一个断言操作集，代理对象将会返回会在基于以下条件的对象。

````javascript
let assertions = {
    Eauqls: (object, value) => object === value,
    IsNull: (object, value) => object === null,
    IsUndefined: (object, value) => object === undefined,
    IsEmpty: (object, value) => object.length === 0,
    Includes: (object, value) => object.includes(value),
    IsLowerThan: (object, value) => object < value,
    IsGreaterThan: (object, value) => object > value,
    EndsWidth: (object, value) => object.endsWidth(value),
    StartsWidth: (object, value) => object.startsWith(value)
}

````

### 代理对象
Here we going to trap get, of the target object to read the name of the property and make sure it starts width findWhere the we use the Object field name we want to query, then the assertion(from the assertion above), So the syntax will looks something like this:

所以语法将会看起来像下面这样
````
arr.findWhereNameStartsWidth('Jo')
````

````
let wrap = arr => {
    const prefix = 'findWhere'
    let assertionNames = Object.keys(assertions)

    return new Proxy(arr, {
        get (target, propKey) {
            if (propKey in target) return target[propKey]

            var assertionsName = assertionsNames.find(assertion => propKey.endsWidth(assertion))

            if (propKey.startsWidth(prefix)) {
                var field = propKey.substring(prefix.length, propKey.length - assertionName.length).toLowerCase()

                var assertion = assertion[assertionName];

                return value => {
                    return target.filter(item => assertion(item[field], value))
                }
            }
        }
    })
}
````

### 定义对象

下一步就是定义对象，并用该代理包裹

let arr = wrap([
  { name: 'John', age: 23, skills: ['mongodb', 'javascript'] },
  { name: 'Lily', age: 20, skills: ['redis'] },
  { name: 'Iris', age: 43, skills: ['python', 'javascript'] }
])

### 调用方法
````
console.log(arr.findWhereNameEquals('LiLy')) // finds Lily
console.log(arr.findWhrerSkillsIncludes('javascript')) // finds Iris and John
````
上面代码第一行返回“Name”属性等于“Lily”的对象并打印出来，第二行返回“skills"属性包含“javascript”的对象。

#### 完整源码
````
let assertions = {
  Equals: (object, value) => object === value,
  IsNull: (object, value) => object === null,
  IsUndefined: (object, value) => object === undefined,
  IsEmpty: (object, value) => object.length === 0,
  Includes: (object, value) => object.includes(value),
  IsLowerThan: (object, value) => object < value,
  IsGreaterThan: (object, value) => object > value,
  EndsWith: (object, value) => object.endsWith(value),
  StartsWith: (object, value) => object.startsWith(value),
}


let wrap = arr => {
  const prefix = 'findWhere'
  let assertionNames = Object.keys(assertions)
  return new Proxy(arr, {
    get(target, propKey) {
      if (propKey in target) return target[propKey]
      var assertionName = assertionNames.find(assertion =>
        propKey.endsWith(assertion))
      if (propKey.startsWith(prefix)) {
        var field = propKey.substring(prefix.length,
          propKey.length - assertionName.length).toLowerCase()
        // debugger;
        var assertion = assertions[assertionName]
        return value => {
          return target.filter(item => assertion(item[field], value))
        }
      }
    }
  })
}


let arr = wrap([
  { name: 'John', age: 23, skills: ['mongodb', 'javascript'] },
  { name: 'Lily', age: 20, skills: ['redis'] },
  { name: 'Iris', age: 43, skills: ['python', 'javascript'] }
]);

console.log(arr.findWhereNameEquals('Lily')) // finds Lily
console.log(arr.findWhereSkillsIncludes('javascript')) // finds Iris

const appDiv = document.getElementById('code');
appDiv.innerHTML = JSON.stringify(arr.findWhereNameStartsWith('Jo'));

// const appDiv = document.getElementById('code1');
// appDiv.innerHTML = JSON.stringify(arr.findWhereSkillsIncludes('javascript'));

const appDiv = document.getElementById('code1');
appDiv.innerHTML = JSON.stringify(arr.findWhereAgeIsLowerThan(30));
````

### 需要注意的地方
注意我们如何调用数组中我们之前没有定义过的方法

````
arr.findWhereAgeIsLowerThan(30)
````

我们通过代理来读我们的方法，理想的语法如下：
````
findWhere + <array_field_name> + <assertion>
````
这样允许我们通过数组域名称和我们想查询范围方法名称来创建自定义查询

- arr.findWhere**Age**IsLowerThan(27)
- arr.findWhere**Name**Equals('Lily')
- arr.findWhere**Skills**Includes('javascript')

## Wrapping up
通过Javascript代理，你可以包裹一个已经存在的对象，并且拦截任意访问该对象的属性和方法，甚至时不存在的属性和方法。

代理在现实中有很多实际的应用：
- Create SDK for an API
- validation
- value correction
- Revoke access to an object property/method
- Querying arrays width more readable methods(the main topic of this post)
- property lookup extensions
- tracing property accesses
- Monitoring async functions

## 总结
Javascript proxy enables a lot of possibilities to create more readable APIs where you can see in the link below IE dosen't support
it and this can be a downside to create codes that needs to be supported by all major browsers. Unfortunately, it's not possible to polyfill or transpile ES6 proxy code using tools such as Babel, because porxies have no ES5 equivalent.

For more info about javascirpt proxy and browser compatibility you can [Learn more here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy).

## 参考
1. [Javascript proxy - Querying arrays with more readable method](http://www.jomendez.com/2018/12/29/querying-arrays-with-more-readable-methods-using-javascript-proxy/)