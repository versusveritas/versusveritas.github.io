<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Image Zone Selector – JotForm Widget</title>
<style>
* { box-sizing: border-box; margin: 0; padding: 0; }

body {
  font-family: Arial, sans-serif;
  background: #fff;
  padding: 12px;
  font-size: 14px;
}

#widget-container {
  width: 100%;
  max-width: 720px;
  margin: 0 auto;
}

/* ── Toolbar ── */
#toolbar {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 8px;
  flex-wrap: wrap;
}

.tb-btn {
  padding: 5px 12px;
  border: 1px solid #ccc;
  border-radius: 4px;
  background: #f5f5f5;
  cursor: pointer;
  font-size: 13px;
  line-height: 1.4;
}
.tb-btn:hover { background: #e8e8e8; }
.tb-btn.active { background: #1d4ed8; color: #fff; border-color: #1d4ed8; }
.tb-btn.danger { background: #fee2e2; border-color: #fca5a5; color: #b91c1c; }
.tb-btn.danger:hover { background: #fecaca; }

#mode-label {
  margin-left: auto;
  font-size: 12px;
  color: #666;
  background: #f0f0f0;
  padding: 4px 10px;
  border-radius: 4px;
}
#mode-label.select-mode { background: #dbeafe; color: #1d4ed8; }

/* ── Image wrapper ── */
#image-wrapper {
  position: relative;
  display: inline-block;
  width: 100%;
  user-select: none;
  border: 1px solid #ccc;
  border-radius: 4px;
  overflow: hidden;
  background: #f0f0f0;
  cursor: crosshair;
}

#image-wrapper.select-mode { cursor: default; }

#zone-image {
  display: block;
  width: 100%;
  height: auto;
  pointer-events: none;
  -webkit-user-drag: none;
}

/* ── Zone overlays ── */
.zone-overlay {
  position: absolute;
  border: 2px solid rgba(255,255,255,0.7);
  background: rgba(59,130,246,0.15);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background 0.12s;
}

.zone-overlay.select-mode {
  cursor: pointer;
}
.zone-overlay.select-mode:hover {
  background: rgba(59,130,246,0.32);
}
.zone-overlay.selected {
  background: rgba(59,130,246,0.55);
  border-color: #fff;
}

.zone-overlay.build-mode {
  cursor: default;
}

.zone-label {
  font-size: 11px;
  font-weight: bold;
  color: #fff;
  background: rgba(0,0,0,0.5);
  padding: 2px 6px;
  border-radius: 3px;
  text-align: center;
  pointer-events: none;
  max-width: 90%;
  word-break: break-word;
  line-height: 1.3;
}

.zone-delete-btn {
  position: absolute;
  top: 2px;
  right: 2px;
  width: 18px;
  height: 18px;
  background: rgba(185,28,28,0.85);
  color: #fff;
  border: none;
  border-radius: 3px;
  font-size: 12px;
  line-height: 1;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0;
}
.zone-delete-btn:hover { background: #b91c1c; }

/* ── Drag preview ── */
#drag-preview {
  position: absolute;
  border: 2px dashed #1d4ed8;
  background: rgba(59,130,246,0.15);
  pointer-events: none;
  display: none;
}

/* ── Name modal ── */
#name-modal-backdrop {
  display: none;
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.35);
  z-index: 100;
  align-items: center;
  justify-content: center;
}
#name-modal-backdrop.open { display: flex; }

#name-modal {
  background: #fff;
  border-radius: 8px;
  padding: 20px 24px;
  min-width: 280px;
  box-shadow: 0 4px 24px rgba(0,0,0,0.18);
}
#name-modal h3 { margin-bottom: 12px; font-size: 15px; }
#name-modal input {
  width: 100%;
  padding: 7px 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 14px;
  margin-bottom: 14px;
}
#name-modal input:focus { outline: 2px solid #1d4ed8; border-color: transparent; }
.modal-btns { display: flex; gap: 8px; justify-content: flex-end; }

