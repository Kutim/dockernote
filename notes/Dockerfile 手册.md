Docker 可以通过读取  **Dockerfile**  中的指令来自动构建镜像，使用 **docker build** 来自动构建。

以下说明Dockerfile中常用到的命令：

## 1. Usage

​	**docker build** 命令使用 **Dockerfile** 和 *context* 来构建镜像。构建的 *context* 是 **PATH** 或 **URL** 指定的那些文件。**PATH** 是本地文件系统指定的目录，**URL** 是Git的仓库。



​	*context* 是递归处理的。因此， **PATH** 包含子目录；**URL** 包含源和子模块。最简单的就是使用当前目录作为 *context*。

```dockerfile
$ docker build .
Sending build context to Docker daemon  6.51 MB
...
```

​	通过 **Docker daemon** ，而不是 **CLI** 来构建。构建时将会把整个 *context* 递归的发送给 **daemon**。通常以一个空目录作为 *context*，然后添加 **Dockfile** 以及生成 **Dockerfile** 需要的文件。

> ⚠️ 警告：不要使用根目录（/）作为 **PATH**，这样会将整个磁盘的文件发送给 **Docker daemon**

​	**Dockerfile** 使用指令来说明使用 *context* 中的文件，例如， **COPY** 指令。为了加快构建性能，使用 **.dockerignore** 文件来排除不需要的文件和目录。

​	一般我们将 **Dockerfile** 放在 *context* 的根下面，我们也可以使用 **-f**  标志来指定 **Dockerfile** 文件的位置。	

```docker
$ docker build -f /path/to/a/Dockerfile .
```

​	使用 **-t** 来指定构建成功后镜像的 仓库 和标签

```docker
$ docker build -t shykes/myapp .
```

​	也可以使用多个 **-t** 来指定多个仓库

```docker
$ docker build -t shykes/myapp:1.0.2 -t shykes/myapp:latest .
```

​	在 **Docker daemon** 执行 **Dockerfile** 中的指令之前，会对 **Dockerfile** 进行验证

```docker
$ docker build -t test/myapp .
Sending build context to Docker daemon 2.048 kB
Error response from daemon: Unknown instruction: RUNCMD
```

​	**Docker daemon** 逐条执行 **Dockerfile** 中的指令，在生成最终的镜像ID 之前，将必要的指令执行的结果生成新的镜像。**Docker daemon** 会自动清理已经发送的 *context*。

​	注意每一条指令都是独立的，会引起一个新镜像的创建，所以 **RUN cd /tmp** 不会影响下一条指令。

​	如果可以， Docker 会重用中间已缓存的镜像来加速 **docker build** 构建效率。我们可以从控制台的 **Using cache** 消息看到使用了缓存。

```docker
$ docker build -t svendowideit/ambassador .
Sending build context to Docker daemon 15.36 kB
Step 1/4 : FROM alpine:3.2
 ---> 31f630c65071
Step 2/4 : MAINTAINER SvenDowideit@home.org.au
 ---> Using cache
 ---> 2a1c91448f5f
Step 3/4 : RUN apk update &&      apk add socat &&        rm -r /var/cache/
 ---> Using cache
 ---> 21ed6e7fbb73
Step 4/4 : CMD env | grep _TCP= | (sed 's/.*_PORT_\([0-9]*\)_TCP=tcp:\/\/\(.*\):\(.*\)/socat -t 100000000 TCP4-LISTEN:\1,fork,reuseaddr TCP4:\2:\3 \&/' && echo wait) | sh
 ---> Using cache
 ---> 7ea8aef582cc
Successfully built 7ea8aef582cc
```

​	如果想使用某个镜像的缓存，可以使用 **- -cache-from** 。

## 2. 格式

​	Dockerfile 的格式如下：

```docker
# Comment
INSTRUCTION arguments
```

​	**INSTRUCTION** 不是大小写敏感的，通常会使用大写以方便区别参数。

​	Docker 会逐条执行 **Dockerfile** 中的指令。第一条指令必须是 **FROM** 说明构建的基础镜像。

