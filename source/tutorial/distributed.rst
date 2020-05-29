.. _distributed:

分布式优化
========================

Optuna的分布式优化中没有复杂的设置，你只需让不同的节点/进程共享一个相同的 study 名。

首先，使用 ``optuna create-study`` 命令 (如果是在Python脚本中的化，就用 :func:`optuna.create_study`) 创建一个共享的 study.

.. code-block:: bash

    $ optuna create-study --study-name "distributed-example" --storage "sqlite:///example.db"
    [I 2018-10-31 18:21:57,885] A new study created with name: distributed-example

然后写一个包含如下代码的脚本，``foo.py``, 来进行优化。

.. code-block:: python

    import optuna

    def objective(trial):
        x = trial.suggest_uniform('x', -10, 10)
        return (x - 2) ** 2

    if __name__ == '__main__':
        study = optuna.load_study(study_name='distributed-example', storage='sqlite:///example.db')
        study.optimize(objective, n_trials=100)

最后，从不同的进程中分别运行这个 share study。
比如说，在一个终端中运行 ``Process 1``，在另一个终端中运行 ``Process 2``.
这些进程基于这个共享 study 的 trial 历史记录来获取参数建议 (parameter suggestion).

进程 1:

.. code-block:: bash

    $ python foo.py
    [I 2018-10-31 18:46:44,308] Finished a trial resulted in value: 1.1097007755908204. Current best value is 0.00020881104123229936 with parameters: {'x': 2.014450295541348}.
    [I 2018-10-31 18:46:44,361] Finished a trial resulted in value: 0.5186699439824186. Current best value is 0.00020881104123229936 with parameters: {'x': 2.014450295541348}.
    ...

进程 2 （使用和进程 1 相同的命令）:

.. code-block:: bash

    $ python foo.py
    [I 2018-10-31 18:47:02,912] Finished a trial resulted in value: 29.821448668796563. Current best value is 0.00020881104123229936 with parameters: {'x': 2.014450295541348}.
    [I 2018-10-31 18:47:02,968] Finished a trial resulted in value: 0.7962498978463782. Current best value is 0.00020881104123229936 with parameters: {'x': 2.014450295541348}.
    ...

.. note::
    我们不推荐在大型的分布式优化中使用 SQLite，因为这可能导致性能问题。在这种情况下，请考虑使用其他数据库，比如 PostgreSQL 或 MySQL.

.. note::
    在运行分布式优化时，请不要将 SQLite 数据库文件放在 NFS (Network File System) 文件系统中。具体原因见 : https://www.sqlite.org/faq.html#q5
