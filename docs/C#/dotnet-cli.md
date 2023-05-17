# DOTNET CLI 快速入门

[观看 B 站视频](https://www.bilibili.com/video/BV1xg4y1T7eu){ .md-button }

## 创建项目

```shell
# 要跟 dotnet 打交道
dotnet
# 查看已经安装的模板
dotnet new list
# 新建一个 console 项目，会在当前文件夹中新建一个 ConsoleApp 目录，里面存放 csrproj 文件
dotnet new console -n ConsoleApp
# 可以试着编译一下，并运行
dotnet build
dotnet run
# 使用 Release 模式运行
dotnet run -c Release
```

## 安装 NuGet 包

```shell
dotnet add package Newtonsoft.Json
dotnet remove package Newtonsoft.Json
```

## 创建一个测试项目

```shell
dotnet new xunit -n ConsoleApp.Test
# 此时 build 会因为有不同的 project，而不知道该 build 哪个
dotnet build ConsoleApp
# 为测试项目添加引用
dotnet add ConsoleApp.Test reference ConsoleApp
# 书写测试代码并运行
dotnet test ConsoleApp.Test
```

## 添加解决方案

```shell
dotnet new sln -n ConsoleApp
# 添加项目
dotnet sln add ConsoleApp
dotnet sln add ConsoleApp.Test
# 查看现有项目
dotnet sln list
# 编译解决方案
dotnet build
# 清理解决方案
dotnet clean
```

## 添加 git 版本控制

```shell
# 初始化本地 git 仓库
git init
# 从模板创建一个 gitignore 文件
dotnet new gitignore
```

