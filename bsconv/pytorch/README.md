# BSConv implementation for PyTorch

We provide several implementation variants for BSConv:

* *Ready-to-use model definitions*
    * suited for models which require special considerations when transforming them to BSConv variants (e.g., MobileNets, ResNets-50 and larger, EfficientNets, etc.)
    * can be used to reproduce the results reported in the paper
* *BSConv as general drop-in replacement*
    * replaces convolutions in existing model definitions by BSConv
    * suited for CNNs which use regular convolutions (without groups, bottlenecks, etc.), e.g. ResNets (up to ResNet-34), VGGs, DenseNets, etc.
    * for other models (e.g. MobileNets, ResNets-50 and larger, EfficientNets, etc.) use our ready-to-use model definitions (see below)
* *BSConv PyTorch modules*
    * these modules can be used instead of regular convolution layers
    * suited for building custom models from scratch


## Ready-to-use model definitions

Coming soon.


## BSConv as general drop-in replacement

1. Load an existing model definition
2. Replace convolution layers by BSConv modules
3. Add regularization loss (BSConv-S only)

Concrete examples will follow soon.


### 1. Load an existing model definition

Currently supported are models with `torch.nn.Conv2d` layers without any bottleneck structure.

#### Using `torchvision` ([https://github.com/pytorch/vision](https://github.com/pytorch/vision))

Currently supported torchvision models are:
* ResNet-18, ResNet-34
* VGG-11, VGG-13, VGG-16, VGG-19 (with and without batch normalization)

Example (ResNet-18):

```python
import torchvision.models
model = torchvision.models.resnet18()
```

#### Using `pytorchcv` ([https://github.com/osmr/imgclsmob](https://github.com/osmr/imgclsmob/tree/master/pytorch/pytorchcv))


Example (ResNet-18):

```python
import pytorchcv.model_provider
model = pytorchcv.model_provider.get_model("resnet18")
```

A full list of supported pytorchcv models will follow soon.

### 2. Replace convolution layers by BSConv modules

Replace each `torch.nn.Conv2d` by BSConv modules:

For unconstrained BSConv (BSConv-U):

```python
replacer = bsconv.pytorch.BSConvU_Replacer()
model = replacer.apply(model)
```

For subspace BSConv (BSConv-S):

```python
replacer = bsconv.pytorch.BSConvS_Replacer()
model = replacer.apply(model)
```

### 3. Add regularization loss (BSConv-S only)

When calculating the loss, the regularization can easily be added with a weighting coefficient with only one modified line of code!

```python
...
output = model(images)
loss = criterion(output, target)

# THIS LINE MUST BE ADDED, everything else remains unchanged
loss += model.reg_loss(alpha=0.1)

optimizer.zero_grad()
loss.backward()
optimizer.step()
...
```

That's all you need to do in your training script!


## BSConv PyTorch modules

We provide two PyTorch modules `bsconv.pytorch.BSConvU` and `bsconv.pytorch.BSConvS` which can be used instead of `torch.nn.Conv2d` layers.
Building a custom AlexNet model with BSConv modules:

```python
import torch
import bsconv.pytorch

class BSConvAlexNet(torch.nn.Module):

	def __init__(self, num_classes=1000):
		super().__init__()
		self.features = torch.nn.Sequential(
			bsconv.pytorch.BSConvU(3, 64, kernel_size=11, stride=4, padding=2),
			torch.nn.ReLU(inplace=True),
			torch.nn.MaxPool2d(kernel_size=3, stride=2),
			bsconv.pytorch.BSConvU(64, 192, kernel_size=5, padding=2),
			torch.nn.ReLU(inplace=True),
			torch.nn.MaxPool2d(kernel_size=3, stride=2),
			bsconv.pytorch.BSConvU(192, 384, kernel_size=3, padding=1),
			torch.nn.ReLU(inplace=True),
			bsconv.pytorch.BSConvU(384, 256, kernel_size=3, padding=1),
			torch.nn.ReLU(inplace=True),
			bsconv.pytorch.BSConvU(256, 256, kernel_size=3, padding=1),
			torch.nn.ReLU(inplace=True),
			torch.nn.MaxPool2d(kernel_size=3, stride=2),
		)
		self.avgpool = torch.nn.AdaptiveAvgPool2d((6, 6))
		self.classifier = torch.nn.Sequential(
			torch.nn.Dropout(),
			torch.nn.Linear(256 * 6 * 6, 4096),
			torch.nn.ReLU(inplace=True),
			torch.nn.Dropout(),
			torch.nn.Linear(4096, 4096),
			torch.nn.ReLU(inplace=True),
			torch.nn.Linear(4096, num_classes),
		)

	def forward(self, x):
		x = self.features(x)
		x = self.avgpool(x)
		x = torch.flatten(x, 1)
		x = self.classifier(x)
		return x
```

Note that if your network uses subspace BSConv-S layers (`bsconv.pytorch.BSConvS`), you must add the regularization loss to your classification loss:

```python
loss = criterion(output, target) + model.reg_loss(alpha=0.1)
```

Concrete examples will follow soon.

## Improving MobileNets with BSConv

Coming soon.