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