<script>
	import { onMount } from 'svelte';

	let now = $state(new Date());

	onMount(() => {
		let timeout;

		const tick = () => {
			now = new Date();

			const msIntoMinute =
				now.getSeconds() * 1000 + now.getMilliseconds();

			timeout = setTimeout(tick, 60_000 - msIntoMinute + 50);
		};

		tick();

		return () => clearTimeout(timeout);
	});

	const currentHour = $derived(now.getHours() % 12 || 12);
	const currentMinute = $derived(now.getMinutes());

	const roundedMinutes = $derived(Math.floor(currentMinute / 5) * 5);
	const minuteRemainder = $derived(currentMinute % 5);

	const nextHour = $derived(currentHour === 12 ? 1 : currentHour + 1);

	const matrix = [
		['ה', 'ש', 'ע', 'ה', 'ק', 'ד', 'נ', 'ז', 'ט', 'ג', 'פ'],
		['ע', 'ש', 'ר', 'י', 'ם', 'ו', 'ח', 'מ', 'י', 'ש', 'ה'],
		['ר', 'ב', 'ע', 'נ', 'ע', 'ש', 'ר', 'ה', 'ט', 'ל', 'ם'],
		['א', 'ח', 'ת', 'ע', 'ש', 'ר', 'ה', 'ש', 'ל', 'ו', 'ש'],
		['ש', 'ת', 'י', 'ם', 'ע', 'ש', 'ר', 'ה', 'ח', 'מ', 'ש'],
		['ש', 'ת', 'י', 'י', 'ם', 'ש', 'מ', 'ו', 'נ', 'ה', 'ד'],
		['א', 'ר', 'ב', 'ע', 'ש', 'ב', 'ע', 'ת', 'ש', 'ע', 'ק'],
		['ש', 'ש', 'ע', 'ש', 'ר', 'ז', 'ט', 'ד', 'נ', 'ק', 'ם'],
		['ו', 'ע', 'ש', 'ר', 'י', 'ם', 'ק', 'ו', 'ר', 'ב', 'ע'],
		['ו', 'ח', 'מ', 'י', 'ש', 'ה', 'ו', 'ע', 'ש', 'ר', 'ה'],
		['ו', 'ח', 'צ', 'י', 'ט', 'ם', 'ז', 'ד', 'ג', 'ק', 'נ']
	];

	const words = {
		prefix: { row: 0, col: 0, len: 4, text: 'השעה' },

		'to:25': { row: 1, col: 0, len: 11, text: 'עשרים וחמישה' },
		'to:20': { row: 1, col: 0, len: 5, text: 'עשרים' },
		'to:5': { row: 1, col: 6, len: 5, text: 'חמישה' },
		'to:15': { row: 2, col: 0, len: 3, text: 'רבע' },
		'to:10': { row: 2, col: 4, len: 4, text: 'עשרה' },
		to: { row: 2, col: 9, len: 1, text: 'ל' },

		'hour:11': { row: 3, col: 0, len: 7, text: 'אחת עשרה' },
		'hour:1': { row: 3, col: 0, len: 3, text: 'אחת' },
		'hour:3': { row: 3, col: 7, len: 4, text: 'שלוש' },
		'hour:12': { row: 4, col: 0, len: 8, text: 'שתים עשרה' },
		'hour:5': { row: 4, col: 8, len: 3, text: 'חמש' },
		'hour:2': { row: 5, col: 0, len: 5, text: 'שתיים' },
		'hour:8': { row: 5, col: 5, len: 5, text: 'שמונה' },
		'hour:4': { row: 6, col: 0, len: 4, text: 'ארבע' },
		'hour:7': { row: 6, col: 4, len: 3, text: 'שבע' },
		'hour:9': { row: 6, col: 7, len: 3, text: 'תשע' },
		'hour:6': { row: 7, col: 0, len: 2, text: 'שש' },
		'hour:10': { row: 7, col: 2, len: 3, text: 'עשר' },

		'past:20': { row: 8, col: 0, len: 6, text: 'ועשרים' },
		'past:15': { row: 8, col: 7, len: 4, text: 'ורבע' },
		'past:5': { row: 9, col: 0, len: 6, text: 'וחמישה' },
		'past:10': { row: 9, col: 6, len: 5, text: 'ועשרה' },
		'past:30': { row: 10, col: 0, len: 4, text: 'וחצי' }
	};

	const pastMinutes = {
		5: ['past:5'],
		10: ['past:10'],
		15: ['past:15'],
		20: ['past:20'],
		25: ['past:20', 'past:5'],
		30: ['past:30']
	};

	const toMinutes = {
		5: ['to:5'],
		10: ['to:10'],
		15: ['to:15'],
		20: ['to:20'],
		25: ['to:25']
	};

	const tokens = $derived.by(() => {
		if (roundedMinutes === 0) {
			return ['prefix', `hour:${currentHour}`];
		}

		if (roundedMinutes <= 30) {
			return [
				'prefix',
				`hour:${currentHour}`,
				...pastMinutes[roundedMinutes]
			];
		}

		return [
			'prefix',
			...toMinutes[60 - roundedMinutes],
			'to',
			`hour:${nextHour}`
		];
	});

	const activeCells = $derived.by(() => {
		const cells = new Set();

		for (const token of tokens) {
			const word = words[token];

			if (!word) continue;

			for (let i = 0; i < word.len; i++) {
				cells.add(`${word.row}-${word.col + i}`);
			}
		}

		return cells;
	});

	function isActive(row, col) {
		return activeCells.has(`${row}-${col}`);
	}

	const spoken = $derived.by(() => {
		if (roundedMinutes === 0) {
			return `השעה ${words[`hour:${currentHour}`].text}`;
		}

		if (roundedMinutes <= 30) {
			const minutes = pastMinutes[roundedMinutes]
				.map((token) => words[token].text)
				.join(' ');

			return `השעה ${words[`hour:${currentHour}`].text} ${minutes}`;
		}

		const minutes = toMinutes[60 - roundedMinutes]
			.map((token) => words[token].text)
			.join(' ');

		return `השעה ${minutes} ל${words[`hour:${nextHour}`].text}`;
	});

	function isDotActive(index) {
		return index < minuteRemainder;
	}
