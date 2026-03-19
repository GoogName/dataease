# DataEase迁移方案

## 初始化安装
### 代码拉取
项目原始仓库地址：https://github.com/dataease/dataease.git
浏览器中打开上面地址，直接下载源码ZIP包即可


### 数据迁移
#### 全新创建数据库
执行代码目录`core/core-backend/src/main/resources/db/migration`下的所有SQL即可
#### Docker中旧数据迁移
```shell
# mysql容器
docker exec -it 678d870089aa bash
# 执行数据dump操作
mysqldump -u root -p'Password123@mysql' dataease > /var/lib/mysql/数据库备份文件.sql
# 退出容器，回到宿主机
exit
# 进入宿主机目录，查看文件
# /home/admin/soft/dataease/dataease2.0/data/mysql/数据库备份文件.sql
# docker容器的/var/lib/mysql目录是映射到宿主机的/home/admin/soft/dataease/dataease2.0/data/mysql目录下的
# 使用navicat执行 数据库备份文件.sql 即可
```

### 后端

#### 添加本地依赖库
在idea的项目结构中，将`drivers`目录下的jar包添加到项目中，防止后面启动报错

#### 配置修改
修改`application-standalone.yml`配置文件中的连接配置
```yml
spring:
  datasource:
    url: jdbc:mysql://192.168.2.202:3306/dataease?autoReconnect=false&useUnicode=true&characterEncoding=UTF-8&characterSetResults=UTF-8&zeroDateTimeBehavior=convertToNull&useSSL=false&allowPublicKeyRetrieval=true
    username: root
    password: Lxs123456..
  messages:
    basename: i18n/lic,i18n/core,i18n/permissions,i18n/xpack,i18n/sync
  flyway:
    enabled: true
    table: de_standalone_version
    validate-on-migrate: false
    locations: classpath:db/migration
    baseline-on-migrate: true
    out-of-order: true

mybatis-plus:
  mapper-locations: classpath:mybatis/*.xml
```
#### 启动
最低jdk版本： 21  
启动类： `io.dataease.CoreApplication`  
命令行启动, 需要指定各个数据库的驱动包目录`drivers`
```shell
java -Dloader.path=./drivers -jar ./CoreApplication.jar
```


### 前端
前端代码目录：`core/core-frontend/`

#### 安装依赖
```shell
npm install
```

#### 启动
```shell
# windows
npm run dev:win

# linux or mac
npm run dev
```

#### 异常
1. 页面访问报换行符异常： 修改`.editorconfig`文件
```ini
# windows
end_of_line = crlf

# linux or mac
end_of_line: lf
```


### 打包
#### 前端打包
##### 添加windows打包命令
编辑`core/core-frontend/package.json`文件，添加如下内容
```json
{
  "build:base:win": "set NODE_OPTIONS=--max_old_space_size=4096 && vite build --mode base && npm run build:flush",
  "build:distributed:win": "set NODE_OPTIONS=--max_old_space_size=5020 && vite build --mode distributed && npm run build:flush"
}
```
##### 执行打包
```shell
# Windows
npm run build:distributed:win
# Linux or Mac
npm run build:distributed
```
前端打包完成后，将`core/core-frontend/dist`目录下的文件复制到`core/core-backend/src/main/resources/static`目录下

#### 后端打包
##### 方式一
在项目根目录下执行打包命令，需要提前在系统中配置号maven的环境变量
```shell
mvn clean package -Dmaven.test.skip=true
```
##### 方式二
在idea的maven面板中直接点package打包

#### 访问地址
前端文件打入jar包访问地址：`http://192.168.2.202:8100/`


## 版本升级

### 代码合并
以当前为例，目前DataEase的主分支为`dev-v2`，我们本地开发的分支为`dev-my`。
要合并原始项目中的更新，整体思路是给当前项目设置两个远程地址，分别为`origin`和`upstream`。
先拉取`upstream`的更新到本地，这个更新在分支`dev-v2`中，然后将`dev-v2`合并到`dev-my`中，然后将`dev-my`提交推送到`origin`远程。

#### 设置远程地址
##### 方法一
命令行中执行如下命令添加远程操作
```shell
git remote add upstream https://github.com/dataease/dataease.git
git remote add origin https://codeup.aliyun.com/632c1b15257dab51ddaa571c/e00/dataease.git
```
##### 方法二
idea中设置远程地址： 菜单栏`Git` -> `Remote`，在弹出窗中直接添加上面两个远程地址

### 升级数据库
系统功能升级时，在上线前可能需要执行一段SQL脚本用于建表或初始化一些数据，可以将这些SQL脚本放在`core/core-backend/src/main/resources/db/migration`目录下，
脚本的名称需要遵循版本号的规律。在新版本项目启动时，会自动执行这些SQL脚本，完成数据库升级。  
SQL文件名称命名规则：`V${version}__ddl.sql`，如： `V2.10.19__ddl.sql`  
修改版本号只需要修改项目更目录下的`pom.xml`文件中的`<dataease.version>`标签即可；如`<dataease.version>2.10.19</dataease.version>`




