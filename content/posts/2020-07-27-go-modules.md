---
title: "Go Modules"
date: 2020-07-27T23:23:19+01:00
draft: true
tags:
  - go
categories:
  - go
---

### Go modules 常用命令
+ `go mod init`：创建一个新模块，初始化 go.mod 文件，参数为该模块的导入路径，推荐使用这种形式。如：`go mod init github.com/anggao/example`。
+ `go get`：更改依赖项版本（或添加新的依赖项）。
+ `go build、go test` 等命令：Go 命令行工具会根据需要添加新的依赖项。如：`go test ./...`，测试当前模块。
+ `go list -m all`：打印当前模块依赖。
+ `go mod tidy`：移除无用依赖。
+ `go list -m -versions github.com/gin-gonic/gin`：列出该模块的所有版本。
+ `go mod verify`：验证哈希。