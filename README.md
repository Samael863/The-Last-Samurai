<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8">

<meta name="viewport"
      content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">

<title>RONIN: Shadows of the Shogun</title>

<style>
* {
    box-sizing: border-box;
}

:root {
    --gold: #e8bd68;
    --red: #c83d3d;
    --dark: rgba(12, 13, 17, .92);
    --border: rgba(232, 189, 104, .4);
}

html, body {
    margin: 0;
    width: 100%;
    height: 100%;
    overflow: hidden;
    background: #10171b;
    color: white;
    font-family: Arial, Helvetica, sans-serif;
    touch-action: none;
}

button {
    color: white;
    font: inherit;
    cursor: pointer;
}

#game {
    position: fixed;
    inset: 0;
}

canvas {
    display: block;
    width: 100%;
    height: 100%;
}

/* HUD */

#hud {
    position: fixed;
    inset: 0;
    pointer-events: none;
}

.panel {
    background: var(--dark);
    border: 1px solid var(--border);
    padding: 12px;
    box-shadow: 0 5px 20px #0006;
    backdrop-filter: blur(5px);
}

#playerPanel {
    position: absolute;
    left: 16px;
    top: 16px;
    width: 255px;
    border-radius: 8px;
}

.gameTitle {
    color: var(--gold);
    font-size: 18px;
    font-weight: 900;
    letter-spacing: 2px;
    margin-bottom: 8px;
}

.small {
    color: #b6b6b6;
    font-size: 11px;
}

.bar {
    height: 9px;
    margin-top: 5px;
    background: #28282b;
    overflow: hidden;
    border-radius: 3px;
}

.bar > div {
    width: 100%;
    height: 100%;
    transition: width .15s;
}

#healthBar {
    background: linear-gradient(90deg, #8d1717, #ff4949);
}

#staminaBar {
    background: linear-gradient(90deg, #a87b20, #ffe18a);
}

.statLine {
    margin-top: 9px;
    font-size: 12px;
    color: #ddd;
}

#questPanel {
    position: absolute;
    right: 16px;
    top: 16px;
    width: 275px;
    border-radius: 8px;
}

#questTitle {
    color: var(--gold);
    font-size: 13px;
    font-weight: bold;
    margin-bottom: 8px;
}

#questText {
    font-size: 12px;
    line-height: 1.7;
    color: #ddd;
}

#targetPanel {
    position: absolute;
    top: 115px;
    left: 50%;
    transform: translateX(-50%);
    width: 230px;
    display: none;
    text-align: center;
}

#targetName {
    color: #ff8c8c;
    font-weight: bold;
    margin-bottom: 7px;
}

#targetHealth {
    background: #8d1717;
}

#prompt {
    position: absolute;
    left: 50%;
    bottom: 110px;
    transform: translateX(-50%);
    padding: 11px 16px;
    border-radius: 6px;
    background: #101010e8;
    border: 1px solid var(--border);
    color: var(--gold);
    font-size: 13px;
    text-align: center;
    display: none;
    max-width: 90%;
}

#toast {
    position: absolute;
    top: 35%;
    left: 50%;
    transform: translate(-50%, -50%);
    padding: 15px 24px;
    border: 1px solid var(--gold);
    border-radius: 8px;
    background: #101010ed;
    color: var(--gold);
    font-weight: bold;
    text-align: center;
    display: none;
    max-width: 90%;
}

#crosshair {
    position: absolute;
    left: 50%;
    top: 50%;
    width: 5px;
    height: 5px;
    border: 1px solid #ffffffa0;
    border-radius: 50%;
    transform: translate(-50%, -50%);
}

/* MENU */

.overlay {
    position: fixed;
    inset: 0;
    z-index: 10;
    display: flex;
    align-items: center;
    justify-content: center;
    background: linear-gradient(135deg, #090d12ee, #17130ff2);
    padding: 18px;
}

.menuBox {
    width: min(650px, 100%);
    max-height: 92vh;
    overflow-y: auto;
    padding: 28px;
    background: #11151beF;
    border: 1px solid var(--gold);
    border-radius: 12px;
    box-shadow: 0 0 80px #000;
}

.menuTitle {
    color: var(--gold);
    font-size: clamp(27px, 5vw, 45px);
    font-weight: 900;
    letter-spacing: 3px;
    text-align: center;
}

.subtitle {
    text-align: center;
    color: #aaa;
    letter-spacing: 4px;
    margin-top: 8px;
    font-size: 12px;
}

.menuDescription {
    line-height: 1.8;
    font-size: 14px;
    color: #ddd;
    margin: 24px 0;
}

.goldButton {
    width: 100%;
    padding: 14px;
    margin-top: 9px;
    background: linear-gradient(120deg, #76501a, #e8bd68);
    color: #160f06;
    font-weight: 900;
    border: 0;
    border-radius: 5px;
    letter-spacing: 1px;
}

.goldButton:hover {
    filter: brightness(1.15);
}

.secondaryButton {
    width: 100%;
    padding: 12px;
    margin-top: 8px;
    background: #20252b;
    border: 1px solid #555;
    border-radius: 5px;
}

/* INVENTAR */

#inventory {
    position: fixed;
    z-index: 8;
    inset: 0;
    display: none;
    justify-content: center;
    align-items: center;
    background: #000b;
    padding: 12px;
}

.inventoryBox {
    width: min(800px, 100%);
    max-height: 90vh;
    overflow-y: auto;
    background: #10151c;
    border: 1px solid var(--gold);
    border-radius: 8px;
    padding: 18px;
}

.inventoryHeader {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 10px;
    margin-bottom: 16px;
}

.inventoryHeader h2 {
    color: var(--gold);
    margin: 0;
}

#inventoryItems {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
    gap: 10px;
}

.itemCard {
    border: 1px solid #42464b;
    border-left: 4px solid var(--itemColor, #999);
    border-radius: 5px;
    padding: 12px;
    background: #191e25;
}

.itemName {
    font-weight: bold;
    margin-bottom: 5px;
}

.itemDescription {
    color: #aaa;
    font-size: 12px;
    line-height: 1.6;
}

.itemCard button {
    margin-top: 10px;
    width: 100%;
    padding: 8px;
    background: #303944;
    border: 1px solid #59616a;
    border-radius: 4px;
}

#inventoryClose {
    padding: 9px 15px;
    background: #333;
    border: 1px solid #777;
}

/* MOBILE */

#mobileControls {
    display: none;
    position: fixed;
    inset: 0;
    pointer-events: none;
}

#joystick {
    position: absolute;
    left: 22px;
    bottom: 25px;
    width: 125px;
    height: 125px;
    border-radius: 50%;
    border: 2px solid #ffffff55;
    background: #0005;
    pointer-events: auto;
    touch-action: none;
}

#stick {
    position: absolute;
    left: 32px;
    top: 32px;
    width: 57px;
    height: 57px;
    border-radius: 50%;
    background: #d4b46d99;
    border: 2px solid #fff9;
    pointer-events: none;
}

.mobileButtons {
    position: absolute;
    right: 17px;
    bottom: 22px;
    display: grid;
    grid-template-columns: repeat(2, 67px);
    gap: 9px;
    pointer-events: auto;
}

.mobileButtons button {
    width: 67px;
    height: 58px;
    background: #101820d9;
    border: 1px solid var(--gold);
    border-radius: 10px;
    font-size: 11px;
    font-weight: bold;
    touch-action: none;
}

.mobileButtons button:active {
    background: #8d692b;
}

@media (pointer: coarse), (max-width: 850px) {
    #mobileControls {
        display: block;
    }

    #playerPanel {
        left: 8px;
        top: 8px;
        width: 185px;
        padding: 8px;
    }

    .gameTitle {
        font-size: 13px;
    }

    #questPanel {
        right: 8px;
        top: 8px;
        width: 170px;
        padding: 8px;
    }

    #questText {
        font-size: 10px;
    }

    .statLine {
        font-size: 10px;
    }

    #targetPanel {
        top: 100px;
    }

    #prompt {
        bottom: 170px;
    }

    #crosshair {
        opacity: .7;
    }
}
</style>
</head>

<body>

<div id="game"></div>

