# vue的methods使用bind绑定到vue的实例上

## bind
我们回顾下`bind`的手写方法

```js
Function.prototype.mockBind = function () {

    var args = Array.prototype.slice.call(arguments);
    var context = args[0] || window;
    var params = args.slice(1) || [];
    var me = this;

    // this 的指向，是在调用函数时根据执行上下文所动态确定的。
    return function newFn() {
        var params1 = Array.prototype.slice.call(arguments);
        var finalArgs = params.concat(params1);
        // return me.apply(me, finalParams);
        // 上面没有考虑到new function()的情况
        // 如果没有new newFn()，this=window，因为我们拿到的是一个裸方法
        // 如果有new newFn()，this=newFn，虽然拿到的是一个裸方法，但是new改变了内部的指向
        console.warn("newFn this", this);
        return me.apply(this instanceof newFn ? this : context, finalArgs);
    }
}
```

从上面代码我们可以知道，当我们使用`bind`之后，里面的创建的属性都会丢失，比如下面的`test.a=123`
```js
function test(){};
test.a = 123;

const temp = test.bind();
console.log(test.a);
console.log(temp.a);
```

## vue的methods使用bind
而vue的methods会使用bind将方法绑定到vue的实例上，这样我们才能使用`this.temp`拿到对应的方法，`this`代表`vue的实例`

而在bind的过程中会丢失一些属性，比如下面的`test.cancel()`，解决方式是将`temp`放在`data(){}`中，就不会使用bind了

```js
function test() {

}
test.cancel = function() {};
methods: {
    temp: test,
    temp1() {
        this.temp.cancel(); // 报错！
    }
}
```