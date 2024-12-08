<script lang="ts">
	import { onDestroy } from 'svelte';

	let audioContext: AudioContext | null;
	let oscillator: OscillatorNode;
	let gainNode: GainNode;
	let volumeOscillator: OscillatorNode;
	let currentVolume = 0; // Volume actuel observé
	let intervalId: number | null = null;

	const frequency = 44; // Fréquence principale en Hz
	const volumeFrequency = 8; // Fréquence de l'oscillation du volume (en Hz)

	const start = () => {
		if (!audioContext) {
			audioContext = new AudioContext();

			// Oscillateur principal
			oscillator = audioContext.createOscillator();
			oscillator.type = 'sine';
			oscillator.frequency.setValueAtTime(frequency, audioContext.currentTime);

			// Gain node pour le contrôle du volume
			gainNode = audioContext.createGain();
			gainNode.gain.setValueAtTime(0.5, audioContext.currentTime); // Amplitude initiale

			// Oscillateur de contrôle pour le volume
			volumeOscillator = audioContext.createOscillator();
			volumeOscillator.type = 'sine';
			volumeOscillator.frequency.setValueAtTime(volumeFrequency, audioContext.currentTime);

			// Moduler le gain avec l'oscillateur de volume
			const volumeGain = audioContext.createGain();
			volumeGain.gain.setValueAtTime(0.5, audioContext.currentTime); // Amplitude de l'oscillation

			volumeOscillator.connect(volumeGain);
			volumeGain.connect(gainNode.gain);

			// Connecter l'oscillateur principal à la sortie audio
			oscillator.connect(gainNode);
			gainNode.connect(audioContext.destination);

			// Démarrer les oscillateurs
			oscillator.start();
			volumeOscillator.start();

			// Démarrer l'observation périodique
			startVolumeObservation();
		}
	};

	const stop = () => {
		if (audioContext) {
			oscillator?.stop();
			volumeOscillator?.stop();
			audioContext.close();
			audioContext = null;
		}
		stopVolumeObservation();
	};

	const startVolumeObservation = () => {
		if (intervalId === null) {
			intervalId = setInterval(() => {
				if (gainNode) {
					// Lire la valeur actuelle modulée
					currentVolume = gainNode.gain.value;
				}
			}, 50); // Rafraîchir toutes les 50 ms
		}
	};

	const stopVolumeObservation = () => {
		if (intervalId !== null) {
			clearInterval(intervalId);
			intervalId = null;
		}
	};

	onDestroy(() => {
		stop();
	});
</script>

<!-- ****************************************************** CONTENT -->
<div>
	<button on:click={start}>Start</button>
	<button on:click={stop}>Stop</button>
</div>
<div>
	<p>Fréquence principale: {frequency} Hz</p>
	<p>Fréquence de l'oscillation du volume: {volumeFrequency} Hz</p>
	<p>Volume actuel: {currentVolume.toFixed(2)}</p>
</div>

<!-- ****************************************************** STYLE -->
<style lang="css">
	button {
		margin: 0.5rem;
		padding: 0.5rem 1rem;
		font-size: 1rem;
		cursor: pointer;
	}
</style>
