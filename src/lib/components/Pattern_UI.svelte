<script>
   import Slider from "$lib/ui/Slider.svelte"
  console.log(Array(10));

  const squareSize = 50
  function getTransform(xi, yi) {
    const x = xi * squareSize;
    const y = yi * squareSize;
    return `translate(${x}, ${y})`;
  }
</script>

<div class="svg-container">
  <svg class="svg-canvas" viewBox="0 0 1000 1000">
    {#each Array(10) as _, yi}
      {#each Array(10) as _, xi}
       <polygon transform="(getTransform(xi, yi))" points="0,0 (squareSize), 0 (squareSize), (squareSize) 0, (squareSize)" fill={(xi + yi) % 2 === 0 ? 
       'white' : '#CCCCCC'}></polygon>
      {/each}
    {/each}
  </svg>
</div>


<div class="sidebar-right">
  <Slider min= {50} max={100} label="Width"/>
</div>