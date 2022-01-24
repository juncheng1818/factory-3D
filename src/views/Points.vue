<!-- Points -->
<template>
	<div class="">ponits</div>
</template>

<script>
import * as THREE from 'three'
var scene = null
var renderer = null
var camera = null
export default {
	//import引入的组件需要注入到对象中才能使用
	components: {},
	data() {
		//这里存放数据
		return {
			geo: null,
		}
	},
	//监听属性 类似于data概念
	computed: {},
	//监控data中的数据变化
	watch: {},
	//方法集合
	methods: {
		init() {
			scene = new THREE.Scene()
			scene.fog = new THREE.Fog(0x05050c, 10, 60)
			scene.add(new THREE.GridHelper(2000, 1)) // 添加网

			camera = new THREE.PerspectiveCamera(
				45,
				window.innerWidth / window.innerHeight,
				5,
				100
			)
			camera.position.set(10, -10, -40)
			scene.add(camera)

			let ambientLight = new THREE.AmbientLight(0x000000, 0.4)
			scene.add(ambientLight)
			let pointLight = new THREE.PointLight(0xe42107)
			pointLight.castShadow = true
			pointLight.position.set(-10, -5, -10)
			pointLight.distance = 20
			scene.add(pointLight)

			this.geo = new THREE.Geometry() // 创建一个 Geometry 来存储顶点信息
			this.geo.center()
			this.geo.normalize()
			const vertices = json.vertices // json 是从 obj 模型导出的数据，这里获取顶点信息
			for (let i = 0; i < vertices.length / 3; i++) {
				const particle = new THREE.Vector3(
					vertices[i * 3],
					vertices[i * 3 + 1],
					vertices[i * 3 + 2]
				)
				this.geo.vertices.push(particle)
			}
			this.points = new THREE.Points(this.geo, this.particleMaterial) // 创建粒子
			this.scene.add(this.points)
		},
	},
	//生命周期 - 创建完成（可以访问当前this实例）
	created() {},
	//生命周期 - 挂载完成（可以访问DOM元素）
	mounted() {
		this.init()
	},
	destroyed() {}, //生命周期 - 销毁完成
	activated() {}, //如果页面有keep-alive缓存功能，这个函数会触发
}
</script>
<style scoped>
</style>