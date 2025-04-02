## 这里利用jenkins/jenkins:2.392-jdk11作为基础镜像，并在镜像内添加maven和sonar-scanner-cli并构建成新的镜像举例


### 1.首先下载要添加到基础镜像中的组件
wget https://dlcdn.apache.org/maven/maven-3/3.9.9/binaries/apache-maven-3.9.9-bin.tar.gz

wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.8.0.2856-linux.zip


### 2. vim创建Dockerfile文件添加以下内容
```dockerfile
#FROM：指定了构建镜像时的基础镜像。在这里，使用的是 jenkins/jenkins:2.392-jdk11
FROM jenkins/jenkins:2.392-jdk11
#USER：将用户切换为 root，以便执行需要管理员权限的操作。例如安装软件包等
USER root
#RUN：执行命令，并在镜像中创建新的层。这里是执行一系列命令：

#apt-get update：更新 apt 包管理器的索引。

#apt-get install -y unzip：安装 unzip 工具，用于解压 .zip 文件。

#apt-get clean：清理安装过程中产生的缓存文件，减少镜像的体积。

#rm -rf /var/lib/apt/lists/*：删除 apt 包管理器的缓存，进一步减小镜像体积。
RUN apt-get update && \
    apt-get install -y unzip && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

#COPY：将本地的 apache-maven-3.9.9-bin.tar.gz ,sonar-scanner-cli-4.8.0.2856-linux.zip文件复制到镜像内的 /usr/local/ 目录
COPY apache-maven-3.9.9-bin.tar.gz /usr/local/
COPY sonar-scanner-cli-4.8.0.2856-linux.zip /usr/local/
#WORKDIR：设置工作目录为 /usr/local/。后续的命令将在该目录下执行
WORKDIR /usr/local/
#RUN：tar -xzf 用来解压 .tar.gz 格式的文件。解压 apache-maven-3.9.9-bin.tar.gz 文件到 /usr/local/ 目录，并删除压缩包文件。
RUN tar -xzf apache-maven-3.9.9-bin.tar.gz && \
    rm apache-maven-3.9.9-bin.tar.gz
#unzip 用来解压 .zip 文件
RUN unzip sonar-scanner-cli-4.8.0.2856-linux.zip && \
    rm sonar-scanner-cli-4.8.0.2856-linux.zip
#ENV：更新镜像容器内 PATH 环境变量，确保 JAVA_HOME、MAVEN_HOME 和 SONAR_SCANNER_HOME 的 bin 目录被加入到 PATH 中，这样在容器内可以直接执行 java、mvn 和 sonar-scanner 等命令
ENV MAVEN_HOME=/usr/local/apache-maven-3.9.9
ENV SONAR_SCANNER_HOME=/usr/local/sonar-scanner-cli-4.8.0.2856
ENV PATH=$JAVA_HOME/bin:$MAVEN_HOME/bin:$SONAR_SCANNER_HOME/bin:$PATH
#RUN：允许 jenkins 用户在执行 sudo 命令时不需要输入密码。通过将 jenkins ALL=NOPASSWD: ALL 追加到 /etc/sudoers 文件，给予 jenkins 用户无密码的 sudo 权限
RUN echo "jenkins ALL=NOPASSWD: ALL" >> /etc/sudoers
#SER：切换回 jenkins 用户。这是出于安全考虑，Jenkins 在正常运行时应该使用非 root 用户来执行任务，避免以管理员身份运行不必要的程序
USER jenkins

```
构建镜像 ,（[这里以harbor仓库的格式构建后续需要可以推送到harbor](https://github.com/Zorinman/K8S/blob/main/%E9%83%A8%E7%BD%B2%E6%96%87%E6%A1%A3/habor%E9%95%9C%E5%83%8F%E4%BB%93%E5%BA%93.md)）
`docker build -t 192.168.219.129:80/library/jenkins-maven:v1 .`


构建完成即可查看新生成的镜像
`docker images`

推送到harbor 则在主机上配置好之后  
`docker login 192.168.219.129:80 `
`docker push 192.168.219.129:80/library/jenkins-maven:v1`
