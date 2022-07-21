## Jekyll

Jekyll 是一个基于文件的内容管理系统（CMS）。它使用 Ruby 编写，通过 Markdown 和 Liquid 模板生成内容。



## Jekyll 目录结构

| 文件 / 目录                                               | 描述                                                         |
| --------------------------------------------------------- | ------------------------------------------------------------ |
| `_config.yml`                                             | 存储的是 [配置](https://www.jekyll.com.cn/docs/configuration/) 信息。其中许多 参数是可以在命令行中指定的，但是 在此文件中进行设置会更容易些，并且你不必记住它们。 |
| `_drafts`                                                 | 草稿（Drafts）是未发布的文章（posts）。这些文章的文件名中没有包含 日期： `title.MARKUP`。了解如何 [使用草稿功能](https://www.jekyll.com.cn/docs/posts/#drafts)。 |
| `_includes`                                               | These are the partials that can be mixed and matched by your layouts and posts to facilitate reuse. The liquid tag `{% include file.ext %}` can be used to include the partial in `_includes/file.ext`. |
| `_layouts`                                                | 这里存放的是These are the templates that wrap posts. Layouts are chosen on a post-by-post basis in the [front matter](https://www.jekyll.com.cn/docs/front-matter/), which is described in the next section. The liquid tag `{{ content }}` is used to inject content into the web page. |
| `_posts`                                                  | Your dynamic content, so to speak. The naming convention of these files is important, and must follow the format: `YEAR-MONTH-DAY-title.MARKUP`. The [permalinks](https://www.jekyll.com.cn/docs/permalinks/) can be customized for each post, but the date and markup language are determined solely by the file name. |
| `_data`                                                   | 格式化的站点数据应当放在此目录下。Jekyll 将自动加载此目录下的所有数据文件（支持 `.yml`、 `.yaml`、`.json`、`.csv` 或 `.tsv` 格式和扩展名）， 然后就可以通过 `site.data` 来访问了。假定在 此目录下有一个文件名为 `members.yml` 的文件，那么你可以通过 `site.data.members` 变量来访问该文件中的内容。 |
| `_sass`                                                   | 这些事可以导入（import）到 `main.scss` 文件中的 sass 代码片段， `main.scss` 文件最终经过处理之后形成一个独立的样式文件 `main.css` 并被用于整个站点。 |
| `_site`                                                   | 在 Jekyll 转换完所有的文件之后，将在此目录下放置生成的站点（默认情况下）。 最好将此目录添加到 `.gitignore` 文件中。 |
| `.jekyll-metadata`                                        | This helps Jekyll keep track of which files have not been modified since the site was last built, and which files will need to be regenerated on the next build. This file will not be included in the generated site. It’s probably a good idea to add this to your `.gitignore` file. |
| `index.html` or `index.md` and other HTML, Markdown files | Provided that the file has a [front matter](https://www.jekyll.com.cn/docs/front-matter/) section, it will be transformed by Jekyll. The same will happen for any `.html`, `.markdown`, `.md`, or `.textile` file in your site’s root directory or directories not listed above. |
| 其它文件/目录                                             | 除了上面列出的目录和文件外，其它所有目录和文件（例如 `css` 和 `images` 目录， `favicon.ico` 等文件）都将一一复制到 生成的站点中。已经有大量 [站点 在使用 Jekyll 了](https://www.jekyll.com.cn/showcase/)，你可以参考这些站点以了解如何运用这些文件和目录。 |



### Jekyll Github 同步

```bash
$ git add .
$ git commit -m "statement"   //此处statement填写此次提交修改的内容，作为日后查阅
$ git push origin master
```



## RubyGems

RubyGems 是一个优秀的包管理系统

### RubyGems-China 源

[https://gems.ruby-china.com/](https://gems.ruby-china.com/)





