# ThreeJS

# 结合Vue使用

1.创建项目

```powershell
npm init vite@latest
```

2.安装threejs

```powershell
npm install three
```



# 基本用法

```js
// 导入three
import * as THREE from "three";
// 导入轨道控制器
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
// 导入lil.gui
import { GUI } from "three/examples/jsm/libs/lil-gui.module.min.js";

// 创建场景
const scene = new THREE.Scene();
// 创建相机
const camera = new THREE.PerspectiveCamera(
  45, // 视角
  window.innerWidth / window.innerHeight, // 宽高比
  0.1, //近平面
  1000, //远平面
)

// 设置相机位置
camera.position.z = 5;
camera.position.y = 2;
camera.position.x = 2;
camera.lookAt(0, 0, 0);


// 创建渲染器
const renderer = new THREE.WebGLRenderer();
// 设置渲染器宽高
renderer.setSize(window.innerWidth, window.innerHeight);
// 设置渲染器背景色
renderer.setClearColor(0x000000);
// 添加到dom中
document.body.appendChild(renderer.domElement);

// 创建几何体
const geometry = new THREE.BoxGeometry(1, 1, 1);
// 创建材质meterial
const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
// 创建材质parentMaterial
const parentMaterial = new THREE.MeshBasicMaterial({ color: 0xff0000 });
// 父元素材质为线框材质
parentMaterial.wireframe = true;

// 创建两个网格
let parentCube = new THREE.Mesh(geometry, parentMaterial);
const cube = new THREE.Mesh(geometry, material);
// 将cube添加到parentCube的子元素
parentCube.add(cube);

// 设置parentCube的位置和旋转
parentCube.position.set(-2, 0, 0);
// parentCube绕着x轴旋转15度
parentCube.rotation.x = Math.PI / 12;


// 设置cube的位置和旋转
cube.position.set(2, 0, 0);
// 设置方块的放大，基于父元素
cube.scale.set(2, 2, 2);
// cube绕着x轴旋转45度
cube.rotation.x = Math.PI / 4;


// 把parentCube添加到场景
scene.add(parentCube);


// 添加世界坐标辅助器
const axesHelper = new THREE.AxesHelper(5);
scene.add(axesHelper);


// 添加轨道控制器
const controls = new OrbitControls(camera, renderer.domElement);
// 设置阻尼
controls.enableDamping = true;
// 设置阻尼系数
controls.dampingFactor = 0.05;
// 自动旋转
// controls.autoRotate = true;
// 旋转速度
// controls.autoRotateSpeed = 160;


// 渲染函数
function animate() {
  controls.update();
  requestAnimationFrame(animate);
  // cube.rotation.x += 0.01;
  // cube.rotation.y += 0.01;
  renderer.render(scene, camera);
}
animate();
// 到此，效果就有了


// 监听窗口变化
window.addEventListener("resize", () => {
  // 重置相机宽高比
  camera.aspect = window.innerWidth / window.innerHeight;
  // 更新相机投影矩阵
  camera.updateProjectionMatrix();
  // 重置渲染器宽高比
  renderer.setSize(window.innerWidth, window.innerHeight);
})

let evenObj = {
  // 全屏
  Fullscreen: function () {
    document.body.requestFullscreen();
  },
  exitFullscreen: function () {
    document.exitFullscreen();
  }
}

// 创建GUI
const gui = new GUI();
// 添加按钮
gui.add(evenObj, "Fullscreen");
gui.add(evenObj, "exitFullscreen");
// 控制立方体的位置
let x = 0
let folder = gui.addFolder("立方体位置");
folder.add(cube.position, 'x').min(-5).max(5).step(1).name("x轴位置").onChange((val) => {
  console.log("x轴位置", val);
});
folder.add(cube.position, "y").min(-5).max(5).step(1).name("y轴位置").onFinishChange((val) => {
  console.log("y轴位置", val);
})
folder.add(cube.position, "z").min(-5).max(5).step(1).name("z轴位置");
gui.add(parentMaterial, "wireframe").name("父元素线框");

// 修改材质的颜色
let colorObj = {
  color: "#ff0000"
}
gui.addColor(colorObj, "color").name("立方体颜色").onChange((val) => {
  cube.material.color.set(val);
})
```



# 几何体

由顶点构成，三个为一个顶点，需要使用`索引`，在共用同一个顶点

