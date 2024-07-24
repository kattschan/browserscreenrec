<script>
	import { onMount } from 'svelte';
	let stream;
	let recordedChunks = [];
	let cdn;
	let unsupported;
	async function startRecording() {
		try {
			stream = await navigator.mediaDevices.getDisplayMedia({
				video: { displaySurface: 'monitor' },
				audio: { suppressLocalAudioPlayback: false },
				systemAudio: 'include'
			});
			const audio = new Audio('ding.mp3');
			for (let i = 0; i < 3; i++) {
				cdn = i + 1;
				audio.play();
				await new Promise((resolve) => setTimeout(resolve, 750));
			}
			cdn = 'Go!';
			audio.src = 'begin.mp3';
			audio.play();
			cdn = null;
			const mediaRecorder = new MediaRecorder(stream, { mimeType: 'video/mp4' });
			mediaRecorder.ondataavailable = (e) => {
				if (e.data.size) {
					console.log('pushing');
					recordedChunks.push(e.data);
					stopRecording();
				}
			};
			mediaRecorder.start();
			document.querySelector('video').srcObject = stream;
			document.querySelector('video').volume = 0;
			document.querySelector('video').play();
		} catch (e) {
			console.error(`Error: ${e}`);
		}
		return stream;
	}
	async function stopRecording() {
		document.querySelector('button').textContent = 'All done!';
		stream.getTracks().forEach((track) => track.stop());
		document.querySelector('video').srcObject = null;
		new Audio('end.mp3').play();
		const blob = new Blob(recordedChunks, {
			type: 'video/mp4'
		});
		const url = URL.createObjectURL(blob);
		const a = document.createElement('a');
		document.body.appendChild(a);
		a.style = 'display: none';
		a.href = url;
		a.download = 'test.mp4';
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
	onMount(() => {
		if (!navigator.mediaDevices?.getDisplayMedia) {
			unsupported = true;
		}
	});
</script>

<div class="mx-2 my-1 dark:text-gray-100">
	{#if !unsupported}
		<h1 class="text-2xl font-semibold">Welcome to browserscreenrec!</h1>
		<button on:click={handleButton} class="border-2 p-1 hover:bg-slate-100 dark:hover:bg-slate-900"
			>{stream ? 'Stop Recording' : 'Start Recording'}</button
		>
		{#if cdn}
			<div class="flex justify-center items-center">
				<p class="text-red-400 text-4xl">{cdn}</p>
			</div>
		{/if}
		<!-- svelte-ignore a11y-media-has-caption -->
		<video />
	{:else}
		<p class="text-red-400 text-2xl">Your browser does not support screen recording.</p>
	{/if}
</div>
