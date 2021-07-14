## 1、Emscripten简介
Emscripten 官方是这么描述的：

    Emscripten is a toolchain for compiling to asm.js and WebAssembly, built using LLVM, that lets you run C and C++ on the web at near-native speed without plugins.

翻译下来就是：

    Emscripten是一个工具链，作用是通过LLVM来编译生成asm.js、WebAssembly字节码，目的是让你能够在网页中接近最快的速度运行C和C++，并且不需要任何插件。

## 2、编译
官网上下载源码
~~~sh
# Fetch the latest version of the emsdk (not needed the first time you clone)
cd emsdk-main

# Download and install the latest SDK tools.
./emsdk install latest

# Make the "latest" SDK "active" for the current user. (writes .emscripten file)
./emsdk activate latest

# Activate PATH and other environment variables in the current terminal
source ./emsdk_env.sh
~~~
执行完后用下列命令进行测试
~~~sh
#emcc -v

emcc (Emscripten gcc/clang-like replacement + linker emulating GNU ld) 2.0.25 (4132d4ee02d5298fb88b80d1711ceb9d1d74950a)
clang version 13.0.0 (https://github.com/llvm/llvm-project 3644726a78e37823b1687a7aa8d186e91570ffe2)
Target: wasm32-unknown-emscripten
Thread model: posix
InstalledDir: /home/workspace/third_lib/ffmpeg/emsdk-main/upstream/bin
~~~

### 2.1、编译错误

#### 2.1.1、python版本问题
~~~sh
SyntaxError: invalid syntax
~~~
python解析错误。官网要求python版本不得低于3.6.于是卸载原来自带的python3.5，重新安装python3.9源码。

#### 2.1.2、库文件错误
~~~sh
ModuleNotFoundError: No module named '_ctypes'
~~~
**<font color=lightgray>sudo apt-get install libffi-dev</font>**