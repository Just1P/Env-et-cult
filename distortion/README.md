# Effet de Distortion 3D avec Three.js et GLSL Shaders

Ce projet démontre comment créer un effet de distortion dynamique sur un cube en utilisant Three.js et des shaders GLSL personnalisés. L'animation consiste en une déformation ondulante avec des couleurs qui évoluent dans le temps et offre une interface utilisateur pour contrôler les différents paramètres de l'effet.

## Aperçu technique

### Structure générale

Le projet utilise :

- **Three.js** pour le rendu 3D dans le navigateur
- **GLSL** pour les shaders personnalisés
- **HTML/CSS/JavaScript** pour l'interface utilisateur et les contrôles
- **Contrôle orbital** permettant de tourner autour de la forme à la souris

### Composants principaux

#### Configuration Three.js

```javascript
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
const renderer = new THREE.WebGLRenderer();
```

Cette partie initialise les éléments de base nécessaires pour Three.js : une scène, une caméra et un moteur de rendu WebGL.

#### Géométrie

```javascript
const geometry = new THREE.BoxGeometry(1.5, 1.5, 1.5, 50, 50, 50);
```

Un cube est utilisé comme base pour l'effet. Le cube est subdivisé en 50×50×50 segments pour permettre une déformation détaillée par le vertex shader.

#### Uniforms

```javascript
const warpUniforms = {
  time: { value: 0 },
  strength: { value: params.strength },
  frequency: { value: params.frequency },
};
```

Les uniforms sont des variables qui peuvent être passées aux shaders. Ici :

- `time` : contrôle l'animation au fil du temps
- `strength` : contrôle l'amplitude de la déformation
- `frequency` : contrôle la densité des ondulations

#### Vertex Shader

```glsl
uniform float time;
uniform float strength;
uniform float frequency;
varying vec2 vUv;
varying vec3 vPosition;

void main() {
    vUv = uv;
    vPosition = position;
    vec3 pos = position;

    // Calculer la distance depuis le centre du cube
    float distance = length(pos);

    // Distortion multi-vague en fonction de la position 3D
    float wave1 = sin(time * 0.7 + pos.x * frequency) * sin(time * 0.8 + pos.y * frequency) * sin(time * 0.9 + pos.z * frequency);
    float wave2 = sin(time * 0.5 + pos.x * frequency * 0.5) * sin(time * 0.6 + pos.y * frequency * 0.5) * sin(time * 0.7 + pos.z * frequency * 0.5) * 0.5;

    // Appliquer la distortion dans la direction normale à la surface
    vec3 normal = normalize(pos);
    float distortion = (wave1 + wave2) * strength;

    // Modifie la position en fonction de la distortion
    pos += normal * distortion;

    gl_Position = projectionMatrix * modelViewMatrix * vec4(pos, 1.0);
}
```

Ce shader transforme la position de chaque vertex du cube :

1. Il calcule la direction normale (du centre vers le vertex)
2. Il combine deux ondes sinusoïdales 3D avec des fréquences et des phases différentes
3. Il applique la déformation résultante dans la direction normale, créant un effet de "respiration" du cube
4. Il transmet les coordonnées de position au fragment shader via la variable `vPosition`

#### Fragment Shader

```glsl
varying vec2 vUv;
varying vec3 vPosition;
uniform float time;

void main() {
    // Utiliser la position 3D pour un effet de couleur plus complexe
    vec3 pos = normalize(vPosition);

    // Créer un motif basé sur la position 3D
    float pattern = sin(pos.x * 10.0 + time) * sin(pos.y * 10.0 + time) * sin(pos.z * 10.0 + time);

    // Couleurs changeantes basées sur le temps et la position
    vec3 color1 = vec3(0.5 + 0.5 * sin(time), 0.5 + 0.5 * sin(time * 0.7), 0.5 + 0.5 * sin(time * 0.3));
    vec3 color2 = vec3(0.5 + 0.5 * sin(time * 0.8), 0.5 + 0.5 * sin(time * 0.5), 0.5 + 0.5 * sin(time * 0.9));

    // Mélanger les couleurs en fonction du motif
    vec3 color = mix(color1, color2, pattern * 0.5 + 0.5);

    // Ajouter un effet de profondeur basé sur la position normalisée
    color *= 0.7 + 0.3 * length(pos);

    gl_FragColor = vec4(color, 1.0);
}
```

Ce shader détermine la couleur de chaque pixel :

1. Il utilise la position 3D normalisée pour créer un motif qui varie sur les trois axes
2. Il génère deux couleurs dynamiques qui changent avec le temps
3. Il mélange ces couleurs en fonction du motif pour créer un effet visuel riche
4. Il ajoute un effet de profondeur basé sur la distance du point au centre

#### Contrôles interactifs

```javascript
wireframeCheckbox.addEventListener("change", function () {
  material.wireframe = this.checked;
});
```

Des contrôles HTML permettent de modifier les paramètres en temps réel :

