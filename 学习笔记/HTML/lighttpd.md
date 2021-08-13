# Lighttpd + fastcgi
 
## 编译
### 1、pcre-8.44
~~~sh
./configure --host=aarch64-himix100-linux prefix=/home/workspace/third_lib/libpcre_arm
~~~

<font color=red>**--host=aarch64-himix100-linux**</font>这里的配置如果改成--host=arm CC=xxx，会导致只能编译静态库，无法生成动态库
### 2、openssl-1.1.1d
~~~sh
./config shared no-async no-asm --prefix=$(pwd)/../openssl_arm --cross-compile-prefix=aarch64-himix100-linux-

#还需要屏蔽Makefile中 -m64 的选项
#CNF_CFLAGS=-pthread -m64
#CNF_CXXFLAGS=-std=c++11 -pthread -m64
~~~
### 3. lighttpd-1.4.59
~~~sh
./configure --prefix=/home/workspace/third_lib/lighttpd_arm --with-openssl --with-openssl-includes=/home/workspace/third_lib/openssl_arm/include --with-openssl-libs=/home/workspace/third_lib/openssl_arm/lib --with-pcre --host=aarch64-himix100-linux PCRECONFIG=/home/workspace/third_lib/libpcre_arm/bin/pcre-config --without-zlib
~~~
### 3. fastcgi
#### 1、 fcgi
C语言版本的fastcgi库
#### 2、 fastcfipp
C++版本的fastcgi库
### 3. spawn-fcgi
fcgi 进程管理服务（lighttpd 配置成转发模式，将fcgi消息转给spawn）
## 配置
### 1. fastcgi 配置
    fastcgi.conf中配置

server.modules += ( "mod_fastcgi" )及在module.conf中配置include "conf.d/fastcgi.conf"

    【fastcgi配置选项】

lighttpd通过fastcgi模块的方式实现了对fastcgi的支持，并且在配置文件中提供了三个相关的选项：

    1 fastcgi.debug
    
可以设置一个从0到65535的值，用于设定FastCGI模块的调试等级。当前仅有0和1可用。1表示开启调试（会输出调试信息），0表示禁用。例如：
fastcgi.debug = 1

    2 fastcgi.map-extentsions

同一个fastcgi server能够映射多个扩展名，如.php3和.php4都对应.php。例如：
fastcgi.map-extensions = ( ".php3" => ".php" )
or for multiple
fastcgi.map-extensions = ( ".php3" => ".php", ".php4" => ".php" )

    3 fastcgi.server

这个配置是告诉Web Server将FastCGI请求发送到哪里，其中每一个文件扩展名可以处理一个类型的请求。负载均衡器可以实现对同一扩展名的多个对象的负载均衡。

    fastcgi.server的结构语法如下：  

    ( <extension> =>
    ( [ <name> => ]
    ( # Be careful: lighty does *not* warn you if it doesn't know a specified option here (make sure you have no typos)
      "host" => <string> ,
      "port" => <integer> ,
      "socket" => <string>,                 # either socket or host+port
      "bin-path" => <string>,               # optional
      "bin-environment" => <array>,         # optional
      "bin-copy-environment" => <array>,    # optional
      "mode" => <string>,                   # optional
      "docroot" => <string> ,               # optional if "mode" is not "authorizer"
      "check-local" => <string>,            # optional
      "max-procs" => <integer>,             # optional - when omitted, default is 4
      "broken-scriptfilename" => <boolean>, # optional
      "kill-signal" => <integer>,           # optional, default is SIGTERM(15) (v1.4.14+)
    ),
    ( "host" => ...
    )
    )
    )
