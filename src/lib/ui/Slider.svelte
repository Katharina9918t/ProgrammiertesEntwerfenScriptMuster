<!-- a custom slider component -->
<script>
	let {
		min = $bindable(0),
		max = $bindable(100),
		step = $bindable(1),
		value = $bindable(50),
		label = $bindable('Slider'),
		snapValues = [],
		snapWidth = 16
	} = $props();

	// references to DOM elements
	let trackElement = $state(null);
	let thumbElement = $state(null);
	// resize trigger will be incremented on window resize to force recalculation of track/thumb widths
	let resizeTrigger = $state(0);
	let trackWidth = $derived(trackElement ? trackElement.clientWidth + resizeTrigger * 0 : 0);
	let thumbWidth = $derived(thumbElement ? thumbElement.clientWidth + resizeTrigger * 0 : 0);
	// factor to convert value units to pixel units
	let valueToPixelFactor = $derived((trackWidth - thumbWidth) / (max - min));
	// position of thumb in pixels
	let position = $derived(((value - min) / (max - min)) * (trackWidth - thumbWidth));

	// ensure snapValues are sorted for snapping logic
	let sortedSnapValues = $derived([...snapValues].sort((a, b) => a - b));

	function snapValue(val) {
		if (sortedSnapValues.length === 0) return val;

		// 1. Find the closest snap and its index
		let closestIndex = 0;
		let closestDiff = Math.abs(val - sortedSnapValues[0]);
		for (let i = 1; i < sortedSnapValues.length; i++) {
			let diff = Math.abs(val - sortedSnapValues[i]);
			if (diff < closestDiff) {
				closestIndex = i;
				closestDiff = diff;
			}
		}
		let closest = sortedSnapValues[closestIndex];

		// 2. Calculate distance in pixels
		let distPixels = (val - closest) * valueToPixelFactor;
		let absDist = Math.abs(distPixels);

		// 3. Adaptive regions to avoid overlapping snap zones
		let magneticRegion = snapWidth;
		if (distPixels > 0 && closestIndex < sortedSnapValues.length - 1) {
			let distToNext = (sortedSnapValues[closestIndex + 1] - closest) * valueToPixelFactor;
			magneticRegion = Math.min(magneticRegion, distToNext / 2);
		} else if (distPixels < 0 && closestIndex > 0) {
			let distToPrev = (closest - sortedSnapValues[closestIndex - 1]) * valueToPixelFactor;
			magneticRegion = Math.min(magneticRegion, distToPrev / 2);
		}

		// Stay region scales with the magnetic region
		let stayRegion = magneticRegion * 0.6;

		if (absDist < stayRegion) {
			return closest;
		} else if (absDist < magneticRegion) {
			// t from 0 (Stay-boundary) to 1 (Magnetic-boundary)
			let t = (absDist - stayRegion) / (magneticRegion - stayRegion);

			// Linear pull: thumb travels magneticRegion while mouse travels (magneticRegion - stayRegion)
			// (this was a previous implementation)
			// let pulledDist = t * magneticRegion;

			// Coefficients for a smooth transition (derivative is 0 at t=0 and 1.0 at t=1 relative to the mouse)
			let L = magneticRegion - stayRegion;
			let a = L - 2 * magneticRegion;
			let b = 3 * magneticRegion - L;

			// The cubic polynomial smooths the velocity
			let pulledDist = a * t * t * t + b * t * t;

			return closest + (Math.sign(distPixels) * pulledDist) / valueToPixelFactor;
		}

		return val;
	}

	function handlePointerDownThumb(event) {
		// unified PointerEvents handler
		event.stopPropagation();
		event.preventDefault();
		const clickX = event.clientX;
		const clickValue = value;

		// start a pointer drag from this pointerdown
		startPointerDrag(event.pointerId, event.clientX, value);
	}

	function startPointerDrag(pointerId, startClientX, startValue) {
		// attach move/up handlers and capture pointer to thumbElement if possible
		if (thumbElement && thumbElement.setPointerCapture) {
			try {
				thumbElement.setPointerCapture(pointerId);
			} catch (e) {
				// ignore if cannot capture
			}
		}

		function handlePointerMove(e) {
			const clientX = e.clientX;
			const deltaX = clientX - startClientX;
			const continuousValue = startValue + deltaX / valueToPixelFactor;
			const snappedValue = snapValue(continuousValue);
			const steppedValue = Math.round((snappedValue - min) / step) * step + min;
			value = Math.min(Math.max(steppedValue, min), max);
		}

		function handlePointerUp(e) {
			if (thumbElement && thumbElement.releasePointerCapture) {
				try {
					thumbElement.releasePointerCapture(pointerId);
				} catch (err) {
					// ignore
				}
			}
			window.removeEventListener('pointermove', handlePointerMove);
			window.removeEventListener('pointerup', handlePointerUp);
		}

		window.addEventListener('pointermove', handlePointerMove);
		window.addEventListener('pointerup', handlePointerUp);
	}

	function handleTrackClick(event) {
		// console.log("track click", event);
		const rect = trackElement.getBoundingClientRect();
		const clickX = event.clientX - rect.left;
		const newPosition = Math.min(Math.max(clickX - thumbWidth / 2, 0), trackWidth - thumbWidth);
		const continuousValue = min + (newPosition / (trackWidth - thumbWidth)) * (max - min);
		const snappedValue = snapValue(continuousValue);
		const steppedValue = Math.round((snappedValue - min) / step) * step + min;
		value = Math.min(Math.max(steppedValue, min), max);

		handlePointerDownThumb(event);
	}

	function handleKeyDown(event) {
		if (event.key === 'ArrowLeft' || event.key === 'ArrowDown') {
			event.preventDefault();
			value = Math.max(value - step, min);
		} else if (event.key === 'ArrowRight' || event.key === 'ArrowUp') {
			event.preventDefault();
			value = Math.min(value + step, max);
		}
	}

	// prettify value for display. even though the values are rounded to step,
	// they can still have many decimal places due to floating point precision issues
	function prettifyValue(val) {
		// get decimal places of step
		let stepDecimalPlaces = (step.toString().split('.')[1] || '').length;
		return val.toFixed(stepDecimalPlaces);
	}
