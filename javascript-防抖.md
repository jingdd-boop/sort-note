## javascript-防抖
### 写下前面
举个例子了解事件如何频繁的触发,在页面上显示一个div元素，给它指定宽高和颜色，并使它拥有id属性，便于后面使用js文件：

`index.html`文件：

```xhtml
<!DOCTYPE html>
<html lang="zh-cmn-Hans">
<head>
    <meta charset="utf-8">
    <meta http-equiv="x-ua-compatible" content="IE=edge, chrome=1">
    <title>debounce</title>
    <style>
        #container{
            width: 100%; height: 200px; line-height: 200px; text-align: center; color: #fff; background-color: #444; font-size: 30px;
        }
    </style>
</head>

<body>
    <div id="container"></div>
    <script src="debounce.js"></script>
</body>
</html>
```
debounce.js文件中,首先定义一个计数的count = 1，然后取到在index.html中的container元素，创建一个方法，当执行这个方法是，count的值加1；当鼠标移动时，便触发该函数：

```javascript
var count = 1;
var container = document.getElementById('container')

function debounce() {
  container.innerHTML = count++;
};

container.onmousemove = debounce
```
可以看到，当鼠标移动时，可以发现数字不断的变化，这就是一个不断触发事件的例子。
为什么要限制事件的频繁触发，以及如何做限制：

debounce 防抖

throttle 节流

今天重点讲讲防抖的实现。


<b>因为这个例子很简单，所以浏览器完全反应的过来，可是如果是复杂的回调函数或是 ajax 请求呢？假设 1 秒触发了 60 次，每个回调就必须在 1000 / 60 = 16.67ms 内完成，否则就会有卡顿出现.</b>

为了解决这个问题，一般有两种解决方案：

`1、debounce 防抖`

`2、throttle 节流`

### 防抖
防抖的原理就是：你尽管触发事件，但是我一定在事件触发 n 秒后才执行，如果你在一个事件触发的 n 秒内又触发了这个事件，那我就以新的事件的时间为准，n 秒后才执行，总之，`就是要等你触发完事件 n 秒内不再触发事件，我才执行!!`


```javascript
function  debounce(func,wait) {
  var timeout;
  return function () {
    clearTimeout(timeout)
    timeout = setTimeout(func,wait);
  }
}

//当然我们需要使用它
container.onmousemove =  debounce(getUserAction, 1000);
```
现在随你怎么移动，反正你移动完 1000ms 内不再触发，我才执行事件。

接着开始完善它：

### this
```javascript
function getUserAction() {
  container.innerHTML = count++;
  console.log(this);
  
};
```
当不使用debounce时输出：

`<div id="container"></div>`
 ![Image text](/img/防抖1.png)

当使用debounce时输出：`window`
![Image text](/img/防抖2.png)

所以我们需要将 this 指向正确的对象:
```javascript
function  debounce(func,wait) {
  var timeout;
  return function () {
    var context = this;

    clearTimeout(timeout)
    timeout = setTimeout(function () {
      func.apply(context)
    },wait);
  }
}
```

此时this指向：
![Image text](/img/防抖3.png)


### event 对象
JavaScript 在事件处理函数中会提供事件对象 event，我们修改下 getUserAction 函数：
```javascript
function getUserAction(e) {
  console.log(e);
  container.innerHTML = count++;
  
};
```
如果我们不使用 debouce 函数，这里会打印 MouseEvent 对象，如图所示：

![Image text](/img/防抖4.png)


但是在我们实现的 debounce 函数中，却只会打印 undefined!

![Image text](/img/防抖5.png)

所以我们再修改一下代码：

![Image text](/img/防抖4.png)

```javascript
function  debounce(func,wait) {
  var timeout;
  return function () {
    var context = this;
    var args = arguments;

    clearTimeout(timeout)
    timeout = setTimeout(function () {
      func.apply(context,args)
    },wait);
  }
}
```

到此为止，修复了两个小问题：

`1、this 指向`

`2、event 对象`

### 立即执行

如果不希望非要等到事件停止触发后才执行，`我希望立刻执行函数，然后等到停止触发 n 秒后，才可以重新触发执行`。

想想这个需求也是很有道理的嘛，那我们加个 immediate 参数判断是否是立刻执行。
```javascript
function  debounce(func,wait,immediate) {
  var timeout;
  return function () {
    var context = this;
    var args = arguments;


    if(tiemout) clearTimeout(timeout);
    if(immediate){
      //如果已经执行过了，不再执行
      var callNow = !timeout;
      timeout = setTimeout(function () {
        timeout = null;
      },wait)
      if(callNow) func.apply(context,args)
    }
    else{
      timeout = setTimeout(function () {
        func.apply(context,args)
      },wait);
    }
  }
}
```
### 返回值

```javascript
function  debounce(func,wait,immediate) {
  var timeout,result;
  return function () {
    var context = this;
    var args = arguments;


    if(tiemout) clearTimeout(timeout);
    if(immediate){
      //如果已经执行过了，不再执行
      var callNow = !timeout;
      timeout = setTimeout(function () {
        timeout = null;
      },wait)
      if(callNow) func.apply(context,args)
    }
    else{
      timeout = setTimeout(function () {
        func.apply(context,args)
      },wait);
    }
    return result;
  }
}
```

### 取消

再思考一个小需求，我希望能取消 `debounce `函数，比如说我 `debounce` 的时间间隔是 10 秒钟，`immediate `为 true，这样的话，我只有等 10 秒后才能重新触发事件，现在我希望有一个按钮，点击后，取消防抖，这样我再去触发，就可以又立刻执行啦，是不是很开心？

```javascript
debounced.cancel = function() {
        clearTimeout(timeout);
        timeout = null;
  };

return debounced;
```
使用：
```javascript

var count = 1;
var container = document.getElementById('container');

function getUserAction(e) {
    container.innerHTML = count++;
};

var setUseAction = debounce(getUserAction, 10000, true);

container.onmousemove = setUseAction;

document.getElementById("button").addEventListener('click', function(){
    setUseAction.cancel();
})
```
