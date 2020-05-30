.. _attributes:

用户定义属性
===============

利用用户自定义属性，这个功能可以给实验做注解。



将用户定义属性添加到 Study
---------------------------------

:class:`~optuna.study.Study` 对象提供了一个将键-值对设置为用户自定义属性的方法： :func:`~optuna.study.Study.set_user_attr`.
这里的键应该属于 ``str`` 类型， 而值可以是任何能用 ``json.dumps`` 来序列化的对象。

.. code-block:: python

    import optuna
    study = optuna.create_study(storage='sqlite:///example.db')
    study.set_user_attr('contributors', ['Akiba', 'Sano'])
    study.set_user_attr('dataset', 'MNIST')

我们可以利用 :attr:`~optuna.study.Study.user_attr` 属性来获取所有定义过的属性。

.. code-block:: python

    study.user_attrs  # {'contributors': ['Akiba', 'Sano'], 'dataset': 'MNIST'}

:class:`~optuna.struct.StudySummary` 对象中也包含了用户的自定义属性。
我们可以从 :func:`~optuna.study.get_all_study_summaries` 中获取它。

.. code-block:: python

    study_summaries = optuna.get_all_study_summaries('sqlite:///example.db')
    study_summaries[0].user_attrs  # {'contributors': ['Akiba', 'Sano'], 'dataset': 'MNIST'}

.. seealso::
    在命令行界面里，``optuna study set-user-attr`` 可用于设置用户定义属性。


将用户属性添加到 Trial 中
--------------------------------

和 :class:`~optuna.study.Study` 类似，:class:`~optuna.trial.Trial` 对象也提供了一个设置属性的方法 :func:`~optuna.trial.Trial.set_user_attr` 方法。
这些属性是在目标函数内部设置的。

.. code-block:: python

    def objective(trial):
        iris = sklearn.datasets.load_iris()
        x, y = iris.data, iris.target

        svc_c = trial.suggest_loguniform('svc_c', 1e-10, 1e10)
        clf = sklearn.svm.SVC(C=svc_c)
        accuracy = sklearn.model_selection.cross_val_score(clf, x, y).mean()

        trial.set_user_attr('accuracy', accuracy)

        return 1.0 - accuracy  # return error for minimization


可以用如下方式获取这些注解的属性：

.. code-block:: python

    study.trials[0].user_attrs  # {'accuracy': 0.83}

注意，在本例中，属性不是被注解到 :class:`~optuna.study.Study` 的，它属于一个单独的 :class:`~optuna.trial.Trial`.
