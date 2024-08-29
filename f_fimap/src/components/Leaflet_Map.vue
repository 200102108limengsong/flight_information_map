<template>
  <div>

    <button @click="airportclick">AddAirport</button>
    <button @click="toggleAircraft">AddAircraft</button>
    <div id="Leaflet_Container" ref="mapContainer"></div>

    <Legend/>
  </div>
  <div>
      <information :airinformationdata="aircraftmap"></information>
    </div>
</template>

<script setup>
import "leaflet/dist/leaflet.css";
import L from "leaflet";
import { onMounted, ref, watch } from 'vue';
import { GetWeatherLayer } from '../scripts/WeatherLayer';

import 'leaflet-geosearch/dist/geosearch.css';
import 'leaflet-measure/dist/leaflet-measure.css';
import 'leaflet.fullscreen/Control.FullScreen.css';
import 'leaflet-geosearch/dist/geosearch.js';
import 'leaflet-measure/dist/leaflet-measure.cn.js';
import 'leaflet.fullscreen/Control.FullScreen.js';
import Legend from './Legend.vue';
import information from './infomation.vue'

//从home获取select值-天气
const props = defineProps({
  currentLayer: String,

});

const weatherLayer = ref(null);


let map;
//屏幕范围
const bounds = ref(null);

let planeTracksLayer;


////机场////
//机场可视
const AirportVisible = ref(true);

// 创建机场特征组，并使用更具描述性的名称
const smallAirports = new L.FeatureGroup();
const mediumAirports = new L.FeatureGroup();
const largeAirports = new L.FeatureGroup();
const seaplaneBases = new L.FeatureGroup();
const heliports = new L.FeatureGroup();
const balloonports = new L.FeatureGroup();
const closedAirports = new L.FeatureGroup();

// 创建一个数组，包含所有的机场特征组
const allAirportGroups = [
  smallAirports,
  mediumAirports,
  largeAirports,
  seaplaneBases,
  heliports,
  balloonports,
  closedAirports
];


//创建机场可视组
const smallAirportsMap = new L.FeatureGroup();
const mediumAirportsMap = new L.FeatureGroup();
const largeAirportsMap = new L.FeatureGroup();
const seaplaneBasesMap = new L.FeatureGroup();
const heliportsMap = new L.FeatureGroup();
const balloonportsMap = new L.FeatureGroup();
const closedAirportsMap = new L.FeatureGroup();
//包含所有可视组
const allAirportMaps = [
  smallAirportsMap,
  mediumAirportsMap,
  largeAirportsMap,
  seaplaneBasesMap,
  heliportsMap,
  balloonportsMap,
  closedAirportsMap
];

//定义机场的url链接
//const airporturl = 'http://localhost:8080/geoserver/geo_test1/ows?service=WFS&version=1.0.0&request=GetFeature&typeName=geo_test1%3Aairport_europe_wgs&maxFeatures=100000&outputFormat=application%2Fjson';
const airporturl = 'http://127.0.0.1:8000/api/airports/?format=json';
//获取geojson数据///
function fetchAicportsData(url) {
  fetch(url)
    .then(response => {
      if (!response.ok) {
        throw new Error('Network response was not ok');
      }
      return response.json();
    })
    .then(data => {
      console.log('data fetched and cached.');
      //添加数据

      addAirportToMap(data)////////
    })
    .catch(error => console.error('Error loading the GeoJSON data', error));
}

