<!-- index -->
<template>
	<div ref="container" id="container"></div>
</template>

<script>
import * as THREE from 'three'
import { STLLoader } from 'three/examples/jsm/loaders/STLLoader'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
import { OBJLoader, MTLLoader } from 'three-obj-mtl-loader'
// 在vue里 非响应式变量可声明data之外,提高性能呢

import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { TWEEN } from 'three/examples/jsm/libs/tween.module.min.js'

import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer.js'
import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass.js'
import { OutlinePass } from 'three/examples/jsm/postprocessing/OutlinePass.js'
import { ShaderPass } from 'three/examples/jsm/postprocessing/ShaderPass.js'
import { FXAAShader } from 'three/examples/jsm/shaders/FXAAShader.js'

import {
	CSS3DRenderer,
	CSS3DObject,
} from 'three/examples/jsm/renderers/CSS3DRenderer.js'

// import 'three-onevent/onEvent.js'

const scene = new THREE.Scene()
const scene2 = new THREE.Scene()
const raycaster = new THREE.Raycaster()
const mouse = new THREE.Vector2()
export default {
	//import引入的组件需要注入到对象中才能使用
	components: {},
	data() {
		//这里存放数据
		return {
			camera: null,
			controls: null,
			renderer: null,
			mesh: null,

			posx: require('../assets/posx.jpg'),
			negx: require('../assets/negx.jpg'),
			posy: require('../assets/posy.jpg'),
			negy: require('../assets/negy.jpg'),
			posz: require('../assets/posz.jpg'),
			negz: require('../assets/negz.jpg'),

			publicPath: 'http://192.168.1.23:8080/',

			composer: null,
			outlinePass: null,
			renderPass: null,
			render3D: null,

			css3DObject: null,
		}
	},
	//监听属性 类似于data概念
	computed: {},
	//监控data中的数据变化
	watch: {},
	//方法集合
	methods: {
		init3dScene() {
			// 初始化场景渲染器
			this.initRender()
			// 初始化天空盒
			this.initScene()
			// 初始化相机与相机控制
			this.initCamera()
			// 加载灯光
			this.initLight()
			// 视角控制
			this.animateVis()

			window.addEventListener('click', this.onMouseClick, false)
			// 启动渲染
			this.startAnimate()
			// 加载模型
			this.initMainScene()
		},

		initRender() {
			// 把你的渲染器渲染的场景挂载到哪一个dom上?
			let container = document.getElementById('container')
			// 创建渲染器 全局的renderer
			// webGLRenderer 是最常用的渲染器 其他的还有基于css3D的css3DRenderer ,和基于canvas的canvasRenderer
			this.renderer = new THREE.WebGLRenderer({
				antialias: true, // 抗锯齿
				precision: 'highp', // 着色精度选择
				logarithmicDepthBuffer: true, // 消除模型交错闪烁
				preserveDrawingBuffer: true, // 是否可以提取canvas 绘图的缓冲
				shadowMap: true, // 开启阴影渲染 大量计算
				alpha: true, // 画布是否透明
			})

			// 设置场景显示区背景色
			this.renderer.setClearColor(0x000000, 0)
			// 场景显示区尺寸 (铺满)
			this.renderer.setSize(window.innerWidth, window.innerHeight)
			// 设置像素比 如果移动端掉帧 去掉这个试试
			this.renderer.setPixelRatio(window.devicePixelRatio)
			// 可以如此设置样式 例如确保cssRender与render不遮盖
			this.renderer.domElement.style.position = 'absolute'
			this.renderer.domElement.style.top = 0
			this.renderer.domElement.style.zIndex = '1'

			//新开一个3D渲染器
			this.render3D = new CSS3DRenderer()
			//设置渲染器大小
			this.render3D.setSize(container.clientWidth, container.clientHeight)
			console.log(this.render3D)

			//需要设置位置
			this.render3D.domElement.style.position = 'absolute'
			this.render3D.domElement.style.top = 0
			this.render3D.domElement.style.zIndex = '99'
			// 挂载dom
			container.appendChild(this.render3D.domElement)
			container.appendChild(this.renderer.domElement)
		},
		initScene() {
			// 当你需要去更新你场景中对象矩阵的时候，会涉及到计算，如果只是静态对象，并且操作不频繁，你可以关闭matrixAutoUpdate参数，手动去更新矩阵
			scene.matrixAutoUpdate = false
			// 辅助坐标系
			// var axes = new THREE.AxisHelper(20)
			// scene.add(axes)
			// 设置背景
			// 添加天空盒第一种方式
			/**  我从官网扒的星空天空盒
      skyBox: [
        "skybox/dark-s_px.jpg",
        "skybox/dark-s_nx.jpg",
        "skybox/dark-s_py.jpg",
        "skybox/dark-s_ny.jpg",
        "skybox/dark-s_pz.jpg",
        "skybox/dark-s_nz.jpg"
      ],*/
			scene.background = new THREE.CubeTextureLoader().load([
				this.posx,
				this.negx,
				this.posy,
				this.negy,
				this.posz,
				this.negz,
			])
		},
		initCamera() {
			// 透视相机 与此对应的 是平行视角相机 而 我们看世界就是远小近大的透视相机
			// 具体参数详解搜一下吧 很简单
			this.camera = new THREE.PerspectiveCamera(
				60, // 视界 大部分是 30-90 比如游戏就可以调节视野大小
				window.innerWidth / window.innerHeight, // 投影的宽高比
				1, // 我的理解 : 距离相机多近就不渲染了?
				100000 // 距离相机多远就不渲染了?
			)
			// 相机位置
			this.camera.position.set(0, 100, 250) //设置相机位置

			//相机以Y轴方向为上方（当值为1时即为上） 限制在y轴只能在正方向,比如加了地面元素之后,别跑到地下穿模了
			this.camera.up.x = 0
			this.camera.up.y = 1
			this.camera.up.z = 0
			//相机看向哪个坐标 (原点)
			this.camera.lookAt({
				x: 0,
				y: 0,
				z: 0,
			})

			//加载相机的鼠标操作 左键上下左右拖动 右键平移 滑轮缩放
			// 引入 import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"; // 相机控制
			new OrbitControls(this.camera, this.renderer.domElement)
			new OrbitControls(this.camera, this.render3D.domElement)
		},
		initLight() {
			//环境光 意思就是,为什么明明我在屋子里背光,按理说一片黑暗才对,但是还有光呢,环境反射的光
			let ambientLight = new THREE.AmbientLight(0xf1e2e2, 1.25)
			// 加入世界里去 不加入不会渲染哦
			scene.add(ambientLight)
			//点光源 白色灯光，亮度0.6
			let pointLight = new THREE.PointLight(0xbfbfbf, 0.6)
			//设置点光源位置，改变光源的位置
			pointLight.position.set(0, 40, 80)
			// scene.add(pointLight)
			// 还有聚光灯和平行光  用法同上
		},
		animateVis() {},
		startAnimate: function () {
			TWEEN.update()
			requestAnimationFrame(this.startAnimate) // 递归调用
			this.renderer.render(scene, this.camera)
			this.render3D.render(scene2, this.camera)
			if (this.composer) {
				this.composer.render()
			}
		},

		absPos(myMesh) {
			myMesh.geometry.computeBoundingBox()

			var boundingBox = myMesh.geometry.boundingBox

			var position = new THREE.Vector3()
			position.subVectors(boundingBox.max, boundingBox.min)
			position.multiplyScalar(0.5)
			position.add(boundingBox.min)

			position.applyMatrix4(myMesh.matrixWorld)
			this.initTooltip(position.x, position.y, position.z, myMesh.name)
		},

		initMainScene() {
			let manager = new THREE.LoadingManager()

			new MTLLoader(manager).load(
				`${this.publicPath}/static/工厂.mtl`,
				(materials) => {
					materials.preload()

					new OBJLoader(manager)
						.setMaterials(materials)
						.load(`${this.publicPath}/static/工厂.obj`, (obj) => {
							console.log(obj)

							// obj.traverse((child) => {
							// 	if (child instanceof THREE.Mesh) {
							// 		this.absPos(child) // call our function
							// 	}
							// })

							obj.scale.set(1, 1, 1)
							scene.add(obj)
						})
				}
			)

			// new GLTFLoader().load(`${this.publicPath}/static/工厂.gltf`,(gltf)=>{
			// 	console.log(gltf)
			// 	scene.add(gltf.scene)
			// })
		},

		onMouseClick(event) {
			event.preventDefault()
			mouse.x =
				(event.clientX / this.renderer.domElement.clientWidth) * 2 - 1
			mouse.y =
				-(event.clientY / this.renderer.domElement.clientHeight) * 2 + 1
			raycaster.setFromCamera(mouse, this.camera)

			var intersects = raycaster.intersectObjects(scene.children)
			if (intersects.length > 0) {
				this.absPos(intersects[0].object) //先定个位置，然后渲染信息提示
				this.outlineObj(intersects[0].object) //渲染高亮
			} else {
				if (this.composer && this.outlinePass) {
					this.composer.removePass(this.outlinePass)
					this.removeTooltip()
					this.composer = null
					this.outlinePass = null
				}
			}
		},

		//物体高亮
		outlineObj(selectedObjects) {
			// 创建一个EffectComposer（效果组合器）对象，然后在该对象上添加后期处理通道。
			this.composer = new EffectComposer(this.renderer)
			// 新建一个场景通道  为了覆盖到原理来的场景上
			this.renderPass = new RenderPass(scene, this.camera)
			this.composer.addPass(this.renderPass)
			// 物体边缘发光通道
			this.outlinePass = new OutlinePass(
				new THREE.Vector2(window.innerWidth, window.innerHeight),
				scene,
				this.camera
			)
			this.composer.addPass(this.outlinePass) // 加入高光特效

			this.outlinePass.pulsePeriod = 2 //数值越大，律动越慢
			this.outlinePass.visibleEdgeColor.set(0x00ff00) // 高光颜色
			this.outlinePass.hiddenEdgeColor.set(0x000000) // 阴影颜色
			this.outlinePass.usePatternTexture = false // 使用纹理覆盖？
			this.outlinePass.edgeStrength = 10 // 高光边缘强度
			this.outlinePass.edgeGlow = 1 // 边缘微光强度
			this.outlinePass.edgeThickness = 1 // 高光厚度

			this.outlinePass.selectedObjects = [selectedObjects] // 需要高光的obj
		},

		//生成CSS对象
		createdCSS3DLabel(name) {
			//增加一个CSS3D对象
			let element = document.createElement('div')
			element.className = 'infoBox'
			let lableTitle = document.createElement('div')
			lableTitle.textContent = name
			lableTitle.style.color = 'red'
			lableTitle.style['font-size'] = '12px'
			element.appendChild(lableTitle)

			return element
		},

		initTooltip(x, y, z, name) {
			this.removeTooltip()
			//生成CSSDOM对象
			let element = this.createdCSS3DLabel(name)
			//把生成的CSSDOM对象处理成three的节点对象
			let css3DObject = new CSS3DObject(element)
			//设置CSS3DObject对象
			css3DObject.position.x = x
			css3DObject.position.y = y + 20
			css3DObject.position.z = z
			//在第二个场景中添加这个对象
			scene2.add(css3DObject)
		},

		removeTooltip() {
			if (scene2.children.length > 0) {
				scene2.children.map((item) => {
					scene2.remove(item)
				})
			}
		},
	},
	//生命周期 - 创建完成（可以访问当前this实例）
	created() {},
	//生命周期 - 挂载完成（可以访问DOM元素）
	mounted() {
		this.init3dScene()
	},
	destroyed() {}, //生命周期 - 销毁完成
	activated() {}, //如果页面有keep-alive缓存功能，这个函数会触发
}
</script>
<style scoped>
#container {
	width: 100%;
	height: 100vh;
	margin: 0;
	padding: 0;
	overflow: hidden;
}

.infoBox {
	border: 1px solid pink;
}
</style>