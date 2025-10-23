+++
date = '2025-10-23T10:17:25+08:00'
draft = false 
title = '如何在nvim里配置C++的调试器？'
tags = ['nvim', 'c++']
+++

## 前言

> 如果你是一个追求效率的开发者或者学生，那么vim无疑是最好的选择，我使用的是nvim，用它写文章和代码。
> 过去我在学习和刷题的过程中总是写好代码后，用vscode打开。那么如何直接跳过使用那些IDE，在nvim里调试代码?

本文涉及的工具（插件）：
  - nvim-dap
  - nvim-dap-ui
  - nvim-dap-virtual-text
  - codelldb

## 安装与配置
### 安装
#### 方法一
你可以直接通过lazyvim中的Lazy Extras安装插件  
lazyvim的[官方文档](https://www.lazyvim.org/extras/dap/core)中给出了通过这种方法安装的默认配置  
可以通过mason安装调试适配器codelldb
#### 方法二
```lua
-- 安装插件
{
  "mfussenegger/nvim-dap", -- dap 核心
  "rcarriga/nvim-dap-ui", -- 调试界面
  "theHamsta/nvim-dap-virtual-text", --  虚拟文本显示变量
}
```
### 配置dap
在你的nvim配置文件中的config目录下创建一个dap.lua文件,并在init.lua中引用它
```lua
-- dap.lua
local dap, dapui = require("dap"), require("dapui")
dap.adapters.codelldb = {
  type = "executable",
  command = "codelldb", -- or if not in $PATH: "/absolute/path/to/codelldb"

  -- On windows you may have to uncomment this:
  -- detached = false,
}
-- 定义C++的调试配置
dap.configurations.cpp = {
  {
    name = "Launch file", -- 配置名称
    type = "codelldb", -- 指定适配器为codelldb
    request = "launch", -- 指定为launch模式
    program = function() -- 可执行文件的路径
      return vim.fn.input("Path to executable: ", vim.fn.getcwd() .. "/", "file")
    end,
    cwd = "${workspaceFolder}", -- 工作目录
    stopOnEntry = false, -- 是否在main入口暂停
  },
}
```
如果使用其他调试适配器可以参考nvim-dap的[官方文档](https://codeberg.org/mfussenegger/nvim-dap/wiki/Debug-Adapter-installation)
### 快捷键
在安装插件的文档下配置快捷键，可参照[官方文档](https://www.lazyvim.org/extras/dap/core)

## 了解工具/插件之间的关系
#### DAP(Debug Adapter Protocol)
- 定位：由微软提出的**标准化调试协议**
- 作用：在编辑器/IDE与调试器之间建立通用通信接口
#### codelldb
- 定位：LLDB调试器的DAP适配器实现
- 作用：
  - 作为“翻译官”在DAP和LLDB之间进行协议转换
  - 将DAP的通用调试指令转换为LLDB特定命令
  - 将LLDB的输出转换为DAP标准格式
#### 工作流程
![关系](/images/nvimdap-codelldb.png)

## 使用
```sh
g++ -g -o a.cpp a
# 首先生成可调式的可执行文件
```
然后再通过设置好的快捷键进行调试

## 附录
参考：
1. https://www.lazyvim.org/extras/dap/core
2. https://codeberg.org/mfussenegger/nvim-dap/wiki/Debug-Adapter-installation
