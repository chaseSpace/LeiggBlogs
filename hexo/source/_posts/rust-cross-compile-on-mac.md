---
title: 【Rust】在mac上交叉编译linux和windows程序（包含Docker实现）
date: 2022-12-05
desc: 步骤详解+踩坑说明，来到这，你应得~
tags:
- Rust
- 技术笔记
category:
- 技术
---

###  1.基本步骤
-	使用rustup target add <target>  安装目标的标准库
-	rustup target list  查看支持的target列表
-	安装target链接器
-	更新cargo.toml，让rustc知道使用哪个链接器
-	cargo build --release --target TARGET_NAME

<br>

### 2.准备
整个过程可能需要在命令行下载github包，提前设置代理（根据自身情况修改）：
>export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890

<br>

### 3.在mac上编译linux程序
#### 3.1 准备
- 选择对的target（带有目标平台的标准库）
>linux常见的target有 x86_64-unknown-linux-gnu 和 x86_64-unknown-linux-musl
>-	musl的特点是轻量、无依赖，能运行在任何linux系统。缺点是编译后程序比glibc慢点。
>-	gnu的特点是更快，生成动态链接库，依赖所在平台安装glibc
>- 下文会同时介绍两者的安装步骤，请读者先选一种进行安装（你可能会遇到一些问题）
- 查看Rust支持的target list：
```
lei@WilldeMacBook-Pro learn_rust % rustup target list |sort -R
i686-unknown-freebsd
aarch64-apple-ios
x86_64-unknown-linux-musl (installed)
armv5te-unknown-linux-musleabi
i586-unknown-linux-gnu
arm-linux-androideabi
x86_64-linux-android
...几十种
```
- 查看已安装的target：
```
lei@WilldeMacBook-Pro learn_rust % rustup target list |grep installed
aarch64-apple-darwin (installed)   -- 这个默认安装的，因为当前平台就是m1pro，下面是手动安装的
x86_64-pc-windows-gnu (installed)
x86_64-pc-windows-msvc (installed)
x86_64-unknown-linux-gnu (installed)
x86_64-unknown-linux-musl (installed)
```
#### 3.2 安装target
安装musl：`rustup target add x86_64-unknown-linux-musl`
安装gnu：`rustup target add x86_64-unknown-linux-gnu`
#### 3.3 安装链接器
安装musl链接器：`brew install filosottile/musl-cross/musl-cross`
安装gnu链接器：`brew install SergioBenitez/osxct/x86_64-unknown-linux-gnu`

#### 3.3 更改Cargo.toml
增加以下
```
[target.x86_64-unknown-linux-musl] # 也支持[build]节点 设置target=...
linker = "x86_64-linux-musl-gcc"
[target.x86_64-unknown-linux-gnu]
linker = "x86_64-unknown-linux-gnu-gcc"
```
#### 3.3 开始编译
使用musl:  `TARGET_CC=x86_64-linux-musl-gcc cargo build --release --target x86_64-unknown-linux-musl`
使用gnu： `TARGET_CC=x86_64-unknown-linux-gnu cargo build --release --target x86_64-unknown-linux-gnu`

