# mac
- 下载openjdk源码
- 安装软件

        // 提高编译速度、提供字体、链接X11(需要安装Quartz)
        brew install ccache
        brew install freetype
        sudo ln -s /usr/X11/include/X11 /usr/include/X11
        sudo ln -s /usr/local/X11/include/X11 /usr/include/X11

- 补丁
    - 补丁1

            openjdk/common/autoconf/generated-configure.sh
            注释所有 GCC compiler is required
    - 补丁2

            openjdk/hotspot/src/share/vm/code/relocInfo.hpp
            367
            ---inline friend relocInfo prefix_relocInfo(int datalen = 0)
            +++inline friend relocInfo prefix_relocInfo(int datalen)
            462
            ---inline relocInfo prefix_relocInfo(int datalen) {
            +++inline relocInfo prefix_relocInfo(int datalen = 0) {
    - 补丁3

            openjdk/hotspot/src/share/vm/runtime/virtualspace.cpp
            331
            ---if (base() > 0) {
            +++if (base() != NULL) {
    - 补丁4

            openjdk/hotspot/src/share/vm/opto/loopPredicate.cpp
            775
            ---assert(rng->Opcode() == Op_LoadRange || _igvn.type(rng)->is_int() >= 0, "must be");
            +++assert(rng->Opcode() == Op_LoadRange || _igvn.type(rng)->is_int()->_lo >= 0, "must be");
- 4 编译

        chmod a+x configure
        sudo ./configure
            --with-target-bits=64
            --with-debug-level=fastdebug
            --enable-ccache --with-boot-jdk-jvmargs="-Xlint:deprecation -Xlint:unchecked"
            --with-freetype-include=/usr/local/Cellar/freetype/2.9/include/freetype2
            --with-freetype-lib=/usr/local/Cellar/freetype/2.9/lib
        sudo make all
            LANG=C
            CC=clang
            USE_CLANG=true
            COMPILER_WARNINGS_FATAL=false
            LFLAGS='-Xlinker -lstdc++'
            LOG=debug

# 参考引用
0. [在 macOS 上编译 OpenJDK 8 | 身无所拘，心无疆](https://imkiva.com/2018/02/24/building-openjdk8-on-macos/)
0. [JAVA虚拟机学习笔记（一）Windows10下编译OpenJDK8 - dark_saber - 博客园](https://www.cnblogs.com/lighten/p/5906359.html)
0. [WINDOWS7平台下编译OpenJDK8和Visual Studio 单步调试HotSpot... - 简书](https://www.jianshu.com/p/e85f93cc74cb)
0. [build fail with Cygwin](https://jdk8-dev.openjdk.java.narkive.com/1xjp97JL/build-fail-with-cygwin)