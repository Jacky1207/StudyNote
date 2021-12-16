## HTML笔记

###  1、输入框 Input
####  1.1、限制输入框只输入数字
~~~html
<input class="cs-text-short" type="number" name="types" id="contrast_level">
~~~
将type指定成<font color=red>**number**</font>即可。

####  1.2、限制数字输入的范围
~~~html
<input class="cs-text-short" type="number" name="types" id="contrast_level" min="0" max="9">
~~~
min属性表示最小值、max表示最大值。（input类型必须是number）

####  1.3、显示时间日期
| 属性 | 描述 | 
| :-----:| :---- |
| 值 | 这是HTML里input元素的通用属性。就是输入框里的数据。 |
| min | 日期或时间的最小值 |
| max | 日期或时间的最大值 |
| step | 步长。不同的类型有不同的缺省步长<br>* Date – 缺省是1天<br>* Week – 缺省是1周<br>* Month – 缺省是1月<br>* Time – 缺省是1分钟<br>* DateTime – 缺省是1分钟<br>* Local DateTime – 缺省是1分钟 |
|||

### 2、乱码问题

需要在<head>中加入以下一句
~~~html
	<meta charset="UTF-8">
~~~