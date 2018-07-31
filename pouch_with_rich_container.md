# 富容器

富容器是一种容器化应用程序时非常有用的容器模式。此模式可帮助技术人员轻松打包大型应用。它提供了有效的方法在单个容器中配置目标应用程序以及更多的基础软件或系统服务。容器中的应用程序可以像在虚拟机或物理机中一样顺利运行。这是一种更通用的以应用程序为中心的模式。这种模式对开发人员和运营人员都没有任何侵略性。特别是对于运营人员而言，他们可以像往常一样使用他们可能需要的所有必要工具或服务流程来维护容器中的应用程序。

富容器模式不是PouchContainer提供的默认模式。这是PouchContainer为扩展用户的容器体验而提供的附加模式。用户可以通过关闭富容器标志来使用普通容器。

总之，富容器可以帮助企业实现以下两个目标：

* 与传统操作系统兼容;
* 仍然利用镜像的优势来加快应用程序的交付。

## 基本情况

容器技术和编排平台现在变得非常流行。它们都为应用程序提供了更好的环境。尽管如此，我们不得不说容器化是企业采用容器相关技术的第一步，例如容器，编排，服务网等。将传统应用程序转移到容器中是一个非常实际的问题。虽然一些简单的应用程序能很容易地部署在容器上，但更传统和复杂的企业应用程序可能不那么容易。这些传统应用程序通常与底层基础架构耦合，例如机器架构、旧内核、甚至某些不再维护的软件。当然，“强耦合”不是每个人都希望的，它是企业数字化转型的初始产物。因此，所有行业都在寻求一种可行的方法来解决这个问题。docker提供的方式是一种，但不是最好的。在过去的7年里，阿里巴巴也遇到了同样的问题。幸运的是，富容器模式是一种更好的处理方式。

开发人员有自己的编程风格。 他们的工作是开发有用的应用程序，而不是设计绝对解耦的应用程序，因此他们通常利用工具或系统服务来实现开发。当容纳这些应用程序时，如果仅在容器中为一个应用程序设置一个进程，则相当薄弱。 富容器模式找出了使用户在容器中配置进程的内部启动顺序的方法，包括应用程序和系统服务。

运营人员负有保护应用程序正常运行的神圣职责。为了在应用程序中运行业务，技术必须充分尊重运营人员传统。在线调试和解决问题时，环境变化不是一个好消息。富容器模式可以确保富容器中的环境与传统虚拟机或物理机器中的环境完全相同。 如果操作员需要一些系统工具，它们仍然位于原来的位置。 如果某些前后hooks应该生效，只需在启动富容器时设置它们即可。 如果内部发生了一些问题，富容器启动的系统服务可以像自我修复一样修复它们。

## 架构

富容器模式与运营团队的传统操作方式兼容。 以下架构图显示了如何实现此目的：

