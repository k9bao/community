# 1. build ffmpeg

- [1. build ffmpeg](#1-build-ffmpeg)
  - [1.1. pre-install](#11-pre-install)
  - [1.2. create](#12-create)
  - [1.3. install](#13-install)
  - [1.4. 可能问题](#14-可能问题)

## 1.1. pre-install

- `brew install pkb-config`
- `brew install freetype`

x264 和 x265 依赖 gpl, gpl 依赖 openssl 和 nonfree
所以修改修改 conanfile.py 中的 build_configure 修改如下代码

```python
if self.options.x264 or self.options.x265 or self.options.postproc:
    if self.options.openssl is False:
        args.append("--enable-openssl")
    args.append("--enable-nonfree")
    args.append("--enable-gpl")
```

## 1.2. create

```bash
cd ffmpeg/all
conan create . ffmpeg/4.2.1@xingbao/stable
```

```bash
conan create -o ffmpeg:vpx=False -o ffmpeg:fdk_aac:=False -s:b build_type=Debug . ffmpeg/4.2.1@xingbao/stable
# 执行时，报错没有 fdk_aac 的option，目前直接代码设置 fdk_aac=False，然后这里去掉对fdk_aac的设置
```

- `-o` 等价于 `-o:h`
  - 比如: `-o ffmpeg:vpx=False -o ffmpeg:fdk_aac:=False`
- `-o:b` OPTIONS_BUILD, --options:build OPTIONS_BUILD
- `-o:h` OPTIONS_HOST, --options:host OPTIONS_HOST

- 执行结果
  - 在 `ffmpeg/all/test_package` 下生成 `build` 目录，里边有 `a091445f35b5b19fb7b1db545a35bbd33b4abcae`目录(系统编译参数不一样，名称可能不一样)
  - `ffmpeg/all/test_package/a09...cae/bin` 目录里有测试可执行程序
  - `ffmpeg/all/test_package/a09...cae/conanbuildinfo.cmake` 是ffmpeg依赖的cmake文件，使用方法可以参考 `ffmpeg/all/test_package/CMakeLists.txt`
  - `/Users/guoxb/.conan/data/ffmpeg/4.2.1/xingbao/stable/package/181a1737095256b6265fd78fb9d82f77d2f73dca`目录下生成`include`和`lib`

## 1.3. install

如果没有生成 `conanbuildinfo.cmake` 则可以通过 `conan install` 来生成

`conan install -g cmake -o ffmpeg:vpx=False -o ffmpeg:fdk_aac=False ffmpeg/4.2.1@xingbao/stable`
`conan install -g cmake ffmpeg/4.2.1@xingbao/stable`

`-g` 可选参数可以[参考](https://docs.conan.io/en/latest/reference/generators.html)

## 1.4. 可能问题

libfdk找不到

```shell
conan remove libfdk_aac:2.0.0
git clone https://github.com/bincrafters/conan-libfdk_aac
cd conan-libfdk_aac
conan create . 
conan search libfdk_aac:2.0.0
```
