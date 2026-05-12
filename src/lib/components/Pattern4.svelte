<script>
	import Slider from '$lib/ui/Slider.svelte';
	import Toggle from '$lib/ui/Toggle.svelte';

	const cols = [-5, -4, -3, -2, -1, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
	const rows = [-8, -7, -6, -5, -4, -3, -2, -1, 0, 1, 2, 3, 4, 5, 6, 7, 8];

	// ── Größe / Zoom ─────────────────────────────────────────────────────────
	// Skalierung vom Mittelpunkt der viewBox aus (x=2000, y=1750)
	// 1 = Standard,  >1 = hereinzoomen,  <1 = herauszoomen
	let size  = $state(1);
	let width  = $state(1);
	let height = $state(1);
	let s1dx   = $state(0); // additiver x-Versatz (Pixel): alle Außenpunkte verschieben sich gleichmäßig

	// viewBox-Zentrum: x=2000, y=1750  –  bleibt von width unberührt
	// size  skaliert gleichmäßig (x + y) über die viewBox
	// width verzerrt die Punkte direkt in x (scale auf jede Kachel-Gruppe)
	const viewBoxX = $derived(2000 - 1000 / size);
	const viewBoxY = $derived(1750 -  750 / size);
	const viewBoxW = $derived(2000 / size);
	const viewBoxH = $derived(1500 / size);

	// ── Modus ────────────────────────────────────────────────────────────────
	// true  = Bearbeiten: Beschriftungen sichtbar, Formen ungefüllt (nur Kontur)
	// false = Ansicht:    keine Beschriftungen, Formen mit Farbe gefüllt
	let editMode = $state(true);

	// ── Verformungsparameter ─────────────────────────────────────────────────
	// Höhe von s4 (Pixelwert d–a)
	// 0   → s4 hat keine Fläche (d=a, c=b), s3 kollabiert auf pL4/pR4
	// 200 → s4 + s3 auf Maximum, alle s3-Punkte treffen sich in pL3c/pR3c
	let s4dy = $state(100);

	// ── Punkte ───────────────────────────────────────────────────────────────
	// Mittelachse – bleiben immer bei x=0
	const pO = { x: 0, y:   0 }; // s1L.a  s2L.a  s1R.a  s2R.a
	const pT = { x: 0, y: -50 }; // s2L.b  s2R.b
	const pB = { x: 0, y: 250 }; // s1L.d  s1R.d

	// Alle Außenpunkte werden um s1dx nach außen verschoben (additiv, nicht multiplikativ!)
	// → Winkel, Parallelitäten und alle geometrischen Verhältnisse bleiben exakt erhalten
	// Beweis: Richtung pL4→pL3c = {(-100-s1dx)-(-300-s1dx), 250-150} = {200,100} — konstant ✓

	// Linke Seite: Verschiebung um -s1dx in x
	const pL1  = $derived({ x: -100 - s1dx, y:  50 });  // s1L.b  s4L.a
	const pL4  = $derived({ x: -300 - s1dx, y: 150 });  // s2L.d  s4L.b
	const pL1c = $derived({ x: -100 - s1dx, y: 300 });  // s1L.c
	const pL2c = $derived({ x: -300 - s1dx, y: 100 });  // s2L.c
	const pL3c = $derived({ x: -100 - s1dx, y: 250 });  // s3L.c

	// Rechte Seite: Verschiebung um +s1dx in x
	const pR1  = $derived({ x:  100 + s1dx, y:  50 });  // s1R.b  s4R.a
	const pR4  = $derived({ x:  300 + s1dx, y: 150 });  // s2R.d  s4R.b
	const pR1c = $derived({ x:  100 + s1dx, y: 300 });  // s1R.c
	const pR2c = $derived({ x:  300 + s1dx, y: 100 });  // s2R.c
	const pR3c = $derived({ x:  100 + s1dx, y: 250 });  // s3R.c

	// Bewegliche Punkte (s4dy + s1dx)
	// s4dy=0:   pL2=pL1, pL3=pL4  → s4 flächenlos
	// s4dy=200: pL2=pL3c, pL3=pL3c → alle s3-Punkte treffen sich
	const pL2 = $derived({ x: -100 - s1dx,          y:  50 + s4dy });
	const pR2 = $derived({ x:  100 + s1dx,          y:  50 + s4dy });
	const pL3 = $derived({ x: -300 - s1dx + s4dy,   y: 150 + 0.5 * s4dy });
	const pR3 = $derived({ x:  300 + s1dx - s4dy,   y: 150 + 0.5 * s4dy });

	// ── Farben ───────────────────────────────────────────────────────────────
	const colors = {
		s1L: '#742E30',
		s2L: '#7A7B73',
		s3L: '#1E2424',
		s4L: '#04486B',
		s1R: '#9F2F23',
		s2R: '#FFAC70',
		s3R: '#1E2424',
		s4R: '#B57547'
	};

	// ── Formen ───────────────────────────────────────────────────────────────
	// fill/stroke wechseln je nach editMode:
	//   Bearbeiten (true):  fill='none',  stroke=Farbe  → Kontur sichtbar
	//   Ansicht    (false): fill=Farbe,   stroke='none' → ausgefüllt
	const shape1L = $derived({ label: 'Element 1 links',  a: pO,  b: pL1, c: pL1c, d: pB,  fill: editMode ? 'none' : colors.s1L, stroke: editMode ? colors.s1L : 'none' }); // pL1, pL1c reaktiv
	const shape2L = $derived({ label: 'Element 2 links',  a: pO,  b: pT,  c: pL2c, d: pL4, fill: editMode ? 'none' : colors.s2L, stroke: editMode ? colors.s2L : 'none' });
	const shape3L = $derived({ label: 'Element 3 links',  a: pL2, b: pL3, c: pL3c,         fill: editMode ? 'none' : colors.s3L, stroke: editMode ? colors.s3L : 'none' });
	const shape4L = $derived({ label: 'Element 4 links',  a: pL1, b: pL4, c: pL3,  d: pL2, fill: editMode ? 'none' : colors.s4L, stroke: editMode ? colors.s4L : 'none' });

	const shape1R = $derived({ label: 'Element 1 rechts', a: pO,  b: pR1, c: pR1c, d: pB,  fill: editMode ? 'none' : colors.s1R, stroke: editMode ? colors.s1R : 'none' });
	const shape2R = $derived({ label: 'Element 2 rechts', a: pO,  b: pT,  c: pR2c, d: pR4, fill: editMode ? 'none' : colors.s2R, stroke: editMode ? colors.s2R : 'none' });
	const shape3R = $derived({ label: 'Element 3 rechts', a: pR2, b: pR3, c: pR3c,         fill: editMode ? 'none' : colors.s3R, stroke: editMode ? colors.s3R : 'none' });
	const shape4R = $derived({ label: 'Element 4 rechts', a: pR1, b: pR4, c: pR3,  d: pR2, fill: editMode ? 'none' : colors.s4R, stroke: editMode ? colors.s4R : 'none' });

	// ── Hilfsfunktionen ──────────────────────────────────────────────────────
	const pts = (s) => {
		const points = [`${s.a.x},${s.a.y}`, `${s.b.x},${s.b.y}`, `${s.c.x},${s.c.y}`];
		if (s.d) points.push(`${s.d.x},${s.d.y}`);
		return points.join(' ');
	};

	const pointList = (s) => {
		const points = [s.a, s.b, s.c];
		if (s.d) points.push(s.d);
		return points;
	};

	const centroid = (points) => ({
		x: points.reduce((sum, p) => sum + p.x, 0) / points.length,
		y: points.reduce((sum, p) => sum + p.y, 0) / points.length
	});

	const pointLabelsForShape = (shape) => {
		const points = pointList(shape);
		const letters = ['a', 'b', 'c', 'd'].slice(0, points.length);
		const c = centroid(points);
		return points.map((p, i) => ({
			text: letters[i],
			x: p.x + (c.x - p.x) * 0.22,
			y: p.y + (c.y - p.y) * 0.22
		}));
	};

	const centerLabelForShape = (shapeCode, shape) => {
		const c = centroid(pointList(shape));
		return { text: shapeCode, x: c.x, y: c.y };
	};

	// ── shapeEntries + Labels reaktiv ────────────────────────────────────────
	const shapeEntries = $derived([
		{ code: 's1l', shape: shape1L },
		{ code: 's2l', shape: shape2L },
		{ code: 's3l', shape: shape3L },
		{ code: 's4l', shape: shape4L },
		{ code: 's1r', shape: shape1R },
		{ code: 's2r', shape: shape2R },
		{ code: 's3r', shape: shape3R },
		{ code: 's4r', shape: shape4R }
	]);

	const allCenterLabels = $derived(shapeEntries.map(({ code, shape }) => centerLabelForShape(code, shape)));
	const allPointLabels  = $derived(shapeEntries.flatMap(({ shape }) => pointLabelsForShape(shape)));
</script>

<div class="svg-container">
	<svg class="svg-canvas" viewBox="{viewBoxX} {viewBoxY} {viewBoxW} {viewBoxH}" style="background-color: white;">
		<!-- width/height: gesamtes Muster strecken, Zentrum (2000, 1750) bleibt fix -->
		<g transform="translate(2000, 1750) scale({width}, {height}) translate(-2000, -1750)">
		{#each rows as yi}
			{#each cols as xi}
				{@const translateX = xi * 400 + 1000}
				{@const translateY = yi * 300 + xi * 150 + 1000}

				<g transform="translate({translateX}, {translateY})">
					<polygon points={pts(shape1L)} fill={shape1L.fill} stroke={shape1L.stroke} stroke-width="3" stroke-linejoin="round"><title>{shape1L.label}</title></polygon>
					<polygon points={pts(shape2L)} fill={shape2L.fill} stroke={shape2L.stroke} stroke-width="3" stroke-linejoin="round"><title>{shape2L.label}</title></polygon>
					<polygon points={pts(shape3L)} fill={shape3L.fill} stroke={shape3L.stroke} stroke-width="3" stroke-linejoin="round"><title>{shape3L.label}</title></polygon>
					<polygon points={pts(shape4L)} fill={shape4L.fill} stroke={shape4L.stroke} stroke-width="3" stroke-linejoin="round"><title>{shape4L.label}</title></polygon>

					<polygon points={pts(shape1R)} fill={shape1R.fill} stroke={shape1R.stroke} stroke-width="3" stroke-linejoin="round"><title>{shape1R.label}</title></polygon>
					<polygon points={pts(shape2R)} fill={shape2R.fill} stroke={shape2R.stroke} stroke-width="3" stroke-linejoin="round"><title>{shape2R.label}</title></polygon>
					<polygon points={pts(shape3R)} fill={shape3R.fill} stroke={shape3R.stroke} stroke-width="3" stroke-linejoin="round"><title>{shape3R.label}</title></polygon>
					<polygon points={pts(shape4R)} fill={shape4R.fill} stroke={shape4R.stroke} stroke-width="3" stroke-linejoin="round"><title>{shape4R.label}</title></polygon>

					{#if editMode}
						{#each allPointLabels as pointLabel}
							<text x={pointLabel.x} y={pointLabel.y} class="point-label" text-anchor="middle" dominant-baseline="middle">{pointLabel.text}</text>
						{/each}

						{#each allCenterLabels as centerLabel}
							<text x={centerLabel.x} y={centerLabel.y} class="shape-label" text-anchor="middle" dominant-baseline="middle">{centerLabel.text}</text>
						{/each}
					{/if}
				</g>
			{/each}
		{/each}
		</g>
	</svg>
</div>

<div class="sidebar-right">
	<Toggle
		label="Bearbeiten"
		bind:value={editMode}
	/>
	<Slider
		min={0.5}
		max={4}
		step={0.01}
		label="Size"
		bind:value={size}
	/>
	<Slider
		min={0.25}
		max={4}
		step={0.01}
		label="Width"
		bind:value={width}
	/>
	<Slider
		min={0.25}
		max={4}
		step={0.01}
		label="Height"
		bind:value={height}
	/>
	<Slider
		min={-90}
		max={200}
		step={1}
		label="s1 Breite"
		bind:value={s1dx}
	/>
	<Slider
		min={0}
		max={200}
		step={1}
		label="s4 Höhe (d–a)"
		bind:value={s4dy}
	/>
</div>

<style>
	.point-label {
		font-size: 10px;
		font-weight: 600;
		fill: #111111;
		stroke: #ffffff;
		stroke-width: 1px;
		paint-order: stroke fill;
		pointer-events: none;
		user-select: none;
	}

	.shape-label {
		font-size: 16px;
		font-weight: 700;
		fill: #ffffff;
		stroke: #111111;
		stroke-width: 1.5px;
		paint-order: stroke fill;
		pointer-events: none;
		user-select: none;
	}
</style>
