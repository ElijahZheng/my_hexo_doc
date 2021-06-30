title: three.js 初识
author: Elijah Zheng
tags:
  - three
  - 图形学
categories: []
date: 2017-11-30 14:07:00
---
什么是threejs，很简单，你将它理解成three + js就可以了。three表示3D的意思，js表示javascript的意思。那么合起来，three.js就是使用javascript 来写3D程序的意思。

<!--more-->

在Three.js中，要渲染物体到网页中，我们需要3个组建：场景（scene）、相机（camera）和渲染器（renderer）。有了这三样东西，才能将物体渲染到网页中去。

创建这三要素的代码如下：
```js
var scene = new THREE.Scene();  // 场景
var camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);// 透视相机
var renderer = new THREE.WebGLRenderer();   // 渲染器
renderer.setSize(window.innerWidth, window.innerHeight);    // 设置渲染器的大小为窗口的内宽度，也就是内容区的宽度
document.body.appendChild(renderer.domElement);
```

场景是所有物体的容器，如果要显示一个苹果，就需要将苹果对象加入场景中。

相机，相机决定了场景中那个角度的景色会显示出来。相机就像人的眼睛一样，人站在不同位置，抬头或者低头都能够看到不同的景色。场景只有一种，但是相机却又很多种。

渲染器决定了渲染的结果应该画在页面的什么元素上面，并且以怎样的方式来绘制。

点
```js
var point1 = new THREE.Vector3();

point1.set(4,8,9);
```

线 
```js
var geometry = new THREE.Geometry();
var material = new THREE.LineBasicMaterial( { vertexColors: true } );
var color1 = new THREE.Color( 0x444444 ), color2 = new THREE.Color( 0xFF0000 );

// 线的材质可以由2点的颜色决定
var p1 = new THREE.Vector3( -100, 0, 100 );
var p2 = new THREE.Vector3(  100, 0, -100 );
geometry.vertices.push(p1);
geometry.vertices.push(p2);
geometry.colors.push( color1, color2 );
var line = new THREE.Line( geometry, material, THREE.LinePieces );
scene.add(line);
```

```js
var geometry = new THREE.Geometry();
```

几何体里面有一个vertices变量，可以用来存放点。

```js
LineBasicMaterial( parameters )

Parameters是一个定义材质外观的对象，它包含多个属性来定义材质，这些属性是：

Color：线条的颜色，用16进制来表示，默认的颜色是白色。

Linewidth：线条的宽度，默认时候1个单位宽度。

Linecap：线条两端的外观，默认是圆角端点，当线条较粗的时候才看得出效果，如果线条很细，那么你几乎看不出效果了。

Linejoin：两个线条的连接点处的外观，默认是“round”，表示圆角。

VertexColors：定义线条材质是否使用顶点颜色，这是一个boolean值。意思是，线条各部分的颜色会根据顶点的颜色来进行插值。（如果关于插值不是很明白，可以QQ问我，QQ在前言中你一定能够找到，嘿嘿，虽然没有明确写出）。

Fog：定义材质的颜色是否受全局雾效的影响。

好了，介绍完这些参数，你可以试一试了，在课后，我们会展示不同同学的杰出作品。下面，接着上面的讲，我们这里使用了顶点颜色vertexColors: THREE.VertexColors，就是线条的颜色会根据顶点来计算。

var material = new THREE.LineBasicMaterial( { vertexColors: THREE.VertexColors } );
```

改变视角，让画面动起来

```js
function animation () {
  // renderer.clear()
  camera.position.x = camera.position.x + 1
  renderer.render(scene, camera)
  requestAnimationFrame(animation)
}
```

