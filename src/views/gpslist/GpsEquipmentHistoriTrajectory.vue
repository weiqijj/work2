<template>
  <div id="tol">
    <div id="hiscontai"></div>
    <div class="input-card">
      <h4>轨迹回放控制</h4>
      <div class="input-item">
        <input type="button" class="btn" value="开始动画" id="start" @click="startAnimation()" />
        <input type="button" class="btn" value="暂停动画" id="pause" @click="pauseAnimation()" />
      </div>
      <div class="input-item">
        <input type="button" class="btn" value="继续动画" id="resume" @click="resumeAnimation()" />
        <input type="button" class="btn" value="停止动画" id="stop" @click="stopAnimation()" />
      </div>
    </div>
  </div>
</template>

<script>
  let markerH='';
  let lineArr='';
  export default {
    mounted(){
      this.showTrajectoryInfo();
    },
    methods:  {
      //   页面加载完,开始异步引入高德地图
      //创建了一个回调函数,高德地图加载完毕会调用

      showTrajectoryInfo () {
        // 所有关于地图的逻辑全部都要写在这个回调里面
        // 保证高德地图加载完毕
          markerH,lineArr = [[116.478935,39.997761],[116.478939,39.997825],[116.478912,39.998549],
          [116.478912,39.998549],[116.478998,39.998555],[116.478998,39.998555],[116.479282,39.99856],
          [116.479658,39.998528],[116.480151,39.998453],[116.480784,39.998302],[116.480784,39.998302],
          [116.481149,39.998184],[116.481573,39.997997],[116.481863,39.997846],[116.482072,39.997718],
          [116.482362,39.997718],[116.483633,39.998935],[116.48367,39.998968],[116.484648,39.999861]];

        var mapH = new AMap.Map("hiscontai", {
          zoom: 16, //级别
          resizeEnable: true,
          center: [116.478935,39.997761], //中心点坐标
          viewMode: "3D", //使用3D视图
        });

        // 创建一个 markerH 实例：
         markerH = new AMap.Marker({
          map: mapH,
          position: [116.478935,39.997761],
          icon: "https://webapi.amap.com/images/car.png",
          offset: new AMap.Pixel(-26, -13),
          autoRotation: true,
          angle:-90,
        });

        // 绘制轨迹
        var polyline = new AMap.Polyline({
          map: mapH,
          path: lineArr,
          showDir:true,
          strokeColor: "#2288ff",  //线颜色
          // strokeOpacity: 1,     //线透明度
          strokeWeight: 6,      //线宽
          // strokeStyle: "solid"  //线样式
        });

        var passedPolyline = new AMap.Polyline({
          map: mapH,
          // path: lineArr,
          strokeColor: "#aaff00",  //线颜色
          // strokeOpacity: 1,     //线透明度
          strokeWeight: 6,      //线宽
          // strokeStyle: "solid"  //线样式
        });

        markerH.on('moving', function (e) {
          passedPolyline.setPath(e.passedPath);
        });

        mapH.setFitView();
        // 插件的添加,第一个参数为插件名,第二个为回调函数
        AMap.plugin("AMap.ToolBar", function () {
          //异步加载插件
          var toolbar = new AMap.ToolBar();
          mapH.addControl(toolbar);
        });

        markerH.moveAlong(lineArr, 200);

        // 将创建的点标记添加到已有的地图实例：
        mapH.add(markerH);
      },
      startAnimation () {
        markerH.moveAlong(lineArr, 200);
      },
      pauseAnimation () {
        markerH.pauseMove();
      },
      resumeAnimation () {
        markerH.resumeMove();
      },
      stopAnimation () {
        markerH.stopMove();
      },

    },
  };


</script>

<style>
  @import url('https://a.amap.com/jsapi_demos/static/demo-center/css/demo-center.css');
  #tol,#hiscontai {
    height: calc(100vh -  8.8em);
    width: 100%;
  }

  .input-card .btn {
    margin-right: 1.2rem;
    width: 9rem;
  }

  .input-card .btn:last-child {
    margin-right: 0;
  }

</style>
