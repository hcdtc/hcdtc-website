# 异步编程
ES2017 标准引入了 async 函数，使得异步操作变得更加方便。

`callback -> fibers.js -> promise -> generator + co.js -> async/await`

异步处理的这个演进过程，语法糖发生了改变。但是实际处理问题的关键还是在于分清楚什么时候要并发执行，什么时候要串行执行。明白实际场景所需要的执行顺序，才能写出正确代码。

在开发项目时，项目中使用到`async/await`、`Promise`来处理异步问题，很多同事对`async/await`没有正确的基础的了解或者是使用场景有所误解，导致出现一些性能问题。

本文主要指出滥用`async`带来的问题，引发平时开发的一些警示和思考。


## 实例

### An example of async/await hell

一个滥用的例子，订购披萨和饮料：

```typescript
(async () => {
  const pizzaData = await getPizzaData(); // async call
  const drinkData = await getDrinkData(); // async call
  const chosenPizza = choosePizza(); // sync call
  const chosenDrink = chooseDrink(); // sync call
  await addPizzaToCart(chosenPizza); // async call
  await addDrinkToCart(chosenDrink); // async call
  orderItems(); // async call
})();
```
从表面上看是正确的，而且确实有效。但是这不是一个好的实现，因为它没有考虑并发性。

回到我们吐槽的回调地狱，虽然代码比较丑，但起码两行回调代码并不会带来阻塞。

看来语法的简化，使用者的疏忽，带来了性能问题，而且直接影响到用户体验，是不是值得反思一下？

我们分析一下它在做什么，弄清问题所在。

以下事件的发生顺序如下:

1. 获取披萨清单
2. 拿到饮料清单
3. 从清单中选择一个披萨
4. 从清单中选择一种饮料
5. 将选择的披萨添加到购物车中
6. 将选择的饮料添加到购物车中
7. 把购物车里的东西提交

哪里出问题了？

正如我前面强调的，所有这些语句都是一个一个执行的。这里没有并发。仔细想想:为什么我们要等拿到披萨的清单后才去拿饮料的清单呢?我们应该把这两张清单放在一起。然而，当我们需要选择披萨时，我们确实需要事先有披萨的清单。饮料也是一样。

所以我们可以得出结论，披萨相关的工作和饮料相关的工作可以并行进行，但是披萨相关工作中涉及的各个步骤需要顺序进行(一个接一个)。

await 语法本身没有问题，但是当 `pizzaData` 与 `drinkData` 之间没有依赖时，顺序的 await 会最多让执行时间增加一倍的 `getPizzaData` 函数时间，因此 `getPizzaData` 与 `getDrinkData` 应该并行执行。

正确的做法应该是先同时执行函数，再 await 返回值，这样可以并行执行异步函数：

```typescript
async function selectPizza() {
  const pizzaData = await getPizzaData()    // async call
  const chosenPizza = choosePizza()    // sync call
  await addPizzaToCart(chosenPizza)    // async call
}

async function selectDrink() {
  const drinkData = await getDrinkData()    // async call
  const chosenDrink = chooseDrink()    // sync call
  await addDrinkToCart(chosenDrink)    // async call
}

(async () => {
  const pizzaPromise = selectPizza();
  const drinkPromise = selectDrink();
  await pizzaPromise;
  await drinkPromise;
  orderItems(); // async call
})();
```

或者使用 `Promise.all` 可以让代码更可读：

```typescript
(async () => {
  Promise.all([selectPizza(), selectDrink()]).then(orderItems); // async call
})();
```

看来不要滥用的 await，它很可能让你代码性能降低。

### Another example of bad implementation

此JavaScript代码片段将获取购物车中的商品并发出订单请求。

```typescript
async function orderItems() {
  const items = await getCartItems();   // async call
  const noOfItems = items.length;
  for(let i = 0; i < noOfItems; i++) {
    await sendRequest(items[i])    // async call
  }
}
```

在这种情况下，for循环必须等待sendRequest()函数完成，然后才能继续下一个迭代。然而，我们实际上并不需要等待。我们希望尽可能快地发送所有请求，然后我们可以等待所有请求完成。

我希望现在你够更加了解什么是`async/await`性能问题，严重影响程序的性能。现在我想问你一个问题：

*What if we forget the await keyword ?*

如果在调用异步函数时忘记使用`await`。这意味着执行函数不需要等待。async函数将返回一个promise，您可以稍后使用它。

```typescript
(async () => {
  const value = doSomeAsyncTask();
  console.log(value); // an unresolved promise
})()
```

另一个结果是编译器不知道您想要等待函数完全执行。因此，编译器将异步任务未完成之前就退出程序。所以我们需要await关键字。

`promise`的一个有趣特性是，您可以在一行中获得一个`promise`，然后在另一行中等待它并`resolve`。这是逃离`async/await`性能问题的关键。

```typescript
(async () => {
  const promise = doSomeAsyncTask();
  const value = await promise;
  console.log(value); // the actual value
})()
```
可以看到，`doSomeAsyncTask()`返回一个`promise`。此时，`doSomeAsyncTask()`已经开始执行。要获得承诺的解析值，我们使用`await`关键字，它将告诉`JavaScript`不要立即执行下一行，而是等待`promise resolve`，然后执行下一行。

