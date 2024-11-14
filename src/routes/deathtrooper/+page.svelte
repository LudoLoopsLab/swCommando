<script>
	import { Button } from 'flowbite-svelte';
	import { onMount } from 'svelte';

	let audioContext, source;

	let isPlaying = $state(false);

	async function startVoiceEffect() {
		try {
			// Demande d'accès au micro
			const stream = await navigator.mediaDevices.getUserMedia({ audio: true });

			// Initialisation du contexte audio
			audioContext = new (window.AudioContext || window.webkitAudioContext)();
			source = audioContext.createMediaStreamSource(stream);

			// Créer et configurer les effets
			const lowPassFilter = audioContext.createBiquadFilter();
			lowPassFilter.type = 'lowpass';
			lowPassFilter.frequency.value = 1000;

			const highPassFilter = audioContext.createBiquadFilter();
			highPassFilter.type = 'highpass';
			highPassFilter.frequency.value = 300;

			const distortion = audioContext.createWaveShaper();
			const curve = new Float32Array(256);
			for (let i = 0; i < 256; i++) {
				curve[i] = (Math.random() * 2 - 1) * 0.2;
			}
			distortion.curve = curve;
			distortion.oversample = '4x';

			// Connecter les effets
			source.connect(lowPassFilter);
			lowPassFilter.connect(highPassFilter);
			highPassFilter.connect(distortion);
			distortion.connect(audioContext.destination);
			isPlaying = true;
			console.log('Effet de voix en direct activé !');
		} catch (err) {
			console.error("Erreur d'accès au micro : ", err);
		}
	}

	function stopVoiceEffect() {
		if (audioContext) {
			audioContext.close();
			isPlaying = false;
		}
	}

	onMount(() => {
		return () => {
			stopVoiceEffect();
		};
	});
</script>

<div
	class="flex w-full justify-center
"
>
	<Button
		color="alternative"
		class="mx-auto mt-10"
		id="start"
		on:click={() => (isPlaying ? stopVoiceEffect() : startVoiceEffect())}
	>
		{#if isPlaying}
			Stop Voice Effect
		{:else}
			Start Voice Effect
		{/if}</Button
	>
</div>
