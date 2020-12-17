<script>
	import { onMount } from "svelte";
	import ProgressBar from "@okrad/svelte-progressbar";

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
	let level = 0;

	let timeout;
	let timeoutFactor = 5000;
	let interval;
	let remaining = 100;

	const start = () => {
		started = true;
		setTimeout(() => {
			if (!plus && !minus && !times) plus = true;

			signs = [];
			if (plus) signs.push("+");
			if (minus) signs.push("-");
			if (times) signs.push("*");

			generate();

			setTimeout(() => {
				_input.select();
			});

			runClock();
		}, 500);
	};

	const runClock = () => {
		interval = setInterval(() => {
			remaining = remaining - 25;
		}, timeoutFactor / 5);
	};

	const stop = () => {
		started = false;
		clearTimeout(timeout);
		clearInterval(interval);
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
			clearInterval(interval);
			disabled = true;

			if (timeout) clearTimeout(timeout);
			remaining = 100;

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
						runClock();
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
			level = Math.floor(goed / 5);
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
	@media only screen and (max-device-width: 768px) {
		.main {
			font-size: 20px;
		}
		.main input {
			font-size: 20px !important;
			width: 60px;
		}
	}
	.main input {
		font-size: 70px;
		border: none;
		max-width: 140px;
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
	.score {
		position: fixed;
		top: 5px;
		left: 5px;
	}
	.level {
		position: fixed;
		top: 5px;
		right: 5px;
	}
	.progress {
		position: fixed;
		bottom: 5px;
		left: 50%;
		transform: translateX(-50%);
		width: 400px;
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

	<div class="level">Level {level}</div>

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

	{#if started}
		<div class="progress">
			<ProgressBar
				series={remaining}
				valueLabel=" "
				width="100%"
				style="standard"
				addBackground={true}
				bgColor="black"
				bgFillColor="orange" />
		</div>
	{/if}
</main>