- Force de la distortion
- Vitesse d'animation
- Fréquence des ondulations
- Mode wireframe (affichage filaire)

#### Contrôle orbital de la caméra

```javascript
function onMouseMove(event) {
  if (isDragging) {
    mouseX = (event.clientX - windowHalfX) / 100;
    mouseY = (event.clientY - windowHalfY) / 100;

    targetRotationX = mouseX * 0.5;
    targetRotationY = mouseY * 0.5;
  }
}
```

Ce système permet à l'utilisateur de:

- Cliquer et faire glisser la souris pour tourner autour de la forme
- Observer l'effet de distortion sous différents angles
- Bénéficier d'une inertie qui continue légèrement la rotation après relâchement

#### Boucle d'animation et orbite de la caméra

```javascript
function animate() {
  requestAnimationFrame(animate);

  // Mise à jour du temps pour le shader
  warpUniforms.time.value += params.speed;

  // Animation de la caméra en rotation autour de la forme
  if (!isDragging) {
    // Ralentissement progressif de la rotation
    targetRotationX *= 0.98;
    targetRotationY *= 0.98;
  }

  // Positionner la caméra en orbite
  const radius = 2;
  const theta = targetRotationX;
  const phi = Math.max(
    -Math.PI / 2 + 0.1,
    Math.min(Math.PI / 2 - 0.1, targetRotationY)
  );

  camera.position.x = radius * Math.sin(theta) * Math.cos(phi);
  camera.position.y = radius * Math.sin(phi);
  camera.position.z = radius * Math.cos(theta) * Math.cos(phi);

  camera.lookAt(scene.position);
  renderer.render(scene, camera);
}
```

Cette fonction :

1. Met à jour la valeur du temps pour l'animation du shader
2. Calcule la position orbitale de la caméra basée sur les mouvements de souris
3. Applique une inertie à la rotation quand l'utilisateur relâche la souris
4. Convertit les coordonnées sphériques en coordonnées cartésiennes pour positionner la caméra
5. Oriente la caméra vers le centre de la scène
6. Effectue le rendu

## Système de caméra orbitale

Le contrôle orbital permet à l'utilisateur d'explorer la forme en 3D en faisant glisser la souris:

1. **Événements de souris**

   - `mousedown`: commence le suivi du mouvement
   - `mousemove`: calcule la rotation cible basée sur le mouvement
   - `mouseup`: termine le suivi

2. **Coordonnées sphériques**

   - `theta`: angle horizontal (autour de l'axe Y)
   - `phi`: angle vertical (autour de l'axe X)
   - `radius`: distance au centre (fixe dans cet exemple)

3. **Inertie**
   Le système applique un facteur de décélération (`targetRotation *= 0.98`) pour créer un effet d'inertie quand l'utilisateur relâche la souris, donnant ainsi une sensation plus naturelle au mouvement.

4. **Limites de rotation**
   Des contraintes sont appliquées à l'angle vertical (phi) pour éviter que la caméra ne se retourne complètement.

## Concepts GLSL expliqués

### Uniforms vs Varyings

- **Uniforms** : variables constantes pour un frame donné, modifiables depuis JavaScript
- **Varyings** : variables qui passent du vertex shader au fragment shader, interpolées pour chaque pixel

### Fonctions mathématiques en GLSL

- `sin()` : fonction sinus, utilisée ici pour créer des ondulations cycliques
- `mix()` : fonction de mélange linéaire entre deux valeurs

### Coordonnées UV

Les coordonnées UV (ou textures) varient de 0 à 1 sur la surface et permettent de positionner des motifs sur la géométrie.

## Astuces pour étendre le projet

1. **Ajouter plus de formes d'ondes**

   ```glsl
   float wave3 = cos(time * 0.3 + pos.x * frequency * 0.7 + pos.y * frequency * 0.3) * 0.3;
   ```

2. **Créer des effets de texture**

   ```glsl
   // Dans le fragment shader
   float noise = fract(sin(dot(vUv, vec2(12.9898, 78.233))) * 43758.5453);
   ```

3. **Ajouter des interactions à la souris**

   ```javascript
   // Suivre le mouvement de la souris
   let mouse = new THREE.Vector2();
   window.addEventListener("mousemove", (event) => {
     mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
     mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
   });

   // Ajouter un uniform pour la position de la souris
   mouse: {
     value: new THREE.Vector2();
   }

   // Mettre à jour dans animate()
   warpUniforms.mouse.value = mouse;
   ```

4. **Expérimenter avec des effets de post-processing**
   Three.js offre de nombreux effets comme le bloom, le blur, etc. qui peuvent être combinés avec vos shaders.

## Ressources pour approfondir

- [Documentation Three.js](https://threejs.org/docs/)
- [The Book of Shaders](https://thebookofshaders.com/)
- [GLSL Sandbox](http://glslsandbox.com/)
- [Shadertoy](https://www.shadertoy.com/)