~~~
<extentsion>：文件名后缀或以”/”开头的前缀（也可为文件名）
<name>：这是一个可选项，表示handler的名称，在mod_status中用于统计功能，可以清晰的分辨出是哪一个handler处理了<extension>。
host：FastCGI进程监听的IP地址。此处不支持hostname形式。
port：FastCGI进程所监听的TCP端口号
bin-path：本地FastCGI二进制程序的路径，当本地没有FastCGI正在运行时，会启动这个FastCGI程序。
socket：unix-domain-socket所在路径。
mode：可以选择FastCGI协议的模式，默认是“responder”，还可以选择authorizer。
docroot：这是一个可选项，对于responder模式来讲，表示远程主机docroot；对于authorizer模式来说，它表示MANDATORY，并且指向授权请求的docroot。
check_local：这是一个可选项，默认是enable。如果是enable，那么server会首先在本地（server.document-root）目录中检查被请求的文件是否存在，如果不存在，则给用户返回404（Not Found），而不会把这个请求传递给FastCGI。如果是disable，那么server不会检查本地文件，而是直接将请求转发给FastCGI。（disable的话，server从某种意义上说就变为了一个转发器）
broken-scriptfilename：以类似PHP抽取PATH_INFO的方式，抽取URL中的SCRIPT_FILENAME。
如果bin-path被设置了，那么：
max-procs：设置多少个FastCGI进程被启动
bin-environment：在FastCGI进程启动时设置一个环境变量
bin-copy-environment：清除环境，并拷贝指定的变量到全新的环境中。
kill-signal：默认的话，在停止FastCGI进程时，lighttpd会发送SIGTERM(-15)信号给子进程。此处可以设置发送的信号。
~~~
【举例】

    fastcgi.debug = 1                        
    fastcgi.server = 
    ( ".fcgi" =>                               
        ( "local" =>
            (            
                "socket" => "/app/sd/fastcgi.socket",
                #"host" => "127.0.0.1",     //用于分布式部署，可以指定其他服务器IP
                #"port" => 9000,                         
                "bin-path" => "/app/uiapp/lighttpd/echo.fcgi",
                "max-procs" => 1,                                               
                "check_local" => "disable",
            ),                 
        ),            
    )   
### 2. lighttpd 配置
    lighttpd.conf:
~~~
######################################################################
##
## /etc/lighttpd/lighttpd.conf
##
## check /etc/lighttpd/conf.d/*.conf for the configuration of modules.
##
#######################################################################

#######################################################################
##
## Some Variable definition which will make chrooting easier.
##
## if you add a variable here. Add the corresponding variable in the
## chroot example aswell.
##
var.log_root    = "/app/private/lighttpd"
var.server_root = "/app/uiapp/lighttpd"
var.state_dir   = "/app/uiapp/lighttpd"
var.home_dir    = "/app/uiapp/lighttpd"
var.conf_dir    = "/app/uiapp/lighttpd/conf"

## 
## run the server chrooted.
## 
## This requires root permissions during startup.
##
## If you run Chrooted set the the variables to directories relative to
## the chroot dir.
##
## example chroot configuration:
## 
#var.log_root    = "/logs"
#var.server_root = "/"
#var.state_dir   = "/run"
#var.home_dir    = "/lib/lighttpd"
#var.vhosts_dir  = "/vhosts"
#var.conf_dir    = "/etc"
#
#server.chroot   = "/srv/www"

##
## Some additional variables to make the configuration easier
##

##
## Base directory for all virtual hosts
##
## used in:
## conf.d/evhost.conf
## conf.d/simple_vhost.conf
## vhosts.d/vhosts.template
##
var.vhosts_dir  = server_root + "/vhosts"

##
## Cache for mod_deflate
##
## used in:
## conf.d/deflate.conf
##
var.cache_dir   = "/var/cache/lighttpd"

##
## Base directory for sockets.
##
## used in:
## conf.d/fastcgi.conf
## conf.d/scgi.conf
##
var.socket_dir  = home_dir + "/sockets"

##
#######################################################################

#######################################################################
##
## Load the modules.
include "modules.conf"

##
#######################################################################

#######################################################################
##
##  Basic Configuration
## ---------------------
##
server.port = 80

##
## Use IPv6?
##
server.use-ipv6 = "disable"

##
## bind to a specific IP
##
#server.bind = "localhost"

##
## Run as a different username/groupname.
## This requires root permissions during startup. 
##
#server.username  = "lighttpd"
#server.groupname = "lighttpd"

##
## Enable lighttpd to serve requests on sockets received from systemd
## https://www.freedesktop.org/software/systemd/man/systemd.socket.html
##
#server.systemd-socket-activation = "enable"

## 
## enable core files.
##
#server.core-files = "disable"

##
## Document root
##
server.document-root = server_root + "/www/"

##
## The value for the "Server:" response field.
##
## It would be nice to keep it at "lighttpd".
##
#server.tag = "lighttpd"

##
## store a pid file
##
#server.pid-file = state_dir + "/lighttpd.pid"

##
#######################################################################

#######################################################################
##
##  Logging Options
## ------------------
##
## all logging options can be overwritten per vhost.
##
## Path to the error log file
##
server.errorlog             = log_root + "/error.log"

##
## If you want to log to syslog you have to unset the 
## server.errorlog setting and uncomment the next line.
##
#server.errorlog-use-syslog = "enable"

##
## Access log config
## 
#include "conf.d/access_log.conf"