```js
/*** 场景（scene） ***/
var scene = new THREE.Scene(); // 创建场景
scene.add(x);                  // 插入场景

/*** 相机（camera） ***/
// 正交投影相机
var camera = new THREE.OrthographicCamera(left, right, top, bottom, near, far);
// 透视头像相机
var camera = new THREE.PerspectiveCamera(fov, aspect, near, far); // fov：人眼夹角，aspect：长宽比

/*** 渲染器（renderer） ***/
var renderer = new THREE.WebGLRenderer(options);
// options {} 可选。参数：
// canvas：element <canvas></canvas>
renderer.setSize(长, 宽);
element.appendChild(renderer.domElement); // 插入节点
renderer.setClearColor(color, opacity);   // 设置清除后的颜色 16进制
renderer.clear();                         // 清除面板
renderer.render(scene, camera);           // 渲染

/*** 光照(light) ***/
new THREE.AmbientLight(颜色);                          // 环境光
new THREE.PointLight(颜色, 强度, 距离);                // 点光源
new THREE.DirectionalLight(颜色, 亮度);                // 平行光
new THREE.SpotLight(颜色, 强度, 距离, 夹角, 衰减指数); // 聚光灯

/*** 几何形状 ***/
new THREE.CubeGeometry(长, 宽, 高, 长的分割, 宽的分割, 高的分割);                           // 立方体
new THREE.PlanGeometry(长,宽, 长的分割, 宽的分割);                                          // 平面
new THREE.SphereGeometry(半径, 经度切片, 纬度分割, 经度分割, 经度跨过, 纬度开始, 纬度跨过); // 球体
new THREE.CircleGeometry(半径, 切片数, 开始, 跨过角度);                                     // 圆形
new THREE.CylinderGeometry(顶部面积, 底部面积, 高, 圆分割, 高分割, 是否没有顶面和底面);     // 圆台
new THREE.TetrahedronGeometry(半径, 细节);  // 正四边形
new THREE.OctahedronGeometry(半径, 细节);   // 正八边形
new THREE.IconsahedronGeometry(半径, 细节); // 正十二边形
new THREE.TorusGeometry(半径, 管道半径, 纬度分割, 经度分割, 圆环面的弧度); // 圆环面
// 自定义形状
var geometry = new THREE.Geometry();
geometry.vertices.push(new THREE.Vectory3(x, y, z)); // 点，其中x、y、z为坐标
geometry.faces.push(new THREE.Faces3(x, y, z));      // 面，其中x、y、z为点的数组的索引，三点确定一个面

/*** 材质 ***/
new THREE.MeshBasicMaterial(options); // 基本材质
// options {} 可选。参数：
//   visible：是否可见
//     color：颜色
// wireframe: 是否渲染线而非面
//      side：THREE.FrontSide 正面，THREE.BackSide 反面，THREE.DoubleSide 双面
//       map: 贴图
new THREE.MeshLambertMaterial(options); // Lambert材质，适合光照
//  ambient：反射能力
// emissive：自发光颜色
new THREE.MeshPhongMaterial();  // Phong材质，适合金属和镜面
//  specular：光罩颜色
// shininess：光斑大小（值越大，光斑越小）
new THREE.MeshNormalMaterial(); // 方向材质
/* 贴图 */
var texture = THREE.ImageUtils.loadTexture(url, {}, function(){}); // 载入单个贴图（建议贴图的长宽为256的倍数）
new THREE.MeshFaceMaterial() // 设置不同面的贴图，参数为单个贴图的数组
texture.wrapS texture.wrapT = THREE.RepeatWrapping // 贴图的重复方式
texture.repeat.set(x, y)     // 重复次数
new THREE.texture(canvas)    // 将canvas作为贴图

/*** 将模型和贴图结合 ***/
var mesh = new THREE.Mesh(形状, 材质);
mesh.position // 位置 mesh.position.x（y、z） 或 mesh.position.set(x, y, z)
mesh.scale    // 缩放
mesh.rotation // 旋转

/*** 监视FPS ***/
var stats = new Stats();
stats.domElement // 节点
stats.begin()    // 开始
stats.end()      // 结束
```