<script setup>
import { onMounted, onUnmounted } from "vue";
import AMapLoader from "@amap/amap-jsapi-loader";
var toolBar,
  scale,
  controlBar,
  Geolocation,
  overView,
  MapType,
  ispointinring,
  polygon1,
  mousetool,
  dot,
  dotpath,
  polyEditor;
let map = null;
var path1 = [
  [116.475334, 39.997534],
  [116.476627, 39.998315],
  [116.478603, 39.99879],
  [116.478529, 40.000296],
  [116.475082, 40.000151],
  [116.473421, 39.998717],
];
onMounted(() => {
  AMapLoader.load({
    key: "8e8a107e8ac39b7d118741211cdb2b98", // 申请好的Web端开发者Key，首次调用 load 时必填
    version: "2.0", // 指定要加载的 JSAPI 的版本，缺省时默认为 1.4.15
    plugins: ["AMap.Scale"], // 需要使用的的插件列表，如比例尺'AMap.Scale'等
  })
    .then((AMap) => {
      const map = new AMap.Map("container", {
        // 设置地图容器id
        viewMode: "3D", // 是否为3D地图模式
        zoom: 11, // 初始化地图级别
        center: [116.397428, 39.90923], // 初始化地图中心点位置
      });

      AMap.plugin([
        "AMap.ToolBar",
        "AMap.Scale",
        "AMap.ControlBar",
        "AMap.Geolocation",
        "AMap.HawkEye",
        "AMap.MapType",
        "AMap.PolygonEditor",
        "AMap.Polygon",
        "AMap.MouseTool",
      ]);
      toolBar = new AMap.ToolBar({
        position: "LB",
        offset: [30, 60],
      });
      mousetool = new AMap.MouseTool(map);
      scale = new AMap.Scale();
      controlBar = new AMap.ControlBar({
        position: "RT",
      });
      Geolocation = new AMap.Geolocation({
        position: {
          top: "180px",
          right: "30px",
        },
      });
      overView = new AMap.HawkEye({
        position: "LT",
      });
      polygon1 = new AMap.Polygon({
        path: path1,
      });
      window.polygon1 = polygon1;
      window.mousetool = mousetool;
      map.addControl(scale);
      map.addControl(toolBar);
      map.addControl(controlBar);
      map.addControl(Geolocation);
      map.addControl(overView);
      scale.hide();
      toolBar.hide();
      controlBar.hide();
      overView.hide();
      map.add(polygon1);
      map.setFitView(polygon1);

      // ----------------------------------//

      polyEditor = new AMap.PolygonEditor(map);
      window.polyEditor = polyEditor;
      polyEditor.addAdsorbPolygons(polygon1);
      polyEditor.on("add", function (data) {
        console.log(data);
        var polygon = data.target;
        var polypath = polygon.getPath();
        console.log("New Polygon:", polygon.getPath());
        polyEditor.addAdsorbPolygons(polygon);
        polygon.on("dblclick", (event) => {
          var adjustedPolygon = event.target;
          window.selectedPolygon = adjustedPolygon; // 保存选中的多边形
          console.log("Selected Polygon:", selectedPolygon.getPath());
          polyEditor.setTarget(polygon);
          polyEditor.open();
        });
      });
      polygon1.on("dblclick", () => {
        polyEditor.setTarget(polygon1);
        polyEditor.open();
      });

      window.createPolygon = function () {
        polyEditor.close();
        polyEditor.setTarget();
        polyEditor.open();
      };
      window.removePolygon = function () {
        if (selectedPolygon) {
          // 在这里可以进行其他处理，例如从吸附多边形列表中移除
          console.log("Removing Polygon:", selectedPolygon.getPath());
          polyEditor.close(); // 关闭编辑器
          selectedPolygon.setMap(null); // 从地图上移除多边形
          selectedPolygon = null; // 重置选中的多边形
        } else {
          alert("请先选中一个多边形");
        }
      };
      window.setMarker = function () {
        mousetool.marker();
        mousetool.on("draw", (event) => {
          console.log("绘制完成");
          var markerPosition = event.obj.getPosition(); // 获取绘制的点的位置
          console.log(event);
          if (selectedPolygon) {
            var polypath = selectedPolygon.getPath();
            var isPointInside = AMap.GeometryUtil.isPointInRing(
              markerPosition,
              polypath
            );

            if (isPointInside) {
              console.log("点在多边形内");
              // 在此处执行点在多边形内的操作
            } else {
              console.log("点不在多边形内");
              // 在此处执行点不在多边形内的操作
            }
          } else {
            console.log("请先选中一个多边形");
          }
        });
        // ispointinring = AMap.GeometryUtil.isPointInRing();
      };
      map.on("rightclick", () => {
        mousetool.close();
        console.log("close is ok");
      });
      const marker = new AMap.Marker({
        position: new AMap.LngLat(116.39, 39.9),
        offset: new AMap.Pixel(-10, -10),
        icon: "//vdata.amap.com/icons/b18/1/2.png", // 添加 Icon 图标 URL
        title: "北京",
      });

      map.add(marker);
    })

    .catch((e) => {
      console.log(e);
    });
});
onUnmounted(() => {
  map?.destroy();
});

