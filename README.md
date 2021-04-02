原版仓库：https://github.com/ultralytics/yolov5

环境要求：python version >= 3.6

模型训练：python3 train.py

模型导出：python3 models/export.py --rknn_mode

模型预测：python3 detect.py --rknn_mode



使用说明：

​		将common文件中激活层修改为ReLU, 此外训练、测试及其他操作均与原版本 Yolov5 一致。模型测试、导出时增添 rknn_mode 模式，导出对rknn友好型模型。（opset_version=10, rknn_toolkit_1.6.0）



修改说明：

- onnx.slice 在 rknn_toolkit_1.6.0 的模型载入会有报错。增添等价替换模型。(卷积)



已知问题：

- onnx.opset_version=12 不支持 SiLU 激活层，可增添等价替代模型解决。(x* sigmoid(x))。但是rknn_toolkit_1_6_0 模拟中结果正常，部署到板子端会出现异常。默认暂不使用。
-  onnx.upsample.opset_version=12 在 rknn_toolkit_1.6.0的实现 暂时存在问题，增添等价替换模型。(反卷积) 。rknn_toolkit_1_6_0模拟中结果正常，部署到板子端会出现异常。默认暂不使用。





========================	以下为老版本的npu速度测试   ===========================

修改部分：

* 将激活函数改为 relu

* Focus去除切片操作

  

注意事项：如果训练尺寸不是640那么，anchors会自动聚类重新生成，生成的结果在训练时打印在控制台，或者通过动态查看torch模型类属性获取，如果anchors不对应那么结果就会出现检测框对不上。

建议：在训练时如果size不是640，那么可以先通过聚类得到anchors并将新的anchors写入到模型配置文件中，然后再训练，防止动态获取的anchors在rknn上预测不准的问题。训练参数别忘记加上 --noautoanchor。



边缘端部署速度测试参考(单位:ms)：

| 模型                    | rknn_toolkit_1.6.0模拟器评估(800MHZ)_rv1109 | rv1109*** | rv1109(模型预编译) | rv1126 | rv1126(模型预编译) | rknn_toolkit_1.6.0模拟器评估(800MHZ)_rk1808 | rk1808 | rk1808(模型预编译) |
| :---------------------- | :-----------------------------------------: | :-------: | ------------------ | :----: | :----------------: | :-----------------------------------------: | :----: | :----------------: |
| yolov5s_int8*           |                     92                      |    113    |                    |   80   |         77         |                     89                      |   83   |         81         |
| yolov5s_int8_optimize** |                     18                      |    45     |                    |   36   |         33         |                     15                      |   30   |         29         |
| yolov5s_int16           |                     149                     |    160    |                    |  110   |        108         |                     106                     |  178   |        174         |
| yolov5s_int16_optimize  |                     76                      |    90     |                    |   67   |         64         |                     32                      |  126   |        122         |
| yolov5m_int8            |                     158                     |    192    |                    |  132   |        120         |                     144                     |  132   |        123         |
| yolov5m_int8_optimize   |                     47                      |    88     |                    |   66   |         55         |                     33                      |   54   |         45         |
| yolov5m_int16           |                     312                     |    302    |                    |  212   |        202         |                     187                     |  432   |        418         |
| yolov5m_int16_optimize  |                     202                     |    198    |                    |  147   |        137         |                     76                      |  354   |        344         |
| yolov5l_int8            |                     246                     |    293    |                    |  199   |                    |                     214                     |  192   |                    |
| yolov5l_int8_optimize   |                     98                      |    155    |                    |  110   |                    |                     66                      |   88   |                    |
| yolov5l_int16           |                     577                     |    522    |                    |  362   |                    |                     301                     |  697   |                    |
| yolov5l_int16_optimize  |                     432                     |    384    |                    |  275   |                    |                     154                     |  592   |                    |

\* 是指基于原版的yaml配置，激活层修改为relu，去除头部slice，cat操作。下同。

\*\* optimize是指在导出模型时进行优化，包括但不限于剪枝、稀疏化的方式，优化后精度保持99.5%以上，该方法目前暂不开源，提供代转模型服务。

*\*\*  统计时间包含 **rknn_inputs_set**、**rknn_run**、**rknn_outputs_get** 三部分时间，不包含cpu端后处理时间。除模拟器评估外，本表其他平台的测试均遵循此原则。

\**** 该测试仅供参考，测试时为单线程循环执行计时，仅测试npu效率。实际使用时应该考虑后处理的时间。

\*\*\*\*\* 本库仅涉及 python 端的开发， 暂不涉及 部署端代码。



TODO

- [x] 兼容前端的slice操作。
- [x] 兼容原版激活函数。
- [ ] npu速度依据导出模型结构变动，重测
- [ ] 开源优化方式，时间待定。



参考库：

https://github.com/soloIife/yolov5_for_rknn

https://github.com/ultralytics/yolov5



技术交流群：

QQ群：810456486

