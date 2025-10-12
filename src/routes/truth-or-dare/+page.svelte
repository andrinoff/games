<script lang="ts">
	import { fly, fade } from 'svelte/transition';
	import { quintOut } from 'svelte/easing';

	type GameState = 'setup' | 'playing' | 'question';
	let gameState: GameState = 'setup';

	let players: string[] = [];
	let newPlayerName: string = '';
	let currentPlayerIndex: number = 0;

	const truths: string[] = [
		"What's the most embarrassing thing you've ever done?",
		'What is your biggest fear?',
		'Have you ever lied to get out of trouble?',
		"What's a secret you've never told anyone?",
		'Who is your secret crush?',
		"What's the most childish thing you still do?"
	];
	const dares: string[] = [
		'Do your best impression of another player.',
		'Let the group look through your phone for one minute.',
		'Speak in a funny accent for the next 3 rounds.',
		'Do 20 pushups.',
		'Sing a song chosen by the group.',
		'Post an old, embarrassing photo on social media.'
	];
	let currentQuestion: string = '';
	let questionType: 'truth' | 'dare' | null = null;

	let isTruthHovered = false;
	let isDareHovered = false;

	function generateAnimation() {
		const halfWidth = 125; // Based on button's max-width: 250px
		const halfHeight = 35; // Approximate half-height of the button
		let x_from = 0;
		let y_from = 0;

		const side = Math.floor(Math.random() * 4);

		switch (side) {
			case 0: // Top edge
				x_from = (Math.random() - 0.5) * halfWidth * 2;
				y_from = -halfHeight - 10; // Start slightly outside
				break;
			case 1: // Right edge
				x_from = halfWidth + 10;
				y_from = (Math.random() - 0.5) * halfHeight * 2;
				break;
			case 2: // Bottom edge
				x_from = (Math.random() - 0.5) * halfWidth * 2;
				y_from = halfHeight + 10;
				break;
			case 3: // Left edge
				x_from = -halfWidth - 10;
				y_from = (Math.random() - 0.5) * halfHeight * 2;
				break;
		}

		const duration = 600 + Math.random() * 400; // Random speed
		const delay = Math.random() * 200; // Random start time

		return { x_from, y_from, duration, delay };
	}

	function addPlayer() {
		const trimmedName = newPlayerName.trim();
		if (trimmedName && !players.includes(trimmedName)) {
			players = [...players, trimmedName];
			newPlayerName = '';
		}
	}

	function removePlayer(indexToRemove: number) {
		players = players.filter((_, index) => index !== indexToRemove);
	}

	function startGame() {
		if (players.length >= 2) {
			gameState = 'playing';
		} else {
			alert('You need at least two players to start!');
		}
	}

	function selectChoice(choice: 'truth' | 'dare') {
		questionType = choice;
		const sourceArray = choice === 'truth' ? truths : dares;
		currentQuestion = sourceArray[Math.floor(Math.random() * sourceArray.length)];
		gameState = 'question';
	}

	function nextTurn() {
		currentPlayerIndex = (currentPlayerIndex + 1) % players.length;
		gameState = 'playing';
	}
</script>

