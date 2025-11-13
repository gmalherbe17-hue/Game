<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <title>Jeu 3D Mobile</title>
  <style>
    body { margin: 0; overflow: hidden; font-family: sans-serif; }
    canvas { display: block; }
    #score {
      position: absolute;
      top: 10px;
      left: 10px;
      color: white;
      font-size: 24px;
      background: rgba(0,0,0,0.5);
      padding: 5px 10px;
      border-radius: 5px;
      z-index: 10;
    }
    .btn {
      position: absolute;
      bottom: 20px;
      width: 60px;
      height: 60px;
      background: rgba(255,255,255,0.6);
      border-radius: 50%;
      text-align: center;
      line-height: 60px;
      font-size: 24px;
      user-select: none;
      z-index: 10;
    }
    #up { left: 50%; transform: translateX(-50%); }
    #down { left: 50%; transform: translateX(-50%); bottom: 80px; }
    #left { left: 20%; bottom: 50px; }
    #right { right: 20%; bottom: 50px; }
  </style>
</head>
<body>
<div id="score">Score: 0</div>
<div id="up" class="btn">↑</div>
<div id="down" class="btn">↓</div>
<div id="left" class="btn">←</div>
<div id="right" class="btn">→</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.158.0/build/three.min.js"></script>
<script>
  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
  const renderer = new THREE.WebGLRenderer();
  renderer.setSize(window.innerWidth, window.innerHeight);
  document.body.appendChild(renderer.domElement);

  // Sol
  const floorGeometry = new THREE.PlaneGeometry(50, 50);
  const floorMaterial = new THREE.MeshStandardMaterial({color: 0x444444});
  const floor = new THREE.Mesh(floorGeometry, floorMaterial);
  floor.rotation.x = -Math.PI/2;
  scene.add(floor);

  // Lumière
  const light = new THREE.DirectionalLight(0xffffff, 1);
  light.position.set(5,10,7.5);
  scene.add(light);

  // Joueur
  const playerGeometry = new THREE.BoxGeometry(1,2,1);
  const playerMaterial = new THREE.MeshStandardMaterial({color: 0x00ff00});
  const player = new THREE.Mesh(playerGeometry, playerMaterial);
  player.position.y = 1;
  scene.add(player);

  // Objets à collecter
  const collectibleGeometry = new THREE.SphereGeometry(0.5, 16, 16);
  const collectibleMaterial = new THREE.MeshStandardMaterial({color: 0xff0000});
  const collectibles = [];
  for(let i=0;i<5;i++){
    const obj = new THREE.Mesh(collectibleGeometry, collectibleMaterial);
    obj.position.set(Math.random()*40-20,0.5,Math.random()*40-20);
    scene.add(obj);
    collectibles.push(obj);
  }

  // Obstacles
  const obstacleGeometry = new THREE.BoxGeometry(2,2,2);
  const obstacleMaterial = new THREE.MeshStandardMaterial({color: 0x0000ff});
  const obstacles = [];
  for(let i=0;i<5;i++){
    const obs = new THREE.Mesh(obstacleGeometry, obstacleMaterial);
    obs.position.set(Math.random()*40-20,1,Math.random()*40-20);
    scene.add(obs);
    obstacles.push(obs);
  }

  camera.position.set(0,5,10);
  camera.lookAt(player.position);

  // Déplacement mobile
  const keys = {up:false, down:false, left:false, right:false};

  function setKey(key, value){
    keys[key] = value;
  }

  document.getElementById('up').addEventListener('touchstart', ()=>setKey('up',true));
  document.getElementById('up').addEventListener('touchend', ()=>setKey('up',false));
  document.getElementById('down').addEventListener('touchstart', ()=>setKey('down',true));
  document.getElementById('down').addEventListener('touchend', ()=>setKey('down',false));
  document.getElementById('left').addEventListener('touchstart', ()=>setKey('left',true));
  document.getElementById('left').addEventListener('touchend', ()=>setKey('left',false));
  document.getElementById('right').addEventListener('touchstart', ()=>setKey('right',true));
  document.getElementById('right').addEventListener('touchend', ()=>setKey('right',false));

  let score = 0;
  const scoreDiv = document.getElementById('score');

  function checkCollision(obj1, obj2, distance=1) {
    return obj1.position.distanceTo(obj2.position) < distance;
  }

  function animate() {
    requestAnimationFrame(animate);

    // Déplacement joueur
    if(keys.up) player.position.z -= 0.1;
    if(keys.down) player.position.z += 0.1;
    if(keys.left) player.position.x -= 0.1;
    if(keys.right) player.position.x += 0.1;

    // Collision collectibles
    collectibles.forEach((c,i)=>{
      if(c && checkCollision(player, c, 1)){
        scene.remove(c);
        collectibles[i] = null;
        score++;
        scoreDiv.innerText = `Score: ${score}`;
      }
    });

    // Collision obstacles
    obstacles.forEach(o=>{
      if(checkCollision(player, o, 1.5)){
        if(keys.left) player.position.x += 0.1;
        if(keys.right) player.position.x -= 0.1;
        if(keys.up) player.position.z += 0.1;
        if(keys.down) player.position.z -= 0.1;
      }
    });

    camera.position.x = player.position.x;
    camera.position.z = player.position.z + 10;
    camera.lookAt(player.position);

    renderer.render(scene, camera);
  }
  animate();

  window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth/window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
  });
</script>
</body>
</html>