<div id="hud">

    <div id="playerPanel" class="panel">
        <div class="gameTitle">RONIN</div>

        <div class="small">LEBEN</div>
        <div class="bar">
            <div id="healthBar"></div>
        </div>

        <div class="small" style="margin-top:8px">AUSDAUER</div>
        <div class="bar">
            <div id="staminaBar"></div>
        </div>

        <div class="statLine" id="stats">
            ❤️ 100/100
        </div>

        <div class="statLine" id="equipment">
            ⚔️ Wanderer-Katana
        </div>

        <div class="statLine" id="coins">
            🪙 0 Mon
        </div>
    </div>

    <div id="questPanel" class="panel">
        <div id="questTitle">AKTUELLE MISSION</div>
        <div id="questText">
            Erkunde die Welt.
        </div>
    </div>

    <div id="targetPanel" class="panel">
        <div id="targetName">Gegner</div>
        <div class="bar">
            <div id="targetHealth"></div>
        </div>
    </div>

    <div id="crosshair"></div>

    <div id="prompt"></div>

    <div id="toast"></div>

</div>

<!-- STARTMENÜ -->

<div id="startScreen" class="overlay">

    <div class="menuBox">

        <div class="menuTitle">
            RONIN
        </div>

        <div class="subtitle">
            SHADOWS OF THE SHOGUN
        </div>

        <div class="menuDescription">

            <p>
                Eine vergessene Provinz. Ein zerfallenes Reich.
                Ein Samurai, der seinen eigenen Weg gehen muss.
            </p>

            <p>
                Erkunde Wälder, Tempel und die Festung des
                Shoguns. Sammle Waffen, finde legendäre Rüstungen
                und stelle dich den Kriegern, die das Land
                beherrschen.
            </p>

            <p>
                <b>PC:</b> WASD bewegen · Maus drehen · Linksklick angreifen
                · Shift ausweichen · Leertaste springen · E interagieren
                · I Inventar
            </p>

            <p>
                <b>Mobile:</b> Linker Joystick bewegen · rechte
                Bildschirmhälfte drehen · Aktionstasten verwenden
            </p>

        </div>

        <button id="startButton" class="goldButton">
            SPIEL STARTEN
        </button>

        <button id="loadButton" class="secondaryButton">
            SPIELSTAND LADEN
        </button>

        <div class="small" style="margin-top:16px;text-align:center">
            PAD TO HOME STUDIOS · RONIN V1.0
        </div>

    </div>

</div>

<!-- INVENTAR -->

<div id="inventory">

    <div class="inventoryBox">

        <div class="inventoryHeader">
            <h2>⚔️ AUSRÜSTUNG</h2>

            <button id="inventoryClose">
                SCHLIESSEN
            </button>
        </div>

        <div id="inventoryItems"></div>

    </div>

</div>

<!-- MOBILE STEUERUNG -->

<div id="mobileControls">

    <div id="joystick">
        <div id="stick"></div>
    </div>

    <div class="mobileButtons">

        <button id="btnAttack">⚔️ ANGRIFF</button>

        <button id="btnHeavy">💥 SCHWER</button>

        <button id="btnJump">⬆ SPRUNG</button>

        <button id="btnDodge">💨 AUSWEICHEN</button>

        <button id="btnInteract">✋ AKTION</button>

        <button id="btnInventory">🎒 INVENTAR</button>

    </div>

</div>

<script type="module">

import * as THREE from
'https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js';


/* =====================================================
   RONIN: SHADOWS OF THE SHOGUN
   Vollständige spielbare Browser-Spielversion
   ===================================================== */


/* -----------------------------------------------------
   GRUNDLAGEN
----------------------------------------------------- */

const container = document.getElementById("game");

const scene = new THREE.Scene();

scene.background = new THREE.Color(0x9eb7c0);

scene.fog = new THREE.FogExp2(0x9eb7c0, 0.0017);

const camera = new THREE.PerspectiveCamera(
    65,
    innerWidth / innerHeight,
    0.1,
    1200
);

const renderer = new THREE.WebGLRenderer({
    antialias: true,
    powerPreference: "high-performance"
});

renderer.setSize(innerWidth, innerHeight);

renderer.setPixelRatio(
    Math.min(devicePixelRatio || 1, 1.5)
);

renderer.shadowMap.enabled = true;

renderer.shadowMap.type = THREE.PCFSoftShadowMap;

renderer.outputColorSpace = THREE.SRGBColorSpace;

container.appendChild(renderer.domElement);


/* LICHT */

scene.add(
    new THREE.HemisphereLight(
        0xd9edff,
        0x464032,
        2.0
    )
);

const sun = new THREE.DirectionalLight(
    0xffe4b2,
    2.5
);

sun.position.set(-100, 160, 80);

sun.castShadow = true;

sun.shadow.mapSize.set(1024, 1024);

sun.shadow.camera.left = -100;
sun.shadow.camera.right = 100;
sun.shadow.camera.top = 100;
sun.shadow.camera.bottom = -100;

scene.add(sun);


/* -----------------------------------------------------
   SPIELZUSTAND
----------------------------------------------------- */

let running = false;

let inventoryOpen = false;

let climbing = false;

let regularKills = 0;

let coins = 0;

let toastTimer = 0;

let promptText = "";

const keys = {};

const worldColliders = [];

const platforms = [];

const enemies = [];

const pickups = [];

const chests = [];

let targetEnemy = null;

let currentInteractable = null;

let cameraYaw = 0;

let cameraPitch = 0.24;

const clock = new THREE.Clock();


/* -----------------------------------------------------
   HILFSFUNKTIONEN
----------------------------------------------------- */

const rand = (a, b) =>
    a + Math.random() * (b - a);

const clamp = (v, a, b) =>
    Math.max(a, Math.min(b, v));

function distanceXZ(a, b) {
    return Math.hypot(
        a.x - b.x,
        a.z - b.z
    );
}

function makeMaterial(color, roughness = 0.9) {

    return new THREE.MeshStandardMaterial({
        color,
        roughness
    });

}

function box(
    parent,
    x, y, z,
    sx, sy, sz,
    color,
    options = {}
) {

    const mesh = new THREE.Mesh(
        new THREE.BoxGeometry(sx, sy, sz),
        makeMaterial(color)
    );

    mesh.position.set(x, y, z);

    mesh.castShadow = true;

    mesh.receiveShadow = true;

    if (options.rotationY) {
        mesh.rotation.y = options.rotationY;
    }

    parent.add(mesh);

    return mesh;
}

function cylinder(
    parent,
    x, y, z,
    rt, rb, height,
    color,
    segments = 8
) {

    const mesh = new THREE.Mesh(
        new THREE.CylinderGeometry(
            rt, rb, height, segments
        ),
        makeMaterial(color)
    );

    mesh.position.set(x, y, z);

    mesh.castShadow = true;

    mesh.receiveShadow = true;

    parent.add(mesh);

    return mesh;
}

function sphere(
    parent,
    x, y, z,
    sx, sy, sz,
    color
) {

    const mesh = new THREE.Mesh(
        new THREE.SphereGeometry(1, 10, 8),
        makeMaterial(color)
    );

    mesh.position.set(x, y, z);

    mesh.scale.set(sx, sy, sz);

    mesh.castShadow = true;

    parent.add(mesh);

    return mesh;
}

function showToast(text, duration = 2400) {

    const el = document.getElementById("toast");

    el.textContent = text;

    el.style.display = "block";

    toastTimer = duration / 1000;

}

function playTone(frequency = 240, duration = 0.08) {

    try {

        const AudioContextClass =
            window.AudioContext ||
            window.webkitAudioContext;

        if (!AudioContextClass) return;

        if (!window._roninAudio) {
            window._roninAudio =
                new AudioContextClass();
        }

        const ctx = window._roninAudio;

        if (ctx.state === "suspended") {
            ctx.resume();
        }

        const oscillator = ctx.createOscillator();

        const gain = ctx.createGain();

        oscillator.type = "triangle";

        oscillator.frequency.value = frequency;

        gain.gain.setValueAtTime(
            0.12,
            ctx.currentTime
        );

        gain.gain.exponentialRampToValueAtTime(
            0.001,
            ctx.currentTime + duration
        );

        oscillator.connect(gain);

        gain.connect(ctx.destination);

        oscillator.start();

        oscillator.stop(
            ctx.currentTime + duration
        );

    } catch (error) {

        /* Sound ist optional. */

    }

}


/* -----------------------------------------------------
   TERRAIN UND PHYSIK-GRUNDLAGE
----------------------------------------------------- */

function terrainHeight(x, z) {

    return (
        Math.sin(x * 0.025) *
        Math.cos(z * 0.031) * 1.5 +

        Math.sin(z * 0.042 + x * 0.012) * 0.8
    );

}

