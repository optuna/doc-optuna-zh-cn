.. _configurations:

高级配置
=======================

定义参数空间
-------------------------

Optuna 支持五种类型的参数
Optuna supports five kinds of parameters.

.. code-block:: python

    def objective(trial):
        # 类别参数
        optimizer = trial.suggest_categorical('optimizer', ['MomentumSGD', 'Adam'])

        # 整形参数
        num_layers = trial.suggest_int('num_layers', 1, 3)

        # 均匀分布参数
        dropout_rate = trial.suggest_uniform('dropout_rate', 0.0, 1.0)

        # 对数均匀分布参数
        learning_rate = trial.suggest_loguniform('learning_rate', 1e-5, 1e-2)

        # 离散均匀分布参数
        drop_path_rate = trial.suggest_discrete_uniform('drop_path_rate', 0.0, 1.0, 0.1)

        ...

分支 (Branches) 与 循环 (Loops)
-----------------------------------

根据不同的参数值，你可以选择使用分支或者循环。

.. code-block:: python

    def objective(trial):
        classifier_name = trial.suggest_categorical('classifier', ['SVC', 'RandomForest'])
        if classifier_name == 'SVC':
            svc_c = trial.suggest_loguniform('svc_c', 1e-10, 1e10)
            classifier_obj = sklearn.svm.SVC(C=svc_c)
        else:
            rf_max_depth = int(trial.suggest_loguniform('rf_max_depth', 2, 32))
            classifier_obj = sklearn.ensemble.RandomForestClassifier(max_depth=rf_max_depth)

        ...

.. code-block:: python

    def create_model(trial):
        n_layers = trial.suggest_int('n_layers', 1, 3)

        layers = []
        for i in range(n_layers):
            n_units = int(trial.suggest_loguniform('n_units_l{}'.format(i), 4, 128))
            layers.append(L.Linear(None, n_units))
            layers.append(F.relu)
        layers.append(L.Linear(None, 10))

        return chainer.Sequential(*layers)

更多例子见 `examples <https://github.com/optuna/optuna/tree/master/examples>`_.


参数个数的注意事项
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

随着参数个数的增长，优化的难度约成指数增长。也就是说，当你增加参数的个数的时候，优化所需要的 trial 个数会呈指数增长。因此我们推荐不要增加不必要的参数。


`Study.optimize` 的参数
--------------------------------

:func:`~optuna.study.Study.optimize` (还有命令 ``optuna study optimize``) 有着数个有用的参数，比如``timeout``. 具体细节见 :func:`~optuna.study.Study.optimize` 的API参考资料。

**供參考**: 如果既没有给出 ``n_trials`` 也没有给出 ``timeout`` 参数的话，优化过程将一直持续下去，直到接收到一个诸如 Ctrl+C 或 SIGTERM 的终止信号。
这在难以估算优化目标函数所需的计算成本的情况时是有用的。