## 一、Windows安装Git客户端
请从iSource的3ms社区的文档中心下载Git客户端，或者是Git官网下载最新版本的Git客户端。

根据自己的操作系统位数（32位/64位），下载安装相应的这两个工具：Git（命令行工具）和TortoiseGit（可视化工具）。

## 二、Git配置&添加公钥
1、配置本地Git环境
配置用户名

your_name：用户名是你的域账号名，例如：r00310556

在命令行中输入：
git config --global user.name "r00310556"
配置邮箱
your_email：邮箱为你的华为邮箱，例如：xiaoming@huawei.com
git config --global user.email "xiaoming@huawei.com"
检查配置
最后检查user.name及user.email是否配置正确：
git config -l
2、公钥管理
SSH协议是一种非常常用的Git仓库访问协议，使用公钥认证、无需输入密码，加密传输，操作既便利又保证了安全性。公钥是iSource识别用户身份的一种认证方式，通过公钥，可以将本地git项目与iSource建立联系，然后就可以很方便的将本地代码上传到iSource，或者将iSource代码下载到本地了。

用户可以使用SSH协议访问iSource平台上的所有公共的、有权限的Git仓库 。

2.1、生成公钥

Git工具安装成功后运行 Git Bash，在弹出的客户端命令行界面中输入下面提示的命令。

生成新的密钥：

your_email：如xiaoming@huawei.com

$ ssh-keygen -t rsa -C “xiaoming@huawei.com”
注意：上面命令行中的“C”是大写的

在回车中会提示输入一个密码，这个密码会在提交代码时使用，也可以不输入密码，生成时一直回车，提交代码时则不用输入。在本机系统C盘下，用户文件夹里有一个.ssh文件，其中 id_rsa.pub文件里储存的即为刚生成的ssh公钥，id_rsa文件里储存的即为刚生成的ssh私钥。此时，需要把刚生成的id_rsa.pub文件里的公钥添加到iSource平台。

2.2、添加公钥

登录iSource平台，进入“Profile Settings”，点击左侧栏的“SSH Keys”，点击“Add SSH Key”，将刚生成的公钥文件的内容，复制到“Public Key”栏，保存即可。

注意：复制公钥时不要复制多余的空格，否则添加不成功。

3、TortoiseGit共用Git生成的密钥
TortoiseGit和Git Bash命令行工具，可以共用同一套公钥私钥（git bash生成的），如下操作：

1）开始菜单 -> 所有程序 -> TortoiseGit ->settings->Network->把SSH client的路径改成Git命令行工具安装目录的ssh.exe。

## 三、Fork主干库
Fork一个项目时，可以选择项目Fork到组织还是个人名下。点击“Fork”按钮后，根据页面弹出窗口提示选择相应组织名还是个人名下即可。
Fork成功，这里选择的是Fork到个人名下。

## 四、clone项目库到本地
新创建或者fork的项目，会自动分配一个git仓库地址，项目主页–>左侧点击overview–> 选择SSH或HTTPS协议（此处选择SSH），点击Copy Address：
1、使用TortoiseGit进行clone
第一步：在本地文件夹空白处“右击” -> 选择“Git Clone” 。
第二步：将刚刚copy的git仓地址，添加到URL处，确认无误后点击“OK”进行克隆。
2、使用git bash进行clone
第一步：在本地文件夹空白处“右击” -> 选择“Git Bash Here” 。
第二步：执行克隆命令：git clone xxxxxx.git

## 五、本地修改、提交、push
1、本地修改
代码克隆到本地后，我们会对它进行修改、增加或删除，之后，我们需要把这些修改进行提交并push到平台。

2、如何提交、push
使用TortoiseGit和git bash提交和push

2.1、使用TortoiseGit提交、push代码

第一步：在本地项目根目录下，右键选择“Git Commit -> “xx”…”

第二步：在Message栏填写提交信息->勾选需要提交的文件->点击“OK”

第三步：完成commit之后会出现提示框，点击“Push”将本次提交推送到远端


2.2 使用git bash提交、push代码

第一步：在本地项目根目录下，右键选择“git bash here”
第二步：查看当前代码的修改状态，执行命令：git status

第三步：将修改的文件提交到本地暂存区，执行命令：git add filename

或者将所有修改过的工作文件提交暂存区，执行命令： git add .

第四步：将暂存区的修改文件提交到本地git库中，执行命令：git commit后输入commit message，或者git commit -m "引号内输入commit message"

第五步：将提交的修改推送到远端，执行命令：git push origin master（例子中使用的是默认的远端origin，分支master）
## 
