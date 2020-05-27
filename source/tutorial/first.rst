.. _firstopt:

第一个优化例子
==================


二次函数的例子
--------------------------

Optuna 通常用于优化超参数，但是为了举例说明，我们将直接在 IPython shell 中优化一个二次函数。

.. code-block:: python

    import optuna

目标 (objective) 函数便是待优化的函数。

.. code-block:: python

    def objective(trial):
        x = trial.suggest_uniform('x', -10, 10)
        return (x - 2) ** 2

该函数的返回值是 :math:`(x - 2)^2`. 我们的目标是找到一个 ``x``，使 ``objective`` 函数的输出最小。这被称为 "optimization" (优化)。 在优化过程中，Optuna 反复调用目标函数，在不同的 ``x`` 下对其进行求值。

一个 :class:`~optuna.trial.Trial` 对应着目标函数的单次执行。在每次调用该函数的时候，它都被内部实例化一次。

而 `suggest` API (例如 :func:`~optuna.trial.Trial.suggest_uniform`) 在目标函数内部被调用。它被用于获取单个 trial 的参数。在上面的例子中，:func:`~optuna.trial.Trial.suggest_uniform` 在给定的范围（``-10`` 到 ``10``）内均匀地选择参数。

为了开始优化过程，我们将创建一个 study 对象，并将目标函数传递给它的一个方法 :func:`~optuna.study.Study.optimize`：

.. code-block:: python

    study = optuna.create_study()
    study.optimize(objective, n_trials=100)

输出:

.. code-block:: console

    [I 2020-04-08 10:42:09,028] Finished trial#0 with value: 25.77382032395108 with parameters: {'x': 7.076792326257898}. Best is trial#0 with value: 25.77382032395108.
    [I 2020-04-08 10:42:09,064] Finished trial#1 with value: 1.5189812248635903 with parameters: {'x': 0.7675304365366298}. Best is trial#1 with value: 1.5189812248635903.
    [I 2020-04-08 10:42:09,106] Finished trial#2 with value: 34.4074691838153 with parameters: {'x': -3.865788027521562}. Best is trial#1 with value: 1.5189812248635903.
    [I 2020-04-08 10:42:09,145] Finished trial#3 with value: 3.3601305753722657 with parameters: {'x': 3.8330658949891205}. Best is trial#1 with value: 1.5189812248635903.
    [I 2020-04-08 10:42:09,185] Finished trial#4 with value: 61.16797535698886 with parameters: {'x': -5.820995803412048}. Best is trial#1 with value: 1.5189812248635903.
    [I 2020-04-08 10:42:09,228] Finished trial#5 with value: 90.08665552769618 with parameters: {'x': -7.491399028999686}. Best is trial#1 with value: 1.5189812248635903.
    [I 2020-04-08 10:42:09,274] Finished trial#6 with value: 25.254236332163032 with parameters: {'x': 7.025359323686519}. Best is trial#1 with value: 1.5189812248635903.
    ...
    [I 2020-04-08 10:42:14,237] Finished trial#99 with value: 0.5227007740782738 with parameters: {'x': 2.7229804797352926}. Best is trial#67 with value: 2.916284393762304e-06.

最佳参数可以通过如下方式获得：

.. code-block:: python

    study.best_params

输出:

.. code-block:: console

    {'x': 2.001707713205946}

可以看到，Optuna 找到的最佳 ``x``的值 是 ``2.001707713205946``. 这非常靠近实际的最优值 ``2``.

.. note::
    当Optuna被用于机器学习中的超参数搜索时，目标函数通常是对应模型的损失 (loss) 或者准确度 (accuracy).
    When used to search for hyper-parameters in machine learning, usually the objective function would return the loss or accuracy of the model.

Study 对象
------------

下面是几个常用术语：

* **Trial**: 目标函数的单次调用
* **Study**: 一次优化过程，包含一系列的 rials.
* **Parameter**: 待优化的参数，比如上面例子中的 ``x``.

在 Optuna 中，我们用 study 对象来管理优化过程。 :func:`~optuna.study.create_study` 方法会返回一个 study 对象。该对象包含若干有用的属性，可以用于分析优化结果。

获得最佳参数:

.. code-block:: python

    study.best_params

输出:

.. code-block:: console

    {'x': 2.001707713205946}

获得最佳目标函数值:

.. code-block:: python

    study.best_value

输出:

.. code-block:: console

    2.916284393762304e-06

获得最佳 trial:

.. code-block:: python

    study.best_trial

输出:

.. code-block:: console

    FrozenTrial(number=67, value=2.916284393762304e-06, datetime_start=datetime.datetime(2020, 4, 8, 10, 42, 12, 595884), datetime_complete=datetime.datetime(2020, 4, 8, 10, 42, 12, 639969), params={'x': 2.001707713205946}, distributions={'x': UniformDistribution(high=10, low=-10)}, user_attrs={}, system_attrs={}, intermediate_values={}, trial_id=67, state=TrialState.COMPLETE)

获得所有 trials:

.. code-block:: python

    study.trials

输出:

.. code-block:: console

    [FrozenTrial(number=0, value=25.77382032395108, datetime_start=datetime.datetime(2020, 4, 8, 10, 42, 8, 987277), datetime_complete=datetime.datetime(2020, 4, 8, 10, 42, 9, 27959), params={'x': 7.076792326257898}, distributions={'x': UniformDistribution(high=10, low=-10)}, user_attrs={}, system_attrs={}, intermediate_values={}, trial_id=0, state=TrialState.COMPLETE),
     ...
     user_attrs={}, system_attrs={}, intermediate_values={}, trial_id=99, state=TrialState.COMPLETE)]

获得 trial 的数量:

.. code-block:: python

    len(study.trials)

输出:

.. code-block:: console

    100

（在优化结束后）通过再次执行 :func:`~optuna.study.Study.optimize`，我们可以继续优化过程。

.. code-block:: python

    study.optimize(objective, n_trials=100)

获得更新（再次优化后）的 trial 数量：

.. code-block:: python

    len(study.trials)

输出:

.. code-block:: console

    200
