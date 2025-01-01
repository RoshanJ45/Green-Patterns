<script>
console.clear();

let canvas, canvasCtx;
let canvasSize = [0, 0], scale = 1;
let state;

requestAnimationFrame(main);

function main() {
  canvas = document.createElement('canvas');
  document.body.appendChild(canvas);
  document.body.style.margin = '0';
  canvas.style.display = 'block';
  canvasCtx = canvas.getContext('2d');
  checkResizeAndInit();
  state = createState();
  requestAnimationFrame(mainLoop);
}

function mainLoop() {
  tick();
  requestAnimationFrame(mainLoop);
}

function createState() {
  const state = {
    time: 0,
    timeDelta: 1 / 60,
    pointer: {
      pos: [canvasSize[0] / 2, canvasSize[1] / 2 + 128],
      vel: [640, 8],
    },
  };
  return state;
}

function tick() {
  checkResizeAndInit();
  canvasCtx.fillStyle = `rgba(0, 0, 0, ${1 / 32})`;
  canvasCtx.fillRect(0, 0, canvasSize[0], canvasSize[1]);
  doIt();
  state.time += state.timeDelta;
}

function checkResizeAndInit() {
  if (window.innerWidth === canvasSize[0] && window.innerHeight === canvasSize[1]) return;
  canvasSize[0] = canvas.width = window.innerWidth;
  canvasSize[1] = canvas.height = window.innerHeight;
}

function doIt() {
  const { pointer, timeDelta } = state;
  const oldPos = [pointer.pos[0], pointer.pos[1]];
  let x = pointer.pos[0] - canvasSize[0] / 2;
  let y = pointer.pos[1] - canvasSize[1] / 2;
  const l = Math.hypot(x, y);
  const m = 1 / l * 40000000 / Math.max(32, l) ** 2;
  pointer.vel[0] -= x * m * timeDelta;
  pointer.vel[1] -= y * m * timeDelta;
  pointer.vel[Math.random() < 0.5 ? 0 : 1] += 100 + (Math.random() * 2 - 1);
  const velLimit = 2048;
  if (Math.hypot(pointer.vel[0], pointer.vel[1]) > velLimit) {
    pointer.vel[0] *= 0.95;
    pointer.vel[1] *= 0.95;
  }
  pointer.pos[0] += pointer.vel[0] * timeDelta;
  pointer.pos[1] += pointer.vel[1] * timeDelta;
  const min = Math.min(canvasSize[0], canvasSize[1]);
  const left = canvasSize[0] / 2 - min / 2;
  const right = canvasSize[0] / 2 + min / 2;
  const top = canvasSize[1] / 2 - min / 2;
  const bottom = canvasSize[1] / 2 + min / 2;
  if (pointer.pos[0] < left) {
    pointer.pos[0] = left + (left - pointer.pos[0]);
    pointer.vel[0] *= -1;
  } else if (pointer.pos[0] > right) {
    pointer.pos[0] = right - (pointer.pos[0] - right);
    pointer.vel[0] *= -1;
  }
  if (pointer.pos[1] < top) {
    pointer.pos[1] = top + (top - pointer.pos[1]);
    pointer.vel[1] *= -1;
  } else if (pointer.pos[1] > bottom) {
    pointer.pos[1] = bottom - (pointer.pos[1] - bottom);
    pointer.vel[1] *= -1;
  }
  const pos = [pointer.pos[0], pointer.pos[1]];
  canvasCtx.beginPath();
  const rVec = [0, 0];
  const t = [0, 0];
  const origin = [canvasSize[0] / 2, canvasSize[1] / 2];
  const n = 15;
  for (let i = 0; i < n; ++i) {
    const a = Math.PI / 180 * 55 * i;
    const s = 0.2 + 0.8 / n * i;
    rVec[0] = Math.cos(a);
    rVec[1] = Math.sin(a);
    rotateByVector(t, oldPos, rVec, origin, s);
    canvasCtx.moveTo(t[0], t[1]);
    rotateByVector(t, pos, rVec, origin, s);
    canvasCtx.lineTo(t[0], t[1]);
  }
canvasCtx.strokeStyle = '#00ff00'; 
canvasCtx.stroke(); 
}
function rotateByVector(out, a, v, origin, s){
  const rx = v[0];
  const ry = v[1];
  const x = a[0] - origin[0];
  const y = a[1] - origin[1];
  
  out[0] = origin[0] + (x * rx - y * ry) * s;
  out[1] = origin[1] + (x * ry + y * rx) * s; 
  return out;
}
 </script>