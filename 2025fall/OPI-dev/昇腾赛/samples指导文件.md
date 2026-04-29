## 模型在线推理

本章节将介绍如何在OrangePi AIpro（下称：香橙派开发板）下载昇思MindSpore在线推理案例，并启动Jupyter Lab界面执行推理。

### 1. 下载案例

步骤1 下载案例代码。

```bash
# 打开开发板的一个终端，运行如下命令
cd samples/notebooks/
git clone https://github.com/mindspore-courses/orange-pi-mindspore.git
```

步骤2 进入案例目录。

下载的代码包在香橙派开发板的如下目录中：/home/HwHiAiUser/samples/notebooks。

项目目录如下：

```bash
/home/HwHiAiUser/samples/notebooks/orange-pi-mindspore/tutorial/
01-dev_start
02-ResNet50
03-ViT
04-FCN
05-Shufflenet
06-SSD
07-RNN
08-LSTM+CRF
09-GAN
10-DCGAN
11-Pix2Pix
12-Diffusion
13-ResNet50_transfer
14-qwen1.5-0.5b
15-tinyllama
```

### 2. 推理执行（案例01-13）

步骤1 启动Jupyter Lab界面。

```bash
cd /home/HwHiAiUser/orange-pi-mindspore/
./start_notebook.sh
```

在执行该脚本后，终端会出现如下打印信息，在打印信息中会有登录Jupyter Lab的网址链接。

![model-infer1](https://mindspore-courses.obs.cn-north-4.myhuaweicloud.com/orange-pi-online-infer/images/model_infer1.png)

然后打开浏览器。

![model-infer2](https://mindspore-courses.obs.cn-north-4.myhuaweicloud.com/orange-pi-online-infer/images/model_infer2.png)

再在浏览器中输入上面看到的网址链接，就可以登录Jupyter Lab软件了。

![model-infer3](https://mindspore-courses.obs.cn-north-4.myhuaweicloud.com/orange-pi-online-infer/images/model_infer3.png)

步骤2 在Jupyter Lab界面双击下图所示的案例目录，此处以“04-FCN”为例，即可进入到该案例的目录中。

![model-infer4](https://mindspore-courses.obs.cn-north-4.myhuaweicloud.com/orange-pi-online-infer/images/model_infer4.png)

步骤3 在该目录下有运行该示例的所有资源，其中mindspore_fcn8s.ipynb是在Jupyter Lab中运行该样例的文件，双击打开mindspore_fcn8s.ipynb，在右侧窗口中会显示。mindspore_fcn8s.ipynb文件中的内容，如下图所示：

![model-infer5](https://mindspore-courses.obs.cn-north-4.myhuaweicloud.com/orange-pi-online-infer/images/model_infer5.png)

步骤4 单击⏩按钮运行样例，在弹出的对话框中单击“Restart”按钮，此时该样例开始运行。

![model-infer6](https://mindspore-courses.obs.cn-north-4.myhuaweicloud.com/orange-pi-online-infer/images/model_infer6.png)

**注：如遇报错“线程同步失败”，请尝试关闭swap。执行sudo swapoff /swapfile命令。**

### 3. 推理执行（案例14、15）
**注：此处推荐使用aipro20T 24G版本，aipro8T 16G也可运行，但请注意剩余内存。暂不支持小于16G内存的aipro运行这两个案例。**

步骤1 进入案例目录，以14-qwen1.5-0.5b为例，15号案例类似。

```bash
cd /home/HwHiAiUser/orange-pi-mindspore/Online/14-qwen1.5-0.5b
```

**注：首次启动时会自动从镜像站下载模型，如遇网络原因无法下载，可以通过电脑下载后上传到案例目录下的.mindnlp文件夹内的对应目录下。**

步骤2 将模型路径修改为本地模型存放的路径（可选）

```bash
vim qwen1.5-0.5b.py
# 找到以下两行
tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen1.5-0.5B-Chat", ms_dtype=mindspore.float16)
model = AutoModelForCausalLM.from_pretrained("Qwen/Qwen1.5-0.5B-Chat", ms_dtype=mindspore.float16)
# 修改为
tokenizer = AutoTokenizer.from_pretrained("/home/HwHiAiUser/orange-pi-mindspore-master/Online/14-qwen1.5-0.5b/.mindnlp/model/Qwen/Qwen1.5-0.5B-Chat", ms_dtype=mindspore.float16)
model = AutoModelForCausalLM.from_pretrained("/home/HwHiAiUser/orange-pi-mindspore-master/Online/14-qwen1.5-0.5b/.mindnlp/model/Qwen/Qwen1.5-0.5B-Chat", ms_dtype=mindspore.float16)
```

步骤3 启动推理程序

```bash
python3 qwen1.5-0.5b.py
```
