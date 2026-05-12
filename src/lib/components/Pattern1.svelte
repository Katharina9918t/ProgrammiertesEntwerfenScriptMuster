<script>
  const elementWidth = 150;
  const elementHeight = 200;

  const elementLenghtX = Math.round(1000 / elementWidth);
  const elementLenghtY = Math.round(1000 / elementHeight);

  function getColor(xi, yi) {
    // Erstes Paar von Reihen (yi % 2 === 0): erste Reihe
    if (yi % 2 === 0) {
      return xi % 2 === 0 ? 'white' : '#CCCCCC'; // gerade: weiß, ungerade: hellgrau
    }
    
    // Zweit';
  }

  function getTransform(xi, yi) {
    
    const y = yi * (elementHeight / 2);
    let x = xi * elementWidth - elementWidth / 2; // Standardposition für gerade Reihen
    if (yi % 2 === 0) {
      x = xi * elementWidth; // Versatz für ungerade Reihen
    }
    return `translate(${x}, ${y})`;

  }


</script>
        
        <div class="svg-container">
			<svg class="svg-canvas" viewBox="0 0 1000 1000">
            {#each Array(elementLenghtY) as _, yi}
              {#each Array(elementLenghtX) as _, xi}
                 <g 
                 transform={getTransform(xi, yi)}>

                 <polygon points="0,{elementHeight / 2} {elementWidth / 2}, 0 {elementWidth}, {elementHeight / 2} {elementWidth / 2}, {elementHeight}" 
                 fill={getColor(xi, yi)}>


                 </polygon>
                 </g>
             {/each}
            {/each}
				
			</svg>
		</div>