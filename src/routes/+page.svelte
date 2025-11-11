<script lang="ts">
	import { cubicIn, cubicOut } from "svelte/easing";

	function wipeIn(
		node: Element,
		{
			duration,
			easing,
		}: { duration: number; easing: (t: number) => number }
	) {
		return {
			duration,
			easing,
			css: (t: number) => {
				// t goes from 0 to 1
				// 0: clip-path: inset(0 0 0 100%) (hidden)
				// 1: clip-path: inset(0 0 0 0%) (visible)
				return `
					clip-path: inset(0 0 0 ${100 - t * 100}%);
					z-index: 2; /* Incoming screen is 'on top' */
				`;
			},
		};
	}

	function wipeOut(
		node: Element,
		{
			duration,
			easing,
		}: { duration: number; easing: (t: number) => number }
	) {
		return {
			duration,
			easing,
			css: (t: number) => {
				// t goes from 1 to 0
				// 1: clip-path: inset(0 0% 0 0) (visible)
				// 0: clip-path: inset(0 100% 0 0) (hidden)
				return `
					clip-path: inset(0 ${100 - t * 100}% 0 0);
					z-index: 1; /* Outgoing screen is 'underneath' */
				`;
			},
		};
	}

	let numbersToGenInput = $state("1");
	let incl = $state("including");
	let withRep = $state("without replacement");
	let minInput = $state("0");
	let maxInput = $state("0");
	let numbersToGen = $derived(Number(numbersToGenInput) || 1);
	let min = $derived(Number(minInput) || 0);
	let max = $derived(Number(maxInput) || 0);
	let effMin = $derived(incl === "including" ? min : min + 1);
	let effMax = $derived(incl === "including" ? max : max - 1);
	let rangeSize = $derived(Math.max(0, effMax - effMin + 1));

	let disabled = $derived(
		(() => {
			if (effMax < effMin) {
				return true;
			}

			if (numbersToGen <= 0) {
				return true;
			}

			if (withRep === "without replacement" && rangeSize < numbersToGen) {
				return true;
			}

			return false;
		})()
	);
	let results: number[] = $state([]);
	let isLoading = $state(false);

	function handleInput(e: Event) {
		const target = e.currentTarget as HTMLInputElement;
		if (target.value.length === 0) {
			return "0";
		}

		const digitsOnly = target.value.replace(/\D/g, "");
		const noLeadingZeros = digitsOnly.replace(/^0+/, "");
		if (digitsOnly.length > 0 && noLeadingZeros.length === 0) {
			return "0";
		}

		return noLeadingZeros;
	}

	async function startGeneration() {
		isLoading = true;
		await new Promise((resolve) => setTimeout(resolve, 1500));

		generate();
		isLoading = false;
	}

	async function returnToGenerator() {
		isLoading = true;
		await new Promise((resolve) => setTimeout(resolve, 1500));

		results.length = 0;
		isLoading = false;
	}

	function generate() {
		const actualMin = incl === "including" ? min : min + 1;
		const actualMax = incl === "including" ? max : max - 1;
		const span = actualMax - actualMin + 1;
		while (results.length < numbersToGen) {
			let result = Math.floor(Math.random() * span) + actualMin;
			if (withRep === "without replacement") {
				while (results.includes(result)) {
					result = Math.floor(Math.random() * span) + actualMin;
				}
			}

			results.push(result);
		}
	}
</script>