##
## The debug options are moved into their own file.
## see conf.d/debug.conf for various options for request debugging.
##
include "conf.d/debug.conf"

##
#######################################################################

#######################################################################
##
##  Tuning/Performance
## --------------------
##
## corresponding documentation:
## https://redmine.lighttpd.net/projects/lighttpd/wiki/Docs_Performance
##
## set the event-handler (read the performance section in the manual)
##
## The recommended server.event-handler is chosen for each OS, if available.
##
## epoll  (recommended on Linux)
## kqueue (recommended on *BSD and MacOS X)
## solaris-devpoll (recommended on Solaris)
## poll   (recommended if none of above are available)
## select (not recommended)
## libev  (not recommended)
##
#server.event-handler = "linux-sysepoll"

##
## The basic network interface for all platforms at the syscalls read()
## and write(). Every modern OS provides its own syscall to help network
## servers transfer files as fast as possible 
##
## sendfile       - is recommended for small files.
## writev         - is recommended for sending many large files
##
#server.network-backend = "sendfile"

##
## As lighttpd is a single-threaded server, its main resource limit is
## the number of file descriptors, which is set to 1024 by default (on
## most systems).
##
## If you are running a high-traffic site you might want to increase this
## limit by setting server.max-fds.
##
## Changing this setting requires root permissions on startup. see
## server.username/server.groupname.
##
## By default lighttpd would not change the operation system default.
## But setting it to 2048 is a better default for busy servers.
##
server.max-fds = 2048

##
## listen-backlog is the size of the listen() backlog queue requested when
## the lighttpd server ask the kernel to listen() on the provided network
## address.  Clients attempting to connect() to the server enter the listen()
## backlog queue and wait for the lighttpd server to accept() the connection.
##
## The out-of-box default on many operating systems is 128 and is identified
## as SOMAXCONN.  This can be tuned on many operating systems.  (On Linux,
## cat /proc/sys/net/core/somaxconn)  Requesting a size larger than operating
## system limit will be silently reduced to the limit by the operating system.
##
## When there are too many connection attempts waiting for the server to
## accept() new connections, the listen backlog queue fills and the kernel
## rejects additional connection attempts.  This can be useful as an
## indication to an upstream load balancer that the server is busy, and
## possibly overloaded.  In that case, configure a smaller limit for
## server.listen-backlog.  On the other hand, configure a larger limit to be
## able to handle bursts of new connections, but only do so up to an amount
## that the server can keep up with responding in a reasonable amount of
## time.  Otherwise, clients may abandon the connection attempts and the
## server will waste resources servicing abandoned connections.
##
## It is best to leave this setting at its default unless you have modelled
## your traffic and tested that changing this benefits your traffic patterns.
##
## Default: 1024
##
#server.listen-backlog = 128

##
## Stat() call caching.
##
## lighttpd can utilize FAM/Gamin to cache stat call.
##
## possible values are:
## disable, simple or fam.
##
server.stat-cache-engine = "simple"

##
## Fine tuning for the request handling
##
## max-connections == max-fds/2 (maybe /3)
## means the other file handles are used for fastcgi/files
##
server.max-connections = 1024

##
## How many seconds to keep a keep-alive connection open,
## until we consider it idle. 
##
## Default: 5
##
#server.max-keep-alive-idle = 5

##
## How many keep-alive requests until closing the connection.
##
## Default: 16
##
#server.max-keep-alive-requests = 16

##
## Maximum size of a request in kilobytes.
## By default it is unlimited (0).
##
## Uploads to your server cant be larger than this value.
##
#server.max-request-size = 0

##
## Time to read from a socket before we consider it idle.
##
## Default: 60
##
#server.max-read-idle = 60

##
## Time to write to a socket before we consider it idle.
##
## Default: 360
##
#server.max-write-idle = 360

##
##  Traffic Shaping 
## -----------------
##
## see /usr/share/doc/lighttpd/traffic-shaping.txt
##
## Values are in kilobyte per second.
##
## Keep in mind that a limit below 32kB/s might actually limit the
## traffic to 32kB/s. This is caused by the size of the TCP send
## buffer. 
##
## per server:
##
#server.kbytes-per-second = 128

##
## per connection:
##
#connection.kbytes-per-second = 32

##
#######################################################################

#######################################################################
##
##  Filename/File handling
## ------------------------

##
## files to check for if .../ is requested
## index-file.names            = ( "index.php", "index.rb", "index.html",
##                                 "index.htm", "default.htm" )
##
index-file.names += (
  "index.xhtml", "index.html", "index.htm", "default.htm", "index.php"
)

