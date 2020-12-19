# 部分自定义types


# afl-gcc
此模块功能介绍：

## find_as函数
函数功能：寻找afl-as的路径
- 检查是否存在AFL_PATH这个环境变量,如果存在进一步检查AFL_PATH/as是否可以访问，可以将其作为as_path
- 如果不存在AFL_PATH这个环境变量，则检查函数的输入，并截取尾段的’/‘和其后的字符。拼接上afl-as后检查其是否可以访问，若可以则将其作为as_path

## edit_params函数
函数功能：对传入的命令行参数进行相应处理并转储至cc_params[]
- 为cc_params分配内存，分配的长度为(argc+128)*8
- 过滤argv[0]中的‘/’，将其赋值给name，通过name为afl-clang或afl-clang++设置相关的配置环境
- 从argv[1]开始遍历argv的参数 
  - 跳过 -B(会通过标志位报警但也不会做实质性处理) -integrated-as -pipe参数
  - 如果接收到 -fsanitize=address 或 -fsanitize=memory 参数则设置 asan_set 标志位为1
  - 如果接收到 FORTIFY_SOURCE 则设置fortify_set为1 
  - 将剩余的参数传入cc_params[]
- 处理cc_params(实际上是个参数存储数组)
  - 将 -B 之前得到的 as_path 传入 cc_params[]
  - 检查是否使用clang或是clang++，是则存入 -no-intergrated-as
  - 是否存在AFL_HEADEN环境变量，存在则传入 -fstack-protector-all 参数和是否存在 fortify_set,不存在则传入 -D_FORTIFY_SOURCE=2
- 判断使用 ASAN MASN 
  - 通过asan_set与默认的环境配置AFL_USE_ASAN判断是否使用ASAN (不可同时使用MASN AFL_HEADRN)
  - 通过配置环境AFL_USE_MSAN判断是否使用MASN，如果使用则传入参数 -U_FORTIFY_SOURCE -fsanitize=memory (不可同时使用ASN AFL_HEADRN)
  - 如果不存在环境变量 AFL_DONT_OPTIMIZE 则传入 -g -03 -funroll-loops -DFUZZING_BUILD_MODE_UNSAFE_FOR_PRODUCTION=1       
  - 如果存在环境变量 AFL_NO_BUILTIN 则传入 -fno-builtin-xxxxx
  - 最后传入NULL停止对命令行参数的处理

## main函数
- 找到afl—as的路径  
- 设置gcc需要的一些基本参数   