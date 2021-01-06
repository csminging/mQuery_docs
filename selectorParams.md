# $
* 在选择器，我们已经简单了解到了$函数的的一种用法。这里将详细介绍其他方面的用法。

###### 1. {string,[object]}
* string
    - 在选择器篇章已经介绍了这个参数，传进来字符串选择器

* object[选填]
    - inScreen:[Bolean],决定控件查找的范围，true:只有在屏幕内的控件才被选中。（覆盖全局配置)
    ```js
        var $ = require("./mQuery.js");
        //例如全局配置
        $.setConfig({
            inScreen:false
        })

        $("#a").click()
        .$("#b",{inScreen:true}).click()
        .$("#c").click();
        //我就是想在找#b控件的时候只找屏幕内的，其他不管。
    ```
    - filter:[function(item,i)],当内置的选择器无法满足时使用，把$(string)选择到的控件逐个传进filter函数
    
    返回值：Boolean/uiObject/[uiObject,uiObject...]
    ```js
        //假定页面上有4个id的按钮。仅演示，实际有些直接用内置选择器就行
        //1.筛选偶数位置的按钮
        $("#id",{
            filter:function(item,i){
                if(i%2==0){
                    return true
                }
            }
        })

        //2.筛选text="消息"的按钮,相当于 $("$消息#id")
        $("#id",{
            filter:function(item){
                if(item.text()=="消息"){
                    return item;
                }
            }
        })

        //3.把控件和控件的父级都拿到手,相当于$("#id,#id>>1")
        $("#id",{
            filter:function(item){
                return [item,item.parent()]
            }
        })

    ```

 ###### 2. {Boolean}
 ```js
    //当接受的参数为boolean值时，多用于链式调用的流程控制上
    $("#a").click().$($("#b,#c").success).$("#d").click()
    //含义：点击a按钮之后，判断页面上有没有找到#b按钮或者#c按钮，找到其中之一，那么找#d按钮并点击。
    //同样的也可以用&&链接，代表需要同时找到#b和#c，才执行下去
    $($("#b").success&&$("#c").success)
    //这里还不了解关于链式调用的话：看完后面的篇章再回来看这就懂了
 ```

 ##### 3. { $实例 | uiObject | [uiObject,uiObject...] }
 ```js
    //传进来$实例或者控件实例或者控件实例的数组，都将得到新的$实例，并且参数中的控件被挂在到这个新实例上面。

    //1.返回空的$实例
        $();
    //2.参数的控件挂到新的实例上，而其他内容都被舍弃。
        $($("#id")).click()
    //3.当控件文本和选择器冲突的时候（控件文本="#我是文本"，和id选择器冲突），这个时候不能直接使用选择器，那么就用原生的text
        $(text("#我是文本").findOnce()).click()
 ```