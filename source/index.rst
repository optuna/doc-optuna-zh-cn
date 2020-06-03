|optunalogo|

Optuna: 一个超参数优化框架
===============================================

*Optuna* 是一个特别为机器学习设计的自动超参数优化软件框架。它具有命令式的，*define-by-run* 风格的 API。由于这种 API 的存在，用 Optuna 编写的代码模块化程度很高，Optuna 的用户因此也可以动态地构造超参数的搜索空间。

主要特点
------------

Optuna 有如下现代化的功能：

-  并行的分布式优化
-  对不理想实验 (trial) 的剪枝 (pruning)
-  轻量级、多功能和跨平台架构

基本概念
--------------
我们以如下方式使用 *study* 和 *trial* 这两个术语：

-  Study: 基于目标函数的优化过程
-  Trial: 目标函数的单次执行过程

请参考下面的示例代码。一个 *study* 的目的是通过多次 *trial* (例如 ``n_trials=100`` ) 来找出最佳的超参数值集（比如选择 ``classifier`` 还是 ``svm_c``）。而 Optuna 旨在加速和自动化此类 *study* 优化过程。

|Open in Colab|

.. code:: python

    import ...

    # 定义待优化的目标函数。
    def objective(trial):

        # 调用 suggest 方法用于给该 Trial 生成超参数。
        regressor_name = trial.suggest_categorical('classifier', ['SVR', 'RandomForest'])
        if regressor_name == 'SVR':
            svr_c = trial.suggest_loguniform('svr_c', 1e-10, 1e10)
            regressor_obj = sklearn.svm.SVR(C=svr_c)
        else:
            rf_max_depth = trial.suggest_int('rf_max_depth', 2, 32)
            regressor_obj = sklearn.ensemble.RandomForestRegressor(max_depth=rf_max_depth)

        X, y = sklearn.datasets.load_boston(return_X_y=True)
        X_train, X_val, y_train, y_val = sklearn.model_selection.train_test_split(X, y, random_state=0)

        regressor_obj.fit(X_train, y_train)
        y_pred = regressor_obj.predict(X_val)

        error = sklearn.metrics.mean_squared_error(y_val, y_pred)

        return error  # Trial 对象对应的目标函数值。

    study = optuna.create_study()  # 创建新study.
    study.optimize(objective, n_trials=100)  # 开始目标函数的优化过程。

Communication
-------------

- 可在 `GitHub Issues <https://github.com/optuna/optuna/issues>`__ 报告bug、提feature request 和问问题。
- 可在 `Gitter <https://gitter.im/optuna/optuna>`__ 与开发者互动.
- 可在 `StackOverflow <https://stackoverflow.com/questions/tagged/optuna>`__ 提问。

贡献
------------

欢迎大家对Optuna的一切贡献！但是在发送pull request时请遵从 `contribution guide <https://github.com/optuna/optuna/blob/master/CONTRIBUTING.md>`__ 的规范.

License
-------

MIT License (see `LICENSE <https://github.com/optuna/optuna/blob/master/LICENSE>`__).

Reference
---------

Takuya Akiba, Shotaro Sano, Toshihiko Yanase, Takeru Ohta, and Masanori
Koyama. 2019. Optuna: A Next-generation Hyperparameter Optimization
Framework. In KDD (`arXiv <https://arxiv.org/abs/1907.10902>`__).

.. toctree::
   :maxdepth: 2
   :caption: Contents:

   installation
   tutorial/index
   reference/index
   faq

Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

.. |optunalogo| image:: https://raw.githubusercontent.com/optuna/optuna/master/docs/image/optuna-logo.png
  :width: 800
  :alt: OPTUNA
.. |Open in Colab| image:: https://colab.research.google.com/assets/colab-badge.svg
  :target: http://colab.research.google.com/github/optuna/optuna/blob/master/examples/quickstart.ipynb
