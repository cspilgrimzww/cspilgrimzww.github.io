title: Pycharm中创建的flask项目，不能关联Jinja2模版的问题解决
date: 2015-5-18 03:19:11
tags: Pycharm Jinja2
---
在pycharm中新建一个flask项目，这时候pycharm已经定义好了template目录，并且你会发现在render_template的时候，按住ctrl键是可以跳转到模版页的，而且模版页中的Jinja标签，是有自动感应的。
但我这次创建项目的时候，没有选择flask类型，而是选择了空项目，所以在后面，搞死都不能支持Jinja标签，跳转就更别谈了，后来发现， 在项目的根目录的.idea目录中，有个xxx.iml文件（xxx是项目名称），打开这个iml文件，在现有的component标签的同级，添加如下 代码，即可解决上述问题：
```
<component name="TemplatesService">
<option name="TEMPLATE_CONFIGURATION" value="Jinja2" />
<option name="TEMPLATE_FOLDERS">
<list>
<option value="$MODULE_DIR$/templates" />
</list>
</option>
</component>
```