​	Docker 会把以 **#** 开头的行作为注释，除非此行是一个有效的 **parser directive** 。其他地方的 **#** 被当做参数。

```docker
# Comment
RUN echo 'we are running some # of cool things'
```

​	注释中不支持行连接符

## 3. Parser directives

​	**parser directives**  是可选的，会影响 **Dockerfile** 内容的处理方式。它不会作为一层来构建，也不会显示为构建步骤。**parser directives** 的格式是 **# directive=value** 。

​	一旦 注释，空行或 构建指令 被处理，**Docker** 不会再处理 **parser directives**，将会把它当做注释。因此 **parser directives** 必须在 **Dockerfile** 的顶部。

​	**parser directives** 不是大小写敏感的，一般会使用小写，在 **parser directives** 之后使用空行。不支持行连接符：

由于行连接符无效：

```docker
# direc \
tive=value
```

由于出现两次而无效：

```docker
# directive=value1
# directive=value2

FROM ImageName
```

由于出现在构建指令之后，被当做注释：

```docker
FROM ImageName
# directive=value
```

由于不能解析为 **parser directive** 而被当做注释：

```docker
# About my dockerfile
FROM ImageName
# directive=value
```

非断行空白是允许的，因此，下面的格式是等价的：

```docker
#directive=value
# directive =value
#	directive= value
# directive = value
#	  dIrEcTiVe=value
```

支持下面的 parser directive ：

- escape

## 3. escape

```dockerfile
# escape=\ (backslash)

or

# escape=` (backtick)
```

​	**escape directive** 用于设置 **Dockerfile**  中的转义字符。如果没有指定，默认为 `\`。

​	转义字符既可用于转义字符，也可用于换行。这样可以是 **Dockerfile** 的指令跨越多行。

​	Windows 适合使用 **`** 作为转义字符。

```dockerfile
FROM microsoft/nanoserver
COPY testfile.txt c:\\
RUN dir c:\
```

​	执行后为：

```Cmd
PS C:\John> docker build -t cmd .
Sending build context to Docker daemon 3.072 kB
Step 1/2 : FROM microsoft/nanoserver
 ---> 22738ff49c6d
Step 2/2 : COPY testfile.txt c:\RUN dir c:
GetFileAttributesEx c:RUN: The system cannot find the file specified.
PS C:\John>
```

 使用 **`** 作为转义字符：

```dockerfile
# escape=`

FROM microsoft/nanoserver
COPY testfile.txt c:\
RUN dir c:\
```

执行结果：

```cmd
PS C:\John> docker build -t succeeds --no-cache=true .
Sending build context to Docker daemon 3.072 kB
Step 1/3 : FROM microsoft/nanoserver
 ---> 22738ff49c6d
Step 2/3 : COPY testfile.txt c:\
 ---> 96655de338de
Removing intermediate container 4db9acbb1682
Step 3/3 : RUN dir c:\
 ---> Running in a2c157f842f5
 Volume in drive C has no label.
 Volume Serial Number is 7E6D-E0F7

 Directory of c:\

10/05/2016  05:04 PM             1,894 License.txt
10/05/2016  02:22 PM    <DIR>          Program Files
10/05/2016  02:14 PM    <DIR>          Program Files (x86)
10/28/2016  11:18 AM                62 testfile.txt
10/28/2016  11:20 AM    <DIR>          Users
10/28/2016  11:20 AM    <DIR>          Windows
           2 File(s)          1,956 bytes
           4 Dir(s)  21,259,096,064 bytes free
 ---> 01c7f3bef04f
Removing intermediate container a2c157f842f5
Successfully built 01c7f3bef04f
PS C:\John>

