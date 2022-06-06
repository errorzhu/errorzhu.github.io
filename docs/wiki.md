# 搭建wiki

# 方案

docsify + github pages

# 流程

1. 安装docsify

```
npm i docsify-cli -g
```

2. 初始化文档

```
docsify init ./docs
```

3. 修改目录结构与侧边栏

```
docs/
  README.md               #首页
  _sidebar.md             #侧边栏
  index.html              #网站首页html
  wiki.md                 #本篇文档
```

- 3.1 修改_sidebar.md

```
<!-- docs/_sidebar.md -->

* [首页](README.md)
* [搭建个人wiki](wiki.md)
```

- 3.2 修改首页README.md

```
# wiki
[搭建个人wiki](wiki.md)
```

- 3.3 修改index.html

```
<body>
  <div id="app"></div>
  <script>
    window.$docsify = {
      loadSidebar: true,
      name: '',
      repo: ''
    }
  </script>
  <!-- Docsify v4 -->
  <script src="//cdn.jsdelivr.net/npm/docsify@4"></script>
</body>
```

## 验证效果

```
docsify serve docs
通过http://localhost:3000/ 查看效果
```

![演示](images\wiki\wiki.png)

## 创建github pages

参考 https://docs.github.com/cn/pages/getting-started-with-github-pages/creating-a-github-pages-site

## 将docs文件夹上传至github