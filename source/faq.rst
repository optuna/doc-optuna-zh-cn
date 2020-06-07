FAQ
===

.. contents::
    :local:

某某库可以和 Optuna 配合使用吗？（某某是你最爱的机器学习库）
--------------------------------------------------------------

Optuna 和绝大多数ML库兼容，并且很容易同他们配合使用。
参见 `examples <https://github.com/optuna/optuna/tree/master/examples>`_.


.. _objective-func-additional-args:

如何定义带有独有参数的目标函数？
----------------------------------------------------------

有两种方法可以实现这类函数。

首先，如下的可调用的 objective 类具有这个功能：

.. code-block:: python

    import optuna

    class Objective(object):
        def __init__(self, min_x, max_x):
            # Hold this implementation specific arguments as the fields of the class.
            self.min_x = min_x
            self.max_x = max_x

        def __call__(self, trial):
            # Calculate an objective value by using the extra arguments.
            x = trial.suggest_uniform('x', self.min_x, self.max_x)
            return (x - 2) ** 2

    # 通过使用一个 `Objective` 实例来执行优化过程。
    study = optuna.create_study()
    study.optimize(Objective(-100, 100), n_trials=100)


其次，你可以用 ``lambda`` 或者 ``functools.partial`` 来创建带有额外参数的函数（闭包）。 
下面是一个使用了 ``lambda`` 的例子：

.. code-block:: python

    import optuna

    # 带有三个参数的目标函数。
    def objective(trial, min_x, max_x):
        x = trial.suggest_uniform('x', min_x, max_x)
        return (x - 2) ** 2

    # 额外参数。
    min_x = -100
    max_x = 100

    # 通过 `lambda` 将目标函数包起来用于优化过程。
    study = optuna.create_study()
    study.optimize(lambda trial: objective(trial, min_x, max_x), n_trials=100)

其他例子参见 `sklearn_addtitional_args.py <https://github.com/optuna/optuna/blob/master/examples/sklearn_additional_args.py>`_ .


没有远程 RDB 的情况下可以使用 Optuna 吗？
--------------------------------------------

可以。

在最简单的情况下，Optuna 使用内存 (in-memory) 存储：

.. code-block:: python

    study = optuna.create_study()
    study.optimize(objective)

如果你想能保存和恢复 study 的化，将 SQLite 作为本地存储也是很方便的：

.. code-block:: python

    study = optuna.create_study(study_name='foo_study', storage='sqlite:///example.db')
    study.optimize(objective)  # The state of `study` will be persisted to the local SQLite file.

更多细节请参考 :ref:`rdb`.


如何保存和恢复 study？
----------------------------------------------------

有两种方法可以将 study 持久化。具体采用哪种取决于你是使用内存存储 (in-memory) 还是远程数据库存储 (RDB).
通过 ``pickle`` 或者 ``joblib``, 采用了内存存储的 study 可以和普通的 Python 对象一样被存储和加载。
比如用 ``joblib`` 的话：

.. code-block:: python

    study = optuna.create_study()
    joblib.dump(study, 'study.pkl')

恢复 study:

.. code-block:: python

    study = joblib.load('study.pkl')
    print('Best trial until now:')
    print(' Value: ', study.best_trial.value)
    print(' Params: ')
    for key, value in study.best_trial.params.items():
        print(f'    {key}: {value}')

如果你用的是 RDB, 具体细节请参考 :ref:`rdb`.

如何禁用 Optuna 的日志信息？
---------------------------------------

默认情况下，Optuna 打印处于 ``optuna.logging.INFO`` 层级的日志信息。
通过设置  :func:`optuna.logging.set_verbosity`, 你可以改变这个层级。

比如，下面的代码可以终止打印每一个trial的结果：

.. code-block:: python

    optuna.logging.set_verbosity(optuna.logging.WARNING)

    study = optuna.create_study()
    study.optimize(objective)
    # Logs like '[I 2018-12-05 11:41:42,324] Finished a trial resulted in value:...' are disabled.


更多的细节请参考  :class:`optuna.logging`.


如何在目标函数中保存训练好的机器学习模型？
-------------------------------------------

Optuna 会保存超参数和对应的目标函数值到存储对象中，但是它不会存储诸如机器学习模型或者网络权重这样的中间对象。
要保存模型或者权重的话，请利用你正在使用的机器学习库提供的对应功能。

在保存模型的时候，我们推荐将 :obj:`optuna.trial.Trial.number` 一同存储。这样易于之后确认对应的 trial.
比如，你可以用以下方式在目标函数中保存训练好的 SVM 模型：

.. code-block:: python

    def objective(trial):
        svc_c = trial.suggest_loguniform('svc_c', 1e-10, 1e10)
        clf = sklearn.svm.SVC(C=svc_c)
        clf.fit(X_train, y_train)

        # 将训练好的模型保存到文件。
        with open('{}.pickle'.format(trial.number), 'wb') as fout:
            pickle.dump(clf, fout)
        return 1.0 - accuracy_score(y_valid, clf.predict(X_valid))


    study = optuna.create_study()
    study.optimize(objective, n_trials=100)

    # 加载最佳模型。
    with open('{}.pickle'.format(study.best_trial.number), 'rb') as fin:
        best_clf = pickle.load(fin)
    print(accuracy_score(y_valid, best_clf.predict(X_valid)))


如何获得可复现的优化结果？
---------------------------------------------------

要让 Optuna 生成的参数可复现的话，你可以通过设置 :class:`~optuna.samplers.RandomSampler` 或者 :class:`~optuna.samplers.TPESampler` 中的参数 ``seed`` 来指定一个固定的随机数种子：

