
---
title: ajax 下载文件
date: 2018-03-07 19:29:44
tags: [js]
categories: js
---

原本现在文件直接通过超链接可以完成下载，但现在要在url中附带几个参数，并且这些参数要是点击事件触发时的最新值，所以这里使用ajax的方式进行下载

然而：

>使用ajax，ajax的返回值类型是`json`,`text`,`html`,`xml`类型，或者可以说ajax的发送，接受都只能是string字符串，不能流类型，所以无法实现文件下载，强用会出现response冲突。
如果非要使用ajax的话，只能通过返回值得到生成的文件相关url。然后在回调函数里通过创建一个iframe，并设置其src值为文件url，或者一个对文件生成流的处理url，这样操作来实现文件下载且页面无刷新。

>不使用ajax，通过dom动态操作或创建iframe,form的方式来实现，在下载文件的同时实现页面不刷新，其中iframe的src可以是文件地址url来直接下载文件，也可以是流处理url通过response流输出下载，form的是流处理url通过response流输出下载，dom动态操作的时候实现文件下载，且页面无刷新。


这里就是用两两种操作dom方式实现下载：

- 操作iframe方式：
```js
function ajaxDownload(urlPost,data){
     $.ajax({
         url: urlPost,
         type: "POST",
         cache: false,
         data:data,
         beforeSend:function(){
             $("#grid_crud").pqGrid("showLoading")
         },
         success: function(filename) {
             var url = urlPost + (((urlPost.indexOf("?") > 0) ? "&" : "?") + $.param(data));
             $(document.body).append("<iframe height='0' width='0' frameborder='0'  src=" + url + "></iframe>")
         },
         complete:function(data){
             $("#grid_crud").pqGrid("hideLoading")
         },
         error:function(a,b,c){
             $("#grid_crud").pqGrid("hideLoading")
         }
     });
}
```

- 操作form形式：
```js
function ajaxDownload(url,data,method){
    if(url && data){
        // data 是 string 或者 array/object
        data = typeof data =='string'?data:jQuery.param(data);
        var inputs ='';
        jQuery.each(data.split('&'),function(){
            var pair = this.split('=');
            inputs +='<input type="hidden" name="'+pair[0]+'" value="'+pair[1]+'"/>';
        });
        jQuery('<form action="'+url+'" method="'+(method || 'post')+'">'+inputs+'</form>').appendTo('body').submit().remove();
    }
}
```