/* ── Config export ── */
#config-section {
  margin-top: 10px;
  display: none;
}
#config-section textarea {
  width: 100%;
  height: 80px;
  font-family: monospace;
  font-size: 11px;
  border: 1px solid #ccc;
  border-radius: 4px;
  padding: 6px 8px;
  resize: vertical;
  color: #333;
}
#config-section label {
  font-size: 12px;
  color: #555;
  display: block;
  margin-bottom: 4px;
}

/* ── Summary ── */
#selection-summary {
  margin-top: 8px;
  font-size: 13px;
  color: #444;
  min-height: 20px;
}
#selection-summary span { font-weight: bold; color: #1d4ed8; }

#no-image-msg {
  padding: 40px;
  text-align: center;
  color: #888;
  border: 1px dashed #ccc;
  border-radius: 4px;
}
</style>
</head>
<body>

<div id="widget-container">

  <div id="no-image-msg" style="display:none;">
    No image URL provided. Set <code>imageUrl</code> in the widget URL, e.g.<br>
    <code>widget.html?imageUrl=https://i.imgur.com/3GkBEC8.png</code>
  </div>

  <div id="toolbar">
    <button class="tb-btn active" id="btn-build">✏️ Build mode</button>
    <button class="tb-btn" id="btn-select">✅ Select mode</button>
    <button class="tb-btn danger" id="btn-clear-sel" style="display:none;">Clear selection</button>
    <button class="tb-btn" id="btn-export" style="display:none;">Export config</button>
    <span id="mode-label">Build — drag to draw zones</span>
  </div>

  <div id="image-wrapper" style="display:none;">
    <img id="zone-image" alt="Zone image" draggable="false" />
    <div id="drag-preview"></div>
  </div>

  <div id="config-section">
    <label>Zone config — paste this into the <code>zones</code> URL param to pre-load zones:</label>
    <textarea id="config-output" readonly></textarea>
  </div>

  <div id="selection-summary" style="display:none;">No zones selected.</div>

</div>

<!-- Name modal -->
<div id="name-modal-backdrop">
  <div id="name-modal">
    <h3>Name this zone</h3>
    <input type="text" id="zone-name-input" placeholder="e.g. Left shoulder" maxlength="60" />
    <div class="modal-btns">
      <button class="tb-btn" id="modal-cancel">Cancel</button>
      <button class="tb-btn active" id="modal-confirm">Add zone</button>
    </div>
  </div>
</div>

<script>
// ── Parse URL params ──────────────────────────────────────────────
function getParam(name) {
  var results = new RegExp('[?&]' + name + '=([^&#]*)').exec(window.location.href);
  return results ? decodeURIComponent(results[1]) : null;
}

var imageUrl = getParam('imageUrl') || getParam('src') || getParam('image') || '';
var zonesParam = getParam('zones') || '';

// ── State ─────────────────────────────────────────────────────────
// zones: [{ label, x, y, w, h }]  all values in % of image
var zones = [];
var selectedZones = new Set();
var currentMode = 'build'; // 'build' | 'select'

// drag state
var dragging = false;
var dragStart = null;
var pendingRect = null; // {x,y,w,h} in % awaiting name

// ── Elements ──────────────────────────────────────────────────────
var $wrapper    = document.getElementById('image-wrapper');
var $img        = document.getElementById('zone-image');
var $preview    = document.getElementById('drag-preview');
var $noImg      = document.getElementById('no-image-msg');
var $toolbar    = document.getElementById('toolbar');
var $summary    = document.getElementById('selection-summary');
var $configSec  = document.getElementById('config-section');
var $configOut  = document.getElementById('config-output');
var $modeLabel  = document.getElementById('mode-label');
var $btnBuild   = document.getElementById('btn-build');
var $btnSelect  = document.getElementById('btn-select');
var $btnClearSel = document.getElementById('btn-clear-sel');
var $btnExport  = document.getElementById('btn-export');
var $backdrop   = document.getElementById('name-modal-backdrop');
var $nameInput  = document.getElementById('zone-name-input');
var $modalCancel = document.getElementById('modal-cancel');
var $modalConfirm = document.getElementById('modal-confirm');

