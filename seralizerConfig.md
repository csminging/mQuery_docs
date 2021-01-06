### 扩展
* $.ex  [object,[Object]]
    - 为串行器扩展函数，参数一为{扩展的函数名:需扩展的函数}，参数二是扩展的配置
```js

    // 以扩展mySl为例,
    var sl = function(num){
        sleep(num*1000)
        return true;
    }
    // 扩展
    $.ex({"mySl":sl})
    // 调用
    $("#id").click().mySl(3)
```


### 配置项
* addIndex [boolean]  default:true
    - 当调用成功时，内部记录的下标是否增加。记录的下标主要用作 then(number,function)所用
```js
        0        1         2     3
     $("#id").click().$("#id2").click().fail(1,function(){
        //  当第一个click失败时执行
     })
    // 记录下标的内部函数如下：
    // findColor、click、swipe
    // $() :$函数只有接受参数为 控件选择器或者boolean值 才会记录
                   0         1       2
     $("@color").click().$("#id2").click().fail(1,function(){
        //  当第一个click失败时执行
     })
```

* forceCall [boolean] default:false
    - 声明该函数是否需要被强制调用，不管之前的链条是否成功
```js
    // 以扩展mySlf为例,
    var slf = function(num){
        sleep(num*1000)
        return true;
    }
    // 扩展
    $.ex({"mySlf":slf},{
        forceCall:true
    })
    // 调用:不管前面是否成功，都会延时3秒
    $("#id").click().mySlf(3);
```
* isSuccess [Boolean/function]
    - 自定义函数成功/失败的标识。(链条的执行依据上一链条的是否成功来判定，当函数返回值比较特别时，需要自定义函数判定是否失败/成功)
    - 如isSuccess缺省，默认函数返回值为 ""/null/undefined/false时为失败
```js
    // 延时函数：无返回值，但这种功能应该是一直为成功的
    // 可返回true，或者定义isSuccess为true
    var sl = function(num){
        sleep(num*1000)
    }
    $.ex({"mySl":sl},{isSuccess:true})

    // 当返回值为数组时，自定义空数组为失败，非空数组为成功
    var findEle = function(txt){
        var eles = text(txt).find()
        return eles;
    }
     $.ex({"findEle":findEle},{isSuccess:function(res){
         return res&&res.length>0
     }})
```

* num 和time
    - 声明该函数在调用失败的时候，是否需要继续调用，直至成功或者超过num次，每次间隔time毫秒，当num和time均大于0时生效
```js
    //在之前的例子也可以看到 
 //为获取验证码功能额外添加重复获取，直到成功
  $.ex({
      getVcode:getVcode
  },{
      time:10000,
      num:20
  })


```
### 钩子

* before 参数为调用时的入参
    - 在函数执行前调用
```js
   //例如在执行click前都打印一下
   var myClick = function(){
       return  this.click()//1. 调用内置的click
       return  this.click.apply(this,arguments)//2. 如果需要完全调用，包括传参等。
   }
   $.ex({"myClick":myClick},{
       before:function(){
           console.log("click")
       }
   })
```

* success 参数: [执行结果，调用时的入参数组]
    - 在函数执行成功后调用
```js
   //例如在执行click成功后，延时一下
   var myClick = function(){
       return this.click()//调用内置的click
   }
   $.ex({"myClick":myClick},{
       success:function(){
           sleep(2000)
       }
   })
```

* fail 参数: [执行结果，调用时的入参数组]
    - 在函数执行失败后调用
```js
   //例如在执行click成功后，延时一下
   var myClick = function(){
       return this.click()//调用内置的click
   }
   $.ex({"myClick":myClick},{
       fail:function(){
           sleep(2000)
       }
   })
```

### 运行时配置
* 内部提供的函数，默认了一些配置项提供在全局配置。
* 除了$.setConfig({})修改全局配置外。所有串行器方法均可以在调用时传入参数，以覆盖上述参数/钩子
* 在函数被调用时，在最后的参数传入{_type:"_config"}表明其是配置项即可
```js
    /* 例如 全局配置的查找控件，在查找控件失败时，不会重复查找。
     selectorRepeat:{
         time:500,
         num:0
     }
     */
    // 假定#id这个控件比较特殊，出现的时间可能有延时，就想单独给这个控件加上重复查的功能
    $("#id",{
        _type:"_config",
        time:500,
        num:10
    }).click()
```
* 所有的串行器方法均可以在最后一个参数定义配置项/钩子，但需要注意，内置的函数 $()/click/等等尽量不要覆盖钩子，因为默认已经使用了部分钩子扩展部分功能。
