.. _cli:

命令航界面
======================

.. csv-table::
   :header: 命令, 功能描述
   :widths: 20, 40

    create-study, 创建一个新的 study.
    delete-study, 删除特定 study.
    dashboard, 启动 web 面板 (beta).
    storage upgrade, 升级存储 schema.
    studies, 输出 study 列表。
    study optimize, 针对一个 study 开始优化过程。
    study set-user-attr, 设置 study 的用户属性。

Optuna 提供的命令行界面包括上表中的所有命令。

如果你不使用 Ipython shell, 而是写 Python 脚本文件的话，编写像下面这样的脚本是完全可以的：

.. code-block:: python

    import optuna


    def objective(trial):
        x = trial.suggest_uniform('x', -10, 10)
        return (x - 2) ** 2


    if __name__ == '__main__':
        study = optuna.create_study()
        study.optimize(objective, n_trials=100)
        print('Best value: {} (params: {})\n'.format(study.best_value, study.best_params))

然而，通过 ``optuna`` 命令，我们可以少写一些上面这样的模版代码。
假设有一个 ``foo.py`` 文件，它只包含如下代码：

.. code-block:: python

    def objective(trial):
        x = trial.suggest_uniform('x', -10, 10)
        return (x - 2) ** 2

即使在这种情况下，我们也可以利用如下命令启动优化过程（请暂时忽略 ``--storage sqlite:///example.db`` 的作用, 关于它的具体描述见 :ref:`rdb`）。

.. code-block:: bash

    $ cat foo.py
    def objective(trial):
        x = trial.suggest_uniform('x', -10, 10)
        return (x - 2) ** 2

    $ STUDY_NAME=`optuna create-study --storage sqlite:///example.db`
    $ optuna study optimize foo.py objective --n-trials=100 --storage sqlite:///example.db --study $STUDY_NAME
    [I 2018-05-09 10:40:25,196] Finished a trial resulted in value: 54.353767789264026. Current best value is 54.353767789264026 with parameters: {'x': -5.372500782588228}.
    [I 2018-05-09 10:40:25,197] Finished a trial resulted in value: 15.784266965526376. Current best value is 15.784266965526376 with parameters: {'x': 5.972941852774387}.
    ...
    [I 2018-05-09 10:40:26,204] Finished a trial resulted in value: 14.704254135013741. Current best value is 2.280758099793617e-06 with parameters: {'x': 1.9984897821018828}.

请注意，``foo.py`` 中只包含目标函数的定义。
通过向 ``optuna study optimize`` 命令传递该文件名和对应目标函数的方法名，我们就可以启动优化过程。