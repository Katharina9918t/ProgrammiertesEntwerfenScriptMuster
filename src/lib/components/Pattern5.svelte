<script>
	import Slider from '$lib/ui/Slider.svelte';
	import Toggle from '$lib/ui/Toggle.svelte';

	const cols = Array.from({ length: 31 }, (_, i) => i - 15);
	const rows = Array.from({ length: 31 }, (_, i) => i - 15);

	let size     = $state(1);
	let width    = $state(1);
	let height   = $state(1);
	let editMode = $state(true);

	// viewBox zentriert auf (2000, 2000), zeigt ~4 Kacheln horizontal/vertikal
	const viewBoxX = $derived(2000 - 2400 / size);
	const viewBoxY = $derived(2000 - 2000 / size);
	const viewBoxW = $derived(4800 / size);
	const viewBoxH = $derived(4000 / size);

	// ── Kachel-Versatzvektor ─────────────────────────────────────────────────
	// Abgeleitet aus: F1R a–d ≡ F3L c–b der Nachbarkachel (eine Zeile tiefer)
	//   F1R a (0,100)   → visuell (tx,    ty−100)
	//   F1R d (600,400) → visuell (tx−600, ty−400)
	//   F3L c (−600,800)→ visuell (tx_j+600, ty_j−800)  ⟹  tx_j = tx−600
	//   F3L b (0,1100)  → visuell (tx_j,   ty_j−1100)   ⟹  ty_j = ty+700
	// → pro Zeile: x −= 600, y += 700
	const stepXperCol = 1200;   // x-Schritt pro Spalte
	const stepXperRow =  -600;  // x-Versatz pro Zeile
	const stepYperRow =   700;  // y-Schritt pro Zeile

	// ── Farbthemen ───────────────────────────────────────────────────────────
	let activeTheme = $state(1);

	const colorThemes = {
		1: {
			// Gleiche Farben wie original, neu verteilt
			F1R: '#CA9D52', F2R: '#697267', F3R: '#F0DB94', F4R: '#8A8874',
			F5R: '#080808', F6R: '#38201E', F7R: '#671A15',
			F1L: '#7C7652', F2L: '#C07A2B', F3L: '#382224', F4L: '#5B2710',
			F5L: '#080808', F6L: '#5C675E', F7L: '#671A15',
		},
		2: {
			// Theme 2 Originalfarben mit Sättigung −20 Prozentpunkte (HSL)
			// #E8B54C→#D4AE60  #CF4022→#B7503A  #DB9D6C→#C99F7E
			// #135F7F→#225970  #9FADAE→#A7A7A7  #121210→#111111
			F1R: '#D4AE60', F2R: '#B7503A', F3R: '#C99F7E', F4R: '#225970',
			F5R: '#111111', F6R: '#A7A7A7', F7R: '#111111',
			F1L: '#B7503A', F2L: '#D4AE60', F3L: '#A7A7A7', F4L: '#225970',
			F5L: '#111111', F6L: '#C99F7E', F7L: '#111111',
		},
	};

	// ── Formen-Geometrie (pts) ────────────────────────────────────────────────
	// Koordinaten ×100 aus Nutzerangaben. Linke Seite = x gespiegelt (x → -x).
	const shapeDefs = [
		{ id: 'F1R', pts: '0,100 0,300 600,600 600,400'       },
		{ id: 'F2R', pts: '0,500 0,1000 200,900 200,400'      },
		{ id: 'F3R', pts: '0,1000 0,1100 600,800 600,700'     },
		{ id: 'F4R', pts: '200,900 600,700 400,600 200,700'   },
		{ id: 'F5R', pts: '200,700 400,600 200,500'           },
		{ id: 'F6R', pts: '200,500 600,700 600,600 200,400'   },
		{ id: 'F7R', pts: '0,300 0,500 200,400'               },
		{ id: 'F1L', pts: '0,100 0,300 -600,600 -600,400'     },
		{ id: 'F2L', pts: '0,500 0,1000 -200,900 -200,400'    },
		{ id: 'F3L', pts: '0,1000 0,1100 -600,800 -600,700'   },
		{ id: 'F4L', pts: '-200,900 -600,700 -400,600 -200,700' },
		{ id: 'F5L', pts: '-200,700 -400,600 -200,500'        },
		{ id: 'F6L', pts: '-200,500 -600,700 -600,600 -200,400' },
		{ id: 'F7L', pts: '0,300 0,500 -200,400'              },
	];

	// ── Hilfsfunktionen ───────────────────────────────────────────────────────
	const parsePts = (ptsStr) =>
		ptsStr.split(' ').map((p) => { const [x, y] = p.split(',').map(Number); return { x, y }; });

	const centroid = (pts) => ({
		x: pts.reduce((s, p) => s + p.x, 0) / pts.length,
		y: pts.reduce((s, p) => s + p.y, 0) / pts.length,
	});

	// Formen mit Farbe aus aktivem Theme + berechnete Label-Positionen
	const shapesWithMeta = $derived(shapeDefs.map((s) => {
		const pts = parsePts(s.pts);
		const c = centroid(pts);
		const letters = ['a', 'b', 'c', 'd'];
		const pointLabels = pts.map((p, i) => ({
			text: letters[i],
			x: p.x + (c.x - p.x) * 0.28,
			y: p.y + (c.y - p.y) * 0.28,
		}));
		return { ...s, color: colorThemes[activeTheme][s.id], pts, center: c, pointLabels };
	}));
