<script>
	import { onMount } from 'svelte';
	export let data;
	let stream;
	let recordedChunks = [];
	let cdn;
	let unsupported;
	async function startRecording() {
		try {
			if (document.querySelector('div[role="dialog"]')) {
				document.querySelector('div[role="dialog"]').classList.add('hidden');
			}
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
		console.log('we stoppin');
		document.querySelector('button').textContent = 'All done!';
		stream.getTracks().forEach((track) => track.stop());
		document.querySelector('video').srcObject = null;
		new Audio('end.mp3').play();
		const blob = new Blob(recordedChunks, {
			type: 'video/mp4'
		});
		if (blob.size === 0) {
			return;
		}
		const url = URL.createObjectURL(blob);
		const a = document.createElement('a');
		document.body.appendChild(a);
		a.style = 'display: none';
		a.href = url;
		a.download = 'recording.mp4';
		a.click();
		window.URL.revokeObjectURL(url);
		recordedChunks = [];
	}
	async function handleButton() {
		if (stream) {
			stopRecording();
		} else {
			if (data.visited === false) {
				document.querySelector('.hidden').classList.remove('hidden');
			} else {
				startRecording();
			}
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
		<h1 class="text-2xl font-semibold">In-Browser Screen Recorder!</h1>
		<p class="text-gray-750">Click the button below to start recording your screen.</p>
		<button
			on:click={handleButton}
			class="inline-flex w-full justify-center rounded-md bg-purple-600 px-3 py-2 text-sm font-semibold text-white shadow-sm hover:bg-purple-500 sm:w-auto hover:bg-slate-100 dark:hover:bg-slate-900"
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
<div class="relative z-10 hidden" aria-labelledby="modal-title" role="dialog" aria-modal="true">
	<!--
	  Background backdrop, show/hide based on modal state.
  
	  Entering: "ease-out duration-300"
		From: "opacity-0"
		To: "opacity-100"
	  Leaving: "ease-in duration-200"
		From: "opacity-100"
		To: "opacity-0"
	-->
	<div class="fixed inset-0 bg-gray-500 bg-opacity-75 transition-opacity" aria-hidden="true"></div>

	<div class="fixed inset-0 z-10 w-screen overflow-y-auto">
		<div class="flex min-h-full items-end justify-center p-4 text-center sm:items-center sm:p-0">
			<!--
		  Modal panel, show/hide based on modal state.
  
		  Entering: "ease-out duration-300"
			From: "opacity-0 translate-y-4 sm:translate-y-0 sm:scale-95"
			To: "opacity-100 translate-y-0 sm:scale-100"
		  Leaving: "ease-in duration-200"
			From: "opacity-100 translate-y-0 sm:scale-100"
			To: "opacity-0 translate-y-4 sm:translate-y-0 sm:scale-95"
		-->
			<div
				class="relative transform overflow-hidden rounded-lg bg-white text-left shadow-xl transition-all sm:my-8 sm:w-full sm:max-w-lg"
			>
				<div class="bg-white px-4 pb-4 pt-5 sm:p-6 sm:pb-4">
					<div class="sm:flex sm:items-start">
						<div
							class="mx-auto flex h-12 w-12 flex-shrink-0 items-center justify-center rounded-full bg-purple-100 sm:mx-0 sm:h-10 sm:w-10"
						>
							<svg
								xmlns="http://www.w3.org/2000/svg"
								fill="none"
								viewBox="0 0 24 24"
								stroke-width="1.5"
								stroke="currentColor"
								class="text-purple-600 h-6 w-6"
							>
								<path
									stroke-linecap="round"
									stroke-linejoin="round"
									d="M9.879 7.519c1.171-1.025 3.071-1.025 4.242 0 1.172 1.025 1.172 2.687 0 3.712-.203.179-.43.326-.67.442-.745.361-1.45.999-1.45 1.827v.75M21 12a9 9 0 1 1-18 0 9 9 0 0 1 18 0Zm-9 5.25h.008v.008H12v-.008Z"
								/>
							</svg>
						</div>
						<div class="mt-3 text-center sm:ml-4 sm:mt-0 sm:text-left">
							<h3 class="text-base font-semibold leading-6 text-gray-900" id="modal-title">
								How does this work?
							</h3>
							<div class="mt-2">
								<p class="text-sm text-gray-500">
									Click to begin recording and select what you'd like to record. You'll have 3
									seconds to prepare before recording begins. After you're done, you can stop
									recording using either the browser controls or coming back here. The video will
									download automatically afterwards. Happy recording, you've got this! <br /> FYI: We
									won't show this message again.
								</p>
							</div>
						</div>
					</div>
				</div>
				<div class="bg-gray-50 px-4 py-3 sm:flex sm:flex-row-reverse sm:px-6">
					<button
						type="button"
						class="inline-flex w-full justify-center rounded-md bg-purple-600 px-3 py-2 text-sm font-semibold text-white shadow-sm hover:bg-purple-500 sm:ml-3 sm:w-auto"
						on:click={startRecording}>Begin</button
					>
				</div>
			</div>
		</div>
	</div>
</div>
