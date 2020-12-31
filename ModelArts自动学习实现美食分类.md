summary: ModelArts自动学习实现美食分类
id: ModelArts-AutoML-Food-Recong
categories: Primary
tags: AutoML
status: Published
authors: JamieXu
feedback link: http://www.huaweicloud.ai

# ModelArts自动学习实现美食分类
<!-- ---------------------- -->
## 基础环境准备
Duration: 10:00

在使用 ModelArts 进行 AI 开发前，需先完成以下基础操作哦（**如有已完成部分，请忽略**），主要分为4步（注册-->实名认证-->服务授权-->领代金券）：

1. 使用手机号注册华为云账号：[点击注册](https://reg.huaweicloud.com/registerui/cn/register.html?locale=zh-cn&service=https%3A%2F%2Fconsole.huaweicloud.com%2Fmodelarts%2F%3Fregion%3Dcn-north-4%26cloud_route_state%3D%2FloginIntro%3Fcode%3D7bI2Jqen#/register)


2. **点此去完成[实名认证](https://account.huaweicloud.com/usercenter/?service=https%3A%2F%2Fconsole.huaweicloud.com%2Fmodelarts%2F%3Fregion%3Dcn-north-4%26locale%3Dzh-cn%23%2Fdashboard#/accountindex/realNameAuth)**，账号类型选“个人”，个人认证类型推荐使用“扫码认证”。
 <img src="assets/garbage_recog/image-20201109154809347.png" alt="image-20201109154809347" style="zoom:50%;" />


3. **点此进入 [ModelArts 控制台数据管理页面](https://console.huaweicloud.com/modelarts/?region=cn-north-4#/dataset)**，上方会提示访问授权，点击【服务授权】按钮，按下图顺序操作：
    <img src="assets/basic/auth_1.png" alt="image-20201109153453542" style="zoom:50%;" />
    <img src="assets/basic/auth_2.png" alt="img" style="zoom:80%;" />    
  
    
4. **进入 [ModelArts 控制台首页](https://console.huaweicloud.com/modelarts/?region=cn-north-4#/loginIntro)**，如下图，点击页面上的“彩蛋”，领取新手福利代金券！**后续步骤可能会产生资源消耗费用，请务必领取**。
    <img src="assets/garbage_recog/image-20201109154327008.png" alt="image-20201109154327008" style="zoom:50%;" />

以上操作，我们也提供了详细的视频教程，点此查看：[ModelArts环境配置](https://bbs.huaweicloud.com/videos/102350)

<!-- ---------------------- -->
## 流程概览
Duration: 01:00

完成了基础环境准备后，我们就可以开始基于 ModelArts 开始我们的 AI 开发之旅，此次操作主要分为以下几个流程：

1. 下载数据集并上传到华为云对象存储服务（OBS）
2. 创建 ModelArts 自动学习项目并导入数据集
3. 完成数据标注并进行模型训练
4. 将模型部署成在线服务，进行服务调用并获得结果

接下来就让我们按照这个步骤开始吧！

<!-- ---------------------- -->
## 数据集下载及上传
Duration: 8:00

**点此下载所需美食数据集：[food-dataset](https://modelarts-case-data.obs.cn-north-4.myhuaweicloud.com:443/wuhan-food.zip?AccessKeyId=JL7I7G59X28R2YIRNMTV&Expires=1638203050&Signature=xn35C1ocRg5Ej%2Bj9NOHcE%2B/ReK4%3D)**

下载完成并解压后，可以得到两个文件夹：

* **train**: 训练用数据集，含4种美食，每一种10张图片，用于模型的训练
* **test**: 测试用数据集，含4种美食，每一种2张图片，用于模型训练完成后的测试

接下我们需要将下载的 **train文件夹** 的数据上传至华为云对象存储服务OBS。


### 3.1 创建OBS桶

<aside>OBS大家可以先简单的理解成一个在线网盘，因为ModelArts本身目前没有数据存储的功能，所以需要从OBS里调用我们上传的数据进行训练</aside>

点击进入：[华为云OBS控制台](https://storage.huaweicloud.com/obs/?region=cn-north-4&locale=zh-cn#/obs/manager/buckets)，进入后点击右上角的【创建桶】按钮（这里的桶可以理解成OBS进行存储的基本单位，所有的数据必须存储在某个桶里）：
![](./assets/wuhan_food_recog/2020-12-02-22-33-56.png)
进入新建桶界面， 按照如下示例进行填写：

* **区域**：华北-北京四
* **数据冗余存储策略**：单AZ存储
* **桶名称**：自定义，需要全局唯一，即在整个华为云上的名字唯一
* 其它选项保持默认即可

![](./assets/wuhan_food_recog/2020-12-02-22-36-50.png)
填写完成后，点击右下角的【立即创建】按钮并确认，稍等几秒钟即可完成 OBS桶 的创建。


### 3.2 上传训练数据至OBS
在OBS首页，找到我们刚刚新建的桶，并点击桶名称进入桶内容管理界面。
![](./assets/wuhan_food_recog/2020-12-02-22-48-01.png)

进入后，点击左侧【对象】按钮，进入数据上传界面，点击【上传对象】按钮，弹出上传对象框，**我们直接用鼠标将前面下载好的 train文件夹 拖拽到到上传对象框内**：
![](./assets/wuhan_food_recog/2020-12-02-23-00-23.png)

然后点击【上传】按钮即开始进行图片的上传，本次数据量较少，正常网络情况下约1分钟内即可完成数据的上传，
上传过程中可进入train文件夹查看进度，也可以继续进行下面的步骤。

**在train文件夹的同级目录，我们点击上面的【新建文件夹】按钮**，创建一个新的空文件夹，命名成`out`，完成后，效果如下图（此文件夹后续会用到）：
![](./assets/wuhan_food_recog/2020-12-03-16-25-04.png)


<!-- ---------------------- -->
## 自动学习项目创建
Duration: 5:00

点击访问 [ModelArts自动学习](https://console.huaweicloud.com/modelarts/?region=cn-north-4#/exeml) 页面，选择创建**图像分类**项目，进入项目创建设置页，按照如下示例进行填写：

* **名称**：自定义
* **数据集来源**：新建数据集
* **数据集输入位置**：选择上一步在OBS上传的train文件夹
* **数据集输出位置**：选择上一步在OBS创建的的out文件夹
* 其它保持默认即可

![](./assets/wuhan_food_recog/2020-12-03-16-29-19.png)

填写完成后点击右下角的【创建项目】按钮稍等几秒即可完成项目的创建。

## 数据标注及训练
Duration: 8:00

### 5.1 数据标注
项目创建完成后会自动进入数据标注页面，页面左侧是标注操作区，右边是标签编辑与展示区。

<aside>标签就是对数据集进行分类，比如本案例中的：热干面、鸭脖、豆皮、面窝，需我们自己根据数据集新建不同类型的标签；标注就是将数据集中的图片划归到对应的标签下，告诉模型这个图片里是什么</aside>

我们点击左侧的【未标注】页签，开始数据标注，我们选择同一类型的图片，然后在左侧的**标签名**出输入对应的美食名称并确定，即可对选择的图片做分类，如我们这里选择豆皮作为示例，效果如下：
![](./assets/wuhan_food_recog/2020-12-03-16-51-43.png)

![](./assets/wuhan_food_recog/2020-12-03-16-52-06.png)

我们重复上面的步骤，即可对不同类型的美食进行分类，也可以对某一标签进行图片追加。
<aside>注意：目前标注不支持翻页选择，翻页后上一页选择的数据会丢失，所以请大家一页一页的选择并确认，如果觉得一页展示的图片太少，可以在页面的左下角选择一页图片展示的个数。</aside>

全部标注完成后的效果如下：
![](./assets/wuhan_food_recog/2020-12-03-16-58-22.png)


如果有图片不小心标错了，只要在已标注里选择该选图片，此时可以在右侧看到此图片的被赋予的标签，点击【垃圾桶】按钮删除错误的标签，重新选择正确的标签即可。
![](./assets/wuhan_food_recog/2020-12-03-17-00-50.png)


### 5.2 模型训练
数据标注完成后，我们点击右上角的【开始训练】按钮，弹出模型训练的设置界面，按如下示例填写：

* **最大训练时长（分钟）** ：修改为10
* **计算规格**：选择自动学习免费规格（GPU），并勾选”我已阅读并同意以上内容“
* 其它保持默认即可

![](./assets/wuhan_food_recog/2020-12-03-17-12-23.png)
填写完成后，点击【下一步】进入二次确认界面，点击界面上的【提交】按钮即可提交成功。
![](./assets/wuhan_food_recog/2020-12-03-17-16-12.png)
训练过程约耗时2分钟左右，请耐心等待哦~

## 模型部署并调用
Duration: 5:00

### 6.1 模型部署
训练完成后，我们点击左侧的部署按钮，进行模型的部署操作，如下图示例。
![](./assets/wuhan_food_recog/2020-12-03-17-19-49.png)
![](./assets/wuhan_food_recog/2020-12-03-17-20-56.png)
点击【下一步】按钮，进入二次确认界面，点击界面上的【提交】按钮即可提交成功。
![](./assets/wuhan_food_recog/2020-12-03-17-23-44.png)

部署过程约耗时3分钟，请耐心等待哦~
### 6.2 模型调用
部署完成后，可以看到如下界面：
![](./assets/wuhan_food_recog/2020-12-03-17-29-24.png)
我们点击【上传】按钮，选择上传前面**第3步下载的数据集test文件夹**中的任一图片，然后点击【预测】即可得到结果：
![](./assets/wuhan_food_recog/2020-12-03-17-34-29.png)

<!-- ---------------------- -->
## 试验总结
Duration: 02:00

本实验到此完成，需要请大家注意：

* 因为本实验主要的目的是让大家了解 AI 开发的基本流程和使用 ModelArts 进行 AI 开发的基本操作，**为了减少操作的难度和时间长度，只用了少量的数据集用于训练，可能造成数据预测不是很准确的情况，请大家理解**。
* 示例中我们选择的都是免费规格，如果大家领取了新手代金券，可以尝试使用付费规格获得更好的体验，但记得及时关闭相应服务哦！

**Tips：实验结束后请及时停止在线服务，不然在线服务会持续收费有可能导致欠费，致使华为云账号被冻结而影响使用**。
我们在[在线服务管理页面](https://console.huaweicloud.com/modelarts/?region=cn-north-4&locale=zh-cn#/webservice/realTimeService)单击对应服务列表后的“停止”按钮即可停止本在线服务。

  <img src="assets/garbage_recog/image-20201109212031947.png" alt="image-20201110100520167" style="zoom:50%;" />