// 全局变量用于存储图标
let icons = {};
//创建图标
function createIcon(type) {
  if (!icons[type]) {
    switch (type) {
      case "small_airport":
        icons[type] = L.icon({
          iconUrl: '/src/assets/marker/小型机场.svg',
          iconSize: [15, 15],
          iconAnchor: [0, 8], // 指定图标上与地图标记位置对应的点。调整 iconAnchor 以适应图标尺寸
          popupAnchor: [25, -2],//指定弹出框相对于图标的偏移量
        });
        break;
      case "medium_airport":
        icons[type] = L.icon({
          iconUrl: '/src/assets/marker/中型机场.svg',
          iconSize: [22, 22],
          iconAnchor: [0, 8],
          popupAnchor: [25, -2]
        });
        break;
      case "seaplane_base":
        icons[type] = L.icon({
          iconUrl: '/src/assets/marker/水上飞机.svg',
          iconSize: [15, 15],
          iconAnchor: [0, 8],
          popupAnchor: [25, -2]
        });
        break;
      case "balloonport":
        icons[type] = L.icon({
          iconUrl: '/src/assets/marker/热气球.svg',
          iconSize: [15, 15],
          iconAnchor: [0, 8],
          popupAnchor: [25, -2]
        });
        break;
      case "heliport":
        icons[type] = L.icon({
          iconUrl: '/src/assets/marker/直升机场.svg',
          iconSize: [15, 15],
          iconAnchor: [0, 8],
          popupAnchor: [25, -2]
        });
        break;
      case "large_airport":
        icons[type] = L.icon({
          iconUrl: '/src/assets/marker/大型机场.svg',
          iconSize: [35, 35],
          iconAnchor: [0, 8],
          popupAnchor: [25, -2]
        });
        break;
      case "closed":
        icons[type] = L.icon({
          iconUrl: '/src/assets/marker/停用机场.svg',
          iconSize: [15, 15],
          iconAnchor: [0, 8],
          popupAnchor: [25, -2]
        });
        break;
      default:
        icons[type] = L.icon({
          iconUrl: '/src/assets/marker/default.png',
          iconSize: [10, 10],
          iconAnchor: [0, 8],
          popupAnchor: [25, -2]
        });
    }
  }
  return icons[type];
}

//创建弹出框
function createPopupContent(feature) {
  const content = `
    <div class="pop_1">
      <h2><strong>名字:</strong><span>${feature.properties.name}</span></h2>
      <h2><strong>机场类别:</strong> <span>${feature.properties.type}</span></h2>
      
    </div>
    <div class="pop_2">
      <p><strong>编号:</strong><span>${feature.properties.ident}</span></p>
      <p><strong>国家:</strong> <span>${feature.properties.iso_country}</span></p>
      <p><strong>地区编号:</strong> <span>${feature.properties.iso_region}</span></p>
      <p><strong>城市:</strong> <span>${feature.properties.municipality}</span></p>
    </div>
  `;
  return content;
}

//创建单一标记____有修改！！！
function createMarker(coordinates, icon, popupContent) {
  const marker = L.marker(coordinates, { icon });

  const popup = L.popup()
    .setContent(popupContent)
    .on('open', function (e) {
      const popup = e.popup;
      const popupSize = popup._contentNode.getBoundingClientRect();
      const latLng = popup.getLatLng();
      const offset = L.point(popupSize.width / 2, popupSize.height / 2);
      const newBounds = L.latLngBounds(latLng, latLng).pad(0.5); // 增加一些额外的空间

      // 如果弹出框不在地图视图内，则平移地图
      if (!marker._map.getBounds().contains(latLng)) {
        marker._map.fitBounds(newBounds);
      }
    });

  marker.on("click", function (e) {
    if (this._map) { // 检查地图是否已准备好
      const latLng = this.getLatLng();
      popup.setLatLng(latLng).openOn(this._map);
    }
  });
  marker.on("mouseout", function (e) {
    popup.close();
  });

  return marker;
}

// 添加标记到相应特征组
function addMarkerToFeatureGroup(marker, type) {
  switch (type) {
    case "small_airport":
      smallAirports.addLayer(marker);
      break;
    case "medium_airport":
      mediumAirports.addLayer(marker);
      break;
    case "seaplane_base":
      seaplaneBases.addLayer(marker);
      break;
    case "balloonport":
      balloonports.addLayer(marker);
      break;
    case "heliport":
      heliports.addLayer(marker);
      break;
    case "large_airport":
      largeAirports.addLayer(marker);
      break;
    default:
      closedAirports.addLayer(marker);
      break;
  }
}

function addAirportToMap(geojsonData) {
  //if (geojsonData.type === 'FeatureCollection') {

  geojsonData.forEach(feature => {

    if (feature.geometry.type === 'Point') {

      const [lng, lat] = feature.geometry.coordinates;
      const type = feature.properties.type;

      const popupContent = createPopupContent(feature);
      const icon = createIcon(type);
      const marker = createMarker([lat, lng], icon, popupContent);
      addMarkerToFeatureGroup(marker, type);
    }
  });

}

