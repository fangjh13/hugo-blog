---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: '2018-11-03T08:32:22+08:00'
description: 保护 Python 代码
lastmod: '2019-08-14T08:12:45+08:00'
showToc: true
tags: [Python]
title: 使用 Cython 加密 Python 项目
---

最近公司需要将 python 代码部署到端上查了各种加密方法说到底 python 其实是不建议加密部署的，像什么生成`.pyc`其实都是很容易反编译直接运行的，因为它是解释型语言。不像 C 或者 java 可以编译后生成机器码直接部署。还有看到把项目打包成`.exe`文件，在 windows 上运行，由于我们使用 Linux 平台没有尝试，最后选择了使用[Cython](https://cython.org/)这个库来加密(编译成二进制)。

`Cython`其实就是把py 代码编译成 C或者 C++代码来执行，在Linux 上会生成`.so`二进制文件，Windows下为`.pyd`，所以还有一个作用是加速代码的执行效率。但还有一些限制如项目中不能删除`__init__.py`否者包导入会失败。详细可参考[官方文档](https://cython.readthedocs.io/en/latest/src/userguide/limitations.html#cython-limitations)，Cython 还在持续开发中支持 Python3，下面也用Python3演示。

先来做一些准备工作定义编译后的文件夹`build`和一些部署不需要的文件和文件夹，将待编译的`.py`文件加入`ext_modules`列表

```python
cur_dir = os.path.abspath(os.path.dirname(__file__))
setup_file = os.path.split(__file__)[1]
build_dir = os.path.join(cur_dir, 'build')
build_tmp_dir = os.path.join(build_dir, "temp")
# define exclude dirs, these dirs will be deleted
exclude_dirs = ['.git', '__pycache__', 'test', 'logs', 'venv', 'tests']
# defile exclude files, these files will be deleted
exclude_files = ['*.md', '.gitignore', '.python-version', 'requirements.txt', '*.pyc', '*.c']
# these `.py` files will be retained and don't compile to `.so`
ignore_py_files = ['config.py']

ext_modules = []

# get all build files
for path, dirs, files in os.walk(cur_dir, topdown=True):
    dirs[:] = [d for d in dirs if d not in exclude_dirs]
    # touch a new file when __init__.py not exists
    for _dir in dirs:
        init_file = os.path.join(path, _dir, '__init__.py')
        if not os.path.isfile(init_file):
            print('WARNING: create new empty [{}] file.'.format(init_file))
            with open(init_file, 'a') as f:
                pass
    # create target folder
    if not os.path.isdir(build_dir):
        os.mkdir(build_dir)
    # make empty dirs
    for dir_name in dirs:
        dir = os.path.join(path, dir_name)
        target_dir = dir.replace(cur_dir, build_dir)
        os.mkdir(target_dir)
    for file_name in files:
        file = os.path.join(path, file_name)
        if os.path.splitext(file)[1] == '.py':
            if file_name in ignore_py_files:
                # don't compile to .so
                if file_name not in exclude_files:
                    shutil.copy(file, path.replace(cur_dir, build_dir))
            elif file_name in exclude_files:
                # remove it
                pass
            else:
                # add to compile
                if file_name == '__init__.py':
                    #  copy __init__.py resolve package cannot be imported
                    shutil.copy(file, path.replace(cur_dir, build_dir))
                if file_name != setup_file:
                    ext_modules.append(file)
        else:
            _exclude = False
            for pattern in exclude_files:
                if fnmatch.fnmatch(file_name, pattern):
                    _exclude = True
            if not _exclude:
                shutil.copy(file, path.replace(cur_dir, build_dir))

```

我们需要把原来的每个文件夹下`__init__.py`拷贝一份，不然项目中相对导入这些会失效。然后把`ext_modules`列表传给`cythonize`生成distutils Extension objects再传给`setup`函数。

```python
from distutils.core import setup
from Cython.Build import cythonize
from Cython.Distutils import build_ext

setup(
        ext_modules=cythonize(
            ext_modules,
            compiler_directives=dict(
                always_allow_keywords=True,
                c_string_encoding='utf-8',
                language_level=3
            )
        ),
        cmdclass=dict(
            build_ext=build_ext
        ),
        script_args=["build_ext", "-b", build_dir, "-t", build_tmp_dir]
    )

.....
```

需要注意的是传给`setup`时需要加`always_allow_keywords=True`参数否者默认的python 特性关键字参数编译后运行是会报`TypeError: ... takes no keyword arguments`错的，如 fask 应用上。还有在运行的时候指定了`build_dir`是编译后存放的目录，不指定默认存放当前目录下，还有临时目录`build_tmp_dir`稍后可以删除。

完整代码可以参考我[Github](https://github.com/fangjh13/protect_python_code/blob/master/build_it.py)上的代码，拷贝本脚本到项目根目录可以适当修改如哪些不需要放到部署环境的，安装 Cython 后确保每个子文件夹下有`__init__.py`内容为空的也行，不然生成的`.so`会路径不对。运行`python3 build_it.py`会生成一个build文件夹，之后删掉除 build 文件夹所有源文件进入 build 文件夹运行即可（需要启动脚本）。

### Reference

- [https://bucharjan.cz](https://bucharjan.cz/blog/using-cython-to-protect-a-python-codebase.html)
- [https://web.archive.org](https://web.archive.org/web/20160402091909/http://blog.biicode.com/bii-internals-compiling-your-python-application-with-cython/)
- [https://laucyun.com](https://laucyun.com/ea4eae29d21ea116b24c61b5d61f9f64.html)
- [https://www.cnblogs.com](https://www.cnblogs.com/ke10/p/py2so.html)