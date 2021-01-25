<script>
var audioCtx = new (window.AudioContext || window.webkitAudioContext)();
var analyser = audioCtx.createAnalyser();

let canvasCtx1 = canvas1.getContext("2d");
let canvasCtx2 = canvas2.getContext("2d");
let WIDTH = canvas1.width = canvas2.width = 500;
let HEIGHT = canvas1.height = canvas2.height = 200;

let stream;
startButton.onclick = async () => {
  if (stream) return;
  try {
    stream = await navigator.mediaDevices.getUserMedia({
      audio: true
    });
    audioCtx.createMediaStreamSource(stream).connect(analyser);
    draw();
  } catch (err) {
    errorMessage.textContent = err.toString();
  }
};

analyser.fftSize = 32768;
var bufferLength = analyser.frequencyBinCount;
var dataArray = new Uint8Array(bufferLength);

// draw an oscilloscope of the current audio source

function draw() {
  if (stream) requestAnimationFrame(draw);
  draw1();
  draw2();
}

function draw1() {
  analyser.getByteTimeDomainData(dataArray);
  canvasCtx1.fillStyle = "rgb(200, 200, 200)";
  canvasCtx1.fillRect(0, 0, WIDTH, HEIGHT);
  canvasCtx1.lineWidth = 2;
  canvasCtx1.strokeStyle = "rgb(0, 0, 0)";
  canvasCtx1.beginPath();
  let sliceWidth = (WIDTH * 1.0) / bufferLength;
  for (let i = 0; i < bufferLength; i++) {
    let v = dataArray[i] / 128.0;
    let x = i * sliceWidth;
    let y = v * HEIGHT / 2;
    if (i === 0) {
      canvasCtx1.moveTo(x, y);
    } else {
      canvasCtx1.lineTo(x, y);
    }
  }
  canvasCtx1.lineTo(WIDTH, HEIGHT / 2);
  canvasCtx1.stroke();
}

function draw2() {
  analyser.getByteFrequencyData(dataArray);
  canvasCtx2.fillStyle = "rgb(200, 200, 200)";
  canvasCtx2.fillRect(0, 0, WIDTH, HEIGHT);
  canvasCtx2.lineWidth = 2;
  canvasCtx2.strokeStyle = "rgb(0, 0, 0)";
  canvasCtx2.beginPath();
  let sliceWidth = (WIDTH * 1.0) / bufferLength;
  let peak = 0;
  let peaki = 0;
  for (let i = 0; i < bufferLength; i++) {
    let v = dataArray[i] / 256.0;
    let x = i * sliceWidth;
    let y = (1 - v) * HEIGHT;
    if (i === 0) {
      canvasCtx2.moveTo(x, y);
    } else {
      canvasCtx2.lineTo(x, y);
    }
    if (peak < v) {
      peak = v;
      peaki = i;
    }
  }
  canvasCtx2.lineTo(WIDTH, HEIGHT);
  canvasCtx2.stroke();
  let hz = Math.round(peaki / bufferLength * 24000);
  freq.textContent = hz.toString();
}

draw();

logButton.onclick = () => {
  log.textContent += " " + freq.textContent;
};

</script>

<main>
	<h1>Hello {name}!</h1>
	<p>Visit the <a href="https://svelte.dev/tutorial">Svelte tutorial</a> to learn how to build Svelte apps.</p>
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>