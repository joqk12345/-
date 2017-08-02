---
layout: "post"
title: "ipython2(jupyter)"
date: "2017-05-27 08:31"
version:'1.0'
---

# jupyter 项目可视化视图
## 一个 高级别的可视化项目关系图

![A Visual Overview of Projects](https://jupyter.readthedocs.io/en/latest/_images/repos_map.png)

1. jupyter 包关系


 层次|模块|说明
--|---|--
服务层  | jupyterhub  |  多用户登录
服务层  | tmpnb  |  多用户不登录进入
服务层  | nbgrader  |  笔记本作为一个分配,创建和打分
服务层  | nbviewer  |  HTML,视图
应用层  | jupyter_console  | jupyter 终端应用
应用层  | qtconsole  | jupyter QT控制台
应用层  | notebook  | jupyter notebook应用
应用层  | nbconvert  | jupyter 格式转换层
API层  |  jupyter_core |jupyter:命令/配置文件处理/文件系统定位
API层  |  jupyter_client |消息传递
API层  |  nbformat |ipynb file 加载/保存/格式化版本/移除/
内核层  |   ipykernel| 内核沟通机制
内核层  |   ipywidgets| 交互组件(小器具/装饰品)
内核层  |   ipython| python执行器,magics 和ipython终端接口  
工具层  |   traitlets{特性包}| 配置系统和小器具都依赖于此层  
