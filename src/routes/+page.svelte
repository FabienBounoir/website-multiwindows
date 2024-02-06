<script>
	import { onMount } from 'svelte';
	import background from '$lib/images/background.jpg';
	let style = '';
	let styleSvg = '';

	let Id = generateId();

	let center = {
		x: 0,
		y: 0,
		color: 'transparent'
	};

	let displayTutorial = false;

	let allCirclesCoords = [];
	let otherCirclesInstance = [];

	onMount(() => {
		console.log('ID', Id);
		animate();

		center = {
			x: window.innerWidth / 2,
			y: window.innerHeight / 2,
			color: `#${Math.floor(Math.random() * 16777215).toString(16)}`
		};

		//clear local storage for timestamp upper 3s
		for (let i = 0; i < localStorage.length; i++) {
			let key = localStorage.key(i);
			if (key.includes('Coord-')) {
				const element = JSON.parse(localStorage.getItem(key));
				if (Date.now() - element.timeStamp > 2000) {
					localStorage.removeItem(key);
				}
			} else {
				localStorage.removeItem(key);
			}
		}

		window.onbeforeunload = () => {
			localStorage.removeItem(`Coord-${Id}`);
		};
	});

	//request frame
	function animate() {
		style = `top: ${window.screenY * -1}px; left: ${window.screenX * -1}px;`;
		styleSvg = `top: ${window.screenY * -1}px; left: ${window.screenX * -1}px; width: ${window.innerWidth}px; height: ${window.innerHeight}px;`;

		center = {
			x: window.innerWidth / 2,
			y: window.innerHeight / 2,
			color: center.color
		};

		//update local storage
		let coords = [];

		for (let i = 0; i < localStorage.length; i++) {
			let key = localStorage.key(i);
			if (key.includes('Coord-')) {
				const element = JSON.parse(localStorage.getItem(key));
				if (Date.now() - element.timeStamp < 1000) {
					coords.push(element);
				}
			}
		}

		displayTutorial = coords && coords.length <= 1;

		otherCirclesInstance.forEach((c) => {
			c.remove();
		});

		allCirclesCoords = [];

		//render the other circles
		coords.forEach((c) => {
			if (c.id !== Id) {
				let circle = document.createElement('div');
				circle.style.position = 'absolute';
				circle.style.top = `${c.y - window.screenY}px`;
				circle.style.left = `${c.x - window.screenX}px`;
				circle.style.backgroundColor = c.color;
				circle.style.width = '20px';
				circle.style.height = '20px';
				circle.style.borderRadius = '50%';
				document.body.appendChild(circle);

				otherCirclesInstance.push(circle);
			}

			allCirclesCoords.push({
				id: c.id,
				x: c.x - window.screenX + 10,
				y: c.y - window.screenY + 10,
				color: c.color
			});
		});

		localStorage.setItem(
			'Coord-' + Id,
			JSON.stringify({
				id: Id,
				x: window.innerWidth / 2 + window.screenX,
				y: window.innerHeight / 2 + window.screenY,
				color: center.color,
				timeStamp: Date.now()
			})
		);

		requestAnimationFrame(animate);
	}

	function generateId() {
		return Math.random().toString(36).substr(2, 9) + Math.random().toString(36).substr(2, 9);
	}
</script>

<svelte:head>
	<title>Multi windows site</title>
	<meta name="description" content="Demo for multi windows site dev by @Bouns" />
</svelte:head>

<section>
	<img src={background} alt="background" {style} />
	<div
		style="position: absolute; top: {center.y}px; left: {center.x}px; background-color: {center.color}; width: 20px; height: 20px; border-radius: 50%;"
	/>

	<svg id="svg-container" {styleSvg}>
		{#each allCirclesCoords as c, i (c.id)}
			{#if i < allCirclesCoords.length - 1}
				<line
					x1={c.x}
					y1={c.y}
					x2={allCirclesCoords[i + 1].x}
					y2={allCirclesCoords[i + 1].y}
					stroke="#0075A2"
					stroke-width="2"
				/>
			{:else}
				<line
					x1={c.x}
					y1={c.y}
					x2={allCirclesCoords[0].x}
					y2={allCirclesCoords[0].y}
					stroke="#0075A2"
					stroke-width="2"
				/>
			{/if}
		{/each}
	</svg>

	{#if displayTutorial}
		<div class="tutorial">
			<h1>How to use</h1>
			<p>
				Open this page in multiple windows and see the <a
					href={window.location.href}
					target="_blank"
					rel="noopener noreferrer">magic</a
				>
			</p>
		</div>
	{/if}
</section>

<style>
	svg {
		position: absolute;
		top: 0;
		left: 0;
		width: 100vw;
		height: 100vh;
	}

	section {
		display: flex;
		justify-content: center;
		align-items: center;
		height: 100vh;
	}

	img {
		position: absolute;
		top: 0;
		left: 0;
		z-index: -1;
	}

	#svg-container {
		position: absolute;
		top: 0;
		left: 0;
		width: 100vw;
		height: 100vh;
	}

	.tutorial {
		z-index: 99;
		background-color: white;
		padding: 20px;
		border-radius: 10px;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 10px;
	}

	h1,
	p {
		margin: 0;
	}

	button {
		position: absolute;
		bottom: 20px;
		right: 20px;
		z-index: 9999;
	}
</style>