##
## deny access the file-extensions
##
## ~    is for backupfiles from vi, emacs, joe, ...
## .inc is often used for code includes which should in general not be part
##      of the document-root
url.access-deny             = ( "~", ".inc" )

##
## disable range requests for pdf files
## workaround for a bug in the Acrobat Reader plugin.
## (ancient; should no longer be needed)
##
#$HTTP["url"] =~ "\.pdf$" {
#  server.range-requests = "disable"
#}

##
## url handling modules (rewrite, redirect)
##
#url.rewrite                = ( "^/$"             => "/server-status" )
#url.redirect               = ( "^/wishlist/(.+)" => "http://www.example.com/$1" )

##
## both rewrite/redirect support back reference to regex conditional using %n
##
#$HTTP["host"] =~ "^www\.(.*)" {
#  url.redirect            = ( "^/(.*)" => "http://%1/$1" )
#}

##
## which extensions should not be handle via static-file transfer
##
## .php, .pl, .fcgi are most often handled by mod_fastcgi or mod_cgi
##
static-file.exclude-extensions = ( ".php", ".pl", ".fcgi", ".scgi" )

##
## error-handler for all status 400-599
##
#server.error-handler       = "/error-handler.html"
#server.error-handler       = "/error-handler.php"

##
## error-handler for status 404
##
#server.error-handler-404   = "/error-handler.html"
#server.error-handler-404   = "/error-handler.php"

##
## Format: <errorfile-prefix><status-code>.html
## -> ..../status-404.html for 'File not found'
##
#server.errorfile-prefix    = "/srv/www/htdocs/errors/status-"

##
## mimetype mapping
##
include "conf.d/mime.conf"

##
## directory listing configuration
##
include "conf.d/dirlisting.conf"

##
## Should lighttpd follow symlinks?
## default: "enable"
#server.follow-symlink = "enable"

##
## force all filenames to be lowercase?
##
#server.force-lowercase-filenames = "disable"

##
## defaults to /var/tmp as we assume it is a local harddisk
## default: "/var/tmp"
#server.upload-dirs = ( "/app/private/lighttpd" )

##
#######################################################################


#######################################################################
##
##  SSL Support
## ------------- 
##
## To enable SSL for the whole server you have to provide a valid
## certificate and have to enable the SSL engine.::
##
##   ssl.engine = "enable"
##   ssl.pemfile = "/path/to/server.pem"
##
##   $SERVER["socket"] == "10.0.0.1:443" {
##     ssl.engine                  = "enable"
##     ssl.pemfile                 = "/etc/ssl/private/www.example.com.pem"
##
##     # Check your cipher list with: openssl ciphers -v '...'
##     # (use single quotes as your shell won't like ! in double quotes)
##     #ssl.cipher-list             = "HIGH"   # default
##
##     # (recommended to accept only TLSv1.2 and TLSv1.3)
##     #ssl.openssl.ssl-conf-cmd = ("MinProtocol" => "TLSv1.2")
##
##     server.name                 = "www.example.com"
##
##     server.document-root        = "/srv/www/vhosts/example.com/www/"
##   }
##

## If you have a .crt and a .key file, specify both ssl.pemfile and ssl.privkey,
## or cat them together into a single PEM file:
## $ cat /etc/ssl/private/lighttpd.key /etc/ssl/certs/lighttpd.crt \
##   > /etc/ssl/private/lighttpd.pem
##
#ssl.pemfile = "/etc/ssl/private/lighttpd.pem"
#
# or
#
#ssl.privkey = "/etc/ssl/private/privkey.pem"
#ssl.pemfile = "/etc/ssl/private/cert.pem"

##
## optionally pass the CA certificate here.
##
##
#ssl.ca-file = ""

##
## and the CRL revocation list here.
##
##
#ssl.ca-crl-file = ""

##
#######################################################################

#######################################################################
##
## custom includes like vhosts.
##
#include "conf.d/config.conf"
#include "/etc/lighttpd/vhosts.d/*.conf"
##
#######################################################################
~~~

