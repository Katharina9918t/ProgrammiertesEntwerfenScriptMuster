<script>
	const elementWidth = 300;
	const elementHeight = 300;

	const cols = [-5, -4, -3, -2, -1, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
	const rows = [-8, -7, -6, -5, -4, -3, -2, -1, 0, 1, 2, 3, 4, 5, 6, 7, 8];

	let size4 = 10;

	// Jede Form mit benannten Punkten a-d (Richtung korrigiert: L=negativ, R=positiv)
	// const shape1L = { label: "Element 1 links", a: "0,0", b: "-100,50", c: "-100,300", d: "0,250", fill: "#742E30", stroke: "none" };
	// const shape2L = { label: "Element 2 links", a: "0,0", b: "0,-50", c: "-300,100", d: "-300,150", fill: "#7A7B73", stroke: "none" };
	// const shape3L = { label: "Element 3 links", a: "-100,100", b: "-250,175", c: "-100,250", fill: "#1E2424", stroke: "none" };
	// const shape4L = { label: "Element 4 links", a: "-100,50", b: "-300,150", c: "-250,175", d: "-100,100", fill: "#04486B", stroke: "none" };

	// const shape1R = { label: "Element 1 rechts", a: "0,0", b: "100,50", c: "100,300", d: "0,250", fill: "#9F2F23", stroke: "none" };
	// const shape2R = { label: "Element 2 rechts", a: "0,0", b: "0,-50", c: "300,100", d: "300,150", fill: "#FFAC70", stroke: "none" };
	// const shape3R = { label: "Element 3 rechts", a: "100,100", b: "250,175", c: "100,250", fill: "#1E2424", stroke: "none" };
	// const shape4R = { label: "Element 4 rechts", a: "100,50", b: "300,150", c: "250,175", d: "100,100", fill: "#B57547", stroke: "none" };

	const shape1L = {
		label: 'Element 1 links',
		ax: 0,
		ay: 0,
		bx: -100,
		by: 50,
		cx: -100,
		cy: 300,
		dx: 0,
		dy: 250,
		fill: '#742E30',
		stroke: 'none'
	};
	const shape2L = {
		label: 'Element 2 links',
		ax: 0,
		ay: 0,
		bx: 0,
		by: -50,
		cx: -300,
		cy: 100,
		dx: -300,
		dy: 150,
		fill: '#7A7B73',
		stroke: 'none'
	};
	const shape4L = {
		label: 'Element 4 links',
		ax: -100,
		ay: 50,
		bx: -300,
		by: 150,
		cx: -250,
		cy: 175 + size4,
		dx: -100,
		dy: 100 + size4,
		fill: '#04486B',
		stroke: 'none'
	};
	const shape3L = {
		label: 'Element 3 links',
		ax: shape4L.dx,
		ay: shape4L.dy,
		bx: shape4L.cx,
		by: shape4L.cy,
		cx: -100,
		cy: 250 + size4,
		fill: '#1E2424',
		stroke: 'none'
	};

	const shape1R = {
		label: 'Element 1 rechts',
		ax: 0,
		ay: 0,
		bx: 100,
		by: 50,
		cx: 100,
		cy: 300,
		dx: 0,
		dy: 250,
		fill: '#9F2F23',
		stroke: 'none'
	};
	const shape2R = {
		label: 'Element 2 rechts',
		ax: 0,
		ay: 0,
		bx: 0,
		by: -50,
		cx: 300,
		cy: 100,
		dx: 300,
		dy: 150,
		fill: '#FFAC70',
		stroke: 'none'
	};
	const shape4R = {
		label: 'Element 4 rechts',
		ax: 100,
		ay: 50,
		bx: 300,
		by: 150,
		cx: 250,
		cy: 175 + size4,
		dx: 100,
		dy: 100 + size4,
		fill: '#B57547',
		stroke: 'none'
	};
	const shape3R = {
		label: 'Element 3 rechts',
		ax: shape4R.dx,
		ay: shape4R.dy,
		bx: shape4R.cx,
		by: shape4R.cy,
		cx: 100,
		cy: 250 + size4,
		fill: '#1E2424',
		stroke: 'none'
	};

	// Hilfsfunktion für Punkte-String
	// const pts = (s) => `${s.a} ${s.b} ${s.c} ${s.d || ''}`;
	const pts = (s) => {
		const points = [
			`${s.ax},${s.ay}`,
			`${s.bx},${s.by}`,
			`${s.cx},${s.cy}`
		];

		if (typeof s.dx === 'number' && typeof s.dy === 'number') {
			points.push(`${s.dx},${s.dy}`);
		}

		return points.join(' ');
	};
</script>

<div class="svg-container">
	<svg class="svg-canvas" viewBox="1000 1000 2000 1500" style="background-color: white;">
		{#each rows as yi}
			{#each cols as xi}
				{@const translateX = xi * 400 + 1000}
				{@const translateY = yi * 300 + xi * 150 + 1000}

				<g transform="translate({translateX}, {translateY})">
					<polygon
						points={pts(shape1L)}
						fill={shape1L.fill}
						stroke={shape1L.stroke}
						stroke-width="3"
						stroke-linejoin="round"><title>{shape1L.label}</title></polygon
					>
					<polygon
						points={pts(shape2L)}
						fill={shape2L.fill}
						stroke={shape2L.stroke}
						stroke-width="3"
						stroke-linejoin="round"><title>{shape2L.label}</title></polygon
					>
					<polygon
						points={pts(shape3L)}
						fill={shape3L.fill}
						stroke={shape3L.stroke}
						stroke-width="3"
						stroke-linejoin="round"><title>{shape3L.label}</title></polygon
					>
					<polygon
						points={pts(shape4L)}
						fill={shape4L.fill}
						stroke={shape4L.stroke}
						stroke-width="3"
						stroke-linejoin="round"><title>{shape4L.label}</title></polygon
					>

					<polygon
						points={pts(shape1R)}
						fill={shape1R.fill}
						stroke={shape1R.stroke}
						stroke-width="3"
						stroke-linejoin="round"><title>{shape1R.label}</title></polygon
					>
					<polygon
						points={pts(shape2R)}
						fill={shape2R.fill}
						stroke={shape2R.stroke}
						stroke-width="3"
						stroke-linejoin="round"><title>{shape2R.label}</title></polygon
					>
					<polygon
						points={pts(shape3R)}
						fill={shape3R.fill}
						stroke={shape3R.stroke}
						stroke-width="3"
						stroke-linejoin="round"><title>{shape3R.label}</title></polygon
					>
					<polygon
						points={pts(shape4R)}
						fill={shape4R.fill}
						stroke={shape4R.stroke}
						stroke-width="3"
						stroke-linejoin="round"><title>{shape4R.label}</title></polygon
					>
				</g>
			{/each}
		{/each}
	</svg>
</div>

<style>
	/* Hier könnten noch Stile ergänzt werden, falls nötig */
</style>
