ucore在ppc64le（ubuntu14）上的编译环境部署（交叉编译器的生成适用于所有linux系统）

1.安装qemu
```
sudo apt-get update

sudo apt-get install qemu-system-ppc
```
2.i386-elf-gcc交叉编译器（root权限下，适用于所有linux系统，其余系统见参考文档说明）
([参考文档](http://wiki.osdev.org/GCC_Cross-Compiler))

* (1) 设置环境变量：
```
export PREFIX="/usr/local/cross" 
export TARGET=i386-elf
```
修改/etc/profile文件添加如下内容：
```
export PATH="$PREFIX/bin:$PATH"
```
使配置生效：
```
source /etc/profile
```
进入编译目录
```
cd /usr/local/cross
mkdir src && cd src
```
* (2) binutils编译：

安装依赖
```
sudo apt-get install g++
```
下载源码并解压：
```
nohup curl "ftp://ftp.gnu.org/gnu/binutils/binutils-2.24.tar.gz" -o ./binutils-2.24.tar.gz >tmp.log 2>&1 &
tar zxvf binutils-2.24.tar.gz
```
创建binutils编译目录
```
mkdir binutils && cd binutils
```
编译安装
```
../binutils-2.24/configure --target=$TARGET --prefix="$PREFIX" --with-sysroot --disable-nls --disable-werror
make
make install
```
* (3) gcc编译

安装依赖
```
sudo apt-get install libmpc-dev libmpfr-dev libgmp3-dev
```
下载源码并解压
```
nohup curl "ftp://ftp.gnu.org/gnu/gcc/gcc-4.9.2/gcc-4.9.2.tar.gz" -o ./gcc-4.9.2.tar.gz >tmp.log 2>&1 &
tar zxvf gcc-4.9.2.tar.gz
```
创建gcc编译目录
```
mkdir gcc && cd gcc
```
编译安装
```
../gcc-4.9.2/configure --build=ppc64le --target=$TARGET --prefix="$PREFIX" --disable-nls --enable-languages=c,c++ --without-headers

make all-gcc
make all-target-libgcc
make install-gcc
make install-target-libgcc
```

创建软链接，ucore默认使用i386-elf-gcc-4.8
```
cd /usr/local/cross/bin
ln -s i386-elf-gcc-4.9.2 i386-elf-gcc-4.8
```
修改权限
```
chmod -R og+rx /usr/local/cross
```
测试
```
i386-elf-gcc-4.8 --version
```

