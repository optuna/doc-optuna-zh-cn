.. _sampler:

用户定义的采样器 (Sampler)
===============================

你可以用用户定义的 sampler 来实现：

- 试验你自己的采样算法，
- 实现具体任务对应的算法来改进优化性能，或者
- 将其他的优化框架包装起来，整合进 Optuna 的管线中 (比如  :class:`~optuna.integration.SkoptSampler` ).


本节将介绍 sampler 类的内部行为，并展示一个实现用户自定义 sampler 的例子。
This section describes the internal behavior of sampler classes and shows an example of implementing a user-defined sampler.


Sampler 的概述
------------------

Sampler 负责产生要被用在 trial 中求值的参数值。
在目标函数内，当一个  `suggest` API (比如 :func:`~optuna.trial.Trial.suggest_uniform`) 被调用时，一个对应的分布对象 (比如 :class:`~optuna.distributions.UniformDistribution`) 就会从内部被创建。
Sampler 则从该分布汇总采样一个参数值。该采样值会被返回给 `suggest` API 的调用者，进而在目标函数内被求值。

为了创建一个新的 sampler, 你需要定一个继承 :class:`~optuna.samplers.BaseSampler` 类的类。
该基类提供三个抽象方法：
:meth:`~optuna.samplers.BaseSampler.infer_relative_search_space`,
:meth:`~optuna.samplers.BaseSampler.sample_relative` 和
:meth:`~optuna.samplers.BaseSampler.sample_independent`.

从这些方法名可以看出，Optuna 支持两种类型的采样过程：一种是 **relative sampling**, 它考虑了单个 trial 内参数之间的相关性， 另一种是 **independent sampling**, 它对各个参数的采样是彼此独立的。

在一个 trial 刚开始时，:meth:`~optuna.samplers.BaseSampler.infer_relative_search_space` 会被调用，它向该 trial 提供一个相对搜索空间。
之后， :meth:`~optuna.samplers.BaseSampler.sample_relative` 会被触发，它从该搜索空间中对相对参数进行采样。
在目标函数的执行过程中，:meth:`~optuna.samplers.BaseSampler.sample_independent` 用于对不属于该相对搜索空间的参数进行采样。


.. note::
    更多细节参见 :class:`~optuna.samplers.BaseSampler` 的文档。


案例: 实现模拟退火 Sampler (SimulatedAnnealingSampler)
--------------------------------------------------------

下面的代码根据 `Simulated Annealing (SA) <https://en.wikipedia.org/wiki/Simulated_annealing>`_ 定义类一个 sampler：

.. code-block:: python

    import numpy as np
    import optuna


    class SimulatedAnnealingSampler(optuna.samplers.BaseSampler):
        def __init__(self, temperature=100):
            self._rng = np.random.RandomState()
            self._temperature = temperature  # Current temperature.
            self._current_trial = None  # Current state.

        def sample_relative(self, study, trial, search_space):
            if search_space == {}:
                return {}

            #
            # 实现模拟退火 (SA) 算法。
            #

            # 计算转移概率。
            prev_trial = study.trials[-2]
            if self._current_trial is None or prev_trial.value <= self._current_trial.value:
                probability = 1.0
            else:
                probability = np.exp((self._current_trial.value - prev_trial.value) / self._temperature)
            self._temperature *= 0.9  # Decrease temperature.

            # Transit the current state if the previous result is accepted.
            if self._rng.uniform(0, 1) < probability:
                self._current_trial = prev_trial

            # 从当前点的邻域中对参数采样。
            #
            # 采样结果会用在该目标函数的下一次求值中。
            params = {}
            for param_name, param_distribution in search_space.items():
                if not isinstance(param_distribution, optuna.distributions.UniformDistribution):
                    raise NotImplementedError('Only suggest_uniform() is supported')

                current_value = self._current_trial.params[param_name]
                width = (param_distribution.high - param_distribution.low) * 0.1
                neighbor_low = max(current_value - width, param_distribution.low)
                neighbor_high = min(current_value + width, param_distribution.high)
                params[param_name] = self._rng.uniform(neighbor_low, neighbor_high)

            return params

        #
        # 下面的部分属于模版代码，和 SA 算法无关。
        #
        def infer_relative_search_space(self, study, trial):
            return optuna.samplers.intersection_search_space(study)

        def sample_independent(self, study, trial, param_name, param_distribution):
            independent_sampler = optuna.samplers.RandomSampler()
            return independent_sampler.sample_independent(study, trial, param_name, param_distribution)


.. note::
   为了代码的简洁性，上面的实现没有支持一些特性 (比如 maximization).
   如果你对如何实现这些特性感兴趣，请看
   `examples/samplers/simulated_annealing.py
   <https://github.com/optuna/optuna/blob/master/examples/samplers/simulated_annealing_sampler.py>`_.


你可以像使用内置的 sampler 一样使用  ``SimulatedAnnealingSampler``:

.. code-block:: python

    def objective(trial):
        x = trial.suggest_uniform('x', -10, 10)
        y = trial.suggest_uniform('y', -5, 5)
        return x**2 + y

    sampler = SimulatedAnnealingSampler()
    study = optuna.create_study(sampler=sampler)
    study.optimize(objective, n_trials=100)

在上面这个优化过程中，参数 ``x`` 和 ``y`` 的值都是由 ``SimulatedAnnealingSampler.sample_relative`` 方法采样得出的。

.. note::
    严格意义上说，在第一个 trial 中，``SimulatedAnnealingSampler.sample_independent`` 用于采样参数值。
    因为，如果没有已经完成的 trial 的话， ``SimulatedAnnealingSampler.infer_relative_search_space``  中的 :func:`~optuna.samplers.intersection_search_space` 是无法对搜索空间进行推断的。
