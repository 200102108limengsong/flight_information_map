<template>
  <div id="cesiumContainer">

  </div>
</template>

<script setup>
import * as Cesium from 'cesium'
import { onMounted } from 'vue';
import 'cesium/Build/Cesium/Widgets/widgets.css';

//初始化地图
function InitMap() {
  Cesium.Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI0Y2Q4MzVkNi1lM2JiLTRjYjItYjhkNi02ZGQ1MDc3ZDFlZjAiLCJpZCI6MTYwMjY5LCJpYXQiOjE3MTEyNjA5ODV9.y62hzpo7Asf7GG5dfhII6vRBTsRua6MeAmGqtSq6Sv4';
  const viewer = new Cesium.Viewer("cesiumContainer", {
    imageryProvider: new Cesium.IonImageryProvider({ assetId: 2 }),
    geocoder: true, //位置查找工具,改为false会隐藏        
    animation: false,
    timeline: false,
    shouldAnimate: true,
    sceneModePicker: true,//视角模式切换工具
    navigationHelpButton: true,//导航帮助工具
    terrain: Cesium.Terrain.fromWorldTerrain({
      requestVertexNormals: true, //地形
      requestWaterMask: true,//水面
    }),
    terrainShadows: Cesium.ShadowMode.ENABLED//地图阴影
  })
  viewer.scene.globe.enableLighting = true;//根据太阳表面光照

  
  //添加OSM建筑物
  Cesium.createOsmBuildingsAsync().then(function (osmBuildings) {
    viewer.scene.primitives.add(osmBuildings);
  });


  // 添加天地图标注图层
  const tdtAnnotation = new Cesium.WebMapTileServiceImageryProvider({
    url: 'http://t0.tianditu.gov.cn/cia_w/wmts?service=WMTS&request=GetTile&version=1.0.0&LAYER=cia&STYLE=default&TILEMATRIXSET=w&FORMAT=tiles&tk=7068a62cddf0ed4f16dac9a506b80050',
    layer: 'cia',
    style: 'default',
    format: 'tiles',
    tileMatrixSetID: 'w',
    maximumLevel: 18,
  });
  viewer.imageryLayers.addImageryProvider(tdtAnnotation);

  // 隐藏logo信息
  viewer._cesiumWidget._creditContainer.style.display = 'none'

  
 
  // 设置视图到指定的经纬度和高度
  viewer.camera.setView({
    destination: Cesium.Cartesian3.fromDegrees(15, 50, 5000000), // 1000000 米高度
  });

  fetchAndUpdateAircraftData(viewer);
}

const aircraft_url = 'http://localhost:8000/api/planesnow.geojson/?format=json';
async function fetchAndUpdateAircraftData(viewer) {
    try {
        const response = await fetch(aircraft_url);
        const geojsonData = await response.json();

        // 打印完整的响应数据
        console.log('Full GeoJSON Response:', geojsonData);

        updateAirTotalToMap(viewer, geojsonData);
    } catch (error) {
        console.error('Error loading the GeoJSON data:', error);
    }
}

// 更新飞机数据
async function updateAirTotalToMap(viewer, geojsonData) {
    // 清除旧的实体
    viewer.entities.removeAll();
    console.log(viewer.entities)
    geojsonData.forEach(feature => {
        if (feature.geometry.type === 'Point') {
            // 获取点的经纬度信息
            const [lng, lat] = feature.geometry.coordinates;
            const altitude = feature.properties.baro_altitude;
            const callsign = feature.properties.callsign;
            const icao24 = feature.properties.icao24;
            const country = feature.properties.origin_country;
            const speed = feature.properties.velocity;
            const time = feature.properties.time_position;
            const track = feature.properties.true_track;
            const vertical_rate = feature.properties.vertical_rate;
            // 构建描述信息
            const popupContent = createAircraftPopupContent(icao24, country, altitude, callsign, speed, time, track, vertical_rate);


            //const icon = getAircraftIconByAltitude(altitude);
            const modelUrl=getModelUrlByAltitude(altitude);
            const entity = viewer.entities.add({
                name: callsign,
                position: Cesium.Cartesian3.fromDegrees(lng, lat, altitude),
                model: {
                    uri: modelUrl,
                    minimumPixelSize: 150, // 最小像素大小
                    maximumScale: 100,   // 最大缩放比例
                    scale: 0.1,           // 默认缩放比例
                },
                description: popupContent,


            });
        }
    });
}

function createAircraftPopupContent(icao24, country, altitude, callsign, speed, time, track, vertical_rate) {
    return `<b>ICAO24:</b> ${icao24}<br/>
        <b>Country:</b> ${country}<br/>
        <b>Callsign:</b> ${callsign}<br/>
        <b>Altitude:</b> ${altitude} m<br/>
        <b>Speed:</b> ${speed} m/s<br/>
        <b>Last Position Time:</b> ${time}<br/>
        <b>True Track:</b> ${track}°<br/>
        <b>Vertical Rate:</b> ${vertical_rate} m/min`;

}


// 根据海拔高度选择不同的模型文件
function getModelUrlByAltitude(altitude) {
  if (altitude < 3000) {
    return '/src/assets/aircraft/AircraftRed.gltf';
  } else if (altitude >= 3000 && altitude < 9000) {
    return '/src/assets/aircraft/AircraftBlue.gltf';
  } else if (altitude >= 9000 ) {
    return '/src/assets/aircraft/AircraftWhite.gltf';
  }
}

onMounted(() => {
  InitMap();

  // 创建矩形
  //createRectangle(viewer);
})


function createRectangle(viewer) {
  // 定义左上角和右下角的坐标
  var northWest = Cesium.Cartographic.fromDegrees(-15, 62); // 左上角坐标
  var southEast = Cesium.Cartographic.fromDegrees(47, 36); // 右下角坐标

  // 使用这两个坐标创建一个矩形
  var rectangle = new Cesium.RectangleGraphics({
    rectangle: new Cesium.Rectangle(
      Cesium.Math.toRadians(southEast.longitude), // 最小经度
      Cesium.Math.toRadians(northWest.latitude), // 最小纬度
      Cesium.Math.toRadians(northWest.longitude), // 最大经度
      Cesium.Math.toRadians(southEast.latitude)  // 最大纬度
    ),
    material: Cesium.Color.BLUE.withAlpha(0.2),
    outline: true,
    outlineColor: Cesium.Color.BLUE.withAlpha(0.6),
    outlineWidth: 5
  });

  // 添加矩形到实体
  viewer.entities.add({
    name: 'Rectangle Entity',
    rectangle: rectangle,
    description: new Cesium.CallbackProperty(function (time, result, index) {
      return '<b>空情数据服务区域</b>';
    }, false)
  });
}


</script>

<style scoped>
#cesiumContainer {
  width: 100%;
  height: 100vh;
}
</style>