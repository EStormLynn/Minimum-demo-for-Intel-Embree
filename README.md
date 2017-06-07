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

## 程序：
```C++

#pragma comment(lib,"lib\\embree.lib")
#include <iostream>
#include <xmmintrin.h>
#include <pmmintrin.h>
#include "embree2/rtcore.h"
#include "embree2/rtcore_ray.h"

using namespace std;

int main()
{
	_MM_SET_FLUSH_ZERO_MODE(_MM_FLUSH_ZERO_ON);
	_MM_SET_DENORMALS_ZERO_MODE(_MM_DENORMALS_ZERO_ON);

	RTCDevice device = rtcNewDevice(NULL);
	RTCError error = rtcDeviceGetError(device);
	printf("Device Creation Error = %i\n", error);

	RTCScene scene = rtcDeviceNewScene(device, RTC_SCENE_DYNAMIC, RTC_INTERSECT1);

	unsigned int geoID = rtcNewTriangleMesh(scene, RTC_GEOMETRY_STATIC, 1, 3, 1);

	struct Vertex { float x, y, z, a; };
	struct Triangle { int v0, v1, v2; };

	Vertex* vertices = (Vertex*)rtcMapBuffer(scene, geoID, RTC_VERTEX_BUFFER);
	// fill vertices here
	//(0,0) - (1,0) - (0,1)
	vertices->x = 0;
	vertices->y = 0;
	vertices->z = 0;
	vertices->a = 1;

	(vertices + 1)->x = 1;
	(vertices + 1)->y = 0;
	(vertices + 1)->z = 0;
	(vertices + 1)->a = 1;

	(vertices + 2)->x = 0;
	(vertices + 2)->y = 1;
	(vertices + 2)->z = 0;
	(vertices + 2)->a = 1;

	rtcUnmapBuffer(scene, geoID, RTC_VERTEX_BUFFER);

	Triangle* triangles = (Triangle*)rtcMapBuffer(scene, geoID, RTC_INDEX_BUFFER);
	// fill triangle indices here
	triangles->v0 = 0;
	triangles->v1 = 1;
	triangles->v2 = 2;
	rtcUnmapBuffer(scene, geoID, RTC_INDEX_BUFFER);

	rtcCommit(scene);

	RTCRay TheRay;
	memset(&TheRay, 0, sizeof(TheRay));

	//render a 20x20 ascii grid
	for (int i = 0; i < 21; ++i)
	{
		for (int j = 0; j < 21; ++j)
		{
			//Origin: -2 on Z
			TheRay.org[0] = 0;
			TheRay.org[1] = 0;
			TheRay.org[2] = -2;

			//Direction: scan the 0.5 by 0.5 square on -1 of Z
			TheRay.dir[0] = 0.025f * (float)(20 - i);
			TheRay.dir[1] = 0.025f * (float)j;
			TheRay.dir[2] = 1; //pointing to positive Z

							   //Near Plane
			TheRay.tnear = 0;

			//Far Plane
			TheRay.tfar = 9999;

			//Don't motion blur
			TheRay.time = 0;

			//Trace All Geometries
			TheRay.mask = 0xFFFFFFFF;

			//Initialize Output Fields
			TheRay.instID = RTC_INVALID_GEOMETRY_ID;
			TheRay.geomID = RTC_INVALID_GEOMETRY_ID;		//几何标识符命中几何
			TheRay.primID = RTC_INVALID_GEOMETRY_ID;		//几何命中实例标识符

			//Trace it
			rtcIntersect(scene, TheRay);

			//print it
			if (TheRay.geomID != RTC_INVALID_GEOMETRY_ID)
			{
				printf("0  ");
				printf("%f:%f|\n", TheRay.u, TheRay.v);
			}
			else
			{
				printf("*  ");
			}

		}
		printf("\n");
	}

	rtcDeleteScene(scene);
	rtcDeleteDevice(device);
	return 0;
}
```
