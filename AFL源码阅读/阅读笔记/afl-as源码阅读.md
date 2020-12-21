# afl-as
## edit_params函数
函数功能：检查并传递参数
- u8* tmp_dir
  - 依次检查环境变量中是否存在 TMPDIR/TEMP/TMP/ 存在就取其值存入tmp_dir，如果都不存在，则存入‘/tmp’
- u8* afl_as
  - 读取 AFL_AS 中的环境变量的值，存在则设置为afl_as的值
  - 对于 APPLE 的特殊处理
- as_params[]
  - 如果afl_as不为空，就设置as_params[0]为afl_as，不然就设置为as
  - 从argv[]中获取参数，传入as_params[]中
- input_file
  - 获取0之前的最后一个参数传入 input_file 中
  - 检查 input_file 是否是 -version 是则设置just_version为1 若是 -‘others’则报错中断
  - 检查 input_file 是否是 $tmp_dir /var/tmp/ /tmp/ 不是则设置pass_thru为1
- modified_file
  - 设置其值为 tmp_dir/.afl-pid-time.s
  - 将其传入as_params[]
## add_instrumentation
函数功能：处理输入文件，产生
- 判断 input_file 是否存在，存在则打开，不存在则获取标准输入存入文件
- 读取input_file文件将其转储至modified_file中
  - 判断此行是否以 \t + 字母开头，若是则存储此行至modified_file
  - 判断此行是否以 
    - \t.[text\n|section\t.text|section\t__TEXT,__text|section __TEXT,__text]开头，若是则设置instr_ok为1，继续读取下一行
    - \t.[section\t|section |bss\n|data\n]开头，若是则设置instr_ok为0，继续读取下一行
  - 判断此行是否以 \tj[!m] 开头，则插桩 并在完全拷贝input_file之后向modified_file 中写入
## main函数
函数功能：
- 读取环境变量AFL_INST_RATIO的值，设置为inst_ratio_str
- 设置srandom的随机种子为rand_seed = tv.tv_sec ^ tv.tv_usec ^ getpid()
- fork出一个子进程，让子进程来执行as
- 等待子进程结束，读取环境变量AFL_KEEP_ASSEMBLY的值，如果没有设置这个环境变量，就unlink掉modified_file