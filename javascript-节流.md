## javascript-节流
### 写在前面
为什么要限制事件的频繁触发，以及如何做限制：

debounce 防抖

throttle 节流

今天重点讲讲节流的实现。
### 节流
节流的原理很简单：

`如果你持续触发事件，每隔一段时间，只执行一次事件`。

根据首次是否执行以及结束后是否执行，效果有所不同，实现的方式也有所不同。
我们用 `leading` 代表首次是否执行，`trailing`代表结束后是否再执行一次。

关于节流的实现，有两种主流的实现方式，`一种是使用时间戳`，`一种是设置定时器`。

### 时间戳
使用时间戳，当触发事件的时候，我们取出当前的时间戳，然后减去之前的时间戳(最一开始值设为 0 )，如果大于设置的时间周期，就执行函数，然后更新时间戳为当前的时间戳，如果小于，就不执行。
```javascript
function throttle(func,wait) {
  var context,args;
  var previous = 0;

  return function(){
    var now = +new Date();
    context = this;
    args = arguments;
    if(now - previous > wait){
      func.apply(context,args);
      previous = now;
    }
  }
}
```


当鼠标移入的时候，事件立刻执行，每过 1s 会执行一次，如果在 4.2s 停止触发，以后不会再执行事件。
### 定时器
当触发事件的时候，我们设置一个定时器，再触发事件的时候，如果定时器存在，就不执行，直到定时器执行，然后执行函数，清空定时器，这样就可以设置下个定时器。
```javascript
function throttle(func, wait) {
  var timeout;
  var previous = 0;

  return function() {
      context = this;
      args = arguments;
      if (!timeout) {
          timeout = setTimeout(function(){
              timeout = null;
              func.apply(context, args)
          }, wait)
      }

  }
}
```
当鼠标移入的时候，事件不会立刻执行，晃了 3s 后终于执行了一次，此后每 3s 执行一次，当数字显示为 3 的时候，立刻移出鼠标，相当于大约 9.2s 的时候停止触发，但是依然会在第 12s 的时候执行一次事件。
### 双剑合璧
有人就说了：我想要一个有头有尾的！就是鼠标移入能立刻执行，停止触发的时候还能再执行一次！
```javascript
function throttle(func,wait){
  var timeout,context,args,result;
  var previous = 0;

  var later = function(){
    previous = +new Date();
    timeout = null;
    func.apply(context,args)
  };

  var throttled = function(){
    var now = +new Date();
    //下次触发func剩余的时间
    var remaining = wait - (now - previous);
    context = this;
    args = arguments;
    //如果没有剩余时间或者你改了系统时间
    if(remaining <= 0 || remaining > wait){
      if(timeout){
        clearTimeout(timeout);
        timeout = null;
      }
      previous = now;
      func.apply(context,args);
    } else if (!timeout){
      timeout = setTimeout(later,remaining);
    }
  };
  return throttled;
}
```
鼠标移入，事件立刻执行，晃了 3s，事件再一次执行，当数字变成 3 的时候，也就是 6s 后，我们立刻移出鼠标，停止触发事件，9s 的时候，依然会再执行一次事件。