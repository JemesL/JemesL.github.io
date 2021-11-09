<!--
 * @Description: 
 * @Author: Jemesl
 * @Date: 2021-11-09 11:39:54
-->
# 配置

## 前置依赖安装
- [hexo 官网](https://hexo.io/zh-cn/docs/)
- 安装全局 hexo `npm install hexo-cli -g`
- 安装依赖 `npm i`

## 预览部署
- 生成静态文件 `hexo g` 
- 部署到服务器  `hexo d` 
- 本地预览 `hexo s -w` 

## 新建文章
- 新建文章 `hexo new filename`

## 分支介绍
### hexo
> 仅在此分支下进行操作。
> 例如：新建文章、预览部署等。

### master 分支
> master 分支不做任何操作。
> 在 hexo 分支下进行部署会发布到 `origing/master` 分支, 配置路径再 `./_config.yml`