function groundAt(x, z) {

    let height = terrainHeight(x, z);

    for (const p of platforms) {

        if (
            Math.abs(x - p.x) <= p.halfX &&
            Math.abs(z - p.z) <= p.halfZ
        ) {

            height = Math.max(
                height,
                p.top
            );

        }

    }

    return height;

}


/* TERRAIN-GEOMETRIE */

function createTerrain() {

    const size = 800;

    const segments = 160;

    const geometry = new THREE.PlaneGeometry(
        size,
        size,
        segments,
        segments
    );

    const positions =
        geometry.attributes.position;

    const colors = [];

    const low = new THREE.Color(0x394d35);

    const mid = new THREE.Color(0x65744b);

    const high = new THREE.Color(0x777d71);

    for (let i = 0; i < positions.count; i++) {

        const x = positions.getX(i);

        const z = positions.getY(i);

        const y = terrainHeight(x, z);

        positions.setXYZ(i, x, y, z);

        let color;

        if (y < 0) {

            color = low.clone().lerp(
                mid,
                clamp((y + 3) / 3, 0, 1)
            );

        } else {

            color = mid.clone().lerp(
                high,
                clamp(y / 5, 0, 1)
            );

        }

        colors.push(
            color.r,
            color.g,
            color.b
        );

    }

    geometry.setAttribute(
        "color",
        new THREE.Float32BufferAttribute(colors, 3)
    );

    geometry.computeVertexNormals();

    const material = new THREE.MeshStandardMaterial({
        vertexColors: true,
        roughness: 1
    });

    const ground = new THREE.Mesh(
        geometry,
        material
    );

    ground.receiveShadow = true;

    scene.add(ground);

}

createTerrain();


/* -----------------------------------------------------
   WELTBAU
----------------------------------------------------- */

function addTree(x, z, scale = 1) {

    const group = new THREE.Group();

    group.position.set(
        x,
        terrainHeight(x, z),
        z
    );

    cylinder(
        group,
        0, 2 * scale, 0,
        0.35 * scale,
        0.5 * scale,
        4 * scale,
        0x60452f,
        7
    );

    sphere(
        group,
        0, 4.1 * scale, 0,
        1.7 * scale,
        1.9 * scale,
        1.7 * scale,
        0x2e5538
    );

    sphere(
        group,
        0.5 * scale,
        5 * scale,
        0.1 * scale,
        1.3 * scale,
        1.5 * scale,
        1.3 * scale,
        0x436b43
    );

    scene.add(group);

    worldColliders.push({
        x, z,
        r: 1.0 * scale
    });

}


function addRock(x, z, scale = 1) {

    const rock = sphere(
        scene,
        x,
        terrainHeight(x, z) + scale,
        z,
        scale * 1.5,
        scale,
        scale,
        0x686963
    );

    worldColliders.push({
        x, z,
        r: scale * 1.3
    });

    return rock;

}


function addHouse(x, z, scale = 1) {

    const y = terrainHeight(x, z);

    const group = new THREE.Group();

    group.position.set(x, y, z);

    box(
        group,
        0, 2 * scale, 0,
        5 * scale,
        4 * scale,
        4 * scale,
        0x8a6748
    );

    box(
        group,
        0, 4.3 * scale, 0,
        6 * scale,
        0.6 * scale,
        5 * scale,
        0x46312b
    );

    box(
        group,
        0, 1.1 * scale,
        -2.05 * scale,
        1.1 * scale,
        2.2 * scale,
        0.1 * scale,
        0x35251e
    );

    scene.add(group);

    worldColliders.push({
        x, z,
        r: 3.5 * scale
    });

}


function addTorii(x, z) {

    const y = terrainHeight(x, z);

    const group = new THREE.Group();

    group.position.set(x, y, z);

    box(
        group,
        -2, 2.3, 0,
        0.35, 4.6, 0.35,
        0x9c2929
    );

    box(
        group,
        2, 2.3, 0,
        0.35, 4.6, 0.35,
        0x9c2929
    );

    box(
        group,
        0, 4.6, 0,
        6.1, 0.4, 0.65,
        0x33221e
    );

    box(
        group,
        0, 4.0, 0,
        5.0, 0.35, 0.35,
        0x9c2929
    );

    scene.add(group);

}


function addCastle(x, z) {

    const y = terrainHeight(x, z);

    const group = new THREE.Group();

    group.position.set(x, y, z);

    box(
        group,
        0, 4, 0,
        24, 8, 22,
        0x77766f
    );

    box(
        group,
        0, 8.7, 0,
        29, 1.2, 26,
        0x343b42
    );

    box(
        group,
        0, 12, 0,
        17, 6, 15,
        0xd0c6ac
    );

    box(
        group,
        0, 15.3, 0,
        22, 1, 20,
        0x373c43
    );

    box(
        group,
        0, 17.5, 0,
        11, 4, 10,
        0xe1d5b9
    );

    box(
        group,
        0, 19.8, 0,
        15, 0.8, 14,
        0x343941
    );

    scene.add(group);

    worldColliders.push({
        x, z,
        r: 15
    });

}


/* WELT ERZEUGEN */

function createWorld() {

    for (let i = 0; i < 180; i++) {

        let x = rand(-350, 350);

        let z = rand(-350, 350);

        if (Math.hypot(x, z) < 25) {
            continue;
        }

        if (Math.hypot(x - 70, z + 80) < 24) {
            continue;
        }

        addTree(
            x,
            z,
            rand(0.65, 1.35)
        );

    }

    for (let i = 0; i < 48; i++) {

        let x = rand(-350, 350);

        let z = rand(-350, 350);

        addRock(
            x,
            z,
            rand(0.5, 2.2)
        );

    }

    addHouse(-18, -5, 1);

    addHouse(-30, -15, 0.9);

    addHouse(-15, -27, 1.1);

    addHouse(22, 20, 1.2);

    addHouse(32, 26, 0.8);

    addTorii(0, -29);

    addTorii(-55, 48);

    addCastle(95, -110);

    /* Erhöhte Kletterplattform */

    const cliffX = 70;

    const cliffZ = -80;

    const base = terrainHeight(
        cliffX,
        cliffZ
    );

    box(
        scene,
        cliffX,
        base + 8,
        cliffZ,
        22,
        16,
        4,
        0x77786f
    );

    box(
        scene,
        cliffX,
        base + 15.5,
        cliffZ - 7,
        25,
        1,
        15,
        0x65665f
    );

    platforms.push({
        x: cliffX,
        z: cliffZ - 7,
        halfX: 12,
        halfZ: 7,
        top: base + 16
    });

    worldColliders.push({
        x: cliffX,
        z: cliffZ,
        halfX: 11,
        halfZ: 2.5,
        rect: true
    });

}

createWorld();


/* -----------------------------------------------------
   CHARAKTER-MODELL
----------------------------------------------------- */

