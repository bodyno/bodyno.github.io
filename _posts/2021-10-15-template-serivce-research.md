---
layout: post
title: "模板语言调研"
date: 2021-10-15
tags: [template service frontend]
categories: cd
---

### CODING 插件系统参数界面自动生成原理

在流水线中选择插件时会调用接口获取插件参数`variables` 数组，结果如下：
```
help: "请使用 制品名称:Tag 定义镜像，确保本步骤执行前构建环境中存在此镜像。"
label: "推送镜像"
name: "image"
placeholder: "my-image:1.1.0"
required: true
type: "text"
```
前端遍历`type` 字段，根据预设好的模板生成相应的`input` 界面
![图片](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/391de318ac3840678546f29a0c01c089~tplv-k3u1fbpfcp-watermark.image?)

插件参数`variables` 来源于编写参数时填写的[描述文件](https://help.coding.net/docs/ci/plugins/customize/format.html)

### KubeVela 从定义中生成表单原理

在KubeVela 中定义的CRD，[链接](https://kubevela.io/docs/platform-engineers/openapi-v3-json-schema)，会自动生成OpenAPI v3 JSON schema，并存在ConfigMap 中

渲染表单时，取出JSON定义，使用前端开源组件`react-jsonschema-form` 直接生成相应UI即可

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cf398a7d6a524c17a09ec8f7ea882af2~tplv-k3u1fbpfcp-watermark.image?)

CUE 的使用其实就是定义模板，通过CUE 语法的灵活强大性，把复杂的yaml封装起来，通过parameter 定义参数（如下），生成最终需要的文件
```
parameter: {
    image: string
    replicas: int
}
```

### Jsonnet、GCL、HCL
Jsonnet 语法更简单，是JSON 语法的扩展，可以输出为yaml

CUE 更关注语法的校验，Jsonnet 更关注数据模版（样板代码移除），CUE最大的特点是有`类型`

GCL 和 HCL 类似，抽象层级不是特别高
