<template>
  <div id="centerDiv" class="mapcontainer">
    <div class="infoview">
      <button @click="plainArea">绘制区域</button>
      <button @click="plainPloy">画线</button>
      <button @click="plainCirl">画圆</button>
      <button @click="closePlainArea">清除绘制区域</button>
      <button @click="cluster">聚合</button>
      <button @click="unCluster">非聚合</button>
    </div>
    <Map :url="configUrl" @onload="onMapload" />
  </div>
</template>

<script>
import Map from '../components/mars2d/Map.vue'
let canvasMarkerLayer = new mars2d.layer.CanvasMarkerLayer();//画布
//创建矢量数据图层
let graphicLayer = new mars2d.layer.GraphicLayer({
  // hasEdit: true,
  // isAutoEditing: true, //绘制完成后是否自动激活编辑
})
let clusterLayer = new mars2d.layer.ClusterLayer({
  chunkedLoading: true, //间隔添加数据，以便页面不冻结。
  showCoverageOnHover: false, //是否显示聚合标记的边界。
  disableClusteringAtZoom: 18, //此级别下不聚合
  maxClusterRadius: 100,
})
export default {
  name: 'Index',
  components: {
    Map
  },
  data() {
    const basePathUrl = window.basePathUrl || ''
    return {
      configUrl: basePathUrl + 'config/config.json',
      map: {},
    }
  },

  methods: {
    // 地图构造完成回调
    onMapload(map) {

      this.map = map
      // var mapSwich = new mars2d.control.MapSwich({
      //   selected: mars2d.control.MapSwich.Type.Map3D,
      //   // hasPano: true,
      // });
      // map.addControl(mapSwich);    
      this.map.addLayer(canvasMarkerLayer)//图层等级较低
      this.map.addLayer(graphicLayer)
      this.map.addLayer(clusterLayer)
      this.showArea()
      this.addMarkers(10)
    },
    cluster() {
      graphicLayer.remove()
      graphicLayer.eachGraphic(function (event) {
        if (event.type == 'marker') {
          clusterLayer.addGraphic(event)
        }
      })
    },
    unCluster() {
      clusterLayer.clearLayers()
      this.map.addLayer(graphicLayer)
    },
    fitView() {
      if (this.map) {
        this.map.zoomIn(2)
      }
    },
    addMarkers(numPoints) {
      for (let i = 0; i < numPoints; i++) {
        let latlng = this.getRandomLatLng();
        let graphic = mars2d.graphic
        let marker = new graphic.Marker({
          latlng: latlng,
          style: {
            image: "img/problemplain.png",
            width: 60,
            height: 60,
            draggable: false
          },
        });
        let label = new graphic.Label({
          latlng: latlng,
          style: {
            text: `第${i}个点`,
            className: 'graphic-label',
            verticalOrigin: mars2d.VerticalOrigin.BOTTOM,
            horizontalOrigin: mars2d.HorizontalOrigin.CENTER,
          }
        })
        graphicLayer.addGraphic(label)
        graphicLayer.addGraphic(marker);
      }
    },
    getRandomLatLng() {
      let bounds = this.map.getBounds().pad(0.5),
        southWest = bounds.getSouthWest(),
        northEast = bounds.getNorthEast(),
        lngSpan = northEast.lng - southWest.lng,
        latSpan = northEast.lat - southWest.lat;

      return L.latLng(southWest.lat + latSpan * Math.random(), southWest.lng + lngSpan * Math.random());
    },
    showArea() {
      let color = ['#F51515', '#2136A6', '#FEBF00', '#00E5C1', '#2B92C9', '#DAC3FF', '#199F86', '#83CADE', '#FE6700', '#C368E2', '#9014FF', '#7EB712', '#0EA5AA']
      let i = 0
      let geoJsonLayer = new mars2d.layer.GeoJsonLayer({
        name: "安徽各市",
        url: "http://data.mars2d.cn/file/geojson/areas/340100_full.json",
        symbol: {
          type: "polygon",
          styleOptions: {
            fill: true,
            fillOpacity: 0.3,
            outline: true,
            outlineWidth: 2,
            outlineOpacity: 1,
            outlineColor: "white",
            fillColor: 'pink'
          },
          callback: function (attr, styleOpt) { return { fillColor: color[i++] }; }
        },
        flyTo: true,
      });
      this.map.addLayer(geoJsonLayer);
    },
    plainArea() {
      graphicLayer.startDraw({
        type: "polygon",
        style: {
          fill: true,
          fillColor: "#000dfc",
          fillOpacity: 0.3,
          outline: true,
          outlineWidth: 2,
          outlineColor: "#254dc4",
          outlineOpacity: 1,
        },
        success: function (graphic) {
          console.log("标绘完成", graphic);
        },
      });
    },
    plainPloy() {
      graphicLayer.startDraw({
        type: "polyline",
        style: {
          width: 3,
          color: "#000dfc",
        },
      });
    },
    plainCirl() {
      graphicLayer.startDraw({
        type: "circle",
        style: {
          fill: true,
          fillColor: "#000dfc",
          fillOpacity: 0.3,
          outline: true,
          outlineWidth: 2,
          outlineColor: "#254dc4",
          outlineOpacity: 1,
        },
        success: function (graphic) {
          console.log("标绘完成", graphic);
        },
      });

      //加一些演示数据
      this.addGraphic_01(graphicLayer)
    },
    addGraphic_01(graphicLayer) {

      var graphic = new mars2d.graphic.Circle({
        latlng: [31.854628, 117.245425],
        style: {
          radius: 6500, //单位：米
          fill: true,
          fillColor: "#00ffff",
          fillOpacity: 0.3,
          outline: true,
          outlineWidth: 2,
          outlineColor: "#00f",
          outlineOpacity: 0.5,
        },
      });

      graphicLayer.addGraphic(graphic);
    },
    closePlainArea() {
      graphicLayer.clear()
    }

  }
}
</script>

<style >
.mapcontainer {
  position: relative;
  height: 100%;
  overflow: hidden;
}
.infoview {
  position: absolute;
  z-index: 999;
  left: 50%;
}
button {
  display: inline-block;
  padding: 10px 20px;
}
.graphic-label {
  padding: 0px 10px;
  background-color: #fff;
  border: solid 1px #000;
  margin-top: -48px;
}
</style>
