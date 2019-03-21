> WebGL（全称 Web Graphics Library）是一种 3D 绘图协议，这种绘图技术标准允许把 JavaScript 和 OpenGL ES 2.0 结合在一起，通过增加 OpenGL ES 2.0 的一个 JavaScript 绑定，WebGL 可以为 HTML5 Canvas 提供硬件 3D 加速渲染，这样 Web 开发人员就可以借助系统显卡来在浏览器里更流畅地展示 3D 场景和模型了，还能创建复杂的导航和数据视觉化。Three.js 是一款开源的主流 3D 绘图 JS 引擎，它就像 jQuery 简化了 HTML DOM 操作一样，可以简化 WebGL 编程。

### 一、Three.js 坐标介绍

三维空间中主要有两种几何变换，一种是位置的变换，位置变换和二维空间的是一样的，坐标值进行增减就可。另一种就是旋转变换，二维空间的旋转可以看作是围绕点的旋转，只能基于平面上某个点为圆心进行旋转，而三维空间的旋转则是围绕一条线旋转的，有多个方向。Three.js 是自持旋转矩阵，欧拉角，四元数这三种旋转方式。

对于三维转换空间的坐标系，我的理解就是可以分为：世界坐标系、屏幕坐标系。世界坐标系就是空间坐标系，屏幕坐标系就是我们看到的平面上的坐标系，其实应该有个中间过度的坐标系，就是标准设备坐标系 [-1,1]，通过现有的数据处理函数来进行转换，这个只是个人理解，可能有偏差。下面就介绍如何进行世界坐标系和屏幕坐标系的转换。

**屏幕坐标转世界坐标：**

屏幕坐标是从左上角为原点，这里全景展示设备坐标是以画布的中心为原点，需要处理一下，得到相机坐标。

```
    function convertTo3DCoordinate(clientX,clientY){
        var mv = new THREE.Vector3(
            (clientX / window.innerWidth) * 2 - 1,
            -(clientY / window.innerHeight) * 2 + 1,
            0.5 );  //0.5可以改变，改变后获得的threejs坐标的x,y，z数值上会改变，但是差值上不会改变
        mv.unproject(this.camera);   //用相机反投影该向量  这个地方我的理解就是设备坐标[-1,1]转化为世界坐标，这个函数实现的是先将设备坐标转换为齐次坐标，之后与相机的MP矩阵的逆矩阵相乘。有兴趣的可以先看下线性代数。
        return mv;
    }

```

**世界坐标转屏幕坐标：**

```
    function convertTo2DCoordinate() {
        //获取网格模型boxMesh的世界坐标
        var worldVector = new THREE.Vector3(
            boxMesh.position.x,
            boxMesh.position.y,
            boxMesh.position.z
            );
        var standardVector = worldVector.project(camera);//用相机投影该向量，世界坐标转标准设备坐标
        var a = window.innerWidth / 2;
        var b = window.innerHeight / 2;
        var param = {}
        param.x = Math.round(standardVector.x * a + a);//标准设备坐标转屏幕坐标
        param.y = Math.round(-standardVector.y * b + b);//标准设备坐标转屏幕坐标
        return param
    }
    其中相机投影和反投影的源码为下：
    project: function () {
        var matrix = new Matrix4();
            return function project( camera ) {
            matrix.multiplyMatrices( camera.projectionMatrix, matrix.getInverse( camera.matrixWorld ) );
            return this.applyMatrix4( matrix );
        };
    }(),
    unproject: function () {
        var matrix = new Matrix4();
        return function unproject( camera ) {
            matrix.multiplyMatrices( camera.matrixWorld, matrix.getInverse( camera.projectionMatrix ) );
            return this.applyMatrix4( matrix );
        };
    }(),

```

**经纬度转化为空间坐标：**

```
    //经、纬度   球体半径
    function lgltToxyz(longitude,latitude,radius){
            //返回角度转换成弧度之后的值
        var lg = degToRad(longitude) , lt = degToRad(latitude);
        var y = radius * Math.sin(lt);
        var temp = radius * Math.cos(lt);
        var x = temp * Math.sin(lg);
        var z = temp * Math.cos(lg);
        return {x:x , y:y ,z:z}
    }

```

### 二、全景图展示的原理

全景图是一种广角图，它的原理是等距圆柱投影，引用百科的解释：