</script>

<svelte:head>
	<title>QLOCKTWO Hebrew</title>

	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link
		rel="preconnect"
		href="https://fonts.gstatic.com"
		crossorigin="anonymous"
	/>
	<link
		href="https://fonts.googleapis.com/css2?family=Assistant:wght@200;300;400&display=swap"
		rel="stylesheet"
	/>
</svelte:head>

<div class="clock" role="img" aria-label={spoken}>
	<div class="dot dot-top-left" class:active={isDotActive(0)}></div>
	<div class="dot dot-top-right" class:active={isDotActive(1)}></div>
	<div class="dot dot-bottom-right" class:active={isDotActive(2)}></div>
	<div class="dot dot-bottom-left" class:active={isDotActive(3)}></div>

	<div class="matrix" dir="rtl" aria-hidden="true">
		{#each matrix as row, rowIndex}
			{#each row as letter, colIndex}
				<div class="letter" class:active={isActive(rowIndex, colIndex)}>
					{letter}
				</div>
			{/each}
		{/each}
	</div>
</div>

<style>
	:global(*) {
		box-sizing: border-box;
	}

	:global(body) {
		margin: 0;
		background: #080808;
	}

	.clock {
		position: relative;

		width: min(90vw, 620px);
		aspect-ratio: 1 / 1;

		margin: 40px auto;

		background: radial-gradient(
			circle at 50% 40%,
			#292929 0%,
			#1d1d1d 45%,
			#171717 100%
		);

		border: 1px solid #252525;

		box-shadow:
			0 30px 80px rgba(0, 0, 0, 0.65),
			inset 0 0 40px rgba(255, 255, 255, 0.015);

		overflow: hidden;
	}

	.clock::after {
		content: '';

		position: absolute;
		inset: 0;

		pointer-events: none;

		background: linear-gradient(
			135deg,
			rgba(255, 255, 255, 0.025),
			transparent 35%
		);

		mix-blend-mode: screen;
	}

	.matrix {
		position: absolute;

		inset: 14% 11%;

		display: grid;
		grid-template-columns: repeat(11, 1fr);
		grid-auto-rows: 1fr;

		align-items: center;
		justify-items: center;
	}

	.letter {
		position: relative;

		width: 100%;
		height: 100%;

		display: flex;
		align-items: center;
		justify-content: center;

		font-family: 'Assistant', 'Arial', sans-serif;

		font-size: clamp(18px, 3.4vw, 31px);
		font-weight: 300;

		line-height: 1;

		color: rgba(255, 255, 255, 0.2);

		transition:
			color 180ms ease,
			text-shadow 180ms ease;
	}

	.letter.active {
		color: #f4f4f4;

		text-shadow:
			0 0 5px rgba(255, 255, 255, 0.22),
			0 0 14px rgba(255, 255, 255, 0.08);
	}

	.dot {
		position: absolute;

		width: 5px;
		height: 5px;

		border-radius: 50%;

		background: rgba(255, 255, 255, 0.55);

		box-shadow: 0 0 4px rgba(255, 255, 255, 0.3);

		transition:
			background 180ms ease,
			box-shadow 180ms ease;
	}

	.dot.active {
		background: #fff;

		box-shadow:
			0 0 7px rgba(255, 255, 255, 0.85),
			0 0 14px rgba(255, 255, 255, 0.25);
	}

	.dot-top-left {
		top: 5.2%;
		left: 5.2%;
	}

	.dot-top-right {
		top: 5.2%;
		right: 5.2%;
	}

	.dot-bottom-right {
		right: 5.2%;
		bottom: 5.2%;
	}

	.dot-bottom-left {
		bottom: 5.2%;
		left: 5.2%;
	}

	@media (prefers-reduced-motion: reduce) {
		.letter,
		.dot {
			transition: none;
		}
	}

	@media (max-width: 600px) {
		.clock {
			width: 94vw;
		}

		.matrix {
			inset: 16% 8%;
		}

		.letter {
			font-size: 4.2vw;
		}
	}
</style>
