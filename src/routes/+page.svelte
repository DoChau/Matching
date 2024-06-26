<script lang="ts">
	import { emoji } from './emoji'
	import coin1 from '../lib/images/Beon-Coin-Gold.png'
	import coin2 from '../lib/images/Beon-Coin-Silver.png'
	import coin3 from '../lib/images/Beon-Coin-Bronze.png'

	const coin = [
		coin1,
		coin2,
		coin3
	]
	type State = 'start' | 'playing' | 'paused' | 'won' | 'lost'

	let state: State = 'start'
	let size = 20
	let grid = createGrid()
	let maxMatches = grid.length / 2
	let selected: number[] = []
	let matches: string[] = []
	let timerId: number | null = null
	let time = 60
	coin.forEach(item => {
		emoji.push=(item)
	})
	function createGrid() {
		// only want unique cards
		let cards = new Set<string>()
		// half because we duplicate the cards
		let maxSize = size / 2

		while (cards.size < maxSize) {
			// pick random emoji
			const randomIndex = Math.floor(Math.random() * emoji.length)
			cards.add(emoji[randomIndex])
		}

		// duplicate and shuffle cards
		return shuffle([...cards, ...cards])
	}

	function shuffle<Items>(array: Items[]) {
		return array.sort(() => Math.random() - 0.5)
	}

	function startGameTimer() {
		function countdown() {
			state !== 'paused' && (time -= 1)
		}
		timerId = setInterval(countdown, 1000)
	}

	function selectCard(cardIndex: number) {
		selected = selected.concat(cardIndex)
	}

	function matchCards() {
		// array destructuring can have any name for the values
		const [first, second] = selected

		if (grid[first] === grid[second]) {
			matches = matches.concat(grid[first])
		}

		// clear selected
		setTimeout(() => (selected = []), 300)
	}

	function pauseGame(e: KeyboardEvent) {
		if (e.key === 'Escape') {
			switch (state) {
				case 'playing':
					state = 'paused'
					break
				case 'paused':
					state = 'playing'
					break
			}
		}
	}

	function resetGame() {
		timerId && clearInterval(timerId)
		grid = createGrid()
		maxMatches = grid.length / 2
		selected = []
		matches = []
		timerId = null
		time = 60
	}

	export function gameWon() {
		state = 'won'
		resetGame()
	}

	export function gameLost() {
		state = 'lost'
		resetGame()
	}

	$: if (state === 'playing') {
		//	in case you pause the game
		!timerId && startGameTimer()
	}

	$: selected.length === 2 && matchCards()
	$: maxMatches === matches.length && gameWon()
	$: time === 0 && gameLost()


</script>

<svelte:window on:keydown={pauseGame} />
<body>

{#if state === 'start'}
	<h1>Matching game</h1>
	<button on:click={() => (state = 'playing')}>Play</button>
{/if}

{#if state === 'paused'}
	<h1>Game paused</h1>
{/if}

{#if state === 'playing'}
	<div class="logo">
		{#each coin as coin}
			<img src={coin} alt="coin" width="100px"/>
		{/each}
	</div>
	<div class="progress">
		<h1 class="timer" class:pulse={time <= 5}>
			{time}
		</h1>
	</div>
	
	<div class="cards">
		{#each grid as card, cardIndex}
			{@const isSelected = selected.includes(cardIndex)}
			{@const isSelectedOrMatch =
				selected.includes(cardIndex) || matches.includes(card)}
			{@const match = matches.includes(card)}

			<button
				class="card"
				class:selected={isSelected}
				class:flip={isSelectedOrMatch}
				disabled={isSelectedOrMatch}
				on:click={() => selectCard(cardIndex)}
			>
				<div class="back" class:match>
					{card}
				</div>
			</button>
		{/each}
	</div>

	<div class="matches">
		{#each matches as card}
			<div>{card}</div>
		{/each}
	</div>

{/if}

{#if state === 'lost'}
	<h1>You lost! 💩</h1>
	<button on:click={() => (state = 'playing')}>Play again</button>
{/if}

{#if state === 'won'}
	<h1>You win! 🎉</h1>
	<button on:click={() => (state = 'playing')}>Play again</button>
	
	<div class="pyro">
		<script src="https://unpkg.com/@dotlottie/player-component@latest/dist/dotlottie-player.mjs" type="module"></script>
		<dotlottie-player src="https://lottie.host/16d47f14-fa4c-464c-87b5-2bedfc307e96/POP0jGqTWe.json" background="transparent" speed="0.5" style="width: 800px; height: 800px" direction="1" playMode="normal" loop autoplay></dotlottie-player>	
	</div>

{/if}
</body>

<style>
	.cards {
		display: grid;
		max-width: 90%;
		align-items: center;
		justify-content: center;
		grid-template-columns: repeat(5, 1fr);
		gap: 1rem;
		background-color: var(--bg-2);
		padding: 2rem;
		//border-radius: 1rem;
		
		background: linear-gradient(45deg, var(--primary) 0%, var(--secondary) 100%);
		animation: morph 8s ease-in-out infinite;
		border-radius: 60% 40% 30% 70% / 60% 30% 70% 40%;
		transition: all 1s ease-in-out;
		z-index: 5;
		
	}

	.card {
		height: 15vw;
		width: 15vw;
		font-size: 3rem;
		background-color: var(--bg-3);
		transition: rotate 0.3s ease-out;
		transform-style: preserve-3d;
		
		border: 1px solid rgba(255, 255, 255, 0.2);

		&.selected {
			//border: 4px solid var(--border);
			box-shadow: 0 0 10px 5px var(--border), 0 0 10px var(--border), 0 0 15px var(--border), 0 0 20px var(--border);
		}


		&.flip {
			rotate: y 180deg;
			pointer-events: none;
		}

		& .back {
			position: absolute;
			inset: 0;
			display: grid;
			place-content: center;
			backface-visibility: hidden;
			rotate: y 180deg;
		}

		& .match {
			transition: opacity 0.3s ease-out;
			opacity: 0.4;
		}
	}

	.matches {
		display: flex;
		gap: 1rem;
		margin-block: 2rem;
		font-size: 2rem;
	}
	.logo{
		position: relative;
		display: flex;
		align-items: flex-end;
		height: 20vh;

	}
	.progress{
		position: relative;
		justify-content: center;		
	}
	.timer {
		
		transition: color 0.3s ease;
		width:80px;
		height:80px;
	}

	.pulse {
		
		color: var(--pulse);
		width:80px;
		height:80px;
		animation: pulse 1s infinite ease;
	}
	@keyframes pulse {
		to {
			scale: 1.4;
		}
	}

	@keyframes morph {
		0% {
			border-radius:  60% 40% 30% 70% / 60% 30% 70% 40%;
			background: linear-gradient(45deg, var(--primary) 0%, var(--secondary) 100%);
		} 
		
		50% {
			border-radius:  30% 60% 70% 40% / 50% 60% 30% 60%;
			background: linear-gradient(45deg, var(--third) 0%, var(--secondary) 100%);
		}
		
		100% {
			border-radius:  60% 40% 30% 70% / 60% 30% 70% 40%;
			background: linear-gradient(45deg, var(--primary) 0%, var(--secondary) 100%);
		} 
	}

</style>