> 等距圆柱投影，又称方格投影，是假想球面与圆筒面相切于赤道，赤道为没有变形的线。经纬线网格，同一般正轴圆柱投影，经纬线投影成两组相互垂直的平行直线。其特性是：保持经距和纬距相等，经纬线成正方形网格；沿经线方向无长度变形；角度和面积等变形线与纬线平行，变形值由赤道向高纬逐渐增大。该投影适合于低纬地区制图。

说白了就是将一个球体上的所有的点，全部投影到一个圆柱体的侧面上去，圆柱侧面展开图上包含了球体上所有的像素点，所以，一张标准的全景图的长款比例为 2:1。

现在明白了全景图的形成过程，那么我们要做的就是展示全景图，怎么办，就是进行圆柱侧面投影的逆运算，将这个过程反转一下！所以我们就可以运用 WebGL 绘制一个球体，然后将全景图直接贴在球体上就可以了，当然，需要进行映射的处理过程。

贴图的过程：

```
    //新建一个球体 （球体半径 /水平方向上分段数/和垂直方向上分段数）
    //这里分割数就脑补一下吧   想想西瓜上的条纹，水平方向上再脑补上就出来了这个概念
     var geometry = new THREE.SphereGeometry( 500, 100, 100 );
    //沿x轴进行-1的scale，让球体的面朝内（因为我们将从球内进行观看）。 
    geometry.scale( - 1, 1, 1 ); 
    //加载图片, 新建材质
    var material = new THREE.MeshBasicMaterial( {
    //设置纹理贴图
     map: new THREE.TextureLoader().load( '1.jpg' )
      } ); 
    //将几何体和材质进行结合。 
    mesh = new THREE.Mesh( geometry, material );
    //场景添加进去
    scene.add( mesh )
到这里，再设置下相机为球体的中心点

    var camera;
        function initCamera() {
            camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 1, 10000);
            camera.target = new THREE.Vector3( 0, 100, 0 );
        }

```

基本上我们已经可以展示全景图了，还可以再优化一下，因为球状模型的顶点与面的数量十分庞大， 这些元素的数量越多, 耗费的浏览器资源就会越多。我们可以创建一个立方体，将一张全景图分割为立方体的六个面，贴到立方体上，如果这个立方体足够大，我们就不足以感觉到它的边际存在，这样就避免了两极像素点的重叠。代码如下：

```
    let loader = new THREE.TextureLoader();  //设置纹理贴图
    //全景图分解
    const imgList = [
        'top.png',
        'right.png',
        'bottom.png',
        'left.png',
        'front.png',
        'behind.png'
    ];
    //采用异步编程
    Promise.all(imgList.map(function (val) {
        //加载图片, 新建材质, 传给下一个步骤.
        return new Promise(function (resolve, reject) {
            loader.load(val, function (texture) {
                        //定义材料外观的对象
                resolve(new THREE.MeshBasicMaterial({
                    map: texture,
                    side: THREE.BackSide
                }));
            });
        });
    })).then(function (materials) {
        //创建正方体
        geometry = new THREE.BoxGeometry(200, 200, 200);
        cube = new THREE.Mesh(
            geometry,
            new THREE.MeshFaceMaterial(materials) //可以在该容器中为物体的各表面贴上图片
        );
        scene.add(cube);
        animate();
    });

```

最后通过 WebGLRenderer 就可以了。

### 三、photo-sphere-viewer 介绍

前面介绍了全景图的相关背景，接下来开始使用 photo-sphere-viewer（以下简称 view.js）。three.js 可以理解为 jq，view.js 就是 jq 的插件。

需要的资源：view.min.css、three.js、D.min.js、uEvent.js、doT.js、CanvasRenderer.js、Projector.js、DeviceOrientationControls.js、view.min.js、zepto_1.1.3.js。

参数介绍：