//获取屏幕zoom基本添加数据
function handleZoomLevels() {
  const zoom = map.getZoom();
  if (zoom < 7) {
    map.addLayer(largeAirportsMap);
  } else if (zoom < 9) {
    map.addLayer(largeAirportsMap);
    map.addLayer(mediumAirportsMap);
  } else if (zoom < 10) {
    map.addLayer(largeAirportsMap);
    map.addLayer(mediumAirportsMap);
    map.addLayer(smallAirportsMap);
    map.addLayer(heliportsMap);

  } else {
    map.addLayer(largeAirportsMap);
    map.addLayer(mediumAirportsMap);
    map.addLayer(smallAirportsMap);
    map.addLayer(closedAirportsMap);
    map.addLayer(seaplaneBasesMap);
    map.addLayer(balloonportsMap);
    map.addLayer(heliportsMap);
  }
}

// 定义一个函数来检查标记是否位于当前地图可视区域内，并添加到相应的特征组
function addMarkersIfInBounds(featureGroup, featureGroupMap) {
  featureGroup.eachLayer(function (layer) {
    if (layer instanceof L.Marker && bounds.value.contains(layer.getLatLng())) {
      featureGroupMap.addLayer(layer);
    }
  });
}

// 定义 airportclick 方法
const airportclick = () => {
  if (AirportVisible.value) {
    fetchAicportsData(airporturl);
    AirportVisible.value = false;
  } else {
    allAirportMaps.forEach(fg => {
      fg.clearLayers();
    });
    allAirportGroups.forEach(fg => {
      fg.clearLayers();
    });
    AirportVisible.value = true;
  }
};

////机场结束////


////飞机开始////
// 用于存储定时器
let refreshIntervalId;
const Aircraft = ref(true);
const aircraftmap = new L.FeatureGroup();
const aircraft = new L.FeatureGroup();

//创建飞机图标函数
function createAircraftIcon(iconUrl) {
  return L.icon({
    iconUrl: iconUrl,
    iconSize: [25, 25],
    iconAnchor: [0, 16],
    popupAnchor: [-0, -32]
  });
}
//创建飞机图标
const aircraftIcon1 = createAircraftIcon('/src/assets/aircraft/aircraft1.png');
const aircraftIcon2 = createAircraftIcon('/src/assets/aircraft/aircraft2.png');
const aircraftIcon3 = createAircraftIcon('/src/assets/aircraft/aircraft3.png');
const aircraftIcon4 = createAircraftIcon('/src/assets/aircraft/aircraft4.png');
const aircraftIcon5 = createAircraftIcon('/src/assets/aircraft/aircraft5.png');
const aircraftIcon6 = createAircraftIcon('/src/assets/aircraft/aircraft6.png');
const aircraftIcon7 = createAircraftIcon('/src/assets/aircraft/aircraft7.png');
const aircraftIcon8 = createAircraftIcon('/src/assets/aircraft/aircraft8.png');
const aircraftIcon9 = createAircraftIcon('/src/assets/aircraft/aircraft9.png');
const aircraftIcon0 = createAircraftIcon('/src/assets/aircraft/aircraft0.png');
const aircraftIconHighlight = createAircraftIcon('/src/assets/aircraft/aircraft_highlight.png');



const aircraft_url = 'http://localhost:8000/api/planesnow.geojson/?format=json';
//获取飞机数据
function fetchAircraftData() {
  fetch(aircraft_url)
    .then(response => {
      if (!response.ok) {
        throw new Error('Network response was not ok');
      }
      console.log('success get json')
      return response.json();
    })
    .then(geojsonData => addAirTotalToMap(geojsonData))
    .catch(error => console.error('Error loading the GeoJSON data :', error));
}

