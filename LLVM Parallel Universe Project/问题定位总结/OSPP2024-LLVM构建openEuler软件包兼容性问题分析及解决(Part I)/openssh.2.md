# openssh.2 #

## 1、问题现象 ##
出自：https://gitee.com/src-openeuler/openssh(openEuler-24.03-LTS)

在修复了openssh.1的问题之后，又出现了第二个问题：
```
[234s] + setenforce 0
[234s] setenforce:security setenforce() failed:Permission denied
```
这意味着在尝试将 SELinux 的强制模式设置为宽容模式时，由于没有足够的权限，操作未能成功

## 2、问题定位及根因分析 ##

源代码中没有出现问题

之所以gcc中没有出现这个问题可能的原因：

1.编译器优化:GCC 和 Clang 在处理代码优化时有不同的策略。GCC 在严格模式下可能会遇到一些优化问题，例如严格别名(StrictAliasing)和整数环绕(Integer Wrap-around)1。这些问题可能会导致编译失败，而 Clang 可能不会受到这些问题的影响。

2.安全策略:SELinux 的严格模式会限制某些系统调用和文件访问权限。GCC 可能需要访问某些文件或执行某些系统调用，而这些操作在严格模式下可能会被阻止。Clang可能使用不同的系统调用或文件访问方式，因此不会受到影响。

3.编译器实现:GCC 和 Clang 的实现方式不同，可能会导致在严格模式下的行为差异。例如，GCC 在某些情况下可能会尝试访问被 SELinux 限制的资源，而Clang 可能不会

spec文件中：
```
%check
if [ -e /sys/fs/selinux/enforce ]; then 
    # Store the SElinux state
    cat /sys/fs/selinux/enforce > selinux.tmp
    setenforce 0
fi
make tests
if [ -e /sys/fs/selinux/enforce ]; then
    # Restore the SElinux state
    cat selinux.tmp > /sys/fs/selinux/enforce
    rm -rf selinux.tmp
fi
```
## 3、修改建议 ##
将%check处改为：
```
%check
if [ -e /sys/fs/selinux/enforce ]; then 
    # Store the SElinux state only if the file exists
    if [ -w /sys/fs/selinux/enforce ] && [ -w. ]; then
        cat /sys/fs/selinux/enforce > selinux.tmp
        setenforce 0
    else
        echo "Insufficient permissions to handle SELinux state. Skipping modification."
    fi
else
    echo "SELinux is not enabled or enforce file not found. Skipping modification."
fi
make tests
if [ -e /sys/fs/selinux/enforce ]; then
    # Restore the SElinux state only if the file exists
    if [ -w /sys/fs/selinux/enforce ] && [ -f selinux.tmp ]; then
        cat selinux.tmp > /sys/fs/selinux/enforce
        rm -rf selinux.tmp
    else
        echo "Insufficient permissions or temp file not found. Skipping restoration of SELinux state."
    fi
fi
```
目的是在尝试修改 SELinux 强制文件之前检查其是否可写。这可以防止由于权限不足而导致脚本失败
## 4、相关PR ##
openEuler：https://gitee.com/src-openeuler/openssh/pulls/307
上游社区：不涉及