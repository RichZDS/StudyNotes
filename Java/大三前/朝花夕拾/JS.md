# HHHHH终于来到了js

===  绝对等于 长得一样 数据类型也要一样

== 等于 但是 如果字符串的 111 与int 的111 是相等的

console.log(X) ->打印X

NAN  not a number NAN===NAN false   只能通过isNAN(X)来判断

```js
        var person = {
            name: '张三',
            age: 30,
            greet: function() {
                console.log('你好，我是' + this.name);
            }
        };
```

## 	Arrylist

```js
arr.push( 可以填入多个)从后面   arr.unshift()从前面加
arr.pop()出去最后一个   arr.shift()从前面减去

```

## 属性 动态删除属性 

```js
delete peroson.name

person.hasdhoa="dasdsa" 就这么动态的增加属性了
```

## 所有键的类型都是用字符串表示

```js
"age" in person  true
"toString" in person true 还可以看父级的东西
person.hasOwnProperty()  看自身有的
```

## 循环

```
for (let n of arr){
输出聂荣
}
for(let n in arr){
输出下标
}
```

![image-20241123201823754](C:\Users\33882\AppData\Roaming\Typora\typora-user-images\image-20241123201823754.png)

![image-20241123201853842](C:\Users\33882\AppData\Roaming\Typora\typora-user-images\image-20241123201853842.png)

所有的变量都是绑定在window下 方法也是

## DOM

将html中的各个元素 封装成为了对象

![image-20241123215025953](C:\Users\33882\AppData\Roaming\Typora\typora-user-images\image-20241123215025953.png)