- `panorama`：（必选）全景图的路径。
- `container`：（必选）放置全景图的容器。
- `autoload`：（默认为 true）true 为自动加载全景图，false 为迟点加载全景图（通过load 方法）。
- `usexmpdata`：（默认值为 true）photo sphere viewer 是否必须读入 xmp 数据，false 为不必须。
- `cors_anonymous`：（默认值为 true）true 为不能通过 cookies 获得用户
- `pano_size`：（默认值为 null）全景图的大小，是否裁切。
- `default_position`：（默认值为 0）定义默认位置，用户看见的第一个点，例如：{long: math.pi, lat: math.pi/2}。
- `min_fov`：（默认值为 30）观察的最小区域，单位 degrees，在 1-179 之间。
- `max_fov`：（默认值为 90）观察的最大区域，单位 degrees，在 1-179 之间。
- `allow_user_interactions`：（默认值为 true）设置为 false，则禁止用户和全景图交互（导航条不可用）。
- `allow_scroll_to_zoom`：（默认值为 true）若设置为 false，则用户不能通过鼠标滚动进行缩放图片。
- `tilt_up_max`：（默认值为 math.pi/2）向上倾斜的最大角度，单位radians。
- `tilt_down_max`：（默认值为 math.pi/2）向下倾斜的最大角度，单位 radians。
- `min_longitude`：（默认值为 0）能够展示的最小经度。
- `max_longitude`：（默认值为 2PI）能够展示的最大维度。
- `zoome_level`：（默认值为 0）默认的缩放级别，值在 0-100 之间。
- `long_offset`：（默认值为 PI/360）mouse/touch 移动时每像素经过的经度值。
- `lat_offset`：（默认值为 PI/180）mouse/touch 移动时每像素经过的纬度值。
- `time_anim`（默认值为 2000）全景图在 time_anim 毫秒后会自动进行动画。（设置为false禁用它）
- `reverse_anim`：（默认值为 true）当水平方向到达最大/最小的经度时，动画方向是否反转（仅仅是不能看到完整的圆）。
- `anim_speed`：（默认值为 2rpm）动画每秒/分钟多少的速度。
- `vertical_anim_speed`：（默认值为2rpm）垂直方向的动画每秒/分钟多少的速度。
- `vertical_anim_target`：（默认值为0）当自动旋转时的维度，默认为赤道。
- `navbar`：（默认为false）显示导航条。
- `navbar_style`：（默认值为false）导航条的样式。有效的属性：
- `backgroundColor`：导航条背景色（默认值rgba(61, 61, 61, 0.5)）；
- `buttonsColor`：按钮前景色（默认值 rgba(255, 255, 255, 0.7)）；
- `buttonBackgroundColor`：按钮激活时的背景色（默认值 rgba(255, 255, 255, 0.1)）；
- `buttonsHeight`：按钮高度，单位px（默认值 20）；
- `autorotateThickness`：自动旋转图片的层（默认值 1）；
- `zoomRangeWidth`：缩放游标的宽度，单位px（默认值 50）；
- `zoomRangeThickness`：缩放游标的层（默认值 1）；
- `zoomRangeDisk`：缩放游标的放大率，单位 px（默认值 7）；
- `fullscreenRatio`：全屏图标的比例（默认值 4/3）；
- `fullscreenThickneee`：全屏图片的层，单位 px（默认值 2）
- `loading_msg`：（默认值为 Loading...）加载信息。
- `loading_img`：（默认值为 null）loading 图片的路径。
- `loading_html`：（默认值 为null）html 加载器（添加到容器中的元素或字符串）。
- `size`：（默认值为 null）全景图容器的最终尺寸，例如 {width: 500, height: 300}。
- `onready`：（默认值为 null）全景图准备好并且第一张图片展示出来后的回调函数。

方法介绍：

- addAction()：添加事件（插件没有提供执行事件的方法，似乎是提供给插件内部使用的）。
- fitToContainer()：调整全景图容器大小为指定大小。
- getPosition()：获取坐标经纬度。
- getPositionInDegrees()：获取经纬度度数。
- getZoomLevel()：获取缩放级别。
- load()：加载全景图（）。
- moveTo(longitude, latitude)：根据经纬度移动到某一点。
- rotate(dlong, dlat)：根据经纬度度数移动到某一点。
- toggleAutorotate()：是否开启全景图自动旋转。
- toggleDeviceOrientation()：是否开启重力感应方向控制。
- toggleFullscreen()：是否开启全景图全屏。
- toggleStereo()：是否开启立体效果（可用于 WebVR 哦）。
- zoom(level)：设置缩放级别。
- zoomIn()：放大。
- zoomOut()：缩小。

使用：

首先创建一个 div 来作为渲染区域

