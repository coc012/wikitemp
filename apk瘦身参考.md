### 一. 扯淡
### 二.瘦身操作参考流程

#### 1. 去除冗余文件
##### 1. 去除不再使用业务模块文件java、xml

##### 2.使用remove unused文件

##### 3.除去多余的so 、jar包

##### 4. 去掉重复依赖库 


#### 2. 压缩资源文件

##### 1. 布局图片无损压缩
##### 2. 非布局图片 有损压缩
##### 3. webp(考虑在api17以后引入)

#### 3. 代码混淆
#### 4. 资源混淆
> 现阶段常见的资源混淆方案是，微信团队开源项目[AndResGuard](https://github.com/shwenzhang/AndResGuard)，
由于AndResGuard 采用7zip处理， v2签名会使得7zip压缩失效。目前采用的美团渠道打包方案中启用了v2，由于AndResGuard暂不可用。
   


### 三.整体说明

### 四.参考文档
1.[美团 Android App包瘦身优化实践](https://tech.meituan.com/android-shrink-overall-solution.html)