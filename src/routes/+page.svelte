<script>
	import { onMount } from 'svelte';
	import background from '$lib/images/background.jpg';
	let style = '';

	let center = {
		x: 0,
		y: 0,
		color: 'transparent'
	};

	let displayTutorial = false;

	let increment = 0;

	const otherCircles = [];

	let Id = generateId();

	onMount(() => {
		console.log('window', window);
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
		increment++;

		style = `top: ${window.screenY * -1}px; left: ${window.screenX * -1}px;`;
		center = {
			x: window.innerWidth / 2,
			y: window.innerHeight / 2,
			color: center.color
		};

		//update local storage
		let coords = [];

		for (let i = 0; i < localStorage.length; i++) {
			let key = localStorage.key(i);
			console.log('localStorage.key(i)', localStorage.key(i));
			if (key.includes('Coord-') && key !== `Coord-${Id}`) {
				const element = JSON.parse(localStorage.getItem(key));
				console.log('element', element, element.timeStamp);
				if (Date.now() - element.timeStamp < 1000) {
					coords.push(element);
				}
			}
		}

		console.log('coords', coords);

		displayTutorial = coords && coords.length < 1;

		//Clear the other circles
		otherCircles.forEach((c) => {
			c.remove();
		});

		//render the other circles
		coords.forEach((c) => {
			if (c.id !== Id) {
				let circle = document.createElement('div');
				circle.style.position = 'absolute';
				circle.style.top = `${c.y - window.screenY}px`;
				circle.style.left = `${c.x - window.screenX}px`;
				circle.style.backgroundColor = c.color;
				circle.style.width = '10px';
				circle.style.height = '10px';
				circle.style.borderRadius = '50%';
				document.body.appendChild(circle);

				otherCircles.push(circle);
			}
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
		style="position: absolute; top: {center.y}px; left: {center.x}px; background-color: {center.color}; width: 10px; height: 10px; border-radius: 50%;"
	/>

	<!-- <svg {style} id="line"> </svg> -->

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
		background-color: red;
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

	#line {
		position: absolute;
		top: 0;
		left: 0;
		width: 100vw;
		height: 100vh;
		border: 30px solid red;
	}
</style>
