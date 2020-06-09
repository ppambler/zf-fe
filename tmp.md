| ✍️ Tangxt | ⏳ 2020-06-01 | 🏷️ 草稿 |

# 草稿

在建立这个仓库之初：

> 笔记之Web前端开发高级工程师（珠峰） -> 我尝试用一种新的姿势去观看视频学习 -> 只记自己不懂的或者听完之后有一种「原来如此」的感觉，而不是之前的听一句文字化敲一句

看了20多节之后：

我觉得我是重学了一遍……

## ★前言

之前说到，闭包有两大作用：

- 保护
- 保存

针对于闭包，衍生了几个所谓的高阶编程技巧：

1. 惰性思想
2. 柯里化函数思想
3. 利用柯里化函数思想加闭包概念 -> 写了一个compose函数

而现在还有第四个，那就是curry了！

那么curry是啥呢？

简单来说就是函数包裹的应用 -> 相当于你写了一个函数xxx，通过这个函数xxx可以创造一个函数yyy，然后让yyy可以实现一些事情

## ★编写一个ADD函数满足如下需求

``` js
add(1);       //1
add(1)(2);    //3
add(1)(2)(3); //6
add(1)(2,3);  //6
add(1,2)(3);  //6
add(1,2,3);   //6
```

分析：

不定执行次数，不定参数个数，最后结果是把所有所传的参数一同相加在一起

如：

- 执行一次`add(1)`，执行两次`add(1)(2)`……
- 传一个参数`add(1)`，传两个参数`add(1,2)`……
- 结果都是所传参数的总和

方案 -> 网上大概有7、8种，但有些方案就目前来说已经不是特别好了 -> 结合老师自己写出来的方案+网上参考的方案，得出有两种方案是比较能说明这件事的！

### ◇方案一

``` js
function add(...outerArgs) {
  add = function (...innerArgs) {
    outerArgs.push(...innerArgs);
    return add;
  };
  add.toString = function () {
    return outerArgs.reduce((x, y) => x + y);
  };
  return add;
}
let res = add(1, 2)(3)(4)(5)(6, 7);
alert(res); //=>alert会把输出的值转换为字符串（toString()）
```

分析：

- 不知道执行多少次 -> 确定返回的是一个函数
- 不确定传几个参数 -> 确定用`...args`

第一次执行需要重写`add`函数 -> 利用闭包不销毁的机制，把所传参数保存到第一次执行的`outerArgs`

为啥要重写`add`的`toString`方法呢？

因为这个函数调用完，肯定是一个`add`函数，但是我们并不想要这个函数值，于是我们重写了`add`的`toString`写法

但add调用`toString`方法，就会拿到`outerArgs`所有保存下来的值 -> 为啥可以拿到呢？ -> 因为`toString`方法的是在`add`函数定义的，所以它的作用域链就指向了`add[AO]`

代入法理解：

``` js
/*
 * 第一次执行ADD  outerArgs=[1,2]  重写了ADD -> outerArgs没有被释放销毁
 * 第二次执行ADD  innerArgs=[3]   outerArgs=[1,2,3]
 * 第三次执行ADD  innerArgs=[4]   outerArgs=[1,2,3,4]
 * ......
 * outerArgs=[1,2,3,4,5,6,7]
 */
console.log(res.toString()); //28
console.log(+res) //28
```

总之，这道题结合了「闭包概念（重写add函数）」+「面向对象概念（重写add的toString方法）」

以上就是老师第一次接触这道题所搞出来的答案……

> `res`只能拿到函数，不能拿到最终的结果，毕竟你无法确定要执行多少次 -> 函数执行完后如何拿到结果呢？ -> `toString`方法，当然用`valueOf`也行，只是老师个人习惯用`toString`罢了！（在上篇文章中为啥老师咩有提valueOf？因为它基本用不上！）

### ◇方案二

> 这个很难理解

``` js
function currying(anonymous, length) {
  return function add(...args) {
    if (args.length >= length) {
      return anonymous(...args);
    }
    return currying(anonymous.bind(null, ...args), length - args.length);
  };
}
let add = currying(function anonymous(...args) {
  return args.reduce((x, y) => x + y);
}, 4);
```

curry，意思是「咖喱」，而咖喱就是把一堆香料**捣碎**制成的调料

> 有一球星 -> 斯蒂芬·库里（Stephen Curry）
> 
> 所谓「捣碎」，就是把一次传参捣碎成多次传参：`xxx(1,2)` -> `xxx(1)(2)`

在编程里边，curry的意思是封装或混装一个函数，简单来说，就是你传个函数值，就可以输出一个别的函数值（添加了作料的函数值）

再次强调JS有四大常见的高阶编程技巧：

1. 惰性函数（思想）
2. 柯里化函数 -> 大函数里边返回小函数
3. curry函数 -> 混装
4. compose函数 -> 把连续调用的函数扁平化

分析上边的代码：

1. 执行currying函数 -> `currying(求和函数,4)`
   1. AO（currying）
      1. anonymous -> 求和函数（所传的参数值相加）
      2. length -> 4 -> 表示我传了4个参数需要求和，如果不传length值或者参数求和个数不对头，那么就会报错。如需要对4个参数求和，那么就填4，不然就会报错 -> 作用 ->递归的终结点判断，即base case -> 如果你一开始就传了4个参数，那么就直接执行求和函数，也就不会体现所谓的递归，毕竟这个问题忒TM简单了……但复杂问题在于「我们无法确定用户是一次性传完一次调用，还是多次传多次调」

小结：

currying这个函数的意思：

1. 你传给我一个函数，那么我就给你返回一个包装函数（可以理解为代理），在这个代理函数里边我们可以去处理传进来的函数
2. 代理函数的执行，就是在递归的执行…… -> 每次递归都会创建一个不销毁的闭包





























