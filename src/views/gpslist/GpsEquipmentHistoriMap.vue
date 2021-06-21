<template>
  <div id="container">

  </div>
</template>


<script>


  export default {
    created() {

    },
    mounted(){
      this.showMapInfo();
      //this.queryCarInfoByImei();
    },

    methods: {
      //   页面加载完,开始异步引入高德地图
      //创建了一个回调函数,高德地图加载完毕会调用

      showMapInfo() {
        //点标签
        let markers=[];
        let positions=[];
        let licensePlates=[];
        let carColour=[];
        let engineNo=[];
        let address=[];
        let contacts=[];
        let contactsPhone=[];
        this.axios({//格式a
          method: 'get',
          url: '/sys/gpslist/gpsEquipmentHistoriData/queryAllInfoList'
        }).then(function (resp) {
          //console.log('最后resp：' + resp.result );
          for(let j = 0; j < resp.result.length; j++) {
            positions.push([resp.result[j].longitude,resp.result[j].latitude]);
            licensePlates.push([resp.result[j].licensePlate]);
            carColour.push([resp.result[j].carColour]);
            engineNo.push([resp.result[j].engineNo]);
            address.push([resp.result[j].address]);
            contacts.push([resp.result[j].contacts]);
            contactsPhone.push([resp.result[j].contactsPhone]);
          }
          //console.log('最后positions：' + positions );
          //

          // 所有关于地图的逻辑全部都要写在这个回调里面
          // 保证高德地图加载完毕
          //positions = [[114.048052,22.623256], [116.408273,39.940016], [116.25031,40.121629]];
         // console.log("进入onLoad事件"+positions);
          let map = new AMap.Map("container", {
            zoom: 10, //级别
            resizeEnable: true,
            center: positions[0], //中心点坐标
            viewMode: "3D", //使用3D视图
          });


          // 创建一个 Marker 实例：

          //console.log('positions.length：' + positions.length );
          //实例化信息窗体
          let infoWindow = new AMap.InfoWindow({
            isCustom: false,  //使用自定义窗体
            //content:  info.join("") ,
            offset: new AMap.Pixel(16, -35),
          });

          for (let i = 0; i < positions.length; i++) {
            //console.log('第i个元素为：' + positions[i] +' ---->'+i);
            let marker = new AMap.Marker({
              map: map,
              position: positions[i], // 地理位置经纬度
              title: "这里我这里位置", // 鼠标移上去时显示的内容
              offset: new AMap.Pixel(-13, -30),
              //offset: new AMap.Pixel(-100, -100), // 偏移量
              // 可以指定图标图片地址
              icon: "https://webapi.amap.com/images/car.png", // 添加 Icon 图标 URL
              // 可以自定义标记点显示的内容,允许插入html字符串
            });

            marker.setLabel({
              offset: new AMap.Pixel(1, 1),  //设置文本标注偏移量
              content: "<div >"+licensePlates[i]+"</div>", //设置文本标注内容
              direction: 'right' //设置文本标注方位
            });

            //活动为每个窗口添加信息
            let info = [];
            info.push("<div class=''><div>车辆信息</div> ");
            info.push("<div style=\"padding:7px 0px 0px 0px;\"><h4>车牌号:"+licensePlates[i]+"</h4>");
            info.push("<p class='input-item'>车辆颜色:"+carColour[i]+"    &nbsp;&nbsp;&nbsp;&nbsp;      车架号:"+engineNo[i]+"</p>");
            info.push("<p class='input-item'>联系人:"+contacts[i]+"       &nbsp;&nbsp;&nbsp;&nbsp;      联系电话:"+contactsPhone[i]+"</p>");
            info.push("<p class='input-item'>地址 :"+address[i]+" </p></div></div>");

            //鼠标点击marker弹出自定义的信息窗体
            AMap.event.addListener(marker, 'click', function () {
              //console.log("open事件:"+marker.getPosition());
              infoWindow.setContent(info.join(""));
              map.setZoomAndCenter(15, marker.getPosition());
              infoWindow.open(map, marker.getPosition());
            });
            markers.push(marker);
          }




          // 插件的添加,第一个参数为插件名,第二个为回调函数
          AMap.plugin("AMap.ToolBar", function () {
            //异步加载插件
            var toolbar = new AMap.ToolBar();
            map.addControl(toolbar);
          });
          // 将创建的点标记添加到已有的地图实例：
          map.add(markers);


        }).catch(resp => {
          console.log('请求失败：' + resp );
        });

      },
      queryCarInfoByImei() {
          //console.log("请求IMEI开始：");
          this.axios({//格式a
            params: {
              imei: 546211335488793
            },
            method: 'get',
            url: '/sys/gpslist/gpsEquipmentHistoriData/queryAllInfoList'
          }).then(function (resp) {
            console.log("请求IMEI最后："+resp.result.length);
            console.log("请求IMEI最后："+resp.result[0].licensePlate);
          }).catch(resp => {
            console.log('请求失败：' + resp );
          });
        }


    },

  };


</script>

<style>
  #container {
    height: calc(100vh - 8.8em);
    width: 100%;
  }
</style>