## 认证
配置说明：
~~~
# 激活mod_auth模块
server.modules = (...,"mod_auth",...)
# 运行lighttpd服务的用户名和组，后面定义密码文件时会用到
server.username            = "lighttpd"
server.groupname           = "lighttpd" 
# 使用明文密码，这是最简单的方式，当然也有安全问题，见后面
auth.backend               = "plain"
# 定义用户名、密码存放的路径
auth.backend.plain.userfile = "/app/lighttpd/.lighttpd.user"
# 根据说明，下面的groupfile还未完全实现，不用设置咯
#auth.backend.plain.groupfile = "lighttpd.group"
# 定义要加密的路径
auth.require  = ( 
    "/server-status" =>
    (
        # 可以使用多种认证方式，这里以basic为例
        "method"  => "basic",
        # 访问时，对话框的提示信息
        "realm"   => "Server Status WebSite",
        # 允许访问的用户名，用“|”号分割多个用户
        "require" => "user=gd001|user=gd002"
    ) ,
    "/server-config" =>
    (
        "method"  => "basic",
        "realm"   => "Server Config WebSite",
        # valid-user用于表示所有合法的用户
        "require" => "valid-user"
    )
)
~~~
~~~
touch /app/lighttpd/.lighttpd.user
echo "gd001:123456" > /app/lighttpd/.lighttpd.user
~~~

### 1、auth.conf
~~~
#######################################################################
##
##  Authentication Module
## -----------------------
##
## See http://www.lighttpd.net/documentation/authentification.html
## for more info.
##

server.modules += ( "mod_auth" )

#server.modules += ( "mod_authn_file" )
#auth.backend                 = "htpasswd"
#auth.backend                   = "plain"
auth.backend                    = "htdigest"
#auth.backend.plain.userfile  = "/home/workspace/tmp/lighttpd/.lighttpd.plain"
auth.backend.htdigest.userfile  = "/app/uiapp/lighttpd/.lighttpd.user"

#server.modules += ( "mod_authn_ldap" )
#auth.backend               = "ldap"
#auth.backend.ldap.hostname = "localhost"
#auth.backend.ldap.base-dn  = "dc=my-domain,dc=com"
#auth.backend.ldap.filter   = "(uid=$)"

auth.require               = ( "/echo.fcgi" =>
                               (
                                 #"method"  => "basic",
                                 "method"  => "digest",
                                 "realm"   => "Server",
                                 "require" => "user=gd001|user=gd002",
                               ),
                             )
##
#######################################################################
~~~
### 2、生成

~~~sh
#auth.sh
printf "%s:%s:%s\n" "$1" "Server" "$(printf "%s" "$1:Server:$2" | md5sum | awk '{print $1}')" > .lighttpd.user
~~~
    ./auth.sh gd001 123456
    gd001:Server:1096ac864338a60557afb3c61190e51d


## 运行
~~~
/lighttpd/sbin/lighttpd -f /lighttpd/config/lighttpd.conf -m /lighttpd/lib
~~~
启动后可以看error.log日志:

    1970-01-01 00:05:09: (log.c.194) server started 
    1970-01-01 00:05:09: (mod_fastcgi.c.1363) --- fastcgi spawning local \n\tproc: /app/sd/cgi/echo.fcgi \n\tport: 0 \n\tsocket /app/sd/fastcgi.socket \n\tmax-procs: 1 
    1970-01-01 00:05:09: (mod_fastcgi.c.1387) --- fastcgi spawning \n\tport: 0 \n\tsocket /app/sd/fastcgi.socket \n\tcurrent: 0 / 1 
    1970-01-01 00:05:30: (mod_fastcgi.c.2879) got proc: pid: 1759 socket: unix:/app/sd/ fastcgi.socket-0 load: 1 
    1970-01-01 00:05:31: (mod_fastcgi.c.1494) released proc: pid: 1759 socket: unix:/app/sd/fastcgi.socket-0 load: 0 

## 访问
~~~
http://192.168.1.15
~~~

## 问题记录

### 1、404 错误
登陆时抓包返回404 file not found。

问题原因： 找不到fcgi程序。

<font color=red>在fastcgi.conf文件中定义了fcgi路径（"bin-path" => "/app/uiapp/lighttpd/echo.fcgi"）,运行服务器后也可以看到cgi程序正常运行。

    1677 root       0:00 ./lighttpd/sbin/lighttpd -f ./lighttpd/config/lighttpd.co
    1678 root       0:00 /app/uiapp/lighttpd/echo.fcgi

为什么会提示找不到文件呢，原来在lighttpd.conf中配置了

    #server_root=/app/uiapp/lighttpd
    server.document-root = server_root + "/www/"

测试发现，只需要在www目录下新建一个echo.fcgi同名的文件即可正常访问。
</font>

### 2、501 Internal error
访问cgi时报错。这种情况多半是cgi程序崩溃。可以单独运行cgi程序测试

    gdb ./echo.fcgi