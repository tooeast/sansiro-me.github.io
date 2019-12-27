防抖和节流是平时经常用到的基础的方法，在前端开发时，对于很多场景都需要做相应的防抖或者节流处理，否则会造成性能的浪费或者影响用户体验，比如说常见到的 ```resize``` 和 ```scroll``` 事件。
 
但是很多同学经常把防抖和节流搞混，防抖当成节流，节流认为是防抖。所以我们重新理解学习一下防抖和节流的知识！

> 浏览器中某些计算和处理要比其他的昂贵很多。例如，DOM 操作比起非 DOM 操作需要更多的内存和 CPU 时间。连续尝试进行过多的 DOM 相关操作可能会导致浏览器挂起，有时候甚至会崩溃。尤其在 IE 中使用 ```onresize``` 事件处理程序的时候容易发生，当调整浏览器大小的时候，该时间会连续触发。在 ```onresize``` 事件处理程序内部如果尝试进行 DOM 操作，其高频率的更改可能会让浏览器崩溃。为了绕开这个问题。你可以使用定时器对该函数进行**节流**。  


> 函数节流背后的思想是指，在某些代码不可以在没有间断的情况下连续重复执行。第一次调用函数，创建一个定时器，在指定的时间间隔之后运行代码。当第二次调用该函数的时候，他会清除第一次的定时器并设置另一个。...... 目的是只有在执行函数的请求停止了一段时间之后才执行。

以上是红宝石书对于函数节流的描述。从中可以理解到，对于连续重复执行的代码，限制其执行。比如说有连续十个请求，间隔一秒钟连续发起请求。如果我们设置的节流时间是两秒的话，只有在最后一个请求后两秒才会真正的执行这个请求。

这种情况经常应用在处理用户输入时，如果要根据用户输入的内容发起请求，不加限制的话，用户每次修改（输入或者删除）都会触发请求，但是用户没有完成输入的话，前面的请求都是浪费的，没必要的。所以我们限制用户输入完成一段时间之后才发起请求，代码如：

```javascript
let input = document.getElementById('input');

let example = debounce(request, 300);

input.onkeyup = e => {
  example(e.target.value); 
}

function debounce(func, delay) {
  let timer;

  return args => {
    clearTimeout(timer);
    timer = setTimeout(() => {
      func(args);
    }, delay);
  }
}

function request(val) {
  console.log('request: ' + val);
}
```

**但是！**

日常使用中，我们认为的节流并不是这样的，这种函数我们一般叫做防抖，顾名思义就是防止不停抖动，等平静下来了不抖了再去处理。那节流是什么啥样子呢？

```javascript
let example = throttle(doSomeThing, 300);

window.onresize = () => {
  example();
}

function throttle(func, delay) {
  let timer;

  return args => {
    if(timer) return;

    func(args);
    
    timer = setTimeout(() => {
      clearTimeout(timer);
      timer = null;
    }, delay);
  }
}

function doSomeThing() {
  console.log('do some thing...');
}
```

这是一个简单的节流函数，节流的意思是某时间段内函数只能执行一次，如上述代码，当我们不停拖动窗口大小的时候，```onresize``` 是不停触发的，但是通过我们限制，允许他最多每 300ms 执行一次，这样可以大大的减少性能的浪费，提高我们程序的性能。

以上都是简单的防抖和节流的用法，对于更复杂的场景或有更高级的写法，我们就不去探究了。这次我们主要是弄清楚防抖和节流的区别，了解各自的应用场景。
