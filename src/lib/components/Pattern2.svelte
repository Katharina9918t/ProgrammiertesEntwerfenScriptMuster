<script>
  // ─── Elementmaße (runde Werte) ───────────────────────────────────────────
  // W   = Gesamtbreite des Elements
  // MX  = X-Mitte (Apex / Firstpunkt oben)
  // RH  = Dach-Höhe (Trapez oben)
  // DH  = Dachstreifen-Höhe (dünner Querstreifen)
  // SH  = Seitenwand-Höhe
  // SW  = Breite der Seitenpaneele

  const W   = 580;
  const MX  = 280;
  const RH  = 95;
  const DH  = 35;
  const SH  = 160;
  const SW  = 68;

  // Y-Positionen
  const y0 = 0;
  const y1 = RH;
  const y2 = RH + DH;
  const y3 = RH + DH + SH;

  // Raster: diagonal versetzt
  // stepX = halbe Breite  → Elemente überlappen horizontal
  // stepY = Dach + Streifen → vertikale Wiederholung
  // ungerade Spalten werden um stepY/2 nach unten versetzt

  const stepX = MX;        // 280
  const stepY = RH + DH;   // 130
  const cols  = 10;
  const rows  = 12;

  const viewW = cols * stepX + MX;
  const viewH = rows * stepY + stepY;

  // Hilfsfunktion: [[x,y], ...] → "x,y x,y ..."
  function pts(arr) {
    return arr.map(([x, y]) => `${x},${y}`).join(' ');
  }

  // ─── 8 Formen (relative Koordinaten, Ursprung = Apex des Elements) ───────
  const shapes = [
    // 1. Linkes Dach-Trapez (hell)
    { points: pts([[MX,y0],[0,y1],[0,y2],[MX,y1]]),                                  fill:'#D9D9D9', stroke:'black', sw:3 },
    // 2. Rechtes Dach-Trapez (mittelgrau)
    { points: pts([[MX,y0],[W,y1],[W,y2],[MX,y1]]),                                  fill:'#707070', stroke:'black', sw:3 },
    // 3. Linker Dachstreifen (dunkel)
    { points: pts([[MX,y1],[0,y2],[SW,y2+DH],[MX,y1+DH]]),                          fill:'#464646', stroke:'black', sw:3 },
    // 4. Rechter Dachstreifen (sehr dunkel)
    { points: pts([[MX+SW,y1],[W,y2],[W-SW,y2+DH],[MX+SW,y1+DH]]),                 fill:'#262626', stroke:'black', sw:3 },
    // 5. Linke Seitenwand (hell)
    { points: pts([[MX,y1+DH],[SW,y2+DH],[SW,y3],[MX,y3-DH]]),                     fill:'#D9D9D9', stroke:'black', sw:3 },
    // 6. Rechte Seitenwand (dunkel)
    { points: pts([[MX+SW,y1+DH],[W-SW,y2+DH],[W-SW,y3],[MX+SW,y3-DH]]),           fill:'#444444', stroke:'black', sw:3 },
    // 7. Linkes unteres Dreieck
    { points: pts([[SW,y2+DH],[0,y2+SH/2],[SW,y2+SH/2-DH]]),                       fill:'#C7C7C7', stroke:'#8F8C8C', sw:1 },
    // 8. Rechtes unteres Dreieck
    { points: pts([[W-SW,y2+DH],[W,y2+SH/2],[W-SW,y2+SH/2-DH]]),                   fill:'#C7C7C7', stroke:'#8F8C8C', sw:1 },
  ];
</script>

<div class="container">
  <svg viewBox="0 0 {viewW} {viewH}" xmlns="http://www.w3.org/2000/svg">
    {#each Array(cols) as _, xi}
      {#each Array(rows) as _, yi}
        {@const tx = xi * stepX}
        {@const ty = yi * stepY + (xi % 2 === 1 ? stepY / 2 : 0)}
        <g transform="translate({tx}, {ty})">
          {#each shapes as s}
            <polygon
              points={s.points}
              fill={s.fill}
              stroke={s.stroke}
              stroke-width={s.sw}
              stroke-linejoin="round"
            />
          {/each}
        </g>
      {/each}
    {/each}
  </svg>
</div>

<style>
  .container {
    width: 100%;
    height: 100vh;
    overflow: hidden;
    background: white;
  }
  svg {
    width: 100%;
    height: 100%;
  }
</style>
