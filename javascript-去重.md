## javascript-去重
### 方法1 双层循环
```javascript
function unique(array){
  for(let i = 0;i<array.length;i++){
    for(let j =i+1;j<array.length;j++){
      if(array[i] === array[j]){
      array.splice(j,1);
      }
    }
  }
  return array;
}
var array = [1,2,3,4,3,5]
console.log(unique(array));
```

通过两次循环遍历，外层遍历全部数组元素从i开始，内层遍历`i+1（j）`开始到数组最后面的元素的元素，当`array[i]` 和 `array[j]`相等时，把这个array[j]用splice方法从数组中去除
返回array

### 方法2 排序后去重

```javascript
function sort(array){
  let sortNum = array.sort(function(a,b){
    return a-b
  });
  for(let i = 0;i < sortNum.length;i++){
    if(sortNum[i] == sortNum[i+1]){
      sortNum.splice(i,1);
    }
  }
  return sortNum;
}

var arr = [1,2,3,2,5,3,4];
console.log(sort(arr))
```
先排序，使用sort方法进行排序，得到一个新的排序数组`sortNum，`这样的话，可以比较数组中相邻的元素是否相同，如果相同则删除一个，再返回该数组就可以了。

sort方法：
```javascript
let sortNum = array.sort(function(a,b){
    return a-b
  });
```

### 方法3 es6
```javascript
function sort(array){
  var sortnum = new Set(array);
  return sortnum;
}

var array = [1,2,2,3,4,4,5]
console.log(array)
```
个人觉得应该是里面最简洁的方法实现去重了
使用的是es6的去重`Set`方法

`
var sortnum = new Set(array);
`
就可以直接返回这个数组