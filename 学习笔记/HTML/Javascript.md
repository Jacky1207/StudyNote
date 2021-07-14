## 1、Javascript

### 1、定时器
    var id = setInterval(func, timeout); //设置定时器，返回定时器id
    clearInterval(id);                   //清空定时器
~~~js
var id = -1;
function getSystemTime(){
    var dateObj = new Date();
    var date = dateObj.getFullYear() + "-" + (dateObj.getMonth() + 1) + "-" + dateObj.getDate() + " " + dateObj.getHours() + ":" + dateObj.getMinutes() + ":" +dateObj.getSeconds();
    document.getElementById("datetime").value = date;
}
setInterval(getSystemTime, 1000);
~~~
间隔1秒执行一次getSystemTime函数。

## jQuery

### input焦点

~~~js
$('#datetime').focus(function(){
    clearInterval(id);
});
$('#datetime').blur(function(){
    id = setInterval(getSystemTime, 1000);
    getSystemTime();
});
~~~
    focus 获取焦点事件
    blur 失去焦点事件