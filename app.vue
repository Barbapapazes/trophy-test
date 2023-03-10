<template>
  <div id="gemWrapper" ref="gemWrapper" :style="{ opacity: ready ? 1 : 0 }" class="transition duration-1000">
    <div ref="gemAnim" />
  </div>
</template>

<script setup>
const ready = ref()
const gemWrapper = ref(null)
const gemAnim = ref(null)

if (process.client) {
  async function loadGem() {
    const THREE = await import('three').then(m => m.default || m)
    const { OrbitControls } = await import('three/examples/jsm/controls/OrbitControls.js' /* webpackChunkName: "trophy" */).then(m => m.default || m)
    const { GLTFLoader } = await import('three/examples/jsm/loaders/GLTFLoader.js' /* webpackChunkName: "trophy" */).then(m => m.default || m)
    const { RGBELoader } = await import('three/examples/jsm/loaders/RGBELoader.js' /* webpackChunkName: "trophy" */).then(m => m.default || m)
    const { DRACOLoader } = await import('three/examples/jsm/loaders/DRACOLoader.js' /* webpackChunkName: "trophy" */).then(m => m.default || m)
    const { RoomEnvironment } = await import('three/examples/jsm/environments/RoomEnvironment.js' /* webpackChunkName: "trophy" */).then(m => m.default || m)

    // Loaders
    const gltfLoader = new GLTFLoader()
    const dracoLoader = new DRACOLoader()
    dracoLoader.setDecoderPath('https://raw.githubusercontent.com/mrdoob/three.js/dev/examples/jsm/libs/draco/')
    gltfLoader.setDRACOLoader(dracoLoader)

    // Renderer
    const renderer = new THREE.WebGLRenderer({ alpha: true, antialias: true })
    renderer.outputEncoding = THREE.sRGBEncoding
    renderer.toneMapping = THREE.ACESFilmicToneMapping
    renderer.toneMappingExposure = 1
    if (window.matchMedia('(min-width: 640px)').matches) {
      renderer.setSize(475, 475)
    } else if (window.matchMedia('(min-width: 400px)').matches) {
      renderer.setSize(240, 240)
    } else {
      renderer.setSize(180, 180)
    }

    // Scene
    const scene = new THREE.Scene()

    const pmremGenerator = new THREE.PMREMGenerator(renderer)
    scene.environment = pmremGenerator.fromScene(new RoomEnvironment(), 0.04).texture

    // Gem
    let gem

    // const hdrDark = new RGBELoader().load(
    //   '/assets/home/environment_D.hdr',
    //   () => {
    //     hdrDark.mapping = THREE.EquirectangularReflectionMapping
    //   }
    // )

    // const hdrLight = new RGBELoader().load(
    //   '/assets/home/environment_L.hdr',
    //   () => {
    //     hdrLight.mapping = THREE.EquirectangularReflectionMapping
    //   }
    // )

    // mtlLoader.load("/assets/home/trophy.mtl", function (materials) {
    //   materials.preload()
    //   objLoader.setMaterials(materials)
    //   objLoader.load("/assets/home/trophy.obj", function (object) {
    //     gem = object
    //     gem.scale.set(0.5, 0.5, 0.5)
    //     gem.position.set(0, 0, 0)
    //     gem.rotation.z = -0.2
    //     scene.add(gem)

    //     ready.value = true
    //   })
    // })

    // const gemMaterial = new THREE.MeshPhysicalMaterial({})

    gltfLoader.load('/assets/home/scene.glb', function (gltf) {
      gem = gltf.scene.children[0]
      gem.traverse((o) => {
        // if (o.isMesh) { o.material = gemMaterial }
        gem.scale.set(1, 1, 1)
        gem.position.set(0, -5, 0)
        gem.rotation.z = 0
        scene.add(gem)

        ready.value = true
      })
    })

    // Renderer
    gemAnim.value.appendChild(renderer.domElement)
    renderer.setPixelRatio(window.devicePixelRatio)
    function onWindowResize() {
      if (window.matchMedia('(min-width: 640px)').matches) {
        renderer.setSize(475, 475)
      } else if (window.matchMedia('(min-width: 400px)').matches) {
        renderer.setSize(240, 240)
      } else {
        renderer.setSize(180, 180)
      }
    }

    // useEventListener(window, 'resize', () => {
    //   onWindowResize()
    // })

    // Transition on load
    gemWrapper.value.style.opacity = 0
    setTimeout(() => {
      gemWrapper.value.style.opacity = 1
    }, 1000)

    // Camera
    const camera = new THREE.PerspectiveCamera(50, 1, 0.1, 2000)
    camera.aspect = 1
    camera.position.z = 14

    // Controls
    const controls = new OrbitControls(camera, renderer.domElement)
    controls.enableDamping = true
    controls.dampingFactor = 0.05
    controls.enableZoom = false
    controls.enablePan = false
    controls.maxPolarAngle = Math.PI * 0.5
    controls.minPolarAngle = Math.PI * 0.5

    controls.update()

    function animate() {
      // gemMaterial.color = new THREE.Color(0x000000)
      // gemMaterial.metalness = 1
      // gemMaterial.roughness = 0
      // gemMaterial.transmission = 0.82
      // gemMaterial.thickness = 5.1
      // gemMaterial.envMap = hdrLight
      // gemMaterial.envMapIntensity = 0.4
      // gemMaterial.specularIntensity = 0.61
      // gemMaterial.specularColor = 0x000000
      // gemMaterial.sheen = 0.78
      // gemMaterial.clearcoat = 0.16
      // gemMaterial.flatShading = true

      requestAnimationFrame(animate)
      if (gem) {
        gem.rotation.y -= 0.01
      }
      controls.update()
      renderer.render(scene, camera)
    }

    animate()
  }

  onMounted(() => requestIdleCallback(() => {
    const observer = new IntersectionObserver((entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          loadGem()
          observer.unobserve(entry.target)
        }
      })
    }, { threshold: 0.5 })
    observer.observe(gemAnim.value)
  }))
}
</script>
