
<div align="center">

# Joint NER-RE on Adverse Drug Events related Datasets

<a href="https://pytorch.org/get-started/locally/"><img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-ee4c2c?logo=pytorch&logoColor=white"></a>
<a href="https://pytorchlightning.ai/"><img alt="Lightning" src="https://img.shields.io/badge/-Lightning-792ee5?logo=pytorchlightning&logoColor=white"></a>
<a href="https://hydra.cc/"><img alt="Config: Hydra" src="https://img.shields.io/badge/Config-Hydra-89b8cd"></a>
<a href="https://github.com/ChristophAlt/pytorch-ie-hydra-template"><img alt="Template" src="https://img.shields.io/badge/-PyTorch--IE--Hydra--Template-017F2F?style=flat&logo=github&labelColor=gray"></a><br>
[![Paper](http://img.shields.io/badge/paper-arxiv.1001.2234-B31B1B.svg)](https://academic.oup.com/jamia/article-abstract/27/1/3/5581277?redirectedFrom=fulltext)
</div>

## 📌 Description

This projects utilizes Pytorch-IE to perform NER, RE and joint NER-RE 
experiments on Adverse Drug Events related datasets such as the dataset 
introduced in 2018 N2C2 Shared Task Track 2.

## 🚀 Quickstart

### Environment Setup

```bash
# clone project
git clone https://github.com/Bhuvanesh-Verma/N2C2-ADE.git
cd N2C2-ADE

# [OPTIONAL] create conda environment
conda create -n your-project-name python=3.9
conda activate n2c2-ade

# install PyTorch according to instructions
# https://pytorch.org/get-started/

# install remaining requirements
pip install -r requirements.txt

# [OPTIONAL] symlink log directories and the default model directory to
# "$HOME/experiments/your-project-name" since they can grow a lot
bash setup_symlinks.sh $HOME/experiments/n2c2-ade

# [OPTIONAL] set any environment variables by creating an .env file
# 1. copy the provided example file:
cp .env.example .env
# 2. edit the .env file for your needs!
```

### Data Access

N2C2-ADE dataset can be made available on request from [DBMI Data Portal](https://portal.dbmi.hms.harvard.edu/projects/n2c2-nlp/).
After acquiring the access save the folder containing train, dev and test 
set as 'n2c2_ade.zip' in `data` folder of this project. You can change the name
of the zip file but that should also be changed in corresponding [dataset config](configs/dataset/n2c2_base.yaml).

```yaml
# configs/dataset/n2c2_base.yaml
_target_: src.utils.execute_pipeline
input:
  _target_: pie_datasets.DatasetDict.load_dataset
  path: pie/brat
  name: merge_fragmented_spans
  base_dataset_kwargs:
    url: "path/to/n2c2_ade.zip" <<----- CHANGE THE NAME OR PATH HERE
    file_name_blacklist:
      - "106967"
      - '110445'
      #- '115157'
    split_paths:
      train: 'n2c2_ade/train'
      test: 'n2c2_ade/test'

```

### Model Training

**Have a look into the [train.yaml](configs/train.yaml) config to see all available options.**

Train model with default configuration

```bash
# train on CPU
python src/train.py experiment=n2c2_ner monitor_metric=metric/span/macro/f1/train

# train on GPU
python src/train.py experiment=n2c2_ner monitor_metric=metric/span/macro/f1/train trainer=gpu
```

Execute a fast development run (train for two steps)

```bash
python src/train.py experiment=n2c2_ner monitor_metric=metric/span/macro/f1/train +trainer.fast_dev_run=true
```

Train model with different experiment configuration from [configs/experiment/](configs/experiment/)

```bash
python src/train.py monitor_metric=metric/span/macro/f1/train experiment=n2c2_re
```

### Model evaluation

This will evaluate the model on the test set of the chosen dataset using the *metrics implemented within the model*.
See [config/dataset/](configs/dataset/) for available datasets.

**Have a look into the [evaluate.yaml](configs/evaluate.yaml) config to see all available options.**

```bash
python src/evaluate.py dataset=n2c2_ner model_name_or_path=path/to/saved/model
```

Notes:

- add the command line parameter `trainer=gpu` to run on GPU

### Inference

This will run inference on the given dataset and split. See [config/dataset/](configs/dataset/) for available datasets.
The result documents including the predicted annotations will be stored in the `predictions/` directory (exact
location will be printed to the console).

**Have a look into the [predict.yaml](configs/predict.yaml) config to see all available options.**

```bash
python src/predict.py dataset=n2c2_ner model_name_or_path=path/to/n2c2-ner-model
```

Notes:

- add the command line parameter `+pipeline.device=0` to run the inference on GPU 0

### Evaluate Serialized Documents

This will evaluate serialized documents including predicted annotations (see [Inference](#inference)) using a
*document metric*. See [config/metric/](configs/metric/) for available metrics.

**Have a look into the [evaluate_documents.yaml](configs/evaluate_documents.yaml) config to see all available options**

```bash
python src/evaluate_documents.py metric=f1 metric.layer=entities +dataset.data_dir=PATH/TO/DIR/WITH/SPLITS
```

Note: By default, this utilizes the dataset provided by the
[from_serialized_documents](configs/dataset/from_serialized_documents.yaml) configuration. This configuration is
designed to facilitate the loading of serialized documents, as generated during the [Inference](#inference) step. It
requires to set the parameter `data_dir`. If you want to use a different dataset,
you can override the `dataset` parameter as usual with any existing dataset config, e.g `dataset=conll2003`. But
calculating the F1 score on the bare `conll2003` dataset does not make much sense, because it does not contain any
predictions. However, it could be used with statistical metrics such as
[count_text_tokens](configs/metric/count_text_tokens.yaml) or
[count_entity_labels](configs/metric/count_entity_labels.yaml).

## Development

```bash
# run pre-commit: code formatting, code analysis, static type checking, and more (see .pre-commit-config.yaml)
pre-commit run -a

# run tests
pytest -k "not slow" --cov --cov-report term-missing
```