#### 3.4 故障排查
问题：若最后编译时报：ld: unknown option: --as-needed
解决：使用另一个辅助工具[cargo-zigbuild](https://github.com/messense/cargo-zigbuild)编译（这个工具也是rust写的专门用来方便交叉编译，解决一些奇奇怪怪问题）
```
cargo install cargo-zigbuild
brew install zig
cargo zigbuild --release --target x86_64-unknown-linux-musl
或
cargo zigbuild --release --target x86_64-unknown-linux-gnu
```
<br>

### 4. 在mac上编译windows程序
简化记录：
```
rustup target add x86_64-pc-windows-gnu
brew install mingw-w64   # 链接器
```
更改Cargo.toml，增加以下
```
[target.x86_64-pc-windows-gnu]
linker = "x86_64-w64-mingw32-gcc"
# [target.x86_64-pc-windows-msvc]  不需要填写linker
```

>说明：若要在mac/linux上编译基于x86_64-pc-windows-msvc的windows程序，请使用另一个辅助工具：[cargo-xwin](https://github.com/messense/cargo-xwin)

#### 4.1 故障排查
问题：最后编译msvc时：`cargo xwin build --target x86_64-pc-windows-msvc`，报错：
```
error: failed to run custom build command for `rust-crypto v0.2.36`

Caused by:
  process didn't exit successfully: `/Users/lei/Desktop/Rust/learn_rust/target/debug/build/rust-crypto-a19246e40d1c0ac5/build-script-build` (exit status: 101)
  --- stdout
  TARGET = Some("x86_64-pc-windows-msvc")
  OPT_LEVEL = Some("0")
  TARGET = Some("x86_64-pc-windows-msvc")
  HOST = Some("aarch64-apple-darwin")
  TARGET = Some("x86_64-pc-windows-msvc")
  TARGET = Some("x86_64-pc-windows-msvc")
  HOST = Some("aarch64-apple-darwin")
  CC_x86_64-pc-windows-msvc = None
  CC_x86_64_pc_windows_msvc = Some("clang-cl --target=x86_64-pc-windows-msvc")
  TARGET = Some("x86_64-pc-windows-msvc")
  HOST = Some("aarch64-apple-darwin")
  CFLAGS_x86_64-pc-windows-msvc = None
  CFLAGS_x86_64_pc_windows_msvc = Some("-Wno-unused-command-line-argument -fuse-ld=lld-link /imsvc/Users/lei/Library/Caches/cargo-xwin/xwin/crt/include /imsvc/Users/lei/Library/Caches/cargo-xwin/xwin/sdk/include/ucrt /imsvc/Users/lei/Library/Caches/cargo-xwin/xwin/sdk/include/um /imsvc/Users/lei/Library/Caches/cargo-xwin/xwin/sdk/include/shared")
  DEBUG = Some("true")
  running: "clang-cl --target=x86_64-pc-windows-msvc" "-O0" "-ffunction-sections" "-fdata-sections" "-fPIC" "-Wno-unused-command-line-argument" "-fuse-ld=lld-link" "/imsvc/Users/lei/Library/Caches/cargo-xwin/xwin/crt/include" "/imsvc/Users/lei/Library/Caches/cargo-xwin/xwin/sdk/include/ucrt" "/imsvc/Users/lei/Library/Caches/cargo-xwin/xwin/sdk/include/um" "/imsvc/Users/lei/Library/Caches/cargo-xwin/xwin/sdk/include/shared" "-g" "--target=x86_64-pc-windows-msvc" "-Wall" "-Wextra" "/Fo/Users/lei/Desktop/Rust/learn_rust/target/x86_64-pc-windows-msvc/debug/build/rust-crypto-21fe17a56301ca99/out/src/util_helpers.o" "/c" "src/util_helpers.c"

  --- stderr
  thread 'main' panicked at '

  Internal error occurred: Failed to find tool. Is `clang-cl --target=x86_64-pc-windows-msvc` installed?
```

是因为代码依赖的rust-cypto库内部依赖了旧版本gcc库，新版的clang-cl并不支持它，所以最简单的办法是替换截止2022超过6年未更新的rust-crypto库，使用新的[RustCrypto](https://github.com/RustCrypto)库。
>博主尝试brew去安装旧版本的gcc，但提示上游已废弃该版本gcc，无法安装；遂不再继续尝试（截止发文，博主已经研究Rust交叉编译超过6h）

解决：修改Cargo.toml
```
[dependencies] 
# rust-crypto = "0.2.36"
sha2 = "0.10.6"
sha3 = "0.10.6"
```
<br>

### 5. 更简单的做法——使用Docker
不论什么语言，想要实现开发者友好的交叉编译，还得把这项工作交给Docker，Rust也不例外。
这样既不用折腾开发者的主机，也不用消耗大量时间在处理各种依赖问题上。
以下参考 [rust-musl-cross](https://github.com/messense/rust-musl-cross)

#### 5.1 拉取需要的image
如x86_64-musl，其他请从上述链接中寻找
```
$ docker pull messense/rust-musl-cross:x86_64-musl
```
#### 5.2 修改config
```
cd PROJECT/src/   # 进入项目的src目录
vi ~/.cargo/config    添加target部分（与Cargo.toml的target一致）如下

[target.x86_64-unknown-linux-musl] 
linker = "x86_64-linux-musl-gcc"
```
注意，这一步很重要，因为在博主操作时，发现原本在mac本地构建时基于Cargo.tom的target配置到了容器中居然不生效了，效果是构建时报错，错误关键字是`cc: error: unrecognized command-line option '-m64'` 或 `ld: unknown option: --as-needed`，其实就是rust编译器无法通过配置找到正确的linker导致的，所以请读者切记。
另外，如果你想要知道其他target的信息怎么填，在下一步骤中，去掉.cargo/config映射，然后直接执行rust_builder，进入容器后，`cat ~/.cargo/config`，将其内容复制到你本地的~/.cargo/config即可。
#### 5.3 构建docker命令
```
alias rust_builder='docker run -it --rm -v "$(pwd)":/home/rust/src \
                -v "/Users/lei/.cargo/config:/root/.cargo/config" \
                -v "/Users/lei/.cargo/registry:/root/.cargo/registry" \
                -v "/Users/lei/.cargo/git:/root/.cargo/git" \
            messense/rust-musl-cross:x86_64-musl'
# 注意替换上面-v选项后面的路径为你本地路径，它们主要提供构建的缓存功能，以实现下次构建的加速，避免重复工作

# nightly构建
rust_builder rustup default nightly && rustup target add x86_64-unknown-linux-musl && cargo build --target x86_64-unknown-linux-musl --release
# stable构建
rust_builder cargo build --target x86_64-unknown-linux-musl --release
```


#### 5.4 故障排查
在尝试构建target=`armv7-unknown-linux-musleabihf`时，build时遇到如下报错：
```
error: linker `armv7-unknown-linux-musleabihf-gcc` not found
  |
  = note: No such file or directory (os error 2)
```
然后博主转念一想，要是进入容器中再执行命令会不会有所不同，于是有以下步骤:
```
lei@WilldeMacBook-Pro learn_rust % alias rust_builder='docker run -it --rm -v "$(pwd)":/home/rust/src \                                                                                
                -v "/Users/lei/.cargo/config:/root/.cargo/config" \
                -v "/Users/lei/.cargo/registry:/root/.cargo/registry" \
                -v "/Users/lei/.cargo/git:/root/.cargo/git" \
            messense/rust-musl-cross:armv7-musleabihf'
lei@WilldeMacBook-Pro learn_rust % rust_builder
root@2fb71f01c5f4:/home/rust/src# rustup default nightly && rustup target add armv7-unknown-linux-musleabihf && cargo build --target armv7-unknown-linux-musleabihf
info: syncing channel updates for 'nightly-aarch64-unknown-linux-gnu'
info: latest update on 2022-10-23, rust version 1.66.0-nightly (6e95b6da8 2022-10-22)
info: downloading component 'cargo'
info: downloading component 'rust-std'
...

    Building [=========================> ] 94/95: learn_rust(bin)                                                                                                                                                                                                                                                     
warning: `learn_rust` (bin "learn_rust") generated 151 warnings (1 duplicate)
    Finished dev [unoptimized + debuginfo] target(s) in 2m 11s
root@2fb71f01c5f4:/home/rust/src# 
```
Wow！如你所见，这样居然就可以了，博主也不知道问题所在，读者朋友若知晓一二还请评论区留言指导一下~
<br>

### 总结
本文系博主花不少时间踩坑（结合cnblog/知乎/stackoverflow/github/rust官网等站点）总结而来，写文不易，点赞再走，交个朋友~



<br>
<br>

---
##### 其他资料
- [缩小Rust编译产物体积](https://github.com/johnthagen/min-sized-rust#strip-symbols-from-binary)