```

## 4. 环境置换

​	通过 **ENV** 语句声明的环境变量可以用在 **Dockerfile** 中某些指令的变量。

​	Dockerfile中的环境变量记法：**$variable_name** 或 **${variable_name}**。

​	 **${variable_name}** 也支持一些标准的 **bash** 修饰：

- **${variable_name:-world}** 如果变量设置过，结果为变量的值；否则为 **world**；
- **${variable_name:+world}** 如果变量设置过，结果为 **world**；否则为空字符串；



​	**Dockerfile** 中以下指令支持环境变量：

- ADD
- COPY
- ENV
- EXPOSE
- LABEL
- USER
- WORKDIR
- VOLUME
- STOPSIGNAL
- ONBUILD（1.4 版本之前不支持）

示例：

```dockerfile
ENV abc=hello
ENV abc=bye def=$abc
ENV ghi=$abc
```

def 的值为 hello，ghi 的值为 bye，如果在 第二行值应该是 hello

## 5. .dockerignore 文件

​	在 **docker CLI** 发送 *context* 给 **docker daemon** 前，会查找 **.dockerignore** 来排除匹配的文件和目录。避免了发送大、敏感的文件和目录；避免了使用 **ADD** 或 **COPY** 指令时，潜在的把他们添加到镜像。

​	**.dockerignore** 是以换行符分割的模式。在匹配时，*context* 的根作为根目录和工作目录。例如，模式 **/foo/bar** 和 **foo/bar** 都是排除 **PATH** 或 **URL** 下 **foo** 目录里的 bar 文件或目录。

​	如果 **.dockerignore** 文件中第一列以  **#**  开始，此行将会被当做注释。

​	一个  **.dockerignore**  示例：

```.dockerignore
# comment
	*/temp*
	*/*/temp*
	temp?
```

| 规则          | 行为                             |
| ----------- | ------------------------------ |
| # comment   | 忽略                             |
| `*/temp*`   | 排除根目录下，子目录中以 temp 开头的文和目录      |
| `*/*/temp*` | 排除根目录下，二级子目录中以 temp 开头的文件和目录   |
| temp？       | 排除根目录下，以 temp 开始，一个字符扩展的文件 和目录 |



​	匹配的时候使用 go 语言的 filepath.Match， 使用 filepath.Clean 来清除头尾的空白字符，删除  **.**  和 **..** 元素。

​	超过  go 语言的 filepath.Match 功能， **Docker** 支持通配字符 `**` 匹配零到多层的目录。以`!` 开始的行作为排除的例外。

​	`!` 出现的位置会影响行为：**.dockerignore** 的最后一行决定包含，还是排除。

```.dockerignore
    *.md
    !README*.md
    README-secret.md
```

​	*context* 中只包含 README*.md， 但是不包含 README-secret.md。

```.dockerignore
    *.md
    README-secret.md
    !README*.md
```

​	最后一句包含了上一句，因此 README 开头的md 文件都会发送给 **docker daemon**

​	**.dockerignore** 也可以排除  **Dockerfile** 和 **.dockerignore** 文件。但是由于需要还是会发送给 **docker daemon** ，在使用 **ADD** 和 **COPY** 指令时不会将其拷贝到镜像。

## 6. FROM

```dockerfile
FROM <iamge>
```

或者

```dockerfile
FROM <iamge>:<tag>
```

或者

```dockerfile
FROM <image>@<digest>
```

​	**FROM** 设置后续指令的基础镜像。一个有效的 **Dockerfile** 必须把 **FROM** 作为第一条指令。

- **FROM** 必须是 **Dockerfile** 中第一条非注释指令；
- 在一个 **Dockerfile** 中 可以出现多个 **FROM** 指令来创建多个镜像（还未遇到）
- **tag** 和 **digest** 的值是可选的。如果省略，默认为 **latest** 。



## 7. RUN

​	**RUN** 有两种格式：

- RUN <command> 	(shell 格式，命令在 **shell** 中运行，在 linux 下默认为 **/bin/sh -c** ,windows 默认为 **cmd /S /C** )
- RUN ["executable","param1","param2"]        (exec 格式)



​	**RUN** 指令会在当前镜像之上执行命令，生成的结果作为 **Dockerfile** 中下一条指令的基础镜像。

