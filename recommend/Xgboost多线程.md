### xgboost多线程

## xgboost简介

&nbsp;&nbsp;&nbsp;&nbsp;XGBoost是一个开源软件库，它为 C++、Java、Python、R、和Julia提供了一个梯度提升框架，适用于Linux、Windows和 mac os。根据项目的描述，它的目的在于提供一个"可扩展、可移植和分布式梯度提升(GBM、GBRT、GBDT)库"。 XGBoost除了可以在单一机器上运行，也支持运行在分布式框架Apache Hadoop、Apache Spark、Apache Flink。 近几年，由于这个算法受到许多在机器学习竞赛中获奖团队的青睐，因而受到了广泛的欢迎和关注。
<p align="right">——维基百科</p>


## 背景
 &nbsp;&nbsp;&nbsp;&nbsp;优化推荐引擎平均耗时的过程中，因当时大多模型基于xgboost进行预测，所以基于xgboost预测分数时进行优化耗时。
 ## xgboost版本
  &nbsp;&nbsp;&nbsp;&nbsp;本文基于xgboost 0.90进行叙述。
 github： 
 &nbsp;&nbsp;&nbsp;&nbsp;[https://github.com/dmlc/xgboost](https://github.com/dmlc/xgboost)

## 第一版优化
### 问题分析
 &nbsp;&nbsp;&nbsp;&nbsp;在查看xgboost java源码时，在Booster类中predict方法为同步方法，含有synchronized关键字，这意味着在模型并发预测过程中，含有大量的阻塞状态。


### 优化方式
 &nbsp;&nbsp;&nbsp;&nbsp;取消java源码中synchronized关键字，采用ReentrantLock中lockInterruptibly的方式来解决大量阻塞情况。

### 代码
```java
	  private synchronized float[][] predict(DMatrix data,
	                                         boolean outputMargin,
	                                         int treeLimit,
	                                         boolean predLeaf,
	                                         boolean predContribs) throws XGBoostError {
	    int optionMask = 0;
	    // ......
	    // ......
	    return predicts;
	  }
   

```
```java
	  //修改后
    private  float[][] predict(DMatrix data, boolean outputMargin, int treeLimit, boolean predLeaf, boolean predContribs) throws XGBoostError {
		// 修改代码块start
        try {
			// 加锁方式更换为ReentrantLock，支持中断操作
            final ReentrantLock lock = this.lock;
            lock.lockInterruptibly();
	
            try {
            // 修改代码块end
                int optionMask = 0;
                // ......
				// ......
                return predicts;
            //修改代码块start
            } finally {
                lock.unlock();
            }

        }   catch (InterruptedException inException) {
            throw new XGBoostError("InterruptedException");
        }
        //修改代码块end
    }
```

### 实现原因

 1. &nbsp;&nbsp;xgboost源码中含有线程不安全问题。在Java源码侧必须加入锁来控制线程安全问题;
 2. &nbsp;&nbsp;取消synchronized关键字，更换为ReentrantLock;
 2.1.  &nbsp;&nbsp;synchronized利用JVM控制锁的操作，其中含有偏向锁、轻量级锁、重量级锁。在高并发的情况下，synchronized通过锁升级的操作变成重量级锁，从而性能变差。而ReentrantLock利用CAS方式进行加锁操作，性能优于重量级锁;
 2.2.&nbsp;&nbsp; ReentrantLock中包含lockInterruptibly加锁方式。lockInterruptibly允许其他线程在等待锁的过程中调用Thread.interrupt方法进行中断。从而在达到外围对象的耗时阈值时进行中断。保证了整体耗时的稳定性;  

## 第二版优化
### 问题分析
1. &nbsp;&nbsp;将java中的synchronized更换为ReentrantLock后，依然解决不了大量并发预测问题。在查看源码与调研过程中，发现在cpu_predictor.cc中 PredLoopSpecalize方法含有线程不安全问题。在CPUPredictor类中含有std::vector\<RegTree::FVec> thread_temp的成员变量。在PredLoopSpeclize中，会对thread_temp分配空间，再进行OpenMp操作。在多线程模式下，成员变量会导致线程安全问题。
2. &nbsp;&nbsp;在Booster(c_api.cc)类中的LazyInit方法，对配置与模型进行初始化的操作。由于存在大量的成员变量，当多线程模式下，会导致线程安全问题。
### 优化方式
1. 将PredLoopSpeclize中的thread_temp变量更改为方法中定义的变量。

2. 将Booster类中的LazyInit方法进行加锁控制。

### 代码
路径： src/predictor/cpu_predictor.cc
```c++

inline void PredLoopSpecalize(DMatrix* p_fmat,
                                std::vector<bst_float>* out_preds,
                                const gbm::GBTreeModel& model, int num_group,
                                unsigned tree_begin, unsigned tree_end) {
    const MetaInfo& info = p_fmat->Info();
    const int nthread = omp_get_max_threads();
    
    InitThreadTemp(nthread, model.param.num_feature);
    std::vector<bst_float>& preds = *out_preds;
    CHECK_EQ(model.param.size_leaf_vector, 0)
        << "size_leaf_vector is enforced to 0 so far";
    CHECK_EQ(preds.size(), p_fmat->Info().num_row_ * num_group);
    // start collecting the prediction
    // ......
    }

```
```c++
    //修改后
    inline void PredLoopSpecalize(DMatrix* p_fmat,
                                std::vector<bst_float>* out_preds,
                                const gbm::GBTreeModel& model, int num_group,
                                unsigned tree_begin, unsigned tree_end) {
    const MetaInfo& info = p_fmat->Info();
    const int nthread = omp_get_max_threads();
    // 定义本地变量，避免出现线程安全问题
    // 修改块start
    std::vector<RegTree::FVec> local_thread_temp;
    InitThreadTemp(nthread, model.param.num_feature, local_thread_temp);
    // 修改块end
    std::vector<bst_float>& preds = *out_preds;
    CHECK_EQ(model.param.size_leaf_vector, 0)
        << "size_leaf_vector is enforced to 0 so far";
    CHECK_EQ(preds.size(), p_fmat->Info().num_row_ * num_group);
    // start collecting the prediction
    // ......
    }
```
路径：src/predictor/cpu_predictor.cc
```c++

 inline void InitThreadTemp(int nthread, int num_feature) {
     int prev_thread_temp_size = thread_temp.size();
     if (prev_thread_temp_size < nthread) {
         thread_temp.resize(nthread, RegTree::FVec());
         for (int i = prev_thread_temp_size; i < nthread; ++i) {
             thread_temp[i].Init(num_feature);
         }
     }
 }
 
```
```c++
 	//修改后
    // 传入本地变量local_thread_temp
    // 将thread_temp更改为local_thread_temp
  inline void InitThreadTemp(int nthread, int num_feature, std::vector<RegTree::FVec>& local_thread_temp) {
    int prev_thread_temp_size = local_thread_temp.size();
    if (prev_thread_temp_size < nthread) {
      local_thread_temp.resize(nthread, RegTree::FVec());
      for (int i = prev_thread_temp_size; i < nthread; ++i) {
        local_thread_temp[i].Init(num_feature);
      }
    }
  }
```
路径： src/c_api/c_api.cc
```c++

inline void LazyInit() {
  if (!configured_) {
    LoadSavedParamFromAttr();
    learner_->Configure(cfg_);
    configured_ = true;
  }
  if (!initialized_) {
    learner_->InitModel();
    initialized_ = true;
  }
}

```
```c++

//修改后
//修改代码块start
pthread_mutex_t lock_ = PTHREAD_MUTEX_INITIALIZER;
// 修改代码块end
inline void LazyInit() {
	//修改代码块start
    if (!configured_ || !initialized_) {// 加锁
        pthread_mutex_lock(&lock_);
        // 修改代码块end
        if (!configured_) {
            LoadSavedParamFromAttr();
            learner_->Configure(cfg_);
            configured_ = true;
        }
        if (!initialized_) {
            learner_->InitModel();
            initialized_ = true;
        }
        //修改代码块start
        pthread_mutex_unlock(&lock_);
        // 修改代码块end
    }
 
}
```
### 实现原因
1. &nbsp;&nbsp;利用本地变量替换成员变量，保证变量在多线程模式中出现多个线程修改同一变量的问题。
2. &nbsp;&nbsp;初始化加锁，保证更改成员变量的逻辑串行化。



## 打包方式
 从git上将代码拉到本地。本文xgboost拉取版本为0.90。
```shell
git clone https://github.com/dmlc/xgboost/tree/release_0.90
```

编译：
```shell
cd xgboost
mkdir build
cd build
cmake ..
make -j$(nproc)
```
进入jvm-packages中更新xgboost相关项目版本。
sample：
```shell
cd xgboost/jvm-packages/xgboost4j
vim pom.xml 
```
更改版本XXX
```xml
<groupId>ml.dmlc</groupId>
<artifactId>xgboost-jvm</artifactId>
<version>XXX</version>
```
打成jar包
```shell
mvn -DskipTests package #跳过测试
```

## 参考文章
&nbsp;&nbsp;&nbsp;&nbsp;Xgboost C++预测模块线程安全修复：
&nbsp;&nbsp;&nbsp;&nbsp;[https://blog.csdn.net/zc02051126/article/details/79427605](https://blog.csdn.net/zc02051126/article/details/79427605)
&nbsp;&nbsp;&nbsp;&nbsp;Java源码阅读之ReentrantLock - lockInterruptibly和tryLock方法
&nbsp;&nbsp;&nbsp;&nbsp;[https://cloud.tencent.com/developer/article/1195775](https://cloud.tencent.com/developer/article/1195775)