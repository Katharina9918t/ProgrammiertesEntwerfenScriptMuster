<script>
  let {
    dragAreaElement,
    color,
    x = $bindable(),
    y = $bindable(),
    minX = 0,
    maxX = 1,
    minY = 0,
    maxY = 1,
    onchange,
  } = $props();

  $inspect(dragAreaElement);

  let dragging = $state(false);
  let clickMX, clickMY;
  let clickX, clickY;
  let activePointerId = null;
  let pointerCaptureElement = $state(null);
  let windowListenersAdded = false;

  function enter(e) {}

  function down(e) {
    // start dragging and capture the pointer so we continue receiving events
    // even if the pointer leaves the element (important for touch)
    dragging = true;
    activePointerId = e.pointerId ?? null;
    // pointerCaptureElement = e.currentTarget ?? null;
    try {
      if (
        pointerCaptureElement &&
        activePointerId != null &&
        pointerCaptureElement.setPointerCapture
      )
        pointerCaptureElement.setPointerCapture(activePointerId);
    } catch (err) {
      // ignore if capture is not available
    }

    // prevent default to avoid e.g. scrolling on touch devices while dragging
    if (e.preventDefault) e.preventDefault();

    clickMX = e.clientX;
    clickMY = e.clientY;
    clickX = x;
    clickY = y;
  }

  function move(e) {
    // ignore other pointers when one pointer is active
    if (activePointerId != null && e.pointerId !== activePointerId) return;
    if (e.preventDefault) e.preventDefault();
    let dx = e.clientX - clickMX;
    let dy = e.clientY - clickMY;
    x = clickX + dx;
    y = clickY + dy;
    if (minX != undefined && maxX != undefined)
      x = Math.min(Math.max(x, minX), maxX);
    if (minY != undefined && maxY != undefined)
      y = Math.min(Math.max(y, minY), maxY);

    if (onchange) onchange();
  }

  function up(e) {
    // only handle the pointer that started the drag
    if (activePointerId != null && e && e.pointerId !== activePointerId) return;

    // stop dragging and release pointer capture if we set it
    dragging = false;
    try {
      if (
        pointerCaptureElement &&
        activePointerId != null &&
        pointerCaptureElement.releasePointerCapture
      )
        pointerCaptureElement.releasePointerCapture(activePointerId);
    } catch (err) {
      // ignore
    }
    activePointerId = null;
    // pointerCaptureElement = null;
    // remove any fallback window listeners we added
    try {
      if (windowListenersAdded) {
        window.removeEventListener("pointermove", move);
        window.removeEventListener("pointerup", up);
        window.removeEventListener("pointercancel", up);
        windowListenersAdded = false;
      }
    } catch (err) {}
  }

  function leave(e) {}

  // make possible to click on the drag area element to start dragging
  $effect(() => {
    if (dragAreaElement) {
      // console.log("setting up drag area pointerdown", dragAreaElement);
      // ensure the drag area doesn't allow native touch gestures to interfere
      try {
        dragAreaElement.style.touchAction = "none";
      } catch (err) {}

      dragAreaElement.onpointerdown = function (e) {
        console.log("drag area pointerdown", e);
        if (e.preventDefault) e.preventDefault();
        // move the node to the mouse position
        // clickMX = e.clientX;
        // clickMY = e.clientY;
        x = e.clientX - dragAreaElement.getBoundingClientRect().left + minX;
        y = e.clientY - dragAreaElement.getBoundingClientRect().top + minY;
        if (minX != undefined && maxX != undefined)
          x = Math.min(Math.max(x, minX), maxX);
        if (minY != undefined && maxY != undefined)
          y = Math.min(Math.max(y, minY), maxY);

        if (onchange) onchange();

        // ensure the drag start also uses pointer flow
        // try to set pointer capture; if not available, add window-level listeners as a fallback
        down(e);
        let pointerCaptured = false;
        try {
          const el = e.currentTarget || e.target;
          if (el && e.pointerId != null && el.setPointerCapture) {
            el.setPointerCapture(e.pointerId);
            pointerCaptured = true;
            // pointerCaptureElement = el;
            activePointerId = e.pointerId;
          }
        } catch (err) {
          pointerCaptured = false;
        }

        if (!pointerCaptured) {
          // fallback: listen on window so we don't lose events
          if (!windowListenersAdded) {
            window.addEventListener("pointermove", move, { passive: false });
            window.addEventListener("pointerup", up);
            window.addEventListener("pointercancel", up);
            windowListenersAdded = true;
          }
        }
      };
    }
  });
</script>

<svelte:window
  onpointermove={dragging ? move : null}
  onpointerup={dragging ? up : null}
  onpointercancel={dragging ? up : null}
/>

<!-- svelte-ignore a11y_no_static_element_interactions -->
<g
  bind:this={pointerCaptureElement}
  onpointerdown={down}
  onpointerenter={enter}
  onpointerleave={leave}
  transform="translate({x} {y})"
  style="touch-action: none; -webkit-user-drag: none;"
>
  <!-- <slot {dragging} {x} {y} /> -->
  <filter id="shadow">
    <feDropShadow dx="1" dy="1" stdDeviation="1" flood-opacity="0.5" />
  </filter>
  <g filter="url(#shadow)">
    <circle cx="0" cy="0" r="8" fill="#f000" stroke="none" />
    <circle
      cx="0"
      cy="0"
      r="5"
      fill={color}
      stroke="#fff"
      stroke-width="2"
    />
  </g>
</g>
