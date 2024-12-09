# 0x1 frida 编译流程

> 建立一个项目目录并拉下frida源码，并进入项目目录

```shell
git clone --recurse-submodules -b 16.2.1 https://github.com/frida/frida
cd frida
```

此时执行ls ，看到的文件应当是如此

```log
(base) r@ubuntu20:~/Documents/FRIDA/frida$ ls
BSDmakefile      COPYING     frida-gum     frida.sln    Makefile.freebsd.mk  Makefile.toolchain.mk
build            frida-clr   frida-node    frida-swift  Makefile.linux.mk    README.md
config.mk        frida-core  frida-python  frida-tools  Makefile.macos.mk    releng
CONTRIBUTING.md  frida-go    frida-qml     Makefile     Makefile.sdk.mk
```

> 一键 安装nodejs22

```shell
# 构造下载 URL
NODE_TAR_URL="https://nodejs.org/dist/v22.12.0/node-v22.12.0-linux-x64.tar.xz"
wget $NODE_TAR_URL 
# 解压 Node.js 安装包到用户目录
tar -xf node-v22.12.0-linux-x64.tar.xz -C $HOME/bin
rm -r node-v22.12.0-linux-x64.tar.xz
# 设置 NODE_HOME 和 PATH
export NODE_HOME=$HOME/bin/node-v22.12.0-linux-x64
export PATH=${NODE_HOME}/bin:$PATH
# 打印 Node.js 版本以确认安装成功
node -v
```

```log
(base) r@ubuntu20:~/Documents/FRIDA/frida$ # 构造下载 URL
(base) r@ubuntu20:~/Documents/FRIDA/frida$ NODE_TAR_URL="https://nodejs.org/dist/v22.12.0/node-v22.12.0-linux-x64.tar.xz"
(base) r@ubuntu20:~/Documents/FRIDA/frida$ wget $NODE_TAR_URL 
--2024-12-09 23:31:18--  https://nodejs.org/dist/v22.12.0/node-v22.12.0-linux-x64.tar.xz
正在解析主机 nodejs.org (nodejs.org)... 198.18.1.205
正在连接 nodejs.org (nodejs.org)|198.18.1.205|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度： 29734248 (28M) [application/x-xz]
正在保存至: “node-v22.12.0-linux-x64.tar.xz”

node-v22.12.0-linux-x64.tar.xz          100%[============================================================================>]  28.36M  8.89MB/s    用时 3.2s  

2024-12-09 23:31:21 (8.89 MB/s) - 已保存 “node-v22.12.0-linux-x64.tar.xz” [29734248/29734248])

(base) r@ubuntu20:~/Documents/FRIDA/frida$ # 解压 Node.js 安装包到用户目录
(base) r@ubuntu20:~/Documents/FRIDA/frida$ tar -xf node-v22.12.0-linux-x64.tar.xz -C $HOME/bin
(base) r@ubuntu20:~/Documents/FRIDA/frida$ # 设置 NODE_HOME 和 PATH
(base) r@ubuntu20:~/Documents/FRIDA/frida$ export NODE_HOME=$HOME/bin/node-v22.12.0-linux-x64
(base) r@ubuntu20:~/Documents/FRIDA/frida$ export PATH=${NODE_HOME}/bin:$PATH
(base) r@ubuntu20:~/Documents/FRIDA/frida$ # 打印 Node.js 版本以确认安装成功
(base) r@ubuntu20:~/Documents/FRIDA/frida$ node -v
v22.12.0
(base) r@ubuntu20:~/Documents/FRIDA/frida$ ls
BSDmakefile      COPYING     frida-go    frida-python  frida-swift  Makefile.freebsd.mk  Makefile.sdk.mk                 README.md
config.mk        frida-clr   frida-gum   frida-qml     frida-tools  Makefile.linux.mk    Makefile.toolchain.mk           releng
CONTRIBUTING.md  frida-core  frida-node  frida.sln     Makefile     Makefile.macos.mk    node-v22.12.0-linux-x64.tar.xz
(base) r@ubuntu20:~/Documents/FRIDA/frida$ rm -r node-v22.12.0-linux-x64.tar.xz
(base) r@ubuntu20:~/Documents/FRIDA/frida$ 
```

> 再make一下

```shell
make
```

> 安装ndk

执行以下命令，查看所需要的ndk版本，得到以下输出

```shell
cat releng/setup-env.sh |grep "ndk_required="
```

```log
(base) r@ubuntu20:~/Documents/FRIDA/frida$ cat releng/setup-env.sh |grep "ndk_required="
  ndk_required=25
```

一键安装ndk25

```shell
wget https://dl.google.com/android/repository/android-ndk-r25c-linux.zip
unzip android-ndk-r25c-linux.zip $HOME/bin/
rm -r android-ndk-r25c-linux.zip
export ANDROID_NDK_ROOT=$HOME/bin/android-ndk-r25c
export PATH=$ANDROID_NDK_ROOT:$PATH
ndk-build -v
```

