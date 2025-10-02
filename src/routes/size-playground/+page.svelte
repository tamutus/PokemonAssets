<script lang="ts">
	import { onMount } from 'svelte';
	import Brendan from '$lib/assets/trainers/Brendan_E.png';
	import Ethan from '$lib/assets/trainers/Ethan_HGSS.png';
	import Gloria from '$lib/assets/trainers/Gloria.png';
	import Hilbert from '$lib/assets/trainers/Hilbert.png';
	import Leaf from '$lib/assets/trainers/Leaf.png';
	import Lucas from '$lib/assets/trainers/Lucas_Platinum.png';
	import Red from '$lib/assets/trainers/Red_FRLG.png';
	import Serena from '$lib/assets/trainers/Serena.png';
	import Victor from '$lib/assets/trainers/Victor.png';

	const specialRatios = [
		{
			number: 6,
			ratio: 1.7
		},
		{
			number: 23,
			ratio: 0.3
		},
		{
			number: 24,
			ratio: 0.45
		},
		{
			number: 26,
			ratio: 1.45
		},
		{
			number: 41,
			ratio: 1.3
		},
		{
			number: 42,
			ratio: 2.3
		},
		{
			number: 63,
			ratio: 1
		},
		{
			number: 68,
			ratio: 1.15
		},
		{
			number: 82,
			ratio: 1.1
		},
		{
			number: 86,
			ratio: 0.6
		},
		{
			number: 87,
			ratio: 0.7
		},
		{
			number: 89,
			ratio: 1.5
		},
		{
			number: 93,
			ratio: 1.3
		},
		{
			number: 95,
			ratio: 0.5
		},
		{
			number: 98,
			ratio: 2
		},
		{
			number: 99,
			ratio: 1.7
		},
		{
			number: 103,
			ratio: 1.05
		},
		{
			number: 109,
			ratio: 2
		},
		{
			number: 110,
			ratio: 1.7
		},
		{
			number: 118,
			ratio: 0.9
		},
		{
			number: 119,
			ratio: 0.9
		},
		{
			number: 119,
			ratio: 0.9
		},
		{
			number: 122,
			ratio: 1.1
		},
		{
			number: 123,
			ratio: 1.4
		},
		{
			number: 126,
			ratio: 1.2
		},
		{
			number: 130,
			ratio: 0.4
		},
		{
			number: 131,
			ratio: 0.8
		},
		{
			number: 140,
			ratio: 0.8
		},
		{
			number: 142,
			ratio: 2.4
		},
		{
			number: 144,
			ratio: 2.2
		},
		{
			number: 145,
			ratio: 1.5
		},
		{
			number: 146,
			ratio: 2.2
		},
		{
			number: 147,
			ratio: 0.55
		},
		{
			number: 148,
			ratio: 0.45
		}
	];

	const trainerData = [
		{
			name: 'Red',
			image: Red
		},
		{
			name: 'Leaf',
			image: Leaf
		},
		{
			name: 'Brendan',
			image: Brendan
		},
		{
			name: 'Ethan',
			image: Ethan
		},
		{
			name: 'Gloria',
			image: Gloria
		},
		{
			name: 'Hilbert',
			image: Hilbert
		},
		{
			name: 'Lucas',
			image: Lucas
		},
		{
			name: 'Serena',
			image: Serena
		},
		{
			name: 'Victor',
			image: Victor
		}
	];

	const activePokemon = $state({
		number: 148,
		name: '',
		height_m: 0
	});
	let activeTrainer = $state(trainerData[0]);

	const imageRepos = ['HD 3D Animated Gen 1', 'Gen 9'] as const;
	type Repo = (typeof imageRepos)[number];
	let imageRepo: Repo = $state(imageRepos[0]);
	let inFront: 'trainer' | 'pokemon' = $state('pokemon');
	let flipMon = $state(false);
	let flipTrainer = $state(false);
	let rotatePokemon = $state(0);
	let rotateTrainer = $state(0);

	const pokeTotalInches = $derived(39.3701 * activePokemon.height_m);
	const pokeHeight_ft = $derived(Math.floor(pokeTotalInches / 12));
	const pokeHeight_in = $derived(pokeTotalInches % 12);
	const adjustment = $derived(
		specialRatios.find((special) => special.number == activePokemon.number)?.ratio ?? 1
	);

	const pokemonSrc = $derived(
		imageRepo === 'HD 3D Animated Gen 1'
			? `https://raw.githubusercontent.com/Nackha1/Hd-sprites/refs/heads/master/${formatPokeNameForFetch(activePokemon.name)}.gif`
			: ''
	);

	let trainerName = $state('');
	let trainerHeight_ft = $state(4);
	let trainerHeight_in = $state(7);
	const trainerHeight_m = $derived((trainerHeight_ft * 12 + trainerHeight_in) / 39.3701);

	let picBox: HTMLDivElement;
	let pokemonContainer: HTMLDivElement;
	let gifElement: HTMLImageElement;
	let trainerContainer: HTMLDivElement;
	let trainerImage: HTMLImageElement;

	let availableHeight: number = $state(0);

	function formatPokeNameForFetch(name: string) {
		return name
			.replace('Mr-mime', 'Mr._Mime')
			.replace('-f', '_Female')
			.replace('-m', '_Male')
			.replace('Farfetchd', "Farfetch'd");
	}

	let draggingPokemon = $state(false),
		draggingTrainer = $state(false);
	let mousedownPokemonOffsetX = $state(0),
		mousedownPokemonOffsetY = $state(0),
		mousedownTrainerOffsetX = $state(0),
		mousedownTrainerOffsetY = $state(0),
		pokemonOffsetX = $state(),
		pokemonOffsetY = $state(),
		trainerOffsetX = $state(),
		trainerOffsetY = $state();

	const resetDrag = () => {
		pokemonOffsetX = undefined;
		pokemonOffsetY = undefined;
		trainerOffsetX = undefined;
		trainerOffsetY = undefined;
	};

	async function newFetch(target: string) {
		return fetch(target)
			.then((res) => {
				if (!res.ok) {
					throw Error(String(res.status));
				}
				return res;
			})
			.then((res) => res.json());
	}
	let updateAvailableHeight = () => {};

	let picScale: number = $derived(
		availableHeight / Math.max(trainerHeight_m, activePokemon.height_m * adjustment) / 100
	);
	const updatePokemon = async () => {
		const sourceUrl = `https://pokeapi.co/api/v2/pokemon/${activePokemon.number}`;
		const apiData = await newFetch(sourceUrl);
		activePokemon.name = apiData.name.slice(0, 1).toUpperCase() + apiData.name.slice(1);
		activePokemon.height_m = apiData.height / 10;
	};

	onMount(() => {
		updateAvailableHeight = () => {
			availableHeight = picBox.getBoundingClientRect().height;
		};
		updatePokemon();
		updateAvailableHeight();

		window.onresize = updateAvailableHeight;

		// Dragging

		function isOpaque(
			event: MouseEvent,
			imgElement: HTMLImageElement
			// flippedHorizontally: boolean
		) {
			// 1. Create a temporary canvas and context
			const canvas = document.createElement('canvas');
			const ctx = canvas.getContext('2d');
			if (!ctx) {
				return;
			}

			canvas.width = imgElement.naturalWidth;
			canvas.height = imgElement.naturalHeight;
			// 2. Draw the image onto the canvas
			ctx.drawImage(imgElement, 0, 0);
			// 3. Get the click coordinates relative to the image
			// We need to account for scaling if the image is displayed smaller/larger
			const rect = imgElement.getBoundingClientRect();
			const scaleX = imgElement.naturalWidth / rect.width;
			const scaleY = imgElement.naturalHeight / rect.height;

			// Apply scaling to get the coordinate on the full-size canvas
			const x = Math.floor((event.clientX - rect.left) * scaleX);
			const y = Math.floor((event.clientY - rect.top) * scaleY);

			// 4. Read the pixel data (RGBA for 1 pixel)
			const pixelData = ctx.getImageData(x, y, 1, 1).data;

			// 5. Check the Alpha Channel (the 4th value)
			const alpha = pixelData[3];

			// Return true if alpha is above a threshold (e.g., 5 out of 255)
			return alpha > 5;
		}

		gifElement.addEventListener('mousedown', (e) => {
			if (isOpaque(e, gifElement)) {
				draggingPokemon = true;
				mousedownPokemonOffsetX = e.clientX - gifElement.offsetLeft * (flipMon ? 1 : 1);
				mousedownPokemonOffsetY = e.clientY - gifElement.offsetTop;
			} else {
				gifElement.style.pointerEvents = 'none';
				document.elementFromPoint(e.clientX, e.clientY)?.dispatchEvent(new MouseEvent(e.type, e));
				setTimeout(() => {
					gifElement.style.pointerEvents = 'auto';
				}, 0);
			}
		});

		trainerImage.addEventListener('mousedown', (e) => {
			if (isOpaque(e, trainerImage)) {
				draggingTrainer = true;
				mousedownTrainerOffsetX = e.clientX - trainerImage.offsetLeft * (flipTrainer ? 1 : 1);
				mousedownTrainerOffsetY = e.clientY - trainerImage.offsetTop;
			} else {
				trainerImage.style.pointerEvents = 'none';
				document.elementFromPoint(e.clientX, e.clientY)?.dispatchEvent(new MouseEvent(e.type, e));
				setTimeout(() => {
					trainerImage.style.pointerEvents = 'auto';
				}, 0);
			}
		});

		document.addEventListener('mousemove', (e) => {
			if (draggingPokemon) {
				pokemonOffsetX = e.clientX - mousedownPokemonOffsetX;
				pokemonOffsetY = e.clientY - mousedownPokemonOffsetY;
			} else if (draggingTrainer) {
				trainerOffsetX = e.clientX - mousedownTrainerOffsetX;
				trainerOffsetY = e.clientY - mousedownTrainerOffsetY;
			}
		});

		document.addEventListener('mouseup', () => {
			draggingPokemon = false;
			draggingTrainer = false;
		});
	});
