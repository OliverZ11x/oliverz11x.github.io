---
date created: 2024/8/21 15:52
date modified: 2024/8/22 11:0
---

[【避坑】paddlepaddle-gpu安装报错：The GPU architecture in your current machine is Pascal, which is not-CSDN博客](https://blog.csdn.net/qq_37085158/article/details/132598829)

```shell
conda create -n paddle python==3.7.16
```

```shell
conda install cudatoolkit=11.2 cudnn=8.2.1 -c conda-forge
```

```shell
pip install .\paddlepaddle_gpu-2.3.2.post112-cp37-cp37m-win_amd64.whl
```

pip3 install paddle-serving-client==0.7.0 -i https://pypi.tuna.tsinghua.edu.cn/simple

pip3 install paddle-serving-server==0.7.0 -i https://pypi.tuna.tsinghua.edu.cn/simple

# CPU

pip3 install paddle-serving-app==0.7.0 -i https://pypi.tuna.tsinghua.edu.cn/simple

# GPU with CUDA10.2 + TensorRT6

pip3 install paddle-serving-server-gpu==0.7.0.post102 -i https://pypi.tuna.tsinghua.edu.cn/simple

其他GPU环境需要确认环境再选择执行哪一条

# GPU with CUDA10.1 + TensorRT6

pip3 install paddle-serving-server-gpu==0.7.0.post101 -i https://pypi.tuna.tsinghua.edu.cn/simple

# GPU with CUDA11.2 + TensorRT8

pip3 install paddle-serving-server-gpu==0.7.0.post112 -i https://pypi.tuna.tsinghua.edu.cn/simple