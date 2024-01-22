<script lang="ts">
	import { emoji } from './emoji'
	type State = 'start' | 'playing' | 'paused' | 'won' | 'lost'

	let state: State = 'start'
	let size = 20
	let grid = createGrid()
	let maxMatches = grid.length / 2
	let selected: number[] = []
	let matches: string[] = []
	let timerId: number | null = null
	let time = 60

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

	function gameWon() {
		state = 'won'
		resetGame()
	}

	function gameLost() {
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

{#if state === 'start'}
	<h1>Matching game</h1>
	<button on:click={() => (state = 'playing')}>Play</button>
{/if}

{#if state === 'paused'}
	<h1>Game paused</h1>
{/if}

{#if state === 'playing'}
	<div class="progress">
		<h1 class="timer" class:pulse={time <= 10}>
			{time}
		</h1>

		<div class="timerround" class:pulseround={time <= 10} style="--duration: 60;--size: 150;">
			<div class="mask"></div>
		</div>
	</div>

	<div class="matches">
		{#each matches as card}
			<div>{card}</div>
		{/each}
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
{/if}

{#if state === 'lost'}
	<h1>You lost! 💩</h1>
	<button on:click={() => (state = 'playing')}>Play again</button>
{/if}

{#if state === 'won'}
	<h1>You win! 🎉</h1>
	<button on:click={() => (state = 'playing')}>Play again</button>
	
	//FIREWORK
	<div class="pyro">
		<div class="before"></div>
		<div class="after"></div>
	</div>

{/if}

<style>
	.cards {
		display: grid;
		grid-template-columns: repeat(5, 1fr);
		gap: 1rem;
		background-color: var(--bg-2);
		padding: 5rem;
		//border-radius: 1rem;
		

		background: linear-gradient(45deg, var(--primary) 0%, var(--secondary) 100%);
		animation: morph 8s ease-in-out infinite;
		border-radius: 60% 40% 30% 70% / 60% 30% 70% 40%;
		transition: all 1s ease-in-out;
z-index: 5;
		
	}

	.card {
		height: 140px;
		width: 140px;
		font-size: 4rem;
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
		font-size: 3rem;
	}

	.progress{
		display: flex;
		justify-content: center;		
	}
	.timer {
		transition: color 0.3s ease;
		width:80px;
		height:80px;
	}

	.timerround {
		background: -webkit-linear-gradient(left, skyBlue 50%, #eee 50%);
		border-radius: 100%;
		height: calc(var(--size) * 1px);
		width: calc(var(--size) * 1px);
		margin-top: -30px;
		margin-left: -115px;
		-webkit-animation: timeround calc(var(--duration) * 1s) steps(1000, start) infinite;
		-webkit-mask: radial-gradient(transparent 50%,#000 50%);
		mask: radial-gradient(transparent 50%,#000 50%);
	}
	.mask {
		border-radius: 100% 0 0 100% / 50% 0 0 50%;
		height: 100%;
		left: 0;
		position: absolute;
		top: 0;
		width: 50%;
		-webkit-animation: mask calc(var(--duration) * 1s) steps(500, start) infinite;
		-webkit-transform-origin: 100% 50%;
	}
	
	.pulse {
		color: var(--pulse);
		width:80px;
		height:80px;
		animation: pulse 1s infinite ease;
	}
	.pulseround {
		display:none;
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

	@-webkit-keyframes timeround {
		100% {
			-webkit-transform: rotate(360deg);
		}
	}
	@-webkit-keyframes mask {
		0% {
			background: #eee;
			-webkit-transform: rotate(0deg);
		}
		50% {
			background: #eee;
			-webkit-transform: rotate(-180deg);
		}
		50.01% {
			background: skyBlue;
			-webkit-transform: rotate(0deg);
		}
		100% {
			background: skyBlue;
			-webkit-transform: rotate(-180deg);
		}
	}
	

</style>
