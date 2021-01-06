##### 简介
* 串行流程，这个我也不知道叫什么名字，随便起一个吧。
* 简单来说就是一个链条，中间断了，那么后面的操作都被跳过。

##### 功能
* 可用于替代大部分场景下的if else逻辑
* 为函数添加更灵活的功能和钩子

```js
    /*
    举个简单的例子，a操作--b操作--c操作--d操作
    需求：当前面的某一步操作失败，后面的操作需要自动跳过
    这里暂且先假定每个操作都回返回false/true
    */
    
    var a = function(){}
    var b = function(){}
    var c = function(){}
    var d = function(){}

    //框架实现
    var $ = require("./mQuery.js");
    //扩展方法到串行器上。
    $.ex({a:a,b:b,c:c,d:d})
    $().a().b().c().d();

    // if else 最简单的实现
    if(a()){
        if(b()){
            if(c()){
                d()
            }
        }
    }
    // 当然假定都返回false/true了，还有更简单的
    a()&&b()&&c()&&d();
    
    //但是在实际应用中，操作不一定返回true/false。
    // 返回一个空数组和非空数组。空数组等同false,非空等同true
    //类似场景或更复杂的场景下，那么原生的js就不会这么简单了。一堆的逻辑需要写
    //并且可以看到if-else的代码阅读性很差，而框架的逻辑一目了然。


```
##### 我们以一个完整注册机的示例来比对一下
* 流程：
    - 1. 获取一个手机号，假定  入参：无，返回值：{code:0,tel:"137XXXXXXXX"},失败返回：{code:1,msg:"失败"}
    - 2. 发送验证码，入参：{tel} ,返回值：{code:0,msg:"发送成功"}
    - 3. 获取验证码，入参：{tel},返回值：{code:0,vCode:"XXXX"}。每隔10秒获取一次，获取20次后默认作为失败(3分钟左右)
    - 4. 成功获取然后： 释放手机号：入参：{tel}
    - 5. 不成功则：拉黑手机号：入参：{tel}
    - 6. 注册 入参：{tel,vCode}，返回值：{code:0,msg:"注册成功"}

* 公共代码（框架和原生实现都一样的代码）
```js

    //我们假定这个函数是发送http的，入参为params,
    //返回值为http返回值
   var ajax = function(params){
       var res = httpxx(params) 
       return  res;
   }

    //获取手机号
   var getPhone = function(){
       var res = ajax()
       if(res.code==0){
           return res.tel
       }
   }
    //发送验证码
    var sendVCode= function(tel){
        var res = ajax({tel:tel})
        if(res.code==0){
            return true;
        }
    }
    //获取验证码。
    var getVcode =function(tel){
        var res = ajax({tel:tel})
        if(res.code==0){
            return res.vCode
        }
    }
    // 释放手机号
    var sfTel = function(tel){
        var res = ajax({tel:tel})
        if(res.code==0){
            return true;
        }
    }
    //拉黑手机号
    var lhTel = function(tel){
        var res = ajax({tel:tel})
        if(res.code==0){
            return true;
        }
    }
    // 注册
    var regTel = function(tel,vCode){
        var res = ajax({tel,vCode})
        if(res.code==0){
            return true;
        }
    }
```

* 原生js实现

```js
    var tel = "";
    var code = "";

    //获取手机号
    tel = getPhone()
    if(tel){
        // 发送验证码
        var flag = sendVCode(tel)
        if(flag){
            // 定时获取验证码
            for(var i =0;i<20;i++){
                sleep(10000)
                var res = getVcode(tel)
                if(res){
                    code = res;
                    break;
                }
            }
            // 如果获取到验证码
            if(code){
                //注册
                regTel(tel,code)
                // 释放手机号
                sfTel(tel)
            }else{
                // 拉黑手机号
                lhTel(tel)
            }
        }
    }
```

* 框架实现
```js
    var $ = require("./mQuery.js");
    //在扩展函数到串行器上
    $.ex({
        getPhone:getPhone,
        sendVCode:sendVCode,
        sfTel:sfTel,
        lhTel:lhTel,
        regTel:regTel
    })
    //为获取验证码功能额外添加重复获取，直到成功
    $.ex({
        getVcode:getVcode
    },{
        time:10000,
        num:20
    })

    var tel = "";
    var code = "";
    $tel =  getPhone()
    tel = $tel.res;
    $tel.sendVcode(tel).getVcode(tel);
    code = $tel.res;
    if(code){
        regTel(tel,code)
        sfTel(tel)
    }else{
        lhTel(tel)

    }
   
    // 或者，store()：将结果存至$.$d.xx下面，then:成功即调用，fail：失败即调用
    $()
    .getPhone().store("tel")
    .sendVCode($.$d.tel)
    .getVcode($.$d.tel).store("code")
    .then(function(res){
        //下面这两个直接调函数
        regTel($.$d.tel,$.$d.code);
        sfTel($.$d.tel)
        
    })
    .fail(function(){
        lhTel($.$d.tel)
    })
   
    /*
    then/fail/store都是内部提供的方法,
    then:上一步成功即调用，
    fail：上一步失败即调用，
    store：将结果存至全局变量上。;
    res也是内部的变量，是上一步执行的结果。
    */
```
`所以，串行流程管理器就是这么个东西，在某些场景下可以简化逻辑。当然提供的功能不止这些。后面会更具体地介绍`