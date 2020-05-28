.. _rdb:

用 RDB 后端保存/恢复 Study
==========================================

RDB后端可以实现持久化实验（即保存和恢复 study）以及访问 study 的历史记录。
此外，我们还可以利用这个特点来进行多节点优化。具体描述见 :ref:`distributed`.

在本部分中，我们将尝试一个在本地环境下运行SQLite DB的简单例子。

.. note::
    通过设置 DB 的 storage URL 参数，你也可以使用其他的 RDB 后端，比如 PostgreSQL 或者 MySQL.
    设置URL的方式参见 `SQLAlchemy 的文档 <https://docs.sqlalchemy.org/en/latest/core/engines.html#database-urls>`_.


新建 Study
-------------

通过调用函数 :func:`~optuna.study.create_study`，我们可以创建一个持久化的 study.
创建新 study 会自动初始化一个 SQLite 文件 ``example.db``.

.. code-block:: python

    import optuna
    study_name = 'example-study'  # 一个study 的唯一标识符。
    study = optuna.create_study(study_name=study_name, storage='sqlite:///example.db')

为了运行一个 study, 我们需要将目标函数传入 :func:`~optuna.study.Study.optimize` 方法并调用它。

.. code-block:: python

    def objective(trial):
        x = trial.suggest_uniform('x', -10, 10)
        return (x - 2) ** 2

    study.optimize(objective, n_trials=3)

恢复 Study
------------

为了恢复 study, 首先需要初始化一个 :class:`~optuna.study.Study` 对象， 并将该study 的名字 ``example-study`` 和 DB URL参数 ``sqlite:///example.db`` 传入其中。

.. code-block:: python

    study = optuna.create_study(study_name='example-study', storage='sqlite:///example.db', load_if_exists=True)
    study.optimize(objective, n_trials=3)

实验历史记录
--------------------

我们可以通过 :class:`~optuna.study.Study` 类来获得 study 和对应 trials的历史记录。
比如，下面的语句可以获取 ``example-study`` 的所有 trials.

.. code-block:: python

    import optuna
    study = optuna.create_study(study_name='example-study', storage='sqlite:///example.db', load_if_exists=True)
    df = study.trials_dataframe(attrs=('number', 'value', 'params', 'state'))

:func:`~optuna.study.Study.trials_dataframe` 方法会返回一个如下的 pandas dataframe：

.. code-block:: python

    print(df)

输出:

.. code-block:: console

            number       value  params_x     state
         0       0   25.301959 -3.030105  COMPLETE
         1       1    1.406223  0.814157  COMPLETE
         2       2   44.010366 -4.634031  COMPLETE
         3       3   55.872181  9.474770  COMPLETE
         4       4  113.039223 -8.631991  COMPLETE
         5       5   57.319570  9.570969  COMPLETE

:class:`~optuna.study.Study` 对象也有一些其他属性，比如 :attr:`~optuna.study.Study.trials`, :attr:`~optuna.study.Study.best_value` 和 :attr:`~optuna.study.Study.best_params` (见 :ref:`firstopt`).

.. code-block:: python

    study.best_params  # 获取目标函数的最佳参数。
    study.best_value  # 获取目标函数的最佳值。
    study.best_trial  # 获得最佳 trial 的相关信息。
    study.trials  # 获得所有 trial 的信息。