//根据高度选择图标
function getAircraftIconByAltitude(altitude) {
  if (altitude < 600) return aircraftIcon1;
  if (altitude === 0) return aircraftIcon0;
  if (altitude >= 600 && altitude < 2000) return aircraftIcon2;
  if (altitude >= 2000 && altitude < 4000) return aircraftIcon3;
  if (altitude >= 4000 && altitude < 6000) return aircraftIcon4;
  if (altitude >= 6000 && altitude < 8000) return aircraftIcon5;
  if (altitude >= 8000 && altitude < 10000) return aircraftIcon6;
  if (altitude >= 10000 && altitude < 12000) return aircraftIcon7;
  if (altitude >= 12000 && altitude < 13000) return aircraftIcon8;
  return aircraftIcon9;
}

//添加飞机事件
function setupMarkerEvents(marker, map) {
  let clickCount = 0;
  const originalIcon = marker.options.icon;
  const icao24 = marker.options.icao24;

  marker.on("click", function (e) {
    const markerInstance = e.target; // 获取标记实例
    clickCount++; // 每次点击递增点击次数
    if (clickCount === 1) {
      // 这是第一次点击
      (e.target || this).setIcon(aircraftIconHighlight);
      // 进行高亮
      aircraftmap.eachLayer(function (layer) {
        if (layer !== markerInstance) {
          map.removeLayer(layer);
        }
      });
      fetchPlaneTracks(icao24);
      // 移除图层
    } else if (clickCount === 2) {
      (e.target || this).setIcon(originalIcon);
      // 取消高亮
      aircraftmap.eachLayer(function (layer) {
        if (layer !== markerInstance) {
          map.addLayer(layer);
        }
      });
      // 加载飞机数据
      // 删除航线
      if (planeTracksLayer) {
        planeTracksLayer.clearLayers();
        map.removeLayer(planeTracksLayer);
        //planeTracksLayer = null; // 也可以设置为 null，以便下次点击时重新创建
      }
    } else {
      // 重置点击计数器
      clickCount = 0;
      // 可以在这里处理后续的点击，或者实现其他逻辑
    }
  });
}

//创建飞机弹窗
function createAircraftPopupContent(icao24, country, callsign) {
  return `
    <div class="pop_1">
      <p><strong>Icao24编号:</strong><span>${icao24}</span></p>
      <p><strong>呼号:</strong><span>${callsign}</span></p>
      <p><strong>国家:</strong><span>${country}</span></p>
    </div>
  `;
}

//获取飞机信息并加载到地图上
function addAirTotalToMap(geojsonData) {

  // 清除特征组中的所有飞机标记
  aircraft.clearLayers();

  geojsonData.forEach(feature => {
    if (feature.geometry.type === 'Point') {
      // 获取点的经纬度信息
      const [lng, lat] = feature.geometry.coordinates;
      // 在地图上添加标记
      const altitude = feature.properties.baro_altitude;
      const callsign = feature.properties.callsign;
      const icao24 = feature.properties.icao24;
      const country = feature.properties.origin_country;
      const speed = feature.properties.velocity;
      const time = feature.properties.time_position;
      const track = feature.properties.true_track;
      const vertical_rate = feature.properties.vertical_rate;

      const popupContent = createAircraftPopupContent(icao24, country, callsign)
      const icon = getAircraftIconByAltitude(altitude);
      const marker = L.marker([lat, lng], { icon: icon, icao24: icao24 }).bindPopup(popupContent);
      // 添加 tooltip 显示 ICAO24 编号
      marker.bindTooltip(callsign, {
        permanent: true,
        direction: 'bottom',
        className: 'aircraft-tooltip',
        offset: new L.Point(0, 10) // 调整偏移量以确保标签正确显示
      });

      setupMarkerEvents(marker, map); // 注意这里的 `this.map` 是组件的属性
      aircraft.addLayer(marker);
    }
  });

}

//按钮函数////
const toggleAircraft = () => {
  if (Aircraft.value) {
    fetchAircraftData();
    map.addLayer(aircraftmap);
    // 启动定时器
    refreshIntervalId = setInterval(() => {
      fetchAircraftData();
      console.log('rich')
    }, 10000); // 每隔 5 秒刷新数据
    Aircraft.value = false;
  } else {
    clearInterval(refreshIntervalId);
    aircraftmap.clearLayers();
    aircraft.clearLayers();
    Aircraft.value = true;
  }
}

////飞机结束////


