# 基于机器学习的DAG调度平台
```
什么是DAG?
有向无环图
树形结构：除根节点，每个节点有且仅有一个上级节点，下级节点不限。根节点没有上级节点。
图结构：每个节点上级、下级节点数不限。
```
## DAG调度平台的定义及场景
任务调度是在各行各业是个基础问题，当任务复杂同时存在任务复杂依赖的时候，就需要DAG调度。如：机器学习的可视化建模（PAI平台、第四范式），数据的抽取、转换、加载（ETL），在业务复杂情况下就需要DAG的调度管理等
接下来说说基于机器学习的DAG调度平台

## 系统架构

构图：

### 系统架构说明
DAG调度平台主要的职责是:
1.接受机器学习web传过来的yaml文件(dag定义文件)
2.解析yaml文件，变成结构化数据存储到mysql数据库
3.开始调度dag定义各个算子任务
4.算子执行引擎根据算子类型分发到各个环境进行执行

```
名词说明
yaml:类型XML的数据描述语言，语法更加简单
算子:机器学习的DAG中各个节点即为算子，在算子执行引擎中称为算子任务。算子背后是python实现的一些算法组件
```

#### 1.机器学习前端交互
机器学习平台前端主要是将机器学习的流程装成一个dag，定义各个算子的出入参，以及算子的配置参数，组装成一个yaml文件，传给DAG调图平台（Azkaban是zip方式交互，Airflow是通过py文件定义，Oozie通过xml）。
**一个完整的DAG定义应包含以下算子：**

- 数据读取/数据预处理
- 特征功能
- 模型训练
- 模型预测
- 模型评估
- 模型部署

下图是个简化版的DAG定义，除去了模型部署算子


#### 2.DAG调度平台各模块介绍
##### dag engine（图引擎）：
负责解析传入的yaml文件。根据yaml的配置生成算子的出入参以及运行配置信息保存到数据库。同时负责任务的调用。
##### opertor engine（算子执行引擎）: 
负责算子执行，根据算子类型分发到不同的执行器中。统一的启停接口，日志查询接口，任务状态查询接口
##### executor（执行器）：
- local executor(本地执行器)：
执行单机的python任务，执行单机文件方式的机器学习算法。当没有大数据平台的时候，只能通过本地执行器执行DAG
- dc executor（分布式计算平台执行器）：
将python算法发送至大数据计算平台，使用大数据平台资源运行算子。
- base executor (执行器接口):
以后的执行器实现需要实现这个基类，方便拓展。

#### 3.分布式计算平台交互
针对不同的的计算平台实现base executor去自定义扩充。本系统通过dc executor实现，
分布式计算平台需要将python code通过http接口发送过去进行执行。

## 部署架构图

### separation方式

### mixture方式

### celery方式

## 实现细节