<div class="flex min-h-screen w-full flex-col items-center justify-center p-4">
	<div class="mx-auto w-full max-w-2xl" transition:fade={{ duration: 300 }}>
		{#if gameState === 'setup'}
			<div class="main-panel" in:fly={{ y: 50, duration: 600, easing: quintOut }}>
				<h1 class="mb-2 text-center text-4xl font-bold text-white">Player Setup</h1>
				<p class="mb-8 text-center text-gray-400">Add at least two players to begin.</p>

				<form on:submit|preventDefault={addPlayer} class="mb-6">
					<div
						class="flex w-full overflow-hidden rounded-xl border-2 border-slate-700 bg-slate-800 transition-all duration-300 focus-within:border-pink-500 focus-within:ring-2 focus-within:ring-pink-500/50"
					>
						<input
							type="text"
							class="inputPlayer"
							bind:value={newPlayerName}
							placeholder="Enter new player.."
						/>
						<button type="submit" class="btn btn-primary">Add</button>
					</div>
				</form>

				<div class="min-h-[150px] rounded-xl border border-slate-700 bg-slate-800/80 p-4">
					<h3 class="mb-4 px-2 text-lg font-bold text-gray-300">Players:</h3>
					{#if players.length === 0}
						<p class="pt-8 text-center text-gray-500">Waiting for players to join...</p>
					{:else}
						<ul class="flex flex-wrap gap-3">
							{#each players as player, i (player)}
								<li
									transition:fly={{ x: -20, duration: 300, easing: quintOut, delay: i * 50 }}
									class="flex items-center rounded-full bg-slate-700 py-1 pr-2 pl-4 text-white shadow"
								>
									<span>{player}</span>
									<button
										on:click={() => removePlayer(i)}
										class="ml-2 flex h-6 w-6 items-center justify-center rounded-full text-lg text-red-400 transition-colors hover:text-red-300"
										aria-label="Remove player"
									>
										&times;
									</button>
								</li>
							{/each}
						</ul>
					{/if}
				</div>
				<div class="mt-8 flex justify-between">
					<a href="/" class="btn btn-secondary">Back to Hub</a>
					<button on:click={startGame} disabled={players.length < 2} class="btn btn-primary"
						>Start Game</button
					>
				</div>
			</div>
		{/if}

		{#if gameState === 'playing'}
			<div
				class="playing-container text-center"
				in:fly={{ y: 50, duration: 600, easing: quintOut }}
			>
				<p class="mb-2 text-2xl text-gray-300">It's your turn,</p>
				<p class="player-name mb-10 text-2xl font-bold">
					{players[currentPlayerIndex]}
				</p>
				<p class="mb-8 text-3xl font-light">Choose your fate...</p>
				<div
					class="button-container flex flex-row justify-center gap-4 sm:flex-col sm:items-center"
				>
					<div
						class="emoji-burst-container"
						role="presentation"
						on:mouseenter={() => (isTruthHovered = true)}
						on:mouseleave={() => (isTruthHovered = false)}
					>
						<button on:click={() => selectChoice('truth')} class="choice-btn truth-btn"
							>truth</button
						>
						{#if isTruthHovered}
							{#each Array(12) as _}
								{@const anim = generateAnimation()}
								<span
									class="emoji"
									style="--x-from: {anim.x_from}px; --y-from: {anim.y_from}px; --duration: {anim.duration}ms; --delay: {anim.delay}ms;"
									>😇</span
								>
							{/each}
						{/if}
					</div>

					<div
						class="emoji-burst-container"
						role="presentation"
						on:mouseenter={() => (isDareHovered = true)}
						on:mouseleave={() => (isDareHovered = false)}
					>
						<button on:click={() => selectChoice('dare')} class="choice-btn dare-btn">dare</button>
						{#if isDareHovered}
							{#each Array(12) as _}
								{@const anim = generateAnimation()}
								<span
									class="emoji"
									style="--x-from: {anim.x_from}px; --y-from: {anim.y_from}px; --duration: {anim.duration}ms; --delay: {anim.delay}ms;"
									>😈</span
								>
							{/each}
						{/if}
					</div>
				</div>
			</div>
		{/if}

		{#if gameState === 'question'}
			<div class="text-center" in:fly={{ y: 50, duration: 600, easing: quintOut }}>
				<div class="question-box">
					<h1
						class="mb-6 text-3xl font-bold tracking-widest uppercase"
						class:text-truth={questionType === 'truth'}
						class:text-dare={questionType === 'dare'}
					>
						{questionType}
					</h1>
					<p class="text-2xl leading-relaxed text-white">{currentQuestion}</p>
				</div>
				<button on:click={nextTurn} class="btn btn-primary mt-8 text-xl">Next Player →</button>
			</div>
		{/if}
	</div>
</div>

<style>
	@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;600;700&display=swap');

	:global(body) {
		font-family: 'Poppins', sans-serif;
		background-color: #111827;
		background-image: radial-gradient(circle at 1px 1px, #374151 1px, transparent 0);
		background-size: 25px 25px;
		color: white;
		overflow-x: hidden;
	}
	.playing-container {
		background: #1f293799;
		backdrop-filter: blur(8px);
		border-radius: 1.5rem;
		padding: 2rem;
		border: 1px solid #374151;
		box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.25);
	}
	.button-container {
		margin-top: 1rem;
		gap: 1.5rem;
	}
	.inputPlayer {
		flex: 1;
		padding: 1rem;
		background: transparent;
		border: none;
		color: white;
		font-size: 1rem;
		border: 1px solid white;
		border-color: white;
		border-radius: 20px 20px 20px 20px;
	}
	.main-panel {
		background: #1f293799;
		backdrop-filter: blur(8px);
		border-radius: 1.5rem;
		padding: 2rem;
		border: 1px solid #374151;
		box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.25);
	}
	.player-name {
		color: #22d3ee;
		text-shadow: 0 0 12px rgba(34, 211, 238, 0.5);
	}
	.text-truth {
		color: #f472b6;
	}
	.text-dare {
		color: #fbbf24;
	}

	.btn {
		font-weight: 600;
		padding: 0.75rem 2rem;
		border-radius: 9999px;
		transition: all 0.2s ease-in-out;
		transform-origin: center;
	}
	.btn:active {
		transform: scale(0.95);
	}
	.btn-primary {
		background-color: #0891b2;
		color: white;
	}
	.btn-primary:hover {
		background-color: #0e7490;
	}
	.btn-primary:disabled {
		background-color: #4b5563;
		color: #9ca3af;
		cursor: not-allowed;
	}
	.btn-secondary {
		background-color: #4b5563;
		color: white;
	}
	.btn-secondary:hover {
		background-color: #6b7280;
	}
	.choice-btn {
		font-size: 1.5rem;
		font-weight: 700;
		color: white;
		padding: 1.25rem 0;
		width: 100%;
		border-radius: 1rem;
		transition: all 0.2s ease;
		border: 2px solid;
		transform: translateY(0);
	}
	.choice-btn:hover {
		box-shadow: 0 8px 20px rgba(0, 0, 0, 0.3);
	}
	.truth-btn {
		background: linear-gradient(45deg, #ec4899, #db2777);
		border-color: #ec4899;
	}
	.dare-btn {
		background: linear-gradient(45deg, #f59e0b, #d97706);
		border-color: #f59e0b;
	}
	.question-box {
		background-color: #111827;
		border-radius: 1rem;
		padding: 2rem;
		min-height: 200px;
		display: flex;
		align-items: center;
		justify-content: center;
		border: 1px solid #374151;
	}

	.emoji-burst-container {
		position: relative;
		width: 100%;
		max-width: 250px;
		overflow: hidden;
		border-radius: 1rem;
	}

	@keyframes fly-in {
		from {
			transform: translate(calc(var(--x-from) - 50%), calc(var(--y-from) - 50%)) scale(1);
			opacity: 1;
		}
		to {
			transform: translate(-50%, -50%) scale(0.2);
			opacity: 0;
		}
	}

	.emoji {
		position: absolute;
		left: 50%;
		top: 50%;
		font-size: 1.5rem;
		pointer-events: none;
		z-index: 10;
		text-shadow: 0 2px 4px rgba(0, 0, 0, 0.4);
		opacity: 0; /* Start invisible to prevent the initial flash */
		animation: fly-in ease-in forwards;
		animation-duration: var(--duration);
		animation-delay: var(--delay);
	}
</style>
