### Mac编译libevent库（iOS & MacOS）【高性能并发socket】

* 编译环境

```
Mac系统 High Sierra 版本10.13.6
Xcode Version 10.1
```


* 把https://github.com/OnionBrowser/OnionBrowser工程clone到本地

```
git clone https://github.com/OnionBrowser/OnionBrowser
```


* 进入OnionBrowser目录，并切换到ipv6test分支

```
cd OnionBrowser
git checkout ipv6test
```


* 复制我们需要的两个脚本文件到自己的目录中

```
cp build-libevent.sh ~/Desktop/libevent-build
cp build-libssl.sh ~/Desktop/libevent-build
```

* 终端下执行脚本./build-libevent.sh，可以看到下面的结果

```
bogon:libevent-build zhuruhong$ ./build-libevent.sh 
Downloading libevent-2.0.22-stable.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   618    0   618    0     0    400      0 --:--:--  0:00:01 --:--:--   400
100  834k  100  834k    0     0   161k      0  0:00:05  0:00:05 --:--:--  247k
Using libevent-2.0.22-stable.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   622    0   622    0     0    415      0 --:--:--  0:00:01 --:--:--   415
100   636  100   636    0     0    232      0  0:00:02  0:00:02 --:--:--  621k
Using libevent-2.0.22-stable.tar.gz.asc

COULD NOT VERIFY PACKAGE SIGNATURE...
bogon:libevent-build zhuruhong$ 
```

* 查看脚本，可以看到校验失败，我们增加命令参数执行./build-libevent.sh --noverify

```
bogon:libevent-build zhuruhong$ ./build-libevent.sh --noverify
Using libevent-2.0.22-stable.tar.gz
Building without ccache
checking for a BSD-compatible install... /usr/local/bin/ginstall -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... /usr/local/bin/gmkdir -p
checking for gawk... no
checking for mawk... no
checking for nawk... no
checking for awk... awk
checking whether make sets $(MAKE)... yes
checking build system type... i386-apple-darwin17.7.0
checking host system type... i386-apple-darwin17.7.0
checking for gcc... /Applications/Xcode.app/Contents/Developer/usr/bin/gcc -arch i386 -miphoneos-version-min=8.2
checking whether the C compiler works... no
configure: error: in `/Users/zhuruhong/Desktop/libevent-build/build/src/libevent-2.0.22-stable':
configure: error: C compiler cannot create executables
See `config.log' for more details
bogon:libevent-build zhuruhong$
```


* 根据提示，查看config.log中的错误信息，针对我的编译环境，修改脚本内容

```
USERSDKVERSION="12.1"
```

* 修改后，再次执行./build-libevent.sh --noverify，看到如下错误：

```
bogon:libevent-build zhuruhong$ ./build-libevent.sh --noverify
Using libevent-2.0.22-stable.tar.gz
Building without ccache
checking for a BSD-compatible install... /usr/local/bin/ginstall -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... /usr/local/bin/gmkdir -p
checking for gawk... no
checking for mawk... no
checking for nawk... no
checking for awk... awk
checking whether make sets $(MAKE)... yes
checking build system type... i386-apple-darwin17.7.0
checking host system type... i386-apple-darwin17.7.0
checking for gcc... /Applications/Xcode.app/Contents/Developer/usr/bin/gcc -arch i386 -miphoneos-version-min=8.2
checking whether the C compiler works... yes
checking for C compiler default output file name... a.out
checking for suffix of executables... 
checking whether we are cross compiling... configure: error: in `/Users/zhuruhong/Desktop/libevent-build/build/src/libevent-2.0.22-stable':
configure: error: cannot run C compiled programs.
If you meant to cross compile, use `--host'.
See `config.log' for more details
bogon:libevent-build zhuruhong$ 
```


* 上面的错误是交叉编译的问题，configure中缺少--host配置，我们再次修改脚本中的配置。找到EXTRA_CONFIG的位置，增加一个--host配置项，完成后如下：

```
上面省略
        if [ "${ARCH}" == "i386" ] || [ "${ARCH}" == "x86_64" ];
        then
                PLATFORM="iPhoneSimulator"
                EXTRA_CONFIG="--host=${ARCH}-apple-darwin"
        else
                PLATFORM="iPhoneOS"
                EXTRA_CONFIG="--host=arm-apple-darwin14"
        fi