// ── Boot ──────────────────────────────────────────────────────────
if (!imageUrl) {
  $noImg.style.display = 'block';
} else {
  $img.src = imageUrl;
  $img.onload = function() {
    $wrapper.style.display = 'inline-block';
    if (zonesParam) {
      try { zones = JSON.parse(zonesParam); } catch(e) {}
    }
    renderAllZones();
    setMode('build');
  };
  $img.onerror = function() {
    $noImg.textContent = 'Failed to load image. Check the imageUrl parameter.';
    $noImg.style.display = 'block';
  };
}

// ── Mode switching ─────────────────────────────────────────────────
function setMode(mode) {
  currentMode = mode;
  if (mode === 'build') {
    $btnBuild.classList.add('active');
    $btnSelect.classList.remove('active');
    $wrapper.classList.remove('select-mode');
    $wrapper.classList.add('build-mode');
    $modeLabel.textContent = 'Build — drag to draw zones';
    $modeLabel.classList.remove('select-mode');
    $btnClearSel.style.display = 'none';
    $btnExport.style.display = zones.length ? 'inline-block' : 'none';
    $summary.style.display = 'none';
    $configSec.style.display = 'none';
  } else {
    $btnSelect.classList.add('active');
    $btnBuild.classList.remove('active');
    $wrapper.classList.add('select-mode');
    $wrapper.classList.remove('build-mode');
    $modeLabel.textContent = 'Select — click zones to toggle';
    $modeLabel.classList.add('select-mode');
    $btnClearSel.style.display = selectedZones.size ? 'inline-block' : 'none';
    $btnExport.style.display = 'none';
    $summary.style.display = 'block';
    $configSec.style.display = 'none';
    updateSummary();
  }
  renderAllZones();
}

$btnBuild.addEventListener('click', function() { setMode('build'); });
$btnSelect.addEventListener('click', function() { setMode('select'); });
$btnClearSel.addEventListener('click', function() {
  selectedZones.clear();
  $btnClearSel.style.display = 'none';
  renderAllZones();
  updateSummary();
  reportValue();
});
$btnExport.addEventListener('click', function() {
  $configSec.style.display = $configSec.style.display === 'none' ? 'block' : 'none';
  $configOut.value = JSON.stringify(zones);
});

// ── Drag to draw ───────────────────────────────────────────────────
function getRelativePos(e) {
  var rect = $wrapper.getBoundingClientRect();
  var clientX = e.touches ? e.touches[0].clientX : e.clientX;
  var clientY = e.touches ? e.touches[0].clientY : e.clientY;
  return {
    x: Math.max(0, Math.min(100, (clientX - rect.left) / rect.width * 100)),
    y: Math.max(0, Math.min(100, (clientY - rect.top)  / rect.height * 100))
  };
}

$wrapper.addEventListener('mousedown', onDragStart);
$wrapper.addEventListener('touchstart', onDragStart, { passive: false });

function onDragStart(e) {
  if (currentMode !== 'build') return;
  if (e.target.classList.contains('zone-delete-btn')) return;
  e.preventDefault();
  dragging = true;
  dragStart = getRelativePos(e);
  $preview.style.display = 'block';
  updatePreview(dragStart, dragStart);
}

document.addEventListener('mousemove', onDragMove);
document.addEventListener('touchmove', onDragMove, { passive: false });

function onDragMove(e) {
  if (!dragging) return;
  e.preventDefault();
  var cur = getRelativePos(e);
  updatePreview(dragStart, cur);
}

document.addEventListener('mouseup', onDragEnd);
document.addEventListener('touchend', onDragEnd);

function onDragEnd(e) {
  if (!dragging) return;
  dragging = false;
  $preview.style.display = 'none';

  var cur = getRelativePos(e.changedTouches ? { clientX: e.changedTouches[0].clientX, clientY: e.changedTouches[0].clientY } : e);
  var rect = normalizeRect(dragStart, cur);

  // ignore tiny drags (< 2% in either dim)
  if (rect.w < 2 || rect.h < 2) return;

  pendingRect = rect;
  openNameModal();
}

function updatePreview(a, b) {
  var r = normalizeRect(a, b);
  $preview.style.left   = r.x + '%';
  $preview.style.top    = r.y + '%';
  $preview.style.width  = r.w + '%';
  $preview.style.height = r.h + '%';
}