//飞机轨迹函数
function fetchPlaneTracks(icao24) {
  fetch(`http://localhost:8000/api/planetracks/?format=json&icao24=${icao24}`)
    .then(response => response.json())
    .then(data => {
      console.log(data); // 输出 data 的结构

      // 清除旧的数据
      if (planeTracksLayer) {
        planeTracksLayer.clearLayers();
      }

      // 创建一个新的图层来存放飞机轨迹
      planeTracksLayer = L.layerGroup().addTo(map);

      // 使用坐标数组创建一条轨迹线
      const polylineOptions = {
        color: 'blue', // 设置颜色为红色
        weight: 2,   // 设置线宽为10像素
        opacity: 0.8,  // 设置不透明度为100%
        smoothFactor: 1,// 设置平滑因子
        dashArray: (10, 10)//设置虚线的样式。
      };

      // 遍历 data 数组中的每个特征
      const coordinates = data.map((feature, index) => {
        // 调整坐标顺序
        const coord = [feature.geometry.coordinates[1], feature.geometry.coordinates[0]];

        // 添加航点标记
        const marker = L.marker(coord).addTo(planeTracksLayer);

        // 为每个航点添加弹出窗口
        marker.bindPopup(`
          <strong>Call Sign:</strong> ${feature.properties.callsign}<br>
          <strong>True Track:</strong> ${feature.properties.true_track}<br>
          <strong>Velocity:</strong> ${feature.properties.velocity}<br>
          <strong>Time Position:</strong> ${feature.properties.time_position}<br>
          <strong>Barometric Altitude:</strong> ${feature.properties.baro_altitude}
        `);

        return coord;
      });

      // 使用坐标数组创建一条轨迹线
      const polyline = L.polyline(coordinates, polylineOptions).addTo(planeTracksLayer);

      // 调整地图的中心位置和缩放级别以适应坐标范围
      const bounds = L.latLngBounds(coordinates);
      map.fitBounds(bounds, { padding: [50, 50] });

      console.log(polyline)
    })
    .catch(error => console.error('Error fetching plane tracks:', error));
}





// 定义一个函数来处理地图移动事件
function handleMapMove() {
  bounds.value = map.getBounds();

  // 清除所有机场特征组中的图层
  allAirportMaps.forEach(fg => {
    map.removeLayer(fg);
    fg.clearLayers();
  });

  // 将可见范围内的机场添加到地图
  allAirportGroups.forEach((fg, index) => {
    addMarkersIfInBounds(fg, allAirportMaps[index]);
  });

  // 处理地图缩放级别的逻辑
  handleZoomLevels();

  //飞机
  aircraftmap.clearLayers();
  let counts = 0;
  aircraft.eachLayer(function (layer) {
    if (layer instanceof L.Marker) {
      // 获取标记的坐标
      if (bounds.value.contains(layer.getLatLng()) && (counts < 100)) {

        aircraftmap.addLayer(layer);
        counts++;
      }

    }

  });
  
  
}

//定义矩形边界框
function rectangleMap() {
  // 定义左上角和右下角的坐标
  var northWest = [62, -15]; // 左上角坐标
  var southEast = [36, 47]; // 右下角坐标
  // 使用这两个坐标创建一个LatLngBounds对象
  var bounds = L.latLngBounds(northWest, southEast);

  // 创建一个矩形并添加到地图上
  var rectangle = L.rectangle(bounds, { color: 'red' });

  rectangle = L.rectangle(bounds, {
    color: 'blue',
    weight: 5,
    opacity: 0.6,//填充颜色
    fill: false,//是否填充多边形
    fillColor: 'rgba(0, 0, 255, 0.2)',//填充颜色
    fillOpacity: 0.5,//填充不透明度
    dashArray: '5,15',//虚线的模式，例如'5, 10'表示5像素的实线后跟10像素的空白
    lineJoin: 'round',//线段交汇处的样式
  }).addTo(map);

  // 绑定一个工具提示
  rectangle.bindTooltip('空情数据服务区域', {
    permanent: false, // 默认为false，意味着只有在悬停时才显示
    direction: 'top', // 方向可以是'top', 'bottom', 'left', 'right'等
    opacity: 0.9, // 不透明度
    offset: [0, -10] // 相对于矩形的位置偏移
  });
}


