# Simulateur de Fluides en Temps Réel avec Three.js et GLSL

Ce projet est une implémentation d'une simulation de dynamique des fluides en 2D dans le navigateur, utilisant WebGL via Three.js et des shaders GLSL. La simulation est basée sur les équations de Navier-Stokes simplifiées et utilise une approche Eulérienne sur une grille.

## Principes physiques

La simulation repose sur plusieurs principes fondamentaux de la dynamique des fluides :

1. **Advection** : Le transport des propriétés du fluide (vélocité et densité) en fonction du champ de vélocité
2. **Diffusion** : L'étalement progressif des propriétés du fluide au fil du temps
3. **Projection** : La conservation de la masse (fluide incompressible) via la résolution d'une équation de Poisson

## Architecture du code

### Structures de données principales

- **Textures de vélocité** : Stockent le champ de vélocité du fluide (directions x et y)
- **Textures de densité** : Stockent la concentration du colorant (représentation visuelle)
- **Framebuffer Objects (FBOs)** : Permettent de faire du rendu vers des textures

### Pipeline de simulation

La simulation se fait en plusieurs étapes pour chaque frame :

1. **Advection de la vélocité** : Déplacement du champ de vélocité en fonction de lui-même
2. **Diffusion de la vélocité** : Application de la viscosité du fluide
3. **Calcul de la divergence** : Évaluation de la conservation de la masse
4. **Résolution de la pression** : Calcul de la pression nécessaire pour maintenir l'incompressibilité
5. **Projection** : Soustraction du gradient de pression du champ de vélocité
6. **Advection de la densité** : Déplacement de la densité en fonction du champ de vélocité
7. **Diffusion de la densité** : Étalement progressif de la densité

## Implémentation GLSL

### Shaders principaux

#### Advection Shader

```glsl
varying vec2 vUv;
uniform sampler2D uVelocity;
uniform sampler2D uSource;
uniform float uDt;
uniform float uDissipation;
uniform vec2 uTexelSize;

void main() {
  vec2 coord = vUv - uDt * texture2D(uVelocity, vUv).xy * uTexelSize;
  gl_FragColor = uDissipation * texture2D(uSource, coord);
}
```

Ce shader est responsable du mouvement des propriétés du fluide. Il utilise la technique du "backtracing" pour déterminer d'où vient chaque particule.

#### Diffusion Shader

```glsl
varying vec2 vUv;
uniform sampler2D uVelocity;
uniform sampler2D uSource;
uniform float uDiffusion;
uniform float uDt;
uniform vec2 uTexelSize;

void main() {
  // Échantillonnage des pixels voisins et application de la diffusion
  // ...
}
```

Implémente une solution itérative de l'équation de diffusion.

#### Projection Shaders

Comprend les shaders de divergence, de pression et de soustraction du gradient pour garantir l'incompressibilité du fluide.

### Paramètres ajustables

- **Viscosité** : Contrôle la résistance du fluide à la déformation (entre 0 et 0.2)
- **Diffusion** : Contrôle la vitesse de diffusion du colorant (entre 0 et 0.01)
- **Force d'injection** : Contrôle l'intensité des forces appliquées (entre 1 et 50)
- **Intensité des couleurs** : Contrôle la saturation visuelle (entre 1 et 10)

## Aspects techniques avancés

### Ping-Pong rendering

Le code utilise une technique appelée "ping-pong rendering" où deux textures sont alternativement utilisées comme source et destination pour chaque étape de la simulation.

```javascript
// Exemple de ping-pong
renderer.setRenderTarget(velocityTexture2);
renderer.render(mesh, camera);
const temp = velocityTexture1;
velocityTexture1 = velocityTexture2;
velocityTexture2 = temp;
```

### Méthode de Jacobi

La résolution de l'équation de pression utilise une méthode itérative de Jacobi pour résoudre l'équation de Poisson discrétisée.

### Interaction utilisateur

Les interactions de l'utilisateur (souris/toucher) sont converties en impulsions de vélocité et de densité qui sont injectées dans la simulation via le "Splat Shader".

## Optimisations et considérations de performance

1. **Résolution adaptative** : Possibilité d'avoir des résolutions différentes pour la simulation de vélocité et le rendu visuel
2. **Itérations paramétrables** : Le nombre d'itérations pour la diffusion et la résolution de pression peut être ajusté selon les besoins
3. **GPU-driven** : Toute la simulation est exécutée sur le GPU, ce qui permet des performances élevées

## Extensions possibles

1. **Obstacles et frontières** : Ajouter des objets solides avec lesquels le fluide peut interagir
2. **Vorticity Confinement** : Ajouter un terme de confinement de vorticité pour préserver les tourbillons
3. **Multiples fluides** : Simuler plusieurs fluides avec des densités et viscosités différentes
4. **Température** : Ajouter un champ de température qui affecte la dynamique du fluide

## Ressources et références

- Jos Stam, "Real-Time Fluid Dynamics for Games" (2003)
- [GPU Gems, Chapter 38: Fast Fluid Dynamics Simulation on the GPU](https://developer.nvidia.com/gpugems/gpugems/part-vi-beyond-triangles/chapter-38-fast-fluid-dynamics-simulation-gpu)
- [WebGL Fluid Simulation par Pavel Dobryakov](https://github.com/PavelDoGreat/WebGL-Fluid-Simulation)
