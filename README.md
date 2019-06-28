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