> 安装依赖

```shell
sudo apt update
sudo apt-get install build-essential git lib32stdc++-9-dev libc6-dev-i386
```

```shell
pip3 install lief
```

> 编译

查看编译选项

```shell
(frida-compile) r@ubuntu20:~/Documents/FRIDA/frida$ make
make[1]: 进入目录“/home/r/Documents/FRIDA/frida”

Usage: make TARGET [VARIABLE=value]

Where TARGET specifies one or more of:

  /* gum */
  gum-linux-x86                     Build for Linux/x86
  gum-linux-x86_64                  Build for Linux/x86-64
  gum-linux-x86-thin                Build for Linux/x86 without cross-arch support
  gum-linux-x86_64-thin             Build for Linux/x86-64 without cross-arch support
  gum-linux-x86_64-gir              Build for Linux/x86-64 with shared GLib and GIR
  gum-linux-arm                     Build for Linux/arm
  gum-linux-armbe8                  Build for Linux/armbe8
  gum-linux-armhf                   Build for Linux/armhf
......等等
```

编译安卓arm64的frida

```shell
make core-android-arm64
```

编译完成

```log
Installing lib/base/libfrida-base-1.0.a to /home/r/Documents/FRIDA/frida/build/frida-android-arm64/lib
Installing lib/base/frida-base.h to /home/r/Documents/FRIDA/frida/build/frida-android-arm64/include/frida-1.0
Installing lib/base/frida-base-1.0.vapi to /home/r/Documents/FRIDA/frida/build/frida-android-arm64/share/vala/vapi
Installing lib/payload/libfrida-payload-1.0.a to /home/r/Documents/FRIDA/frida/build/frida-android-arm64/lib
Installing lib/payload/frida-payload.h to /home/r/Documents/FRIDA/frida/build/frida-android-arm64/include/frida-1.0
Installing lib/payload/frida-payload-1.0.vapi to /home/r/Documents/FRIDA/frida/build/frida-android-arm64/share/vala/vapi
Installing lib/gadget/frida-gadget.so to /home/r/Documents/FRIDA/frida/build/frida-android-arm64/lib/frida/64
Installing src/api/frida-core.h to /home/r/Documents/FRIDA/frida/build/frida-android-arm64/include/frida-1.0
Installing src/api/frida-core-1.0.vapi to /home/r/Documents/FRIDA/frida/build/frida-android-arm64/share/vala/vapi
Installing src/api/frida-core-1.0.deps to /home/r/Documents/FRIDA/frida/build/frida-android-arm64/share/vala/vapi
Installing src/api/libfrida-core-1.0.a to /home/r/Documents/FRIDA/frida/build/frida-android-arm64/lib
Installing server/frida-server to /home/r/Documents/FRIDA/frida/build/frida-android-arm64/bin
Installing portal/frida-portal to /home/r/Documents/FRIDA/frida/build/frida-android-arm64/bin
Installing inject/frida-inject to /home/r/Documents/FRIDA/frida/build/frida-android-arm64/bin
Installing /home/r/Documents/FRIDA/frida/frida-core/lib/selinux/frida-selinux.h to /home/r/Documents/FRIDA/frida/build/frida-android-arm64/include/frida-1.0
Installing /home/r/Documents/FRIDA/frida/build/tmp-android-arm64/frida-core/meson-private/frida-base-1.0.pc to /home/r/Documents/FRIDA/frida/build/frida-android-arm64/lib/pkgconfig
Installing /home/r/Documents/FRIDA/frida/build/tmp-android-arm64/frida-core/meson-private/frida-payload-1.0.pc to /home/r/Documents/FRIDA/frida/build/frida-android-arm64/lib/pkgconfig
Installing /home/r/Documents/FRIDA/frida/build/tmp-android-arm64/frida-core/meson-private/frida-core-1.0.pc to /home/r/Documents/FRIDA/frida/build/frida-android-arm64/lib/pkgconfig
make[1]: 离开目录“/home/r/Documents/FRIDA/frida”
```

查看编译后的文件

```shell
cd build/frida-android-arm64/bin && ls
```

```log
(frida-compile) r@ubuntu20:~/Documents/FRIDA/frida$ cd build/frida-android-arm64/bin && ls
frida-inject  frida-portal  frida-server  gum-graft
(frida-compile) r@ubuntu20:~/Documents/FRIDA/frida/build/frida-android-arm64/bin$ 
```

# 0x2 修改frida

> 先把所有的 "frida_agent_main" 换成"main"
> 
> ![](/home/r/.config/marktext/images/2024-12-09-23-42-57-image.png)