### 理解语法糖

虽然要正确理解 `async/await` 的真实效果比较反人类，但为了清爽的代码结构，以及防止写出低性能的代码，还是挺有必要认真理解 `async/await` 带来的改变。

首先 `async/await` 只能实现一部分回调支持的功能，也就是仅能方便应对层层嵌套的场景。其他场景，就要动一些脑子了。

比如两对回调：

```typescript
a(() => {
  b();
});

c(() => {
  d();
});
```

如果写成下面的方式，虽然一定能保证功能一致，但变成了最低效的执行方式：

```typescript
await a();
await b();
await c();
await d();
```

因为翻译成回调，就变成了：

```typescript
a(() => {
  b(() => {
    c(() => {
      d();
    });
  });
});
```

然而我们发现，原始代码中，函数 `c` 可以与 `a` 同时执行，但 async/await 语法会让我们倾向于在 `b` 执行完后，再执行 `c`。

所以当我们意识到这一点，可以优化一下性能：

```typescript
const resA = a();
const resC = c();

await resA;
b();
await resC;
d();
```

但其实这个逻辑也无法达到回调的效果，虽然 `a` 与 `c` 同时执行了，但 `d` 原本只要等待 `c` 执行完，现在如果 `a` 执行时间比 `c` 长，就变成了:

```typescript
a(() => {
  d();
});
```

看来只有完全隔离成两个函数：

```typescript
(async () => {
  await a();
  b();
})();

(async () => {
  await c();
  d();
})();
```

或者利用 `Promise.all`:

```typescript
async function ab() {
  await a();
  b();
}

async function cd() {
  await c();
  d();
}

Promise.all([ab(), cd()]);
```

这就是我想表达的可怕之处。回调方式这么简单的过程式代码，换成 `async/await` 居然写完还要反思一下，再反推着去优化性能，这简直比回调地狱还要可怕。

而且大部分场景代码是非常复杂的，同步与 `await` 混杂在一起，想捋清楚其中的脉络，并正确优化性能往往是很困难的。但是我们为什么要自己挖坑再填坑呢？很多时候还会导致忘了填。

原文作者给出了 `Promise.all` 的方式简化逻辑，不要一昧追求 `async/await` 语法，在必要情况下适当使用回调，是可以增加代码可读性的。

## 优化

如何处理`async/await`问题？

### 1. 查找依赖于其他语句执行的语句

在我们的第一个例子中，我们选择了比萨饼和饮料。我们的结论是，在选择比萨饼之前，我们需要有比萨饼列表。在将比萨饼添加到购物车之前，我们需要选择比萨饼。所以我们可以说这三个步骤相互依赖。在完成之前的事情之前，我们不能做一件事。

但是如果我们更广泛地看一下，我们发现选择比萨饼不依赖于选择饮料，所以我们可以同时选择它们。这是机器可以做得比我们做得更好的一件事。

因此，我们发现了一些依赖于其他语句执行的语句，而另一些语句则没有。

### 2. 异步函数中依赖于组的语句

正如我们所看到的，选择披萨涉及依赖性陈述，例如获取比萨饼列表，选择一个比萨饼，然后将所选比萨饼添加到购物车中。我们应该在异步函数中对这些语句进行分组。这样，我们得到两个异步函数，`selectPizza()`和`selectDrink()` 。

### 3. 同时执行这些异步功能

然后，我们利用事件循环同时运行这些异步非阻塞函数。这样做的两种常见模式是`returning promises early`和`Promise.all`方法。

按照这三个步骤，让我们试着优化之前的示例2。

在第二个例子中，我们需要处理未知数量的`promise`。处理这种情况非常简单:我们只需创建一个数组并在其中`push promise`。然后使用`Promise.all()`，我们同时等待所有的promise被解析。
```typescript
async function orderItems() {
  const items = await getCartItems();    // async call
  const noOfItems = items.length;
  const promises = [];
  for(let i = 0; i < noOfItems; i++) {
    const orderPromise = sendRequest(items[i]);    // async call
    promises.push(orderPromise)    // sync call
  }
  await Promise.all(promises)    // async call
}

// Although I prefer it this way 

async function orderItems() {
  const items = await getCartItems();    // async call
  const promises = items.map((item) => sendRequest(item));
  await Promise.all(promises);    // async call
}
```
## 总结

希望本文能帮助您了解`async/await`的基础之外的内容，并帮助您改进应用程序的性能。

最后，不要过度依赖新特性，决定代码质量的是思维，而非框架或语法。


## 相关资料

> [`async` 详解](http://es6.ruanyifeng.com/#docs/async)
>
> [`promise` 详解](http://es6.ruanyifeng.com/#docs/promise)
>
> [`promise` 问题](https://github.com/sindresorhus/promise-fun)

原文出处：

> [How to escape async/await hell](https://medium.freecodecamp.org/avoiding-the-async-await-hell-c77a0fb71c4c)
