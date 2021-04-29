## 转换模型小工具

rknn转换模型所需要修改的参数写出yaml格式，方便修改切换。



下面给出各部分的简短解释。



```yaml
running:						# 执行部分，设置为false时对应操作将不会执行
  model_type: onnx				# 选择导入模型的类型，必选。为待 转换模型/ rknn模型
  export: True					# 执行模型导出(含转换行为，转换过程中的参数在后面)
  inference: False				# 执行推断
  eval_perf: False 				# 执行速度评估

parameters:						# 选择模型类型，各模型参数说明请查看rknn文档
  caffe:
    model: './mobilenet_v2.prototxt'
    proto: 'caffe' #lstm_caffe
    blobs: './mobilenet_v2.caffemodel'
  
  tensorflow:
    tf_pb: './ssd_mobilenet_v1_coco_2017_11_17.pb'
    inputs: ['FeatureExtractor/MobilenetV1/MobilenetV1/Conv2d_0/BatchNorm/batchnorm/mul_1']
    outputs: ['concat', 'concat_1']
    input_size_list: [[300, 300, 3]]

  tflite:
    model: './sample/tflite/mobilenet_v1/mobilenet_v1.tflite'

  onnx:
    model: './best.onnx'

  darknet:
    model: './yolov3-tiny.cfg'
    weight: './yolov3.weights'

  pytorch:
    model: './yolov5.pt'
    input_size_list: [[3, 512, 512]]

  mxnet:
    symbol: 'resnext50_32x4d-symbol.json'
    params: 'resnext50_32x4d-4ecf62e2.params'
    input_size_list: [[3, 224, 224]]

  rknn:
    path: './test.rknn'

config:							# 模型转换参数，具体子参数意义详见rk文档
  channel_mean_value: '0 0 0 255' # 123.675 116.28 103.53 58.395 # 0 0 0 255
  reorder_channel: '2 1 0' # '0 1 2' '2 1 0'
  need_horizontal_merge: True
  batch_size: 10
  epochs: 100
  target_platform: ['rk1808']
  quantized_dtype: 'asymmetric_quantized-u8' # asymmetric_quantized-u8,dynamic_fixed_point-8,dynamic_fixed_point-16
  optimization_level: 1

build:
# 模型构建参数，具体子参数意义详见rk文档。当模型为rknn类型时，会跳过这一步骤。
  do_quantization: True
  dataset: './single_dataset.txt' 
  pre_compile: False

export_rknn:					# 模型导出的路径
  export_path: './best.rknn'

init_runtime:
# 模型初始化参数，一般无需改动。当 inference 和 eval_perf 都不执行时，会跳过这一步骤。
  target: null
  device_id: null
  perf_debug: False
  eval_mem: False
  async_mode: False

img: &img						# 推断、评估模型所使用的图片路径
  path: './test.jpg'

inference:						# 推断参数，具体子参数意义详见rk文档
  inputs: *img
  data_type: 'uint8'
  data_format: 'nhwc' # 'nchw', 'nhwc'
  inputs_pass_through: None 

eval_perf:						# 评估参数，具体子参数意义详见rk文档
  inputs: *img
  data_type: 'uint8'
  data_format: 'nhwc'
  is_print: True

```

