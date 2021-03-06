# $函数

没错，控件选择只有一个函数$()，id、className、depth、text、desc、find、findOne、findOnce、parent、children等等这些通通都被替代了，并且还扩充了更强大的控件选择器。



##### 下面将逐一介绍$函数在控件方面的使用。


# $
* {string,[object]}
第一个参数是字符串，以不同的符号开头表明不同的含义，用过jquery的，相信你一眼就看懂了。

* id选择器: 以 #  开头
```js
    //选择id为 btn 的控件
    $("#btn")
```

* className选择器: 以 *  开头
```js
    //选择className为 btn 的控件（由于className可以包含小数点，这里就以*了，避免和小数点混淆）
    $("*btn")
```

* depth选择器: 以 /  开头
```js
    //选择depth为 10 的控件,注意 / 后只能跟数字，如果带字母的话，那会把整个当text处理了。
    $("/10")
```

* desc选择器: 以 ~  开头
```js
    //选择desc为 描述 的控件
    $("~描述")
```

* text选择器:无特殊符号开头
```js
    //选择text为 我是文本 的控件
    $("我是文本")
```

* 后代选择器：空格
```js
    //选择id为btn的控件下面的"消息" 按钮。
    //假定页面上有两个文本为 "消息"的按钮，第一个的父/祖控件带有id  "btn",那么下面的例子只会选到第一个消息按钮
    $("#btn 消息")
```

* 多个选择器：,
```js
    //多个选择器以逗号分隔
    //将选择到 id=btn  和 文本= 消息  的控件,都会选择到。
    $("#btn,消息")
```

* 组合选择器: 以：$  开头
```js
    //将选择到 id=btn  并且 文本= 消息  的控件。
    //注意如果带有文本，文本需要放在最前面。
    $("$消息#btn")

    //其他示例，id和className同时带有才选择中
    $("$#btn*class.android")
```


* 父级选择器：选择器>>number    (以符号 >>  连着选择器和数字)
```js
    //选择id=btn的父控件,后面跟的数值代表向上多少层父级
    $("#btn>>1")
    //相当于  
    id("btn").findOnce().parent()

    $("#btn>>2")
    //相当于  
    id("btn").findOnce().parent().parent() 

    //以此类推
```

* 兄弟选择器： 选择器>>number>>选择器   或者   选择器>>>number>>选择器
```js
    /*主要用于筛选同级(拥有共同父/祖级控件)的选择器。
    例如：XX1控件下面有#a1控件和#c控件 , XX2控件下面有#b1控件和#c控件。
    这个时候如果想定位到 #b1下面的#c控件，就可以用到这个选择器。中间的数值为向上多少个父级查找*/

    $("#b1>>3>>#c")

    $("#b1>>>3>>#c")

    /*注意 上述  >>   和  >>>是有区别的
    >>：当number>1的时候，首先找到#b1控件，然后找到#b1控件的父级，这个父级找children:#c，
    如果找到了，直接返会#c, 找不到会继续找父级，然后这个父级找children:#c, 总共向上number层,类推。
    >>>：这个的话先找到#b1控件，然后直接向上number个parent()。然后找children:#c;

    所以当number>1的时候，>>可能只找到一个控件，而>>>有可能找到多个控件，具体看number和布局的关系。
    比较常用的应该是>> */
```

* 图色选择器： 以 @  开头
```js
    //这里了解一下就行了，后面会在图色分辨率篇章详细介绍，主要用作找图找色的。
    $("@color")
```

###### 上述选择器可以混用，只需要每个符合对应的逻辑即可（图色选择器除外，图色选择器和控件类的选择器只允许用逗号组合起来）

```js
    //例如：后代选择器和兄弟选择器、组合选择器混用

    $("#btn $消息#id>>1>>#c")

    //类同于以下：
    var btn = id("btn").findOnce();
    var xx = btn.find(text("消息").id("id"))
    var idParent = xx.parent()
    var c = idParent.find(id("c"))



    //反例:
    $("@color #id")
    //控件选择器和颜色选择器使用后代组合起来，完全没有意义。

    //正例：
    $("@color,#id").swipe()
    //将找图找色@color的位置滑动到控件#id所在位置


```

###### 注意：$("#id") 调用之后的返回值并非返回控件/控件数组列表。
###### 而是返回一个可以链式调用的实例对象。而选中的控件被挂载在对象下面。和数组类似，但非数组。
```js
    var $doms = $("#id");

    //是否获取到有控件，true/false
    console.log($doms.success)

    //获取到控件的数量
    console.log($doms.length)

    //和数组一样取到控件,例如取出第一个控件，但注意length==0的时候，取出来的是undefined
    console.log($doms[0])

    //也可以遍历
    for(var i =0;i<$doms.length;i++){
        console.log($doms[i])
    }

    //当然：在大多数场景下都不需要用到这些，因为框架已经封装了click、swipe等函数配合使用。
    $("#a").click().$("#b").click().$("#c").click().$("#d1,#d2").swipe();
    //click:函数将点击选择器的第一个控件，swipe：函数将 把#d1控件滑动到#d2控件位置。
    //链式调用：上述如果在某一个步骤中断，都将导致后面的所有操作直接跳过。

    $("#a").click();
    $("#b").click();
    $("#c").click();
    $("#d1,#d2").swipe();
    //和这样调用有很大的区别，这里某个控件点击不成功，即中途一步不成功，后续依然会执行。
    //这里简单描述一下，后面的篇章会重点介绍这个。


```