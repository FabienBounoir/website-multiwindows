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

		/**
		 * @type {Array<{id: string, x: number, y: number, color}>}
		 */
		let coords = localStorage.getItem('Coords');

		if (coords) {
			coords = JSON.parse(coords);
		} else {
			coords = [];
		}

		coords.push({
			id: Id,
			x: window.innerWidth / 2 + window.screenX,
			y: window.innerHeight / 2 + window.screenY,
			color: center.color
		});

		localStorage.setItem('Coords', JSON.stringify(coords));

		window.onbeforeunload = () => {
			console.log('unmount');
			//remove from local storage
			coords = coords.filter((c) => c.id !== Id);
			// localStorage.setItem('Coords', JSON.stringify(coords));
			localStorage.removeItem('Coords');
		};
	});

	//request frame
	function animate() {
		increment++;
		// if (increment % 120 == 0) {
		// 	console.log('animate');

		style = `top: ${window.screenY * -1}px; left: ${window.screenX * -1}px;`;
		center = {
			x: window.innerWidth / 2,
			y: window.innerHeight / 2,
			color: center.color
		};

		//update local storage
		let coords = localStorage.getItem('Coords');
		if (coords) {
			coords = JSON.parse(coords);
		} else {
			coords = [];
		}

		displayTutorial = coords && coords.length <= 1;

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

		coords = coords.map((c) => {
			if (c.id === Id) {
				return {
					id: Id,
					x: window.innerWidth / 2 + window.screenX,
					y: window.innerHeight / 2 + window.screenY,
					color: center.color
				};
			}
			return c;
		});

		if (false && otherCircles.length > 0) {
			let svg = document.getElementById('line');

			//delete svg
			svg?.remove();

			svg = document.createElementNS('http://www.w3.org/2000/svg', 'svg');
			svg.setAttribute('id', 'line');
			svg.setAttribute('style', style + ' position: absolute; width: 100vw; height: 100vh;');

			// let line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
			// line.setAttribute('stroke', '#000');
			// line.setAttribute('stroke-width', '10');

			// line.setAttribute('x1', 200);
			// line.setAttribute('y1', 400);
			// line.setAttribute('x2', 340);
			// line.setAttribute('y2', 340);

			// svg.appendChild(line);

			// line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
			// line.setAttribute('stroke', '#000');
			// line.setAttribute('stroke-width', '10');

			// line.setAttribute('x1', 340);
			// line.setAttribute('y1', 340);
			// line.setAttribute('x2', 500);
			// line.setAttribute('y2', 500);

			// svg.appendChild(line);

			// line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
			// line.setAttribute('stroke', '#000');
			// line.setAttribute('stroke-width', '10');

			// line.setAttribute('x1', 500);
			// line.setAttribute('y1', 500);
			// line.setAttribute('x2', 200);
			// line.setAttribute('y2', 400);

			// svg.appendChild(line);

			// svg.setAttribute('id', 'line');
			// svg.setAttribute('style', style + ' position: absolute; width: 100vw; height: 100vh;');
			// svg.classList.add('line');

			console.clear();

			// //relis chaque otherCircles avec la suivante
			for (let i = 0; i < otherCircles.length - 1; i++) {
				let line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
				line.setAttribute('x1', otherCircles[i].style.left.replace('px', ''));
				line.setAttribute('y1', otherCircles[i].style.top.replace('px', ''));
				line.setAttribute('stroke', '#000');
				line.setAttribute('stroke-width', '10');

				if (i < otherCircles.length - 1) {
					line.setAttribute('x2', otherCircles[i + 1].style.left.replace('px', ''));
					line.setAttribute('y2', otherCircles[i + 1].style.top.replace('px', ''));
				} else {
					line.setAttribute('x2', otherCircles[0].style.left.replace('px', ''));
					line.setAttribute('y2', otherCircles[0].style.top.replace('px', ''));
				}

				console.log(
					'line',
					line,
					line.getAttribute('x1'),
					line.getAttribute('y1'),
					line.getAttribute('x2'),
					line.getAttribute('y2')
				);

				svg.appendChild(line);
			}

			document.body.appendChild(svg);
		}

		localStorage.setItem('Coords', JSON.stringify(coords));
		// }
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

	<!-- <svg {style} id="line">
		<line x1="55" y1="100" x2="205" y2="200" stroke="#000" stroke-width="10" />
	</svg> -->

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
