<template>
    <div id="cesiumContainer">

    </div>
</template>

<script setup>
import * as Cesium from 'cesium'
import { onMounted } from 'vue';


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
            requestVertexNormals: false, //地形
            requestWaterMask: false,//水面
        }),
    })
    /*
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
    */

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

function getAircraftIconByAltitude(altitude) {
    // 根据海拔高度选择不同的图标
    if (altitude < 5000) {
        return '/src/assets/aircraft/aircraft2.png';
    } else if (altitude >= 5000 && altitude < 10000) {
        return '/src/assets/aircraft/aircraft6.png';
    } else {
        return '/src/assets/aircraft/aircraft8.png';
    }
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


})


</script>

<style scoped>
#cesiumContainer {
    width: 100%;
    height: 98vh;
}
</style>