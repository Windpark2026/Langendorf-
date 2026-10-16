<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="apple-mobile-web-app-title" content="Langendorf">
<title>IST-LEISTUNG</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=IBM+Plex+Mono:wght@400;500;600&family=Oswald:wght@400;500;600&display=swap" rel="stylesheet">
<style>
  :root {
    --navy: #05090f;
    --navy-panel: #0e1c2b;
    --amber: #ffb454;
    --amber-dim: #a87639;
    --teal: #4fd8c4;
    --teal-dim: #2c6e64;
    --red: #ff5c5c;
    --text-lo: #7d8d99;
    --hairline: rgba(255, 180, 84, 0.22);
    --brass-lt: #d8b177;
    --brass-dk: #6b4a26;
    --font-mono: 'IBM Plex Mono', 'SFMono-Regular', Consolas, monospace;
    --font-display: 'Oswald', 'Arial Narrow', sans-serif;
  }
  * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
  html, body { margin:0; padding:0; height:100%; overflow:hidden; }
  body {
    background: radial-gradient(ellipse at center, #0a1420 0%, #05090f 70%, #030608 100%);
    display:flex; align-items:center; justify-content:center;
    padding-top: env(safe-area-inset-top,0px);
    padding-bottom: env(safe-area-inset-bottom,0px);
    height:100%;
  }
  .instrument-wrap {
    position: relative;
    width: min(92vmin, 720px);
    height: min(92vmin, 720px);
  }
  .instrument-wrap::before {
    content: "";
    position: absolute;
    inset: 2%;
    border-radius: 50%;
    box-shadow: 0 0 20px rgba(0,0,0,0.5);
    z-index: -1;
  }
  svg { width:100%; height:100%; display:block; }

  .backlight {
    position: fixed;
    inset: 0;
    background: rgba(79,216,196,0.14);
    z-index: 0;
    pointer-events: none;
  }
  .instrument-wrap { z-index: 1; }

  .seg { fill: var(--amber); opacity: 0; }
  .seg.lit { opacity: 1; filter: drop-shadow(0 0 4px rgba(255,180,84,0.8)); }

  .status {
    position: fixed;
    top: calc(10px + env(safe-area-inset-top, 0px));
    right: 10px;
    display: flex; align-items: center; gap: 6px;
    font-family: var(--font-display); font-size: 11px; letter-spacing: 0.08em;
    color: #e8f2f0; opacity: 0.95;
  }
  .dot { width: 7px; height: 7px; border-radius: 50%; background: var(--amber-dim); }
  .dot.live { background: var(--teal); animation: none; box-shadow: 0 0 4px var(--teal); }
  .dot.demo { background: var(--amber-dim); animation: blink 1.4s steps(1) infinite; }
  @keyframes blink { 50% { opacity: 0.25; } }

  .lastupdate {
    position: fixed;
    top: calc(26px + env(safe-area-inset-top, 0px));
    right: 10px;
    font-family: var(--font-mono); font-size: 10px; letter-spacing: 0.03em;
    color: #e8f2f0; opacity: 0.85;
  }

  .windbox {
    position: fixed;
    top: calc(6px + env(safe-area-inset-top, 0px));
    left: 6px;
    display: flex; flex-direction: column; align-items: flex-start; gap: 2px;
    padding: 4px 6px 4px 6px;
    background: radial-gradient(ellipse at 30% 25%, rgba(111,227,208,0.16), rgba(5,9,15,0.6) 72%);
    border-radius: 3px;
    box-shadow: 0 0 10px rgba(111,227,208,0.22), inset 0 0 6px rgba(111,227,208,0.1);
  }
  .windbox .windlabel {
    font-family: var(--font-mono); font-size: 10.5px; letter-spacing: 0.1em;
    color: #8fe8d8; opacity: 0.85;
  }
  .windbox svg { display: block; }

  /* HUD corner-bracket frame (windbox is already position:fixed, which also
     serves as the containing block for these absolutely-positioned corners) */
  .hudFrame::before, .hudFrame::after,
  .hudFrame .hc1, .hudFrame .hc2 {
    content: ""; position: absolute; width: 8px; height: 8px;
    border: 1.4px solid #6fe3d0; opacity: 0.85;
  }
  .hudFrame::before { top: 0; left: 0; border-right: none; border-bottom: none; }
  .hudFrame::after   { bottom: 0; right: 0; border-left: none; border-top: none; }
  .hudFrame .hc1 { top: 0; right: 0; border-left: none; border-bottom: none; }
  .hudFrame .hc2 { bottom: 0; left: 0; border-right: none; border-top: none; }

  .gear {
    position: fixed;
    bottom: calc(10px + env(safe-area-inset-bottom, 0px));
    right: 10px;
    width: 30px; height: 30px;
    border: 1px solid var(--hairline); border-radius: 50%;
    display: flex; align-items: center; justify-content: center;
    color: var(--text-lo); font-size: 14px; opacity: 0.35; background: transparent;
  }

  .settings-overlay {
    position: fixed; inset: 0; background: rgba(6, 13, 22, 0.96);
    display: none; align-items: center; justify-content: center; padding: 24px; z-index: 10;
  }
  .settings-overlay.open { display: flex; }
  .settings-box {
    width: 100%; max-width: 380px; background: var(--navy-panel);
    border: 1px solid var(--hairline); padding: 20px; font-family: var(--font-display);
  }
  .settings-box label { display:block; font-size:12px; letter-spacing:0.05em; color:var(--text-lo); margin-bottom:6px; }
  .settings-box input {
    width:100%; background:var(--navy); border:1px solid var(--hairline); color:var(--amber);
    font-family: var(--font-mono); font-size:14px; padding:10px; margin-bottom:16px;
  }
  .settings-box .hint { font-family: var(--font-mono); font-size:11px; color:var(--text-lo); line-height:1.5; margin-bottom:16px; }
  .settings-box button {
    width:100%; background:var(--amber-dim); color:var(--navy); border:none; padding:12px;
    font-family: var(--font-display); font-size:13px; letter-spacing:0.05em; font-weight:500;
  }
</style>
</head>
<body>

<div class="backlight"></div>

<div class="status">
  <span class="dot demo" id="statusDot"></span>
  <span id="statusText">DEMO</span>
</div>
<div class="lastupdate" id="lastUpdateBox">Aktualisiert: --:--:--</div>

<div class="windbox hudFrame">
  <span class="hc1"></span><span class="hc2"></span>
  <span class="windlabel">Wind &#216; (m/s)</span>
  <svg id="windDotsSvg" width="128" height="30" viewBox="0 0 128 30" xmlns="http://www.w3.org/2000/svg">
    <defs>
      <filter id="windDotGlow" x="-200%" y="-200%" width="500%" height="500%">
        <feGaussianBlur stdDeviation="1.0" result="blur1"/>
        <feGaussianBlur stdDeviation="0.5" result="blur2"/>
        <feMerge><feMergeNode in="blur1"/><feMergeNode in="blur2"/><feMergeNode in="SourceGraphic"/></feMerge>
      </filter>
    </defs>
    <g id="windDots" filter="url(#windDotGlow)"></g>
  </svg>
</div>

<div class="instrument-wrap">
  <svg id="dial" viewBox="0 0 400 400" xmlns="http://www.w3.org/2000/svg">
    <defs>
      <radialGradient id="bezelGrad" cx="35%" cy="30%" r="105%">
        <stop offset="0%" stop-color="#f3d9a8"/>
        <stop offset="18%" stop-color="#e9cb98"/>
        <stop offset="35%" stop-color="var(--brass-lt)"/>
        <stop offset="52%" stop-color="#b8925e"/>
        <stop offset="70%" stop-color="#8a6636"/>
        <stop offset="86%" stop-color="#78582f"/>
        <stop offset="100%" stop-color="var(--brass-dk)"/>
      </radialGradient>
      <radialGradient id="faceGrad" cx="42%" cy="38%" r="95%">
        <stop offset="0%" stop-color="#122436"/>
        <stop offset="65%" stop-color="#0d1c2a"/>
        <stop offset="100%" stop-color="#060d16"/>
      </radialGradient>
      <radialGradient id="hubGrad" cx="35%" cy="30%" r="105%">
        <stop offset="0%" stop-color="#f3d9a8"/>
        <stop offset="45%" stop-color="var(--brass-lt)"/>
        <stop offset="100%" stop-color="var(--brass-dk)"/>
      </radialGradient>
      <clipPath id="faceClip"><circle cx="200" cy="200" r="176"/></clipPath>
      <filter id="ledGlow" x="-200%" y="-200%" width="500%" height="500%">
        <feGaussianBlur in="SourceGraphic" stdDeviation="1.4" result="blur"/>
        <feMerge>
          <feMergeNode in="blur"/>
          <feMergeNode in="SourceGraphic"/>
        </feMerge>
      </filter>
      <linearGradient id="glassGrad" x1="0" y1="0" x2="1" y2="1">
        <stop offset="0%" stop-color="#ffffff" stop-opacity="0.10"/>
        <stop offset="40%" stop-color="#ffffff" stop-opacity="0.02"/>
        <stop offset="100%" stop-color="#ffffff" stop-opacity="0"/>
      </linearGradient>
    </defs>

    <!-- Bezel -->
    <circle cx="200" cy="200" r="196" fill="url(#bezelGrad)"/>
    <circle cx="200" cy="200" r="196" fill="none" stroke="#3a2712" stroke-width="1.5"/>
    <g id="rivets"></g>
    <circle cx="200" cy="200" r="180" fill="none" stroke="#3a2712" stroke-width="2"/>

    <!-- Face -->
    <circle cx="200" cy="200" r="176" fill="url(#faceGrad)"/>
    <g id="zoneArc"></g>
    <g id="ticks"></g>
    <g id="tickLabels"></g>

    <g id="ledRing"></g>

    <path id="labelArc" d="M 16.7 184.0 A 184 184 0 0 1 383.3 184.0" fill="none"/>
    <text font-family="Oswald" font-size="14" font-weight="600" letter-spacing="6" fill="#2a1c0d">
      <textPath href="#labelArc" startOffset="50%" text-anchor="middle">EINSPEISELEISTUNG&#8209;UW</textPath>
    </text>

    <path id="labelArcBottom" d="M 16.7 216.0 A 184 184 0 0 0 383.3 216.0" fill="none"/>
    <text font-family="Oswald" font-size="13" font-weight="600" letter-spacing="4" fill="#2a1c0d" dy="9">
      <textPath href="#labelArcBottom" startOffset="50%" text-anchor="middle">GESAMTERTRAG SEIT IBN</textPath>
    </text>

    <!-- digital readout window -->

    <g id="digitalKw"></g>
    <text id="kwUnit" font-family="Oswald" font-size="11" letter-spacing="0.5" fill="var(--amber)"></text>
    <text id="mwText" x="200" y="289.4" dominant-baseline="middle" text-anchor="middle" font-family="IBM Plex Mono" font-size="18.7" fill="var(--teal)"></text>

    <!-- Gesamtertrag readout window -->
    <g id="digitalYield"></g>
    <text id="yieldUnit" font-family="Oswald" font-size="9.9" letter-spacing="1" fill="var(--teal)"></text>

    <!-- needle -->
    <g id="needleGroup">
      <polygon id="needle" points="0,-150 5,10 0,22 -5,10" fill="var(--amber)"/>
      <polygon points="0,10 8,26 0,40 -8,26" fill="#7a4f22"/>
    </g>

    <text text-anchor="middle" font-family="Oswald" font-size="13" font-weight="600" letter-spacing="0.5" fill="var(--text-lo)" stroke="var(--navy)" stroke-width="3" stroke-linejoin="round" paint-order="stroke fill">
      <tspan x="200" y="130">LANGENDORF</tspan>
      <tspan x="200" y="146">8 Anlagen GE&#8209;1,5sl</tspan>
    </text>

    <circle cx="200" cy="200" r="24" fill="url(#hubGrad)" stroke="#3a2712" stroke-width="1.5"/>
    <circle cx="200" cy="200" r="6" fill="#2a1c0d"/>

    <!-- glass reflection on top -->
    <ellipse cx="150" cy="140" rx="150" ry="90" fill="url(#glassGrad)" clip-path="url(#faceClip)" transform="rotate(-18 150 140)"/>
  </svg>
</div>

<button class="gear" id="gearBtn">&#9881;</button>

<div class="settings-overlay" id="settingsOverlay">
  <div class="settings-box">
    <label for="urlInput">ADRESSE DES RELAY-SERVERS</label>
    <input type="text" id="urlInput" placeholder="http://192.168.1.50:3000/leistung">
    <div class="hint">Leer lassen = Demo-Modus mit simulierten Werten. Adresse eintragen, sobald der Relay-Server im Netzwerk erreichbar ist.</div>
    <button id="saveBtn">SPEICHERN</button>
  </div>
</div>

<script>
(function () {
  const STORAGE_KEY = 'leistungsanzeige_endpoint';
  const NS = 'http://www.w3.org/2000/svg';
  const CX = 200, CY = 200;
  const START_ANGLE = -130, END_ANGLE = 130, SWEEP = END_ANGLE - START_ANGLE;
  const MAX_MW = 12, RATED_MW = 6.2;

  function polar(r, angleDeg) {
    const a = (angleDeg - 0) * Math.PI / 180;
    return { x: CX + r * Math.sin(a), y: CY - r * Math.cos(a) };
  }
  function valueToAngle(mw) {
    const clamped = Math.max(0, Math.min(MAX_MW, mw));
    return START_ANGLE + (clamped / MAX_MW) * SWEEP;
  }

  // Rivets
  const rivetsG = document.getElementById('rivets');
  for (let i = 0; i < 12; i++) {
    if (i === 11 || i === 0 || i === 1 || i === 5 || i === 6 || i === 7) continue; // Nieten unter den Schriftzügen oben/unten entfernt
    const ang = (360 / 12) * i;
    const p = polar(188, ang);
    const c = document.createElementNS(NS, 'circle');
    c.setAttribute('cx', p.x.toFixed(1));
    c.setAttribute('cy', p.y.toFixed(1));
    c.setAttribute('r', '3');
    c.setAttribute('fill', '#4a3218');
    c.setAttribute('stroke', '#f3d9a8');
    c.setAttribute('stroke-width', '0.6');
    rivetsG.appendChild(c);
  }

  // Colour zone arc (teal -> amber -> red)
  function arcPath(r, a1, a2) {
    const p1 = polar(r, a1), p2 = polar(r, a2);
    const large = (a2 - a1) > 180 ? 1 : 0;
    return 'M ' + p1.x.toFixed(1) + ' ' + p1.y.toFixed(1) + ' A ' + r + ' ' + r + ' 0 ' + large + ' 1 ' + p2.x.toFixed(1) + ' ' + p2.y.toFixed(1);
  }
  const zoneG = document.getElementById('zoneArc');
  const zones = [];
  zones.forEach(function (z) {
    const path = document.createElementNS(NS, 'path');
    path.setAttribute('d', arcPath(163, valueToAngle(z.from), valueToAngle(z.to)));
    path.setAttribute('fill', 'none');
    path.setAttribute('stroke', z.color);
    path.setAttribute('stroke-width', '6');
    path.setAttribute('opacity', '0.55');
    zoneG.appendChild(path);
  });

  // Ticks + labels
  const ticksG = document.getElementById('ticks');
  const labelsG = document.getElementById('tickLabels');
  const MAJOR_STEP = 1, MINOR_STEP = 0.5;
  for (let mw = 0; mw <= MAX_MW + 0.001; mw += MINOR_STEP) {
    const isMajor = Math.abs(mw / MAJOR_STEP - Math.round(mw / MAJOR_STEP)) < 0.001;
    const ang = valueToAngle(mw);
    const p1 = polar(isMajor ? 150 : 156, ang);
    const p2 = polar(172, ang);
    const line = document.createElementNS(NS, 'line');
    line.setAttribute('x1', p1.x.toFixed(1)); line.setAttribute('y1', p1.y.toFixed(1));
    line.setAttribute('x2', p2.x.toFixed(1)); line.setAttribute('y2', p2.y.toFixed(1));
    line.setAttribute('stroke', isMajor ? 'var(--amber)' : 'var(--amber-dim)');
    line.setAttribute('stroke-width', isMajor ? '2.4' : '1');
    line.setAttribute('opacity', isMajor ? '0.95' : '0.5');
    ticksG.appendChild(line);
    if (isMajor) {
      const lp = polar(132, ang);
      const t = document.createElementNS(NS, 'text');
      t.setAttribute('x', lp.x.toFixed(1)); t.setAttribute('y', (lp.y + 5).toFixed(1));
      t.setAttribute('text-anchor', 'middle');
      t.setAttribute('font-family', 'IBM Plex Mono');
      t.setAttribute('font-size', '13');
      t.setAttribute('fill', 'var(--text-lo)');
      t.textContent = String(Math.round(mw));
      labelsG.appendChild(t);
    } else {
      const lp = polar(120, ang);
      const t = document.createElementNS(NS, 'text');
      t.setAttribute('x', lp.x.toFixed(1)); t.setAttribute('y', (lp.y + 3).toFixed(1));
      t.setAttribute('text-anchor', 'middle');
      t.setAttribute('font-family', 'IBM Plex Mono');
      t.setAttribute('font-size', '8');
      t.setAttribute('fill', 'var(--amber-dim)');
      t.textContent = mw.toFixed(1);
      labelsG.appendChild(t);
    }
  }

  // 7-segment digital readout for kW value
  const SEG_MAP = {
    '0':'abcdef','1':'bc','2':'abged','3':'abgcd','4':'fgbc',
    '5':'afgcd','6':'afgecd','7':'abc','8':'abcdefg','9':'abcdfg','-':'g',' ':''
  };
  function segRects(w, h, t) {
    return {
      a:[t*0.6,0,w-t*1.2,t], g:[t*0.6,h/2-t/2,w-t*1.2,t], d:[t*0.6,h-t,w-t*1.2,t],
      f:[0,t*0.6,t,h/2-t*0.9], b:[w-t,t*0.6,t,h/2-t*0.9],
      e:[0,h/2+t*0.3,t,h/2-t*0.9], c:[w-t,h/2+t*0.3,t,h/2-t*0.9]
    };
  }
  function drawDigit(svg, xOffset, ch, scale) {
    const w=34*scale, h=40*scale, t=6*scale;
    const rects = segRects(w,h,t);
    const active = SEG_MAP[ch] || '';
    Object.keys(rects).forEach(function(key){
      const r = rects[key];
      const rect = document.createElementNS(NS,'rect');
      rect.setAttribute('x',(xOffset+r[0]).toFixed(1));
      rect.setAttribute('y',r[1].toFixed(1));
      rect.setAttribute('width',r[2].toFixed(1));
      rect.setAttribute('height',r[3].toFixed(1));
      rect.setAttribute('rx',(t*0.35).toFixed(1));
      rect.setAttribute('class','seg'+(active.indexOf(key)>-1?' lit':''));
      svg.appendChild(rect);
    });
    return w;
  }
  function drawSep(svg, xOffset, scale, h) {
    const size = 6*scale*0.9;
    const rect = document.createElementNS(NS,'rect');
    rect.setAttribute('x',xOffset.toFixed(1));
    rect.setAttribute('y',(h-size).toFixed(1));
    rect.setAttribute('width',size.toFixed(1));
    rect.setAttribute('height',size.toFixed(1));
    rect.setAttribute('class','seg lit');
    svg.appendChild(rect);
    return size+4*scale;
  }
  function renderKwDigits(display) {
    const svg = document.getElementById('digitalKw');
    while (svg.firstChild) svg.removeChild(svg.firstChild);
    const KW_SCALE = 1.2;
    const h = 24 * KW_SCALE;
    let x = 0;
    for (let i=0;i<display.length;i++){
      const ch = display[i];
      if (ch==='.') { x += drawYieldSep(svg,x,ch,h) + 2; }
      else { x += drawYieldDigit(svg,x,ch,KW_SCALE) + 3 * KW_SCALE; }
    }
    // im Fenster (x118-282, y238-294) zentrieren, Einheit "kW" mit einrechnen
    const windowLeft = 118, windowWidth = 164, windowTop = 238;
    const unitWidth = 17;
    const totalWidth = x + 4 + unitWidth;
    const startX = windowLeft + (windowWidth - totalWidth) / 2;
    const startY = windowTop + 8;
    svg.setAttribute('transform', 'translate(' + startX.toFixed(1) + ',' + startY.toFixed(1) + ')');
    const unit = document.getElementById('kwUnit');
    unit.setAttribute('x', (startX + x + 4).toFixed(1));
    unit.setAttribute('y', (startY + h - 2).toFixed(1));
    unit.textContent = 'kW';
  }

  // Sehr kleine LEDs entlang des schmalen inneren Rings, 0-12 MW in 0,1-MW-Schritten
  // (echte 1-kW-Auflösung = 12.000 LEDs ließe sich auf diesem Ring nicht überlappungsfrei
  // darstellen, daher die feinste praktikable Auflösung von 100 kW pro LED)
  const ledG = document.getElementById('ledRing');
  const LED_STEP = 0.1;
  const leds = [];
  for (let mw = 0; mw <= MAX_MW + 0.001; mw += LED_STEP) {
    const p = polar(178, valueToAngle(mw));
    const c = document.createElementNS(NS, 'circle');
    c.setAttribute('cx', p.x.toFixed(2));
    c.setAttribute('cy', p.y.toFixed(2));
    c.setAttribute('r', '1.3');
    c.setAttribute('fill', '#1a3a36');
    ledG.appendChild(c);
    leds.push(c);
  }
  function updateLeds(mw) {
    const litCount = Math.round(Math.min(MAX_MW, Math.max(0, mw)) / LED_STEP);
    leds.forEach(function (c, i) {
      const lit = i <= litCount;
      c.setAttribute('fill', lit ? '#4fd8c4' : '#1a3a36');
      c.setAttribute('opacity', lit ? '1' : '0.5');
      c.setAttribute('r', lit ? '1.7' : '1.1');
      c.setAttribute('filter', lit ? 'url(#ledGlow)' : 'none');
    });
  }

  // 7-segment renderer for the Gesamtertrag row (kleinere, eigene Maße)
  function drawYieldDigit(svg, xOffset, ch, scale) {
    scale = scale || 1;
    const w = 13 * scale, h = 24 * scale, t = 3 * scale;
    const rects = segRects(w, h, t);
    const active = SEG_MAP[ch] || '';
    Object.keys(rects).forEach(function (key) {
      const r = rects[key];
      const rect = document.createElementNS(NS, 'rect');
      rect.setAttribute('x', (xOffset + r[0]).toFixed(1));
      rect.setAttribute('y', r[1].toFixed(1));
      rect.setAttribute('width', r[2].toFixed(1));
      rect.setAttribute('height', r[3].toFixed(1));
      rect.setAttribute('rx', (t * 0.35).toFixed(1));
      rect.setAttribute('class', 'seg' + (active.indexOf(key) > -1 ? ' lit' : ''));
      svg.appendChild(rect);
    });
    return w;
  }
  function drawYieldSep(svg, xOffset, kind, h) {
    const size = 4.4;
    if (kind === ',') {
      const s = size;
      const poly = document.createElementNS(NS, 'polygon');
      const pts = [
        (xOffset + s * 0.95).toFixed(2) + ',' + (h - s * 0.55).toFixed(2),
        (xOffset + s * 0.4).toFixed(2) + ',' + (h - s * 0.55).toFixed(2),
        (xOffset - s * 0.15).toFixed(2) + ',' + (h + s * 1.45).toFixed(2)
      ].join(' ');
      poly.setAttribute('points', pts);
      poly.setAttribute('class', 'seg lit');
      svg.appendChild(poly);
      return size + 2.6;
    }
    const rect = document.createElementNS(NS, 'rect');
    rect.setAttribute('x', xOffset.toFixed(1));
    rect.setAttribute('y', (h - size).toFixed(1));
    rect.setAttribute('width', size.toFixed(1));
    rect.setAttribute('height', size.toFixed(1));
    rect.setAttribute('rx', (size * 0.3).toFixed(1));
    rect.setAttribute('class', 'seg lit');
    svg.appendChild(rect);
    return size + 2.6;
  }
  // Format: 9 Ziffern -> XXX.XXX,XXX  (Tausendertrenner nach Stelle 3, Komma nach Stelle 6)
  function formatYield(mwh) {
    const clampedWhole = Math.max(0, Math.min(999999, Math.floor(mwh)));
    let decimals = Math.round((mwh - Math.floor(mwh)) * 1000);
    if (decimals >= 1000) decimals = 999;
    const wholeStr = String(clampedWhole).padStart(6, '0');
    const decStr = String(decimals).padStart(3, '0');
    return wholeStr.slice(0, 3) + '.' + wholeStr.slice(3, 6) + ',' + decStr;
  }
  function renderYieldDigits(display) {
    const svg = document.getElementById('digitalYield');
    while (svg.firstChild) svg.removeChild(svg.firstChild);
    const h = 24;
    let x = 0;
    for (let i = 0; i < display.length; i++) {
      const ch = display[i];
      if (ch === '.' || ch === ',') {
        x += drawYieldSep(svg, x, ch, h) + 2;
      } else {
        x += drawYieldDigit(svg, x, ch) + 3;
      }
    }
    // gesamten Block innerhalb des Fensters (x 110-290, y 300-344) horizontal zentrieren
    const windowLeft = 110, windowWidth = 180, windowTop = 300, padY = 4;
    const startX = windowLeft + (windowWidth - x) / 2;
    svg.setAttribute('transform', 'translate(' + startX.toFixed(1) + ',' + (windowTop + padY).toFixed(1) + ')');
    const unit = document.getElementById('yieldUnit');
    unit.setAttribute('x', (windowLeft + windowWidth / 2).toFixed(1));
    unit.setAttribute('y', (windowTop + padY + h + 16).toFixed(1));
    unit.setAttribute('text-anchor', 'middle');
    unit.textContent = 'MWh GESAMTERTRAG';
  }

  // ---- Data binding ----
  const needleGroup = document.getElementById('needleGroup');
  const statusDot = document.getElementById('statusDot');
  const statusText = document.getElementById('statusText');
  const mwText = document.getElementById('mwText');
  const gearBtn = document.getElementById('gearBtn');
  const overlay = document.getElementById('settingsOverlay');
  const urlInput = document.getElementById('urlInput');
  const saveBtn = document.getElementById('saveBtn');
  const lastUpdateBox = document.getElementById('lastUpdateBox');

  function formatPair(kw) {
    const isNegative = kw < 0;
    const absKw = Math.min(19999, Math.round(Math.abs(kw)));
    const kwStr = (isNegative ? '-' : '') + String(absKw);
    const mwValue = (absKw / 1000).toFixed(3);
    const mwStr = (isNegative ? '\u2212' : '') + mwValue.replace('.', ',') + ' MW';
    return { isNegative, kwStr, mwStr };
  }
  function setLive(isLive) {
    statusDot.className = 'dot ' + (isLive ? 'live' : 'demo');
    statusText.textContent = isLive ? 'LIVE' : 'DEMO';
  }

  let demoKw = 2441, demoDirection = 1;
  let currentAngle = START_ANGLE;
  let totalYieldMwh = 128456.734;
  let lastTickMs = Date.now();

  async function tick() {
    const endpoint = localStorage.getItem(STORAGE_KEY) || 'https://api.npoint.io/671e4ab68138cc5a0c82';
    let kw = null;
    let liveYieldMwh = null;
    let liveWindMs = null;
    if (endpoint) {
      try {
        const res = await fetch(endpoint, { cache: 'no-store' });
        const data = await res.json();
        const raw = parseFloat(data.leistung_kw ?? data.leistung_mw * 1000 ?? data.value ?? data);
        if (isNaN(raw)) throw new Error('Ungültige Antwort');
        kw = raw;
        setLive(true);
        if (data.gesamtertrag_mwh !== undefined && data.gesamtertrag_mwh !== null) {
          const y = parseFloat(data.gesamtertrag_mwh);
          if (!isNaN(y)) liveYieldMwh = y;
        }
        if (data.wind_ms !== undefined && data.wind_ms !== null) {
          const w = parseFloat(data.wind_ms);
          if (!isNaN(w)) liveWindMs = w;
        }
        if (data.ts !== undefined && data.ts !== null) {
          const tsNum = parseInt(data.ts, 10);
          if (!isNaN(tsNum)) {
            lastUpdateBox.textContent = 'Aktualisiert: ' + new Date(tsNum).toLocaleTimeString('de-DE');
          }
        }
      } catch (e) { setLive(false); kw = null; }
    } else { setLive(false); }

    if (liveWindMs !== null && typeof window.setWindText === 'function') {
      window.setWindText(liveWindMs.toFixed(1).replace('.', ',') + ' m/s');
    }

    if (kw === null) {
      demoKw += demoDirection * (Math.random() * 70);
      if (demoKw > 11700 || demoKw < -300) demoDirection *= -1;
      kw = demoKw;
    }

    const { kwStr, mwStr } = formatPair(kw);
    renderKwDigits(kwStr);
    mwText.textContent = mwStr;

    const nowMs = Date.now();
    const elapsedHours = Math.max(0, (nowMs - lastTickMs) / 3600000);
    lastTickMs = nowMs;
    if (liveYieldMwh !== null) {
      // Echter Gesamtertrag von clavis - wird direkt übernommen, nicht hochgezählt.
      totalYieldMwh = liveYieldMwh;
    } else if (kw > 0) {
      // Kein Live-Wert vorhanden (z.B. Demo-Modus) - wie bisher simulieren.
      totalYieldMwh += (kw / 1000) * elapsedHours;
    }
    renderYieldDigits(formatYield(totalYieldMwh));

    const targetAngle = valueToAngle(kw / 1000);
    updateLeds(kw / 1000);
    currentAngle = targetAngle;
    needleGroup.setAttribute('transform', 'translate(200,200) rotate(' + currentAngle.toFixed(1) + ')');
  }

  needleGroup.style.transition = 'transform 0.9s cubic-bezier(.34,1.4,.4,1)';
  tick();
  setInterval(tick, 2000);

  gearBtn.addEventListener('click', () => {
    urlInput.value = localStorage.getItem(STORAGE_KEY) || '';
    overlay.classList.add('open');
  });
  saveBtn.addEventListener('click', () => {
    const v = urlInput.value.trim();
    if (v) localStorage.setItem(STORAGE_KEY, v); else localStorage.removeItem(STORAGE_KEY);
    overlay.classList.remove('open');
    tick();
  });
  overlay.addEventListener('click', (e) => { if (e.target === overlay) overlay.classList.remove('open'); });

  if ('wakeLock' in navigator) {
    document.body.addEventListener('click', async function requestWakeLock() {
      try { await navigator.wakeLock.request('screen'); } catch (e) {}
    }, { once: true });
  }
})();
</script>

<script>
(function () {
  const NS = 'http://www.w3.org/2000/svg';
  // 5x7 dot-matrix font. Slash redrawn as a clean unbroken diagonal (no flat "feet" at the ends).
  const FONT = {
    '0':['01110','10001','10011','10101','11001','10001','01110'],
    '1':['00100','01100','00100','00100','00100','00100','01110'],
    '2':['01110','10001','00001','00010','00100','01000','11111'],
    '3':['11111','00010','00100','00010','00001','10001','01110'],
    '4':['00010','00110','01010','10010','11111','00010','00010'],
    '5':['11111','10000','11110','00001','00001','10001','01110'],
    '6':['00110','01000','10000','11110','10001','10001','01110'],
    '7':['11111','00001','00010','00100','01000','01000','01000'],
    '8':['01110','10001','10001','01110','10001','10001','01110'],
    '9':['01110','10001','10001','01111','00001','00010','01100'],
    ',':['00000','00000','00000','00000','00110','00100','01000'],
    '.':['00000','00000','00000','00000','00000','01100','01100'],
    '/':['00001','00010','00010','00100','01000','01000','10000'],
    ' ':['00000','00000','00000','00000','00000','00000','00000'],
    'm':['00000','00000','11010','10101','10101','10101','10101'],
    's':['00000','00000','01111','10000','01110','00001','11110']
  };
  function renderDotText(container, text, opts) {
    opts = opts || {};
    const dot = opts.dot || 2.6, pitch = opts.pitch || 3.4, colGap = opts.colGap || 1.5;
    const color = opts.color || '#ffb454';
    while (container.firstChild) container.removeChild(container.firstChild);
    let xCursor = 0;
    for (let i = 0; i < text.length; i++) {
      const glyph = FONT[text[i]] || FONT[' '];
      for (let row = 0; row < 7; row++) {
        const bits = glyph[row];
        for (let col = 0; col < 5; col++) {
          if (bits[col] === '1') {
            const c = document.createElementNS(NS, 'circle');
            c.setAttribute('cx', (xCursor + col * pitch).toFixed(2));
            c.setAttribute('cy', (row * pitch).toFixed(2));
            c.setAttribute('r', (dot / 2).toFixed(2));
            c.setAttribute('fill', color);
            container.appendChild(c);
          }
        }
      }
      xCursor += 5 * pitch + colGap * pitch;
    }
    return xCursor - colGap * pitch;
  }
  function setWindText(text) {
    const svg = document.getElementById('windDotsSvg');
    const g = document.getElementById('windDots');
    if (!g || !svg) return;
    const opts = { dot: 1.3, pitch: 2.45, colGap: 1.3, color: '#8dffe8' };
    const tmp = document.createElementNS(NS, 'g');
    const totalW = renderDotText(tmp, text, opts);
    const totalH = 6 * opts.pitch;
    svg.setAttribute('viewBox', '0 0 ' + (totalW + 4).toFixed(1) + ' ' + (totalH + 4).toFixed(1));
    svg.setAttribute('width', (totalW + 4).toFixed(1));
    svg.setAttribute('height', (totalH + 4).toFixed(1));
    g.setAttribute('transform', 'translate(2,2)');
    renderDotText(g, text, opts);
  }
  setWindText('6,4 m/s');
  window.setWindText = setWindText;
})();
</script>

</body>
</html>