```js
// 创建几何体
const geometry = new THREE.BufferGeometry();
// 创建顶点数据，顶点是有序的，每三个为一个顶点，逆时针为正面，后面看不见，顺时针反之
// 但是后果是，6个节点形成一个面。因为 没有使用索引
// const vertices = new Float32Array([
//   -1, -1, 1,
//   1, -1, 1,
//   1, 1, 1,
//   1, 1, 1,
//   -1, 1, 1,
//   -1, -1, 1
// ]);
// 使用索引的
const vertices = new Float32Array([
  -1, -1, 1,  // 0
  1, -1, 1,   // 1
  1, 1, 1,  // 2
  -1, 1, 1, // 3
  1, -1, -1, // 4
  1, 1, -1, // 5
  -1, -1, -1,//6
  -1, 1, -1,//7
]);

// 创建顶点属性
geometry.setAttribute("position", new THREE.BufferAttribute(vertices, 3));
// 创建索引
const indices = new Uint16Array([
  0, 1, 2,//前
  2, 3, 0, //前
  1, 4, 2,//右
  4, 5, 2,//右
  4, 6, 5,//后
  6, 7, 5,//后,
  6, 0, 7,//左
  0, 3, 7,//左
  3, 2, 7,//上
  2, 5, 7,//上
  6, 4, 0,//下
  4, 1, 0,//下
]);
// 设置索引，达到共用顶点的目的
geometry.setIndex(new THREE.BufferAttribute(indices, 1));
console.log(geometry);
// 创建材质
const material = new THREE.MeshBasicMaterial({ color: 0x00ff00, side: THREE.DoubleSide, wireframe: false });
const plane = new THREE.Mesh(geometry, material);
scene.add(plane);
```





# 材质Material

## 基础网格材质

```js
// 创建纹理加载器
const textureLoader = new THREE.TextureLoader();
// 加载纹理
const texture = textureLoader.load("./zwtx.png");
// 使用SRGB颜色
texture.colorSpace = THREE.SRGBColorSpace;
// 加载ao贴图
const aoTexture = textureLoader.load("./zwtx.png");
// 创建透明度提贴图
const alphaTexture = textureLoader.load("./zwtx.png");
// 光照贴图
const lightTexture = textureLoader.load("./bilibili.png");



// rgbLoader
const rgbLoader = new RGBELoader();
// 加载hdr贴图
rgbLoader.load("../../public/room.hdr", (texture) => {
    // 设置球形贴图
    texture.mapping = THREE.EquirectangularReflectionMapping;
    // 背景图
    scene.background = texture
    // 设置环境贴图
    scene.environment = texture
    // 设置plan的环境贴图
    planeMaterial.envMap = texture;

})


// 创建材质
let planeMaterial = new THREE.MeshBasicMaterial({
    // 材质样色
    color: 0xffffff,
});
// 允许透明，png图片会透明
planeMaterial.transparent = true;
// 贴图纹理
planeMaterial.map = texture;
// ao贴图，环境光遮蔽贴图，模拟场景真实感
planeMaterial.alphaMap = texture;
// 设置透明贴图
planeMaterial.alphaMap = alphaTexture;
// 设置光照贴图
planeMaterial.lightMap = lightTexture;
```





## 雾

```js
// 创建长方体
const boxGeometry = new THREE.BoxGeometry(1, 1, 100)
const boxMaterial = new THREE.MeshBasicMaterial({
    color: 0x00ff00
});
const box = new THREE.Mesh(boxGeometry, boxMaterial);
scene.add(box);

// 创建场景fog
// scene.fog = new THREE.Fog(0x999999, 0.1, 50)
// 创建场景指数fog
scene.fog = new THREE.FogExp2(0x999999, 0.1)
scene.background = new THREE.Color(0x999999);
```





# GLTF加载器

```js
// 导入gltf加载器
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";


// 实例化gltf加载器，路径从public 开始
const gltfLoader = new GLTFLoader();
scene.background = new THREE.Color(0xffffff);
gltfLoader.load("./tifi.glb", (gltf) => {
    scene.add(gltf.scene)
})
```

## DracoLoader加载器

压缩和解压缩三维模型。文件后缀名为.drc





# 光线投射，3d场景交互

创建三个球，鼠标点击变红，再点重置

