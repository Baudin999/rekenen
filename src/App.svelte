<script>
	import { onMount } from "svelte";
	let settingsShown = false;
	let min = 0;
	let max = 10;
	let dif = 2;
	let left = 0;
	let right = 0;
	let result = 0;
	let disabled = false;
	let status = "neutral";
	let _input;
	let message;
	let started = false;

	let plus = true;
	let minus = false;
	let times = false;

	let signs = [];
	let sign;

	let goed = 0;
	let fout = 0;

	let timeout;
	let timeoutFactor = 5000;

	const start = () => {
		if (!plus && !minus && !times) plus = true;

		signs = [];
		if (plus) signs.push("+");
		if (minus) signs.push("-");
		if (times) signs.push("*");

		generate();

		setTimeout(() => {
			_input.select();
		});
	};
	const stop = () => {
		started = false;
		clearTimeout(timeout);
	};

	const generate = () => {
		started = true;
		if (fout >= 5) {
			message = "Game over!";
			started = false;
			return;
		}

		let l = Math.floor(Math.random() * Math.floor(max));
		let r = Math.floor(Math.random() * Math.floor(max));

		if (sign === "+") result = l + r;
		else if (sign === "-") result = l - r;
		else if (sign === "*") result = l * r;
		else result = l + r;

		if (result < max && result > min) {
			left = l;
			right = r;

			timeout = setTimeout(() => {
				submit({ key: "Enter", target: { value: _input.value } });
			}, timeoutFactor);
		} else {
			generate();
		}
	};
	const submit = (e) => {
		if (e.key == "Enter") {
			disabled = true;

			if (timeout) clearTimeout(timeout);

			// reset status and exercise
			setTimeout(() => {
				// set the sign
				let signIndex = Math.floor(Math.random() * signs.length);
				sign = signs[signIndex] || "+";

				generate();
				if (started) message = undefined;
				disabled = false;
				status = "neutral";

				if (_input) {
					_input.value = "";
					setTimeout(() => {
						_input.select();
					});
				}
			}, 2000);

			let val = +e.target.value;
			if (val === result) {
				status = "correct";
				message = "Goed gedaan!";
				goed++;
				if (goed % 2 === 0) {
					max = +max + +dif;
				}
			} else {
				status = "incorrect";
				message = "Jammer: " + result;
				fout++;
			}
		}
	};

	const showSettings = () => {
		settingsShown = !settingsShown;
	};

	onMount(() => {
		// set the sign
		let signIndex = Math.floor(Math.random() * signs.length);
		sign = signs[signIndex] || "+";
	});
</script>

<style>
	:root {
		font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
			Oxygen, Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif;
	}
	.main {
		height: 150px;
		text-align: center;
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		font-size: 70px;
	}
	.main input {
		font-size: 70px;
		border: none;
		width: 140px;
	}
	.main input:hover,
	.main input:focus,
	.main input:active,
	.main input:focus-visible,
	.main input:visited {
		border: none;
		outline: none;
	}
	.neutral,
	.neutral input {
		color: black;
	}
	.correct,
	.correct input {
		color: green;
	}
	.incorrect,
	.incorrect input {
		color: red;
	}

	.settings {
		position: fixed;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);

		border: 1px solid gray;
		border-radius: 5px;
		padding: 30px;

		box-shadow: 0px 0px 15px 10px rgba(0, 0, 0, 0.25);
		-webkit-box-shadow: 0px 0px 15px 10px rgba(0, 0, 0, 0.25);
		-moz-box-shadow: 0px 0px 15px 10px rgba(0, 0, 0, 0.25);
	}

	input[type="number"]::-webkit-inner-spin-button,
	input[type="number"]::-webkit-outer-spin-button {
		-webkit-appearance: none;
		margin: 0;
	}
</style>

<main>
	<div class={'main ' + status}>
		{#if message}
			<div>{message}</div>
		{/if}
		{#if started}
			<span>{left}</span>
			<span>{sign || '+'}</span>
			<span>{right}</span>
			<span>=</span>
			<input
				{disabled}
				on:keypress={submit}
				bind:this={_input}
				type="number"
				on:blur={() => {
					if (started) _input.select();
				}} />
		{/if}
	</div>

	<div class="score">
		<div class="correct">Goed: {goed}</div>
		<div class="incorrect">Fout: {fout}</div>
		{#if !started}
			<button on:click={start}>Start!</button>
			<button on:click={showSettings}>Settings</button>
		{:else}<button on:click={stop}>Stop!</button>{/if}
	</div>

	{#if settingsShown}
		<div class="settings">
			<button on:click={showSettings}>Close</button>
			<hr />
			<div>Minimum: <input bind:value={min} /></div>
			<div>Maximum: <input bind:value={max} /></div>
			<div>Moeilijker: <input bind:value={dif} /></div>

			<div><input type="checkbox" bind:checked={plus} /> +</div>
			<div><input type="checkbox" bind:checked={minus} /> -</div>
			<div><input type="checkbox" bind:checked={times} /> *</div>
		</div>
	{/if}
</main>