下面省略
```

* 执行./build-libevent.sh --noverify，编译成功。在dependencies中保留了编译结果。可以用lipo命令查看静态库的信息

```
上面省略
Build library...
libevent_openssl.a does not exist, skipping (are the dependencies installed?)
Building done.
Cleaning up...
Done.
bogon:libevent-build zhuruhong$
```

```
bogon:lib zhuruhong$ lipo -info libevent.a 
Architectures in the fat file: libevent.a are: i386 armv7 x86_64 arm64 
bogon:lib zhuruhong$
```

* 从上面的编译结果中，我们可以看到没有生成libevent_openssl.a库，我们接下来继续处理。执行./build-libssl.sh --noverify命令

```
//同样的把脚本中的Xcode环境，配置到本机安装配置
USERSDKVERSION="12.1"
```


### Mac编译libevent库（MacOS平台）

* 进入build中的src目录，解压libevent-2.0.22-stable.tar.gz

```
tar zxf libevent-2.0.22-stable.tar.gz
```

* 进入libevent-2.0.22-stable目录，执行./configure && make命令

```
./configure && make
```

```
上面省略
In file included from cryptlib.c:117:
./cryptlib.h:62:11: fatal error: 'stdlib.h' file not found
# include <stdlib.h>
          ^~~~~~~~~~
1 error generated.
make[1]: *** [cryptlib.o] Error 1
make: *** [build_crypto] Error 1
bogon:libevent-build zhuruhong$ 
```

* 不开心，'stdlib.h' file not found，又出错了。这里是因为找不到openssl中的编译头文件，我们看下电脑上的头文件信息。使用命令<mark>brew info openssl</mark>

```
//这里我本地已经安装过openssl了，所以只需要查看编译头文件位置即可。
bogon:libevent-build zhuruhong$ brew info openssl
openssl: stable 1.0.2p (bottled) [keg-only]
SSL/TLS cryptography library
https://openssl.org/
/usr/local/Cellar/openssl/1.0.2p (1,793 files, 12.3MB)
  Poured from bottle on 2018-10-19 at 12:36:32
From: https://github.com/Homebrew/homebrew-core/blob/master/Formula/openssl.rb
==> Dependencies
Build: makedepend ✘
==> Options
--without-test
	Skip build-time tests (not recommended)
==> Caveats
A CA file has been bootstrapped using certificates from the SystemRoots
keychain. To add additional certificates (e.g. the certificates added in
the System keychain), place .pem files in
  /usr/local/etc/openssl/certs

and run
  /usr/local/opt/openssl/bin/c_rehash

openssl is keg-only, which means it was not symlinked into /usr/local,
because Apple has deprecated use of OpenSSL in favor of its own TLS and crypto libraries.

If you need to have openssl first in your PATH run:
  echo 'export PATH="/usr/local/opt/openssl/bin:$PATH"' >> ~/.bash_profile

For compilers to find openssl you may need to set:
  export LDFLAGS="-L/usr/local/opt/openssl/lib"
  export CPPFLAGS="-I/usr/local/opt/openssl/include"

For pkg-config to find openssl you may need to set:
  export PKG_CONFIG_PATH="/usr/local/opt/openssl/lib/pkgconfig"

==> Analytics
install: 421,421 (30 days), 1,491,514 (90 days), 5,833,616 (365 days)
install_on_request: 53,051 (30 days), 199,643 (90 days), 765,069 (365 days)
build_error: 0 (30 days)
bogon:libevent-build zhuruhong$ 
```
注意这一节的信息，在编译libevent前，我们设置一下这个CPPFLAGS信息
```
For compilers to find openssl you may need to set:
  export LDFLAGS="-L/usr/local/opt/openssl/lib"
  export CPPFLAGS="-I/usr/local/opt/openssl/include"
```

* 后面继续编译指令，如下：

```
export CPPFLAGS="-I/usr/local/opt/openssl/include"
./configure && make
```

### 参考资料 ios
[https://github.com/libevent/libevent/issues/360#issuecomment-225661036](https://github.com/libevent/libevent/issues/360#issuecomment-225661036)

[https://stackoverflow.com/questions/15666547/how-to-build-a-cross-compiler-for-i386-apple-darwin-target-from-x86-64-apple-dar](https://stackoverflow.com/questions/15666547/how-to-build-a-cross-compiler-for-i386-apple-darwin-target-from-x86-64-apple-dar)

### MacOS参考资料
[http://ju.outofmemory.cn/entry/230606](http://ju.outofmemory.cn/entry/230606)


