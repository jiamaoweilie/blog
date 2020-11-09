# 持续集成之我见（三）

## 使用Jenkins搭建持续集成环境

下面我们以Jenkins为例，介绍一些如何快速搭建一个持续集成环境。

### 安装Jenkins

Jenkins可以运行在多种操作系统上，这里主要介绍如何使用docker安装。首先，你需要在本机上安装[docker](http://docker.io/),然后执行一下命令来pull Jenkins的官方镜像：

```
docker pull jenkins
```

然后运行如下命令来run该镜像：

```
docker run -d -p 49001:8080 -v $PWD/jenkins:/var/jenkins_home -t jenkins
```

在上述命令中，container中的 `/var/jenkins_home` 文件夹被映射到当前路径的 `jenkins/`路径下，container中的8080端口被映射到本地的49001端口。

至此，我们已经可以 http://localhost:49001 访问Jenkins, 并看到如下 页面。

![输入初始密码解锁Jenkins](/images/img_for_ci/jenkins_start.jpg)

系统要求你输入Administrator的初始密码来解锁系统，并提示了该密码的所在位置。对于docker来说，你可以输入如下操作查看该container的log，找到密码。

```
docker ps -a // 查看当前运行container
```

![container的运行状态](/images/img_for_ci/image.jpg)

```
docker logs 136d491d56d8 //数字表示Jenkins的container id
```

然后就可以在log中找到如下的初始密码：

![初始密码](/images/img_for_ci/password.jpg)

然后我们可以根据页面提示，安装推荐的plugin，完成注册等初始化操作，就可以看着Jenkins的工作页面。

![Jenkins 首页](/images/img_for_ci/jenkins.jpg)

至此，我们就完成了所有的安装和初始化的工作。

### 使用Jenkins

对于Jenkins 2.x版本，你可以使用以前的方式创建pipeline，当然比较方便的是Pipeline as code。下面我用一个简单的例子，说明如何使用脚本创建pipeline。

#### 准备工作

本文中的例子是一个简单的maven构建的java项目，所以准备工作为jenkins安装好java和maven。在Jenkins的首页点击Manage Jenkins -> Global Tool Configuration，然后如下图所示配置：

![jdk](/images/img_for_ci/jdk.jpg)

![maven](/images/img_for_ci/maven.jpg)

#### Pipeline创建

完成环境准备工作，我们就可以开始创建Pipeline。在首页点击New Item，然后输入一个item name，选择Pipeline，最后点击OK。

![创建item](/images/img_for_ci/new.jpg)

创建完item之后，就会进入配置页面，我们在这里需要配置trigger方式，以及Pipeline脚本等。配置pipeline脚本有两种方式，一种是写在页面上，一种是写在源代码中。

![配置item](/images/img_for_ci/script.jpg)

配置完成后，点击Build Now，就可以开始build我们的项目了。

![构建状态](/images/img_for_ci/build.jpg)

如图是我们的构建结果，一列是一个stage，每个stage执行脚本中定义的操作。

#### Pipeline说明

上述Pipeline有如下的基本概念：

* Stage: 一个Pipeline可以划分为若干个Stage，每个Stage代表一组操作。注意，Stage是一个逻辑分组的概念，可以跨多个Node。
* Node: 一个Node就是一个Jenkins节点，或者是Master，或者是Agent，是执行Step的具体运行期环境。
* Step: Step是最基本的操作单元，小到创建一个目录，大到构建一个Docker镜像，由各类Jenkins Plugin提供。

具体构成如下：

* Stage View: Pipeline的视觉展现，类似于上图。
* Jenkinsfile: Pipeline的定义文件，由Stage，Node，Step组成，一般存放于代码库根目录下，如下所示。

``` js
node {
   def mvnHome
   stage('Preparation') { // for display purposes
      // Get some code from a GitHub repository
      git 'https://github.com/jglick/simple-maven-project-with-tests.git'
      // Get the Maven tool.
      // ** NOTE: This 'M3' Maven tool must be configured
      // **       in the global configuration.           
      mvnHome = tool 'M3'
   }
   stage('Build') {
      // Run the maven build
      if (isUnix()) {
         sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"
      } else {
         bat(/"${mvnHome}\bin\mvn" -Dmaven.test.failure.ignore clean package/)
      }
   }
   stage('Results') {
      junit '**/target/surefire-reports/TEST-*.xml'
      archive 'target/*.jar'
   }
}
```

Jenkins 2.x 默认支持三种类型的Pipeline有三种，普通Pipeline，Multibranch Pipeline 和 Organization Folders，后两种其实是批量创建一组普通 Pipeline 的快捷方式，分别对应于多分支的应用和多应用的大型组织。注意，要获取 Organization Folders 的支持需要额外安装 Plugin。

至此我们已经介绍了 Jenkins 的安装以及 Pipeline as Code 的基本用法，更多关于 Jenkins 的操作请参见[官方文档](https://jenkins.io/doc/)。


