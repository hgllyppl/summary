# 本篇说明
本篇为jvm故障诊断总结。<br>
<span style="color:red">红字</span>为重要说明，<span style="color:green">绿字</span>为高亮提示，<span style="color:orange">橙字</span>为不确定说明。

# 诊断手段
## 查看资源占用
- CPU
    - top -H -p pid，查看进程内线程对CPU占用率
- 内存
- 磁盘
- 网络
    - tcpdump -i any -nn -S -vvv port 8087

## 查看堆栈
- 栈
    - jstack -l pid，打印栈信息
        - l，打印线程持有的锁信息
        - m，打印本地栈
        - F，强制打印栈，此操作会导致<span style="color:red">STW</span>
- 堆
    - jstat -gcutil pid intervalTime，已固定时间间隔打印堆区占用率
    - jmap -dump:live, format=b, file=pid.bin pid，导出堆，此操作会导致<span style="color:red">STW</span>
- FAQ
    0. jstack、jstat等命令出现pid not found<br>
        查看java.io.tmpdir下有没有进程文件，linux下默认为/tmp/hsperfdata_user<br>
            1）如果有文件<br>
                <span style="margin-left: 3%">a）切换到<span style="color:green">启动jvm的用户</span>下执行命令</span><br>
            2）如果没有文件<br>
                <span style="margin-left: 3%">a）启动jvm的用户没有权限写java.io.tmpdir</span><br>
                <span style="margin-left: 3%">b）-XX:+PerfDisableSharedMem 此参数禁止写入perfdata</span><br>

## 查看本地栈
- pstack pid，查看本地调用
- strace -fcp pid，统计本地调用频率

# 问题场景
## 高负载|响应慢 场景
- 原因
    - 计算任务
    - 死循环
    - 热锁(大量线程导致频繁的上下文切换)
- 排查方法
    - 记录高负载线程
    - 打印栈信息
    - 统计本地调用频率(如果需要)
- 案例
    0. zbj-ub项目导致的CPU负载飙高
        - 现场
            - 高负载线程[30727]
            - 栈文件[./jvm/zbj-ub.thread.dump]
        - 条件，mysql-5.1.31、linux-kernel-2.6，网络闪断?
        - 分析，linux内核bug和mysql驱动bug同时发生导致死循环出现

## 低负载|响应慢 场景
- 原因
    - 死锁
- 排查方法
    - 打印栈信息