function makeHumanoid(
    armorColor = 0x392d2c,
    clothColor = 0x202d37,
    bladeColor = 0xbfcbd2
) {

    const root = new THREE.Group();

    const armorMat = makeMaterial(armorColor);

    const clothMat = makeMaterial(clothColor);

    const bladeMat = new THREE.MeshStandardMaterial({
        color: bladeColor,
        metalness: 0.8,
        roughness: 0.25
    });

    /* Körper */

    const torso = new THREE.Mesh(
        new THREE.BoxGeometry(0.85, 0.95, 0.45),
        armorMat
    );

    torso.position.y = 1.35;

    torso.castShadow = true;

    root.add(torso);

    /* Untergewand */

    const tunic = new THREE.Mesh(
        new THREE.BoxGeometry(0.72, 0.8, 0.48),
        clothMat
    );

    tunic.position.y = 1.3;

    root.add(tunic);

    /* Kopf */

    sphere(
        root,
        0, 2.1, 0,
        0.29, 0.34, 0.29,
        0xc89d7d
    );

    /* Helm */

    const helmet = new THREE.Mesh(
        new THREE.SphereGeometry(
            0.34, 10, 8, 0, Math.PI * 2, 0, Math.PI / 2
        ),
        armorMat
    );

    helmet.position.y = 2.28;

    helmet.castShadow = true;

    root.add(helmet);

    /* Hörner */

    cylinder(
        root,
        -0.27, 2.48, 0,
        0.045, 0.11, 0.38,
        armorColor,
        6
    );

    cylinder(
        root,
        0.27, 2.48, 0,
        0.045, 0.11, 0.38,
        armorColor,
        6
    );

    /* Arme */

    cylinder(
        root,
        -0.55, 1.35, 0,
        0.13, 0.16, 0.68,
        armorColor
    );

    cylinder(
        root,
        0.55, 1.35, 0,
        0.13, 0.16, 0.68,
        armorColor
    );

    /* Beine */

    cylinder(
        root,
        -0.22, 0.48, 0,
        0.15, 0.19, 0.7,
        clothColor
    );

    cylinder(
        root,
        0.22, 0.48, 0,
        0.15, 0.19, 0.7,
        clothColor
    );

    /* Schulterplatten */

    box(
        root,
        -0.55, 1.72, 0,
        0.42, 0.24, 0.5,
        armorColor
    );

    box(
        root,
        0.55, 1.72, 0,
        0.42, 0.24, 0.5,
        armorColor
    );

    /* Schwertgriff und Klinge */

    const sword = new THREE.Group();

    cylinder(
        sword,
        0, 0, 0,
        0.055, 0.055, 0.28,
        0x171514,
        6
    );

    const blade = new THREE.Mesh(
        new THREE.BoxGeometry(
            0.075, 0.95, 0.035
        ),
        bladeMat
    );

    blade.position.y = 0.57;

    blade.castShadow = true;

    sword.add(blade);

    box(
        sword,
        0, 0.16, 0,
        0.18, 0.055, 0.08,
        0x8c713d
    );

    sword.position.set(
        0.65, 1.2, -0.1
    );

    sword.rotation.z = 0.3;

    root.add(sword);

    /* Schwertscheide */

    box(
        root,
        -0.35, 0.85, 0.22,
        0.1, 0.85, 0.12,
        0x1b1919
    );

    return {
        root,
        sword,
        armorMat,
        clothMat,
        bladeMat
    };

}


/* -----------------------------------------------------
   SPIELER
----------------------------------------------------- */

const player = {

    root: new THREE.Group(),

    model: null,

    sword: null,

    armorMat: null,

    bladeMat: null,

    x: 0,

    z: 0,

    y: 0,

    vy: 0,

    grounded: true,

    health: 100,

    maxHealth: 100,

    stamina: 100,

    maxStamina: 100,

    invulnerable: 0,

    attackTimer: 0,

    attackCooldown: 0,

    dodgeTimer: 0,

    heavyAttack: false,

    dodgeX: 0,

    dodgeZ: -1,

    climbing: false,

    coins: 0,

    inventory: [],

    weapon: null,

    armor: null

};

player.model = makeHumanoid(
    0x702e31,
    0x222a36,
    0xd6e2e9
);

player.root.add(player.model.root);

player.sword = player.model.sword;

player.armorMat = player.model.armorMat;

player.bladeMat = player.model.bladeMat;

scene.add(player.root);


/* -----------------------------------------------------
   WAFFEN UND SELTENHEITEN
----------------------------------------------------- */

const rarityData = {

    "Gewöhnlich": {
        color: 0xb4b4b4,
        multiplier: 1
    },

    "Ungewöhnlich": {
        color: 0x56d16b,
        multiplier: 1.15
    },

    "Selten": {
        color: 0x4798ff,
        multiplier: 1.4
    },

    "Episch": {
        color: 0xc16bff,
        multiplier: 1.8
    },

    "Legendär": {
        color: 0xffc52d,
        multiplier: 2.5
    },

    "Mythisch": {
        color: 0xff4545,
        multiplier: 3.5
    }

};


const weaponDefinitions = [

    {
        name: "Wanderer-Katana",
        damage: 28,
        speed: 1,
        type: "weapon"
    },

    {
        name: "Wakizashi",
        damage: 22,
        speed: 1.35,
        type: "weapon"
    },

    {
        name: "Naginata",
        damage: 34,
        speed: 0.85,
        type: "weapon"
    },

    {
        name: "Yari-Speer",
        damage: 31,
        speed: 0.95,
        type: "weapon"
    },

    {
        name: "Kusarigama",
        damage: 26,
        speed: 1.2,
        type: "weapon"
    },

    {
        name: "Kanabō",
        damage: 45,
        speed: 0.65,
        type: "weapon"
    },

    {
        name: "Drachen-Katana",
        damage: 55,
        speed: 1.1,
        type: "weapon",
        legendary: true
    },

    {
        name: "Klinge des Dämonenkönigs",
        damage: 75,
        speed: 1.05,
        type: "weapon",
        mythical: true
    }

];


const armorDefinitions = [

    {
        name: "Wanderer-Rüstung",
        defense: 2,
        color: 0x54423b,
        type: "armor"
    },

    {
        name: "Samurai-Rüstung",
        defense: 5,
        color: 0x702e31,
        type: "armor"
    },

    {
        name: "Schattenrüstung",
        defense: 8,
        color: 0x252a35,
        type: "armor"
    },

    {
        name: "Drachenrüstung",
        defense: 12,
        color: 0xc08a30,
        type: "armor"
    },

    {
        name: "Rüstung des Oni",
        defense: 16,
        color: 0x7e2531,
        type: "armor"
    }

];


function rollRarity() {

    const r = Math.random();

    if (r < 0.005) return "Mythisch";

    if (r < 0.035) return "Legendär";

    if (r < 0.12) return "Episch";

    if (r < 0.32) return "Selten";

    if (r < 0.65) return "Ungewöhnlich";

    return "Gewöhnlich";

}


function createItem(definition, rarity = rollRarity()) {

    const rarityInfo = rarityData[rarity];

    const item = {
        ...definition,
        rarity,
        rarityColor: rarityInfo.color
    };

    if (item.type === "weapon") {

        item.damage = Math.round(
            item.damage * rarityInfo.multiplier
        );

    } else {

        item.defense = Math.round(
            item.defense * rarityInfo.multiplier
        );

    }

    if (item.mythical) {

        item.rarity = "Mythisch";

        item.rarityColor =
            rarityData["Mythisch"].color;

    }

    if (item.legendary) {

        item.rarity = "Legendär";

        item.rarityColor =
            rarityData["Legendär"].color;

    }

    return item;

}


/* STARTAUSRÜSTUNG */

player.weapon = createItem(
    weaponDefinitions[0],
    "Gewöhnlich"
);

player.armor = createItem(
    armorDefinitions[0],
    "Gewöhnlich"
);

player.inventory.push(
    player.weapon,
    player.armor
);


/* -----------------------------------------------------
   AUSRÜSTUNG ANZEIGEN
----------------------------------------------------- */

function equipItem(item) {

    if (item.type === "weapon") {

        player.weapon = item;

        player.bladeMat.color.setHex(
            item.rarityColor
        );

        player.bladeMat.metalness = 0.8;

        showToast(
            "AUSGERÜSTET: " + item.name
        );

    } else if (item.type === "armor") {

        player.armor = item;

        player.armorMat.color.setHex(
            item.color
        );

        showToast(
            "RÜSTUNG AUSGERÜSTET: " + item.name
        );

    }

    renderInventory();

    updateHUD();

    saveGame();

}


function renderInventory() {

    const container =
        document.getElementById("inventoryItems");

    container.replaceChildren();

    for (const item of player.inventory) {

        const card = document.createElement("div");

        card.className = "itemCard";

        card.style.setProperty(
            "--itemColor",
            "#" + item.rarityColor
                .toString(16)
                .padStart(6, "0")
        );

        const name = document.createElement("div");

        name.className = "itemName";

        name.textContent = item.name;

        name.style.color =
            "#" + item.rarityColor
                .toString(16)
                .padStart(6, "0");

        const rarity = document.createElement("div");

        rarity.className = "small";

        rarity.textContent = item.rarity;

        const description =
            document.createElement("div");

        description.className = "itemDescription";

        if (item.type === "weapon") {

            description.textContent =
                "Waffe · Schaden: " + item.damage;

        } else {

            description.textContent =
                "Rüstung · Verteidigung: " +
                item.defense;

        }

        const button = document.createElement("button");

        button.textContent = "AUSRÜSTEN";

        if (
            player.weapon === item ||
            player.armor === item
        ) {

            button.textContent = "AUSGERÜSTET";

        }

        button.addEventListener(
            "click",
            () => equipItem(item)
        );

        card.append(
            name,
            rarity,
            description,
            button
        );

        container.appendChild(card);

    }

}


function toggleInventory() {

    inventoryOpen = !inventoryOpen;

    document.getElementById("inventory").style.display =
        inventoryOpen ? "flex" : "none";

    if (inventoryOpen) {

        renderInventory();

    }

}


