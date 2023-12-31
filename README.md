# 基于Paddle复现《Dual Residual Attention Network for Image Denoising》

## 1.简介

In image denoising, deep convolutional neural networks (CNNs) can obtain favorable performance on removing spatially invariant noise. However, many of these networks cannot perform well on removing the real noise (i.e. spatially variant noise) generated during image acquisition or transmission, which severely sets back their application in practical image denoising tasks. Instead of continuously increasing the network depth, many researchers have revealed that expanding the width of networks can also be a useful way to improve model performance. It also has been verified that feature filtering can promote the learning ability of the models. Therefore, in this paper, we propose a novel Dual-branch Residual Attention Network (DRANet) for image denoising, which has both the merits of a wide model architecture and attention-guided feature learning. The proposed DRANet includes two different parallel branches, which can capture complementary features to enhance the learning ability of the model. We designed a new residual attention block (RAB) and a novel hybrid dilated residual attention block (HDRAB) for the upper and the lower branches, respectively. The RAB and HDRAB can capture rich local features through multiple skip connections between different convolutional layers, and the unimportant features are dropped by the residual attention modules. Meanwhile, the long skip connections in each branch, and the global feature fusion between the two parallel branches can capture the global features as well. Moreover, the proposed DRANet uses downsampling operations and dilated convolutions to increase the size of the receptive field, which can enable DRANet to capture more image context information. Extensive experiments demonstrate that compared with other state-of-the-art denoising methods, our DRANet can produce competitive denoising performance both on synthetic and real-world noise removal.


## 2.复现精度

SIDD添加高斯噪声

| Network | opt   | iters  | learning rate | batch_size | dataset | GPUS | PSNR    |
| ------- | ----- | ------ | ------------- | ---------- | ------- | ---- | ------- |
| DRANet  | AdamW | 500000 | 1.5e-4        | 8          | SIDD    | 1    | 40.23 |


## 3.数据集

下载地址:

[https://www.eecs.yorku.ca/~kamel/sidd/](https://www.eecs.yorku.ca/~kamel/sidd/)


最优权重:

链接：https://pan.baidu.com/s/1aBXFV6801n1GTevbaQv4Ow?pwd=hh66 
提取码：hh66



## 4.环境依赖

PaddlePaddle == 2.2.0

scikit-image == 0.19.2

## 5.快速开始

### 模型训练

多卡训练，启动方式如下：

```shell
python -u -m paddle.distributed.launch  train.py -opt configs/GaussianColorDenoising_DRANet.yml 
```

多卡恢复训练，启动方式如下：

```shell
python -u -m paddle.distributed.launch  train.py -opt configs/GaussianColorDenoising_DRANet.yml --resume ../245_model
```

参数介绍：

opt: 配置路径

resume: 从哪个模型开始恢复训练，需要pdparams和pdopt文件。


### 模型验证

除了可以再训练过程中验证模型精度，还可以是val.py脚本加载模型验证精度，执行以下命令。
验证数据的地址需要设置configs/GaussianColorDenoising_DRANet.yml中的datasets.val.dataroot_gt参数。

```shell
python val.py -opt configs/GaussianColorDenoising_DRANet.yml --weights output/model/best_model.pdparams --sigmas 15 
```

[Eval] PSNR: 40.23

```
参数说明：

opt: 配置路径

weights: 模型权重地址

sigmas: 噪声等级

### 单张图片预测
本项目提供了单张图片的预测脚本，可根据输入图片生成噪声，然后对图片进行降噪。会在result_dir指定的目录下生成denoise_0000.png和noise_0000.png两张图片。使用方法如下：
​```shell
python predict.py --input_images demo/0000.png \
--weights best_model.pdparams \
--model_type blind --sigmas 15 --result_dir ./output/
```

参数说明：

input_images:需要预测的图片

weights: 模型路径

result_dir: 输出图片保存路径

model_type: 模型类型，本项目只训练了blind模式。

sigmas: 噪声等级。



### 模型导出

模型导出可执行以下命令：

```shell
python export_model.py -opt ./test_tipc/configs/GaussianColorDenoising_DRANet.yml --model_path ./output/model/last_model.pdparams --save_dir ./test_tipc/output/
```

参数说明：

opt: 模型配置路径

model_path: 模型路径

save_dir: 输出图片保存路径

### Inference推理

可使用以下命令进行模型推理。该脚本依赖auto_log, 请参考下面TIPC部分先安装auto_log。infer命令运行如下：

```shell
python infer.py
--use_gpu=False --enable_mkldnn=False --cpu_threads=2 --model_file=./test_tipc/output/model.pdmodel --batch_size=2 --input_file=../data/SIDD --enable_benchmark=True --precision=fp32 --params_file=.output/best_model.pdiparams 
```

参数说明:

use_gpu:是否使用GPU

enable_mkldnn:是否使用mkldnn

cpu_threads: cpu线程数

model_file: 模型路径

batch_size: 批次大小

input_file: 输入文件路径

enable_benchmark: 是否开启benchmark

precision: 运算精度

params_file: 模型权重文件，由export_model.py脚本导出。



### TIPC基础链条测试

该部分依赖auto_log，需要进行安装，安装方式如下：

auto_log的详细介绍参考[https://github.com/LDOUBLEV/AutoLog](https://github.com/LDOUBLEV/AutoLog)。

```shell
git clone https://gitee.com/Double_V/AutoLog
cd AutoLog/
pip3 install -r requirements.txt
python3 setup.py bdist_wheel
pip3 install ./dist/auto_log-1.2.0-py3-none-any.whl
```


```shell
bash test_tipc/prepare.sh ./test_tipc/configs/Restormer/train_infer_python.txt 'lite_train_lite_infer'

bash test_tipc/test_train_inference_python.sh ./test_tipc/configs/Restormer/train_infer_python.txt 'lite_train_lite_infer'
```


## 6.代码结构与详细说明

```
Restormer_Paddle
├── README.md # 说明文件
├── logs # 训练日志
├── configs # 配置文件
├── data # 数据变换
├── dataset.py # 数据集路径
├── demo # 样例图片
├── export_model.py # 模型导出
├── infer.py  # 推理预测
├── metrics  # 指标计算方法
├── models # 网络模型
├── predict.py # 图像预测
├── test_tipc # TIPC测试链条
├── train.py # 训练脚本
├── utils # 工具类
└── val.py # 评估脚本

```

## 7.模型信息

| 信息     | 描述                |
| -------- | ------------------- |
| 模型名称 | DRANet              |
| 框架版本 | PaddlePaddle==2.2.0 |
| 应用场景 | 降噪                |

