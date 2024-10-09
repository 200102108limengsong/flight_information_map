# Flight map based on Cesium and Leaflet
## This is a very immature and nascent study project that just started in May 2024.
I will continuously update this project over the next two months (from June 2024 to August 2024) until the overall functionality of the project is usable.

## Description
The program will follow a complete development process, with the front-end components including: Vite, VUE3, Element Plus, Cesium, and Leaflet.
Currently, the resources being used include: OpenWeather (weather information), OpenSky (ADS-B aircraft information), OpenStreetMap (3D buildings), and Tianditu (maps and annotations for China).
(Tentative, as development time may be insufficient) The backend framework will use Django, GeoDjango, GeoServer, PostgreSQL, and PostGIS.

# Vue 3 + Vite
This template should help get you started developing with Vue 3 in Vite. The template uses Vue 3 `<script setup>` SFCs, check out the [script setup docs](https://v3.vuejs.org/api/sfc-script-setup.html#sfc-script-setup) to learn more.

## Recommended IDE Setup
- [VS Code](https://code.visualstudio.com/) + [Vue - Official](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (previously Volar) and disable Vetur


# 基于Cesium和Leaflet的飞行地图
## 这是一个非常不成熟且刚刚起步于2024年6月的学习项目
我会在未来的两个月（2024.6-2024.8）时间里持续更新，直到项目整体功能可以使用。

## 描述
b_fimap用于存储项目后端文件  
f_fimap用于存储项目前端文件  

程序将会采用一个完整的开发流程，前端组件包括：Vite+VUE3+Element plus+Cesium+Leaflet。 

（后端待定，可能开发时间不足）后端框架将采用Django + GeoDjango+ GeoServer+ PostGIS 。

目前使用到的资源有：OpenWeather（天气信息）+OpenSky（ADS-B飞行器信息）+OpenStreetMap（3D建筑物、路网等）、OpenAIP（（欧洲空域数据）、Ourairports（机场数据）

## 不同时间段进展

2024.6.9-2024.7  确定项目目标与需求分析：1.原定计划为低空飞行航图，数据源不足，暂时取消。2.欧洲地区航空数据库开源，研究区由中国南部该为欧洲西部地区。3.开发环境与技术路线选择。

2024.7.2-2024.7.10 完成后端框架搭建与主体项目内容构建，实现通过Opensky-api自动获取数据并通过postgis实现分表存储与简易查询。采用DRF实现后端数据发布。

2024.7.11-2024.7.25 完成前端项目主体构建，模块化架构分离地图数据展示和表格数据展示。实现飞机数据分高度、地区渲染。针对leaflet加载大量点集数据进行分级渲染优化，基本解决卡顿问题。cesium搜集飞行器模型，实现飞机数据初步加载，# 飞机渲染仍有bug 。ourairports机场数据分级渲染完成，无问题。

2024.7.26-2024.7.30 针对前端存在的bug进行初步解决，完成地图说明文本显示。实现通过Openweather调用实时天气数据瓦片并加载到地图上。

2024.8.1-2024.8.5 后端bug修订，解决数据库清表问题与模型数据更新问题。

2024.8.6-2024.8.8 ICP域名备案与阿里云服务器申请，服务器环境初步部署，简历完善。

2024-8.9-2024.8.12 对技术人员与飞行员等相关人员进行访问与调查，判断其对产品的兴趣与需求，准备进行下一步优化。

技术架构图：
![image](https://github.com/200102108limengsong/fwmapPNG/blob/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20241009140647.png)

数据源
![image](https://github.com/200102108limengsong/fwmapPNG/blob/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20241009140809.png)

预览图：
二维数据总览
![image](https://github.com/200102108limengsong/fwmapPNG/blob/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20241009140753.png)
三维数据总览
![image](https://github.com/200102108limengsong/fwmapPNG/blob/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20241009140812.png)

气象模块
![image](https://github.com/200102108limengsong/fwmapPNG/blob/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20241009140726.png)
![image](https://github.com/200102108limengsong/fwmapPNG/blob/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20241009140735.png)

机场信息叠加
![image](https://github.com/200102108limengsong/fwmapPNG/blob/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20241009140739.png)
![image](https://github.com/200102108limengsong/fwmapPNG/blob/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20241009140737.png)

飞机数据叠加
![image](https://github.com/200102108limengsong/fwmapPNG/blob/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20241009140741.png)
![image](https://github.com/200102108limengsong/fwmapPNG/blob/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20241009140746.png)
![image](https://github.com/200102108limengsong/fwmapPNG/blob/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20241009140743.png)

德国空域数据叠加
![image](https://github.com/200102108limengsong/fwmapPNG/blob/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20241009140748.png)
![image](https://github.com/200102108limengsong/fwmapPNG/blob/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20241009140750.png)

三维飞机渲染
![image](https://github.com/200102108limengsong/fwmapPNG/blob/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20241009140806.png)

图例
![image](https://github.com/200102108limengsong/fwmapPNG/blob/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20241009140815.png)
![image](https://github.com/200102108limengsong/fwmapPNG/blob/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20241009140818.png)
![image](https://github.com/200102108limengsong/fwmapPNG/blob/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20241009140820.png)