function normalizeRect(a, b) {
  return {
    x: Math.min(a.x, b.x),
    y: Math.min(a.y, b.y),
    w: Math.abs(b.x - a.x),
    h: Math.abs(b.y - a.y)
  };
}

// ── Name modal ────────────────────────────────────────────────────
function openNameModal() {
  $nameInput.value = '';
  $backdrop.classList.add('open');
  setTimeout(function() { $nameInput.focus(); }, 50);
}

function closeNameModal() {
  $backdrop.classList.remove('open');
  pendingRect = null;
}

$modalCancel.addEventListener('click', closeNameModal);

$backdrop.addEventListener('click', function(e) {
  if (e.target === $backdrop) closeNameModal();
});

$nameInput.addEventListener('keydown', function(e) {
  if (e.key === 'Enter') confirmZone();
  if (e.key === 'Escape') closeNameModal();
});

$modalConfirm.addEventListener('click', confirmZone);

function confirmZone() {
  var label = $nameInput.value.trim();
  if (!label || !pendingRect) return;
  zones.push({ label: label, x: pendingRect.x, y: pendingRect.y, w: pendingRect.w, h: pendingRect.h });
  closeNameModal();
  renderAllZones();
  $btnExport.style.display = 'inline-block';
  $configSec.style.display = 'none';
}

// ── Render zones ───────────────────────────────────────────────────
function renderAllZones() {
  // Remove existing overlays
  document.querySelectorAll('.zone-overlay').forEach(function(el) { el.remove(); });

  zones.forEach(function(zone, idx) {
    var el = document.createElement('div');
    el.className = 'zone-overlay ' + (currentMode === 'select' ? 'select-mode' : 'build-mode');
    el.style.left   = zone.x + '%';
    el.style.top    = zone.y + '%';
    el.style.width  = zone.w + '%';
    el.style.height = zone.h + '%';

    var lbl = document.createElement('div');
    lbl.className = 'zone-label';
    lbl.textContent = zone.label;
    el.appendChild(lbl);

    if (currentMode === 'build') {
      var delBtn = document.createElement('button');
      delBtn.className = 'zone-delete-btn';
      delBtn.textContent = '×';
      delBtn.title = 'Delete zone';
      delBtn.addEventListener('click', function(e) {
        e.stopPropagation();
        zones.splice(idx, 1);
        selectedZones.delete(idx);
        // Remap selectedZones indices above idx
        var newSel = new Set();
        selectedZones.forEach(function(i) {
          if (i < idx) newSel.add(i);
          else if (i > idx) newSel.add(i - 1);
        });
        selectedZones = newSel;
        renderAllZones();
        if (!zones.length) $btnExport.style.display = 'none';
        $configSec.style.display = 'none';
      });
      el.appendChild(delBtn);
    }

    if (currentMode === 'select') {
      if (selectedZones.has(idx)) el.classList.add('selected');
      el.addEventListener('click', function() {
        if (selectedZones.has(idx)) {
          selectedZones.delete(idx);
        } else {
          selectedZones.add(idx);
        }
        el.classList.toggle('selected');
        updateSummary();
        $btnClearSel.style.display = selectedZones.size ? 'inline-block' : 'none';
        reportValue();
      });
    }

    $wrapper.appendChild(el);
  });
}

// ── Summary & JotForm ─────────────────────────────────────────────
function updateSummary() {
  if (selectedZones.size === 0) {
    $summary.innerHTML = 'No zones selected.';
  } else {
    var labels = Array.from(selectedZones).map(function(i) { return zones[i].label; });
    $summary.innerHTML = 'Selected: <span>' + labels.join(', ') + '</span>';
  }
}

function reportValue() {
  var labels = Array.from(selectedZones).map(function(i) { return zones[i].label; });
  if (window.JFCustomWidget) {
    JFCustomWidget.sendSubmit({ value: labels.join(', '), valid: true });
  }
}

if (window.JFCustomWidget) {
  JFCustomWidget.subscribe('ready', function() {});
}
</script>
</body>
</html>
