# How to Use SSH
> Update: 2024/2/29 by Xuan Li

## Step 1: 隧道穿透
这里以常用的*Sakura Frp*为例
- 访问[SakuraFrp](https://www.natfrp.com/user/)注册账号并登陆
- 在网站主页上方栏选中 `服务` - `隧道列表`
- 在`隧道列表`界面, 单击右上角的`创建隧道`
- 在普通节点中选择含有 `推荐SSH` 标签的节点 (e.g. `枣庄多线1`) 单击 
- 创建隧道界面, 填写方式如下:
    - 隧道类型选择`TCP隧道`
    - 隧道名: 填写你要连接的服务器名称, 参考群文件, 如`lb_4090`
    - 备注: 可以不填
    - 本地IP: `127.0.0.1` (留空会默认填写此IP, 可以不填)
    - 本地端口: 下拉栏选择 `[22] SSH`
    - 远程端口: 不填, 其会自动生成
    - 自动HTTPS: 不填 (默认`禁用`)
    - 自动HTTPS工作模式: 不填
    - 访问密码: 单击`生成`后自动填写
    - 访问认证TOTP: 不填
    - 端口导出: 不填 (默认`禁用`)
    - IP白名单: 不填
    - IP黑名单: 不填

    上述项均填完后, 单击右下角的`创建`, 此时隧道列表界面会有一条刚刚创建的隧道信息, 隧道创建完成
- 在隧道配置文件中获取隧道密钥:
    - 单击该隧道的操作按钮`...`'选择`配置文件`
    - 配置文件界面顶端`-f`开头之后, 以一段数字结尾的即为隧道密钥, 单击隧道密钥右侧复制按钮可以复制密钥
    - 假设我们获取的隧道密钥为:
        
        `abcdefghij1kl2mno3pqr3st456uv7w8:66666666 `

- 启动隧道, 首先要在服务器上获取许可, 具体步骤为
    - 使用*ToDesk*访问你要连接的服务器
    - 按`Ctrl + Alt + T`调出控制台
    - 控制台中输入`frpc -f <隧道密钥>`以启动隧道, 例如
  
        `frpc -f abcdefghij1kl2mno3pqr3st456uv7w8:66666666`
    
    启动成功后控制台会输出包含以下内容的提示:
    ```
        TCP 类型隧道启动成功
        使用 [frp-abc.top:12345] 来连接到你的隧道
    ```
    > *Tips*: 启动成功后，回到你个人电脑的*SakuraFrp*网站的隧道列表界面并刷新, 会发现该隧道名称前的点变成了绿色, 说明目前该隧道是启动状态.
- 密钥认证, 需要认证才能允许你的个人电脑连接到此隧道
  - 点击该隧道的操作按钮`...`'选择`一键认证`
  - 下拉栏中选择`Windows(x86)`
  - 点击右下角`开始生成`
  - 点击下载生成的以*exe*结尾的认证程序, 并保存
  - 打开程序所在地址, 双击该程序运行, 弹出以下提示说明认证成功, 后可关闭该程序:
    ```
    认证成功, 现在可以正常连接了
    ```
    > *Tips*: 此程序有一定概率在多次运行后提示失败, 若后续可正常*SSH*访问则不必理会.
- 上述步骤完成后, 可以在配置文件(打开方法见上)中查看隧道关键信息, 例如
    ```
    user: abcdefghij1kl2mno3pqr3st456uv7w8 // 用户识别码
    server_addr: frp-abc.top                // 这是服务器地址
    remote_port: 12345                     // 这是远程端口号
    auth_pass: password                    // 这是之前生成的密码
    ```
## Step 2: 配置连接/访问
> *Warning*: 务必完成第一步后再进行此步

这里简单介绍两种常用远程方式, 可以自由选择也可以两种都尝试一下, 建议第一种成功后再尝试第二种.

### Method 1 通过PC的cmd直接进行SSH连接控制(易上手) 
#### For Windows 
- 获取需要连接**服务器**的当前用户名. 可以直接询问我们或者通过*ToDesk*连接相应服务器后, 在终端输入`whoami`获取, 假设我们获取到的用户名为:
    ```
        username = cathedral
    ```
- 在本地电脑安装*SSH*服务, 安装方法可以参考:
[Windows下在线与离线安装OpenSSH](https://zhuanlan.zhihu.com/p/666273532)
- 按下`WIN + R`组合键后输入`cmd`后单击确定, 启动控制台
- 假设第一步中我们获得
    ```
        server_addr = frp-abc.top
        remote_port = 12345
    ```
    则直接在cmd中输入
    ```
        ssh cathedral@frp-abc.top -p 12345 
    ```
- 第一次连接成功后会要求输入系统密码, 各个服务器密码可以私聊询问
    > *Tips*: 输入密码时默认不显示, 不要以为是键盘坏了, 输入完后直接回车即可.

### Method 2 通过PyCharm配置Deployment进行远程部署(更加系统)
在进行此步之前, 请先确认个人电脑上是否已安装**PyCharm专业版**而非**PyCharm社区版**(*PyCharm Community*). 若未安装, 可以到官网完成学生认证后安装专业版: [JetBrainsAccount](https://account.jetbrains.com/licenses)

- 在**本地**创建一个文件夹, 用于放置所有的远程项目, 将你要进行远程的项目放入此文件夹中. 例如创建的文件夹为`local_dep`, 项目为`demo_project`, 移动后的目录可以为: `E:\local_dep\demo_project`
- 在**服务器**上创建一个个人目录, 进入`/home/<username>/Proj`目录, 创建一个以你的**姓名首字母大写**作为名称的目录, 可以在控制台输入以下指令(假设`username: cathedral`), 便会在`Proj`文件夹下创建一个名为`QWE`的目录
    ```
        cd /home/cathedral/Proj
        mkdir QWE
    ```
    > *Tips*: 不是所有的服务器的工程目录名都为`Proj`, 建议先通过`whoami`指令获取*username*, 再`cd /home/username`到用户目录下, 通过`ls`指令查看所有的子目录名, 然后`cd <子目录名>`到相应的工程目录, 最后再通过`mkdir`创建你的个人目录.
- 在**服务器**上的个人目录中, 创建一个与你将要进行远程的项目同名的目录:
    ```
        cd QWE
        mkdir demo_project
    ```
    在`QWE`目录下, 可以使用`ls`指令查看是否创建成功
- 在**本地**使用*PyCharm*打开`demo_project`项目
  > *Windows*用户可以右击`demo_project`文件夹, 选择`显示更多选项`-`Open Folder as Pycharm Project`.
- 并在*PyCharm*主页面, 按`Alt + \`打开`Main Menu`, 依次选择`Tools` - `Deployment` - `Configuration`进入远程部署配置
- 先需要创建一个远程服务器, 单击左上角的`+`, 弹出栏中选择`SFTP`, `server name`仍然按照群中文件规则填入服务器命名, 如`lb_4090`
- 配置*SSH*, 需要用到我们**Step 1**中得到的`serve_addr`和`remote_port`:
  - 单击图中`SSH configuration`右侧的三个点, 打开`SSH configurations`界面
    ![dep_configure](./imgs/dep_config_img.png)
  - 在`SSH configurations`界面中, 点击左上角`+`新建一项配置并选中, 并在右侧填写**Step 1**中已经获得的配置信息, 填写完毕后点击`Test Connection`测试连接, 成功后点击`OK`完成*SSH*配置的添加, 如图所示
    ![ssh_configure](./imgs/ssh_config_img.png)
    > 需要注意, `Host`项填写相应的`<server_addr>`, `Port`项填写相应的`<remote_port>`, `Username`项填写相应的`<username>`, `Password`项填写相应的`<系统密码>`, 密码可以询问我们. 
- 返回到`Deployment`界面后, 选中添加的`lb_4090`, 在右侧`Connection`选项卡中的`SSH configuration`框中选择刚刚配置好的`cathedral@frp-abc.top:12345`, 再点击`Root path`项右侧的`Autodetect`自动检测当前用户目录, 生成后可查看是否符合`/home/<username>`的格式
- 配置项目目录映射. 单击`Mappings`选项卡, 确认当前`local path`是否是**本地**项目目录, 例如`E:\local_dep\demo_project`; 将此前在**服务器**上创建的相应的项目目录, 例如`/home/cathedral/Proj/QWE/demo_project`, 填入`Deployment path`, 配置完成后单击`OK`即可完成配置
    > Deployment path也可以通过点击右侧的小文件夹按钮逐级目录选中, 注意无论如何这个映射不要填写错误, 确认前务必检查目录是否正确.

恭喜到此你已经完成了*PyCharm*远程部署的基本配置, 下面是基于此的几个常用操作:
- **排除目录(非常重要!!!)**
    
    *PyCharm*中, 排除一个目录需要右击该目录后依次选择`Mark Directory as` - `Excluded`, 操作完后该目录变为橙色则说明操作成功
    > 对于任何一个项目, 在尝试上传到服务器之前, **务必**首先*排除*(*Exclude*)**数据文件夹**, 其他例如模型权重, 图片, 文档等目录也最好排除, 尽量保证只有模型和训练/测试等核心代码部分保留, 以防止其自动上传破坏源数据.
- 上传文件(或目录)
  
  右击想要上传的文件(或目录), 依次选择`Deployment` - `Upload to` - `<server name>`, `<server name>`为想要上传至的服务器名
    > *Tips 1:* 建议只选择小文件/目录(例如`.py`文件)进行上传, 大文件上传速度较慢且容易损坏;

    > *Tips 2:* 在**完成非核心文件夹的排除后**, 谨慎开启自动上传功能, 其可通过按`Alt + \`后依次选择`Tools` - `Deployment` - `Automatic Upload`开启.
- 浏览服务器远程目录
  
  按`Alt + \`后依次选择`Tools` - `Deployment` - `Browse Remote Host`后即可在页面右侧打开服务器目录, 右键一个目录同样可以直接进行操作, 但不建议通过此方法进行目录操作, 操作有延迟且误操作率高
  > 若进行上述操作后发现右侧`Remote Host`界面空白, 在确认先前配置无误的情况下, 可尝试重启本地*PyCharm*客户端.

- 内嵌*SSH*访问远程终端

    *PyCharm*提供了直接访问服务器终端的方法. 按`Alt + F12`打开`Terminal`界面后, 单击`Terminal`顶端的下箭头$\vee$, 下拉栏中选中此前添加的服务器, 即可开启远程终端
    > *Tips 1*: 连接前需要启动之前下载的密钥认证*exe*文件进行认证.
 
    >*Tips 2*: 远程终端操作与服务器终端操作完全一致, 但可能由于网络原因自动关闭, 因此较不稳定, 建议使用*screen*等进行会话管理, 详细可见: [Linux screen命令：如何在后台运行和管理多个终端会话](https://zhuanlan.zhihu.com/p/637765824)

- 配置远程*Python*解释器
  > 可以按照本教程配置, 也可以按照这个链接[Pycharm实现远程部署和调试](https://zhuanlan.zhihu.com/p/323261131)进行配置.
  
  为了更好的利用*PyCharm*的代码纠错和*Debug*功能, 建议配置远程编译器, 以方便使用服务器上配置好的环境进行代码纠错和*Debug*: 
  - 按`Alt + /`打开菜单后依次选择`File` - `Settings` - `Project` - `Python interpreter`
  - 单击右侧`Add Interpreter` - `On SSH`, 弹出菜单中`SSH connection` 选择 `Existing`, `SSH server`选择之前配置好的服务器, 单击`next`, 会自动进行连接测试, 成功后再单击`next`
  - 此时进入项目目录配置界面后, `Environment`选择`Existing`, `Interpreter`选择到服务器上你想使用的环境中的*python*解释器地址, 一般位于`/home/<username>/anaconda3/envs/<your_env_name>/bin/python`; `Sync folders`中确认`Local Path`为本地此项目的目录, `Remote Path`为服务器上该项目的目录, 一般`Remote Path`需要重新定位
  - 填写完毕后单击`create`即可, 使用新配置的解释器作为本项目的远程解释器

## 其他
- Linux终端命令可以参考: [常用的Linux命令行大全](https://zhuanlan.zhihu.com/p/420247468)
- 环境配置和PyTorch安装可以参考: [PyTorch安装教程(2022-02-22 知乎)](https://zhuanlan.zhihu.com/p/470841101)
- Pytorch官网： [Pytorch](https://pytorch.org/)
> *Tips 1* : **切勿泄露**任何服务器密码以及隧道配置文件;

> *Tips 2* : 使用服务器时, 请**及时关闭**不再使用的界面或终端, 以防引起卡顿;

> *Tips 3* : 尽量使用*SSH*访问服务器并直接在终端操作, 减少在图形界面的操作;

> *Tips 4* : 大家对于本说明文档有任何问题可以私聊我, 我将收集问题并不定期进行更新.