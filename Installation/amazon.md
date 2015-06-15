# AMAZON EC2 
---------------------

在Amazon EC2上安装Docker有许多方法。你可以使用Amazon Linux，因为在它的软件库中包含了Docker安装包，你也可以选择其他支持Linux的镜像，比如 Standard Ubuntu Installation.

##### 当然，前提是你要有一个AWS的账号。

## Amazon QuickStart with Amazon Linux AMI 2014.09.1 ##

最新的Amazon Linux AMI, 2014.09.1 已经支持Docker了，可以从AWS的软件库中获取Docker的安装包，步骤如下：

1. 选择你要使用的镜像
2. 在AWS控制台中点击[创建实例向导](https://console.aws.amazon.com/ec2/v2/home?#LaunchInstanceWizard:)
3. 在快速开始菜单中，选择 **AMI for Amazon Linux 2014.09.1**
4. 如果是为了简单的练习使用，你可以使用默认的 ```t2.micro``` 实例
5. 点击右下角的 **Next: Configure Instance Details**
6. 后面几步默认就可以了，现在你的Amazon Linux实例就已经跑起来了！
7. 现在，我们要在这个实例中安装Docker，首先要通过SSH连接到我们的实例：```ssh -i <path to your private key> ec2-user@<your public IP address>```
8. 连接上去后，安装Docker：```sudo yum install -y docker```
9. 运行Docker： ``` sudo service docker start``` 

##### 如果这是你第一次使用AWS ，你可能需要对SSH进行一些设置

默认情况下，实例中所有的输入端口都会被AWS安全组（ AWS Security Group）屏蔽，所以你在连接时可能会出现连接超时的错误。

Docker安装完毕后，你就可以参考[用户指南](../UserGuide/README.md)

## Standard Ubuntu Installation

如果你想自己动手安装Docker，你可以参照[Ubuntu安装方法](ubuntulinux.md)在你的EC2 ubuntu实例中安装。
