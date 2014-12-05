title: 搞定JavaScript 内存泄露
speaker: mdemo

[slide data-transition="kontext"]

# 搞定 JavaScript 内存泄露
----
<style type="text/css">
iframe{
    background: transparent;
}
.cases .user {
    margin-right: 80px;
    min-width: 40%;
}
.case {
    float: left;
}

.cases {
    margin-top: 60px;
    zoom: 1;
}
</style>


<div class="cases">
        <div class="case user">
          <h2>By mdemo</h2>
          <p>GitHub profile</p>
          <div class="github-card" data-user="demohi"></div>
        </div>
        <div class="case repo">
          <h2>Repo Card</h2>
          <p>GitHub repository</p>
          <div class="github-card" data-user="demohi" data-repo="memory-leaks">></div>
        </div>
      </div>
<script src="http://lab.lepture.com/github-cards/widget.js"></script>

[slide data-transition="vkontext"]

# 内存泄露？
## JavaScript不是具有自动垃圾回收机制吗？

[slide data-transition="circle"]
## 先了解一下背景
----
* JavaScript 变量的内存占用 {:&.moveIn}
* chrome profile 工具
* GC 回收原理

[slide data-transition="cover-circle"]
## 5种基本类型
----
* 数字(Numbers) (如 3.14159..) {:&.moveIn}
* 布尔值(Booleans) (true或false)
* 字符型(Strings) (如 'mdemo')
* Null
* Undefined
* > 它们不会引用别的值，它们只会是叶子节点或终止节点。

[slide data-transition="cover-diamond"]
## 动态的属性
----
```
typeof '111'; 
typeof new String('111')
var s = '123';
var s2 = new String('123');
s.color = 'red';
s2.color = 'red';
console.log(s.color);
console.log(s2.color);
```

* string {:&.moveIn}
* object
* undefined
* red

[slide data-transition="earthquake"]
## 动态的属性
----
```
var s1 = 'mdemo';
s1.color = 'red';
var s2 = s1.substring(2);

var s3 = new String('mdemo);
s3.color = 'red';
s3 = null;

var s4 = new String('mdemo);
var s2 = s4.substring(2);
s4 = null;
```

[slide data-transition="cards"]
## 复制变量值
----
```
var num1 = 1;
var num2 = num1;
num1 = 3;
console.log(num2);
var obj1 = new Object();
var obj2 = obj1;
obj1.name = 'mdemo';
console.log(obj2.name);
```

* 1 {:&.moveIn}
* mdemo 

[slide data-transition="glue"]

## 传递参数
----
```
function setName(obj){
    obj.name = 'mdemo';
}
var person = new Object();
setName(person);
console.log(person.name);
```
```
function setName(obj){
    obj.name = 'mdemo';
    obj = new Object();
    obj.name = 'demohi';
}
var person = new Object();
setName(person);
console.log(person.name);
```
* mdemo {:&.moveIn}
* mdemo 

[slide data-transition="glue"]

## 传递参数
----
```
function setName(obj){
    obj.name = 'mdemo';
    obj2 = obj;
    obj2.name = 'demohi';
}
var person = new Object();
setName(person);
console.log(person.name);
```
* demohi {:&.moveIn}

[slide data-transition="stick"]

