<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>Nube Naranja Roja 2D Diagonal</title>
<style>
    body {
        margin: 0;
        background: black;
        overflow: hidden;
    }
    canvas {
        display: block;
    }
</style>
</head>
<body>

<canvas id="canvas"></canvas>

<script>
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');

canvas.width = innerWidth;
canvas.height = innerHeight;

let centerX = canvas.width / 2;
let centerY = canvas.height / 2;

// Núcleo y nube
const coreRadius = 40;
const baseBorder = 50;
const extraBorder = 30;
const cloudRadius = coreRadius + baseBorder + extraBorder;

// ===== ESFERAS BLANCAS =====
const spheres = [];
for (let i = 0; i < 150; i++) {
    spheres.push({
        angle: Math.random() * Math.PI * 2,
        radius: Math.random() * (canvas.width / 2),
        speed: 0.002 + Math.random() * 0.003,
        size: 3.5
    });
}

// ===== PALABRAS =====
const wordSets = [
    ["GUAPA,","HERMOSA,","GENTIL,","AMABLE,","DIVERTIDA,","INTELIGENTE,","MARAVILLOSA","Y LOCA",],
    ["AMO","PASAR","TIEMPO","CON","USTED","QUE","NI TE","IMAGINAS"],
    ["TE","VOLVISTE","MUY","ESPECIAL","PARA","MI","NO LO","DUDES"],
    ["CADA","DIA","ME","SORPRENDES","MAS","Y","ME","ENCANTA"],
    ["GRACIASE","EN","SERIO","GRACIAS","POR","LLEGAR","A","MI VIDA"],
    ["TE QUIERO","MUCHO","Y SIEMPRE","VOY","A","ESTAR","PARA","TI"],
];
let currentSet = 0;

// ===== ESFERAS ROJAS =====
const initialRadius = 280;
const angles = [
    -Math.PI/2, -Math.PI/4, 0, Math.PI/4,
     Math.PI/2, Math.PI*3/4, Math.PI, -Math.PI*3/4
];

const redSpheres = angles.map((a, i) => ({
    angle: a,
    radius: initialRadius,
    speed: 0.010,
    shrink: 0.30,
    index: i
}));

const diagonalAngle = Math.PI / 6;

// ===== CONTROL DE UNION =====
let isMerging = false;
let mergeTimer = 0;
const MERGE_DURATION = 160 // ~1 segundo

function draw() {

    ctx.fillStyle = 'rgba(0,0,1,0.1)';
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    // ===== ESCENA ROTADA =====
    ctx.save();
    ctx.translate(centerX, centerY);
    ctx.rotate(diagonalAngle);

    // Esferas blancas
    ctx.fillStyle = 'white';
    spheres.forEach(s => {
        ctx.beginPath();
        ctx.arc(Math.cos(s.angle)*s.radius, Math.sin(s.angle)*s.radius, s.size, 0, Math.PI*2);
        ctx.fill();
        s.angle += s.speed;
    });

    // Esferas rojas
    let reachedCenter = true;

    if (!isMerging) {
        ctx.fillStyle = 'red';
        redSpheres.forEach(r => {
            ctx.beginPath();
            ctx.arc(Math.cos(r.angle)*r.radius, Math.sin(r.angle)*r.radius, 3.5, 0, Math.PI*2);
            ctx.fill();

            r.angle += r.speed;
            if (r.radius > coreRadius + 15) {
                r.radius -= r.shrink;
                reachedCenter = false;
            }
        });

        if (reachedCenter) {
            isMerging = true;
            mergeTimer = 0;
        }
    }

    ctx.restore();

    // ===== TEXTO =====
    ctx.save();
    ctx.fillStyle = '#ffffff';
    ctx.font = '26px Arial';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';

    if (isMerging) {
        const phrase = wordSets[currentSet].join(" ");
        ctx.fillText(phrase, centerX, centerY + 120);

        mergeTimer++;
        if (mergeTimer > MERGE_DURATION) {
            isMerging = false;
            currentSet = (currentSet + 1) % wordSets.length;
            redSpheres.forEach(r => r.radius = initialRadius);
        }
    } else {
        redSpheres.forEach(r => {
            const xL = Math.cos(r.angle) * r.radius;
            const yL = Math.sin(r.angle) * r.radius;

            const x = centerX + (xL*Math.cos(diagonalAngle) - yL*Math.sin(diagonalAngle));
            const y = centerY + (xL*Math.sin(diagonalAngle) + yL*Math.cos(diagonalAngle));

            ctx.fillText(wordSets[currentSet][r.index], x, y - 6);
        });
    }

    ctx.restore();

    // ===== NUBE =====
    ctx.save();
    ctx.translate(centerX, centerY);
    ctx.rotate(diagonalAngle);

    const g1 = ctx.createRadialGradient(0,0,coreRadius,0,0,coreRadius+baseBorder);
    g1.addColorStop(0,'rgba(255,140,0,1)');
    g1.addColorStop(0.4,'rgba(255,60,0,0.9)');
    g1.addColorStop(0.7,'rgba(255,0,0,0.7)');
    g1.addColorStop(1,'transparent');
    ctx.fillStyle = g1;
    ctx.beginPath();
    ctx.arc(0,0,coreRadius+baseBorder,0,Math.PI*2);
    ctx.fill();

    const g2 = ctx.createRadialGradient(0,0,coreRadius+baseBorder,0,0,cloudRadius);
    g2.addColorStop(0,'rgba(255,100,0,0.7)');
    g2.addColorStop(0.5,'rgba(255,40,0,0.5)');
    g2.addColorStop(1,'transparent');
    ctx.fillStyle = g2;
    ctx.beginPath();
    ctx.arc(0,0,cloudRadius,0,Math.PI*2);
    ctx.fill();

    ctx.fillStyle = 'black';
    ctx.beginPath();
    ctx.arc(0,0,coreRadius,0,Math.PI*2);
    ctx.fill();

    ctx.restore();

    requestAnimationFrame(draw);
}

draw();

addEventListener('resize', () => {
    canvas.width = innerWidth;
    canvas.height = innerHeight;
    centerX = canvas.width / 2;
    centerY = canvas.height / 2;
});
</script>

</body>
</html>
