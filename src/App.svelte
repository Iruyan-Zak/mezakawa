<script>
	let audioCtx;
	let analyser;
	let bufferLength;
	let dataArray;
	const WIDTH = 500;
	const HEIGHT = 200;
	const initialize = async () => {
		try {
			audioCtx = new (window.AudioContext || window.webkitAudioContext)();
			analyser = audioCtx.createAnalyser();
			analyser.fftSize = 32768;
			bufferLength = analyser.frequencyBinCount;
			dataArray = new Uint8Array(bufferLength);
			const stream = await navigator.mediaDevices.getUserMedia({
				audio: true,
			});
			audioCtx.createMediaStreamSource(stream).connect(analyser);
		} catch (err) {
			errorMessage.textContent = err.toString();
		}
	};

	let canvas_waveTime;
	let canvas_waveFreqency;
	const startRenderWaveTime = () => {
		const ctx = canvas_waveTime.getContext("2d");
		let frame;
		loop();

		function loop(t) {
			console.log(t);
			frame = requestAnimationFrame(loop);
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

	const startRenderWaveFreqency = () => {
		const ctx = canvas_waveFreqency.getContext("2d");
		let frame = requestAnimationFrame(loop);

		function loop(t) {
			frame = requestAnimationFrame(loop);
			analyser.getByteFrequencyData(dataArray);
			ctx.fillStyle = "rgb(200, 200, 200)";
			ctx.fillRect(0, 0, WIDTH, HEIGHT);
			ctx.lineWidth = 2;
			ctx.strokeStyle = "rgb(0, 0, 0)";
			ctx.beginPath();
			let sliceWidth = (WIDTH * 1.0) / bufferLength;
			let peak = 0;
			let peaki = 0;
			for (let i = 0; i < bufferLength; i++) {
				let v = dataArray[i] / 256.0;
				let x = i * sliceWidth;
				let y = (1 - v) * HEIGHT;
				if (i === 0) {
					ctx.moveTo(x, y);
				} else {
					ctx.lineTo(x, y);
				}
				if (peak < v) {
					peak = v;
					peaki = i;
				}
			}
			ctx.lineTo(WIDTH, HEIGHT);
			ctx.stroke();
			// let hz = Math.round((peaki / bufferLength) * 24000);
		}
	};

	// draw an oscilloscope of the current audio source

	let systemStarted = Promise.reject("開始してください。");
	const onClickStart = () => {
		systemStarted = initialize();
	};
</script>

<main>
	<div>
		{#await systemStarted then _}
			<div>
				<button>stop</button>
			</div>
			<div>
				<canvas
					bind:this={canvas_waveTime}
					on:click={startRenderWaveTime}
					width={500}
					height={200}
				/>
				<canvas
					bind:this={canvas_waveFreqency}
					on:click={startRenderWaveFreqency}
					width={500}
					height={200}
				/>
			</div>
		{:catch message}
			<div>
				<p>メッセージエリア</p>
				<p>
					message: {message}
				</p>
				<div>
					<button on:click={onClickStart}>start</button>
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
