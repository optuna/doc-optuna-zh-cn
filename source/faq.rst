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


How to save machine learning models trained in objective functions?
-------------------------------------------------------------------

Optuna saves hyperparameter values with its corresponding objective value to storage,
but it discards intermediate objects such as machine learning models and neural network weights.
To save models or weights, please use features of the machine learning library you used.

We recommend saving :obj:`optuna.trial.Trial.number` with a model in order to identify its corresponding trial.
For example, you can save SVM models trained in the objective function as follows:

.. code-block:: python

    def objective(trial):
        svc_c = trial.suggest_loguniform('svc_c', 1e-10, 1e10)
        clf = sklearn.svm.SVC(C=svc_c)
        clf.fit(X_train, y_train)

        # Save a trained model to a file.
        with open('{}.pickle'.format(trial.number), 'wb') as fout:
            pickle.dump(clf, fout)
        return 1.0 - accuracy_score(y_valid, clf.predict(X_valid))


    study = optuna.create_study()
    study.optimize(objective, n_trials=100)

    # Load the best model.
    with open('{}.pickle'.format(study.best_trial.number), 'rb') as fin:
        best_clf = pickle.load(fin)
    print(accuracy_score(y_valid, best_clf.predict(X_valid)))


How can I obtain reproducible optimization results?
---------------------------------------------------

To make the parameters suggested by Optuna reproducible, you can specify a fixed random seed via ``seed`` argument of :class:`~optuna.samplers.RandomSampler` or :class:`~optuna.samplers.TPESampler` as follows:

.. code-block:: python

    sampler = TPESampler(seed=10)  # Make the sampler behave in a deterministic way.
    study = optuna.create_study(sampler=sampler)
    study.optimize(objective)

However, there are two caveats.

First, when optimizing a study in distributed or parallel mode, there is inherent non-determinism.
Thus it is very difficult to reproduce the same results in such condition.
We recommend executing optimization of a study sequentially if you would like to reproduce the result.

Second, if your objective function behaves in a non-deterministic way (i.e., it does not return the same value even if the same parameters were suggested), you cannot reproduce an optimization.
To deal with this problem, please set an option (e.g., random seed) to make the behavior deterministic if your optimization target (e.g., an ML library) provides it.


How are exceptions from trials handled?
---------------------------------------

Trials that raise exceptions without catching them will be treated as failures, i.e. with the :obj:`~optuna.trial.TrialState.FAIL` status.

By default, all exceptions except :class:`~optuna.exceptions.TrialPruned` raised in objective functions are propagated to the caller of :func:`~optuna.study.Study.optimize`.
In other words, studies are aborted when such exceptions are raised.
It might be desirable to continue a study with the remaining trials.
To do so, you can specify in :func:`~optuna.study.Study.optimize` which exception types to catch using the ``catch`` argument.
Exceptions of these types are caught inside the study and will not propagate further.

You can find the failed trials in log messages.

.. code-block:: sh

    [W 2018-12-07 16:38:36,889] Setting status of trial#0 as TrialState.FAIL because of \
    the following error: ValueError('A sample error in objective.')

You can also find the failed trials by checking the trial states as follows:

.. code-block:: python

    study.trials_dataframe()

.. csv-table::

    number,state,value,...,params,system_attrs
    0,TrialState.FAIL,,...,0,Setting status of trial#0 as TrialState.FAIL because of the following error: ValueError('A test error in objective.')
    1,TrialState.COMPLETE,1269,...,1,

.. seealso::

    The ``catch`` argument in :func:`~optuna.study.Study.optimize`.


How are NaNs returned by trials handled?
----------------------------------------

Trials that return :obj:`NaN` (``float('nan')``) are treated as failures, but they will not abort studies.

Trials which return :obj:`NaN` are shown as follows:

.. code-block:: sh

    [W 2018-12-07 16:41:59,000] Setting status of trial#2 as TrialState.FAIL because the \
    objective function returned nan.


What happens when I dynamically alter a search space?
-----------------------------------------------------

Since parameters search spaces are specified in each call to the suggestion API, e.g.
:func:`~optuna.trial.Trial.suggest_uniform` and :func:`~optuna.trial.Trial.suggest_int`,
it is possible to in a single study alter the range by sampling parameters from different search
spaces in different trials.
The behavior when altered is defined by each sampler individually.

.. note::

    Discussion about the TPE sampler. https://github.com/optuna/optuna/issues/822


How can I use two GPUs for evaluating two trials simultaneously?
----------------------------------------------------------------

If your optimization target supports GPU (CUDA) acceleration and you want to specify which GPU is used, the easiest way is to set ``CUDA_VISIBLE_DEVICES`` environment variable:

.. code-block:: bash

    # On a terminal.
    #
    # Specify to use the first GPU, and run an optimization.
    $ export CUDA_VISIBLE_DEVICES=0
    $ optuna study optimize foo.py objective --study foo --storage sqlite:///example.db

    # On another terminal.
    #
    # Specify to use the second GPU, and run another optimization.
    $ export CUDA_VISIBLE_DEVICES=1
    $ optuna study optimize bar.py objective --study bar --storage sqlite:///example.db

Please refer to `CUDA C Programming Guide <https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#env-vars>`_ for further details.


How can I test my objective functions?
--------------------------------------

When you test objective functions, you may prefer fixed parameter values to sampled ones.
In that case, you can use :class:`~optuna.trial.FixedTrial`, which suggests fixed parameter values based on a given dictionary of parameters.
For instance, you can input arbitrary values of :math:`x` and :math:`y` to the objective function :math:`x + y` as follows:

.. code-block:: python

    def objective(trial):
        x = trial.suggest_uniform('x', -1.0, 1.0)
        y = trial.suggest_int('y', -5, 5)
        return x + y

    objective(FixedTrial({'x': 1.0, 'y': -1}))  # 0.0
    objective(FixedTrial({'x': -1.0, 'y': -4}))  # -5.0


Using :class:`~optuna.trial.FixedTrial`, you can write unit tests as follows:

.. code-block:: python

    # A test function of pytest
    def test_objective():
        assert 1.0 == objective(FixedTrial({'x': 1.0, 'y': 0}))
        assert -1.0 == objective(FixedTrial({'x': 0.0, 'y': -1}))
        assert 0.0 == objective(FixedTrial({'x': -1.0, 'y': 1}))