</script>

<div class="svg-container">
	<svg class="svg-canvas" viewBox="{viewBoxX} {viewBoxY} {viewBoxW} {viewBoxH}" style="background-color: white;">
		<g transform="translate(2000, 2000) scale({width}, {height}) translate(-2000, -2000)">
		{#each rows as yi}
			{#each cols as xi}
				{@const translateX = xi * stepXperCol + yi * stepXperRow + 2000}
				{@const translateY = yi * stepYperRow + 2000}

				<g transform="translate({translateX}, {translateY}) scale(-1, -1)">
					{#each shapesWithMeta as s}
						<polygon
							points={s.pts.map(p => `${p.x},${p.y}`).join(' ')}
							fill={editMode ? 'none' : s.color}
							stroke={editMode ? s.color : 'none'}
							stroke-width="5"
							stroke-linejoin="round"
						><title>{s.id}</title></polygon>

						{#if editMode}
							{@const displayId = s.id.includes('R') ? s.id.replace('R', 'L') : s.id.replace('L', 'R')}
							<text
								x={s.center.x}
								y={s.center.y}
								transform="translate({s.center.x},{s.center.y}) scale(-1,-1) translate({-s.center.x},{-s.center.y})"
								class="shape-label"
								text-anchor="middle"
								dominant-baseline="middle"
							>{displayId}</text>

							{#each s.pointLabels as pl}
								<text
									x={pl.x}
									y={pl.y}
									transform="translate({pl.x},{pl.y}) scale(-1,-1) translate({-pl.x},{-pl.y})"
									class="point-label"
									text-anchor="middle"
									dominant-baseline="middle"
								>{pl.text}</text>
							{/each}
						{/if}
					{/each}
				</g>
			{/each}
		{/each}
		</g>
	</svg>
</div>

<div class="sidebar-right">
	<Toggle label="Bearbeiten" bind:value={editMode} />
	<Slider min={0.5} max={8}    step={0.01} label="Size"   bind:value={size}   />
	<Slider min={0.5} max={2.5}  step={0.01} label="Width"  bind:value={width}  />
	<Slider min={0.5} max={2.5}  step={0.01} label="Height" bind:value={height} />

	<div class="theme-section">
		<span class="theme-label">Farbthema</span>
		<div class="theme-buttons">
			<button
				class="theme-btn"
				class:active={activeTheme === 1}
				onclick={() => activeTheme = 1}
			>Theme 1</button>
			<button
				class="theme-btn"
				class:active={activeTheme === 2}
				onclick={() => activeTheme = 2}
			>Theme 2</button>
		</div>
	</div>
</div>

<style>
	.shape-label {
		font-size: 40px;
		font-weight: 700;
		fill: #ffffff;
		stroke: #111111;
		stroke-width: 3px;
		paint-order: stroke fill;
		pointer-events: none;
		user-select: none;
	}

	.point-label {
		font-size: 22px;
		font-weight: 600;
		fill: #111111;
		stroke: #ffffff;
		stroke-width: 2px;
		paint-order: stroke fill;
		pointer-events: none;
		user-select: none;
	}

	.theme-section {
		margin-top: 1.5rem;
		display: flex;
		flex-direction: column;
		gap: 0.5rem;
	}

	.theme-label {
		font-size: 0.75rem;
		text-transform: uppercase;
		letter-spacing: 0.05em;
		color: #aaa;
	}

	.theme-buttons {
		display: flex;
		gap: 0.5rem;
	}

	.theme-btn {
		flex: 1;
		padding: 0.4rem 0;
		background: #2a2a2a;
		border: 1px solid #444;
		color: #ccc;
		border-radius: 4px;
		cursor: pointer;
		font-size: 0.85rem;
		transition: background 0.15s, border-color 0.15s;
	}

	.theme-btn:hover {
		background: #333;
		border-color: #666;
	}

	.theme-btn.active {
		background: #555;
		border-color: #aaa;
		color: #fff;
	}
</style>
