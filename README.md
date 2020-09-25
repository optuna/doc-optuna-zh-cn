# Optuna 简体中文文档

本仓库为 [Optuna](https://optuna.org) 文档的的简体中文翻译。我们通过 [sphinx-intl](http://sphinx-intl.rtfd.io) 来进行翻译。



## Optuna 版本更新后文档更新的步骤

克隆仓库到本地，并创建一个新分支。

安装依赖（这一步推荐在虚拟环境下进行）：

`pip install -r requirements.txt` 

生成当前版本的翻译文件：

`sphinx-intl update -p build/locale -l zh`

在 `requirements.txt` 中修改Optuna到想要的版本号。

安装新版的 Optuna：

`pip install -r requirements.txt`


克隆 Optuna 的[主仓库](https://github.com/optuna/optuna) 到本地。

将 Optuna 的主仓库reset 到你准备更新的版本发布的commit。


将Optuna 主仓库内的 `doc/source` 目录下的除 `config.py` 之外的所有文件拷贝到此仓库的 `source` 目录下，覆盖当前文件。

更新翻译文件:

`make gettext`

`sphinx-intl update -p build/gettext -l zh`

更新翻译

1. 通过gitdiff查看所有 `.po` 文件的变动
2. 对存在变动而且带有 `#, fuzzy` 的行进行修改（注意，通常情况下请忽略每个文件开头的 `#, fuzzy`）

生成更新后的文档的静态网页，检查是否有错：

`make html` 

确认无误后，在当前分支提交修改并创建PR，等待更新被merge到主分支中。