//定义主地图
function initMap() {
   map = L.map(Leaflet_Container, {  // 假设 Leaflet_Container 是你的地图容器的 ID
    minZoom: 4,  // 设置最小缩放级别
    maxBounds: [[35, -15], [63, 69]],  // 设置最大边界
    maxBoundsViscosity: 1.0  // 确保用户无法拖动超出边界
  }).setView([47, 22], 5);



  var baseLayers = {
    "交通线路": L.tileLayer('https://{s}.tile.openstreetmap.de/tiles/osmde/{z}/{x}/{y}.png', {
      maxZoom: 18
    }).addTo(map),

    "暗色地图": L.tileLayer('http://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png', {
      maxZoom: 18
    }),

    "天地图基础图层": L.tileLayer('http://{s}.tianditu.gov.cn/DataServer?T=vec_w&x={x}&y={y}&l={z}&tk=81c20353cd8642da074545424165aecd', {
      subdomains: ['t0', 't1', 't2', 't3', 't4', 't5', 't6', 't7'],
      maxZoom: 18,
      attribution: '地图数据 &copy; <a href="http://www.tianditu.cn">天地图</a>'
    }),

    "高程地图": L.tileLayer('https://{s}.tile.opentopomap.org/{z}/{x}/{y}.png', {
      maxZoom: 18
    }),

    "遥感影像": L.tileLayer('	https://t0.tianditu.gov.cn/img_w/wmts?tk=81c20353cd8642da074545424165aecd&service=wmts&request=GetTile&version=1.0.0&layer=img&tilematrixSet=w&TileMatrix={z}&TileRow={y}&TileCol={x}', {
      attribution: '天地图影像数据', // 版权声明
      subdomains: ['t0', 't1', 't2', 't3', 't4', 't5', 't6', 't7'], // 子域，用于平衡负载
      maxZoom: 18, // 最大缩放级别
    }),

  };

  var layerControl = L.control.layers(baseLayers, {}, {
    position: 'topleft',
    collapsed: true
  }).addTo(map);







  //矩形边框
  rectangleMap()

  L.control.scale({
    position: 'topleft',
    maxWidth: '100',
    imperial: false
  }).addTo(map);




  // 监听地图移动事件，并控制机场加载
  map.on('move', handleMapMove);
  const aspUrl = 'http://127.0.0.1:8000/api/asps/?format=json';

  // 使用fetch获取数据
  fetch(aspUrl)
    .then(response => response.json())
    .then(data => {
      console.log('Data fetched and cached.');

      // 循环遍历数据
      data.forEach(feature => {
        if (feature.geometry.type === 'Polygon') {
          // 创建一个GeoJSON层
          const geoJsonLayer = L.geoJSON(feature, {
            style: function (feature) {
              return {
                color: 'blue',
                weight: 2,
                opacity: 1,
                fillOpacity: 0.5
              };
            },
            onEachFeature: function (feature, layer) {
              layer.bindPopup(feature.properties.id);
            }
          }).addTo(map);
        }
      });
    })
    .catch(error => console.error('Error fetching data:', error));


}


//天气图层
function setweatherLayer(layerName) {
  // 输出当前图层的值
  console.log('Current Layer:', layerName);

  // 检查当前是否有图层存在
  if (weatherLayer.value && map.hasLayer(weatherLayer.value)) {
    //console.log('Removing existing layer:', weatherLayer.value);
    // 移除当前存在的图层
    map.removeLayer(weatherLayer.value);
  }

  // 使用 GetWeatherLayer 函数获取对应的图层
  weatherLayer.value = GetWeatherLayer(layerName);
  //console.log(' Layer:', weatherLayer.value);

  // 添加新的图层
  if (weatherLayer.value) {
    //console.log('Adding new layer:', weatherLayer.value);
    weatherLayer.value.addTo(map);


  } else {
    console.log('No layer to add.');
  }
}

onMounted(() => {
  initMap();

  setweatherLayer(props.currentLayer);
  // 监听 currentLayer 的变化(天气)
  watch(() => props.currentLayer, (newVal) => {
    setweatherLayer(newVal);
  });
});



</script>

<style scoped>
#Leaflet_Container {
  height: 92.5vh;
  width: 100vw;
  bottom: 0;
}
</style>