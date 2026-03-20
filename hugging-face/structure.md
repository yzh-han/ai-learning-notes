# Structure

一个完整的模型:

```bash
my_model/
├── configuration_mymodel.py   
│   # 定义配置类 class MyModelConfig(PretrainedConfig):
│   # 保存模型结构参数(hidden size, num_layers) -> config.json
│
├── modeling_mymodel.py
│   # 定义 模型本体 class MyModel(PreTrainedModel):
│   # model.save_pretrained("mymodel")
│
├── processing_mymodel.py
│   # processor.save_pretrained("toy-vlm")
├── image_processing_mymodel.py
├── tokenization_mymodel.py
│
├── __init__.py
└── convert_weights.py   # (optional)

# 完整 再加上
├── train.py
├── inference.py
└── utils.py
```

## 命名规则

```bash
# 文件名
configuration_<model>.py
modeling_<model>.py
processing_<model>.py
image_processing_<model>.py
tokenization_<model>.py

# 类名规则：
<Model>Config
<Model>Model
<Model>Processor
<Model>ImageProcessor
<Model>Tokenizer
```
