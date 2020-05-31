.. _pruning:

对无望的 Trial 进行剪枝 (Pruning)
====================================
该功能可以在训练的早期阶段自动终止无望的 Trial (a.k.a., 自动化 early-stopping).
Optuna 提供了一些接口，可以用于在迭代训练算法中简洁地实现剪枝 (Pruning)。


开启 Pruner
---------------------

为了打开 Pruning 功能，你需要在迭代式训练的每一步完成后调用函数 :func:`~optuna.trial.Trial.report` 和 :func:`~optuna.trial.Trial.should_prune` 
:func:`~optuna.trial.Trial.report` 定期监测这个过程中的目标函数值。
:func:`~optuna.trial.Trial.should_prune` 根据提前定义好的条件，判定该 trial 是否需要终止。

.. code-block:: python

    """filename: prune.py"""

    import sklearn.datasets
    import sklearn.linear_model
    import sklearn.model_selection

    import optuna

    def objective(trial):
        iris = sklearn.datasets.load_iris()
        classes = list(set(iris.target))
        train_x, valid_x, train_y, valid_y = \
            sklearn.model_selection.train_test_split(iris.data, iris.target, test_size=0.25, random_state=0)

        alpha = trial.suggest_loguniform('alpha', 1e-5, 1e-1)
        clf = sklearn.linear_model.SGDClassifier(alpha=alpha)

        for step in range(100):
            clf.partial_fit(train_x, train_y, classes=classes)

            # Report intermediate objective value.
            intermediate_value = 1.0 - clf.score(valid_x, valid_y)
            trial.report(intermediate_value, step)

            # Handle pruning based on the intermediate value.
            if trial.should_prune():
                raise optuna.exceptions.TrialPruned()

        return 1.0 - clf.score(valid_x, valid_y)

    # 将中位数终止规则作为 pruning 条件。
    study = optuna.create_study(pruner=optuna.pruners.MedianPruner())
    study.optimize(objective, n_trials=20)


运行上述脚本:

.. code-block:: bash

    $ python prune.py
    [I 2018-11-21 17:27:57,836] Finished trial#0 resulted in value: 0.052631578947368474. Current best value is 0.052631578947368474 with parameters: {'alpha': 0.011428158279113485}.
    [I 2018-11-21 17:27:57,963] Finished trial#1 resulted in value: 0.02631578947368418. Current best value is 0.02631578947368418 with parameters: {'alpha': 0.01862693201743629}.
    [I 2018-11-21 17:27:58,164] Finished trial#2 resulted in value: 0.21052631578947367. Current best value is 0.02631578947368418 with parameters: {'alpha': 0.01862693201743629}.
    [I 2018-11-21 17:27:58,333] Finished trial#3 resulted in value: 0.02631578947368418. Current best value is 0.02631578947368418 with parameters: {'alpha': 0.01862693201743629}.
    [I 2018-11-21 17:27:58,617] Finished trial#4 resulted in value: 0.23684210526315785. Current best value is 0.02631578947368418 with parameters: {'alpha': 0.01862693201743629}.
    [I 2018-11-21 17:27:58,642] Setting status of trial#5 as TrialState.PRUNED.
    [I 2018-11-21 17:27:58,666] Setting status of trial#6 as TrialState.PRUNED.
    [I 2018-11-21 17:27:58,675] Setting status of trial#7 as TrialState.PRUNED.
    [I 2018-11-21 17:27:59,183] Finished trial#8 resulted in value: 0.39473684210526316. Current best value is 0.02631578947368418 with parameters: {'alpha': 0.01862693201743629}.
    [I 2018-11-21 17:27:59,202] Setting status of trial#9 as TrialState.PRUNED.
    ...

我们可以在输出信息中看到 ``Setting status of trial#{} as TrialState.PRUNED``.
这意味着这些 trial 在他们完成迭代之前就被终止了。

用于 Pruning 的集成模块
--------------------------
为了能更加方便地实现 pruning, Optuna 为以下框架提供了集成模块。

- XGBoost: :class:`optuna.integration.XGBoostPruningCallback`
- LightGBM: :class:`optuna.integration.LightGBMPruningCallback`
- Chainer: :class:`optuna.integration.ChainerPruningExtension`
- Keras: :class:`optuna.integration.KerasPruningCallback`
- TensorFlow :class:`optuna.integration.TensorFlowPruningHook`
- tf.keras :class:`optuna.integration.TFKerasPruningCallback`
- MXNet :class:`optuna.integration.MXNetPruningCallback`
- PyTorch Ignite :class:`optuna.integration.PyTorchIgnitePruningHandler`
- PyTorch Lightning :class:`optuna.integration.PyTorchLightningPruningCallback`
- FastAI :class:`optuna.integration.FastAIPruningCallback`

比如, :class:`~optuna.integration.XGBoostPruningCallback` 在无需修改训练迭代逻辑的情况下引入了 pruning.
(完整脚本见 `example <https://github.com/optuna/optuna/blob/master/examples/pruning/xgboost_integration.py>`_ .)

.. code-block:: python

        pruning_callback = optuna.integration.XGBoostPruningCallback(trial, 'validation-error')
        bst = xgb.train(param, dtrain, evals=[(dvalid, 'validation')], callbacks=[pruning_callback])
