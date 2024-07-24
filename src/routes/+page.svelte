<script>
	let stream;
	let recordedChunks = [];
	async function startRecording() {
		try {
			stream = await navigator.mediaDevices.getDisplayMedia({
				video: { displaySurface: 'monitor' },
				audio: { suppressLocalAudioPlayback: false },
				systemAudio: 'include'
			});
			const mediaRecorder = new MediaRecorder(stream, { mimeType: 'video/webm; codecs=vp9' });
			mediaRecorder.ondataavailable = (e) => {
				if (e.data.size) {
					console.log('pushing');
					recordedChunks.push(e.data);
					stopRecording();
				}
			};
			mediaRecorder.start();
			document.querySelector('video').srcObject = stream;
			document.querySelector('video').play();
		} catch (e) {
			console.error(`Error: ${e}`);
		}
		return stream;
	}
	async function stopRecording() {
		stream.getTracks().forEach((track) => track.stop());
		document.querySelector('video').srcObject = null;
		//  output recorded video
		const blob = new Blob(recordedChunks, {
			type: 'video/webm'
		});
		const url = URL.createObjectURL(blob);
		const a = document.createElement('a');
		document.body.appendChild(a);
		a.style = 'display: none';
		a.href = url;
		a.download = 'test.webm';
		a.click();
		window.URL.revokeObjectURL(url);
		recordedChunks = [];
	}
	async function handleButton() {
		if (stream) {
			stopRecording();
		} else {
			startRecording();
		}
	}
</script>

<div class="mx-2 my-1">
	<h1 class="text-2xl font-semibold">Screen recorder</h1>
	<button on:click={handleButton} class="border-2 p-1 hover:bg-slate-100">Record screen!</button>
	<video />
</div>
