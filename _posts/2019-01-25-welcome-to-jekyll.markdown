---
layout: post
title:  "Jekyll CheatSheet"
date:   2019-01-25 16:54:13 +0800
categories: jekyll update
---

* 创建站点
  1. [安装ruby开发环境](https://jekyllrb.com/docs/installation/)

  2. 安装jekyll和bundler
  ```bash
  gem install jekyll bundler
  ```
  3. 创建新的站点 _myblog_
  ```bash
  jekyll new myblog
  ```
  4. 进入站点目录运行站点
  ```bash
  $ cd myblog
  $ bundle exec jekyll serve
  ```

* 本地构建启动服务
```bash
jekyll serve
```

* 查找主题目录位置
```bash
bundle show minima
```

* 安装使用新主题
  1. 更改站点Gemfile文件:
  ```bash
  # ./Gemfile
  # This is an example, declare the theme gem you want to use here
  gem "jekyll-theme-cayman"
  ```
  2. 安装主题
  ```bash
  bundle install
  ```
  3. 修改站点_config.yml文件：
  ```bash
  theme: jekyll-theme-cayman
  ```
  4. 构建站点
  ```bash
  jekyll serve
  ```

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
