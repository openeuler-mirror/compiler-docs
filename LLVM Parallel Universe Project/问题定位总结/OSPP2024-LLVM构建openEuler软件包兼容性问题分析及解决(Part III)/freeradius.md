## freeradius

### 问题描述

#### 报错信息

修改前clang构建失败，出现.d和.lo文件不生成的问题。出现类似：

```
[  726s] mkdir -p build/objs/src/lib/
[  726s] echo CC src/lib/cbuff.c
[  726s] CC src/lib/cbuff.c
[  726s] build/make/jlibtool --silent --mode=compile clang -o build/objs/src/lib/cbuff.lo -c -MD -I. -Isrc -include src/freeradius-devel/autoconf.h -include src/freeradius-devel/build.h -include src/freeradius-devel/features.h -include src/freeradius-devel/radpaths.h -fno-strict-aliasing -Wno-date-time -O2 -g -grecord-gcc-switches -pipe -fstack-protector-strong -Wall -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -Wp,-D_GLIBCXX_ASSERTIONS --config /usr/lib/rpm/generic-hardened-clang.cfg -m64 -mtune=generic -fasynchronous-unwind-tables -fstack-clash-protection -fsigned-char -Wall -std=c99 -D_GNU_SOURCE -Wno-unknown-warning-option -D_REENTRANT -D_POSIX_PTHREAD_SEMANTICS -DOPENSSL_NO_KRB5 -DNDEBUG -D_LIBRADIUS -I/home/abuild/rpmbuild/BUILD/freeradius-server-3.2.3/src src/lib/cbuff.c
[  726s] mkdir -p build/make/src/src/lib/
[  726s] mkdir -p build/objs/src/lib/
[  726s] sed  -e 's/#.*//' -e 's,^/home/abuild/rpmbuild/BUILD/freeradius-server-3.2.3,${top_srcdir},' -e 's, /home/abuild/rpmbuild/BUILD/freeradius-server-3.2.3, ${top_srcdir},' -e 's,^build,${BUILD_DIR},' -e 's, build/make/include/[^ :]*,,' -e 's, build, ${BUILD_DIR},' -e 's, /[^: ]*,,g' -e 's,^ *[^:]* *: *$,,' -e '/: </ d' -e 's/\.o: /.${OBJ_EXT}: /' -e '/^ *\\$/ d' < build/objs/src/lib/cbuff.d | sed -e '$!N; /^\(.*\)\n\1$/!P; D' >  build/make/src/src/lib/cbuff.mk
[  726s] sed -e 's/#.*//' -e 's, build/make/include/[^ :]*,,' -e 's, /[^: ]*,,g' -e 's,^ *[^:]* *: *$,,' -e '/: </ d' -e 's/^[^:]*: *//' -e 's/ *\\$//' -e 's/$/ :/' < build/objs/src/lib/cbuff.d | sed -e '$!N; /^\(.*\)\n\1$/!P; D' >> build/make/src/src/lib/cbuff.mk
[  726s] /bin/sh: line 1: build/objs/src/lib/cbuff.d: No such file or directory
[  726s] /bin/sh: line 1: build/objs/src/lib/cbuff.d: No such file or directory
```

等的很多.d文件`No such file or directory`的报错，最终在链接阶段由于.lo文件缺失导致构建失败：

```
[  198s] build/make/jlibtool --silent --mode=link clang -o build/lib/libfreeradius-radius.la -rpath /usr/lib64/freeradius   -Wl,-z,relro   -Wl,-z,now     -Wl,-z,relro   -Wl,-z,now    build/objs/src/lib/cbuff.lo build/objs/src/lib/cursor.lo build/objs/src/lib/debug.lo build/objs/src/lib/dict.lo build/objs/src/lib/filters.lo build/objs/src/lib/hash.lo build/objs/src/lib/hmacmd5.lo build/objs/src/lib/hmacsha1.lo build/objs/src/lib/isaac.lo build/objs/src/lib/log.lo build/objs/src/lib/misc.lo build/objs/src/lib/missing.lo build/objs/src/lib/md4.lo build/objs/src/lib/md5.lo build/objs/src/lib/net.lo build/objs/src/lib/pair.lo build/objs/src/lib/pcap.lo build/objs/src/lib/print.lo build/objs/src/lib/radius.lo build/objs/src/lib/rbtree.lo build/objs/src/lib/regex.lo build/objs/src/lib/sha1.lo build/objs/src/lib/snprintf.lo build/objs/src/lib/strlcat.lo build/objs/src/lib/strlcpy.lo build/objs/src/lib/socket.lo build/objs/src/lib/token.lo build/objs/src/lib/udpfromto.lo build/objs/src/lib/value.lo build/objs/src/lib/fifo.lo build/objs/src/lib/packet.lo build/objs/src/lib/event.lo build/objs/src/lib/getaddrinfo.lo build/objs/src/lib/heap.lo build/objs/src/lib/tcp.lo build/objs/src/lib/base64.lo build/objs/src/lib/version.lo build/objs/src/lib/atomic_queue.lo build/objs/src/lib/talloc.lo  -lcrypto -lssl -ltalloc -latomic  -lpcre -lresolv -ldl -lpthread  -lreadline -lpcap 
[  198s] Can not find suitable object file for build/objs/src/lib/cbuff.lo
[  198s] make: *** [scripts/boiler.mk:638: build/lib/libfreeradius-radius.la] Error 1
```