window.toggleScale = function (checkbox) {
  if (checkbox.checked) {
    scale.show();
  } else {
    scale.hide();
  }
};

window.toggleToolBar = function (checkbox) {
  if (checkbox.checked) {
    showToolBar();
  } else {
    hideToolBar();
  }
};
window.toggleControlBar = function (checkbox) {
  if (checkbox.checked) {
    document.getElementById("controlBar").checked = true;
    controlBar.show();
  } else {
    document.getElementById("controlBar").checked = false;
    controlBar.hide();
  }
};
window.toggleOverViewShow = function (checkbox) {
  if (checkbox.checked) {
    overView.show();
  } else {
    overView.hide();
  }
};
function showToolBar() {
  document.getElementById("toolbar").checked = true;
  toolBar.show();
}
function hideToolBar() {
  document.getElementById("toolbar").checked = false;
  toolBar.hide();
}
</script>
<template>
  <div id="container"></div>
  <div
    class="flex flex-col min-w-0 break-words bg-white bg-clip-border rounded-md shadow-md bottom-4 right-4 flex-1 p-3 fixed"
    style="width: 120px"
  >
    <button
      class="inline-block font-normal text-center whitespace-nowrap align-middle select-none border border-transparent bg-none text-blue-500 py-1 px-2 leading-[1.5] rounded-full appearance-none border-blue-500"
      onclick="setMarker()"
      style="margin-bottom: 5px"
    >
      标记点位
    </button>
    <button
      class="inline-block font-normal text-center whitespace-nowrap align-middle select-none border border-transparent bg-none text-blue-500 py-1 px-2 leading-[1.5] rounded-full appearance-none border-blue-500"
      onclick="createPolygon()"
      style="margin-bottom: 5px"
    >
      新建多边形
    </button>
    <button
      class="inline-block font-normal text-center whitespace-nowrap align-middle select-none border border-transparent bg-none text-blue-500 py-1 px-2 leading-[1.5] rounded-full appearance-none border-blue-500"
      onclick="polyEditor.open()"
      style="margin-bottom: 5px"
    >
      多边形编辑
    </button>
    <button
      class="inline-block font-normal text-center whitespace-nowrap align-middle select-none border border-transparent bg-none text-blue-500 py-1 px-2 leading-[1.5] rounded-full appearance-none border-blue-500"
      onclick="polyEditor.close()"
    >
      结束编辑
    </button>
    <button
      class="inline-block font-normal text-center whitespace-nowrap align-middle select-none border border-transparent bg-none text-blue-500 py-1 px-2 leading-[1.5] rounded-full appearance-none border-blue-500"
      onclick="removePolygon()"
    >
      删除多边形
    </button>
  </div>
  <!-- <div class="input-card">
    <div class="input-item">
      <input type="checkbox" onclick="toggleScale(this)" />比例尺
    </div>

    <div class="input-item">
      <input type="checkbox" id="toolbar" onclick="toggleToolBar(this)" />工具条
    </div>

    <div class="input-item">
      <input
        type="checkbox"
        id="controlBar"
        onclick="toggleControlBar(this)"
      />工具条方向盘
    </div>

    <div class="input-item">
      <input
        type="checkbox"
        id="overview"
        onclick="toggleOverViewShow(this)"
      />显示鹰眼
    </div>
  </div> -->
</template>

<style scoped>
#container {
  width: 100%;
  height: 100vh;
}
#myPageTop {
  position: absolute;
  top: 5px;
  right: 10px;
  background: #fff none repeat scroll 0 0;
  border: 1px solid #ccc;
  margin: 10px auto;
  padding: 6px;
  font-family: "Microsoft Yahei", "微软雅黑", "Pinghei";
  font-size: 14px;
}
#myPageTop label {
  margin: 0 20px 0 0;
  color: #666666;
  font-weight: normal;
}
#myPageTop input {
  width: 170px;
}
#myPageTop .column2 {
  padding-left: 25px;
}
</style>