​	exec 格式可以避免 shell string munging，**RUN** 命令可以使用一个并不存在可执行脚本的的基础镜像。

​	shell 格式的默认脚本可以使用 **SHELL** 指令来改变。

> 注意：exec 格式解析为 json 数组，因此必须使用双引号(")

> 注意：注意：与 shell 格式不同，exec 格式不会调用命令行脚本。这意味着：脚本处理不会发生。
>
> ​	例如， RUN ["echo", "$HOME"] 不会在 **$HOME** 上做变量替换。如果希望脚本处理，要么使用 shell 格				   				   式，要么执行一个shell，RUN [ "sh","-c","echo $HOME"]。
>
> ​	使用 exec 格式执行一个 shell 时，如同 shell 格式一样，环境变量的扩展是通过 shell 而不是docker。
>
> ​	**RUN** 指令的缓存是在下一次构建时是有效的。可以使用 **docker build —no-cache** 来使缓存无效。

​	**ADD** 指令可以使用**RUN** 指令的缓存无效（详见 **ADD** 指令）

## 8. CMD

​	**CMD** 指令有3种格式：

- CMD ["executable","param1","param2"] 		(exec 格式）

- CMD ["param1","param2"]                                     (作为 **ENTRYPOINT** 的参数)

- CMD command param1 param2                   (shell form)

  在 **Dockerfile** 中只能有一个 **CMD**。如果出现多个 **CMD**，只有最后一个起作用。

  **CMD** 的主要作用是给执行容器提供默认执行程序，省略执行程序时，可以在 **ENTRYPOINT** 中指定。

> 注意：如果 **CMD** 用于给 **ENTRYPOINT** 指令提供参数时，**CMD** 和 **ENTRYPOINT** 指令都应该使用 json 数组格式

​	使用 **shell** 或 **exec** 格式时， **CMD** 指令用于设置镜像执行命令。

​	如果使用 **shell** 格式的 **CMD**，命令将会以 **/bin/sh -c** 执行。

​	**exec** 是 **CMD** 执行的最佳格式，所有的参数都必须以双引号的字符串形式。

​	如果想每次启动容器时，执行相同的可执行文件，应当将 **ENTRYPOINT** 和**CMD** 联合使用。

​	在 **docker run** 中指定的参数会覆盖 **CMD** 中的默认值。

> 注意：不要将 **RUN** 和 **CMD** 混淆。**RUN** 指令会运行、提交结果；**CMD** 指令在构建时不会执行。

## 9. LABEL

```dockerfile
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```

​	**LABEL** 指令给镜像添加元数据。格式为键值对。值中如果有空格，使用引号和反斜线。

```dockerfile
LABEL "com.example.vendor"="ACME Incorporated"
LABEL com.example.label-with-value="foo"
LABEL version="1.0"
LABEL description="This text illustrates \
that label-values can span multiple lines."
```

​	一个镜像可以有多个 **LABEL**。**Docker** 推荐将多个 **LABEL** 合并为一个  **LABEL**——每个 **LABEL** 指令会产生一个新层。

```dockerfile
LABEL multi.label1="value1" multi.label2="value2" other="value3"

# 或者

LABEL multi.label1="value1" \
      multi.label2="value2" \
      other="value3"
```

​	如果 **Docker** 遇到一个已存在的标签，新值会覆盖旧值。

​	可以使用 **docker inspect** 查看镜像的标签。

## 10. MAINTAINER (deprecated)

```dockerfile
MAINTAINER <name>
```

​	**MAINTAINER** 指令设置镜像的作者。使用 **LABEL** 会更加灵活，建议使用 **LABEL** 指令。

```dockerfile
LABEL maintainer "SvenDowideit@home.org.au"
```

## 11. EXPOSE

```dockerfile
EXPOSE <port> [<port>...]
```

​	EXPOSE 指令指定 Docker 容器在运行时监听的网络端口。

## 12. ENV

```dockerfile
ENV <key> <value>			# 第一种形式，空格之后的所有字符串都会被当做值
ENV <key>=<value> ...		# 第二种形式，可以同时设置多个，以空格分割
```

```dockerfile
ENV myName="John Doe" myDog=Rex\ The\ Dog \
    myCat=fluffy
    
# 作用等同于下面，但是只产生一个层
ENV myName John Doe
ENV myDog Rex The Dog
ENV myCat fluffy
```

​	**ENV** 指令设置的环境变量会在镜像运行的容器中持续。可以使用 **docker inspect** 查看值，使用 **docker run —env <key>=<value>** 来设置新值。

> 只给一个命令设置一个变量，使用 **RUN <key>=<value> <command>**

## 13. ADD

​	**ADD** 有两种格式：

- **ADD** <src>…	<dest>

- **ADD** ["<src>",…,"<dest>"]               (这种格式的路径可以包含空白字符)

  ​

  *ADD** 指令复制 **<src>** 中新文件、目录或远程的URLS，把它们放在镜像 **<dest>** 路径下的文件系统中。

  **<src>** 可以指定多个，都是相对于 **context** 的路径。**<src>** 包含的通配符将使用 Go 语言的 filepath.Match。

  **<dest>** 是绝对路径，或相对于 **WORKDIR** 的路径。

  创建文件和目录的 **UID** 和 **GID** 都是0。

  如果 **<src>** 是一个远程文件的URL，那么复制之后会有权限 600。 如果获取远程文件得到 Last-Modified 头，http头部的时间戳将被设为目的文件的 **mtime**。然而，在 **ADD** 处理其他文件时，mtime 不会包含 

> 注意：如果构建的 **Dockerfile** 通过 **STDIN**（**docker build - < somefile**），那就没有构建的 **context**，这个 **Dockerfile** 只能包含 基于**URL** 的**ADD** 指令。也可以通过 **STDIN**（**docker build -< archive.tar.gz**）传递一个压缩包。压缩包里的 除 **Dockerfile** 的其他文件将作为构建的 **context**。

> 注意：如果一个URL文件通过认证保护，由于 **ADD** 不支持认证，应该使用 **RUN wget** ，**RUN curl** 其他工具。

> 注意：如果 **ADD** 指令的  **<src>** 文件的内容改变，将会使之后的缓存无效。

​	**ADD** 指令遵守如下规则：

-  **<src>** 的路径必须包含正在构建的 *context* 中

- 如果 **<src>** 是 URL，同时 **<dest>** 没有以反斜线结束，将会从 URL 下载文件并拷贝到 **<dest>**，文件内容

- 如果 **<src>** 是 URL，同时 **<dest>** 以反斜线结束， URL中的文件及文件名会被下载到** <dest>/<filename>**

- 如果 **<src>** 是目录，文件夹的内容，以及文件系统的元数据都会被拷贝。

  > 注意：拷贝的是文件夹的内容，不是文件夹

- 如果 **<src>** 是一个本地压缩包（依据文件内容，不是文件名），将会把它解压为目录。 远程 URL 的资源不会被解压缩。

- 如果 **<src>** 是其他格式的文件，将会随元数据一起拷。这种情况下，如果 **<dest>** 以反斜线结束，将会被当做目录，**<src>** 的内容将会被写在 **<dest>/base(src)**

- 如果指定多个 **<src>** (直接使用多个，或者使用通配符)，**<dest>** 必须是目录，必须以 / 结束

- 如果 **<dest>** 没有以 / 结束，将会被当做一般文件，**<src>** 的内容将会被写在 **<dest>** 中。

- 如果**<dest>** 不存在，那么不存在的目录都会被创建。

## 14. COPY

​	**COPY** 指令有两种格式：

- **COPY** <src>… <dest>

- **COPY** ["src",…,"src"]            (这种格式的路径可以包含空格)

  **COPY** 指令把 **<src>** 指定的文件或目录，添加到 容器 **<dest>** 目录下文件系统里。

  **<src>** 可以指定多个，都是相对于 **context** 的路径。**<src>** 包含的通配符将使用 Go 语言的 filepath.Match。

  **<dest>** 是绝对路径，或相对于 **WORKDIR** 的路径。

  创建文件和目录的 **UID** 和 **GID** 都是0。

  > 如果使用 **STDIN** （docker build - < somefile）,不存在构建上下文，不能使用 **COPY**



​	**COPY** 遵循下面规则：

-  **<src>** 的路径必须包含正在构建的 *context* 中

- 如果 **<src>** 是目录，文件夹的内容，以及文件系统的元数据都会被拷贝。

  > 注意：拷贝的是文件夹的内容，不是文件夹


- 如果 **<src>** 是其他格式的文件，将会随元数据一起拷。这种情况下，如果 **<dest>** 以反斜线结束，将会被当做目录，**<src>** 的内容将会被写在 **<dest>/base(src)**
- 如果指定多个 **<src>** (直接使用多个，或者使用通配符)，**<dest>** 必须是目录，必须以 / 结束
- 如果 **<dest>** 没有以 / 结束，将会被当做一般文件，**<src>** 的内容将会被写在 **<dest>** 中。
- 如果**<dest>** 不存在，那么不存在的目录都会被创建

## 15. ENTRYPOINT

​	**ENTRYPOINT** 有两种格式：

- ENTRYPOINT ["executable", "param1", "param2"]        (exec form, preferred)

- ENTRYPOINT command param1 param2 (shell form)

  **ENTRYPOINT**  指定运行时的执行文件：**docker run <image>** 指定的参数会附加到 exec 格式的参数之后。使用  **docker run —entrypoint**   来覆盖 ENTRYPOINT 指令。

  shell 格式不会使用 **CMD** 和 **run** 的参数，会启动 **/bin/sh -c** 子命令。这意味着 执行程序的 PID 不会是 1，不会加收 Unix 的信号.

  shell 格式为了保证 docker stop信号正确的到 **ENTRYPOINT** 执行点，必须记住使用 **exec ** 开始：

  ```dockerfile
  FROM ubuntu
  ENTRYPOINT exec top -b
  ```

  ```docker
  $ docker run -it --rm --name test top
  Mem: 1704520K used, 352148K free, 0K shrd, 0K buff, 140368121167873K cached
  CPU:   5% usr   0% sys   0% nic  94% idle   0% io   0% irq   0% sirq
  Load average: 0.08 0.03 0.05 2/98 6
    PID  PPID USER     STAT   VSZ %VSZ %CPU COMMAND
      1     0 root     R     3164   0%   0% top -b
  ```

  ​

### 15.1 CMD 和 ENTRYPOINT的交互

​	以下为两者协作的规则：

- Dockfile 至少指定一条 **CMD** 或 **ENTRYPOINT** 指令
- 当作为可执行的容器时，应该定义 **ENTRYPOINT**
- **CMD** 应该被用于指定 **ENTRYPOINT** 的默认参数。
- 当运行时指定参数时，**CMD** 中的参数会被覆盖

|                                | No ENTRYPOINT              | ENTRYPOINT exec_entry p1_entry | ENTRYPOINT [“exec_entry”, “p1_entry”]    |
| ------------------------------ | -------------------------- | ------------------------------ | ---------------------------------------- |
| **No CMD**                     | *error, not allowed*       | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry                      |
| **CMD [“exec_cmd”, “p1_cmd”]** | exec_cmd p1_cmd            | /bin/sh -c exec_entry p1_entry | **exec_entry p1_entry exec_cmd p1_cmd**  |
| **CMD [“p1_cmd”, “p2_cmd”]**   | p1_cmd p2_cmd              | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry p1_cmd p2_cmd        |
| **CMD exec_cmd p1_cmd**        | /bin/sh -c exec_cmd p1_cmd | /bin/sh -c exec_entry p1_entry | **exec_entry p1_entry /bin/sh -c exec_cmd p1_cmd** |



## 16. VOLUME

```dockerfile
VOLUME ["/data"]
```

​	**VOLUME** 指令创建了一个命名的挂载点，来挂载本地货其他容器的卷。其值可以是 json 数组 **VOLUME ["/var/log/"]** ,或者是多个参数的字符串，**VOLUME /var/log** 或者 **VOLUME /var/log /var/db**

​	**docker run** 命令使用基础镜像中已存在位置的数据初始化卷

```dockerfile
FROM ubuntu
RUN mkdir /myvol
RUN echo "hello world" > /myvol/greeting
VOLUME /myvol
```

​	使用 **docker run** 命令，创建了挂载点 /myvol，同时将greeting文件复制到新卷里。

### 16.1 关于 volumes的注意点

​	在 **Dockerfile** 中使用 **volume** 注意一下几点：

- 基于windows 容器的 **volume**:容器中目标卷的必须是
  - 不存在或空文件夹
  - 其他驱动，而不是 C：
- 在 **Dockerfile** 中改变 volume 的内容：在声明 volume 之后，改变其中的数据都会被丢掉
- JSON 格式
- 主机的文件夹在容器运行时声明

## 17. USER

```dockerfile
USER daemon
```

​	USER 指令用于说明：在**Dockerfile** 中声明之后，**RUN**、**CMD**、**ENTRYPOINT**指令的用户或 UID

## 18. WORKDIR

```dockerfile
WORKDIR /path/to/workdir
```

​	**WORKDIR** 指令用于说明：在**Dockerfile** 中出现声明之后 **RUN** 、**CMD**、**ENTRYPOINT**、**COPY** 和 **ADD**  的工作目录。如果 **WORKDIR** 不存在，即使之后没使用，也会进行创建。

​	在 **dockerfile** 中可以出现多次，如果出现了相对路径，将是相对于上一个 **WORKDIR** 的路径：

```dockerfile
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd				#   /a/b/c
```

​	**WORKDIR** 指令可以解析之前使用 **ENV** 设置的环境变量，只能使用在 **Dockerfile** 中设置的环境变量：

```dockerfile
ENV DIRPATH /path
WORKDIR $DIRPATH/$DIRNAME
RUN pwd 			# 	/path/$DIRNAME
```

## 19. ARG

```dockerfile
ARG <name>[=<default value>]
```

​	**ARG** 指令定义在使用 **docker build** 命令，**—build-arg <varname>=<value>** 标志在构建阶段传递的参数。如果用户指定的参数不在 **Dockfile** 中，会输出一条警告：

```dockerfile
[Warning] One or more build-args [foo] were not consumed.
```

​	定义**Dockerfile** 的作者可以多次使用 **ARG** 来定义多个变量：

```dockerfile
FROM busybox
ARG user1
ARG buildno
...
```

​	也可以指定默认值：

```dockerfile
FROM busybox
ARG user1=someuser
ARG buildno=1
...
```

​	 从定义的行开始起作用。

​	对于

```dockerfile
1 FROM busybox
2 USER ${user:-some_user}		# 此处没有定义，因此值为 some_user
3 ARG user
4 USER $user					# 此处的值为 what_user
...
```

​	使用

```dockerfile
$ docker build --build-arg user=what_user Dockerfile
```

> 警告: 强烈建议不要使用构建时的变量来传递秘钥，认证信息。因为可以使用 docker history 查找到。

​	可以使用 **ARG** 和 **ENV** 指令来指定 **RUN** 指令运行时的变量。使用 **ENV** 指令定义的环境变量会覆盖 **ARG** 定义的变量。

​	对于

```Dockerfile
1 FROM ubuntu
2 ARG CONT_IMG_VER
3 ENV CONT_IMG_VER v1.0.0
4 RUN echo $CONT_IMG_VER			# v1.0.0
```

使用

```Dockerfile
$ docker build --build-arg CONT_IMG_VER=v2.0.1 Dockerfile
```

​	docker 预定义了一些 **ARG** ：

- HTTP_PROXY
- http_proxy
- HTTPS_PROXY
- https_proxy
- FTP_PROXY
- ftp_proxy
- NO_PROXY
- no_proxy

使用时，直接在命令行指定

```docker
--build-arg <varname>=<value>
```

### 19.1 对缓存的影响

​	当 **ARG** 的值不同于之前的版本，会造成缓存失效。不是在定义的时候缓存失效。

## 20. ONBUILD

```dockerfile
ONBUILD [INSTRUCTION]
```

​	当镜像作为构建其他镜像的基础镜像时候，才会触发。

​	**ONBUILD** 不会继承。

> 警告： **ONBUILD** **ONBUILD** 是不允许的

> 警告： **ONBUILD** 指令不会触发 **FROM** 或 **MAINTAINER** 指令

## 21. STOPSIGNAL

```dockerfile
STOPSIGNAL signal
```

​	用于设置退出时发送给容器的信号。

## 22. HEALTHCHECK

​	**HEALTHCHECK** 指令有两种格式：

- HEALTHCHECK [OPTIONS] CMD command       (通过运行容器内部的命令)

- HEALTHCHECK NONE （ 禁止从基础镜像继承的 healthcheck）

  这个指令告诉 Docker 如何测试一个容器是否在工作状态。可以检测运行中的Web 应用是否卡在了无线循环，是否能处理新连接。

  ​


  当容器指定了 healthcheck，在状态中多了一个健康状态。这个状态初始化为 **starting**，当 healthcheck 通过，状态转为 **healthy**，当连续几次失败后，状态变为 **unhealthy**。

  在 **CMD**之前的可选参数可以是：

- —interval = DURATION (default: 30 s)

- —timeout = DURATION (default: 30 s)

- —retries = N (default: 3)

  在 Dockerfile 中只能有一条 HEALTHCHECK 指令，如果出现多个，只有最后一条起作用。

  **CMD** 之后的命令可以是 shell 格式，也可以是 exec 格式。命令的退出状态指示容器的健康状态，可能的取值：

- 0：success

- 1：unhealthy

- 2:   reserved

```dockerfile
HEALTHCHECK --interval=5m --timeout=3s \
  CMD curl -f http://localhost/ || exit 1
```

## 23. SHELL

```dockerfile
SHELL ["executable","parameters"]
```

​	**SHELL** 指令可以使 linux 上的 ["/bin/sh","-c"] ，windows 上的 ["cmd","/S","/C"] 默认 shell 被覆盖。

​	**SHELL** 指令可以出现多次，每一条 **SHELL** 指令覆盖之前的 **SHELL** 指令，影响之后的指令。

```dockerfile
FROM microsoft/windowsservercore

# Executed as cmd /S /C echo default
RUN echo default

# Executed as cmd /S /C powershell -command Write-Host default
RUN powershell -command Write-Host default

# Executed as powershell -command Write-Host hello
SHELL ["powershell", "-command"]
RUN Write-Host hello

# Executed as cmd /S /C echo hello
SHELL ["cmd", "/S"", "/C"]
RUN echo hello

```

​	**SHELL** 指令可以影响到的指令有：**RUN**、**CMD**、**ENTRYPOINT**



​	在windows 下的一个示例：

```dockerfile
...
RUN powershell -command Execute-MyCmdlet -param1 "c:\foo.txt"
...
```

​	被docker调用时：

```docker
cmd /S /C powershell -command Execute-MyCmdlet -param1 "c:\foo.txt"
```

 	有两点是不有效的：1. 没有必要的 cmd.exe 命令；2. 需要 powershell 来修正之前的命令。

​	为了更高效，有以下两种方式：

	1. 使用 json 格式 的run 命令

```dockerfile
...
RUN ["powershell", "-command", "Execute-MyCmdlet", "-param1 \"c:\\foo.txt\""]
...

```

2. 使用 **SHELL** 指令，和shell 个是的命令

```dockerfile
# escape=`

FROM microsoft/nanoserver
SHELL ["powershell","-command"]
RUN New-Item -ItemType Directory C:\Example
ADD Execute-MyCmdlet.ps1 c:\example\
RUN c:\example\Execute-MyCmdlet -sample 'hello world'
```















