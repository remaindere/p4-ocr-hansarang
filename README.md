# Pstage 4 – 수식인식기 – 한사랑개발회

<img src="https://user-images.githubusercontent.com/43458619/122523967-8e67e000-d052-11eb-94d2-01cae3d3e3e2.PNG" width="60%" height="50%">


수식인식기는 OCR(Optical Character Recognition) task의 일환으로써 수식 이미지 latex 포맷의 텍스트로 변환하는 task입니다. 수식 인식의 경우, 기존의 OCR과 달리 multi line recogniton을 필요로 한다는 점에서 기존의 single line recognition OCR task와 차별점을 가집니다.

저희 수식인식기 task는 SATRN model을 기본 틀로 설정하여 수식 이미지 학습을 진행했습니다. Multi line recognition 문제 이외에도 다른 문제를 해결하기 위해 기존 model에 개선 사항을 적용했습니다. 추가적으로 streamlit를 이용해 간단한 웹 데모를 만듦으로써 저희가 만든 모델을 서비스로 출시할 수 있음을 보이고자 했습니다.

:page_with_curl:[Wiki](https://github.com/bcaitech1/p4-ocr-hansarang/wiki)에 문제 인식, EDA, 모델 설계, 실험 관리 및 검증 전략 등 저희가 고안한 기술의 흐름과 고민의 흔적을 기록했습니다.

 

# Table of Contents

* [Demo](#Demo)
* [Getting started](#Getting-started)
  * [Dependencies](#Dependencies)
  * [Installation](#Installatio)
  * [Train](#Train)
  * [Inference](#Inference)

* [File Structure](#File-Structure)
  * [Baseline code](#Baseline-code)
  * [Input](#Input)
* [Yaml file example](#Yaml-file-example)
* [Contributors](Contributors)
* [Reference](Reference)
* [Licence](Licence)

## Demo

~~[[Click here!]Demo link]~~
현재는 데모 서비스를 지원하지 않습니다.

아래의 링크에서 Demo의 세부 사항을 확인하실 수 있습니다.

* [streamlit_test](https://github.com/bcaitech1/p4-ocr-hansarang/tree/main/streamlit_test)

<img src = "https://user-images.githubusercontent.com/43458619/122248828-54d48f00-cf03-11eb-81cd-d3ba2aa3dd2a.png" width="50%" height="50%">



## Getting started

### Dependencies
- scikit_image==0.14.1
- opencv_python==3.4.4.19
- tqdm==4.28.1
- torch==1.4.0
- scipy==1.2.0
- numpy==1.15.4
- torchvision==0.2.1
- Pillow==8.1.1
- tensorboardX==1.5
- editdistance==0.5.3
- opencv-python==3.4.4.19
- adamp==0.3.0 
- madgrad==1.1 
- timm==0.4.9
- wandb==0.10.31

### Installation

```
$ git clone https://github.com/bcaitech1/p4-ocr-hansarang.git
$ cd code 
```

### Train

```
$ python train.py --config_file ./configs/SATRN.yaml
```

### Inference

```
$ python inference.py --config_file ./log/satrn/checkpoints/0050.pth
```



## File Structure

### Baseline code

```
code
├── README.md
├── __pycache__
│   ├── checkpoint.cpython-37.pyc
│   ├── dataset.cpython-37.pyc
│   ├── flags.cpython-37.pyc
│   ├── metrics.cpython-37.pyc
│   ├── scheduler.cpython-37.pyc
│   ├── train.cpython-37.pyc
│   └── utils.cpython-37.pyc
├── checkpoint.py
├── configs - arguments config files
│   ├── Attention.yaml
│   └── SATRN.yaml
├── data_tools
│   ├── extract_tokens.py
│   ├── parse_upstage.py
│   └── train_test_split.py
├── dataset.py - make dataset objects
├── download.sh
├── flags.py - make flag object from config
├── inference.py - do inference
├── log
├── metrics.py
├── networks - structure of models
│   ├── Attention.py
│   ├── Mish.py
│   ├── SATRN.py
│   ├── SATRN_Effnet.py
│   ├── SATRN_Mish.py
│   ├── Swish.py
│   ├── __pycache__
│   │   ├── Attention.cpython-37.pyc
│   │   ├── SATRN.cpython-37.pyc
│   │   └── SATRN_Effnet.cpython-37.pyc
│   └── spatial_transformation.py
├── requirements.txt
├── scheduler.py - set of scheduler function
├── submission.txt
├── submit
│   └── output.csv
├── train.py - do train
├── utils.py - setting for model, optimizer by config file
```



### Input

```
input
└── data
    ├── eval_dataset - dataset for evaluation
    │   ├── images
    │   └── input.txt
    └── train_dataset - dataset for train
        ├── gt.txt - ground truth file
        ├── images - image files
        ├── level.txt - level for each images
        ├── source.txt - handwrite image(value : 1) or printed image(value : 0)
        └── tokens.txt - all tokens using for description of operations
```



## Yaml file example

모델 학습에 사용할 때 사용하는 config 파일(ex. SATRN.yaml) 예시입니다.

예시 config 파일들은 [code/configs]()(경로 수정 해야 함)에 존재하니, 참고하여 config 파일을 작성해 자유롭게 학습을 진행하실 수 있습니다.

```yaml
network: SATRN
input_size:
  height: 100
  width: 400
SATRN:
  encoder:
    hidden_dim: 300
    filter_dim: 600
    layer_num: 6
    head_num: 8
  decoder:
    src_dim: 300
    hidden_dim: 128
    filter_dim: 512
    layer_num: 3
    head_num: 8
checkpoint: ""
prefix: "./log/satrn"

data:
  train:
    - "../../train_dataset/gt.txt"
  test:
    - ""
  token_paths:
    - "- "../../train_dataset/tokens.txt""  # 241 tokens
  dataset_proportions:  # proportion of data to take from train (not test)
    - 1.0
  random_split: True # if True, random split from train files
  test_proportions: 0.2 # only if random_split is True
  crop: True
  rgb: 1    # 3 for color, 1 for greyscale
  
batch_size: 25
num_workers: 8
num_epochs: 50
print_epochs: 1
dropout_rate: 0.1
teacher_forcing_ratio: 0.8
max_grad_norm: 2.0
seed: 1234
optimizer:
  optimizer: 'NAdam' # Adam, Adadelta, ...extra
  lr: 5e-4 # learning rate
  weight_decay: 1e-4
  is_cycle: True
```



## Contributors

[박재우(JJayy)](https://github.com/JJayy) | :floppy_disk:[송광원(remaindere)](https://github.com/remaindere) | :sparkles:[신찬엽(chanyub)](https://github.com/chanyub) | [조원(jo-member)](https://github.com/jo-member) | :no_entry_sign:[탁금지(Atica57)](https://github.com/Atica57) | :spades:[허재섭(shjas94)](https://github.com/shjas94)



## Reference

* [On Recognizing Texts of Arbitrary Shapes with 2D Self-Attention](https://arxiv.org/abs/1910.04396)



## License

 본 프로젝트는 아래의 license를 따릅니다.

```
Copyright (c) 2020-present NAVER Corp.

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF ERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.  IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
```

