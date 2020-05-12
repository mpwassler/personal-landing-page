<script>
	import { onMount } from 'svelte'
	import {
		Scene,
		PerspectiveCamera,
		WebGLRenderer,
		PlaneBufferGeometry,
		MeshBasicMaterial,
		MeshPhongMaterial,
		PointLight,
		AmbientLight,
		FogExp2,
		Mesh,
		Raycaster,
		Vector2,
		Vector3
	} from 'three'

	import { SceneUtils } from 'three/examples/jsm/utils/SceneUtils.js'

	let container

	const makeScene = () => {
		var scene = new Scene()
		var camera = new PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 0.1, 1000000 )
		var renderer = new WebGLRenderer({
			alpha: true
		})
		scene.fog = new FogExp2( 0xffffff, 0.000009)
		camera.position.z = 10500
		camera.position.y = 25000
		renderer.setSize( window.innerWidth, window.innerHeight )
		container.appendChild( renderer.domElement )
		return { scene, renderer, camera }
	}

	const addPointLight = (scene, opts) => {
		var pointLight = new PointLight(opts.color, opts.intesity)
		pointLight.position.z = opts.position.z
		pointLight.position.x =  opts.position.x
		pointLight.position.y = opts.position.y
		scene.add(pointLight)
	}

	const addMesh = (scene, opts) => {
		var geometry = new PlaneBufferGeometry(
			opts.size,
			opts.size,
			opts.resolution,
			opts.resolution
		)
		var mesh = SceneUtils.createMultiMaterialObject(geometry, [
			new MeshPhongMaterial( { color: 0xced1d1, emissive: 0x000000, flatShading: true} ),
			new MeshBasicMaterial( { color: 0xe2e2e2, wireframe: true} )
		])
		mesh.rotateX( - Math.PI / 3.8 )
		mesh.position.set(opts.position.x, opts.position.y, opts.position.z)
		scene.add(mesh)
		return { mesh, geometry }
	}

	const addAbientLight = (scene, opts) => {
		var light = new AmbientLight( opts.color, opts.intesity )
		scene.add( light )
	}

	onMount(() => {
		var vertexHeight = 15000
		var count = 0

		const mouse = new Vector2()
		const raycaster = new Raycaster()

		let mousePoint3D = null

		const { scene, renderer, camera } = makeScene()

		window.addEventListener( 'resize', () => {
			camera.aspect = window.innerWidth / window.innerHeight
			camera.updateProjectionMatrix()
			renderer.setSize( window.innerWidth, window.innerHeight )
		}, false )



		addPointLight( scene, {
			position: { z : 14000, x : -1000, y : 0 },
			intesity: 1.2,
			color: 0xa8a8a8
		})

		addAbientLight(scene, {
			color: 0xffffff,
			intesity: 0.75
		})

		const { mesh, geometry } = addMesh( scene, {
			resolution: 50,
			size: 1245000 / 2,
			position: { z : 0, x : 0, y : -60000 }
		})

		document.querySelector('.content').addEventListener( 'mousemove', (event) => {
			console.log("ran")
			mouse.x = ( event.clientX / renderer.domElement.clientWidth ) * 2 - 1
			mouse.y = - ( event.clientY / renderer.domElement.clientHeight ) * 2 + 1
			raycaster.setFromCamera( mouse, camera )

			// See if the ray from the camera into the world hits one of our meshes
			var intersects = raycaster.intersectObject( mesh.children[0] )

			// Toggle rotation bool for meshes that we clicked
			// console.log(intersects)
			if ( intersects.length > 0 ) {
				mousePoint3D = new Vector2(intersects[0].point.x, intersects[0].point.y)
			}
		}, false )

		const vertices = geometry.attributes.position.array

		const toZvalues = new Float32Array(vertices.length / 3)

		for (var i = 0, j = 0; i < toZvalues.length; i++, j += 3) {
      vertices[j + 2] += Math.random() * vertexHeight - vertexHeight
      toZvalues[i] = vertices[j + 2]
		}

		(function animate() {
			requestAnimationFrame( animate )
			for (var i = 0, j = 0; i < toZvalues.length; i++, j += 3) {
				let x = vertices[j],
				    y = vertices[j + 1],
				    z = vertices[j + 2]
				var vertPoint = new Vector2(x,y)
				if (mousePoint3D && vertPoint) {
					let distance = mousePoint3D.distanceTo(vertPoint)
					let multiplier = distance * 0.00001
					vertices[j + 2] = Math.sin(( i + count * 0.00002)) * (toZvalues[i]* multiplier - (toZvalues[i]))
				} else {
	      	vertices[j + 2] = Math.sin(( i + count * 0.00002)) * (toZvalues[i] - (toZvalues[i]* 0.6))

				}
	      count += 0.4

	    }
	    mesh.children.forEach( subMesh => {
	    	subMesh.geometry.attributes.position.needsUpdate = true
	    })
			renderer.render( scene, camera )
		})();
	})
</script>
<style>
	h1, figure, p {
	}

	h1 {
		font-size: 2.8em;
		text-transform: uppercase;
		font-weight: 700;

	}

	figure {
		margin: 0 0 1em 0;
	}

	img {
		width: 100%;
		max-width: 400px;
		margin: 0 0 1em 0;
	}

	@media (min-width: 480px) {
		h1 {
			font-size: 4em;
		}
	}

	.page {
		position: relative;
	}
	.content {
		position: absolute;
		top:0;
		bottom: 0;
		right: 0;
		left: 0;
	}
	.title {
		font-family: 'bb-1';
		font-size: 3.5em;
		color: #020202;
		/*background: #000;*/
		color: #4c4c4c;
		padding: 2.5rem 3rem 0rem;
	}
	.copy {
	 padding: 0rem 3rem;
	 font-family: 'bb-7';
	 font-size: 1.25em;
	 color: #4c4c4c;
	}

	.links {
		position: absolute;
		bottom: 2rem;
		left: 3rem;
	}

	.link {
		font-size: 2.25rem;
		margin: 0 1.5rem 0 0;
		color: #4c4c4c;
	}

</style>

<svelte:head>
	<title>Mitchel Wassler</title>
</svelte:head>
<div class="page">
	<div bind:this={container} class="scene">

	</div>
	<div class="content">
		<h2 class="title">Hi. My name is Mitchel Wassler.</h2>
		<p class="copy">I am a software developer from Cincinnati Ohio. I specialize in UI design, <br>Interactive content, devops and cloud computing.</p>

		<div class="links">
			<a class="link" href="https://github.com/mpwassler">
				<i class="fab fa-github"></i>
			</a>

			<a class="link" href="https://www.linkedin.com/in/mitchel-wassler-818a23a6/">
				<i class="fab fa-linkedin"></i>
			</a>

			<a class="link" href="https://www.instagram.com/mitchelwassler/">
				<i class="fab fa-instagram"></i>
			</a>
		</div>

	</div>

</div>
