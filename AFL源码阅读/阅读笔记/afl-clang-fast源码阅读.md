# afl-fast-clang
## find_obj函数
函数功能：寻找 afl-llvm-rt.o 文件路径作为obj_path
- 获取 afl-llvm-rt.o 的路径作为obj_path
  - 获取环境变量 AFL_PATH，如果存在，就去读取AFL_PATH/afl-llvm-rt.o是否可以访问，如果可以就设置AFL_PATH为obj_path，然后直接返回
  - 如果没有，就检查argv0中的文件同级路径其下是否存在 afl-llvm-rt.o 文件，看是否能够访问，如果可以就设置这个目录为obj_path，然后直接返回
  - 最后如果上面两种都找不到，因为默认的AFL的MakeFile在编译的时候，会定义一个名为AFL_PATH的宏，其指向/usr/local/lib/afl，会到这里找是否存在afl-llvm-rt.o，如果存在设置AFL_PATH为obj_path并直接返回
  - 如果都找不到则报错
## edit_params函数
函数功能：将给定参数导入 cc_params
- 读取第一个参数的文件名作为name
  - 若name为afl-clang-fast++则获取环境变量 AFL_CXX 传给 alt_cxx 并将其作为 cc_params[0]
  - 若非 则获取环境变量 AFL_CC 传给 alt_cc 并将其作为 cc_params[0]
- 判断是否定义了USE_TRACE_PC的宏
  - 是 则将 '-fsanitize-coverage=trace-pc-guard' 作为 cc_params 后续参数 
    - 再通过判断是否定义 __ANDROID__  是则将'-mllvm' '-sanitizer-coverage-block-threshold=0' 作为 cc_params 后续参数
  - 否 则将 '-Xclang' '-load' '-Xclang' 'obj_path/afl-llvm-pass.so' '-Qunused-arguments' 依次添加到 cc_params 里
- 依次读取我们传给afl-clang-fast的参数，并添加到 cc_params
  - 如果传入参数里有 -m32 或者 armv7a-linux-androideabi，就设置bit_mode为32
  - 如果传入参数里有-m64，就设置 bit_mode 为64
  - 如果传入参数里有 -x，就设置 x_set 为1
  - 如果传入参数里有 -fsanitize=address 或者 -fsanitize=memory，就设置asan_set为1
  - 如果传入参数里有 -Wl,-z,defs 或者 -Wl,--no-undefined，就直接pass掉，不传给clang
- 读取环境变量AFL_HARDEN
  - 存在，就在 cc_params 里添加 -fstack-protector-all
    - 如果 fortify_set 为0则添加 -D_FORTIFY_SOURCE=2 至 cc_params
  - 如果参数里没有 -fsanitize=address/memory 就读取环境变量AFL_USE_ASAN，如果存在就添加 -U_FORTIFY_SOURCE -fsanitize=address 到cc_params里，环境变量 AFL_USE_MSAN 同理
- 如果定义了USE_TRACE_PC 宏，就检查是否存在环境变量 AFL_INST_RATIO，如果存在就抛出异常
- 读取环境变量AFL_DONT_OPTIMIZE，如果不存在就添加-g -O3 -funroll-loops到参数里
- 读取环境变量AFL_NO_BUILTIN，如果存在就添加-fno-builtin-strcmp -fno-builtin-strncmp -fno-builtin-strcasecmp -fno-builtin-strncasecmp -fno-builtin-memcmp
- 添加参数-D__AFL_HAVE_MANUAL_CONTROL=1 -D__AFL_COMPILER=1 -DFUZZING_BUILD_MODE_UNSAFE_FOR_PRODUCTION=1 至 cc_params
- 添加两条宏指令__AFL_LOOP( ), __AFL_INIT( )
  ```shell
  #define __AFL_LOOP() \
  do { \
      static char *_B; \
      _B = (char*)"##SIG_AFL_PERSISTENT##"; \
      __afl_persistent_loop(); \
  }while (0)

  #define __AFL_INIT() \
  do { \
      static char *_A;  \
      _A = (char*)"##SIG_AFL_DEFER_FORKSRV##"; \
      __afl_manual_init(); \
  } while (0)
  ```
- 如果x_set为1，则添加参数-x none
- 为android做的处理
## main函数
- 寻找obj_path路径
- 编辑参数cc_params
- 替换进程空间，执行要调用的clang和为其传递参数
