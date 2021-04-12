# Optuna 简体中文文档

本仓库为 [Optuna](https://optuna.org) 文档的的简体中文翻译。我们通过 [sphinx-intl](http://sphinx-intl.rtfd.io) 来进行翻译。

## Optuna 版本更新后文档更新的步骤

### 文档准备

Clone [Optuna main repo](https://github.com/optuna/optuna) 到本地

```
git clone git@github.com:optuna/optuna.git
```

将 Optuna repo 切换到你准备更新的版本的 commit 或者 tag. 假设我们要更新到 v2.6

```
git checkout tags/v2.6.0
```

Clone 本 repo 到本地, 并创建一个新分支用于更新翻译

```
git clone git@github.com:optuna/doc-optuna-zh-cn.git
git checkout -b translate-v2.6.0
```

### 对比文档变化

#### 依赖变化

检查 Optuna 中的 `setup.py`, 确认其中 `get_extras_require` 下的 `document` 中列出的依赖都已经包含在本 repo 的 `requirements.txt` 中.
如果存在本 repo 的 `requirements.txt` 中没有的依赖, 将它们添加到 `requirements.txt` 中.

#### 版本变化

如果目前的文档翻译是针对 `2.x` 版本的, 你想要更新翻译到 `2.y` 版本, 将本 repo 下 `requirements.txt` 中的 `optuna==2.x` 修改为 `optuna==2.y`.

安装新依赖:

```
pip install -r requirements.txt
```

#### Config 变化

对比 Optuna 中的 `docs/source/config.py` 和本 repo 下的 `source/config.py`. 除了最后两行 (见下一个代码块) 以外, 保持 `source/config.py` 与 `docs/source/config.py` 一致, 因为这些配置是关于翻译文件位置的, 故原文档不包含此部份.

```python
locale_dirs = ['locale/']
gettext_compact = False
```

#### 更新文档源文件

用 Optuna 中的 `docs` 目录下的 `image` 和 `source` 文件夹覆盖本 repo 下的 `image` 和 `source`文件夹.

**注意**

- 请跳过 Optuna 中的 `docs/source/config.py`
- 不要覆盖或者删除 本 repo 中的 `source/locale` 文件夹

### 提取翻译所需文档

```
make gettext
sphinx-intl update -p build/gettext -l zh_CN
```

### 更新翻译

1. 通过 gitdiff 查看所有 `.po` 文件的变动
2. 对存在变动而且带有 `#, fuzzy` 的行进行修改（注意，通常情况下请忽略每个文件开头的 `#, fuzzy`）
3. 查看所有的 `msgstr ""` 并对它们进行翻译 （`msgstr ""` 代表需要翻译但是没完成翻译的句子, 但是多行翻译也会存在首行为 `msgstr ""` 的情况，这时不一定代表翻译没完成）

### 检查

编译文档到 html:

```
make -e SPHINXOPTS="-Dlanguage='zh_CN'" html
```

运行

```
cd build/html && python -m http.server
```

然后在 `http://[::]:8000` 检查翻译是否有错误.
