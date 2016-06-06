---
layout: post 
---

#### [1. 问题详细描述](1)

- 环境描述描述：

  Jenkins ver. 1.647

  OSX EI Caption 10.11.4

- 错误日志

  ```sh
  $ gradle clean build --stacktrace --debug
  FATAL: command execution failed
  java.io.IOException: Cannot run program "gradle" (in directory "/Users/name/.jenkins/workspace/app"): error=2, No such file or directory
  ```



#### [2. 问题分析](2)

根据错误信息可知：`gradle` 命令没有找到，此编译过程和我们平时执行 `shell` 编译是一样的，那么这个错误和我们使用 `shell` 执行某个命令时，遇到的 `Unknown command` 一个意思。平时我们是怎么解决这个问题的呢，大概两种方式 `cd` 到目录下或者通过添加环境变量的方式，其实只有一个目的，就是为了能够访问到该命令。



#### [3. 解决方案](3)

这里提供两种思路：常规解决方式，即能访问到该命令，这种方式更具有通用性； `gradle wrapper` 是 `gradle` 特有方式，比较简单方便。



##### [3.1 gradle wrapper 方式](#3.1)

此种方式比较简单快捷，就放到前面说，毕竟快速解决问题是我们的目的（：

进入**Project** 打开 **配置**，找到 **构建** 勾选 **Use Gradle Wrapper** 保存即可。



##### [3.2 环境变量方式](#3.2)

- 进入**Project** 打开 **配置**，找到 **构建** 勾选 **Invoke Gradle** ，这个是默认的状态。


- 进入 `tomcat bin` 目录下，修改 `catalina.sh` 文件，97 行添加(去掉注释空行就是第一行)

  ```sh
  export GRADLE_HOME="/Users/name/Desktop/gradle-2.10/bin"
  ```


- 重启 `tomcat` 即可。



#### [4. 总结](4)



> **踩坑之旅，一路惊喜，enjoy it，this is life !**



参考：xyz