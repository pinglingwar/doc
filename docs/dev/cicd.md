# 持续集成

## github action 自托管运行器

参考 [链接](https://docs.github.com/zh/actions/how-tos/manage-runners/self-hosted-runners/add-runners)  添加 runner。

###### 下载
使用管理员权限打开 Windows PowerShell，运行以下命令：
```shell
mkdir actions-runner; cd actions-runner
Invoke-WebRequest -Uri https://github.com/actions/runner/releases/download/v2.328.0/actions-runner-win-x64-2.328.0.zip -OutFile actions-runner-win-x64-2.328.0.zip
if((Get-FileHash -Path actions-runner-win-x64-2.328.0.zip -Algorithm SHA256).Hash.ToUpper() -ne 'a73ae192b8b2b782e1d90c08923030930b0b96ed394fe56413a073cc6f694877'.ToUpper()){ throw 'Computed checksum did not match' }
Add-Type -AssemblyName System.IO.Compression.FileSystem ; [System.IO.Compression.ZipFile]::ExtractToDirectory("$PWD/actions-runner-win-x64-2.328.0.zip", "$PWD")
```

###### 配置
```shell
./config.cmd --url https://github.com/OpenHUTB --token {TOKEN}
# 回车
# hutb
# 回车
# Y 作为服务运行
# 重新配置后，如果报错：Cannot configure the runner because it is already configured.
# 解决：删除.runner文件，然后重新运行命令
./run.cmd
# 如果碰到问题：A session for this runner already exists.
# 运行 services.msc，停止 Github Actions Runner 服务后问题就解决了。
./config.sh remove --token {TOKEN}
```

###### 使用
在项目中的`.github\workflows\*.yml`文件中添加`runs-on: self-hosted`，比如：
```yaml
jobs:

  windows-dev:
    name: Windows Dev
    runs-on: self-hosted
    steps:
      - name: Set up Git repository
        uses: actions/checkout@v3
```

`runs-on` 后面的名字为标签，支持 [自定义标签](https://docs.github.com/zh/actions/how-tos/manage-runners/self-hosted-runners/apply-labels#creating-a-custom-label-for-an-organization-runner) 。





## [Jenkins安装](https://sdpro.top/blog/html/article/1051.html)

1.安装 [JDK21](https://java.com/java/technologies/downloads/#jdk21-windows)

2.[安装 Jenkins 2.492.3 ](https://blog.csdn.net/qq_36746815/article/details/127393076)

作为本地服务；端口 8080


3.打开 [http://172.21.108.56:8080](http://172.21.108.56:8080)

b00acffa035946e18bb20924c6f77181

```text
用户名：cc
密码：a5300066
```

重启服务：
在地址栏中输入：`http://172.21.108.56:8080/restart/`


配置所使用的节点：`http://172.21.108.56:8080/manage/computer/`

需要配置jenkins代理，否则会出现各种连接不到github的错误。
Manager Jenkins -> System Configuration -> System -> 
Http Proxy Configuration：
```shell
# 服务器
127.0.0.1
# 端口
7890
```

## 亚马逊云配置
进入 [登录页面](https://signin.aws.amazon.com/) ，使用`Sign in using root user email`，然后依次输入邮箱和密码。

安装 [亚马逊云的命令行工具](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) ，
[创建访问密钥](https://us-east-1.console.aws.amazon.com/iam/home?region=ap-southeast-2#/security_credentials) ：
```shell
# 配置ID和访问密钥
aws configure
# 查看配置（可选）
aws configure list
# 下载文件
aws s3 cp s3://hutb/hutb.png .
# 上传文件
aws s3 cp Jenkinsfile s3://hutb/
```

（可能非必需）安装 AWS Credentials 插件，然后在设置中添加AWS Credentials。

在脚本中直接指定key_id和secret_access_key：
```shell
aws configure set aws_access_key_id XXX
aws configure set aws_secret_access_key XXX
```


## Jenkins配置

这个是因为我使用的本地的git用来测试，所以需要配置一下，在 jenkins安装目录（默认是C:\Program Files\Jenkins）找到 jenkins.xml

1.在配置页面的`流水线`下的`定义`下拉菜单中选择`Pipeline script from SCM` (Software Configuration Management) 。
仓库配置为`C:\ProgramData\Jenkins\.jenkins\workspace`，并制定分支。


2.开始构建

默认克隆在`C:\ProgramData\Jenkins\.jenkins\workspace`目录下。


## [Gitlab 安装](https://hub.docker.com/r/twang2218/gitlab-ce-zh)
Gitlab镜像拉取和安装
```shell
docker run -d -p 3000:80 twang2218/gitlab-ce-zh:11.1.4
```
配置(不适合中文版)
```shell
# 进入容器
docker exec -it gitlab bash

vi /etc/gitlab/gitlab.rb
#gitlab访问地址，可以写域名。如果端口不写的话默认为80端口
external_url 'http://172.21.108.56:9980' 
#ssh主机ip
gitlab_rails['gitlab_ssh_host'] = '172.21.108.56'
#ssh连接端口
gitlab_rails['gitlab_shell_ssh_port'] = 9922

# 重新编译gitlab配置文件
gitlab-ctl reconfigure
# 重启gitlab服务
gitlab-ctl restart

#退出容器
exit
# 重启gitlab容器
docker restart gitlab

# 查看初始密码
docker exec -it gitlab cat /etc/gitlab/initial_root_password
```

访问：[http://172.21.108.56:9980/](http://172.21.108.56:9980/)

数据持久化参考 [链接](https://blog.csdn.net/qq934235475/article/details/112864332) 。

问题：`HTTP 502: Waiting for GitLab to boot`

错误排查：
```shell
gitlab-ctl tail
```


## 虚幻引擎

根据UE4.sln找到对应的vs工程文件为`Engine\Intermediate\ProjectFiles\UE4.vcxproj`，得到构建虚幻引擎的命令为：
```shell
"C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\MSBuild\Current\Bin\MSBuild.exe" Engine\Intermediate\ProjectFiles\UE4.vcxproj
```

步骤同上，但是由于UnrealEngine是私有项目，在填写`Repository URL`时，需要带上github账户的Token，比如： `https://{token}@github.com/OpenHUTB/UnrealEngine.git`。

拉取 [中文版gitlab](https://hub.docker.com/r/twang2218/gitlab-ce-zh) 镜像
```shell
https://hub.docker.com/r/twang2218/gitlab-ce-zh
```

```shell
git remote add hutb http://172.21.108.56:3000/root/UnrealEngine
```

!!! 注意
    虚幻引擎编译命令参考 [链接](https://www.cnblogs.com/kekec/p/8684068.html)


### github action 编译

* [Github action](https://www.reddit.com/r/unrealengine/comments/1ajjmzq/anyone_tried_using_github_actions/)

这条路远非完美，仍然需要你付出一些代价，但我做了以下事情：

* 创建了一个 Azure 文件共享，其中包含一份虚幻引擎副本（我只是将 c:\program files\Epic Games\UE_5.3 目录复制到了该文件共享中。）
* 将 2022 年构建工具的配置放在该文件共享的配置目录中。我是通过在本地安装构建工具时选择正确的内容并导出来做到这一点的。
* 将 Azure 文件共享作为 GitHub 运行器的一部分挂载。
* 将存储密钥放在 GitHub 上的“settings/secrets”中。

注意：我无法让已安装的 VS 2022 构建工具正常工作。如果可以的话，运行器可以节省 7 分钟。但这种方法允许我选择一个我知道可以正常工作的特定编译器版本（并且该版本在 VS 构建工具配置中）。

我的运行器 YAML 如下所示：
```yaml
---
name: CI Build
# Controls when the workflow will be run
on:
push:
branches: [main]
pull_request:
branches: [main]
workflow_dispatch:


jobs:
    build:
    # The type of runner that the job will run on
    runs-on: windows-latest
    steps:     
        - name: Checkout Repository
          uses: actions/checkout@v4
          with:
            # Do a shallow fetchfetch-
            depth: 1
                  - name: Map Azure File Share
            run: |          
              net use Z: \\<StorageAccountName>.file.core.windows.net\u<AzureFileShareName> /user:<StorageAccountName> ${{ secrets.AZURE_STORAGE_KEY }}
              
        - name: Install Visual Studio Build Tools
          run: choco install visualstudio2022buildtools --package-parameters "--wait --quiet --addProductLang En-us --config Z:\config\v143-14.34-buildtools.vsconfig"
          
        - name: Setup MSVC v143 14.34 build tools
          uses: ilammy/msvc-dev-cmd@v1
          with:
            toolset: 14.34
            arch: x64

        - name: Build with Unreal
            uses: OrchidIsle/UE5-Build-Project@0.2.2
            with:
                RUNUAT_PATH: 'Z:/UE_5.3/Engine/Build/BatchFiles/RunUAT.bat'
                UPROJECT_PATH: ${{ github.workspace }}/<ProjectName>.uproject
                BUILD_CONFIG: Development
                PLATFORM: Win64
                COOK: false
                STAGE: false
```


## [gitlab提交代码触发Jenkins构建](https://blog.csdn.net/Habo_/article/details/123379435)

1.Jenkins页面中，系统管理 -> 插件管理 -> 可选插件 -> 搜索要**安装的插件**(`gitlab-plugin` 和　`Generic Webhook Trigger`)


2.在 Jenkins 的项目（**接收方**）配置中勾选`Generic Webhook Trigger` （按 [链接1](https://blog.csdn.net/qq_31594665/article/details/136995439) 和 [链接2](https://www.cnblogs.com/jiaxzeng/p/17104250.html) 中的说明进行设置）：

2.1 在 Post content parameters 中定义 post 请求的变量。Variable 中的 Name of variable 设置为：`ref`(在构建过程中需要使用到的变量名；将匹配到的值，复制给 ref 变量)，`Expression`设置为 `$.ref`(获取变量的值，匹配 gitlab 请求的参数)，勾选 `JSONPath`；

2.2 输入 Token（比如`hutb-admin-dev`），这个 token 在第 3 步中填写到 gitlab 的 webhook 链接的最后（防止其他人触发CICD），要保证在 Jenkins 中唯一；

2.3 Optional filter 中的 Expression为：`^(refs/heads/dev)$` （匹配构建条件的正则表达式，这里的hutb是匹配的分支名，可根据实际的分支名情况修改），`Text`为：`$ref` （匹配的值，可使用上面配置的任意变量或组合，构建只有在此处的值与给定的正则匹配时才会触发）。


3.在Gitlab（**请求方**）的项目页面：设置->导入所有仓库->链接(URL) 中填入`http://172.21.108.56:8080/generic-webhook-trigger/invoke?token=TOKEN` （该token和上面第2.2步Jenkins->UnrealEngine->Configuration->Triggers->Token里一致），取消`开启SSL证书验证`，点击 Test-> Push events 来触发 Jeinkins 的构建。



##### 添加钩子时报错：`Urlis blocked: Requests to the local network are not allowed`
> 原因：向同一台机器的IP发送请求不允许
> 
> 解决：Gitlab主页中的工具栏中的扳手->设置->外发请求，勾选`允许钩子和服务访问本地网络`（注意不是项目的设置）


参考：



## 高级配置


##### 轮询设置

默认不是东八区时间。
每隔2分钟执行一次查询，有更新就执行构建。
```text
TZ=Asia/Shanghai
H/2 * * * *
```


##### Jenkins关闭静默期

提交通常会以突发的方式出现。这主要有两个原因：一是人们有时会忘记提交某些文件；二是人们在等待 SCM 完成提交的过程中，有时会意识到提交中的问题，并迅速进行后续修改。传统观点认为，CI 服务器应该等待突发提交完成后再尝试构建。据说这可以降低构建失败的风险，有时也有助于缩短构建时间较长的平均周转时间。

关闭静默期可以实现提交代码后立即开始编译：Manage Jenkins -> System -> 静默期 设置为0。

##### 清除历史的构建的包
每次所构建的包位于`存放的路径是jobs/JOB_NAME/builds/BUILD_NUMBER/archive`，比如：`C:\ProgramData\Jenkins\.jenkins\jobs\carla\builds`，可能非常大，需要定时清除。

##### 将运行中的gitlab容器打包为镜像
1.将运行中的Docker容器保存为镜像
```shell
# 容器ID使用 docker ps 查看
docker commit <容器ID或名称> <镜像名称>:<标签> 
```

2.将镜像保存为tar文件
```shell
docker save -o <tar文件名>.tar <镜像名称>:<标签>
```

3.将镜像tar文件复制到本地
```shell
docker load -i <tar文件名>.tar
```

##### [配置构建失败时发送邮件](https://juejin.cn/post/6844904119707123719)

下载两个插件：Email Extension, Email Extension Template， 这两个插件可以帮助我们进行邮件的编写以及格式化。

在Jenkins的“设置->Account”中设置邮件地址，如“123456@qq.com”

在Jenkins的“Manage Jenkins -> System” 中的 `Extended E-mail Notification`：

问题: `org.eclipse.angus.mail.smtp.SMTPSenderFailedException: 501 Mail from address must be same as authorization user.`

> 需要将`Manage Jenkins —> System` 中的 `系统管理员邮件地址` 要和`邮件通知`中的地址一致（`123456@qq.com`）。

问题：`HTTP ERROR 403 No valid crumb was included in the request`

> 将`Manage Jenkins -> Security`中的`跨站请求伪造保护`中的`启用代理兼容`勾选。

其中的`Test e-mail recipient`为接收方邮箱地址（QQ邮箱为发送方）。[open3d_lidar.py](../../../carla/PythonAPI/examples/open3d_lidar.py)


##### 修改Jenkins主目录

修改配置文件`C:\ProgramData\Jenkins\.jenkins\config.xml`中的
```xml
<workspaceDir>${JENKINS_HOME}/workspace/${ITEM_FULL_NAME}</workspaceDir>
```
然后重启。


##### 日志太大
点开页面报错：`处理当前请求时发生了错误。` 


## Github Action 配置

UE4 发布工作流程需要手动触发，并且必须按照命名约定`ue4/x.y.z`在分支上执行。

## FAQ

###### make 等系统命令无法识别
报错信息：
```text
'make' is not recognized as an internal or external command,
operable program or batch file.
'make' is not recognized as an internal or external command,
operable program or batch file.
```

[解决办法](https://blog.csdn.net/sugar_shu/article/details/116301813) ：在 jenkins 上相应 node 的设定中（Dashboard -> Manage Jenkins -> Nodes -> Built-In Nodes，点击设置的齿轮按钮），在`节点属性`下，勾选`Environment variables`，`新增` make 的环境变量：PATH

```shell
%PATH%;C:\software\GnuWin32\bin;C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Windows\System32\OpenSSH\;C:\Users\Administrator\AppData\Local\Microsoft\WindowsApps;C:\software\anaconda3\condabin;C:\Users\Administrator\AppData\Roaming\npm;C:\software\anaconda3\envs\carla_dev;C:\jenkins\software\Git\bin
```

!!! 注意
    除了 make 相关的要添加进去，其他的 PATH 均需要复制，不然使用jenkins调用其他命令可能出错，且需要把其他环境变量中的%SYSTEMROOT%替换为C:\Windows。已有的 PATH 路径通过`echo %PATH%`来进行查看（后缀`%PATH%`？）。

##### 后台的 TGitCache.exe 导致文件不能正常删除

原因：TortoiseGit默认启动了状态缓存

解决：打开TortoiseGit设置>图标缓存部分的"Status Overlays
In“选择”None“单击OK。
TGitCache.exe将从进程列表中消失。

## 参考

* [社区的持续集成地址](http://158.109.8.172:8080)
* [Jenkins之Email配置与任务邮件发送实践与踩坑](https://juejin.cn/post/6844904119707123719)
* [QQ邮箱：什么是授权码，它又是如何设置？]()
* [gitlab仓库的CICD功能](https://mp.weixin.qq.com/s/AcmvXyWrdpIAmemfdkvgtA)