### yaml定义格式
```
dag:
 operator_list: [algo_local_read_file_45_1517360824080,algo_local_split_data_45_1517360836712,algo_local_model_2c_l_45_1517362008544,algo_local_model_predict_45_1517362016532,algo_local_model_2c_eval_45_1517362022452,algo_local_model_gbdt_111_1517801573063]
 operator_rels:
  algo_local_read_file_45_1517360824080: [{"target":"algo_local_split_data_45_1517360836712","source_index":0,"target_index":0}]
  algo_local_split_data_45_1517360836712: [{"target":"algo_local_model_2c_l_45_1517362008544","source_index":0,"target_index":0},{"target":"algo_local_model_gbdt_111_1517801573063","source_index":1,"target_index":0}]
  algo_local_model_predict_45_1517362016532: [{"target":"algo_local_model_2c_eval_45_1517362022452","source_index":0,"target_index":0}]
  algo_local_model_gbdt_111_1517801573063: [{"target":"algo_local_model_predict_45_1517362016532","source_index":0,"target_index":0}]
  algo_local_model_2c_l_45_1517362008544: [{"target":"algo_local_model_predict_45_1517362016532","source_index":0,"target_index":1}]
 operator_details:
  
  algo_local_read_file_45_1517360824080:
   algo_name: algo_local_read_file
   data_type: 本地python
   type: 数据源
   cn_name: 读文件
   coordinate:
    x: 137
    y: 69
   params:
    data_id: 40
  algo_local_split_data_45_1517360836712:
   algo_name: algo_local_split_data
   data_type: 本地python
   type: 数据预处理
   cn_name: 拆分组件
   coordinate:
    x: 226
    y: 164
   params:
    split_type: 1
    ext1: 0.8
    ext2: null
  algo_local_model_2c_l_45_1517362008544:
   algo_name: algo_local_model_2c_l
   data_type: 本地python
   type: 模型算法
   cn_name: 逻辑回归二分类
   coordinate:
    x: 130
    y: 262
   params:
    x_cols: [LIMIT_BAL,SEX,EDUCATION,MARRIAGE,AGE,PAY_0,PAY_2,PAY_3,PAY_4,PAY_5,PAY_6,BILL_AMT1,BILL_AMT2,BILL_AMT3,BILL_AMT4,BILL_AMT5,BILL_AMT6,PAY_AMT1,PAY_AMT2,PAY_AMT3,PAY_AMT4,PAY_AMT5,PAY_AMT6]
    y_col: next_month
    pre_value: 1
    penalty: l2
    C: 1
    max_iter: 100
    senior: true
    class_weight: null
    dual: false
    fit_intercept: true
    intercept_scaling: 1
    multi_class: ovr
    n_jobs: 1
    random_state: null
    solver: liblinear
    tol: 0.0001
    verbose: 0
    warm_start: false
  algo_local_model_predict_45_1517362016532:
   algo_name: algo_local_model_predict
   data_type: 本地python
   type: 模型预测
   cn_name: 模型预测
   coordinate:
    x: 258
    y: 396
   params:
    x_cols: [LIMIT_BAL,SEX,EDUCATION,MARRIAGE,AGE,PAY_0,PAY_2,PAY_3,PAY_4,PAY_5,PAY_6,BILL_AMT1,BILL_AMT2,BILL_AMT3,BILL_AMT4,BILL_AMT5,BILL_AMT6,PAY_AMT1,PAY_AMT2,PAY_AMT3,PAY_AMT4,PAY_AMT5,PAY_AMT6]
  algo_local_model_2c_eval_45_1517362022452:
   algo_name: algo_local_model_2c_eval
   data_type: 本地python
   type: 模型评估
   cn_name: 二分类评估
   coordinate:
    x: 270
    y: 503
   params:
    y_col: next_month
    pre_col: predict_result
    pre_value: 1
  algo_local_model_gbdt_111_1517801573063:
   algo_name: algo_local_model_gbdt
   data_type: 本地python
   type: 模型算法
   cn_name: GBDT
   coordinate:
    x: 432.1111111111111
    y: 295.3333333333333
   params:
    x_cols: [LIMIT_BAL,SEX,EDUCATION,MARRIAGE,AGE,PAY_0,PAY_2,PAY_3,PAY_4,PAY_5,PAY_6,BILL_AMT1,BILL_AMT2,BILL_AMT3,BILL_AMT4,BILL_AMT5,BILL_AMT6,PAY_AMT1,PAY_AMT2,PAY_AMT3,PAY_AMT4,PAY_AMT5,PAY_AMT6]
    y_col: next_month
    pre_value: 1
    n_estimators: 10
    max_depth: 5
    senior: true
    criterion: friedman_mse
    init: null
    learning_rate: 0.1
    loss: deviance
    max_features: null
    max_leaf_nodes: null
    min_impurity_decrease: 0
    min_impurity_split: null
    min_samples_leaf: 1
    min_samples_split: 2
    min_weight_fraction_leaf: 0
    presort: auto
    random_state: null
    subsample: 1
    verbose: 0
    warm_start: false
 params:
  translate: [41,-20]
  scale: 0.9
```

### dag engine实现逻辑
1.当前节点，采用广度优先遍历获取所有需要执行的算子(节点)信息。
2.轮询所有算子(节点)，判断上算子(节点)是否全部执行完成，执行完成开始执行当前算子(节点)。
3.发送请求到operator engine开始执行当前算子(节点)任务。

### operator engine实现逻辑
1.主进程接受task请求，添加任务执行队列、任务监听队列。
2.任务执行进程轮询接受到的队列，根据不同任务类型调用不同executor
3.任务监听进程轮询接受到的队列，调用不同executor查询任务执行状态，是任务执行的最终状态（成功、失败）回调dag engine


### local executor实现逻辑
1.local executor接受任务，发送到队列中。
2.local worker进程池(cpu数*2个进程)，轮询获取队列中任务，使用importlib的python去执行对应算子。