#### 相关PR

- 上游社区pr：[jlibtool: avoid conflict with Clang's --config option by yanyir · Pull Request #5443 · FreeRADIUS/freeradius-server (github.com)](https://github.com/FreeRADIUS/freeradius-server/pull/5443)
- openEuler仓库pr：[Fix clang build: avoid conflict with clang's "--config" option in jlibtool · Pull Request !92 · src-openEuler/freeradius - Gitee.com](https://gitee.com/src-openeuler/freeradius/pulls/92)

### 修复过程

- 在构建目录下命令行单独执行

  ```
  build/make/jlibtool --silent --mode=compile clang -o build/objs/src/lib/cbuff.lo -c -MD -I. -Isrc -include src/freeradius-devel/autoconf.h -include src/freeradius-devel/build.h -include src/freeradius-devel/features.h -include src/freeradius-devel/radpaths.h -fno-strict-aliasing -Wno-date-time -O2 -g -grecord-gcc-switches -pipe -fstack-protector-strong -Wall -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -Wp,-D_GLIBCXX_ASSERTIONS --config /usr/lib/rpm/generic-hardened-clang.cfg -m64 -mtune=generic -fasynchronous-unwind-tables -fstack-clash-protection -fsigned-char -Wall -std=c99 -D_GNU_SOURCE -Wno-unknown-warning-option -D_REENTRANT -D_POSIX_PTHREAD_SEMANTICS -DOPENSSL_NO_KRB5 -DNDEBUG -D_LIBRADIUS -I/home/abuild/rpmbuild/BUILD/freeradius-server-3.2.3/src src/lib/cbuff.c
  ```

  发现没有产生新的.d和.lo文件，但执行其中的clang -o ...可以正常执行并产生对应的.d和.lo文件，判断问题出现在jlibtool上。

  而使用gcc构建的工程可以成功产生.d和.lo文件，对应的命令为：

  ```
  build/make/jlibtool --silent --mode=compile gcc -o build/objs/src/lib/cbuff.lo -c -MD -I. -Isrc -include src/freeradius-devel/autoconf.h -include src/freeradius-devel/build.h -include src/freeradius-devel/features.h -include src/freeradius-devel/radpaths.h -fno-strict-aliasing -Wno-date-time -O2 -g -grecord-gcc-switches -pipe -fstack-protector-strong -Wall -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -Wp,-D_GLIBCXX_ASSERTIONS -specs=/usr/lib/rpm/generic-hardened-cc1 -fasynchronous-unwind-tables -fstack-clash-protection -Wall -std=c99 -D_GNU_SOURCE -D_REENTRANT -D_POSIX_PTHREAD_SEMANTICS -DOPENSSL_NO_KRB5 -DNDEBUG -D_LIBRADIUS -I/home/abuild/rpmbuild/BUILD/freeradius-server-3.2.3/src src/lib/cbuff.c
  ```

  对比clang和gcc使用的命令发现问题可能出现在gcc构建时配置中是以`-specs=/usr/lib/rpm/generic-hardened-cc1`而非clang时的`--config /usr/lib/rpm/generic-hardened-clang.cfg`来指定的配置文件。

- 尝试在上述`build/make/jlibtool --silent --mode=compile clang -o...`中去掉`--config /usr/lib/rpm/generic-hardened-clang.cfg`，发现可以正常产生.d和.lo文件的。

- 查看jlibtool的代码，发现问题出现在jlibtool有一个`--config`参数，用于展示当前的配置项内容。然而clang中的`--config`参数是用于指定使用的配置文件，见：
  [Clang command line argument reference — Clang 17.0.1 documentation (llvm.org)](https://gitee.com/link?target=https%3A%2F%2Freleases.llvm.org%2F17.0.1%2Ftools%2Fclang%2Fdocs%2FClangCommandLineReference.html%23cmdoption-clang-config)
  如果放任冲突存在，会导致原本CFLAGS中的 `--config <cfg-file>`参数错误地被jlibtool接收，然后误认为只是要打印配置项，而非执行期望中传递给clang编译器进行编译的行为，从而不产生构建时需要的各种文件。

- **修改建议**：将`script/jlibtool.c`中的`--config`参数改为不与clang冲突的其他名称，如`--print-config`
