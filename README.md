# hexo
该博客通过hexo搭建，采用的是next主题。

博客域名为：http://www.godliusheng.com

博客github pages：https://github.com/liusir0215/liusir0215.github.io

通过git subtree获取next主题 
```shell
git subtree pull --prefix=themes/next git@github.com:liusir0215/hexo-theme-next.git master --squash
```

#### 新增博客
```shell
hexo new post xxx
```

#### 新增布局（分类和标签也是使用这个）
```shell
hexo new page xxx
```

#### 新增草稿
```shell
hexo new draft xxx
```

#### 草稿正式发布
```shell
hexo publish draft xxx
```

#### 列出现有文章
```shell
hexo list [type]
```

#### 生成页面
```shell
npm run build
```

#### 运行服务
```shell
npm run server
```

#### 发布上线
```shell
npm run deploy
```

#### 清除已生成的文件并清除缓存文件 (db.json) 
```shell
npm run clean
```
