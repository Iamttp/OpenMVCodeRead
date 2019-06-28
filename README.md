# OPENMV 代码阅读

* 默认执行点检查
```python
class ctrl(object):
    work_mode = 0x01 #工作模式.默认是点检测，可以通过串口设置成其他模式
    check_show = 1   #开显示，在线调试时可以打开，离线使用请关闭，可提高计算速度
```

* 默认分辨率80x60
```python
sensor.set_framesize(sensor.QQQVGA) # 80x60
```

* 循环执行
```python
while(True):
    clock.tick()
    if (ctr.work_mode&0x01)!=0:
        img = sensor.snapshot()
        check_dot(img)
    
    #线检测
    if (ctr.work_mode&0x02)!=0:
        img = sensor.snapshot().binary([THRESHOLD])
        check_line(img)
     
    #接收串口数据
    uart_read_buf()
```

1. 工作模式一

* 修改代码，查看最大点的坐标以及总的像素数目。`thresholds = [0, 90]#自定义灰度阈值`
```python
#点检测函数
def check_dot(img):
    for blob in img.find_blobs([thresholds], pixels_threshold=3, area_threshold=3, merge=True, margin=5):
        if dot.pixels<blob.pixels():#寻找最大的黑点
            dot.pixels=blob.pixels()
            dot.x = blob.cx()
            dot.y = blob.cy()
            dot.ok= 1
            img.draw_cross(dot.x, dot.y, color=127, size = 10)
            img.draw_circle(dot.x, dot.y, 5, color = 127)

    #判断标志位 赋值像素点数据
    dot.flag = dot.ok
    dot.num = dot.pixels

    #清零标志位
    dot.pixels = 0
    dot.ok = 0

    # TODO 添加代码查看发送给四轴的数据
    print(dot.x,dot.y,dot.num,dot.flag)
    #发送数据
    uart.write(pack_dot_data())
```

2. 工作模式二

默认`THRESHOLD = (0,100)`， 应该是0-100的像素为设置为255
```python
img = sensor.snapshot().binary([THRESHOLD])
```

Line类继承于Dot增加了角度属性
```python
class Dot(object):
    x = 0
    y = 0
    pixels = 0
    num = 0
    ok = 0
    flag = 0

class Line(Dot):
    x_angle = 0
    y_angle = 0
```

添加打印代码，查看各个ROI区域找到的线
```python
def fine_border(img,area,area_roi):
    line = img.get_regression([(255,255)],roi=area_roi, robust = True)
    # 例如：line: {"x1":67, "y1":0, "x2":67, "y2":39, "length":39, "magnitude":12, "theta":0, "rho":67}
    # x1 y1 x2 y2分别代表线段的两个顶点坐标，length是线段长度，theta是线段的角度。
    # magnitude表示线性回归的效果，它是（0，+∞）范围内的一个数字，其中0代表一个圆。如果场景线性回归的越好，这个值越大。
    # y向下，x向右
    print("line:",line)
    if (line):
        area.ok=1
    #判断标志位
    dot.flag = dot.ok
    #清零标志位

    #dot.pixels = 0
    dot.ok = 0
```

发送的数据， 貌似上部分代码没用到
```python
# rho_err线到(0,0)的距离
# theta_err线的角度
print(singleline_check.rho_err,singleline_check.theta_err )
```