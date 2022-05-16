## 1、Javascript

### 1、**定时器**
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


### 2、**页面跳转问题**

~~~js
top.window.location = 'main.html';
window.location.herf = 'main.html';
~~~
上面两句跳转指令在不同浏览器中存在兼容问题。google浏览器可以正常跳转，火狐就不能正常跳转。
需要额外添加一句

~~~js
    top.window.location = 'main.html';
    window.event.returnValue=false;
~~~