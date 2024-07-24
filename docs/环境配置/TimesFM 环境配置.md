## Condition

TimesFM 环境[要求严苛](https://github.com/google-research/timesfm/issues/1)，需要在 X 86 架构下的 Linux 系统中运行，如需在 windows 环境中运行 TImesFM 可以使用 WSL 2 创建 Linux 环境。

![[Win10 安装 WSL2-Ubuntu 并配置 anaconda]]

## Python 环境配置

1. 创建新的 python 环境

```python
conda create -n tfm_env python=3.10
```

2. 使用 pip 命令安装 `timesfm` 包，

```python
pip install timesfm
```

> [!NOTE]
> 注意请不要使用清华源等国内镜像源（这些包国内镜像站上搜不到，只能去 [http://pypi.org](https://link.zhihu.com/?target=http%3A//pypi.org) 官方镜象站找），如果出现网络问题请参考 [[Win10 安装 WSL2-Ubuntu 并配置 anaconda#配置 VPN（可选）|配置 VPN]]

## 模型测试

由于设备原因，无法进行模型测试，参考 [github issue](https://github.com/google-research/timesfm/issues/80)，在 RAM 为 64 GB 设备该模型能够良好运行。

```note
WARNING:absl:train_state_unpadded_shape_dtype_struct is not provided. We assume `train_state` is unpadded.
ERROR:absl:For checkpoint version > 1.0, we require users to provide
          `train_state_unpadded_shape_dtype_struct` during checkpoint
          saving/restoring, to avoid potential silent bugs when loading
          checkpoints to incompatible unpadded shapes of TrainState.
Restored checkpoint in 3.17 seconds.
Jitting decoding.
/root/anaconda3/envs/tfm/lib/python3.10/multiprocessing/resource_tracker.py:224: UserWarning: resource_tracker: There appear to be 1 leaked semaphore objects to clean up at shutdown
  warnings.warn('resource_tracker: There appear to be %d '
```
