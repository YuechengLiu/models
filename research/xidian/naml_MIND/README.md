# Contents

- [NAML Description](#NAML-description)
- [Dataset](#dataset)
- [Environment Requirements](#environment-requirements)
- [Script Description](#script-description)
    - [Script and Sample Code](#script-and-sample-code)
        - [Training Process](#training-process)
        - [Model Export](#model-export)
- [Model Description](#model-description)
    - [Performance](#performance)
        - [Evaluation Performance](#evaluation-performance)
        - [Inference Performance](#evaluation-performance)
- [Description of Random Situation](#description-of-random-situation)
- [ModelZoo Homepage](#modelzoo-homepage)

# [NAML Description](#contents)

NAML is a multi-view news recommendation approach. The core of NAML is a news encoder and a user encoder. The newsencoder is composed of a title encoder, a abstract encoder, a category encoder and a subcategory encoder. In the user encoder, we learn representations of users from their browsed news. Besides, we apply additive attention to learn more informative news and user representations by selecting important words and news.

[Paper](https://arxiv.org/abs/1907.05576) Chuhan Wu, Fangzhao Wu, Mingxiao An, Jianqiang Huang, Yongfeng Huang and Xing Xie: Neural News Recommendation with Attentive Multi-View Learning, IJCAI 2019

# [Dataset](#contents)

Dataset used: [MIND](https://msnews.github.io/). You can download [MINDlarge_train](https://mind201910small.blob.core.windows.net/release/MINDlarge_train.zip),
[MINDlarge_dev](https://mind201910small.blob.core.windows.net/release/MINDlarge_dev.zip),
[MINDlarge_utils](https://mind201910small.blob.core.windows.net/release/MINDlarge_utils.zip)

MIND contains about 160k English news articles and more than 15 million impression logs generated by 1 million users.

You can download the dataset and put the directory in structure as follows:

```path
└─MINDlarge
  ├─MINDlarge_train
  ├─MINDlarge_dev
  └─MINDlarge_utils
```

# [Environment Requirements](#contents)

- Hardware（Ascend/GPU）
    - Prepare hardware environment with Ascend, GPU processor.
- Framework
    - [MindSpore](https://www.mindspore.cn/install/en)
- For more information, please check the resources below：
    - [MindSpore Tutorials](https://www.mindspore.cn/tutorials/en/master/index.html)
    - [MindSpore Python API](https://www.mindspore.cn/docs/en/master/api_python/mindspore.html)

# [Script description](#contents)

## [Script and sample code](#contents)

```path
├── naml
  ├── README.md                    # descriptions about NAML
  ├── model_utils
  │   ├──__init__.py               # module init file
  │   ├──config.py                 # Parse arguments
  │   ├──device_adapter.py         # Device adapter for ModelArts
  │   ├──local_adapter.py          # Local adapter
  │   ├──moxing_adapter.py         # Moxing adapter for ModelArts
  ├── scripts
  │   ├──run_distribute_train.sh   # shell script for distribute training
  │   ├──run_train.sh              # shell script for training
  │   ├──run_eval.sh               # shell script for evaluation
  │   ├──run_infer_310.sh          # shell script for 310 inference
  ├── src
  │   ├──__init__.py               # module init file
  │   ├──callback.py               # callback file
  │   ├──dataset.py                # creating dataset
  │   ├──naml.py                   # NAML architecture
  │   ├──utils.py                  # utils to load ckpt_file for fine tune or incremental learn
  ├── MINDdemo_config.yaml         # Configurations for demo
  ├── MINDlarge_config.yaml        # Configurations for large
  ├── MINDsmall_config.yaml        # Configurations for small
  ├── MINDslarge_config_cpu.yaml   # Configurations for large on cpu
  ├── ascend310_infer              # application for 310 inference
  ├── train.py                     # training script
  ├── eval.py                      # evaluation script
  ├── export.py                    # export mindir script
  ├── preprocess.py                # preprocess input data
  └── postprocess.py               # post process for 310 inference
```

## [Training process](#contents)

### Usage

You can start training using python or shell scripts. The usage of shell scripts as follows:

- running on Ascend

    ```shell
    # train standalone
    bash run_train.sh [PLATFORM] [DEVICE_ID] [DATASET] [DATASET_PATH]
    # train distribute
    bash run_distribute_train.sh [PLATFORM] [DEVICE_NUM] [DATASET] [DATASET_PATH] [RANK_TABLE_FILE]
    # evaluation
    bash run_eval.sh [PLATFORM] [DEVICE_ID] [DATASET] [DATASET_PATH] [CHECKPOINT_PATH]
    ```

    - `PLATFORM` should be Ascend.
    - `DEVICE_ID` is the device id you want to run the network.
    - `DATASET` MIND dataset, support large, small and demo.
    - `DATASET_PATH` is the dataset path, the structure as [Dataset](#dataset).
    - `CHECKPOINT_PATH` is a pre-trained checkpoint path.
    - `RANK_TABLE_FILE` is HCCL configuration file when running on Ascend.

    For distributed training, a hccl configuration file with JSON format needs to be created in advance.

    Please follow the instructions in the link below:

    <https://gitee.com/mindspore/models/tree/master/utils/hccl_tools>.

- running on CPU

    ```shell
    # train using python
    python train.py --config_path=[CONFIG_PATH]
                    --platform=[PLATFORM]
                    --dataset=[DATASET]
                    --dataset_path=[DATASET_PATH]
                    --save_checkpoint_path=[SAVE_CHECKPOINT_PATH]
                    --weight_decay=False
                    --sink_mode=False
    # example
    python train.py --config_path=MINDlarge_config_cpu.yaml --platform=CPU --dataset=large --dataset_path=MINDlarge --save_checkpoint_path=./script/checkpoint --weight_decay=False --sink_mode=False
    # evaluation using python
    python eval.py --config_path=[CONFIG_PATH]
                    --platform=[PLATFORM]
                    --dataset=[DATASET]
                    --dataset_path=[DATASET_PATH]
                    --checkpoint_path=[CHECKPOINT_PATH]
    # example
    python eval.py --config_path=MINDlarge_config_cpu.yaml --platform=CPU --dataset=large --dataset_path=MINDlarge --checkpoint_path=./script/checkpoint/naml_last.ckpt
    ```

    - `PLATFORM` should be CPU.
    - `DEVICE_ID` is the device id you want to run the network.
    - `DATASET` MIND dataset, support large.
    - `DATASET_PATH` is the dataset path, the structure as [Dataset](#dataset).
    - `SAVE_CHECKPOINT_PATH` is a pre-trained checkpoint path to save.
    - `CHECKPOINT_PATH` is a pre-trained checkpoint path.

- ModelArts (If you want to run in modelarts, please check the official documentation of [modelarts](https://support.huaweicloud.com/modelarts/), and you can start training as follows)

    - Train large dataset 1p/8p on ModelArts

      ```python
      # (1) Add "config_path='/path_to_code/MINDlarge_config.yaml'" on the website UI interface.
      # (2) Perform a or b.
      #       a. Set "enable_modelarts=True" on MINDlarge_config.yaml file.
      #          Set "platform=Ascend" on MINDlarge_config.yaml file.
      #          Set "dataset='large'" on MINDlarge_config.yaml file.
      #          Set "dataset_path='/cache/data/MINDlarge'" on MINDlarge_config.yaml file.
      #          Set "save_checkpoint_path='./checkpoint'" on MINDlarge_config.yaml file.
      #          Set "weight_decay=False" on MINDlarge_config.yaml file.
      #          Set "sink_mode=True" on MINDlarge_config.yaml file.
      #          Set other parameters on MINDlarge_config.yaml file you need.
      #       b. Add "enable_modelarts=True" on the website UI interface.
      #          Add "platform=Ascend" on the website UI interface.
      #          Add "dataset=large" on the website UI interface.
      #          Add "dataset_path=/cache/data/MINDlarge" on the website UI interface.
      #          Add "save_checkpoint_path=./checkpoint" on the website UI interface.
      #          Add "weight_decay=False" on the website UI interface.
      #          Add "sink_mode=True" on the website UI interface.
      #          Add other parameters on the website UI interface.
      # (3) Upload dataset to S3 bucket.
      # (4) Set the code directory to "/path/naml" on the website UI interface.
      # (5) Set the startup file to "train.py" on the website UI interface.
      # (6) Set the "Dataset path" and "Output file path" and "Job log path" to your path on the website UI interface.
      # (7) Create your job.
      ```

    - Eval large dataset 1p on ModelArts

      ```python
      # (1) Add "config_path='/path_to_code/MINDlarge_config.yaml'" on the website UI interface.
      # (2) Perform a or b.
      #       a. Set "enable_modelarts=True" on MINDlarge_config.yaml file.
      #          Set "platform=Ascend" on MINDlarge_config.yaml file.
      #          Set "dataset='large'" on MINDlarge_config.yaml file.
      #          Set "dataset_path='/cache/data/MINDlarge'" on MINDlarge_config.yaml file.
      #          Set "checkpoint_url='s3://dir_to_trained_ckpt/'" on MINDlarge_config.yaml file.
      #          Set "checkpoint_path='/cache/checkpoint_path/model.ckpt'" on MINDlarge_config.yaml file.
      #          Set other parameters on MINDlarge_config.yaml file you need.
      #       b. Add "enable_modelarts=True" on the website UI interface.
      #          Add "platform=Ascend" on the website UI interface.
      #          Add "dataset=large" on the website UI interface.
      #          Add "dataset_path=/cache/data/MINDlarge" on the website UI interface.
      #          Add "checkpoint_url=s3://dir_to_trained_ckpt/" on the website UI interface.
      #          Add "checkpoint_path=/cache/checkpoint_path/model.ckpt" on the website UI interface.
      #          Add other parameters on the website UI interface.
      # (3) Upload or copy your trained model to S3 bucket.
      # (4) Upload dataset to S3 bucket.
      # (5) Set the code directory to "/path/naml" on the website UI interface.
      # (6) Set the startup file to "eval.py" on the website UI interface.
      # (7) Set the "Dataset path" and "Output file path" and "Job log path" to your path on the website UI interface.
      # (8) Create your job.
      ```

    - Export 1p on ModelArts

      ```python
      # (1) Add "config_path='/path_to_code/MINDlarge_config.yaml'" on the website UI interface.
      # (2) Perform a or b.
      #       a. Set "enable_modelarts=True" on MINDlarge_config.yaml file.
      #          Set "platform=Ascend" on MINDlarge_config.yaml file.
      #          Set "file_format='MINDIR'" on MINDlarge_config.yaml file.
      #          Set "batch_size=1" on MINDlarge_config.yaml file.
      #          Set "checkpoint_url='s3://dir_to_trained_ckpt/'" on MINDlarge_config.yaml file.
      #          Set "checkpoint_path='/cache/checkpoint_path/model.ckpt'" on MINDlarge_config.yaml file.
      #          Set other parameters on MINDlarge_config.yaml file you need.
      #       b. Add "enable_modelarts=True" on the website UI interface.
      #          Add "platform=Ascend" on the website UI interface.
      #          Add "file_format=MINDIR" on the website UI interface.
      #          Add "batch_size=1" on the website UI interface.
      #          Add "checkpoint_url=s3://dir_to_trained_ckpt/" on the website UI interface.
      #          Add "checkpoint_path=/cache/checkpoint_path/model.ckpt" on the website UI interface.
      #          Add other parameters on the website UI interface.
      # (3) Upload or copy your trained model to S3 bucket.
      # (4) Set the code directory to "/path/naml" on the website UI interface.
      # (5) Set the startup file to "export.py" on the website UI interface.
      # (6) Set the "Dataset path" and "Output file path" and "Job log path" to your path on the website UI interface.
      # (7) Create your job.
      ```


## [Infer on Ascend310](#contents)

**Before inference, please refer to [MindSpore Inference with C++ Deployment Guide](https://gitee.com/mindspore/models/blob/master/utils/cpp_infer/README.md) to set environment variables.**

```shell
# Ascend310 inference
bash run_infer_310.sh [NEWS_MODEL] [USER_MODEL] [DEVICE_ID]
```

- `NEWS_MODEL` specifies path of news "MINDIR" OR "AIR" model.
- `USER_MODEL` specifies path of user "MINDIR" OR "AIR" model.
- `DEVICE_ID` is optional, default value is 0.

# [Model Description](#contents)

## [Performance](#contents)

### Evaluation Performance

| Parameters                 | Ascend                                                       |
| -------------------------- | ------------------------------------------------------------ |
| Model Version              | NAML                                                         |
| Resource                   | Ascend 910; CPU 2.60GHz, 56cores; Memory 314G; OS Euler2.8   |
| uploaded Date              | 02/23/2021 (month/day/year)                                  |
| MindSpore Version          | 1.2.0                                                        |
| Dataset                    | MINDlarge                                                    |
| Training Parameters        | epoch=1, steps=52869, batch_size=64, lr=0.001                |
| Optimizer                  | Adam                                                         |
| Loss Function              | Softmax Cross Entropy                                        |
| outputs                    | probability                                                  |
| Speed                      | 1pc: 62 ms/step                                              |
| Total time                 | 1pc: 54 mins                                                 |

### Inference Performance

| Parameters          | Ascend                      |
| ------------------- | --------------------------- |
| Model Version       | NAML                        |
| Resource            | Ascend 910; OS Euler2.8     |
| Uploaded Date       | 02/23/2021 (month/day/year) |
| MindSpore Version   | 1.2.0                       |  
| Dataset             | MINDlarge                   |
| batch_size          | 64                          |
| outputs             | probability                 |
| Accuracy            | AUC: 0.66                   |

### Inference on Ascend310 Performance

| Parameters          | Ascend                      |
| ------------------- | --------------------------- |
| Model Version       | NAML                        |
| Resource            | Ascend 310                  |
| Uploaded Date       | 12/10/2021 (month/day/year) |
| MindSpore Version   | 1.6.0                       |  
| Dataset             | MINDlarge                   |
| batch_size          | 16                          |
| outputs             | probability                 |
| Accuracy            | AUC: 0.6669                 |

### Inference on CPU Performance

| Parameters        | CPU                       |
| ----------------- | ------------------------- |
| Model Version     | NAML                      |
| Resource          | CPU                       |
| Uploaded Date     | 8/9/2022 (month/day/year) |
| MindSpore Version | 1.8                       |
| Dataset           | MINDlarge                 |
| batch_size        | 64                        |
| outputs           | probability               |
| Accuracy          | AUC: 0.6727               |

# [Description of Random Situation](#contents)

<!-- In dataset.py, we set the seed inside “create_dataset" function. We also use random seed in train.py. -->
In train.py, we set the seed which is used by numpy.random, mindspore.common.Initializer, mindspore.ops.composite.random_ops and mindspore.nn.probability.distribution.

# [ModelZoo Homepage](#contents)

Note: This model will be move to the `/models/research/` directory in r1.8.

Please check the official [homepage](https://gitee.com/mindspore/models).

# [在NPU上进行训练和推理的命令](#contents)
#### bash run_train.sh Ascend 0 large ../MINDlarge
#### bash run_eval.sh Ascend 0 large ../MINDlarge checkpoint/naml-52_1000.ckpt