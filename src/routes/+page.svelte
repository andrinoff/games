<script lang="ts">
	import { onMount, onDestroy } from 'svelte';

	// --- Type Definition for a Game ---
	interface Game {
		emoji: string;
		title: string;
		description: string;
		color: 'pink' | 'cyan' | 'amber'; // Using literal types for specific colors
		endpoint?: string; // Optional endpoint for redirection
	}

	// Array of games to display, making it easy to add more!
	const games: Game[] = [
		{
			emoji: '😈',
			title: 'Truth or Dare',
			description: 'The classic party starter. Will you tell all, or take on a challenge?',
			color: 'pink',
			endpoint: 'truth-or-dare'
		}
		// {
		// 	emoji: '🤫',
		// 	title: 'Never Have I Ever',
		// 	description: "Discover your friends' wildest secrets with this revealing classic.",
		// 	color: 'cyan',
		// 	endpoint: 'never-have-i-ever'
		// },
		// {
		// 	emoji: '🕵️',
		// 	title: 'Spy',
		// 	description: "One spy, many innocents. Can you find the impostor before it's too late?",
		// 	color: 'amber',
		// 	endpoint: 'spy'
		// }
	];

	let gameCards: NodeListOf<HTMLElement>;

	function handleMouseMove(e: MouseEvent) {
		const card = e.currentTarget as HTMLElement;
		const { left, top, width, height } = card.getBoundingClientRect();

		const x = e.clientX - left;
		const y = e.clientY - top;

		const rotateX = (y - height / 2) / 12;
		const rotateY = (x - width / 2) / -12;
		card.style.transform = `perspective(1000px) rotateX(${rotateX}deg) rotateY(${rotateY}deg) scale(1.05)`;
	}

	function handleMouseLeave(e: MouseEvent) {
		const card = e.currentTarget as HTMLElement;
		card.style.transform = 'perspective(1000px) rotateX(0) rotateY(0) scale(1)';
	}

	onMount(() => {
		gameCards = document.querySelectorAll('.game-card-interactive');
		gameCards.forEach((card) => {
			card.addEventListener('mousemove', handleMouseMove);
			card.addEventListener('mouseleave', handleMouseLeave);
		});
	});

	onDestroy(() => {
		if (gameCards) {
			gameCards.forEach((card) => {
				card.removeEventListener('mousemove', handleMouseMove);
				card.removeEventListener('mouseleave', handleMouseLeave);
			});
		}
	});
</script>

<div class="flex min-h-screen w-full flex-col items-center justify-center p-4 sm:p-6 lg:p-8">
	<header class="mb-12 text-center">
		<h1 class="text-glow text-5xl font-bold tracking-tighter md:text-7xl">
			Nomina <span class="text-pink-500">Games</span>
		</h1>
		<p class="mt-4 max-w-2xl text-lg text-gray-300 md:text-xl">
			Your portal to instant fun. Pick a game and let the good times roll!
		</p>
	</header>

	<main class="grid w-full max-w-6xl grid-cols-1 gap-6 sm:grid-cols-2 md:gap-8 lg:grid-cols-3">
		{#each games as game}
			<div class="game-card-interactive">
				<div
					class="card-content card-glow-border flex h-full flex-col items-center rounded-2xl bg-slate-800/50 p-6 text-center backdrop-blur-sm {game.color}"
				>
					<div class="mb-4 text-5xl">{game.emoji}</div>
					<h2 class="mb-2 text-2xl font-bold">{game.title}</h2>
					<p class="mb-6 flex-grow text-gray-400">{game.description}</p>
					<button
						class="play-button w-full rounded-full px-8 py-3 font-bold shadow-lg"
						on:click={() => (window.location.href = `/${game.endpoint}`)}>Play Now</button
					>
				</div>
			</div>
		{/each}
	</main>
</div>

<style>
	@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;600;700&display=swap');

	:global(body) {
		font-family: 'Poppins', sans-serif;
		background-color: #1a1a2e;
		background-image: radial-gradient(circle at 1px 1px, #3a3a5e 1px, transparent 0);
		background-size: 20px 20px;
		color: white;
		overflow-x: hidden;
	}

	.text-glow {
		text-shadow:
			0 0 8px rgba(255, 255, 255, 0.3),
			0 0 20px rgba(236, 72, 153, 0.4);
	}

	.text-pink-500 {
		color: #ec4899;
	}

	.game-card-interactive {
		transform-style: preserve-3d;
		transition: transform 0.1s ease-out;
		will-change: transform;
	}

	.card-glow-border {
		border: 2px solid transparent;
		transition: all 0.3s ease;
	}

	.game-card-interactive:hover .card-glow-border.pink {
		border-color: rgba(236, 72, 153, 0.7);
		box-shadow:
			0 0 20px rgba(236, 72, 153, 0.5),
			inset 0 0 10px rgba(236, 72, 153, 0.3);
	}
	.game-card-interactive:hover .card-glow-border.cyan {
		border-color: rgba(6, 182, 212, 0.7);
		box-shadow:
			0 0 20px rgba(6, 182, 212, 0.5),
			inset 0 0 10px rgba(6, 182, 212, 0.3);
	}
	.game-card-interactive:hover .card-glow-border.amber {
		border-color: rgba(245, 158, 11, 0.7);
		box-shadow:
			0 0 20px rgba(245, 158, 11, 0.5),
			inset 0 0 10px rgba(245, 158, 11, 0.3);
	}

	.card-glow-border.pink .play-button {
		background-color: #db2777;
		box-shadow: 0 4px 14px 0 rgb(219 39 119 / 39%);
	}
	.card-glow-border.pink .play-button:hover {
		background-color: #be185d;
	}

	.card-glow-border.cyan .play-button {
		background-color: #0891b2;
		box-shadow: 0 4px 14px 0 rgb(8 145 178 / 39%);
	}
	.card-glow-border.cyan .play-button:hover {
		background-color: #0e7490;
	}

	.card-glow-border.amber .play-button {
		background-color: #d97706;
		box-shadow: 0 4px 14px 0 rgb(217 119 6 / 39%);
	}
	.card-glow-border.amber .play-button:hover {
		background-color: #b45309;
	}

	.play-button {
		transition: background-color 0.3s ease;
	}
</style>
