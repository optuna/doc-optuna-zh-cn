Command Line Interface
======================

.. code-block:: 

   optuna
      [--version]
      [-v | -q]
      [--log-file LOG_FILE]
      [--debug]
      [--storage STORAGE]


.. option:: --version

   显示应用的版本号并退出


.. option:: -v, --verbose

   打印冗余内容。可重复。

.. option:: -q, --quiet

   禁止打印除了警告和错误之外的输出。


.. option:: --log-file <LOG_FILE>

   确定用于记录日志的文件名。该选项默认情况下是禁用的。

.. option:: --debug

   显示异常处理和堆栈跟踪信息。

.. option:: --storage <STORAGE>

   数据库 URL. (比如 ``sqlite:///example.db`` )


create-study
^^^^^^^^^^^^^^^

.. code-block:: 

   optuna create-study
      [--study-name STUDY_NAME]
      [--direction {minimize,maximize}]
      [--skip-if-exists]


.. option:: --study-name <STUDY_NAME>

   设定具有可读性的 study 名，以方便和其他的 study 区分。

.. option:: --direction <DIRECTION>

   设定新 study 的优化方向。如果方向是最小化的话就设成 'minimize', 如果是最大化就设置成 'maximize'.

.. option:: --skip-if-exists

   如果开启该选项， 当存在一个同名的 study 时，optuna 在优化开始时将跳过创建 study 的过程，并且不报错。

该命令由 optuna 插件提供。

dashboard
^^^^^^^^^^^^^^^

启动 web dashboard (beta).

.. code-block:: 

   optuna dashboard
      [--study STUDY]
      [--study-name STUDY_NAME]
      [--out OUT]
      [--allow-websocket-origin BOKEH_ALLOW_WEBSOCKET_ORIGINS]



.. option:: --study <STUDY>

   该参数已弃用。作为替代，请使用  –study-name.


.. option:: --study-name <STUDY_NAME>

   设定具有可读性的 study 名，以方便和其他的 study 区分。


.. option:: --allow-websocket-origin <BOKEH_ALLOW_WEBSOCKET_ORIGINS>

   允许从制定的 host 发起的的 websocket 请求。
   该选项用法和 bokeh 的 ``–allow-websocket-origin`` 一样。
   更多细节见 https://docs.bokeh.org/en/latest/docs/reference/command/subcommands/serve.html.
   设定具有可读性的 study 名，以方便和其他的 study 区分。

该命令由 optuna 插件提供。


delete-study
^^^^^^^^^^^^^^^

删除特定的 study.

.. code-block:: 

   optuna delete-study [--study-name STUDY_NAME]


.. option:: --study-name <STUDY_NAME>

   设定具有可读性的 study 名，以方便和其他的 study 区分。

该命令由 optuna 插件提供。


storage upgrade
^^^^^^^^^^^^^^^

升级数据库。


.. code-block:: 

   optuna storage upgrade


该命令由 optuna 插件提供。


studies
^^^^^^^^^^^^^^^

显示 study 列表



.. code-block:: 

   optuna studies
      [-f {csv,json,table,value,yaml}]
      [-c COLUMN]
      [--quote {all,minimal,none,nonnumeric}]
      [--noindent]
      [--max-width <integer>]
      [--fit-width]
      [--print-empty]
      [--sort-column SORT_COLUMN]

.. option:: -f <FORMATTER>, --format <FORMATTER>

   输出格式，默认情况下是表格。

.. option:: -c COLUMN, --column COLUMN

  用于设定要展示的（表格）列。
  可设置多个 column 参数以同时显示多个column.

.. option:: --quote <QUOTE_MODE>

   设置引号模式。默认为数值。

.. option:: --noindent

   设定是否在 json 中禁用缩进。


.. option:: --max-width <integer>

   最大展示宽度。如果设置成小于 1 的数值，就代表禁用最大宽度限制。
   你也可以通过设置 ``CLIFF_MAX_TERM_WIDTH `` 环境变量来限制最大宽度，
   但是 ``--max-width`` 参数的优先级比环境变量高。

.. option:: --fit-width

   将输出表格设置成与显示屏同宽。
   如果 ``-max-width`` > 1 的话，该选项会自动启用。
   设定是否在 json 中禁用缩进。
   也可以通过设置 ``CLIFF_FIT_WIDTH`` 环境变量的值为 1 来启用该选项。

.. option:: --print-empty

   在无数据的情况下也输出空表格

.. option:: --sort-column SORT_COLUMN

   按照指定列来对表格的行进行排序。
   越靠前指定的列在排序中的优先级越高。没有被指定的列不参与排序。

该命令由 optuna 插件提供。

study optimize
^^^^^^^^^^^^^^^

针对特定 study 开始优化过程。


.. code-block:: 

   optuna study optimize
      [--n-trials N_TRIALS]
      [--timeout TIMEOUT]
      [--n-jobs N_JOBS]
      [--study STUDY]
      [--study-name STUDY_NAME]
      file
      method

.. option:: --n-trials <N_TRIALS>

   优化过程将运行的总 trial 数目。
   如果不设定该数目的话，优化过程会一直持续下去。

.. option:: --timeout <TIMEOUT>

   到达指定时间（按秒计）以后终止 study. 
   如果不设定该数目的话，优化过程也会一直持续下去。
   
.. option:: --n-jobs <N_JOBS>

   并行运行的任务个数。如果该参数设置为 -1, 则任务的实际数目将等于 CPU 的核心数。

.. option:: --study <STUDY>

   该参数已经弃用，请使用 ``–study-name`` 来替代它。

.. option:: --study-name <STUDY_NAME>

   设定具有可读性的 study 名，以方便和其他的 study 区分。

.. option:: file

   用于指定定义了目标函数的 Python 脚本文件名。

.. option:: method

   目标函数的方法名。

该命令由 optuna 插件提供。


study set-user-attr
^^^^^^^^^^^^^^^^^^^^^^^

针对特定 study 设定用户属性 (user attribute).


.. code-block:: 

   optuna study set-user-attr
      [--study STUDY]
      [--study-name STUDY_NAME]
      --key KEY
      --value VALUE

.. option:: --study <STUDY>

   该参数已经弃用，请使用 ``–study-name`` 来替代它。

.. option:: --study-name <STUDY_NAME>

   设定具有可读性的 study 名，以方便和其他的 study 区分。

.. option:: --key <KEY>, -k <KEY>

   设定用户属性中的键 (key).

.. option:: --value <VALUE>, -v <VALUE>

   设定用户属性中的值 (value).

该命令由 optuna 插件提供。

.. .. autoprogram-cliff:: optuna.cli._OptunaApp
..    :application: optuna

.. .. autoprogram-cliff:: optuna.command
..    :application: optuna