/* -----------------------------------------------------
   GEGENSTÄNDE IN DER WELT
----------------------------------------------------- */

function createPickup(item, x, z) {

    const group = new THREE.Group();

    const color = item.rarityColor;

    const gem = new THREE.Mesh(
        new THREE.OctahedronGeometry(0.65),
        new THREE.MeshStandardMaterial({
            color,
            emissive: color,
            emissiveIntensity: 0.35,
            metalness: 0.5,
            roughness: 0.3
        })
    );

    gem.position.y = 1.1;

    gem.castShadow = true;

    group.add(gem);

    const ring = new THREE.Mesh(
        new THREE.TorusGeometry(
            0.85,
            0.055,
            6,
            18
        ),
        makeMaterial(color)
    );

    ring.rotation.x = Math.PI / 2;

    ring.position.y = 0.3;

    group.add(ring);

    group.position.set(
        x,
        groundAt(x, z),
        z
    );

    scene.add(group);

    pickups.push({
        group,
        item,
        x,
        z,
        age: Math.random() * 10
    });

}


function openChest(chest) {

    if (chest.opened) {

        showToast("Diese Truhe ist bereits geöffnet.");

        return;

    }

    chest.opened = true;

    chest.group.rotation.x = -0.25;

    const item = createItem(
        Math.random() < 0.7
            ? weaponDefinitions[
                Math.floor(
                    Math.random() *
                    weaponDefinitions.length
                )
            ]
            : armorDefinitions[
                Math.floor(
                    Math.random() *
                    armorDefinitions.length
                )
            ]
    );

    createPickup(
        item,
        chest.x + rand(-2, 2),
        chest.z + rand(-2, 2)
    );

    coins += Math.floor(rand(10, 40));

    playTone(600, 0.15);

    showToast(
        "TRUHE GEÖFFNET · " +
        item.rarity.toUpperCase() +
        " · " +
        item.name
    );

    updateHUD();

    saveGame();

}


function createChest(x, z) {

    const group = new THREE.Group();

    const y = groundAt(x, z);

    box(
        group,
        0, 0.45, 0,
        1.4, 0.9, 1,
        0x704626
    );

    box(
        group,
        0, 0.88, 0,
        1.45, 0.2, 1.05,
        0xd1a34e
    );

    box(
        group,
        0, 0.5, -0.53,
        0.17, 0.3, 0.08,
        0xf3d16d
    );

    group.position.set(x, y, z);

    scene.add(group);

    chests.push({
        group,
        x,
        z,
        opened: false
    });

}


/* Truhen an verschiedenen Orten */

createChest(18, -32);

createChest(-42, 58);

createChest(65, -50);

createChest(112, -105);

createChest(-110, 95);

createChest(175, 65);


/* -----------------------------------------------------
   SCHREIN
----------------------------------------------------- */

const shrine = new THREE.Group();

const shrineX = 0;

const shrineZ = -25;

const shrineY = groundAt(
    shrineX,
    shrineZ
);

box(
    shrine,
    0, 0.3, 0,
    3, 0.6, 3,
    0x8b8374
);

box(
    shrine,
    0, 1.4, -0.7,
    1.4, 2.2, 0.2,
    0x5b342a
);

box(
    shrine,
    0, 2.6, -0.7,
    2.5, 0.25, 0.65,
    0x9c2929
);

sphere(
    shrine,
    0, 1.6, 0.1,
    0.45, 0.6, 0.45,
    0xe7bd69
);

shrine.position.set(
    shrineX,
    shrineY,
    shrineZ
);

scene.add(shrine);


/* -----------------------------------------------------
   KLETTERSYSTEM
----------------------------------------------------- */

function beginClimbing() {

    if (climbing) return;

    climbing = true;

    player.vy = 0;

    player.grounded = false;

    showToast(
        "KLETTERN · W/A/D zum Aufsteigen"
    );

}


function finishClimbing() {

    climbing = false;

    player.x = 70;

    player.z = -87;

    player.y = groundAt(
        player.x,
        player.z
    );

    player.root.position.set(
        player.x,
        player.y,
        player.z
    );

    player.vy = 0;

    player.grounded = true;

    showToast(
        "GIPFEL ERREICHT"
    );

}


/* -----------------------------------------------------
   GEGNER
----------------------------------------------------- */

function createEnemy(
    x,
    z,
    boss = false
) {

    const root = new THREE.Group();

    const model = makeHumanoid(
        boss ? 0x641b25 : 0x3c2925,
        boss ? 0x18151a : 0x30251e,
        boss ? 0xe85c44 : 0xaab6bf
    );

    root.add(model.root);

    if (boss) {

        root.scale.setScalar(1.6);

    }

    root.position.set(
        x,
        groundAt(x, z),
        z
    );

    scene.add(root);

    const enemy = {

        root,

        model,

        x,

        z,

        boss,

        name: boss
            ? "ONI-SHOGUN"
            : "Bandit",

        health: boss ? 600 : 75,

        maxHealth: boss ? 600 : 75,

        damage: boss ? 22 : 9,

        speed: boss ? 2.5 : rand(1.4, 2.1),

        attackCooldown: rand(0.2, 1.1),

        alive: true,

        hitFlash: 0

    };

    enemies.push(enemy);

    return enemy;

}


/* Gegner im Wald */

for (let i = 0; i < 26; i++) {

    let x = rand(-260, 260);

    let z = rand(-260, 260);

    if (Math.hypot(x, z) < 35) {

        x += 65;

        z += 65;

    }

    if (
        distanceXZ(
            {x, z},
            {x: 95, z: -110}
        ) < 25
    ) {

        x -= 50;

    }

    createEnemy(x, z, false);

}


/* Endgegner */

const boss = createEnemy(
    95,
    -110,
    true
);


/* -----------------------------------------------------
   KAMPFSYSTEM
----------------------------------------------------- */

function attack(heavy = false) {

    if (!running || inventoryOpen || climbing) {
        return;
    }

    if (player.attackCooldown > 0) {
        return;
    }

    const staminaCost = heavy ? 24 : 10;

    if (player.stamina < staminaCost) {

        showToast("Nicht genug Ausdauer!");

        return;

    }

    player.stamina -= staminaCost;

    player.attackCooldown =
        (heavy ? 0.9 : 0.48) /
        Math.max(
            0.5,
            player.weapon.speed || 1
        );

    player.attackTimer = 0.28;

    player.heavyAttack = heavy;

    player.sword.rotation.z =
        heavy ? -1.4 : -0.8;

    playTone(
        heavy ? 130 : 240,
        heavy ? 0.15 : 0.07
    );

    let best = null;

    let bestDistance = Infinity;

    for (const enemy of enemies) {

        if (!enemy.alive) continue;

        if (enemy.boss && regularKills < 10) {
            continue;
        }

        const dx = enemy.root.position.x - player.x;

        const dz = enemy.root.position.z - player.z;

        const d = Math.hypot(dx, dz);

        const range = heavy ? 4.1 : 3.1;

        if (d > range || d < 0.01) {
            continue;
        }

        const fx = -Math.sin(
            player.root.rotation.y
        );

        const fz = -Math.cos(
            player.root.rotation.y
        );

        const dot =
            fx * dx / d +
            fz * dz / d;

        if (dot < -0.15) continue;

        if (d < bestDistance) {

            best = enemy;

            bestDistance = d;

        }

    }

    if (!best) return;

    const damage = Math.round(
        player.weapon.damage *
        (heavy ? 1.75 : 1)
    );

    best.health -= damage;

    best.hitFlash = 0.12;

    playTone(100, 0.12);

    showToast(
        "TREFFER · " + damage + " SCHADEN"
    , 900);

    if (best.health <= 0) {

        killEnemy(best);

    }

}


