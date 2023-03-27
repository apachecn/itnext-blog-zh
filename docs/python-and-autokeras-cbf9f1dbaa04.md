# Python 和 AutoKeras

> 原文：<https://itnext.io/python-and-autokeras-cbf9f1dbaa04?source=collection_archive---------4----------------------->

![](img/a096813b7b9dcdb1edd97cd8c1f658db.png)

AutoML 是机器学习行业中一个有趣的领域，有望加快模型生成周期。最近我一直在与 Tensroflow 和 Keras 合作一个深度学习项目。出于纯粹的好奇和总是学习的目的，我决定尝试自动化深度学习，更具体地说是 AutoKeras。

这篇文章背后的动机是因为关于这个主题的资源很少。开始时，我只搜索了一些信息，在这些信息中我发现了非常少的演练，而且没有一个是最新的。也就是说，本文的范围是 AutoKeras 的一个基本用例以及一个文件结构布局..ofc GitHub repo 包括:d .一些关于 Tensorflow 和 Keras 的先验知识是有帮助的。

# 安装

*公平警告:*我已经尝试使用所有软件包的最新版本，然而，在撰写本文时，我遇到了与 Tensorflow==2.3.1 和 AutoKeras==1.0.10 - >降级到 AutoKeras==1.0.8 的兼容性问题。

# 装置

需要做的第一件事是在您打算进行项目的目录中设置一个 python 虚拟环境。Python 为虚拟环境提供了一个非常简单的命令。python 版本将与路径目录中安装的版本相同:

```
python -m venv DL_Env
DL_Env\Scripts\activate.bat # Windows activate
source DL_Env/bin/activate # Linux activate
```

激活虚拟环境后，下一步是安装所需的软件包。我们只需要 Tensorflow==2.3.1 和 AutoKeras==1.0.8，这些也可以在 requirements.txt 文件中的 GitHub Repo 中找到。

```
pip install Tensorflow==2.3.1
pip install AutoKeras=1.0.8
pip install git+https://github.com/keras-team/keras-tuner.git@1.0.2rc4
```

# 文件结构

安装完所有合适的包后，我们可以进入另一个非常重要的步骤，那就是文件结构。自动机器学习将根据脚本中的用户定义生成大量模型和模型的大量信息。有一个好的文件结构布局是很重要的，它既直观又容易理解。就我个人而言，我没有找到任何关于什么是最好的结构的信息，所以在这里我将分享一个自己制作的结构，它将完成工作，并从个人经验中证明是有效的。

```
+-- Project_Folder
|   +-- DL_Env
|   +-- Datasets
|   +-- Logs
|   +-- Models
|   +-- Projects
|   +-- Datasets
|   +-- Tensorboard
|   +-- main.py
```

# AutoKeras

整个过程将包含在一个脚本-> main.py 中。首先，我们应该导入和加载数据集。我们要解决的问题是一个结构化数据分类器，这个任务的数据集将是一个巨大的数据集。titanic 数据集是故意选择的，因为它是数据科学社区中众所周知的，因此将提供 AutoKeras 如何处理它的一种参考点。

```
import tensorflow as tf
import autokeras as ak
import kerastuner
​
TRAIN_DATA_URL = "https://storage.googleapis.com/tf-datasets/titanic/train.csv"
TEST_DATA_URL = "https://storage.googleapis.com/tf-datasets/titanic/eval.csv"
​
train_file_path = tf.keras.utils.get_file("train.csv", TRAIN_DATA_URL)
test_file_path = tf.keras.utils.get_file("eval.csv", TEST_DATA_URL)
​
tensorboard_callback_train = tf.keras.callbacks.TensorBoard(log_dir='Tensorboard//logs_autokeras_f1_score_10%')
tensorboard_callback_test = tf.keras.callbacks.TensorBoard(log_dir='Tensorboard//logs_autokeras_f1_score_10%')
​
Early_Stopping = tf.keras.callbacks.EarlyStopping(monitor='val_loss', patience=101)
```

下一步是调用我们希望 AutoKeras 使用的分类器类型。由于我们正在处理结构化数据，并且存在分类问题，因此我们必须使用 AutoKeras 提供的结构化数据分类器对象。

```
# Initialize the structured data classifier.
clf = ak.StructuredDataClassifier(
    project_name = "Projects/First_Run",
    objective=kerastuner.Objective('accuracy', direction='max'),
    metrics=["accuracy"],
    overwrite=True,
    max_trials=1001)
```

因此，让我们回顾一下每个参数。项目名称参数将在我们之前创建的子目录“项目”中生成一个文件夹。该文件夹将包含该过程创建的所有试验，换句话说就是模型。覆盖真意味着清除文件夹中的所有试验，并保存新的运行。在目标参数中，我们可以定义 AutoKeras 应该优先考虑哪个度量，以及 AutoKeras 在神经架构搜索期间应该最大化还是最小化该度量。您可以添加自定义指标，如 F1、F0.5、AUC 等。相同的指标可以放入“指标”参数，该参数将在培训和评估期间报告。最后，max_trials 参数告诉 AutoKeras 要生成多少不同超参数组合的模型或试验。

既然我们已经定义了分类器及其参数，我们就可以开始这个过程了:

```
clf.fit(
    train_file_path # or  train_x and train_y
    epochs=151,
    verbose=2,
    callbacks=[tensorboard_callback_train, Early_Stopping],
    batch_size=1001
    )
```

为了开始自动化的深度学习过程，我们简单地调用 fit 方法并输入训练数据。此外，我们还会传入一些参数，如每次试验将训练的次数、输出级别、我们的回调(在本例中，回调会产生具有提前停止功能的 tensorboard 信息),以及最终的批量大小。

训练结束后，我们可以导出表现最佳的模型。然后，我们可以将其导出、保存并评估其性能:

```
clf_best_model = clf.export_model()
clf_best_model.save("Models/First_Run", save_format="tf")
print(accuracy=clf.evaluate(test_file_path, "survived"))
```

如果打开另一个脚本，您可以加载模型并使用它进行预测，而无需导入 AutoKeras:

```
from tensorflow.keras.models import load_model
​
Custom_Objects = ak.CUSTOM_OBJECTS
​
# If you have custom metrics add each metric accordingly
# Custom_Objects["f1_score"] = f1_score
​
loaded_model = load_model("Models/First_Run", custom_objects=Custom_Objects)
​
print(loaded_model.evaluate(test_file_path, "survived"))
```

# 结论

这就结束了一个通过 AutoKeras 解决的简单 AutoML 问题。确保以下列方式运行脚本，以便保存日志文件:

```
python main.py > Logs/First_Run.txt
```

谢谢你走到这一步😄

GitHub:[https://github.com/ervin007/Autokeras_And_Python](https://github.com/ervin007/Autokeras_And_Python)