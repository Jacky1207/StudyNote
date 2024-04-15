<!--
 * @Author: Jacky
 * @Date: 2022-05-19 15:20:21
 * @LastEditors: Jacky
 * @LastEditTime: 2022-08-22 13:37:14
 * @FilePath: \StudyNote\VSCode.md
-->
# vscode 使用记录
## 1、代码注释koro1FileHeader

- ctrl+alt+j: 生成文件头注释
- ctrl+alt+t: 生成函数注释
- shift+ctrl+p: 输入> 注释图案 或者是 > codeDesign就可以选择注释图案了，
  快捷键shift+ctrl+p 输入> 随机 ，选中并回车即可随机生成一个注释图案。

图案列表：
~~~
'random', // 随机
'buddhalImg', // 佛祖
'buddhalImgSay', // 佛祖+佛曰
'buddhalSay', // 佛曰
'totemDragon', // 龙图腾
'belle', // 美女
'coderSong', // 程序员之歌
'loitumaGirl', // 甩葱少女
'keyboardAll', // 全键盘 
'keyboardSmall', // 小键盘
'totemWestDragon', // 喷火龙
'jesus', // 耶稣
'dog', // 狗
'grassHorse', // 草泥马
'grassHorse2', // 草泥马2 
'totemBat', // 蝙蝠
~~~

## 2、插件

### 2.1 C/C++ GNU Global
解决函数、参数跳转缓慢或无法跳转的问题。

- 下载 gnuGlobal windows版
- 配置json文件

"gnuGlobal.globalExecutable": "D:\Program Files\glo665wb\bin\global.exe",
"gnuGlobal.gtagsExecutable": "D:\Program Files\glo665wb\bin\gtags.exe",

- Ctrl+Shift+P 运行  Global:Rebuild Gtags Database


## 3、XHR failed
安装插件时报错，进入开发者选项查看 Failed to load resource: net::ERR_CERT_AUTHORITY_INVAL_ID

重新安装vscode 还是存在。

解决思路：
上述错误的意思是证书无效，尝试直接在google浏览器打开 : https://marketplace.visualstudio.com 可以看到左上角有一个红色的按钮，
显示网站不安全。

点击进入，显示证书无效。在证书路径中查看，显示信息是没有添加到 ##受信任的根证书颁发机构## ，问题就很明显了，因为vscode的
商店并不在受信任的证书里面，所以访问被拒绝了。然后就是导出 https://marketplace.visualstudio.com 这个网站的证书，在导入到本地根证书中。就可以了