<div id="page">
	{#if isLoading}
		<div
			id="loading-screen"
			in:wipeIn={{ duration: 700, easing: cubicOut }}
			out:wipeOut={{ duration: 700, easing: cubicIn }}
		>
			<p class="loading-text">Generating</p>
			<div class="funk-loader">
				<div class="dot"></div>
				<div class="dot"></div>
				<div class="dot"></div>
				<div class="dot"></div>
			</div>
		</div>
	{:else if results.length > 0}
		<div
			id="results-screen"
			in:wipeIn={{ duration: 700, easing: cubicOut }}
			out:wipeOut={{ duration: 700, easing: cubicIn }}
		>
			<h1>Results:</h1>

			<div id="results-container">
				<button
					id="copy-button"
					onclick={() =>
						navigator.clipboard.writeText(results.join(", "))}
					>Copy</button
				>
				{#each results as result}
					<p class="result">
						{result}
					</p>
				{/each}
			</div>
			<button
				class="main-action-button"
				onclick={() => returnToGenerator()}
			>
				Return
			</button>
		</div>
	{:else}
		<div
			id="generator-screen"
			in:wipeIn={{ duration: 700, easing: cubicOut }}
			out:wipeOut={{ duration: 700, easing: cubicIn }}
		>
			<p id="statement">
				Generate
				<br />

				<input
					inputmode="numeric"
					bind:value={numbersToGenInput}
					oninput={(e) => (numbersToGenInput = handleInput(e))}
				/>
				<br />
				{#if numbersToGen === 1}
					number
				{:else}
					numbers
				{/if}
				<br />
				<select bind:value={withRep}>
					<option value="with replacement">with replacement</option>
					<option value="without replacement"
						>without replacement</option
					>
				</select>
				<br />

				between
				<br />

				<input
					inputmode="numeric"
					id="left"
					bind:value={minInput}
					oninput={(e) => (minInput = handleInput(e))}
				/>
				and
				<input
					inputmode="numeric"
					id="right"
					bind:value={maxInput}
					oninput={(e) => (maxInput = handleInput(e))}
				/>
				<br />

				<select bind:value={incl}>
					<option value="including">including</option>or
					<option value="not including">not including</option>
				</select>
			</p>
			<button
				class="main-action-button"
				onclick={() => startGeneration()}
				{disabled}
			>
				Generate
			</button>
		</div>
	{/if}
</div>

<style>
	@font-face {
		font-family: "Spicy Rice";
		src: url(/Spicy_Rice/SpicyRice-Regular.ttf) format("truetype");
		font-weight: 400;
		font-style: normal;
	}

	:global(body) {
		--bg: #f5f5dc;
		--text: #7b3f00;
		--primary: #b7410e;
		--secondary: #da9100;
		--tertiary: #568203;
		--light-text: #f5f5dc;
		--shadow: 0 4px 12px 0 rgba(0, 0, 0, 0.08),
			0 6px 20px 0 rgba(0, 0, 0, 0.06);

		font-family: "Spicy Rice", serif;
		background-color: var(--bg);
		color: var(--text);
	}
	#page {
		height: 100dvh;
		width: 100dvw;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;

		font-size: 3em;
		position: relative;
	}

	button {
		appearance: none;
		border: none;
		background-color: transparent;

		color: var(--secondary);
	}
	button:hover {
		color: var(--tertiary);
	}
	button:disabled,
	button:hover:disabled {
		opacity: 60%;
		color: var(--secondary);
	}

	/* Generator Screen */

	#generator-screen {
		height: 100dvh;
		width: 100dvw;

		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: space-evenly;
		position: absolute;
		z-index: 1;
		background-color: var(--bg);
	}
	#statement {
		margin: 0;
		padding: 0;
		text-align: center;
	}
	input,
	select {
		margin: 0;
		padding: 0;

		color: var(--primary);

		appearance: none;
		border: none;
		background-color: transparent;
		text-align: center;
		line-height: 1.8em;
		text-decoration-line: underline;
		text-decoration-style: wavy;
		text-decoration-thickness: 0.1em;
		text-underline-offset: 0.3em;
	}

	select {
		text-align-last: center;
	}
	input {
		width: 25%;
	}
	input#left {
		text-align: right;
	}
	input#right {
		text-align: left;
	}

	/* Loading Screen Styles */
	#loading-screen {
		height: 100dvh;
		width: 100dvw;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		position: absolute;
		gap: 2em;
		z-index: 1;
		background-color: var(--bg);
	}

	.loading-text {
		font-size: 1.5em;
		color: var(--primary);
		text-transform: uppercase;
		letter-spacing: 0.1em;
		margin-bottom: -1em;
	}

	.funk-loader {
		display: flex;
		margin-top: 0;
		gap: 1em;
	}

	.funk-loader .dot {
		width: 2rem;
		height: 2rem;
		border-radius: 50%;
		animation: funk-scale 1.4s infinite ease-in-out both;
	}

	.funk-loader .dot:nth-child(1) {
		background-color: var(--primary);
		animation-delay: -0.32s;
	}
	.funk-loader .dot:nth-child(2) {
		background-color: var(--secondary);
		animation-delay: -0.16s;
	}
	.funk-loader .dot:nth-child(3) {
		background-color: var(--tertiary);
	}
	.funk-loader .dot:nth-child(4) {
		background-color: var(--primary);
		animation-delay: 0.16s;
	}

	@keyframes funk-scale {
		0%,
		80%,
		100% {
			transform: scale(0.8);
			opacity: 0.5;
		}
		40% {
			transform: scale(1.2);
			opacity: 1;
		}
	}

	/* Results Screen */
	#results-screen {
		height: 100dvh;
		width: 100dvw;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: space-evenly;

		position: absolute;
		z-index: 1;
		background-color: var(--bg);
	}
	#copy-button {
		position: absolute;
		font-size: 0.5em;
		right: 0.75em;
		top: 7em;
		transform: rotate(45deg);
	}
	#results-container {
		margin: 0;
		padding: 0;
		height: 55dvh;
		width: 90%;
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(2em, 1fr));
		place-items: center;
		gap: 0.5em;
		overflow-y: scroll;
	}
	.result {
		margin: 0;
		padding: 0;
	}
	h1 {
		margin: 0;
		padding: 0;
	}

	@media (max-width: 640px) {
		#page {
			font-size: 2.5em;
		}
		#copy-button {
			top: 10.75em;
			right: 0.5em;
		}
		#results-container {
			height: 40dvh;
			grid-template-columns: repeat(auto-fit, minmax(2em, 1fr));
		}
	}
</style>
