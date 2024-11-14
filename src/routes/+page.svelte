<script>
	import { onMount } from 'svelte';
	import { Button } from 'flowbite-svelte';
	import { Range, Label } from 'flowbite-svelte';

	let audioContext;
	let isPlaying = $state(false);

	// Paramètres ajustables
	let lowPassFrequency = $state(1200);
	let highPassFrequency = $state(200);
	let distortionAmount = $state(12);
	let compressionThreshold = $state(-20);
	let delayTime = $state(0.02);
	let pitch = $state(1); // Contrôle pour ajuster la tonalité

	let lowPassFilter, highPassFilter, distortion, compressor, delay, feedback, pitchShifter;

	async function startVoiceEffect() {
		try {
			const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
			audioContext = new (window.AudioContext || window.webkitAudioContext)();
			const source = audioContext.createMediaStreamSource(stream);

			// Pitch Shifter (contrôle de la vitesse de lecture)
			pitchShifter = audioContext.createGain();
			pitchShifter.gain.value = pitch;

			// Filtre passe-bas
			lowPassFilter = audioContext.createBiquadFilter();
			lowPassFilter.type = 'lowpass';
			lowPassFilter.frequency.value = lowPassFrequency;

			// Filtre passe-haut
			highPassFilter = audioContext.createBiquadFilter();
			highPassFilter.type = 'highpass';
			highPassFilter.frequency.value = highPassFrequency;

			// Distorsion
			distortion = audioContext.createWaveShaper();
			updateDistortion();

			// Compresseur
			compressor = audioContext.createDynamicsCompressor();
			compressor.threshold.setValueAtTime(compressionThreshold, audioContext.currentTime);
			compressor.knee.setValueAtTime(20, audioContext.currentTime);
			// compressor.ratio.setValueAtTime(8, audioContext.currentTime);
			compressor.attack.setValueAtTime(0.1, audioContext.currentTime);
			compressor.release.setValueAtTime(0.25, audioContext.currentTime);

			// Délai pour l'effet radio
			delay = audioContext.createDelay();
			delay.delayTime.value = delayTime;
			feedback = audioContext.createGain();
			feedback.gain.value = 0.3;

			// Chaînage des effets
			source.connect(lowPassFilter);
			lowPassFilter.connect(highPassFilter);
			highPassFilter.connect(distortion);
			distortion.connect(delay);
			delay.connect(feedback);
			feedback.connect(delay); // Boucle de feedback pour l'écho léger
			delay.connect(compressor);
			compressor.connect(pitchShifter); // Ajout du pitch shifter dans la chaîne
			pitchShifter.connect(audioContext.destination);

			isPlaying = true;
			console.log("Effet de voix 'Republic Commando' activé !");
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

	// Mise à jour des effets en fonction des contrôles
	function updateDistortion() {
		const curve = new Float32Array(256);
		for (let i = 0; i < 256; i++) {
			curve[i] = Math.tanh((i / 128 - 1) * distortionAmount);
		}
		distortion.curve = curve;
		distortion.oversample = '4x';
	}

	$effect(() => {
		distortionAmount;
		updateDistortion();
	});

	function updateFilters() {
		if (lowPassFilter) lowPassFilter.frequency.value = lowPassFrequency;
		if (highPassFilter) highPassFilter.frequency.value = highPassFrequency;
	}

	$effect(() => {
		lowPassFrequency;
		highPassFrequency;
		updateFilters();
	});

	function updateCompressor() {
		if (compressor)
			compressor.threshold.setValueAtTime(compressionThreshold, audioContext.currentTime);
	}
	$effect(() => {
		compressionThreshold;
		updateCompressor();
	});
	function updateDelay() {
		if (delay) delay.delayTime.value = delayTime;
	}

	$effect(() => {
		delayTime;
		updateDelay();
	});

	function updatePitch() {
		if (pitchShifter) pitchShifter.gain.value = pitch;
	}

	$effect(() => {
		pitch;
		updatePitch();
	});

	onMount(() => {
		return () => {
			stopVoiceEffect();
		};
	});
</script>

<div class=" mx-auto mt-10 flex w-96 flex-col justify-center">
	<div class="controls mb-6 flex flex-col items-center gap-6">
		<div
			class="control
		"
		>
			<Label for="lowPassFrequency">Low Pass Frequency: {lowPassFrequency}</Label>
			<Range id="lowPassFrequency" min="500" max="3000" bind:value={lowPassFrequency} />
			<!-- <input
				type="range"
				id="lowPassFrequency"
				min="500"
				max="3000"
				bind:value={lowPassFrequency}
				on:input={updateFilters}
			/> -->
		</div>

		<div class="control">
			<Label for="highPassFrequency">High Pass Frequency: {highPassFrequency}</Label>
			<Range
				id="highPassFrequency"
				min="50"
				max="1000"
				bind:value={highPassFrequency}
				on:input={updateFilters}
			/>
			<!-- <input
				type="range"
				id="highPassFrequency"
				min="50"
				max="1000"
				bind:value={highPassFrequency}
				on:input={updateFilters}
			/> -->
		</div>

		<div class="control">
			<Label for="distortionAmount">Distortion Amount: {distortionAmount}</Label>
			<Range
				id="distortionAmount"
				min="10.0"
				max="20.0"
				step="0.5"
				bind:value={distortionAmount}
				on:input={updateDistortion}
			/>

			<!-- <input
				type="range"
				id="distortionAmount"
				min="10.0"
				max="20.0"
				step="0.5"
				bind:value={distortionAmount}
				on:input={updateDistortion}
			/> -->
		</div>

		<div class="control">
			<Label for="compressionThreshold">Compression Threshold: {compressionThreshold}</Label>
			<Range
				id="compressionThreshold"
				min="-50"
				max="0"
				bind:value={compressionThreshold}
				on:input={updateCompressor}
			/>
			<!-- <input
				type="range"
				id="compressionThreshold"
				min="-50"
				max="0"
				bind:value={compressionThreshold}
				on:input={updateCompressor}
			/> -->
		</div>

		<div class="control">
			<Label for="delayTime">Delay Time:{delayTime}</Label>
			<Range
				id="delayTime"
				min="0.01"
				max="0.1"
				step="0.01"
				bind:value={delayTime}
				on:input={updateDelay}
			/>
			<!-- <input
				type="range"
				id="delayTime"
				min="0.01"
				max="0.1"
				step="0.01"
				bind:value={delayTime}
				on:input={updateDelay}
			/> -->
		</div>

		<div class="control">
			<Label for="pitch">Pitch (Grave - Aigu): {pitch}</Label>
			<Range id="pitch" min="0.5" max="2" step="0.1" bind:value={pitch} on:input={updatePitch} />
			<!-- <input
				type="range"
				id="pitch"
				min="0.5"
				max="2"
				step="0.1"
				bind:value={pitch}
				on:input={updatePitch}
			/> -->
		</div>
	</div>
	<Button
		color="alternative"
		class="mx-auto"
		on:click={() => (isPlaying ? stopVoiceEffect() : startVoiceEffect())}
	>
		{#if isPlaying}
			Stop Voice Effect
		{:else}
			Start Voice Effect
		{/if}
	</Button>
</div>

<style>
	.control {
		display: flex;
		flex-wrap: wrap;
		gap: 10px;
		justify-content: space-between;
		width: 300px;
	}
</style>
