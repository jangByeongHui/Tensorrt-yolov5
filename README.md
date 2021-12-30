# Tensorrt-yolov5
기존 pytorch를 사용하여 학습시킨 yolov5의 가중치를 tensorrt로 동작하도록 하여 더욱 빠른 추론이 가능하도록 한다.
tensorrt는 기존 가중치의 정밀도(가중치들의 bit수를 감소)를 낮추어 더욱 빠른 연산이 가능하도록하여 더욱 빠르고 효율적인 연산을 할 수 있도록 한다.
이를 통해서 동일한 모델에 대해서 더욱 빠른 추론이 가능해지게 된다.
Tensorrt는 FP32의 데이터를 FP16 혹은 INT 8의 데이터 타입으로 정밀도를 낮출 수 있다. 

## pytorch
object를 detect를 할 때 높은 정확도로 detect하는 것을 확인 할 수 있다. 그러나 다음과 결과를 얻기 위해서 175장의 사진에서 차량을 검출하기 위해 몇 십분의 시간이 소요되었다.

<img src="https://github.com/jangByeongHui/car_detect/blob/main/ezgif.com-gif-maker.gif?raw=true">


# tensorrt
pytorch를 사용한 것보다 detection이 떨어지는 모습이며, 엉뚱한 곳을 detect하는 경우도 발생한다. 그러나 175장의 사진을 연속적으로 detect하는데 1분 채 걸리지 않는 시간이 소요되었다.
<img src="https://github.com/jangByeongHui/car_detect/blob/main/test_2.gif?raw=true">


## yolov5s 모델을 tensorrt 사용 방법

1.  yolov5에서 pytorch로 생성한 .pt 가중치 파일을 .wts로 변환

```
// clone code according to above #Different versions of yolov5
// download https://github.com/ultralytics/yolov5/releases/download/v6.0/yolov5s.pt
//tensorrtx안에 gen_wts.py로 .pt -> .wts로 변환
cp {tensorrtx}/yolov5/gen_wts.py {ultralytics}/yolov5
cd {ultralytics}/yolov5
python gen_wts.py -w yolov5s.pt -o yolov5s.wts
//.wts는 .pt위치에 동일한 이름으로 생성 
// a file 'yolov5s.wts' will be generated.
```

2. build tensorrtx/yolov5 and run

```
cd {tensorrtx}/yolov5/
//클래스 수 변경
// update CLASS_NUM in yololayer.h if your model is trained on custom dataset
mkdir build
cd build
cp {ultralytics}/yolov5/yolov5s.wts {tensorrtx}/yolov5/build
cmake ..
make
sudo ./yolov5 -s [.wts] [.engine] [n/s/m/l/x/n6/s6/m6/l6/x6 or c/c6 gd gw]  // serialize model to plan file
sudo ./yolov5 -d [.engine] [image folder]  // deserialize and run inference, the images in [image folder] will be processed.
// For example yolov5s
sudo ./yolov5 -s yolov5s.wts yolov5s.engine s
sudo ./yolov5 -d yolov5s.engine ../samples
// For example Custom model with depth_multiple=0.17, width_multiple=0.25 in yolov5.yaml
sudo ./yolov5 -s yolov5_custom.wts yolov5.engine c 0.17 0.25
sudo ./yolov5 -d yolov5.engine ../samples
```

3. check the images generated, as follows. _zidane.jpg and _bus.jpg

4. optional, load and run the tensorrt model in python

```
// install python-tensorrt, pycuda, etc.
// ensure the yolov5s.engine and libmyplugins.so have been built
python yolov5_trt.py
```