function killEnemy(enemy) {

    if (!enemy.alive) return;

    enemy.alive = false;

    scene.remove(enemy.root);

    coins += enemy.boss
        ? 500
        : Math.floor(rand(5, 22));

    if (!enemy.boss) {

        regularKills++;

        if (Math.random() < 0.65) {

            const item =
                Math.random() < 0.72
                    ? createItem(
                        weaponDefinitions[
                            Math.floor(
                                Math.random() *
                                weaponDefinitions.length
                            )
                        ]
                    )
                    : createItem(
                        armorDefinitions[
                            Math.floor(
                                Math.random() *
                                armorDefinitions.length
                            )
                        ]
                    );

            createPickup(
                item,
                enemy.root.position.x,
                enemy.root.position.z
            );

        }

        showToast(
            "BANDIT BESIEGT · +" + coins + " MON"
        );

    } else {

        showToast(
            "DER ONI-SHOGUN IST GEFALLEN!"
        );

        document.getElementById(
            "startScreen"
        ).style.display = "flex";

        document.querySelector(
            "#startScreen .menuTitle"
        ).textContent = "SIEG";

        document.querySelector(
            "#startScreen .subtitle"
        ).textContent = "DER SHOGUN IST BESIEGT";

        document.querySelector(
            "#startScreen .menuDescription"
        ).textContent =
            "Du hast die Festung erobert. " +
            "Deine Reise ist noch nicht zu Ende.";

        document.getElementById(
            "startButton"
        ).textContent = "WEITER ERKUNDEN";

        document.getElementById(
            "loadButton"
        ).textContent = "SPIELSTAND SPEICHERN";

    }

    updateHUD();

    saveGame();

}


/* -----------------------------------------------------
   SCHADEN UND TOD
----------------------------------------------------- */

function damagePlayer(damage) {

    if (player.invulnerable > 0) {
        return;
    }

    const defense =
        player.armor?.defense || 0;

    const reducedDamage = Math.max(
        1,
        damage * (1 - defense * 0.025)
    );

    player.health -= reducedDamage;

    player.invulnerable = 0.55;

    playTone(75, 0.15);

    if (player.health <= 0) {

        player.health = player.maxHealth;

        player.stamina = player.maxStamina;

        player.x = 0;

        player.z = -25;

        player.y = groundAt(
            player.x,
            player.z
        );

        player.root.position.set(
            player.x,
            player.y,
            player.z
        );

        player.vy = 0;

        player.invulnerable = 2;

        coins = Math.max(
            0,
            coins - Math.floor(coins * 0.1)
        );

        showToast(
            "BESIEGT · DU ERWACHST AM SCHREIN"
        );

    }

    updateHUD();

}


/* -----------------------------------------------------
   INTERAKTIONEN
----------------------------------------------------- */

function interact() {

    if (!running || inventoryOpen) return;

    /* Kletterwand */

    if (
        distanceXZ(
            player.root.position,
            {x: 70, z: -77}
        ) < 5
        &&
        !climbing
    ) {

        beginClimbing();

        return;

    }

    /* Schreine */

    if (
        distanceXZ(
            player.root.position,
            {x: shrineX, z: shrineZ}
        ) < 4
    ) {

        player.health = player.maxHealth;

        player.stamina = player.maxStamina;

        player.x = shrineX;

        player.z = shrineZ;

        showToast(
            "SCHREIN · LEBEN UND AUSDAUER WIEDERHERGESTELLT"
        );

        saveGame();

        updateHUD();

        return;

    }

    /* Gegenstände */

    let nearestPickup = null;

    let nearestDistance = 3.5;

    for (const pickup of pickups) {

        const d = distanceXZ(
            player.root.position,
            pickup.group.position
        );

        if (d < nearestDistance) {

            nearestDistance = d;

            nearestPickup = pickup;

        }

    }

    if (nearestPickup) {

        player.inventory.push(
            nearestPickup.item
        );

        scene.remove(
            nearestPickup.group
        );

        pickups.splice(
            pickups.indexOf(nearestPickup),
            1
        );

        showToast(
            "GEFUNDEN: " +
            nearestPickup.item.name +
            " · " +
            nearestPickup.item.rarity
        );

        playTone(500, 0.12);

        renderInventory();

        updateHUD();

        saveGame();

        return;

    }

    /* Truhen */

    let nearestChest = null;

    let chestDistance = 3.5;

    for (const chest of chests) {

        const d = distanceXZ(
            player.root.position,
            chest.group.position
        );

        if (d < chestDistance) {

            chestDistance = d;

            nearestChest = chest;

        }

    }

    if (nearestChest) {

        openChest(nearestChest);

        return;

    }

    showToast(
        "Nichts in Reichweite."
    );

}


/* -----------------------------------------------------
   KOLLISIONEN
----------------------------------------------------- */

function canMoveTo(x, z) {

    if (
        x < -390 ||
        x > 390 ||
        z < -390 ||
        z > 390
    ) {

        return false;

    }

    for (const collider of worldColliders) {

        if (collider.rect) {

            if (
                Math.abs(x - collider.x) <
                    collider.halfX + 0.5 &&
                Math.abs(z - collider.z) <
                    collider.halfZ + 0.5
            ) {

                return false;

            }

        } else {

            if (
                Math.hypot(
                    x - collider.x,
                    z - collider.z
                ) < collider.r + 0.5
            ) {

                return false;

            }

        }

    }

    return true;

}


/* -----------------------------------------------------
   STEUERUNG
----------------------------------------------------- */

let moveX = 0;

let moveY = 0;

let touchLookId = null;

let lastLookX = 0;

let lastLookY = 0;

let rightMouse = false;

let touchLookActive = false;


window.addEventListener(
    "keydown",
    event => {

        keys[event.code] = true;

        if (
            event.code === "Space" ||
            event.code.startsWith("Arrow")
        ) {

            event.preventDefault();

        }

        if (event.repeat) return;

        if (event.code === "KeyI") {

            toggleInventory();

        }

        if (event.code === "KeyE") {

            if (climbing) {

                climbing = false;

                player.grounded = false;

            } else {

                interact();

            }

        }

        if (
            event.code === "Space" &&
            !inventoryOpen
        ) {

            if (climbing) {

                climbing = false;

                player.grounded = false;

                player.vy = 4;

            } else if (player.grounded) {

                if (player.stamina >= 8) {

                    player.vy = 7;

                    player.grounded = false;

                    player.stamina -= 8;

                }

            }

        }

        if (
            event.code === "ShiftLeft" ||
            event.code === "ShiftRight"
        ) {

            dodge();

        }

    }
);


window.addEventListener(
    "keyup",
    event => {

        keys[event.code] = false;

    }
);


window.addEventListener(
    "contextmenu",
    event => event.preventDefault()
);


renderer.domElement.addEventListener(
    "pointerdown",
    event => {

        if (!running || inventoryOpen) return;

        if (event.pointerType === "mouse") {

            if (event.button === 0) {

                attack(false);

            }

            if (event.button === 2) {

                rightMouse = true;

                lastLookX = event.clientX;

                lastLookY = event.clientY;

            }

        } else {

            if (event.clientX > innerWidth * 0.38) {

                touchLookActive = true;

                touchLookId = event.pointerId;

                lastLookX = event.clientX;

                lastLookY = event.clientY;

                renderer.domElement.setPointerCapture(
                    event.pointerId
                );

            }

        }

    }
);


renderer.domElement.addEventListener(
    "pointermove",
    event => {

        if (!running || inventoryOpen) return;

        if (
            event.pointerType === "mouse" &&
            rightMouse
        ) {

            cameraYaw -=
                (event.clientX - lastLookX) * 0.006;

            cameraPitch = clamp(
                cameraPitch +
                (event.clientY - lastLookY) * 0.004,
                -0.05,
                0.9
            );

        }

        if (
            event.pointerType !== "mouse" &&
            touchLookActive &&
            event.pointerId === touchLookId
        ) {

            cameraYaw -=
                (event.clientX - lastLookX) * 0.007;

            cameraPitch = clamp(
                cameraPitch +
                (event.clientY - lastLookY) * 0.005,
                -0.05,
                0.9
            );

        }

        lastLookX = event.clientX;

        lastLookY = event.clientY;

    }
);


window.addEventListener(
    "pointerup",
    event => {

        if (event.pointerType === "mouse") {

            rightMouse = false;

        } else if (
            event.pointerId === touchLookId
        ) {

            touchLookActive = false;

            touchLookId = null;

        }

    }
);


/* JOYSTICK */

const joystick =
    document.getElementById("joystick");

const stick =
    document.getElementById("stick");

let joyPointer = null;

function updateJoystick(event) {

    const rect = joystick.getBoundingClientRect();

    const cx = rect.left + rect.width / 2;

    const cy = rect.top + rect.height / 2;

    let dx = event.clientX - cx;

    let dy = event.clientY - cy;

    const max = 40;

    const length = Math.hypot(dx, dy);

    if (length > max) {

        dx = dx / length * max;

        dy = dy / length * max;

    }

    stick.style.left =
        (32 + dx) + "px";

    stick.style.top =
        (32 + dy) + "px";

    moveX = dx / max;

    moveY = -dy / max;

}

