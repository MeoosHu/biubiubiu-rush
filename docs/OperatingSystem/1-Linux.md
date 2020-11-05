### 1.Linux常用命令

- 常用指令：

  ls/ll、cd、mkdir、rm-rf、cp、mv、ps -ef | grep xxx、kill、**free-m**、tar -xvf file.tar

- 根据进程id查看堆栈信息:gstack pid

- 查询进程ID（pid）：

  （1）ps -ef | grep xxx

  （2）ps -aux | grep xxx（-aux显示所有状态）

- 杀掉进程：kill -9 [pid]

- 查看具体端口号的状态：netstat -anp |grep 端口号（状态为LISTEN表示被占用）

- 启动/停止服务：cd到bin目录->./startup.sh（打开，需有足够权限）| ./shutdown.sh（关闭）

- 查看日志

  （1）cd到服务器的logs目录（里面有xx.out文件）

  （2）tail -f xx.out --此时屏幕上实时更新日志。ctr+c停止

  （3）查看最后100行日志 tail -100 xx.out 

  （4）查看关键字附件的日志。如：cat filename | grep -C 5 '关键字'（关键字前后五行。B表示前，A表示后，C表示前后） ----使用不多

  （5）还有vi查询啥的。用的也不多。

- 查找文件

  （1）查找大小超过xx的文件： find . -type f -size +xxk -----(find . -type f -mtime -1 -size +100k -size-400k)--查区间大小的文件

  （2）通过文件名：find / -name xxxx  ---整个硬盘查找

- vim（vi）编辑器　　

  1. **命令模式**：查找内容(/abc、跳转到指定行(20gg)、跳转到尾行(G)、跳转到首行(gg)、删除行(dd)、插入行(o)、复制粘贴(yy,p)
  2. **输入模式**：编辑文件内容
  3. **末行模式**：保存退出(wq)、强制退出(q!)、显示文件行号(set number)。在命令模式下，输入a或i即可切换到输入模式，输入冒号(:)即可切换到末行模式；在输入模式和末行模式下，按esc键切换到命令模式













