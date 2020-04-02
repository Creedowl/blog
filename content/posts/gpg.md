---
title: "GPG使用笔记"
date: 2020-04-02T14:47:31+08:00
draft: false
toc: true
images:
tags: 
  - gpg
  - 学习笔记
---

## Why GPG?

[GPG](https://zh.wikipedia.org/wiki/GnuPG) 是一个密码学软件，用于加密、签名通信内容及管理非对称密码学的密钥，具体历史、介绍可以在wiki中看到，[官网](https://gnupg.org/)

而我使用 GPG 的缘由是，在 Github 上提交时，如果使用 GPG 对提交进行签名，会有一个`Verified`标识~~看起来很nb~~

![UTOOLS1585795152275.png](https://blog-1251984664.cos.ap-shanghai.myqcloud.com/UTOOLS1585795152275.png/compress)

当然使用 GPG 不仅是为了好看，它能确保你提供的信息是由你自己发布的，无法被别人篡改、冒充，前提是保护好自己的 GPG 私钥

## 安装

这边以 MacOS 为示例进行操作

**强烈建议 MacOS 用户使用 [GPG Suite](https://gpgtools.org/) 进行操作，可以避免一些奇奇怪怪的问题**

```shell
brew cask install gpg-suite
```

~~仅安装 GPG 本体~~

```shell
# deprecated
brew install gpg
```

## 生成密钥

GPG 采用非对称加密，一切的操作都需要有密钥。网上一些教程中的操作与我使用的 GPG 版本 `gpg (GnuPG/MacGPG2) 2.2.17` 有点出入，以下操作都在我所使用的版本下进行

有两个命令用于生成密钥，为`gpg --gen-key`和`gpg --full-gen-key`，区别是前者采用默认的配置不可更改，后者能自定义配置。我们采用后一种，首先选择密钥类型，默认的`1`就可以了

```shell
$ gpg --full-gen-key
gpg (GnuPG/MacGPG2) 2.2.17; Copyright (C) 2019 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

请选择您要使用的密钥类型：
   (1) RSA 和 RSA （默认）
   (2) DSA 和 Elgamal
   (3) DSA（仅用于签名）
   (4) RSA（仅用于签名）
您的选择是？ 1
```

接着选择密钥长度，默认`2048`即可

```text
RSA 密钥的长度应在 1024 位与 4096 位之间。
您想要使用的密钥长度？(2048) 2048
```

然后设定密钥的有效期限，我们没有特殊需求，设置永不过期即可

```text
请设定这个密钥的有效期限。
         0 = 密钥永不过期
      <n>  = 密钥在 n 天后过期
      <n>w = 密钥在 n 周后过期
      <n>m = 密钥在 n 月后过期
      <n>y = 密钥在 n 年后过期
密钥的有效期限是？(0) 0
```

确认上述设置后，依次输入姓名、邮箱和注释，这里我使用随机信息用于演示

    GnuPG 需要构建用户标识以辨认您的密钥。
    
    真实姓名： naive
    电子邮件地址： naive@test.com
    注释： too young too simple
    您选定了此用户标识：
        “naive (too young too simple) <naive@test.com>”
    
    更改姓名（N）、注释（C）、电子邮件地址（E）或确定（O）/退出（Q）？

确定后，需要设定密钥的密码，如果你采用了 `GPG Suite` ，会出现一个窗口，在此输入密码

![UTOOLS1585796741624.png](https://blog-1251984664.cos.ap-shanghai.myqcloud.com/UTOOLS1585796741624.png/compress)

如果没使用`GPG Suite`，则会在你当前终端弹出一个全屏终端窗口，同样是输入密码。

> 注意，在 mac 终端环境下，如果语言是中文的话，这个终端窗口可能会乱码，但不影响使用，也可以临时设置语言为英文解决这个问题`LANGUAGE=en gpg --full-generate-key`。但千万不要用`ctrl-c`结束这个进程，这会破坏当前终端环境，后面的输入会变得很奇怪，而且后台进程不会结束，会占满你的CPU

```text
我们需要生成大量的随机字节。在质数生成期间做些其他操作（敲打键盘
、移动鼠标、读写硬盘之类的）将会是一个不错的主意；这会让随机数
发生器有更好的机会获得足够的熵。
gpg: 密钥 4C26D2E451FFDAF1 被标记为绝对信任
gpg: 目录‘/Users/creedowl/.gnupg/openpgp-revocs.d’已创建
gpg: 吊销证书已被存储为‘/Users/creedowl/.gnupg/openpgp-revocs.d/0B3C52F1EF07B14EA7C45C9A4C26D2E451FFDAF1.rev’
公钥和私钥已经生成并被签名。

pub   rsa2048 2020-04-02 [SC]
      0B3C52F1EF07B14EA7C45C9A4C26D2E451FFDAF1
uid                      naive (too young too simple) <naive@test.com>
sub   rsa2048 2020-04-02 [E]
```

这样，密钥就生成完成，其中`4C26D2E451FFDAF1`就是你的密钥名，后续操作都需要通过这个密钥名选择密钥，但也可以通过之前设置的姓名或邮箱代替操作

## GPG 常用操作

GPG 最基础的操作就是对文件等信息进行加密、签名，首先创建一个测试文件`test.txt`

```shell
echo test > test.txt
```

### 加密

```shell
gpg --recipient creedowl --output test.txt.gpg --encrypt test.txt
```

其中`--recipient`指定接收者，简写为`-r`，设定接受者时需要你本地有接收者的**公钥**，可以用密钥名、姓名或者邮箱进行指定。`--output`指定输出文件名，可省略。`--encrypt`指定待加密的文件，简写为`-e`。这样在你当前目录下会生成一个`test.txt.gpg`即加密后的文件，你就可以把这个文件发送给接受者，不必担心文件内容被第三方读取

**注意**这里生成加密文件为二进制格式，有时候（如通过邮件发送）不便于操作，可以加上`--armor`参数（简写为`-a`）生成文本格式的密文，如

```shell
gpg -r creedowl -ae test.txt
```

生成的加密文件为`test.txt.asc`

```text
-----BEGIN PGP MESSAGE-----

hQEMA9KmvDhAMTGrAQf+KITqx4HTpWsevIEfRFlw5g+jt8l8ol3fuww0gBoMEPBG
KaqxCmjIpHZdGmaXwBSKadmeieZGLsm1CYuf4gbv4FpdWgveVWnLvB/HmwuvnmJt
YhTtapNgkQ++mwPbH4WKeOFQ58U5dZ7C+R8rrucRRYQBRTH6AJQY82aV1KXTqyF1
D8c/2ANW7aCWr8W/ogCbEqH47VE3foP+aSD+JMPtdnaYWIPzyr5x5lw6ASwxD4hB
ikOUcJTxp+GyFLTd3n1zCvSP9ZQzvolur7soqZVP8t4hxlALGL+KYN4YgubcPt5y
bucMYWkbLlZM+AuAYzSCLaSmkgkiYUFEkwKJWdLr+NJGAY5Mk+bnEP2k/kKxl9vo
/LVsloiYbg9M184LxUNfZRBnKcI9vKw/C6uU2Ee5wKUMsy3Epnp60Fe5u5fj5yn7
0JjMBmY+gQ==
=8K/i
-----END PGP MESSAGE-----
```

### 解密

有了密文就要对其进行解密得到原文。注意要进行解密，你必须有接受者的**私钥**。公钥是公开的，任何人都可以查看、使用，而**私钥**是私有的，一定要保管好，如果泄露了这个 GPG 密钥就等同于失效了，需要吊销。这就是非对称加密

```shell
	gpg --decrypt test.txt.asc --output out.txt
```

其中`--decrypt`声明该命令进行解密，简写为`-d`，`--output`指定输出文件，默认输出到终端。注意解密时需要输入你密钥的密码，所以也一定要保管好密码

### 签名

有的时候不需要对文件进行签名，只需要确保文件是由本人发出的，这个时候就可以用到签名

```shell
gpg --sign test.txt
```

会生成二进制的签名文件`test.txt.gpg`，如果想生成文本类型的签名，使用

```shell
gpg --clearsign test.txt
```

会生成文本类型的签名文件`test.txt.asc`

注意以上两种方式原文和签名是在同一个文件中，查看`test.txt.asc`

```text
-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA256

test
-----BEGIN PGP SIGNATURE-----

iQFDBAEBCAAtFiEECzxS8e8HsU6nxFyaTCbS5FH/2vEFAl6FX+cPHG5haXZlQHRl
c3QuY29tAAoJEEwm0uRR/9rxGNsH/AmjJiT19q5caq9Pe8tMRnR2yvZIGWbMD3Ch
UTkyGMkSPMIo6XRpYPlANe2/dddBT2l7EAPy2lAxaa8LcjsxtRhJG4GxMaCcujik
M6MX3IOfa1m9aDP1IY3Sh2U07j3t+8dawahR8EUF2D+rtvx24ybnwpWpKbDVsGyp
Mt06BI3pWC3opMRYPCl21IBhPhaPPnvXF7X8DMP+2VqZye2juX2oG1KMkZWQuiBb
/D7DYlx6/ainu0cJw6UL8YXHag7tkUeKtTPQbBpCsszOuxJZ74gduIUzDcokayCB
DM9qB/GpUyVOOTwfLMBta6JP5BO8ppPaPdPX46kKDqLKG7eh4bQ=
=Q4/W
-----END PGP SIGNATURE-----
```

因为是签名，不会对内容进行加密，所以直接显示原文。如果想把原始文件分离出来，使用

```shell
gpg --output out.txt -d test.txt.gpg
```

分离的同时还顺便校验了签名。注意这里`test.txt.gpg`是上面生成的二进制**签名**文件，也可以用`test.txt.asc`

如果每次都要分离，就很麻烦，而且没有安装 GPG 的用户就无法使用源文件了。所以更好的方式是分离源文件和签名文件

```shell
gpg --detach-sign --armor test.txt
```

这样会生成一个单独的、文本类型的签名文件`test.txt.sig`

> 注意：如果有多个私钥，可以用`-u`参数选择

### 签名校验

收到文件和签名文件后，需要对签名进行校验

```shell
gpg --verify test.txt.sig test.txt
```

如果签名校验成功，会显示

```shell
gpg: 签名建立于 四  4/ 2 11:49:35 2020 CST
gpg:               使用 RSA 密钥 34B368EFC6FA8941B4A096CFFA4C884FBE8164D7
gpg: 完好的签名，来自于 “creedowl <creedowl@gmail.com>” [绝对]
```

### 导入导出

当你需要与他人交换数据时，需要导出自己的公钥，同时导入他人的公钥；而如果你更换电脑，就需要导出自己的私钥。这就涉及到了密钥的导入导出

```shell
gpg -a --output public_key.txt --export naive
```

上述命令用于导出公钥，相关参数在上文都有介绍。接着你就可以把`public_key.txt`发布给他人

```shell
gpg -a --output private_key.txt --export-secret-keys naive
```

上述命令用于导出私钥，一定要保管好，建议在脱机情况下进行操作

```shell
gpg --improt [密钥文件]
```

上述命令用于导入密钥，不区分公钥私钥

### 查看所有密钥

```shell
gpg -k
```

### 生成吊销证书

当你的私钥泄露或者更换时，需要吊销失效的密钥，这时就需要使用吊销证书

```shell
gpg --gen-revoke naive
```

接着选择吊销原因，就能生成吊销证书

```text
sec  rsa2048/4C26D2E451FFDAF1 2020-04-02 naive (too young too simple) <naive@test.com>

要为这个密钥创建一个吊销证书吗？(y/N) y
请选择吊销的原因：
  0 = 未指定原因
  1 = 密钥已泄漏
  2 = 密钥被替换
  3 = 密钥不再使用
  Q = 取消
（也许您会想要在这里选择 1）
您的决定是什么？ 1
请输入描述（可选）；以空白行结束：
>
吊销原因：密钥已泄漏
（未给定描述）
这样可以吗？ (y/N) y
已强行使用 ASCII 字符封装过的输出。
-----BEGIN PGP PUBLIC KEY BLOCK-----
Comment: This is a revocation certificate

****
-----END PGP PUBLIC KEY BLOCK-----
已创建吊销证书。

请把这个文件转移到一个您可以藏起来的介质上；如果坏人获取到了这
份证书的话，那么他就能使用它并让您的密钥无法继续使用。把此证书
打印出来再存放到安全的地方也是很好的方法，以免您的保存媒体变得
不可读。但是千万小心：您机器上的打印系统可能会在打印过程中储存
这些数据，并使得其他人看到！
```

只需要保存中间的吊销证书，当密钥泄露时将其发送到公钥服务器，就可以吊销旧密钥

### 编辑密钥

有时候需要对密钥进行编辑，如修改信任度，生成子密钥等，使用以下命令

```shell
gpg --edit-key naive
```

接着就会进入交互模式等待命令，输入`help`可以查看所有操作

```text
gpg (GnuPG/MacGPG2) 2.2.17; Copyright (C) 2019 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

私钥可用。

sec  rsa2048/4C26D2E451FFDAF1
     创建于：2020-04-02  有效至：永不       可用于：SC
     信任度：绝对        有效性：绝对
ssb  rsa2048/A86AB46BDC677F8F
     创建于：2020-04-02  有效至：永不       可用于：E
[ 绝对 ] (1). naive (too young too simple) <naive@test.com>

gpg>
```

## 使用 GPG 签名 Git Commit

这就是我使用 GPG 的原因，下面开始配置，首先配置 git

```shell
# 设置全局默认密钥
git config --global user.signingkey <密钥ID>

# 设置强制使用GPG签名
git config --global commit.gpgsign true
```

之后在提交时，只需要带上`-S`参数，就能对提交进行签名

```shell
git commit -S -m "xxx"
```

查看 git 日志时，带上`--show-signature`参数就能看到签名信息

```shell
git log --show-signature
```

最后一步，需要把你的公钥提交到 github，否则它怎么知道这个签名是你的呢？将之前生成的文本类型的公钥`public_key.txt`提交到 [github](https://github.com/settings/keys)，之后你的提交在 github 上就有像开头那样的绿色小勾了

## FAQ

### 解决git提交时出现`error: gpg failed to sign the data`

以下是原始解决方案，虽然能解决问题，但体验不好，在 IDE 中提交时难以输入密码，建议 MacOS 用户采用上面说的 `GPG Suite`进行操作，它可以接管密码输入操作，弹出一个独立窗口用于输入，还能将密码添加到 iCloud 钥匙串中，避免重复输入密码。如果你已经在使用`GPG Suite`，但仍然遇到了这个问题，可以尝试重装`GPG Suite`

1. 导出密钥（私钥）**重要**
2. 卸载 GPG

    ```shell
    brew uninstall gnupg gnutls
    
    brew cask uninstall gpg-suite
    ```

3. 删除 gpg 文件夹，**请确认已经导出密钥**

    ```shell
    rm -rf ~/.gnupg
    ```

4. 重新安装`GPG Suite`，**不需要手动安装 GPG**

    ```shell
    brew cask install gpg-suite
    ```

5. 导入密钥

这时 `GPG Suite`应该已经可以正常工作了

---

*原始方案*

首先测试`gpg`工作是否正常

```shell
echo "test" | gpg --clearsign
```

如果出现下面的错误

```text
-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA256

test
gpg: 签名时失败： Inappropriate ioctl for device
gpg: [stdin]: clear-sign failed: Inappropriate ioctl for device
```

说明`gpg`无法打开输入密码的窗口。在生成`gpg key`时，如果指定了密码，在签名时就需要输入。而这时`gpg`不知道从哪里读取输入，所以可以用以下的命令指定`tty`

```shell
export GPG_TTY=$(tty) 
```

这样就可以正常弹出输入密码的窗口。

---

但我不喜欢这个全屏窗口，主要是在mac上`gpg`语言为中文时，这个窗口会乱码，所以用另一个方法解决。编辑`~/.gnupg/gpg.conf`，添加以下内容

```text
use-agent 
pinentry-mode loopback
```

再编辑`~/.gnupg/gpg-agent.conf`，添加以下内容

```text
allow-loopback-pinentry
```

最后输入`echo RELOADAGENT | gpg-connect-agent`重启`agent`就生效了，这样可以直接在终端输入密码

> [source](https://d.sb/2016/11/gpg-inappropriate-ioctl-for-device-errors)