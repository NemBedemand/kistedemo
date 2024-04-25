<template>
  <div id="container" ref="container"></div>
  <div class="absolute top-4 left-4 z-20 shadow px-4 py-3 rounded-lg text-sm bg-white">
    <div class="flex justify-between">
      <label class="min-w-[12rem]">Top</label>
      <input type="color" v-model="body7" @input="updateColor($event, 'Group40')" />
      <input type="file" @change="updateMaterialFromFile($event, 'Group40')" />
    </div>
    <div class="flex justify-between">
      <label class="min-w-[12rem]">Venstre topside</label>
      <input type="color" v-model="body3" @input="updateColor($event, 'Group20')" />
      <input type="file" @change="updateMaterialFromFile($event, 'Group20')" />
    </div>
    <div class="flex justify-between">
      <label class="min-w-[12rem]">Højre topside</label>
      <input type="color" v-model="body6" @input="updateColor($event, 'Group35')" />
      <input type="file" @change="updateMaterialFromFile($event, 'Group35')" />
    </div>
    <div class="flex justify-between">
      <label class="min-w-[12rem]">Bagende</label>
      <input type="color" v-model="body2" @input="updateColor($event, 'Group15')" />
      <input type="file" @change="updateMaterialFromFile($event, 'Group15')" />
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, onMounted, ref, watch } from 'vue'
import * as THREE from 'three'
// @ts-ignore
import { OBJLoader } from 'three/examples/jsm/loaders/OBJLoader'
// @ts-ignore
import { MTLLoader } from 'three/examples/jsm/loaders/MTLLoader'
// @ts-ignore
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

export default defineComponent({
  name: 'CoffinViewer',
  setup() {
    const color = ref('#f9f9f9')
    const body1 = ref('#ffffff')
    const body2 = ref('#ffffff')
    const body3 = ref('#ffffff')
    const body4 = ref('#ffffff')
    const body5 = ref('#ffffff')
    const body6 = ref('#ffffff')
    const body7 = ref('#ffffff')
    const file7 = ref()
    const body8 = ref('#ffffff')

    const container = ref<HTMLElement | null>(null)
    let scene: THREE.Scene
    let camera: THREE.PerspectiveCamera
    let renderer: THREE.WebGLRenderer
    let controls: OrbitControls
    let loadedObject: THREE.Object3D | null = null

    const initThree = () => {
      scene = new THREE.Scene()
      camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
      renderer = new THREE.WebGLRenderer()
      renderer.setSize(window.innerWidth, window.innerHeight)
      renderer.setClearColor(0xf4f4f4, 1)
      container.value!.appendChild(renderer.domElement)
      controls = new OrbitControls(camera, renderer.domElement)
      camera.position.set(0, 5, 10)
      controls.update()

      // Add Ambient Light with moderate intensity
      const ambientLight = new THREE.AmbientLight(0xffffff, 2) // Not too bright, just to fill the scene lightly
      scene.add(ambientLight)

      // Add Directional Light from above
      const directionalLight = new THREE.DirectionalLight(0xffffff, 1) // Reduced intensity
      directionalLight.position.set(0, 10, 0) // Position it above the object
      directionalLight.castShadow = true // Enable shadows

      // Soften the shadows
      directionalLight.shadow.mapSize.width = 1024 // Increase for higher quality shadows
      directionalLight.shadow.mapSize.height = 1024
      directionalLight.shadow.camera.near = 0.5
      directionalLight.shadow.camera.far = 50

      scene.add(directionalLight)

      const mtlLoader = new MTLLoader()
      mtlLoader.load('../src/assets/no-handles.mtl', (materials: any) => {
        materials.preload()
        const objLoader = new OBJLoader()
        objLoader.setMaterials(materials)

        objLoader.load('../src/assets/no-handles.obj', (object: any) => {
          let body1
          object.traverse(function (child: any) {
            if ((child as THREE.Mesh).isMesh) {
              child.material = new THREE.MeshStandardMaterial({ color: color.value })
            }
            if (child.name === 'Body2') {
              body1 = child
            }
          })

          object.scale.set(0.1, 0.1, 0.1)
          object.rotation.y = 90 * (Math.PI / 180)

          const box = new THREE.Box3().setFromObject(object)
          const center = box.getCenter(new THREE.Vector3())

          // Adjust camera position: Move it to the front of the coffin and slightly above
          camera.position.z = center.z + 20 // Move camera back along Z-axis to see the front
          camera.position.y = center.y + 12 // Lift camera up along Y-axis to view from above
          camera.position.x = center.x // Center camera along X-axis (adjust as needed)

          // Set controls target to the center of the object
          controls.target.set(center.x, center.y, center.z)

          // Add the object to the scene and adjust the camera/controls
          scene.add(object)
        })
      })

      const animate = () => {
        requestAnimationFrame(animate)
        controls.update()
        renderer.render(scene, camera)
      }

      animate()
    }

    const updateColor = (event: any, name: string) => {
      scene.traverse(function (child: any) {
        if (child.isMesh) {
          if (child.name === name) {
            child.material = new THREE.MeshStandardMaterial({ color: event.target.value })
          }
        }
      })
    }

    const updateMaterialFromFile = (event: Event, name: string) => {
      const input = event.target as HTMLInputElement
      if (input.files && input.files[0]) {
        const file = input.files[0]
        const reader = new FileReader()

        reader.onload = (e: any) => {
          const texture = new THREE.TextureLoader().load(e.target.result as string)
          scene.traverse((child: any) => {
            if (child.isMesh && child.name === name) {
              console.log(child)
              ;(child as THREE.Mesh).material = new THREE.MeshBasicMaterial({
                map: texture
              })
            }
          })
        }

        reader.readAsDataURL(file) // Read file as data URL for use as texture
      }
    }

    onMounted(initThree)
    // watch(color, updateColor)

    return {
      container,
      color,
      body1,
      body2,
      body3,
      body4,
      body5,
      body6,
      body7,
      file7,
      body8,
      updateColor,
      updateMaterialFromFile
    }
  }
})
</script>

<style>
#container {
  width: 100vw;
  height: 100vh;
  background-color: #f4f4f4;
}
</style>
