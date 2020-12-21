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
## add_add_instrumentation
函数功能：
