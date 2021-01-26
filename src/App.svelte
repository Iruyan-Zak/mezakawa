<script>
	let audioCtx;
	let analyser;
	let bufferLength;
	let dataArray;
	const WIDTH = 640;
	const HEIGHT = 320;
	const initialize = async () => {
		try {
			audioCtx = new (window.AudioContext || window.webkitAudioContext)();
			analyser = audioCtx.createAnalyser();
			analyser.fftSize = 32768;
			analyser.minDecibels = -120;
			analyser.maxDecibels = 0;
			analyser.smoothingTimeConstant = 0;
			console.log(analyser);
			bufferLength = analyser.frequencyBinCount;
			dataArray = new Uint8Array(bufferLength);
			const stream = await navigator.mediaDevices.getUserMedia({
				audio: true,
			});
			audioCtx.createMediaStreamSource(stream).connect(analyser);
		} catch (err) {
			throw err.toString();
		}
	};

	let canvas_waveTime;
	const startRenderWaveTime = () => {
		const ctx = canvas_waveTime.getContext("2d");
		requestAnimationFrame(loop);
		function loop(t) {
			requestAnimationFrame(loop);
			analyser.getByteTimeDomainData(dataArray);
			ctx.fillStyle = "rgb(200, 200, 200)";
			ctx.fillRect(0, 0, WIDTH, HEIGHT);
			ctx.lineWidth = 2;
			ctx.strokeStyle = "rgb(0, 0, 0)";
			ctx.beginPath();
			let sliceWidth = (WIDTH * 1.0) / bufferLength;
			for (let i = 0; i < bufferLength; i++) {
				let v = dataArray[i] / 128.0;
				let x = i * sliceWidth;
				let y = (v * HEIGHT) / 2;
				if (i === 0) {
					ctx.moveTo(x, y);
				} else {
					ctx.lineTo(x, y);
				}
			}
			ctx.lineTo(WIDTH, HEIGHT / 2);
			ctx.stroke();
		}
	};

	let canvas_waveFreqency;

	const leftEnd_x = 40;
	const leftEnd_log = Math.log10(leftEnd_x);
	const rightEnd_log = Math.log10(32768 / 2);
	const plotPointCount = WIDTH / 2;
	const dx_log = (rightEnd_log - leftEnd_log) / plotPointCount;
	const plotPointUpperLimits = [];
	for (let i = 1; i <= plotPointCount; i++) {
		plotPointUpperLimits.push(
			Math.floor(Math.pow(10, leftEnd_log + i * dx_log))
		);
	}
	console.log(plotPointUpperLimits);

	const startRenderWaveFreqency = () => {
		const ctx = canvas_waveFreqency.getContext("2d");
		requestAnimationFrame(loop);
		function loop(t) {
			requestAnimationFrame(loop);

			analyser.getByteFrequencyData(dataArray);
			ctx.fillStyle = "rgb(200, 200, 200)";
			ctx.fillRect(0, 0, WIDTH, HEIGHT);
			ctx.lineWidth = 2;
			ctx.strokeStyle = "rgb(0, 0, 0)";
			ctx.beginPath();
			ctx.moveTo(0, HEIGHT - 32);

			let lower = leftEnd_x;
			let plotPoint_x = 0;
			for (const upper of plotPointUpperLimits) {
				let sum = 0.0;
				for (let x = lower; x <= upper; x++) {
					sum += dataArray[x];
				}
				const average = sum / (upper - lower + 1);
				lower = upper + 1;

				ctx.lineTo(plotPoint_x, 256 - average + 32);
				plotPoint_x += 2;
			}

			// for (let i = 0; i < bufferLength; i++) {
			// 	let v = dataArray[i] / 256.0;
			// 	let x = i * sliceWidth;
			// 	let y = (1 - v) * HEIGHT;
			// 	if (i === 0) {
			// 		ctx.moveTo(x, y);
			// 	} else {
			// 		ctx.lineTo(x, y);
			// 	}
			// 	if (peak < v) {
			// 		peak = v;
			// 		peaki = i;
			// 	}
			// }
			ctx.lineTo(WIDTH, HEIGHT - 32);
			ctx.stroke();
			// let hz = Math.round((peaki / bufferLength) * 24000);
		}
	};

	// draw an oscilloscope of the current audio source

	let systemStarted;
	const startSystem = () => {
		systemStarted = initialize();
	};
	const resetSystem = () => {
		systemStarted = Promise.reject("開始してください。");
	};
	resetSystem();
</script>

<main>
	<div>
		{#await systemStarted then _}
			<div>
				<button on:click={resetSystem}>stop</button>
			</div>
			<div>
				<canvas
					bind:this={canvas_waveTime}
					on:click={startRenderWaveTime}
					width={WIDTH}
					height={HEIGHT}
				/>
				<canvas
					bind:this={canvas_waveFreqency}
					on:click={startRenderWaveFreqency}
					width={WIDTH}
					height={HEIGHT}
				/>
			</div>
		{:catch message}
			<div>
				<p>メッセージエリア</p>
				<p>
					message: {message}
				</p>
				<div>
					<button on:click={startSystem}>start</button>
				</div>
			</div>
		{/await}
	</div>
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