```js
// 创建一个球
const sphere1 = new THREE.Mesh(
    new THREE.SphereGeometry(1, 32, 32),
    new THREE.MeshBasicMaterial({
        color: 0x0000ff
    })
)
sphere1.position.x = -4;
scene.add(sphere1);

const sphere2 = new THREE.Mesh(
    new THREE.SphereGeometry(1, 32, 32),
    new THREE.MeshBasicMaterial({
        color: 0x00ff00
    })
)
// sphere2.position.x = 4;
scene.add(sphere2);

const sphere3 = new THREE.Mesh(
    new THREE.SphereGeometry(1, 32, 32),
    new THREE.MeshBasicMaterial({
        color: 0xff00ff
    })
)
sphere3.position.x = 4;
scene.add(sphere3);

// 创建射线
const raycaster = new THREE.Raycaster();
// 创建鼠标向量
const mouse = new THREE.Vector2();
window.addEventListener('click', (event) => {
    // 设置鼠标向量的x,y值
    mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
    mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

    //  通过鼠标向量和相机矩阵更新射线
    raycaster.setFromCamera(mouse, camera);

    // 获取射线和球体相交的数组
    const intersects = raycaster.intersectObjects([sphere1, sphere2, sphere3]);

    // 如果有多个相交的物体，就取第一个
    if (intersects.length > 0) {

        // 如果没有选中，则选中，并且材质颜色变为红色，否则恢复原来的颜色
        if (intersects[0].object.isSelect) {
            intersects[0].object.material.color.set(
                intersects[0].object.originalColor
            );
            intersects[0].object.isSelect = false;
            return;
        }

        // 将相交的物体颜色设置为红色
        intersects[0].object.isSelect = true
        intersects[0].object.originalColor = intersects[0].object.material.color.getHex();
        intersects[0].object.material.color.set(0xff0000);
    }

})
```





# 补间动画TWEEN

```js
const tween = new TWEEN.Tween(sphere1.position)
// 循环，无限次
// tween.repeat(Infinity)
// 来回往复
// tween.yoyo(true)
// 延迟
// tween.delay(3000)
tween.to({ x: 10 }, 1000)
// 设置缓动动画
tween.easing(TWEEN.Easing.Quadratic.InOut)
// 启动补间动画
// tween.start()

// sphere1是一个模型
let tween2 = new TWEEN.Tween(sphere1.position)
// tween2.repeat(Infinity)
// tween2.yoyo(true)
// tween2.easing(TWEEN.Easing.Quadratic.InOut)
tween2.to({ y: -10 }, 1000);
// tween2.start()
tween.chain(tween2)
tween2.chain(tween)
tween.start()
setTimeout(() => {
    tween.stop()
}, 4000);
// 其他回调函数
tween.onStart(() => {
    console.log("开始")
})
tween.onComplete(() => {
    console.log("完成")
})
tween.onUpdate(() => {
    console.log("更新")
})
tween.onStop(() => {
    console.log("停止")
})
```





# 阴影

```js
// 创建渲染器
const renderer = new THREE.WebGLRenderer();
.....
// 开启阴影贴图
renderer.shadowMap.enabled = true;


// 开启阴影
// 1、材质需要对光照有反应
// 2、渲染器开启阴影计算：renderer.shadowMap.enabled = true;
// 3、设置光照投射阴影：directionalLight.castShadow = true;
// 4、设置物体投射阴影：sphere.castShadow = true;
// 5、设置物体接收阴影：plane.receiveShadow = true;


// 光
// 环境光
const light = new THREE.AmbientLight(0xffffff, 0.5);
scene.add(light);
// 直线光源
const directionalLight = new THREE.DirectionalLight(0xffffff, 0.5);
directionalLight.position.set(10, 10, 10);
// 开启光照投射阴影
directionalLight.castShadow = true;
scene.add(directionalLight);

// 创建一个球
const sphere = new THREE.Mesh(
    new THREE.SphereGeometry(1, 32, 32),
    new THREE.MeshStandardMaterial({
        color: 0xffffff,
        metalness: 0.3,
        roughness: 0.4,
        envMapIntensity: 5,
    })
);
// 投射阴影
sphere.castShadow = true;
scene.add(sphere);


// 创建平面
const plane = new THREE.Mesh(
    new THREE.PlaneGeometry(10, 10),
    new THREE.MeshStandardMaterial({
        color: 0xffffff,
        metalness: 0.3,
        roughness: 0.4,
        envMapIntensity: 5,
    })
);
plane.position.y = -1;
plane.rotation.x = -Math.PI / 2;
// 接收阴影
plane.receiveShadow = true;
scene.add(plane);
```



## 阴影分辨率

```js
// 设置阴影贴图模糊度
directionalLight.shadow.radius = 20;
// 设置阴影贴图的分辨率
directionalLight.shadow.mapSize.set(2048, 2048);
```

## 相机属性

作用是，让正交相机只计算这一部分的光照和阴影，范围外的不计算

```js
// 设置平行光投射相机的属性
// 近端
directionalLight.shadow.camera.near = 0.1;
// 远端
directionalLight.shadow.camera.far = 100;
directionalLight.shadow.camera.left = -10;
directionalLight.shadow.camera.right = 10;
directionalLight.shadow.camera.top = 10;
directionalLight.shadow.camera.bottom = -10;
```



## 聚光灯

同正交平行光，但是会跟随目标几何体





# demo

1、 普通Vue3项目

2、 安装threejs和gsap