joystick.addEventListener(
    "pointerdown",
    event => {

        event.preventDefault();

        joyPointer = event.pointerId;

        joystick.setPointerCapture(
            event.pointerId
        );

        updateJoystick(event);

    }
);


joystick.addEventListener(
    "pointermove",
    event => {

        if (event.pointerId !== joyPointer) {
            return;
        }

        updateJoystick(event);

    }
);


function resetJoystick() {

    joyPointer = null;

    moveX = 0;

    moveY = 0;

    stick.style.left = "32px";

    stick.style.top = "32px";

}

joystick.addEventListener(
    "pointerup",
    resetJoystick
);

joystick.addEventListener(
    "pointercancel",
    resetJoystick
);


/* MOBILE BUTTONS */

document.getElementById(
    "btnAttack"
).addEventListener(
    "pointerdown",
    event => {

        event.preventDefault();

        attack(false);

    }
);


document.getElementById(
    "btnHeavy"
).addEventListener(
    "pointerdown",
    event => {

        event.preventDefault();

        attack(true);

    }
);


document.getElementById(
    "btnJump"
).addEventListener(
    "pointerdown",
    event => {

        event.preventDefault();

        if (player.grounded && player.stamina >= 8) {

            player.vy = 7;

            player.grounded = false;

            player.stamina -= 8;

        }

    }
);


document.getElementById(
    "btnDodge"
).addEventListener(
    "pointerdown",
    event => {

        event.preventDefault();

        dodge();

    }
);


document.getElementById(
    "btnInteract"
).addEventListener(
    "pointerdown",
    event => {

        event.preventDefault();

        interact();

    }
);


document.getElementById(
    "btnInventory"
).addEventListener(
    "pointerdown",
    event => {

        event.preventDefault();

        toggleInventory();

    }
);


/* AUSWEICHEN */

function dodge() {

    if (!running || inventoryOpen || climbing) {
        return;
    }

    if (
        player.stamina < 25 ||
        player.dodgeTimer > 0
    ) {

        return;

    }

    player.stamina -= 25;

    player.dodgeTimer = 0.38;

    player.invulnerable = 0.45;

    showToast("AUSWEICHEN");

}


/* -----------------------------------------------------
   GEGNER-KI
----------------------------------------------------- */

function updateEnemies(dt) {

    for (const enemy of enemies) {

        if (!enemy.alive) continue;

        enemy.attackCooldown -= dt;

        enemy.hitFlash = Math.max(
            0,
            enemy.hitFlash - dt
        );

        if (
            enemy.boss &&
            regularKills < 10
        ) {

            continue;

        }

        const dx =
            player.x - enemy.root.position.x;

        const dz =
            player.z - enemy.root.position.z;

        const d = Math.hypot(dx, dz);

        if (d > 30) continue;

        if (d > 2.15) {

            const nx = dx / d;

            const nz = dz / d;

            const speed =
                enemy.speed * dt;

            const nextX =
                enemy.root.position.x + nx * speed;

            const nextZ =
                enemy.root.position.z + nz * speed;

            enemy.root.position.x = nextX;

            enemy.root.position.z = nextZ;

            enemy.root.position.y =
                groundAt(nextX, nextZ);

            enemy.root.rotation.y =
                Math.atan2(-dx, -dz);

        } else if (
            enemy.attackCooldown <= 0
        ) {

            enemy.attackCooldown =
                enemy.boss ? 1.2 : 1.45;

            damagePlayer(enemy.damage);

        }

    }

}


/* -----------------------------------------------------
   QUESTS
----------------------------------------------------- */

function updateQuest() {

    const quest =
        document.getElementById("questText");

    if (regularKills < 10) {

        quest.innerHTML =
            "Erkunde die Provinz.<br>" +
            "Besiege Banditen.<br>" +
            "<b>Banditen: " +
            regularKills +
            "/10</b><br>" +
            "<span style='color:#aaa'>" +
            "Besiege 10 Banditen, um den " +
            "Oni-Shogun herauszufordern." +
            "</span>";

    } else {

        quest.innerHTML =
            "<b style='color:#ff6464'>" +
            "DER ONI-SHOGUN</b><br>" +
            "Finde die Festung im Osten.<br>" +
            "Besiege den Endgegner!";

    }

}


/* -----------------------------------------------------
   HUD
----------------------------------------------------- */

function updateHUD() {

    document.getElementById(
        "healthBar"
    ).style.width =
        (
            player.health /
            player.maxHealth * 100
        ) + "%";

    document.getElementById(
        "staminaBar"
    ).style.width =
        (
            player.stamina /
            player.maxStamina * 100
        ) + "%";

    document.getElementById(
        "stats"
    ).textContent =
        "❤️ " +
        Math.ceil(player.health) +
        "/" +
        player.maxHealth +
        " · Banditen " +
        regularKills;

    document.getElementById(
        "equipment"
    ).textContent =
        "⚔️ " +
        player.weapon.name +
        " · " +
        player.weapon.rarity;

    document.getElementById(
        "coins"
    ).textContent =
        "🪙 " + coins + " Mon";

    updateQuest();

}


/* -----------------------------------------------------
   NÄCHSTE INTERAKTION FINDEN
----------------------------------------------------- */

function findInteractable() {

    let best = null;

    let bestDistance = 4;

    for (const pickup of pickups) {

        const d = distanceXZ(
            player.root.position,
            pickup.group.position
        );

        if (d < bestDistance) {

            best = {
                type: "pickup",
                object: pickup,
                text:
                    "E · AUFHEBEN: " +
                    pickup.item.name
            };

            bestDistance = d;

        }

    }

    for (const chest of chests) {

        const d = distanceXZ(
            player.root.position,
            chest.group.position
        );

        if (
            d < bestDistance &&
            !chest.opened
        ) {

            best = {
                type: "chest",
                object: chest,
                text: "E · TRUHE ÖFFNEN"
            };

            bestDistance = d;

        }

    }

    const shrineDistance = distanceXZ(
        player.root.position,
        {x: shrineX, z: shrineZ}
    );

    if (
        shrineDistance < bestDistance
    ) {

        best = {
            type: "shrine",
            text: "E · AM SCHREIN HEILEN"
        };

        bestDistance = shrineDistance;

    }

    const cliffDistance = distanceXZ(
        player.root.position,
        {x: 70, z: -77}
    );

    if (
        cliffDistance < 5 &&
        cliffDistance < bestDistance &&
        !climbing
    ) {

        best = {
            type: "climb",
            text: "E · KLETTERWAND ERKLIMMEN"
        };

    }

    currentInteractable = best;

    const prompt =
        document.getElementById("prompt");

    if (best && running && !inventoryOpen) {

        prompt.textContent = best.text;

        prompt.style.display = "block";

    } else {

        prompt.style.display = "none";

    }

}


/* -----------------------------------------------------
   ZIELGEGNER ANZEIGEN
----------------------------------------------------- */

function updateTarget() {

    let nearest = null;

    let nearestDistance = 22;

    for (const enemy of enemies) {

        if (!enemy.alive) continue;

        const d = distanceXZ(
            player.root.position,
            enemy.root.position
        );

        if (d < nearestDistance) {

            nearest = enemy;

            nearestDistance = d;

        }

    }

    targetEnemy = nearest;

    const panel =
        document.getElementById("targetPanel");

    if (nearest) {

        panel.style.display = "block";

        document.getElementById(
            "targetName"
        ).textContent = nearest.name;

        document.getElementById(
            "targetHealth"
        ).style.width =
            (
                nearest.health /
                nearest.maxHealth * 100
            ) + "%";

    } else {

        panel.style.display = "none";

    }

}


/* -----------------------------------------------------
   SPEICHERN UND LADEN
----------------------------------------------------- */

function saveGame() {

    try {

        const data = {

            x: player.x,

            z: player.z,

            health: player.health,

            stamina: player.stamina,

            coins,

            regularKills,

            inventory: player.inventory,

            weaponName: player.weapon.name,

            armorName: player.armor.name

        };

        localStorage.setItem(
            "ronin_save_v1",
            JSON.stringify(data)
        );

    } catch (error) {

        showToast(
            "Speichern nicht möglich."
        );

    }

}