![pouch_with_rich_container](https://i.loli.net/2018/07/30/5b5efc58d7180.png)

更详细地说，富容器承诺与oci兼容镜像兼容。 在运行富容器时，pouchd会将镜像文件系统作为富容器本身的rootfs。在内部容器运行时，除了内部应用程序和系统服务之外，还有一些hooks，如prestart hooks和poststop hooks。前者负责在systemd和相关进程运行之前准备或初始化环境，后者负责容器停止后的清理工作。

## 开始

用户可以很容易地在PouchContainer中启动富容器模式。比如我们需要通过PouchContainer在富容器模式下运行普通镜像，我们只需要添加两个标志： `--rich`,`--rich-mode`和`--initscript`. 以下是这两个标志的更多描述：

* `--rich`: 标识是否开启富容器模式。这个标志是布尔型，默认值为false.
* `--rich-mode`: 选择初始化容器的方式，支持currently systemd, /sbin/init和dumb-init，默认是dumb-init.
* `--initscript`: 标识在容器中执行的初始脚本。 该脚本将在入口点或命令之前执行。 有时，它被称为prestart hook。 在这个prestart hooks 中可以做很多工作，例如环境检查，环境准备，网络路由准备，各种代理设置，安全设置等。如果pouchd无法在容器的文件系统中找到此initscript，则该脚本可能会失败并且用户会收到相关的错误消息。该文件系统由从镜像构造的rootfs和实际位于容器外部的潜在安装卷提供。 如果initscript正常工作，容器进程的控制将由进程pid1接管，主要是 `/sbin/init` 或者 `dumbinit`.

事实上，PouchContainer团队计划添加另一个标志 `--initcmd` 让用户输入prestart hooks 。 实际上它是 `--initscript` 的简化版本。 同时它带来了比 `--initscript` 更多的便利。 `--initcmd` 可以根据用户的意愿设置任何命令，并且不需要事先将其放置在镜像中。我们可以说该命令与镜像分离。 但是对于 `--initscript` ，脚本文件必须首先位于镜像中，这是某种耦合。

如果用户指定 `--rich` 标志并且未提供 `--initscript` 标志，则仍将启用富容器模式，但不会执行initscript。 如果命令行中出现 `--initscript` 但是没有 `--rich` 标志，PouchContainer命令行或pouchd将返回错误，以表明 `--initscipt` 只能与 `--rich` 标志一起使用。

如果容器正在以 `--rich` 模式运行，那么每次启动或重新启动此容器都会触发相应的initscipt（如果有）。

### 使用 dumb-init

以下是使用dumb-init初始化容器的简单示例：

1. 如下安装dumb-init：

```shell
# wget -O /usr/bin/dumb-init https://github.com/Yelp/dumb-init/releases/download/v1.2.1/dumb-init_1.2.1_amd64
# chmod +x /usr/bin/dumb-init
```

2. 以富容器模式运行容器:

```shell
#pouch run -d --rich --rich-mode dumb-init registry.hub.docker.com/library/busybox:latest sleep 10000
f76ac1e49e9407caf5ad33c8988b44ff3690c12aa98f7faf690545b16f2a5cbd

#pouch exec f76ac1e49e9407caf5ad33c8988b44ff3690c12aa98f7faf690545b16f2a5cbd ps -ef
PID   USER     TIME  COMMAND
1 root      0:00 /usr/bin/dumb-init -- sleep 10000
7 root      0:00 sleep 10000
8 root      0:00 ps -ef
```

### 使用 systemd 或者 sbin-init

为了使用systemd或/sbin/init初始化容器，请确保将它们安装在镜像中。如下所示，centos镜像兼具二者。 此外，在这种情况下需要参数`--privileged`。systemd和sbin-init的示例如下：

```
#cat /tmp/1.sh
#! /bin/sh
echo $(cat) >/tmp/xxx

#pouch run -d -v /tmp:/tmp --privileged --rich --rich-mode systemd --initscript /tmp/1.sh registry.hub.docker.com/library/centos:latest /usr/bin/sleep 10000
3054125e44443fd5ee9190ee49bbca0a842724f5305cb05df49f84fd7c901d63

#pouch exec 3054125e44443fd5ee9190ee49bbca0a842724f5305cb05df49f84fd7c901d63 ps aux
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root          1  7.4  0.0  42968  3264 ?        Ss   05:29   0:00 /usr/lib/systemd/systemd
root         17  0.0  0.0  10752   756 ?        Ss   05:29   0:00 /usr/lib/systemd/systemd-readahead collect
root         18  3.2  0.0  32740  2908 ?        Ss   05:29   0:00 /usr/lib/systemd/systemd-journald
root         34  0.0  0.0  22084  1456 ?        Ss   05:29   0:00 /usr/lib/systemd/systemd-logind
root         36  0.0  0.0   7724   608 ?        Ss   05:29   0:00 /usr/bin/sleep 10000
dbus         37  0.0  0.0  24288  1604 ?        Ss   05:29   0:00 /bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation
root         45  0.0  0.0  47452  1676 ?        Rs   05:29   0:00 ps aux

#cat /tmp/xxx
{"ociVersion":"1.0.0","id":"3054125e44443fd5ee9190ee49bbca0a842724f5305cb05df49f84fd7c901d63","status":"","pid":125745,"bundle":"/var/lib/pouch/containerd/state/io.containerd.runtime.v1.linux/default/3054125e44443fd5ee9190ee49bbca0a842724f5305cb05df49f84fd7c901d63"}

#pouch run -d -v /tmp:/tmp --privileged --rich --rich-mode sbin-init --initscript /tmp/1.sh registry.hub.docker.com/library/centos:latest /usr/bin/sleep 10000
c5b5eef81749ce00fb68a59ee623777bfecc8e07c617c0601cc56e4ae8b1e69f

#pouch exec c5b5eef81749ce00fb68a59ee623777bfecc8e07c617c0601cc56e4ae8b1e69f ps aux
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root          1  7.4  0.0  42968  3260 ?        Ss   05:30   0:00 /sbin/init
root         17  0.0  0.0  10752   752 ?        Ss   05:30   0:00 /usr/lib/systemd/systemd-readahead collect
root         20  3.2  0.0  32740  2952 ?        Ss   05:30   0:00 /usr/lib/systemd/systemd-journald
root         34  0.0  0.0  22084  1452 ?        Ss   05:30   0:00 /usr/lib/systemd/systemd-logind
root         35  0.0  0.0   7724   612 ?        Ss   05:30   0:00 /usr/bin/sleep 10000
dbus         36  0.0  0.0  24288  1608 ?        Ss   05:30   0:00 /bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation
root         45  0.0  0.0  47452  1676 ?        Rs   05:30   0:00 ps aux

#cat /tmp/xxx
{"ociVersion":"1.0.0","id":"c5b5eef81749ce00fb68a59ee623777bfecc8e07c617c0601cc56e4ae8b1e69f","status":"","pid":127183,"bundle":"/var/lib/pouch/containerd/state/io.containerd.runtime.v1.linux/default/c5b5eef81749ce00fb68a59ee623777bfecc8e07c617c0601cc56e4ae8b1e69f"}
```

## 底层实现

在学习底层实现之前，我们先简要回顾一下systemd，entrypoint和cmd。另外，prestart hooks 由runC执行。

### systemd, entrypoint 和 cmd

待更新

### initscript 和 runC

`initscript` 待更新。

`runc` 是一个命令行界面工具，用于根据OCI规范生成和运行容器。
