## 仓库说明：

原版仓库：https://github.com/ultralytics/yolov5

环境要求：python version >= 3.6

模型训练：python3 train.py

模型导出：python3 models/export.py --rknn_mode

模型预测：python3 detect.py --rknn_mode



## 删改说明：

​		将common文件中激活层修改为ReLU，此外模型结构、训练、测试及其他操作均与原版本 Yolov5 一致。模型测试、导出时增添 rknn_mode 模式，导出对rknn友好型模型。（基于opset_version=10, rknn_toolkit_1.6.0测试通过）

实现细节：

- onnx.slice 在 rknn_toolkit_1.6.0 的模型载入会有报错。增添等价替换模型。(卷积)
- ~~onnx.opset_version=12 不支持 SiLU 激活层，可增添等价替代模型解决。(x* sigmoid(x))~~
- ~~onnx.upsample.opset_version=12 在 rknn_toolkit_1.6.0的实现 暂时存在问题，增添等价替换模型。(反卷积) 。~~



***4.29更新***：

- 导出模型可选择去掉尾端的permute层，从而**兼容rknn_yolov5_demo的c++部署代码**

  `python3 models/export.py --rknn_mode --ignore_output_permute`

- 导出模型可选择增加**图片预处理层**，可**有效降低部署段 rknn_input_set 的时间耗时**。具体使用方法参考 models/common_rk_plug_in.py 里的 preprocess_conv_layer 的说明。

  `python3 models/export.py --rknn_mode --add_image_preprocess_layer`

  （rknn_mode、ignore_output_permute、add_image_preprocess_layer 三者不互斥，可同时使用）



## 已知问题说明(不影响目前使用)：

- onnx.opset_version=12 不支持 SiLU 激活层，可增添等价替代模型解决。(x* sigmoid(x)) 但是rknn_toolkit_1_6_0 模拟中结果正常，部署到板子端会出现异常。默认暂不使用，等待rk修复。
-  onnx.upsample.opset_version=12 在 rknn_toolkit_1.6.0的实现 暂时存在问题，增添等价替换模型。(反卷积) 。rknn_toolkit_1_6_0模拟中结果正常，部署到板子端会出现异常。默认暂不使用，等待rk修复。



------

### rk_npu速度测试<sup>[4](#脚注4)</sup> <sup>[5](#脚注5)</sup>(单位:ms)：

| 模型                    | rknn_toolkit_1.6.0模拟器评估(800MHZ)_rv1109 | rv1109<sup>[3](#脚注3)</sup> | rv1109(模型预编译) | rv1126  | rv1126(模型预编译) | rknn_toolkit_1.6.0模拟器评估(800MHZ)_rk1808 | rk1808 | rk1808(模型预编译) |
| :---------------------- | :-----------------------------------------: | :-------: | ------------------ | :-----: | :----------------: | :-----------------------------------------: | :----: | :----------------: |
| yolov5s_int8<sup>[1](#脚注1)</sup> |                     92                      |    113    |                    |   80    |         77         |                     89                      |   83   |         81         |
| yolov5s_int8_optimize<sup>[2](#脚注2)</sup> |                   **18**                    |  **45**   |                    | **36**  |       **33**       |                   **15**                    | **30** |       **29**       |
| yolov5s_int16           |                     149                     |    160    |                    |   110   |        108         |                     106                     |  178   |        174         |
| yolov5s_int16_optimize  |                     76                      |    90     |                    |   67    |         64         |                     32                      |  126   |        122         |
| yolov5m_int8            |                     158                     |    192    |                    |   132   |        120         |                     144                     |  132   |        123         |
| yolov5m_int8_optimize   |                   **47**                    |  **88**   |                    | **66**  |       **55**       |                   **33**                    | **54** |       **45**       |
| yolov5m_int16           |                     312                     |    302    |                    |   212   |        202         |                     187                     |  432   |        418         |
| yolov5m_int16_optimize  |                     202                     |    198    |                    |   147   |        137         |                     76                      |  354   |        344         |
| yolov5l_int8            |                     246                     |    293    |                    |   199   |                    |                     214                     |  192   |                    |
| yolov5l_int8_optimize   |                   **98**                    |  **155**  |                    | **110** |                    |                   **66**                    | **88** |                    |
| yolov5l_int16           |                     577                     |    522    |                    |   362   |                    |                     301                     |  697   |                    |
| yolov5l_int16_optimize  |                     432                     |    384    |                    |   275   |                    |                     154                     |  592   |                    |

<a name="脚注1">1</a>: 是指基于原版的yaml配置，激活层修改为relu。

<a name="脚注2">2</a>: optimize是指在导出模型时进行优化，包括但不限于剪枝、稀疏化的方式，优化后精度保持99.5%以上，该方法目前暂不开源，提供代转模型服务。

<a name="脚注3">3</a>: 统计时间包含 **rknn_inputs_set**、**rknn_run**、**rknn_outputs_get** 三部分时间，不包含cpu端后处理时间。除模拟器评估外，本表其他平台的测试均遵循此原则。

<a name="脚注4">4</a>: 该测试仅供参考，测试时为单线程循环执行计时，仅测试npu效率。实际使用时应该考虑后处理的时间。

<a name="脚注5">5</a>: 本库仅涉及 python 端的开发， 暂不涉及 部署端代码。





## TODO

- [x] 兼容前端的slice操作。
- [x] 兼容原版激活函数。
- [x] npu速度测试
- [ ] 开源优化方式，时间待定。



## 参考库：

https://github.com/soloIife/yolov5_for_rknn

https://github.com/ultralytics/yolov5



## 技术交流群：

QQ群：810456486

