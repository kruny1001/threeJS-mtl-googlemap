<template>
  <section>
    <div>
      <button style="z-index:100;  position: absolute; top: 40px;" @click="model3dRotate">Rotate 3D model {{crntModelRotation}}</button>
      <div style="width:100%; height:100%;"  id="map" ref="map"></div>
    </div>
  </section>
</template>

<script>
  import THREE from '~/plugins/mtl-loader'
  export default {
    data() {
      return{
        crntModelRotation: 0, 
        model3d:null,
        container: null, 
        stats: null,
        camera: null,
        scene: null,
        cssScene: null,
        renderer: null,
        cssRenderer: null,
        light: null,
        side: null,
        mouseX: 0, 
        mouseY: 0,
        windowHalfX : 0,
        windowHalfY : 0,
      }
    },
    ready: function () {
      // console.log( "here are my comp2 refs:", this.$refs.map );
      var vm = this;
      vm.windowHalfX = window.innerWidth / 2,
      vm.windowHalfY = window.innerHeight / 2
      // vm.container = document.getElementById('map');
      
    },
    mounted(){
      var vm = this;
      vm.$nextTick(function() {
        window.addEventListener( 'resize', vm.onWindowResize, false )
        document.addEventListener( 'mousemove', vm.onDocumentMouseMove, false )
      })
      vm.container = this.$refs.map
      vm.init()
      vm.animate()
    },
    methods: {
      model3dRotate() {
        this.model3d.rotation.y  -= .1
        this.crntModelRotation = this.model3d.rotation.y
      },
      renderCSS(){
        var vm = this;
        vm.cssRenderer.render(vm.cssScene, vm.camera);
      },
      addMap(){
        var vm = this;
        vm.create3dPage(
          5000, 5000,
          new THREE.Vector3(-1050, 0, 400),
          new THREE.Vector3(0, 45, 0),
          'https://www.google.com/maps/embed/v1/view?zoom=10&maptype=satellite&center=52.3702%2C4.8952&key=AIzaSyAVUZTKZz1e6hbEwZOT8CmWfoMhegHL7bs')
      },
      render() {
          var vm = this;
          vm.renderer.render( vm.scene, vm.camera );  
          vm.cssRenderer.render(vm.cssScene, vm.camera);
      },
      onDocumentMouseMove: function(event) {
        var vm = this;
      },
      onWindowResize: function(event){
        console.log("change window size", event)
        var vm = this;

        vm.windowHalfX = window.innerWidth / 2;
        vm.windowHalfY = window.innerHeight / 2;
        vm.camera.aspect = window.innerWidth / window.innerHeight;
        vm.camera.updateProjectionMatrix();
        vm.renderer.setSize( window.innerWidth, window.innerHeight );
        // console.log(event.clientX, vm.windowHalfX)
        // vm.mouseX = ( event.clientX - vm.windowHalfX ) / 2;
        // vm.mouseY = ( event.clientY - vm.windowHalfY ) / 2;
      },
      createPlane(w, h, position, rotation) {
          var material = new THREE.MeshBasicMaterial({color: 0x0002100,opacity: 0.0});
          var geometry = new THREE.PlaneGeometry(w, h);
          var mesh = new THREE.Mesh(new THREE.PlaneGeometry( 5000, 5000 ), material);
          mesh.rotation.set(-Math.PI/2, Math.PI/2000, Math.PI); 
          mesh.position.y = - 250;
          return mesh;
      },
      createCssObject(w, h, position, rotation, url) {
        url="https://www.google.com/maps/embed/v1/view?zoom=20&maptype=satellite&center=42.4563187%2C-71.3582864&key=AIzaSyAVUZTKZz1e6hbEwZOT8CmWfoMhegHL7bs"
        var vm = this
          var html = ['<div style="width:'+w+'px; height:'+h+'px;" v-html="iframe">',
          '<iframe src="' + url + '" width="' + w + '" height="' + h + '">','</iframe>','</div>'].join('\n');
          var div = document.createElement("div")
          div.width = w
          div.height = h
          var iframe = document.createElement("iframe")
          iframe.src = url
          iframe.width = w 
          iframe.height = h
          div.appendChild(iframe)
          var cssObject = new THREE.CSS3DObject(div);
          cssObject.rotation.set(-Math.PI/2, Math.PI/2000, Math.PI); 
          cssObject.position.y = - 250;
          return cssObject;
      },
      create3dPage(w, h, position, rotation, url) {
        var vm = this
        position=new THREE.Vector3(-1050, 0, 400)
        rotation=new THREE.Vector3(2000, 45, 0)
        url="https://www.google.com/maps/embed/v1/view?zoom=10&maptype=satellite&center=52.3702%2C4.8952&key=AIzaSyAVUZTKZz1e6hbEwZOT8CmWfoMhegHL7bs"
        var plane = vm.createPlane(5000, 5000,position,rotation);
        vm.scene.add(plane);
        var cssObject = vm.createCssObject(5000, 5000,position,rotation,url);
        vm.cssScene.add(cssObject);
      },
      createCssRenderer() {
        var vm = this;
        vm.cssRenderer = new THREE.CSS3DRenderer();
        vm.cssRenderer.setSize(window.innerWidth, window.innerHeight)
        vm.cssRenderer.domElement.style.position = 'absolute'
        vm.renderer.domElement.style.zIndex = 0
        vm.cssRenderer.domElement.style.top = 0
      },

      init: function(){
        var vm = this;
        vm.camera = new THREE.PerspectiveCamera(  30, window.innerWidth / window.innerHeight, 1, 10000 );
        vm.camera.position.set( 1000, 50, 1500 );

        // scene
        vm.scene = new THREE.Scene();
        vm.cssScene = new THREE.Scene();
        vm.scene.background = new THREE.Color( 0xcce0ff );
        var ambientLight = new THREE.AmbientLight( 0xcccccc, 0.4 );
        vm.scene.add( ambientLight );
        var pointLight = new THREE.PointLight( 0xffffff, 0.8 );
        vm.camera.add( pointLight );
        var d = 2300;

        vm.light = new THREE.DirectionalLight( 0xdfebff, 1 );
        vm.light.shadow.camera.left = - d;
        vm.light.shadow.camera.right = d;
        vm.light.shadow.camera.top = d;
        vm.light.shadow.camera.bottom = - d;
        vm.light.shadow.camera.far = 1000;
        vm.scene.add( vm.light);
        vm.scene.add( vm.camera );

        // model
        var onProgress = function ( xhr ) {
          if ( xhr.lengthComputable ) {
            var percentComplete = xhr.loaded / xhr.total * 100;
            console.log( Math.round(percentComplete, 2) + '% downloaded' );
          }
        };
        var onError = function ( xhr ) { };

        THREE.Loader.Handlers.add( /\.dds$/i, new THREE.DDSLoader() );
        var mtlLoader = new THREE.MTLLoader();
        mtlLoader.setPath( '/' );
        mtlLoader.load( 'cc.mtl', function( materials ) {
          materials.preload();
          var objLoader = new THREE.OBJLoader();
          objLoader.setMaterials( materials );
          objLoader.setPath( '/' );
          objLoader.load( 'cc.obj', function ( object ) {
            vm.model3d = object
            object.position.y = -230
            // object.rotation.y = Math.PI - .3
            vm.model3d.rotation.y = -1.658407

            vm.scene.add( object );
          }, onProgress, onError );
        });

        // ground
        var loader = new THREE.TextureLoader();
        var groundTexture = loader.load( '/grasslight-big.jpg' );
        groundTexture.wrapS = groundTexture.wrapT = THREE.RepeatWrapping;
        groundTexture.repeat.set( 25, 25 );
        groundTexture.anisotropy = 16;
        var groundMaterial = new THREE.MeshLambertMaterial( { map: groundTexture } );

        // add it to the WebGL scene    
        var mesh = new THREE.Mesh( new THREE.PlaneBufferGeometry( 1000, 1000 ), groundMaterial );
        mesh.position.y = - 250;
        mesh.rotation.x = - Math.PI / 2;
        // mesh.receiveShadow = true;
        // vm.scene.add( mesh );
        // vm.scene.add(vm.side);
        vm.renderer = new THREE.WebGLRenderer({ alpha: true });
        vm.renderer.setPixelRatio( window.devicePixelRatio );
        vm.renderer.setSize( window.innerWidth, window.innerHeight );
        // vm.renderer.gammaInput = true;
        vm.renderer.domElement.style.zIndex = 99;
        vm.renderer.gammaOutput = true;
        vm.renderer.domElement.style.position = 'absolute';
        vm.renderer.domElement.style.top = 0;
        vm.renderer.shadowMap.enabled = true;

        vm.createCssRenderer();
        
        // controls
        var controls = new THREE.OrbitControls( vm.camera, vm.renderer.domElement );
        controls.maxPolarAngle = Math.PI * 0.5;
        controls.minDistance = 1000;
        controls.maxDistance = 5000;
        vm.container.appendChild(vm.cssRenderer.domElement);
        vm.cssRenderer.domElement.appendChild(vm.renderer.domElement);
        vm.addMap()
      },
      animate: function(){
        var vm= this;
        requestAnimationFrame( vm.animate );
        this.render();
      }
    }
  }
</script>

<style>
  .container {
    min-height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    text-align: center;
  }

  .title {
    font-family: "Quicksand", "Source Sans Pro", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif; /* 1 */
    display: block;
    font-weight: 300;
    font-size: 100px;
    color: #35495e;
    letter-spacing: 1px;
  }

  .subtitle {
    font-weight: 300;
    font-size: 42px;
    color: #526488;
    word-spacing: 5px;
    padding-bottom: 15px;
  }

  .links {
    padding-top: 15px;
  }
</style>