.. code-block:: python

    sampler = TPESampler(seed=10)  # Make the sampler behave in a deterministic way.
    study = optuna.create_study(sampler=sampler)
    study.optimize(objective)

但是这么做的话有两个风险。

首先，如果一个 study 的优化过程本身是分布式的或者并行的，那么这个过程中存在着固有的不确定性。
因此，在这种情况下我们很难复现出同样的结果。
如果你想复现结果的话，我们建议用顺序执行的方式来优化你的 study.

其次，如果你的目标函数的行为本身就是不确定的（也就是说，即使送入同样的参数，其返回值也不是唯一的），那么你就无法复现这个优化过程。
要解决这个问题的话，请设置一个选项（比如随机数种子）来让你的优化目标的行为变成确定性的，前提是你用的机器学习库支持这一功能。

Trial 抛出的异常是如何处理的？
---------------------------------------

那些抛出异常却没有对应的捕获机制的 trial 会被视作失败的 trial, 也就是处于 :obj:`~optuna.trial.TrialState.FAIL` 状态的 trial.

在默认情况下，除了目标函数中抛出的 :class:`~optuna.exceptions.TrialPruned`, 其他所有异常都会被传回给调用函数 :func:`~optuna.study.Study.optimize`.
换句话说，当此类异常被抛出时，对应的 study 就会被终止。
但有时候我们希望能用剩余的 trial 将该 study 继续下去。
要这么做的话，你得通过 :func:`~optuna.study.Study.optimize` 函数中的 ``catch`` 参数来指定要捕获的异常类型。
这样，此类异常就会在 study 内部被捕获，而不会继续向外层传递。

你可以在日志信息里找到失败的 trial.

.. code-block:: sh

    [W 2018-12-07 16:38:36,889] Setting status of trial#0 as TrialState.FAIL because of \
    the following error: ValueError('A sample error in objective.')

也可以用如下方式来找出失败的 trial:

.. code-block:: python

    study.trials_dataframe()

.. csv-table::

    number,state,value,...,params,system_attrs
    0,TrialState.FAIL,,...,0,Setting status of trial#0 as TrialState.FAIL because of the following error: ValueError('A test error in objective.')
    1,TrialState.COMPLETE,1269,...,1,

.. seealso::

    :func:`~optuna.study.Study.optimize` 中的参数 ``catch``.


Trial 返回的 NaN 是如何处理的？
----------------------------------------

返回 NaN 的 trial 被视为失败的 trial, 但是它们并不会导致 study 被终止。

这些 trial 在日志里长这样：

.. code-block:: sh

    [W 2018-12-07 16:41:59,000] Setting status of trial#2 as TrialState.FAIL because the \
    objective function returned nan.


动态地改变搜索空间会导致怎样的结果？
-------------------------------------------

由于参数空间只在调用 suggestion API (比如 :func:`~optuna.trial.Trial.suggest_uniform` 和 :func:`~optuna.trial.Trial.suggest_int`) 的时候才会被确定，
因此，即使在同一个 study 中，我们也可以通过在不同 trial 里对不同的参数空间进行采样来实现对搜索空间的改变。

参数空间改变之后的行为是由 sampler 来决定的。

.. note::
    关于 TPE sampler 的 讨论：https://github.com/optuna/optuna/issues/822


如何用两块 GPU 同时对两个 trial 进行求值？
----------------------------------------------------------------

如果你的优化目标支持 GPU (CUDA) 加速，你又想指定优化所用的 GPU 的话，
设置 ``CUDA_VISIBLE_DEVICES`` 环境变量可能是实现这一目标最轻松的方式了：

.. code-block:: bash

    # 打开一个命令行窗口。
    #
    # 指定使用第一块 GPU 来跑优化。
    $ export CUDA_VISIBLE_DEVICES=0
    $ optuna study optimize foo.py objective --study foo --storage sqlite:///example.db

    # 打开一个新的命令行窗口。
    #
    # 指定第二块 GPU 用于跑新的优化。
    $ export CUDA_VISIBLE_DEVICES=1
    $ optuna study optimize bar.py objective --study bar --storage sqlite:///example.db

更多细节见 `CUDA C Programming Guide <https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#env-vars>`_.


如何对目标函数进行测试？
--------------------------------------

在对目标函数的测试中，我们总倾向于使用固定的，而不是随机采样的参数。
这时，你可以选择用 :class:`~optuna.trial.FixedTrial` 作为目标函数的输入参数。
它会从一个给定的参数字典中送入固定的参数值。
比如，针对函数 :math:`x + y`, 你可以用如下方式送入两个任意的 :math:`x` 和 :math:`y`:

.. code-block:: python

    def objective(trial):
        x = trial.suggest_uniform('x', -1.0, 1.0)
        y = trial.suggest_int('y', -5, 5)
        return x + y

    objective(FixedTrial({'x': 1.0, 'y': -1}))  # 0.0
    objective(FixedTrial({'x': -1.0, 'y': -4}))  # -5.0

如果使用 :class:`~optuna.trial.FixedTrial` 的话，你也可以用如下方式写单元测试：

.. code-block:: python

    # 一个 pytest 的测试函数。
    def test_objective():
        assert 1.0 == objective(FixedTrial({'x': 1.0, 'y': 0}))
        assert -1.0 == objective(FixedTrial({'x': 0.0, 'y': -1}))
        assert 0.0 == objective(FixedTrial({'x': -1.0, 'y': 1}))