```
    <div id="photosphere"></div>
    //初始化
    var PSV = new PhotoSphereViewer({
    container: 'photosphere',
                    panorama: panos[0].url
    })

```

### 四、创建标记

展示全景图的时候一般我们会遇到如下场景：展示一个大范围的区域，里面包含了很多需要介绍的详细地点，所以我们可以通过创建标记点，来给出相关的信息展示。

使用：

```
    //定义在new里面  自执行
    markers: (function() {
               return common();  //将创建marker  封装
            }())
    function common() {
           var a = [];
           for(var i = 0; i < Math.PI * 2; i += Math.PI / 4) {
              for(var j = -Math.PI / 2 + Math.PI / 4; j < Math.PI / 2; j += Math.PI / 4) {
                   a.push({
                       id: '#' + a.length,
                       tooltip: '点我啊',  //点击的提示信息
                       latitude: j,    //经纬度   可以使用 x, y替换   代表全景图上的像素坐标
                       longitude: i,
                       image: '../img/pin2.png',   //图片标记背景图
                       width: 32,   //宽高
                       height: 32,
                       anchor: 'bottom center',
                       data: {  //设置数据
                           deletable: true
                       }
                   });
                  }
               }
               return a  //marker接受数组   所以以数组的形式反出
         }

```

自定义 SVG：

```
    a.push({
        id: 'circle',   //id必选
        tooltip: 'A circle of radius 30',
        circle: 20,  //半径
        svgStyle: {
            fill: 'rgba(255,255,0,0.3)',
            stroke: 'yellow',
            strokeWidth: '2px'
        },
        //                      longitude: 0.14842681258549928,
        //                      latitude: -0.8678522571819425,
        x: 8395,
        y: 6827,
        anchor: 'center right'
    });
自定义html

    a.push({
        id: 'text',
        longitude: -0.5,
        latitude: -0.28,
        html: '♥爱你爱你么么哒♥',  
        anchor: 'bottom right',
        style: {  //定义样式
        maxWidth: '320px',
        color: 'red',
        fontSize: '20px',
        fontFamily: 'Helvetica, sans-serif',
        textAlign: 'center'
        },
        tooltip: {  //提示
        content: 'An HTML marker',
        position: 'right'
        }
    });

```

### 五、景图的自动播放加背景音乐

```
    <audio src="../libs/source.mp3" id="audios" style="position: absolute;left: 99999999999999px;top: 999999999px;" loop="true"></audio>

    PSV.on('ready',function(){  //准备就绪
         PSV.toggleAutorotate();   //自动播放
         $('#audios')[0].play();   //播放背景音乐
         $('.scene').hide();       //loading   下面提到
         $('.playBtn').show();   //音乐播放按钮
     });

```

### 六、图片缓存，场景切换

设置 cache_texture:number，参数为缓存图片，加载后不会重复加载，number 自定义。

点击 marker 进行场景的切换：

```
    PSV.on('select-marker', function(marker) { //监听marker点击事件。
    //进行场景的切换
    PSV.setPanorama(url, {longitude: 3.848,
                        latitude: -0.244}, true)
                            .then(function() {
                                $('.back').show();
                                $('.scene').hide();
                                PSV.setCaption('场景描述');
                            });
    })

    setPanorama参数：图片地址、下一个场景的初始经纬度、transition 默认（false）
还可以设置

    navbar: [
              'autorotate', 'zoom', 'download', 'markers',
              'spacer-1',
              {
                  title: 'Change image',
                  className: 'custom-button',
                  content: '点我',
                  onClick: (function() {
                      //自定义逻辑
                  }())
              }]

```

来自定义导航的内容。

再补充一下 panorama 可以接受一个数组（六张 url），也可以接受一个单独的图片 url。如果一张的话在 `_loadEquirectangularTexture` 这个函数里面内部进行了裁剪，如果是一个数组，直接渲染，提高了性能。

切换场景的过程中加载 loading，因为我们开启了缓存，所以需要设置一个变量（开关思想）来控制 loading 框的显示次数，比如第一次场景 1 进入场景 2，需要 loading 加载，但是从场景 2 再跳回到场景 1 因为缓存就不不要 loading，只加载一次。loading 的制作是通过 svg 制作的，这里没有贴出代码。

源代码下载地址：

> <https://download.csdn.net/download/susuzhe123/10454743>