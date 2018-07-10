# TypeScript学习记录

> 深入学习TS的笔记

## 接口

TS中的接口，是在JS中并不存在的一部分，用来对参数的结构进行规范和检查

```typescript
interface person {
    name:string;
    age:number;
    sex:number;
    country?:string;
}
```

我们定义了这样一个接口，也就定义了一个类似的结构类型

```typescript
function sayOut(who:person) {
    console.log(`the person's name is ${who.name}, age is ${who,age}`);
}
let jack = {
    name:'jack',
    age:18,
    sex:0,
    country:'US'
}
let lily = {
    name:'lily',
    age:16,
    country:'US'
}
sayOut(jack)
sayOut(lily)
```

我么在调用sayOut函数的时候，就对所传入的参数进行接口匹配，传入的参数对象中就必须拥有name，age，sex属性以及可选的属性country，如果另一个lily，传入sayOut就会产生错误，提示缺少age属性。

### 可选属性

用 ? 代表该属性是可选属性，可有可无，不影响检查

```typescript
interface good {
    length?:number;
    height?:number;
}
```

### 只读属性

用 readonly 代表只读属性，一般表示对象创建时指定该属性的数值，之后就不可修改

```typescript
interface circle {
    readonly radius:number;
}
```

