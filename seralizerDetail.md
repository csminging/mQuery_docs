##### 串行器对象
* $()函数将返回一个串行器的实例对象，这个对象的方法可以链式调用下去。当链条中断时，后续直接跳过。

##### 属性
* success 
    - 链条的执行状态。链条最终是否执行成功
```js
    //是否找到id控件
    $("#id").success   

    //点击id控件是否成功
    $("#id").click().success   

    //#id2是否点击成功（注：#id点击不成功，后面不会执行，success=false)
    $("#id").click().$("#id2").click().success
```
* res
    - 当前链条步骤的执行结果

```js
    var $ = require("mQuery.js")
    var getPhone = function(){
        return "137XXXX"
    }
    $.ex({getPhone:getPhone})
    $().getPhone().res;//得到137XXXX

    // 注：$("#id").res;选择器的res，有可能和结果不全等，
    // 例如 inScreen 参数，和 filter函数会影响结果。


```


##### 函数

* 所有串行器的对象均具备以下方法

* then  [[index],function]
    - 当链条上一个步骤执行成功后，那么then将被执行，用于执行一片代码

```js
    $("#id").click()
    .then(function(res){
        // res为上一链条的执行结果
        // xxxx
        // xxxxx
    })

    // 等同于
    var $res = $("#id").click()
    if($res.success){
        // xxx
        // xxxx
    }

    // 当传入index 参数时
    // 0       1         2       3 
    $("#id").click().$("#id2").click().then(1,function(res){
        // res为第一个click的执行结果
        // xxx
        // xxxx
    })
    // 执行顺序依然按照链条的顺序执行。但then是否执行，将根据第一个click()是否成功来判定
    // 串行器会记录链条的下标，当然在某些情况下是不记录的，后面会有详细介绍。
    // 这种情况应该也比较少用

```

* fail  [[index],function]
    - 当链条上一个步骤执行失败的时候，fail将被执行
```js
    //  fail的执行仅仅是在click失败的时候执行，当$("#id")查找控件导致的失败，并不会执行fail
    $("#id").click()
    .fail(function(){
        // xxxx
        // xxxxx
    })

    // 注意：上述代码并不等同于下面。
    var $res = $("#id").click()
    if(!$res.success){
        // xxx
        // xxxx
    }
```

* fails  [function]
    - 当链条之前步骤执行失败的时候，fails将被执行
```js
    // 不管是$("#id")导致的失败，还是click()导致的失败，都将执行fails
    $("#id").click()
    .fails(function(){
        // xxxx
        // xxxxx
    })

    //等同于
    var $res = $("#id").click()
    if(!$res.success){
        // xxx
        // xxxx
    }
```

* final  [function]
    - 无论之前的执行结果怎样，final都将被执行
```js
    
    $("#id").click()
    .final(function(){
        // xxxx
        // xxxxx
    })
```

* store [String]
    - 将当前链条执行结果存至 $.$d 上

```js
    $("#id").click().store("isSuccess")
    console.log($.$d.isSuccess)
```

##### 其他内置的串行器函数

* findColor 
    - 多点找色，
```js
   var c =  $("@color").findColor()
   
    //如果找色成功，可以这样拿坐标。当然这比较少用
    if(c.success){
        var x = c[0].x
        var y = c[0].y
    }
```


* click [Object]
    - 点击控件/颜色点
```js
    // 1.按坐标点击
    $().click({
        x:10,
        y:10
    })
    // 2.按控件点击
    $("#id").click()
    // 3.按多点找色点击
    $("@color").click()
```

* longClick [object]
```js
      // time 不传默认是2-3秒
    $("#id").longClick({
        time:5000
    })
```

* swipe
    - 滑动
```js
    $("#id,#id1").swipe({
        from:{
            x:100,
            y:100
        },
        to:{
            x:10,
            y:10
        },
        time:1000
    })
    //time 不传默认1秒
    // to/from的坐标受全局配置参数：transformPointScreen 影响，是否自动转换不同分辨率坐标
    // 当to或者from 缺省时，将按顺序从$("#id,#id1")选中的控件/多点找色，自动补充进来to/from
    $("#id,#id1").swipe();//从#id控件滑动到#id1控件位置。
    $("#id,@color").swipe();//从#id控件滑动到多点找色位置。
    $("#id,@color").swipe({from:{x:10,y:10}});//从（10,10)位置滑动到#id控件位置
```

* sl  [number,[number2]]
    - sleep多少秒，注意：参数单位是秒

```js
    $("#id").click().sl(2.5,5);//在点击成功后延时2.5~5秒
```

* slf [number,[number2]] 
    - 同sl 一致，但有区别

```js
    $("#id").click().slf(2.5);//不管前面链条是否执行成功，都将延时2.5秒
```