</script>

<svelte:window
	on:resize={() => {
		resizeTrigger++;
	}}
/>

<div class="slider">
	<div class="label">{label}</div>
	<div bind:this={trackElement} class="track" onpointerdown={handleTrackClick} role="presentation">
		<div
			bind:this={thumbElement}
			class="thumb"
			onpointerdown={handlePointerDownThumb}
			onkeydown={handleKeyDown}
			role="slider"
			tabindex="0"
			aria-valuemin={min}
			aria-valuemax={max}
			aria-valuenow={value}
			aria-label={label + ' slider thumb'}
			style="left: {position}px;"
		>
			{prettifyValue(value)}
		</div>
	</div>
</div>

<style>
	.slider {
		width: 100%;
		margin-bottom: 1rem;
	}
	.label {
		font-size: 0.75rem;
		margin-top: 0;
		margin-bottom: 0.3rem;
		color: #ccc;
	}
	.track {
		position: relative;
		height: 20px;
		background: #444;
		border-radius: 4px;
		/* prevent browser touch gestures (scroll/zoom) from interfering with dragging */
		touch-action: none;
	}
	.thumb {
		position: relative;
		top: 0px;
		left: 1px;
		width: 45px;
		height: 20px;
		background: #666;
		border: 1px solid #777;
		border-radius: 4px;
		text-align: center;
		line-height: 18px;
		color: #fff;
		font-size: 0.75rem;
		font-variant-numeric: tabular-nums;
		user-select: none;
		touch-action: none;
	}
</style>
