# Minimum-demo-for-Intel-Embree
使用Embree实现简单的rayTrace
## 引用库
* [embree](https://github.com/embree/embree)(Intel embree raytrace kernels)

## 程序流程

 步骤|   说明
:----|:----
rtcInit()|初始化
rtcDeviceNewScene()|创建场景
rtcNewTriangleMesh()|创建mesh
rtcUnmapBuffer()|写入顶点buffer
rtcCommit()|提交场景
trace rays|光线跟踪
rtcExit()|清除退出

## 实现效果
0.5x0.5范围，发生碰撞打印*

![](http://oo8jzybo8.bkt.clouddn.com/RayTracing.png)