## 对象删除
----
```
var obj1 = new Object();
var obj2 = obj1;
obj1.name = 'mdemo';
obj1 = null;
console.log(obj2.name);
```
* mdemo {:&.moveIn}
[slide data-transition="move"]
## 内存中的对象
----
![](http://boke.io/content/images/2014/12/objectsizes-1.png)

* 如果我们把内存想象成一个包含基本类型(像数字和字符串)和对象(关联数组)的图表。它可能看起来像下面这幅一系列相关联的点组成的图。 {:&.moveIn}
* 对象自身直接使用
* 隐含的保持对其它对象的引用，这种方式会阻止垃圾回收(简称GC)对那些对象的自动回收处理。

[slide data-transition="newspaper"]
## 内存中的对象
----
![](http://boke.io/content/images/2014/12/profile.png)

* 直接占用内存(Shallow Size，不包括引用的对象占用的内存) 
* 占用总内存(Retained Size，包括引用的对象所占用的内存)

[slide data-transition="slide"]
## GC 回收
----
![](http://boke.io/content/images/2014/12/gc.png)

* 内存图从 root 开始，root 可以理解是浏览器中的 window 对象，Node.js 中的 Global 对象 {:&.moveIn}
* 9与10无法与 root 相连，将会被 GC 回收掉
* 节点离 root 的最近距离被称为distance，如下图可以在Heap Profiler中查看到
* 上图2的 distance 为1，8的 distance 为4

[slide data-transition="slide3"]
## GC 回收
----
![](http://boke.io/content/images/2014/12/distance.png)

[slide data-transition="horizontal3d"]
# 准备一下调试环境
## 最新版的 chrome + 隐私模式
----
* 最新版的 Chrome 可以拥有最佳的 devtools {:&.moveIn}
* 隐私模式是为了避免插件等的干扰（调试的时候，关闭所有插件在隐私模式下开启）

[slide data-transition="horizontal"] 
# 工欲善其事,必先利其器
##  学习使用 chrome devtools
----
* 使用 timeline发现内存泄露 {:&.moveIn}
* 使用 timeline 查看 GC Event 耗时
* 使用Heap Snapshot查看详细内存情况
* 使用  Heap Allocations

[slide data-transition="vertical3d"]
## 使用 timeline发现内存泄露
----

[Example 1: Watch the memory grow](http://demohi.github.io/memory-leaks/examples/1-watch-the-memory-grow.html)

![](http://boke.io/content/images/2014/12/Snip20141203_22.png)

[slide data-transition="zoomin"]
## 使用 timeline 查看 GC Event 耗时
----
[Example 2: Watching the GC work](http://demohi.github.io/memory-leaks/examples/2-watching-the-GC-work.html)


![](http://boke.io/content/images/2014/12/Snip20141203_23.png)

[slide data-transition="zoomout"]
## 使用Heap Snapshot查看详细内存情况

###Summary 视图
----
[Example 3: Scattered objects](http://demohi.github.io/memory-leaks/examples/3-scattered-objects.html)


![](http://boke.io/content/images/2014/12/Snip20141203_24.png)
[slide data-transition="pulse"]
## 使用Heap Snapshot查看详细内存情况
### Compare 视图
----
[Example 11:Verifying Action Cleanness](http://demohi.github.io/memory-leaks/examples/11-compare.html)


![](http://boke.io/content/images/2014/12/Snip20141203_25.png)
[slide]
##  使用  Heap Allocations
----
[Example 6: Leaking DOM nodes](http://demohi.github.io/memory-leaks/examples/6-leaking-DOM-nodes.html)

![](http://boke.io/content/images/2014/12/Snip20141203_31.png)

[slide]
##  使用  Heap Allocations
----
* 每条竖线代表一次新的内存生成 {:&.moveIn}
* 灰色的竖线说明内存被回收
* 蓝色的竖线说明生成后，没有被回收，有可能是内存泄露
* 可以选定特定的蓝线，查看这次生成的内容，方便查找内存泄露问题

[slide]
## 实战
----
* Eval is evil (almost always) {:&.moveIn}
* dom引起的内存泄露

[slide]
## Eval is evil (almost always)
----
```
      function createEvalClosure() {
        var smallStr = 'x';
        var largeStr = new Array(1000000).join('x');
        return function eC() {
          eval('');
          return smallStr;
        };
      }
```

[slide]
## Eval is evil (almost always)
----
[Example 7: Eval is evil (almost always)](http://demohi.github.io/memory-leaks/examples/7-eval-is-evil-almost-always.html)
![](http://boke.io/content/images/2014/12/Snip20141203_27.png)

[slide]

## dom清空的内存泄露
----
![](http://staticc.qiniudn.com/detached-nodes.gif) 

* 黄色是指直接引用，红色是指间接引用 {:&.moveIn}
* remove refB 变成黄色，由于有 js 直接引用
* refB=null 后，变成红色，由于 refA 对 refB 间接引用


[slide]

## dom清空的内存泄露
----
[Example 9: DOM leaks bigger than expected](http://demohi.github.io/memory-leaks/examples/9-DOM-leaks-bigger-than-expected.html)

![](http://boke.io/content/images/2014/12/Snip20141203_34.png)

[slide]

## 总结
----
* 一个小的 dom 引用，会导致巨大的内存泄露
* 谨慎使用全局变量，尤其是全局变量存在 dom 引用
* 即使手动清空
* chrome 是神器


[slide]
## 相关链接
----
* [文中所有 example 源码](https://github.com/demohi/memory-leaks/tree/gh-pages/examples)

* [example 索引](http://demohi.github.io/memory-leaks/)

[slide]
## 参考的页面&致谢
----
* [Chrome 开发者工具中文手册，感谢良心翻译](https://github.com/CN-Chrome-DevTools/CN-Chrome-DevTools)

* [chrome 官方文档需翻墙](https://developer.chrome.com/devtools/docs/javascript-memory-profiling)

* [App之性能优化](http://www.cnblogs.com/stenson/p/3951035.html)

* [Understand memory leaks in JavaScript applications](http://www.ibm.com/developerworks/library/wa-jsmemory/index.html)

* [JavaScript Profiling With The Chrome Developer Tools](http://www.smashingmagazine.com/2012/06/12/javascript-profiling-chrome-developer-tools/)

* [Memory leaks](http://javascript.info/tutorial/memory-leaks)
* 刘指导 & 张小俊



[slide]
## 广告时间---加入进来
----

* [ 已经三个月大的移动前端技术周刊](https://github.com/mobframe/mobile-front-end-info-collection/issues)

* [文章是最好的积累---boke.io](http://boke.io)

![](http://bokeio.qiniudn.com/Snip20141204_48.png)












