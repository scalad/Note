### SpringMVC @RequestBody接收JSON参数

以前，一直以为在SpringMVC环境中，@RequestBody接收的是一个Json对象，一直在调试代码都没有成功，后来发现，其实 @RequestBody接收的是一个Json对象的字符串，而不是一个Json对象。然而在ajax请求往往传的都是Json对象，后来发现用 JSON.stringify(data)的方式就能将对象变成字符串。同时ajax请求的时候也要指定dataType: "json",contentType:"application/json" 这样就可以轻易的将一个对象或者List传到Java端，使用@RequestBody即可绑定对象或者List.

```javascript  
$(document).ready(function(){  
    var saveDataAry=[];  
    var data1={"userName":"test","address":"gz"};  
    var data2={"userName":"ququ","address":"gr"};  
    saveDataAry.push(data1);  
    saveDataAry.push(data2);         
    $.ajax({ 
        type:"POST", 
        url:"user/saveUser", 
        dataType:"json",      
        contentType:"application/json",               
        data:JSON.stringify(saveData), 
        success:function(data){ 
                                   
        } 
     }); 
});  
```

```Java
@RequestMapping(value = "saveUser", method = {RequestMethod.POST }}) 
@ResponseBody  
public void saveUser(@RequestBody List<User> users) { 
     userService.batchSave(users); 
} 
```