function loadGame() {

    try {

        const raw = localStorage.getItem(
            "ronin_save_v1"
        );

        if (!raw) {

            showToast(
                "Kein Spielstand gefunden."
            );

            return false;

        }

        const data = JSON.parse(raw);

        player.x = Number(data.x) || 0;

        player.z = Number(data.z) || 0;

        player.y = groundAt(
            player.x,
            player.z
        );

        player.health = clamp(
            Number(data.health) || 100,
            1,
            player.maxHealth
        );

        player.stamina = clamp(
            Number(data.stamina) || 100,
            0,
            player.maxStamina
        );

        coins = Math.max(
            0,
            Number(data.coins) || 0
        );

        regularKills = Math.max(
            0,
            Number(data.regularKills) || 0
        );

        if (
            Array.isArray(data.inventory) &&
            data.inventory.length
        ) {

            player.inventory = data.inventory;

            player.weapon =
                player.inventory.find(
                    item =>
                        item.name === data.weaponName &&
                        item.type === "weapon"
                ) ||
                player.inventory.find(
                    item => item.type === "weapon"
                );

            player.armor =
                player.inventory.find(
                    item =>
                        item.name === data.armorName &&
                        item.type === "armor"
                ) ||
                player.inventory.find(
                    item => item.type === "armor"
                );

            player.armorMat.color.setHex(
                player.armor.color
            );

            player.bladeMat.color.setHex(
                player.weapon.rarityColor
            );

        }

        player.root.position.set(
            player.x,
            player.y,
            player.z
        );

        updateHUD();

        renderInventory();

        showToast("SPIELSTAND GELADEN");

        return true;

    } catch (error) {

        showToast(
            "Spielstand konnte nicht geladen werden."
        );

        return false;

    }

}


/* -----------------------------------------------------
   SPIEL-UPDATE
----------------------------------------------------- */

function updatePlayer(dt) {

    player.attackCooldown = Math.max(
        0,
        player.attackCooldown - dt
    );

    player.attackTimer = Math.max(
        0,
        player.attackTimer - dt
    );

    player.dodgeTimer = Math.max(
        0,
        player.dodgeTimer - dt
    );

    player.invulnerable = Math.max(
        0,
        player.invulnerable - dt
    );

    player.stamina = Math.min(
        player.maxStamina,
        player.stamina + 13 * dt
    );

    /* Schwertanimation */

    if (player.attackTimer > 0) {

        const progress =
            player.attackTimer / 0.28;

        player.sword.rotation.z =
            -0.8 + (1 - progress) * 2.2;

    } else {

        player.sword.rotation.z = 0.3;

    }

    /* Bewegungseingabe */

    let ix =
        (keys.KeyD ? 1 : 0) -
        (keys.KeyA ? 1 : 0);

    let iy =
        (keys.KeyW ? 1 : 0) -
        (keys.KeyS ? 1 : 0);

    ix += moveX;

    iy += moveY;

    const inputLength = Math.hypot(ix, iy);

    if (inputLength > 1) {

        ix /= inputLength;

        iy /= inputLength;

    }

    const forwardX = Math.sin(cameraYaw);

    const forwardZ = -Math.cos(cameraYaw);

    const rightX = Math.cos(cameraYaw);

    const rightZ = Math.sin(cameraYaw);

    let dx =
        forwardX * iy +
        rightX * ix;

    let dz =
        forwardZ * iy +
        rightZ * ix;

    const length = Math.hypot(dx, dz);

    if (length > 0.001) {

        dx /= length;

        dz /= length;

    }

    /* Klettern */

    if (climbing) {

        const wallBase =
            terrainHeight(70, -80);

        if (iy > 0.1) {

            player.y += 4 * dt;

        }

        if (iy < -0.1) {

            player.y -= 3 * dt;

        }

        player.x += dx * dt * 2;

        player.x = clamp(
            player.x,
            59,
            81
        );

        if (
            player.y >= wallBase + 15.5
        ) {

            finishClimbing();

        }

        if (
            player.y <= wallBase
        ) {

            climbing = false;

            player.y = groundAt(
                player.x,
                player.z
            );

        }

        player.root.position.set(
            player.x,
            player.y,
            player.z
        );

        return;

    }

    /* Ausweichen */

    let speed = 5;

    if (player.dodgeTimer > 0) {

        speed = 12;

        if (length > 0.01) {

            player.dodgeX = dx;

            player.dodgeZ = dz;

        }

        dx = player.dodgeX;

        dz = player.dodgeZ;

    }

    if (length > 0.01) {

        const nextX =
            player.x + dx * speed * dt;

        const nextZ =
            player.z + dz * speed * dt;

        if (canMoveTo(nextX, player.z)) {

            player.x = nextX;

        }

        if (canMoveTo(player.x, nextZ)) {

            player.z = nextZ;

        }

        player.root.rotation.y =
            Math.atan2(-dx, -dz);

    }

    /* Schwerkraft und Sprung */

    player.vy -= 18 * dt;

    player.y += player.vy * dt;

    const ground = groundAt(
        player.x,
        player.z
    );

    if (player.y <= ground) {

        player.y = ground;

        player.vy = 0;

        player.grounded = true;

    } else {

        player.grounded = false;

    }

    player.root.position.set(
        player.x,
        player.y,
        player.z
    );

}


function updatePickups(dt) {

    for (const pickup of pickups) {

        pickup.age += dt;

        pickup.group.position.y =
            groundAt(
                pickup.x,
                pickup.z
            ) +
            0.15 +
            Math.sin(pickup.age * 2) * 0.15;

        pickup.group.rotation.y += dt * 0.7;

    }

}


/* -----------------------------------------------------
   KAMERA
----------------------------------------------------- */

function updateCamera() {

    const distance = 7.5;

    const horizontal =
        distance * Math.cos(cameraPitch);

    camera.position.set(
        player.x +
            Math.sin(cameraYaw) * horizontal,

        player.y +
            2.0 +
            distance * Math.sin(cameraPitch),

        player.z +
            Math.cos(cameraYaw) * horizontal
    );

    camera.lookAt(
        player.x,
        player.y + 1.25,
        player.z
    );

}


/* -----------------------------------------------------
   START UND MENÜ
----------------------------------------------------- */

function startGame(load = false) {

    if (load) {

        loadGame();

    } else {

        player.x = 0;

        player.z = 0;

        player.y = groundAt(0, 0);

        player.root.position.set(
            player.x,
            player.y,
            player.z
        );

    }

    running = true;

    document.getElementById(
        "startScreen"
    ).style.display = "none";

    player.root.position.set(
        player.x,
        player.y,
        player.z
    );

    updateHUD();

    saveGame();

    playTone(400, 0.1);

}


/* -----------------------------------------------------
   BUTTONS
----------------------------------------------------- */

document.getElementById(
    "startButton"
).addEventListener(
    "click",
    () => {

        startGame(false);

    }
);


document.getElementById(
    "loadButton"
).addEventListener(
    "click",
    () => {

        startGame(true);

    }
);


document.getElementById(
    "inventoryClose"
).addEventListener(
    "click",
    () => {

        inventoryOpen = false;

        document.getElementById(
            "inventory"
        ).style.display = "none";

    }
);


/* -----------------------------------------------------
   HAUPTSCHLEIFE
----------------------------------------------------- */

let hudTimer = 0;

function animate() {

    requestAnimationFrame(animate);

    const dt = Math.min(
        clock.getDelta(),
        0.04
    );

    if (running && !inventoryOpen) {

        updatePlayer(dt);

        updateEnemies(dt);

        updatePickups(dt);

        updateCamera();

        findInteractable();

        updateTarget();

        hudTimer += dt;

        if (hudTimer > 0.2) {

            updateHUD();

            hudTimer = 0;

        }

        if (toastTimer > 0) {

            toastTimer -= dt;

            if (toastTimer <= 0) {

                document.getElementById(
                    "toast"
                ).style.display = "none";

            }

        }

    } else {

        updateCamera();

    }

    renderer.render(
        scene,
        camera
    );

}


/* -----------------------------------------------------
   FENSTERGRÖSSE
----------------------------------------------------- */

window.addEventListener(
    "resize",
    () => {

        camera.aspect =
            innerWidth / innerHeight;

        camera.updateProjectionMatrix();

        renderer.setSize(
            innerWidth,
            innerHeight
        );

        renderer.setPixelRatio(
            Math.min(devicePixelRatio || 1, 1.5)
        );

    }
);


/* -----------------------------------------------------
   INITIALISIERUNG
----------------------------------------------------- */

player.y = groundAt(0, 0);

player.root.position.set(
    player.x,
    player.y,
    player.z
);

updateHUD();

renderInventory();

updateCamera();

animate();

</script>

</body>
</html>