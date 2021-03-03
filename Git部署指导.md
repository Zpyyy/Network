一、权限获取：

OLT领域支撑库权限      ： @杨静(y00317699)(杨静)

ONT领域支撑库权限     ： @吴灿(w00169332)(吴灿)/swx319135

MXU支撑库权限           ： @张啸虎(z00176502)(张啸虎)

XPON驱动领域权限      ： @王思更(w00347813)(王思更)

硬件集成与验证部权限  ： @江宏(j00320922)(江宏)

解决方案集成与验证部   :   swx319135

其他部门分支权限        ：  可咨询swx319135

二、安装包获取

注意：A、windows server 2003 32bit请务必安装Git-2.8.1-32-bit或以上版本
          B、如遇windows server 2003不能安装“TortoiseGit”，需先安装该补丁WindowsServer2003（附件），再进行安装

下载：根据本机类型下载“Git”和“TortoiseGit”的32位和64位版本

链接：http://3ms.huawei.com/hi/index.php?app=group&mod=File&act=showList&gid=2031557#category=1276461

三、安装和配置

安装：

执行下载的“Git”和“TortoiseGit”相关的".exe"文件即可，"TortoiseGit”全部默认安装，“Git”安装需注意以下选项，其他默认安装

配置：

本机配置：

1、桌面空白处右击，选择“Git Bash Here”

2、Git命令配置（工号和账号请更换为需配置相关人员）

$ git config --global user.name swx319135
$ git config --global user.email sunjuan2@huawei.com
$ git config -l
$ ssh-keygen -t rsa -C sunjuan2@huawei.com （此命令配置时提示需要填写内容时可以直接使用“回车”）

配置成功界面：

3、“TortoiseGit”秘钥配置

进行“PuTTYgen”和“Pageant"配置

4、免重启执行机反复进行“Pageant配置"设置

     桌面空白处右击---->TortoiseGit---->Settings--->Network
四、设置永久记住密码

git config --global credential.helper store 

五、账号复用
用于多台执行机需配置同一账号
1、进行如上配置中”本机配置“中的 ”1“、”2“后，复制本机.ssh文件替换需配置执行机.ssh文件夹
2、进行如上配置中”本机配置“中的”Pageant"配置“即可（远端配置无需配置）