> 然后打上patch, 如果不会打的话其实一个个改也不费事，字符串的话可以直接替换，我这里把frida改成了rusda，你也可以改成其他的
> 
> ![](/home/r/.config/marktext/images/2024-12-10-00-56-07-image.png)

github: [GitHub - taisuii/rusda: 对frida 16.2.1的patch](https://github.com/taisuii/rusda)

> python脚本新建在frida-core/src目录下

如果你提示No module named 'lief' 说明Python模块没有装好 pip3 install lief

> 然后编译，这里可以过滤日志编译，如果编译成功还是有很多特征大部分原因是python脚本没有打上patch

```shell
make core-android-arm64 | grep Patch
```

```log
(base) r@ubuntu20:~/Documents/FRIDA/frida$ make core-android-arm64 | grep Patch
[*] Patch frida-agent: /home/r/Documents/FRIDA/frida/build/tmp-android-arm64/frida-core/src/frida-agent@emb/frida-agent-64.so
[*] Patch `frida` to `rusda`
[*] Patching section name=.rodata offset=0x1c4a26 orig:FridaScriptEngine new:enignEtpircSadirF
[*] Patching section name=.rodata offset=0x1d24db orig:FridaScriptEngine new:enignEtpircSadirF
[*] Patching section name=.rodata offset=0x1d9472 orig:GLib-GIO new:OIG-biLG
[*] Patching section name=.rodata offset=0x1959df orig:GDBusProxy new:yxorPsuBDG
[*] Patching section name=.rodata offset=0x1c4b31 orig:GDBusProxy new:yxorPsuBDG
[*] Patching section name=.rodata offset=0x1b1746 orig:GumScript new:tpircSmuG
[*] Patching section name=.rodata offset=0x210bed orig:GumScript new:tpircSmuG
[*] Patching section name=.rodata offset=0x238393 orig:GumScript new:tpircSmuG
[*] Patching section name=.rodata offset=0x246184 orig:GumScript new:tpircSmuG
[*] Patch `gum-js-loop` to `russellloop`
[*] Patch `gmain` to `rmain`
[*] Patch `gdbus` to `rubus`
[*] Patch Finish
[*] Patch frida-agent: /home/r/Documents/FRIDA/frida/build/tmp-android-arm64/frida-core/src/frida-agent@emb/frida-agent-32.so
[*] Patch `frida` to `rusda`
[*] Patching section name=.rodata offset=0xcc3a3 orig:FridaScriptEngine new:enignEtpircSadirF
[*] Patching section name=.rodata offset=0xd984c orig:FridaScriptEngine new:enignEtpircSadirF
[*] Patching section name=.rodata offset=0xe066f orig:GLib-GIO new:OIG-biLG
[*] Patching section name=.rodata offset=0x9e15e orig:GDBusProxy new:yxorPsuBDG
[*] Patching section name=.rodata offset=0xcc4ae orig:GDBusProxy new:yxorPsuBDG
[*] Patching section name=.rodata offset=0xb96c5 orig:GumScript new:tpircSmuG
[*] Patching section name=.rodata offset=0x115e26 orig:GumScript new:tpircSmuG
[*] Patching section name=.rodata offset=0x13d0a3 orig:GumScript new:tpircSmuG
[*] Patching section name=.rodata offset=0x14aa2d orig:GumScript new:tpircSmuG
[*] Patch `gum-js-loop` to `russellloop`
[*] Patch `gmain` to `rmain`
[*] Patch `gdbus` to `rubus`
[*] Patch Finish
[*] Patch frida-agent: /home/r/Documents/FRIDA/frida/build/tmp-android-arm64/frida-core/src/frida-agent@emb/frida-agent-arm64.so
[*] Patch `frida` to `rusda`
[*] Patch frida-agent: /home/r/Documents/FRIDA/frida/build/tmp-android-arm64/frida-core/src/frida-agent@emb/frida-agent-arm.so
[*] Patch `frida` to `rusda`
(base) r@ubuntu20:~/Documents/FRIDA/frida$ ls 
```

# 0x3 运行测试

> 这里换个端口，就是全绿

```shell
cd build/frida-android-arm64/bin
adb push frida-server /data/local/tmp
adb shell
chmod +x frida-server
./frida-server -l 127.0.0.1:12345
```

```shell
frida -H 127.0.0.1:12345 -f com.yimian.envcheck
```

![](/home/r/.config/marktext/images/2024-12-10-01-06-20-image.png)

# 0x4 参考

> [GitHub - Ylarod/Florida: 基础反检测 frida-server / Basic anti-detection frida-server](https://github.com/Ylarod/Florida/tree/main)
> 
> [GitHub - hluwa/Patchs: strongR-frida](https://github.com/hluwa/Patchs)
> 
> [[原创]FRIDA 最新版编译 | 16.0.9-Android安全-看雪-安全社区|安全招聘|kanxue.com](https://bbs.kanxue.com/thread-276076.htm)