</script>

<div id="content">
	<div id="explanation" class="z-[3]">
		<p>
			See how big different Pokemon are in comparison to <em>your own</em> height. Not everybody is 4'7".
		</p>
		<p>You can move the pokemon and trainer around by dragging</p>
		<div>
			<input id="flip-pokemon" type="checkbox" bind:checked={flipMon} />
			<label for="flip-pokemon">Flip {activePokemon.name}</label>
		</div>
		<div>
			<label for="rotate-pokemon">Rotate {activePokemon.name}</label>
			<input type="number" step="1" bind:value={rotatePokemon} />
		</div>
		<div>
			<input id="flip-trainer" type="checkbox" bind:checked={flipTrainer} />
			<label for="flip-trainer">Flip {activeTrainer.name}</label>
		</div>
		<div>
			<label for="rotate-trainer">Rotate {activeTrainer.name}</label>
			<input type="number" step="1" bind:value={rotateTrainer} />
		</div>

		<div id="front-choice">
			<div>
				<input
					type="radio"
					id="pokemon-in-front"
					name="front"
					value="pokemon"
					bind:group={inFront}
				/>
				<label for="pokemon-in-front">{activePokemon.name} in front</label>
			</div>
			<div>
				<input
					type="radio"
					id="trainer-in-front"
					name="front"
					value="trainer"
					bind:group={inFront}
				/>
				<label for="trainer-in-front">{activeTrainer.name} in front</label>
			</div>
		</div>
	</div>
	<div id="interactive">
		<div id="choosers">
			<div id="pokemon-chooser" class="border-r">
				<span class="pokemon-name">{activePokemon.name}</span> #
				<input
					id="pokemon-number"
					type="number"
					min="1"
					max="151"
					bind:value={activePokemon.number}
					onchange={updatePokemon}
				/>
			</div>

			<div id="trainer-chooser">
				Trainer:
				<select bind:value={activeTrainer}>
					{#each trainerData as option (option.name)}
						<option value={option}>{option.name}</option>
					{/each}
				</select>
			</div>
		</div>

		<div id="heights">
			<div id="pokemon-height">
				{activePokemon.height_m} m / {pokeHeight_ft}' {pokeHeight_in.toFixed(1)}"
			</div>

			<div id="player-height">
				<input
					type="number"
					id="player-feet"
					min="0"
					max="8"
					step="1"
					bind:value={trainerHeight_ft}
				/>
				'
				<input
					type="number"
					id="player-inches"
					bind:value={trainerHeight_in}
					min="0"
					max="11.5"
					step=".5"
				/>
				" /
				{trainerHeight_m.toFixed(1)} m
				<button id="reset-drag" onclick={resetDrag}>Reset drag</button>
			</div>
		</div>

		<div id="pics" bind:this={picBox}>
			<div id="pokemon-container" draggable="false" bind:this={pokemonContainer}>
				<img
					id="pokemon-gif"
					draggable="false"
					crossorigin="anonymous"
					bind:this={gifElement}
					alt={activePokemon.name}
					src={pokemonSrc}
					class={inFront === 'pokemon' ? 'z-[2]' : 'z-[1]'}
					style={`
          transform: rotateY(${flipMon ? '180deg' : '0deg'}) rotateZ(${rotatePokemon}deg);
          height: ${activePokemon.height_m * 100 * adjustment}px;
          scale: ${picScale};
          left: ${pokemonOffsetX}px;
          top: ${pokemonOffsetY}px;
          `}
				/>
			</div>
			<div id="trainer-container" draggable="false" bind:this={trainerContainer}>
				<img
					id="trainer-avatar"
					draggable="false"
					crossOrigin="anonymous"
					alt={trainerName}
					bind:this={trainerImage}
					src={activeTrainer.image}
					class={inFront === 'trainer' ? 'z-[2]' : 'z-[1]'}
					style={`
          transform: rotateY(${flipTrainer ? '180deg' : '0deg'}) rotateZ(${rotateTrainer}deg);
          height: ${trainerHeight_m * 100}px;
          scale: ${picScale};
          left: ${trainerOffsetX}px;
          top: ${trainerOffsetY}px;
          `}
				/>
			</div>
		</div>
	</div>
	<div id="interactive-spacer"></div>
</div>
<div id="links">
	<a href="https://bulbapedia.bulbagarden.net/wiki/Player_character">More trainer images</a>
	<a href="https://github.com/Nackha1/Hd-sprites">Sprites</a>
</div>

<style>
	#content {
		display: flex;
		flex-wrap: wrap;
	}
	#explanation {
		width: 300px;
	}
	#interactive {
		display: flex;
		flex-flow: column;
		height: 98vh;
		flex-basis: 800px;
		flex-grow: 1;
		padding: 2vh 0 0 0;
	}
	#interactive-spacer {
		flex-basis: 100px;
		flex-shrink: 1;
	}

	input {
		padding: 5px 0;
		border: 1px solid grey;
	}

	#player-height input {
		width: 45px;
	}

	#pokemon-number {
		width: 50px;
	}

	#front-choice,
	#reset-drag {
		margin-left: 2rem;
	}

	#reset-drag {
		color: rgb(232, 232, 137);
		background-color: rgb(73, 105, 235);
	}

	#pics,
	#choosers,
	#heights {
		display: flex;
		flex-flow: row nowrap;
		align-items: center;
		justify-content: center;
		margin-bottom: 1rem;
	}
	#choosers,
	#heights {
		gap: 50px;
		z-index: 3;
	}
	#choosers > *,
	#heights > * {
		display: flex;
		align-items: center;
	}
	#pics {
		gap: 10px;
		align-items: flex-end;
		flex-grow: 1;
	}

	#pics > *,
	#heights > *,
	#choosers > * {
		width: 30%;
		display: flex;
		flex-flow: row nowrap;
		justify-content: flex-start;
		gap: 3px;
		position: relative;
	}
	#pics > * {
		height: 100%;
		justify-content: center;
	}

	#heights > :first-child,
	#choosers > :first-child {
		justify-content: flex-end;
	}

	img {
		object-fit: contain;
		position: absolute;
		bottom: 0;
		user-select: none;
	}
	#pokemon-gif {
		transform-origin: bottom;
	}
	#trainer-avatar {
		transform-origin: bottom;
	}
	#links {
		display: flex;
		flex-flow: row;
		gap: 1rem;
	}
</style>
