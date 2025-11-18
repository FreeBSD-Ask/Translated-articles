# 修复 FreeBSD 上的依赖

- [Fix Broken Dependency on FreeBSD](https://vermaden.wordpress.com/2019/03/21/fix-broken-dependency-on-freebsd/)
- 作者：𝚟𝚎𝚛𝚖𝚊𝚍𝚎𝚗
- 2019/03

不知道你怎么样，但我经常更新我的包……而且数量多达一千多个。

```
% pkg info | wc -l
    1051
```

……不过这没什么，大多数都是我使用软件的依赖包。

例如，我需要 Openbox 和 X11，但为了使用它们，我需要 300 多个库和协议依赖，这很正常，这就是它的工作方式……不过有时升级之后，一两个应用程序会因为缺失依赖而无法启动。我会说这种情况大约每二十到三十次更新发生一次（1/20 – 1/30），非常罕见，而且即便发生，也很容易解决。我在 Linux 系统上也遇到过很多次，所以这不仅仅是 FreeBSD 的问题，这就是开源桌面/笔记本生态的运作方式 🙂。

今天的受害者是 Chromium。我平时一般使用 Firefox，但有时当某个页面在 Firefox 上表现异常时，我会用 Chromium 验证这个行为。我也把 Chromium 用作本地 ***.htm**/***.html**/***.chm** 文件的文件打开器（或者说文件浏览器）。但这次它无法启动，于是我去命令行检查出了什么问题。

```sh
% chrome
Shared object "libx264.so.155" not found, required by "libavcodec.so.58"
```

……缺失的依赖是库 **libx264.so.155**。

## 鲁莽的符号链接

这种方法被认为是危险的，或者说是一种快速且粗糙的修复方式——它本身也可能引入其他问题——但在很多情况下，它可以暂时解决问题。

……而这正是它的作用——一个临时修复，直到 **ffmpeg** 包完成重建——这比执行 **pkg upgrade** 命令需要更长时间，但当我现在需要 Chromium 时，就是“现在”，而不是等 **ffmpeg** 包重建完成之后再说。这个问题的根源在于 FreeBSD 项目缺乏提供 **lame** 包的勇气。OpenBSD 的开发者没有这个问题，但 FreeBSD 的开发者有。因此，要在 **ffmpeg** 中支持 MP3，你必须先手动编译 **lame** 包，然后在 **ffmpeg** 中选择该选项，再将其重新构建为包……而且每次执行 **pkg upgrade** 命令都要重复这一过程……至少可以说这是非常麻烦的。

这就是我使用 **[pkg-recompile.sh](https://github.com/vermaden/scripts/blob/master/pkg-recompile.sh)** 脚本的原因——为了避免每次更新包（大约每周两次）都手动重复上述操作。如果可以称之为“工作流程”，它大致是这样的：

```sh
# pkg upgrade
# pkg-recompile.sh build
```

那我们来验证一下，看看 Chromium 是否还有其他缺失的依赖。

```sh
% which chrome
/usr/local/bin/chrome

% ldd /usr/local/bin/chrome
ldd: /usr/local/bin/chrome: not a dynamic executable
```

所以 **/usr/local/bin/chrome** 只是一个包装脚本，让我们看看它的内容。

```sh
% cat /usr/local/bin/chrome
#!/bin/sh

SYSCTL=kern.ipc.shm_allow_removed
if [ "`/sbin/sysctl -n $SYSCTL`" = 0 ] ; then
        cat << EOMSG
For correct operation, shared memory support has to be enabled
in Chromium by performing the following command as root :

sysctl $SYSCTL=1

To preserve this setting across reboots, append the following
to /etc/sysctl.conf :

$SYSCTL=1
EOMSG
        exit 1
fi
ulimit -c 0
exec /usr/local/share/chromium/chrome ${1+"$@"}
```

所以我们的真正二进制文件是 **/usr/local/share/chromium/chrome**，接下来用 **ldd(8)** 检查它。

```sh
% ldd /usr/local/share/chromium/chrome
/usr/local/share/chromium/chrome:
        libthr.so.3 => /lib/libthr.so.3 (0x809b78000)
        libX11.so.6 => /usr/local/lib/libX11.so.6 (0x809da0000)
        libX11-xcb.so.1 => /usr/local/lib/libX11-xcb.so.1 (0x80a0df000)
        libxcb.so.1 => /usr/local/lib/libxcb.so.1 (0x80a2e0000)
        libXcomposite.so.1 => /usr/local/lib/libXcomposite.so.1 (0x80a506000)
        libXcursor.so.1 => /usr/local/lib/libXcursor.so.1 (0x80a708000)
        libXdamage.so.1 => /usr/local/lib/libXdamage.so.1 (0x80a913000)
        libXext.so.6 => /usr/local/lib/libXext.so.6 (0x80ab15000)
        libXfixes.so.3 => /usr/local/lib/libXfixes.so.3 (0x80ad26000)
        libXi.so.6 => /usr/local/lib/libXi.so.6 (0x80af2b000)
        libXrender.so.1 => /usr/local/lib/libXrender.so.1 (0x80b139000)
        libXtst.so.6 => /usr/local/lib/libXtst.so.6 (0x80b342000)
        libgmodule-2.0.so.0 => /usr/local/lib/libgmodule-2.0.so.0 (0x80b547000)
        libglib-2.0.so.0 => /usr/local/lib/libglib-2.0.so.0 (0x80b74a000)
        libgobject-2.0.so.0 => /usr/local/lib/libgobject-2.0.so.0 (0x80ba61000)
        libgthread-2.0.so.0 => /usr/local/lib/libgthread-2.0.so.0 (0x80bcab000)
        libintl.so.8 => /usr/local/lib/libintl.so.8 (0x80beac000)
        libnss3.so => /usr/local/lib/nss/libnss3.so (0x80c0b7000)
        libsmime3.so => /usr/local/lib/nss/libsmime3.so (0x80c3e3000)
        libnssutil3.so => /usr/local/lib/nss/libnssutil3.so (0x80c60d000)
        libplds4.so => /usr/local/lib/libplds4.so (0x80c83d000)
        libplc4.so => /usr/local/lib/libplc4.so (0x80ca40000)
        libnspr4.so => /usr/local/lib/libnspr4.so (0x80cc44000)
        libdl.so.1 => /usr/lib/libdl.so.1 (0x80ce83000)
        libcups.so.2 => /usr/local/lib/libcups.so.2 (0x80d084000)
        libxml2.so.2 => /usr/local/lib/libxml2.so.2 (0x80d315000)
        libfontconfig.so.1 => /usr/local/lib/libfontconfig.so.1 (0x80d6a8000)
        libdbus-1.so.3 => /usr/local/lib/libdbus-1.so.3 (0x80d8ef000)
        libexecinfo.so.1 => /usr/lib/libexecinfo.so.1 (0x80db40000)
        libkvm.so.7 => /lib/libkvm.so.7 (0x80dd43000)
        libutil.so.9 => /lib/libutil.so.9 (0x80df51000)
        libXss.so.1 => /usr/local/lib/libXss.so.1 (0x80e165000)
        libwebpdemux.so.2 => /usr/local/lib/libwebpdemux.so.2 (0x80e367000)
        libwebpmux.so.3 => /usr/local/lib/libwebpmux.so.3 (0x80e56b000)
        libwebp.so.7 => /usr/local/lib/libwebp.so.7 (0x80e775000)
        libfreetype.so.6 => /usr/local/lib/libfreetype.so.6 (0x80ea05000)
        libjpeg.so.8 => /usr/local/lib/libjpeg.so.8 (0x80ecbb000)
        libexpat.so.1 => /usr/local/lib/libexpat.so.1 (0x80ef4e000)
        libharfbuzz.so.0 => /usr/local/lib/libharfbuzz.so.0 (0x80f179000)
        libdrm.so.2 => /usr/local/lib/libdrm.so.2 (0x80f458000)
        libXrandr.so.2 => /usr/local/lib/libXrandr.so.2 (0x80f66b000)
        libgio-2.0.so.0 => /usr/local/lib/libgio-2.0.so.0 (0x80f875000)
        libavcodec.so.58 => /usr/local/lib/libavcodec.so.58 (0x80fe00000)
        libavformat.so.58 => /usr/local/lib/libavformat.so.58 (0x811800000)
        libavutil.so.56 => /usr/local/lib/libavutil.so.56 (0x811c52000)
        libopenh264.so.4 => /usr/local/lib/libopenh264.so.4 (0x811eca000)
        libasound.so.2 => /usr/local/lib/libasound.so.2 (0x8121da000)
        libsnappy.so.1 => /usr/local/lib/libsnappy.so.1 (0x8124de000)
        libopus.so.0 => /usr/local/lib/libopus.so.0 (0x8126e6000)
        libpangocairo-1.0.so.0 => /usr/local/lib/libpangocairo-1.0.so.0 (0x812956000)
        libpango-1.0.so.0 => /usr/local/lib/libpango-1.0.so.0 (0x812b63000)
        libcairo.so.2 => /usr/local/lib/libcairo.so.2 (0x812db1000)
        libGL.so.1 => /usr/local/lib/libGL.so.1 (0x8130d8000)
        libpci.so.3 => /usr/local/lib/libpci.so.3 (0x813366000)
        libatk-1.0.so.0 => /usr/local/lib/libatk-1.0.so.0 (0x813571000)
        libatk-bridge-2.0.so.0 => /usr/local/lib/libatk-bridge-2.0.so.0 (0x81379c000)
        libatspi.so.0 => /usr/local/lib/libatspi.so.0 (0x8139cc000)
        libFLAC.so.8 => /usr/local/lib/libFLAC.so.8 (0x813bfd000)
        libgtk-3.so.0 => /usr/local/lib/libgtk-3.so.0 (0x814000000)
        libgdk-3.so.0 => /usr/local/lib/libgdk-3.so.0 (0x8148b9000)
        libcairo-gobject.so.2 => /usr/local/lib/libcairo-gobject.so.2 (0x814bb0000)
        libgdk_pixbuf-2.0.so.0 => /usr/local/lib/libgdk_pixbuf-2.0.so.0 (0x814db8000)
        libxslt.so.1 => /usr/local/lib/libxslt.so.1 (0x814fdb000)
        libz.so.6 => /lib/libz.so.6 (0x815218000)
        liblzma.so.5 => /usr/lib/liblzma.so.5 (0x815430000)
        libm.so.5 => /lib/libm.so.5 (0x815659000)
        librt.so.1 => /usr/lib/librt.so.1 (0x815886000)
        libc++.so.1 => /usr/lib/libc++.so.1 (0x815a8c000)
        libcxxrt.so.1 => /lib/libcxxrt.so.1 (0x815d5a000)
        libc.so.7 => /lib/libc.so.7 (0x800823000)
        libXau.so.6 => /usr/local/lib/libXau.so.6 (0x815f79000)
        libXdmcp.so.6 => /usr/local/lib/libXdmcp.so.6 (0x81617c000)
        libiconv.so.2 => /usr/local/lib/libiconv.so.2 (0x816381000)
        libpcre.so.1 => /usr/local/lib/libpcre.so.1 (0x81667c000)
        libffi.so.6 => /usr/local/lib/libffi.so.6 (0x81691a000)
        libgnutls.so.30 => /usr/local/lib/libgnutls.so.30 (0x816b21000)
        libavahi-common.so.3 => /usr/local/lib/libavahi-common.so.3 (0x816ed4000)
        libavahi-client.so.3 => /usr/local/lib/libavahi-client.so.3 (0x8170e0000)
        libcrypt.so.5 => /lib/libcrypt.so.5 (0x8172ef000)
        libelf.so.2 => /lib/libelf.so.2 (0x81750e000)
        libgcc_s.so.1 => /lib/libgcc_s.so.1 (0x817725000)
        libbz2.so.4 => /usr/lib/libbz2.so.4 (0x817934000)
        libgraphite2.so.3 => /usr/local/lib/libgraphite2.so.3 (0x817b48000)
        libswresample.so.3 => /usr/local/lib/libswresample.so.3 (0x817d71000)
        libvpx.so.6 => /usr/local/lib/libvpx.so.6 (0x818000000)
        libdav1d.so.1 => /usr/local/lib/libdav1d.so.1 (0x818411000)
        libmp3lame.so.0 => /usr/local/lib/libmp3lame.so.0 (0x818732000)
        libtheoraenc.so.1 => /usr/local/lib/libtheoraenc.so.1 (0x8189b3000)
        libtheoradec.so.1 => /usr/local/lib/libtheoradec.so.1 (0x818be2000)
        libvorbis.so.0 => /usr/local/lib/libvorbis.so.0 (0x818df3000)
        libvorbisenc.so.2 => /usr/local/lib/libvorbisenc.so.2 (0x819024000)
        libx264.so.155 => not found (0)
        libx265.so.170 => /usr/local/lib/libx265.so.170 (0x819400000)
        libxvidcore.so.4 => /usr/local/lib/libxvidcore.so.4 (0x819b4b000)
        libva.so.2 => /usr/local/lib/libva.so.2 (0x819e70000)
        libgmp.so.10 => /usr/local/lib/libgmp.so.10 (0x81a096000)
        libva-drm.so.2 => /usr/local/lib/libva-drm.so.2 (0x81a316000)
        libva-x11.so.2 => /usr/local/lib/libva-x11.so.2 (0x81a518000)
        libvdpau.so.1 => /usr/local/lib/libvdpau.so.1 (0x81a71d000)
        libpangoft2-1.0.so.0 => /usr/local/lib/libpangoft2-1.0.so.0 (0x81a920000)
        libfribidi.so.0 => /usr/local/lib/libfribidi.so.0 (0x81ab36000)
        libpixman-1.so.0 => /usr/local/lib/libpixman-1.so.0 (0x81ad4c000)
        libEGL.so.1 => /usr/local/lib/libEGL.so.1 (0x81b016000)
        libpng16.so.16 => /usr/local/lib/libpng16.so.16 (0x81b24e000)
        libxcb-shm.so.0 => /usr/local/lib/libxcb-shm.so.0 (0x81b489000)
        libxcb-render.so.0 => /usr/local/lib/libxcb-render.so.0 (0x81b68b000)
        libxcb-dri3.so.0 => /usr/local/lib/libxcb-dri3.so.0 (0x81b898000)
        libxcb-xfixes.so.0 => /usr/local/lib/libxcb-xfixes.so.0 (0x81ba9b000)
        libxcb-present.so.0 => /usr/local/lib/libxcb-present.so.0 (0x81bca2000)
        libxcb-sync.so.1 => /usr/local/lib/libxcb-sync.so.1 (0x81bea4000)
        libxshmfence.so.1 => /usr/local/lib/libxshmfence.so.1 (0x81c0aa000)
        libglapi.so.0 => /usr/local/lib/libglapi.so.0 (0x81c2ab000)
        libxcb-glx.so.0 => /usr/local/lib/libxcb-glx.so.0 (0x81c505000)
        libxcb-dri2.so.0 => /usr/local/lib/libxcb-dri2.so.0 (0x81c71e000)
        libXxf86vm.so.1 => /usr/local/lib/libXxf86vm.so.1 (0x81c922000)
        libogg.so.0 => /usr/local/lib/libogg.so.0 (0x81cb26000)
        libXinerama.so.1 => /usr/local/lib/libXinerama.so.1 (0x81cd2c000)
        libxkbcommon.so.0 => /usr/local/lib/libxkbcommon.so.0 (0x81cf2e000)
        libwayland-cursor.so.0 => /usr/local/lib/libwayland-cursor.so.0 (0x81d16b000)
        libwayland-egl.so.1 => /usr/local/lib/libwayland-egl.so.1 (0x81d372000)
        libwayland-client.so.0 => /usr/local/lib/libwayland-client.so.0 (0x81d573000)
        libepoxy.so.0 => /usr/local/lib/libepoxy.so.0 (0x81d782000)
        libp11-kit.so.0 => /usr/local/lib/libp11-kit.so.0 (0x81da91000)
        libtasn1.so.6 => /usr/local/lib/libtasn1.so.6 (0x81ddb2000)
        libnettle.so.6 => /usr/local/lib/libnettle.so.6 (0x81dfc7000)
        libhogweed.so.4 => /usr/local/lib/libhogweed.so.4 (0x81e1ff000)
        libidn2.so.0 => /usr/local/lib/libidn2.so.0 (0x81e435000)
        libunistring.so.2 => /usr/local/lib/libunistring.so.2 (0x81e653000)
        libgbm.so.1 => /usr/local/lib/libgbm.so.1 (0x81ea07000)
        libwayland-server.so.0 => /usr/local/lib/libwayland-server.so.0 (0x81ec15000)
        libepoll-shim.so.0 => /usr/local/lib/libepoll-shim.so.0 (0x81ee28000)
```

依赖很多，我们直接用 **grep(1)** 筛选，示例如下。

```sh
% ldd /usr/local/share/chromium/chrome | grep found
        libx264.so.155 => not found (0)
```

只缺失了一个依赖——**libx264.so.155**。那我们就来修复它。

```sh
% cd /usr/local/lib
% ls -l libx264.so*
lrwxr-xr-x  1 root  wheel       14 2019.03.19 02:11 libx264.so -> libx264.so.157
-rwxr-xr-x  1 root  wheel  2090944 2019.03.19 02:11 libx264.so.157
```

这里有一个稍新的版本 **libx264.so.157**，所以我们用它创建一个符号链接，命名为缺失的 **libx264.so.155**。

```sh
# pwd
/usr/local/lib
# ln -s libx264.so libx264.so.155
# ls -l libx264.so*
lrwxr-xr-x  1 root  wheel       14 2019.03.19 02:11 libx264.so -> libx264.so.157
lrwxr-xr-x  1 root  wheel       10 2019.03.21 15:26 libx264.so.155 -> libx264.so
-rwxr-xr-x  1 root  wheel  2090944 2019.03.19 02:11 libx264.so.157
```

现在应该能正常启动 Chromium 了。

```sh
% ldd /usr/local/share/chromium/chrome | grep found
%
```

没有出现任何 **not found** 的结果。

那我们就用 **chrome** 命令启动 Chromium。

```
% chrome
```

一如既往，一切正常 🙂。

可以通过下面这张简单的截图直观地看到整个过程。

![vermaden\_2019-03-21\_15-47-40.png](https://vermaden.wordpress.com/wp-content/uploads/2019/03/vermaden_2019-03-21_15-47-40.png?w=960)

## 使用 `/etc/libmap.conf` 文件

与其创建符号链接——这种方法会全局生效——你也可以为二进制文件 **/usr/local/share/chromium/chrome** 单独创建一个合适的 **[libmap.conf](https://man.freebsd.org/libmap.conf)** 配置文件。

下面是仅针对 Chromium 浏览器的修复方法。

```sh
# cat /etc/libmap.conf

[/usr/local/share/chromium/chrome]
libx264.so.155 libx264.so
```

……与之等效、可全局生效的解决方案是使用符号链接，如下所示。

```sh
# cat /etc/libmap.conf

libx264.so.155 libx264.so
```
这种方法也更方便迁移或批量应用这些修改，而不需要复制符号链接。

## 修复 pkg(8) 数据库中的破损依赖

我在 **[Less Known pkg(8) Features](https://vermaden.wordpress.com/2019/01/17/less-known-pkg8-features/)** 一文中已经写过，但为了选项的完整性，这里值得再提一次。

曾经有一次，缺失的依赖——涉及易受攻击的 **www/libxul19** 包——折磨了我一段时间。

我甚至已经绝望到想用 **portmaster** 重新编译所有东西。

我一开始使用了 **portmaster --check-depends** 命令，但在提示是否修复时选择了‘**n**’，因为那会不必要地降级很多包。

```sh
# portmaster --check-depends
(...)
Checking dependencies: evince
graphics/evince has a missing dependency: www/libxul19
(...)

>>> Missing package dependencies were detected.
>>> Found 1 issue(s) in total with your package database.

The following packages will be installed:

        Downgrading perl: 5.14.2_3 -> 5.14.2_2
        Downgrading glib: 2.34.3 -> 2.28.8_5
        Downgrading gio-fam-backend: 2.34.3 -> 2.28.8_1
        Downgrading libffi: 3.0.12 -> 3.0.11
        Downgrading gobject-introspection: 1.34.2 -> 0.10.8_3
        Downgrading atk: 2.6.0 -> 2.0.1
        Downgrading gdk-pixbuf2: 2.26.5 -> 2.23.5_3
        Downgrading pango: 1.30.1 -> 1.28.4_1
        Downgrading gtk-update-icon-cache: 2.24.17 -> 2.24.6_1
        Downgrading dbus: 1.6.8 -> 1.4.14_4
        Downgrading gtk: 2.24.17 -> 2.24.6_2
        Downgrading dbus-glib: 0.100.1 -> 0.94
        Installing libxul: 1.9.2.28_1

The installation will require 66 MB more space

38 MB to be downloaded

>>> Try to fix the missing dependencies [y/N]: n
>>> Summary of actions performed:

www/libxul19 dependency failed to be fixed

>>> There are still missing dependencies.
>>> You are advised to try fixing them manually.

>>> Also make sure to check 'pkg updating' for known issues.
```

那我们来看 **pkg(8)** 显示的已安装包情况。

```sh
# pkg info | grep libxul
libxul-10.0.12                 Mozilla runtime package that can be used to bootstrap XUL+XPCOM apps

# pkg info -qoa | grep libxul
www/libxul
```

问题在于我们安装的是 **www/libxul** 而不是 **www/libxul19**，这也是为什么 **portmaster**（以及其他工具）会报错。

在引入 **pkg(8)** 之前，只需对整个 **/var/db/pkg** 目录及其“文件数据库”使用 **grep -r** 就很容易查到，但现在情况复杂得多，因为包数据库已经存储在 SQLite 数据库中。

使用 **pkg shell** 命令可以连接到该数据库。让我们看看能找到些什么。

```sh
# pkg shell
SQLite version 3.7.13 2012-06-11 02:05:22
Enter ".help" for instructions
Enter SQL statements terminated with a ";"
sqlite> .databases
seq  name             file
---  ---------------  ----------------------------------------------------------
0    main             /var/db/pkg/local.sqlite
sqlite> .tables
categories       licenses         pkg_directories  scripts
deps             mtree            pkg_groups       shlibs
directories      options          pkg_licenses     users
files            packages         pkg_shlibs
groups           pkg_categories   pkg_users
sqlite> .header on
sqlite> .mode column
sqlite> pragma table_info(deps);
cid         name        type        notnull     dflt_value  pk
----------  ----------  ----------  ----------  ----------  ----------
0           origin      TEXT        1                       1
1           name        TEXT        1                       0
2           version     TEXT        1                       0
3           package_id  INTEGER     0                       1
sqlite> .quit
```

所以现在我们知道，“**deps**” 表可能就是我们要找的 ;)。

由于 **pkg shell** 在 SQLite “浏览”方面相当有限，我将直接使用 **sqlite3** 命令。所谓有限，是指你不能直接输入 **pkg shell "select * from deps;"** 查询，而是需要先启动 **pkg shell**，然后才能输入查询语句。

```sh
# sqlite3 -column /var/db/pkg/local.sqlite "select * from deps;" | grep libxul
www/libxul19   libxul      1.9.2.28_1  104
```

第二列是 **name**，所以我们可以尝试用它来查询。

```sh
sqlite3 -header -column /var/db/pkg/local.sqlite "select * from deps where name='libxul';"
origin        name        version     package_id
------------  ----------  ----------  ----------
www/libxul19  libxul      1.9.2.28_1  104
```

现在我们已经定位到这个“有问题”的依赖条目，让我们稍微修改它，使其与实际已安装的包状态一致。

```sh
# sqlite3 /var/db/pkg/local.sqlite "update deps set origin='www/libxul' where name='libxul';"
# sqlite3 /var/db/pkg/local.sqlite "update deps set version='10.0.12' where name='libxul';"
```

当然，你也可以使用“官方”方式，通过 **pkg shell** 命令来操作。

```sh
# pkg shell
SQLite version 3.7.13 2012-06-11 02:05:22
Enter ".help" for instructions
Enter SQL statements terminated with a ";"
sqlite> update deps set origin='www/libxul' where name='libxul';
sqlite> update deps set version='10.0.12' where name='libxul';
sqlite> .header on
sqlite> .mode column
sqlite> select * from deps where name='libxul';
origin      name        version     package_id
----------  ----------  ----------  ----------
www/libxul  libxul      10.0.12     104
sqlite> .quit
```

现在 **portmaster** 已经不再抱怨缺失依赖，一切正常。

```sh
# portmaster --check-depends
(...)
Checking dependencies: zenity
Checking dependencies: zip
Checking dependencies: zsh
#
```

完美！问题解决了 😉

……但 **pkg(8)** 本身已经提供了工具来处理这个问题 🙂

它叫做 **pkg set**，在 **man pkg-set** 中最有用的两个选项是：

```
  -n oldname:newname, --change-name oldname:newname
       Change the package name of a given dependency from oldname to newname.
       将指定依赖项的包名从 `oldname` 修改为 `newname`。

(...)

  -o oldorigin:neworigin, --change-origin oldorigin:neworigin
       Change the port origin of a given dependency from oldorigin to neworigin.
       This corresponds to the port directory that the package originated from.
       Typically, this is only needed for upgrading a library or package that
       has MOVED or when the default version of a major port dependency changes.
       (DEPRECATED) Usually this will be explained in /usr/ports/UPDATING.
       Also see pkg-updating(8) and EXAMPLES.
       将指定依赖项的 Port 来源从 oldorigin 修改为 neworigin。
       这对应于该包最初来源的 Port 目录。通常，仅在升级已被 MOVED 的库或包，或者主要 Port 依赖的默认版本发生变化时才需要使用此功能。（已弃用）通常相关说明会在 /usr/ports/UPDATING 中给出。
       另请参见 pkg-updating(8) 和示例。
  ```


在我们的例子中，可以使用以下命令：`pkg set -o www/libxul19:www/libxul`

不过不确定它是否会以同样的方式解决问题，因为我同时还更新了数据库中的版本信息。

## 使用 bsdadminscripts2 包中的 pkg_libchk

还有另一种方法可以修复或检查此类问题——那就是使用 **bsdadminscripts2** 包中的 **pkg_libchk**。请注意，这里有两个名称中带有 **bsdadminscripts** 的冲突包，需要区分清楚。


```
# pkg search bsdadmin
bsdadminscripts-6.1.1_8        Collection of administration scripts
bsdadminscripts2-0.2.1         BSD Administration Scripts 2
```

……只要你安装了 **bsdadminscripts2**，就无法再安装 **bsdadminscripts**，因为它们存在冲突。我之前已经安装了 **bsdadminscripts2**，但想在系统中添加 **bsdadminscripts**。

```sh
# pkg install bsdadminscripts
Updating FreeBSD repository catalogue...
FreeBSD repository is up to date.
All repositories are up to date.
Checking integrity... done (1 conflicting)
  - bsdadminscripts-6.1.1_8 conflicts with bsdadminscripts2-0.2.1 on /usr/local/sbin/distviper
Checking integrity... done (0 conflicting)
The following 2 package(s) will be affected (of 0 checked):

Installed packages to be REMOVED:
        bsdadminscripts2-0.2.1

New packages to be INSTALLED:
        bsdadminscripts: 6.1.1_8

Number of packages to be removed: 1
Number of packages to be installed: 1

Proceed with this action? [y/N]: n
```

下面是 Port/包 **/usr/ports/ports-mgmt/bsdadminscripts2** 的描述。

```sh
# cat /usr/ports/ports-mgmt/bsdadminscripts2/pkg-descr
This is a collection of scripts around the use of ports and packages.

It allows you to: 
- check library dependencies without producing false positives (pkg_libchk)
- lets you manage the autoremove flag for leaf packages (pkg_trim)
- remove obsolete or damaged distfiles (distviper)
- manage build flags (buildflags.conf)
- auto-create pkg-plist files taking port options into account (makeplist)

WWW: https://github.com/lonkamikaze/bsda2
```

这个包正好有四个工具。

```sh
% pkg info -l bsdadminscripts2 | grep bin
        /usr/local/sbin/distviper
        /usr/local/sbin/makeplist
        /usr/local/sbin/pkg_libchk
        /usr/local/sbin/pkg_trim
```

如果不带任何参数调用，它会检查系统中安装的所有包。

```sh
# pkg_libchk
Jobs done:   35 of 1057
bhyve-firmware-1.0_1
bash-5.0.3
beadm-1.2.9_1
```
……所以如果只想对 Chromium 进行检查，需要指定 **chromium** 包，命令如下：

```
pkg_libchk chromium
```

**pkg_libchk** 可以根据哪个包提供了哪些文件来获取缺失的依赖，或者生成需要重建的包列表。

## 使用 Provides 数据库

你也可以使用 **pkg(8)** 的“provides”数据库。

```sh
% pkg provides lib/libx264.so
Name    : libx264-0.157.2945
Desc    : H.264/MPEG-4 AVC Video Encoding (Library)
Repo    : FreeBSD
Filename: /usr/local/lib/libx264.so.155
          /usr/local/lib/libx264.so
```


想了解如何为 **pkg(8)** 命令设置“provides”数据库，请查阅文章 [**Less Known pkg(8) Features**](https://vermaden.wordpress.com/2019/01/17/less-known-pkg8-features/)。

## 更新 1 – 重新整理整篇文章

古罗马哲学家塞涅卡曾说过——**“在教别人的同时，我们也在学习。”**——这句话非常正确——尤其是对本文而言。在我将文章发布到各处后，大家提醒我，仅仅创建符号链接并不是最好的方法。对此我接受指正，并增加了额外章节和方法，介绍如何在 FreeBSD（或 Linux/Illumos）系统上修复破损依赖。
