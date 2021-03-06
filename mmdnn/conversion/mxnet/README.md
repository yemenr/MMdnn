# MXNet README

## MXNet pre-trained model

We tested some MXNet pre-trained models to others, get more detail from [this file](https://github.com/Microsoft/MMdnn/blob/master/mmdnn/conversion/examples/mxnet/extractor.py)

|     Models    | Caffe | Keras | Tensorflow | CNTK | MXNet | PyTorch | CoreML | ONNX |
| :-----------: | :---: | :---: | :--------: | :--: | :---: | :-----: | :----: | :--: |
|     Vgg19     |   √   |   √   |      √     |   √  |   √   |    √    |    √   |   √  |
|  Inception_bn |   √   |   √   |      √     |   √  |   √   |    √    |    √   |   √  |
|   ResNet 18   |   √   |   √   |      √     |   √  |   √   |    √    |    √   |   √  |
|   ResNet 152  |   √   |   √   |      √     |   √  |   √   |    √    |    √   |   √  |
|   ResNext 50  |   √   |   √   |      √     |   √  |   √   |    √    |    √   |   √  |
|  ResNext 101  |   √   |   √   |      √     |   √  |   √   |    √    |    √   |   √  |
| squeezenet_v1 |   √   |   √   |      √     |   √  |   √   |    √    |    √   |   √  |

**√** - Correctness tested

**o** - Some difference after conversion

**space** - not tested

---

# Usage

## Download MXNET pre-trained model

```bash
$ mmdownload -f mxnet

Supported models : ['imagenet1k-resnet-152', 'vgg19', 'imagenet1k-resnet-101', 'imagenet1k-resnet-50', 'vgg16', 'imagenet1k-inception-bn', 'imagenet1k-resnext-101', 'imagenet11k-resnet-152', 'imagenet1k-resnext-50', 'imagenet1k-resnext-101-64x4d', 'imagenet1k-resnet-18', 'imagenet11k-place365ch-resnet-152', 'imagenet1k-resnet-34', 'squeezenet_v1.1', 'imagenet11k-place365ch-resnet-50', 'squeezenet_v1.0']

$ mmdownload -f mxnet -n imagenet1k-resnet-50 -o ./

Downloading file [./resnet-50-symbol.json] from [http://data.mxnet.io/models/imagenet/resnet/50-layers/resnet-50-symbol.json]
progress: 80.0 KB downloaded, 100%
Downloading file [./resnet-50-0000.params] from [http://data.mxnet.io/models/imagenet/resnet/50-layers/resnet-50-0000.params]
progress: 100000.0 KB downloaded, 100%
MXNet Model imagenet1k-resnet-50 saved as [./resnet-50-symbol.json] and [./resnet-50-0000.params].

```

---

## One-step conversion

Above MMdnn@0.1.4, we provide one command to achieve the conversion

```bash
$  mmconvert -sf mxnet -in resnet-50-symbol.json -iw resnet-50-0000.params -df cntk -om mxnet_resnet50.dnn --inputShape 3,224,224
.
.
.
CNTK model file is saved as [mxnet_resnet50.dnn], generated by [4c616299273a42e086b30c6c4d1c64c0.py] and [4c616299273a42e086b30c6c4d1c64c0.npy].

```

Then you get the CNTK original model *mxnet_resnet152.dnn* converted from MXNet. Temporal files are removed automatically.

---

## Step-by-step conversion (for debugging)

### Convert architecture from MXNET to IR (MXNET -> IR)

You can use following bash command to convert the network architecture [*mxnet/models/resnet-50-symbol.json*] to IR architecture file [*resnet50.pb*], [*resnet50.json*]. You can convert only network structure to IR for visualization or training in other frameworks.

```bash
$ mmtoir -f mxnet -n mxnet/models/resnet-50-symbol.json -d resnet50 --inputShape 3,224,224
.
.
.
IR network structure is saved as [resnet50.json].
IR network structure is saved as [resnet50.pb].
Warning: weights are not loaded.
```

### Convert models (including architecture and weights) from MXNet to IR (MXNET -> IR)

You can use following bash command to convert the network architecture [*mxnet/models/resnet-50-symbol.json*] with weights [*mxnet/models/resnet-50-0000.params*] to IR architecture file [*resnet50.pb*], [*resnet50.json*], [*resnet50.npy*].

> The input data shape is not in the architecture description of MXNet, we need to specify the data shape in conversion command.

```bash
$ mmtoir -f mxnet -n mxnet/models/resnet-50-symbol.json -w mxnet/models/resnet-50-0000.params -d resnet50 --inputShape 3,224,224
.
.
.
IR network structure is saved as [resnet50.json].
IR network structure is saved as [resnet50.pb].
IR weights are saved as [resnet50.npy].
```

### Convert models from IR to MXNet code snippet and weights (IR -> MXNet)

We need to generate both MXNet architecture code snippet and weights file to build the MXNet network.

> [Note!] Argument 'dw' is used to specify the converted MXNet model file name for next step use.

```bash
$ mmtocode -f mxnet --IRModelPath inception_v3.pb --dstModelPath mxnet_inception_v3.py --IRWeightPath inception_v3.npy -dw mxnet_inception_v3-0000.params

Parse file [inception_v3.pb] with binary format successfully.
Detect input layer [input_1] using infer batch size, set it as default value [1]
Target network code snippet is saved as [mxnet_inception_v3.py].
```

### Convert models from IR to MXNet checkpoint file

After generating the MXNet code snippet and weights, you can take a further step to generate an original MXNet checkpoint file.

```bash
$ python -m mmdnn.conversion.examples.mxnet.imagenet_test -n mxnet_inception_v3 -w mxnet_inception_v3-0000.params --dump inception_v3
.
.
.
MXNet checkpoint file is saved as [inception_v3], generated by [mxnet_inception_v3.py] and [mxnet_inception_v3-0000.params].
```

Then the output files *inception_v3-symbol.json* and *inception_v3-0000.params* can be loaded by MXNet directly.

---

## Develop version

Ubuntu 16.04 with

- MXNet 0.11.0

@ 11/22/2017

## Limitation

- Currently no RNN related operations support
