# Mac m2 环境下编译ollvm-9.0

ollvm 下载地址

[https://github.com/heroims/obfuscator/tree/llvm-9.0](https://github.com/heroims/obfuscator/tree/llvm-9.0)

ndk 对应版本 21.4.7075529

cmake 版本 3.22.1， 使用Android sdk下的cmake

### 编译

```shell
第一步在你android sdk目录下的cmake目录执行

./cmake -S llvm -B build -G Ninja -DCMAKE_BUILD_TYPE=Release 
-DLLVM_INCLUDE_TESTS=OFF -DLLVM_ENABLE_NEW_PASS_MANAGER=OFF  
obfuscator-llvm-9.0

第一步执行完成会在当前目录下创建一个build文件夹

第二步

./cmake --build build -j8
```

等待编译完成，预计20分钟

### 替换android ndk 目录下的文件

1、先备份 对应的ndk目录，防止损坏

2、把ndk目录下的，clang-9 clang-check clang clang-cl clang-cpp clang-format clang-diff clang++ 这些文件删除

`/Library/Android/sdk/ndk/21.4.7075529/toolchains/llvm/prebuilt/darwin-x86_64/bin/`

3、复制llvm编译出来的文件到ndk目录下
`Library/Android/sdk/cmake/3.22.1/bin/build/bin/` 复制当前目录下刚刚删除的文件到 ndk目录下

4、在android studio 中编译 jni工程，如果出现找不到 .h文件的错误
在编译结果目录下复制对应的 .h文件到ndk目录下,哪个文件找不到就复制哪一个
ndk对应目录：
`/Library/Android/sdk/ndk/21.4.7075529/toolchains/llvm/prebuilt/darwin-x86_64/sysroot/usr/include/`
编译结果对应目录：
`Library/Android/sdk/cmake/3.22.1/bin/build/lib/clang/9.0.0/include/`

5、在你的build.gradle 中配置

```gradle
android {
    defaultConfig {
        externalNativeBuild {
            cmake {
		//混淆参数，一般来说使用这个配置就够了，包体积也在小
                cppFlags "-std=c++14", "-frtti", "-fexceptions", "-DANDROID_TOOLCHAIN=clang",
                        "-ffunction-sections", "-fdata-sections",
                        "-mllvm", "-sub", "-mllvm", "-sub_loop=1",
                        "-mllvm", "-fla",
                        "-mllvm", "-split", "-mllvm", "-split_num=1",
                        "-mllvm", "-bcf", "-mllvm", "-bcf_loop=1",/* "-mllvm", "-bcf_prob=40",*/
                        "-mllvm", "-sobf", "-mllvm", "-aesSeed=0xada46ab5da824b96a18409c49dc91dc3"
            }
            ndk {
                abiFilters 'armeabi-v7a', 'arm64-v8a', "x86", "x86_64"
            }
        }
    }
    externalNativeBuild {
        cmake {
            path "src/main/cpp/CMakeLists.txt"
            version "3.22.1"//指定对应的版本
        }
    }
//指定对应的版本
    ndkVersion '21.4.7075529'
}
```

### 如果无法编译，下载我在mac电脑编译的文件






