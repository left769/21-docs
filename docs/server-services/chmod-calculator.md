# Калькулятор прав доступу Linux

<div class="chmod-wrapper">

<div class="chmod-grid">

  <!-- Owner -->
  <div class="chmod-block">
    <div class="chmod-block-title">
      <span class="chmod-icon">👤</span> Власник <span class="chmod-octet" id="oct-owner">7</span>
    </div>
    <label class="chmod-check"><input type="checkbox" id="ur" checked onchange="update()"><span class="perm-badge perm-r">r</span> Читання</label>
    <label class="chmod-check"><input type="checkbox" id="uw" checked onchange="update()"><span class="perm-badge perm-w">w</span> Запис</label>
    <label class="chmod-check"><input type="checkbox" id="ux" checked onchange="update()"><span class="perm-badge perm-x">x</span> Виконання</label>
  </div>

  <!-- Group -->
  <div class="chmod-block">
    <div class="chmod-block-title">
      <span class="chmod-icon">👥</span> Група <span class="chmod-octet" id="oct-group">5</span>
    </div>
    <label class="chmod-check"><input type="checkbox" id="gr" checked onchange="update()"><span class="perm-badge perm-r">r</span> Читання</label>
    <label class="chmod-check"><input type="checkbox" id="gw" onchange="update()"><span class="perm-badge perm-w">w</span> Запис</label>
    <label class="chmod-check"><input type="checkbox" id="gx" checked onchange="update()"><span class="perm-badge perm-x">x</span> Виконання</label>
  </div>

  <!-- Others -->
  <div class="chmod-block">
    <div class="chmod-block-title">
      <span class="chmod-icon">🌐</span> Інші <span class="chmod-octet" id="oct-other">5</span>
    </div>
    <label class="chmod-check"><input type="checkbox" id="or" checked onchange="update()"><span class="perm-badge perm-r">r</span> Читання</label>
    <label class="chmod-check"><input type="checkbox" id="ow" onchange="update()"><span class="perm-badge perm-w">w</span> Запис</label>
    <label class="chmod-check"><input type="checkbox" id="ox" checked onchange="update()"><span class="perm-badge perm-x">x</span> Виконання</label>
  </div>

</div>

<!-- Results -->
<div class="chmod-results">

  <div class="chmod-result-row">
    <div class="chmod-result-card">
      <div class="chmod-result-label">Числове значення</div>
      <div class="chmod-result-value" id="res-numeric">755</div>
    </div>
    <div class="chmod-result-card">
      <div class="chmod-result-label">Символьне значення</div>
      <div class="chmod-result-value" id="res-symbolic">rwxr-xr-x</div>
    </div>
  </div>

  <!-- Command template -->
  <div class="chmod-cmd-block">
    <div class="chmod-result-label">Шаблон команди</div>
    <div class="chmod-cmd-row">
      <div class="chmod-cmd-display">
        <span class="cmd-chmod">chmod</span><span class="cmd-rec" id="cmd-rec"></span>
        <span class="cmd-perm" id="cmd-perm">755</span>
        <span class="cmd-path" id="cmd-path-display">/шлях/до/файлу</span>
      </div>
      <button class="chmod-copy-btn" id="copy-btn" onclick="copyCommand()" title="Копіювати команду">
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="9" y="9" width="13" height="13" rx="2"/><path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"/></svg>
        Копіювати
      </button>
    </div>
    <div class="chmod-path-row">
      <label class="chmod-path-label">Вкажи свій шлях:</label>
      <input type="text" id="cmd-path-input" placeholder="/var/www/html" oninput="updatePath()" />
      <label class="chmod-check chmod-recursive">
        <input type="checkbox" id="recursive" onchange="update()"> <code>-R</code> рекурсивно
      </label>
    </div>
  </div>

</div>

<!-- Direct input -->
<div class="chmod-direct">
  <div class="chmod-direct-title">Або введи значення напряму:</div>
  <div class="chmod-direct-row">
    <div class="chmod-direct-field">
      <label>Числове (напр. <code>644</code>)</label>
      <input type="text" id="input-numeric" maxlength="3" placeholder="755" oninput="fromNumeric()" />
    </div>
    <div class="chmod-direct-field">
      <label>Символьне (напр. <code>rw-r--r--</code>)</label>
      <input type="text" id="input-symbolic" maxlength="9" placeholder="rwxr-xr-x" oninput="fromSymbolic()" />
    </div>
  </div>
  <div class="chmod-direct-error" id="direct-error"></div>
</div>

<!-- Presets -->
<div class="chmod-presets">
  <div class="chmod-presets-title">Типові значення:</div>
  <div class="chmod-presets-row">
    <button class="chmod-preset-btn" onclick="applyPreset(7,5,5)"><code>755</code><span>Директорії, скрипти</span></button>
    <button class="chmod-preset-btn" onclick="applyPreset(6,4,4)"><code>644</code><span>Файли конфігурацій</span></button>
    <button class="chmod-preset-btn" onclick="applyPreset(6,0,0)"><code>600</code><span>SSH ключі</span></button>
    <button class="chmod-preset-btn" onclick="applyPreset(7,0,0)"><code>700</code><span>Приватні скрипти</span></button>
    <button class="chmod-preset-btn" onclick="applyPreset(7,7,7)"><code>777</code><span>Повний доступ ⚠️</span></button>
    <button class="chmod-preset-btn" onclick="applyPreset(4,4,4)"><code>444</code><span>Тільки читання</span></button>
  </div>
</div>

<!-- Reference table -->
<details class="chmod-ref">
  <summary>📖 Довідка: таблиця значень</summary>
  <div class="chmod-ref-content">
    <table>
      <thead><tr><th>Цифра</th><th>Права</th><th>Символи</th><th>Пояснення</th></tr></thead>
      <tbody>
        <tr><td><code>0</code></td><td>---</td><td>0+0+0</td><td>Немає прав</td></tr>
        <tr><td><code>1</code></td><td>--x</td><td>0+0+1</td><td>Тільки виконання</td></tr>
        <tr><td><code>2</code></td><td>-w-</td><td>0+2+0</td><td>Тільки запис</td></tr>
        <tr><td><code>3</code></td><td>-wx</td><td>0+2+1</td><td>Запис + виконання</td></tr>
        <tr><td><code>4</code></td><td>r--</td><td>4+0+0</td><td>Тільки читання</td></tr>
        <tr><td><code>5</code></td><td>r-x</td><td>4+0+1</td><td>Читання + виконання</td></tr>
        <tr><td><code>6</code></td><td>rw-</td><td>4+2+0</td><td>Читання + запис</td></tr>
        <tr><td><code>7</code></td><td>rwx</td><td>4+2+1</td><td>Повний доступ</td></tr>
      </tbody>
    </table>
  </div>
</details>

</div>

<style>
.chmod-wrapper {
  margin: 1.5rem 0;
  font-family: var(--md-text-font, sans-serif);
}
.chmod-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
  margin-bottom: 1rem;
}
@media (max-width: 600px) { .chmod-grid { grid-template-columns: 1fr; } }
.chmod-block {
  background: var(--md-code-bg-color, #f5f5f5);
  border: 1px solid var(--md-default-fg-color--lighter, #e0e0e0);
  border-radius: 10px;
  padding: 1rem;
  display: flex;
  flex-direction: column;
  gap: 0.55rem;
}
.chmod-block-title {
  font-size: 0.9rem;
  font-weight: 600;
  color: var(--md-default-fg-color, #222);
  display: flex;
  align-items: center;
  gap: 0.4rem;
  margin-bottom: 0.3rem;
}
.chmod-icon { font-size: 1rem; }
.chmod-octet {
  margin-left: auto;
  background: var(--md-primary-fg-color, #3f51b5);
  color: #fff;
  border-radius: 6px;
  padding: 0.1rem 0.5rem;
  font-size: 1rem;
  font-family: var(--md-code-font, monospace);
  min-width: 1.6rem;
  text-align: center;
}
.chmod-check {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  cursor: pointer;
  font-size: 0.9rem;
  color: var(--md-default-fg-color, #333);
  user-select: none;
  padding: 0.15rem 0;
}
.chmod-check input[type=checkbox] { cursor: pointer; width: 16px; height: 16px; accent-color: var(--md-primary-fg-color, #3f51b5); }
.perm-badge {
  display: inline-block;
  width: 22px;
  height: 22px;
  line-height: 22px;
  text-align: center;
  border-radius: 4px;
  font-family: var(--md-code-font, monospace);
  font-size: 0.85rem;
  font-weight: 700;
}
.perm-r { background: #e3f2fd; color: #1565c0; }
.perm-w { background: #fce4ec; color: #c62828; }
.perm-x { background: #e8f5e9; color: #2e7d32; }

.chmod-results { margin-bottom: 1rem; }
.chmod-result-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
  margin-bottom: 1rem;
}
@media (max-width: 480px) { .chmod-result-row { grid-template-columns: 1fr; } }
.chmod-result-card {
  background: var(--md-code-bg-color, #f5f5f5);
  border: 1px solid var(--md-default-fg-color--lighter, #e0e0e0);
  border-radius: 8px;
  padding: 0.75rem 1rem;
}
.chmod-result-label {
  font-size: 0.72rem;
  font-weight: 500;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: var(--md-default-fg-color--light, #888);
  margin-bottom: 0.3rem;
}
.chmod-result-value {
  font-size: 1.4rem;
  font-weight: 700;
  font-family: var(--md-code-font, monospace);
  color: var(--md-default-fg-color, #111);
  letter-spacing: 0.05em;
}

.chmod-cmd-block {
  background: var(--md-code-bg-color, #f5f5f5);
  border: 1px solid var(--md-default-fg-color--lighter, #e0e0e0);
  border-radius: 8px;
  padding: 0.85rem 1rem;
}
.chmod-cmd-row {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  margin: 0.4rem 0 0.7rem;
  flex-wrap: wrap;
}
.chmod-cmd-display {
  flex: 1;
  background: var(--md-default-bg-color, #fff);
  border: 1px solid var(--md-default-fg-color--lighter, #ddd);
  border-radius: 6px;
  padding: 0.45rem 0.75rem;
  font-family: var(--md-code-font, monospace);
  font-size: 0.95rem;
  min-width: 0;
  white-space: nowrap;
  overflow-x: auto;
  color: var(--md-default-fg-color, #222);
}
.cmd-chmod { color: var(--md-primary-fg-color, #4c8dff); font-weight: 600; }
.cmd-perm  { color: #e65100; font-weight: 700; margin: 0 0.4rem; }
.cmd-path  { color: var(--md-default-fg-color--light, #888); }
.cmd-rec { color: #4c8dff; font-weight: 600; margin: 0 0.3rem 0 0.4rem; }
.chmod-copy-btn {
  display: flex;
  align-items: center;
  gap: 0.4rem;
  padding: 0.45rem 0.9rem;
  border-radius: 6px;
  border: 1px solid var(--md-primary-fg-color, #3f51b5);
  background: transparent;
  color: var(--md-primary-fg-color, #3f51b5);
  font-size: 0.85rem;
  cursor: pointer;
  white-space: nowrap;
  transition: background .12s, color .12s;
}
.chmod-copy-btn:hover { background: var(--md-primary-fg-color, #3f51b5); color: #fff; }
.chmod-copy-btn.copied { background: #2e7d32; border-color: #2e7d32; color: #fff; }
.chmod-path-row {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  flex-wrap: wrap;
}
.chmod-path-label { font-size: 0.8rem; color: var(--md-default-fg-color--light, #888); white-space: nowrap; }
.chmod-path-row input {
  flex: 1;
  min-width: 160px;
  height: 34px;
  padding: 0 0.65rem;
  border-radius: 6px;
  border: 1px solid var(--md-default-fg-color--lighter, #ccc);
  background: var(--md-default-bg-color, #fff);
  color: var(--md-default-fg-color, #222);
  font-family: var(--md-code-font, monospace);
  font-size: 0.88rem;
  outline: none;
  box-sizing: border-box;
}
.chmod-path-row input:focus { border-color: var(--md-primary-fg-color, #3f51b5); }
.chmod-recursive { font-size: 0.85rem; white-space: nowrap; }

.chmod-direct {
  background: var(--md-code-bg-color, #f5f5f5);
  border: 1px solid var(--md-default-fg-color--lighter, #e0e0e0);
  border-radius: 8px;
  padding: 0.85rem 1rem;
  margin-bottom: 1rem;
}
.chmod-direct-title {
  font-size: 0.8rem;
  font-weight: 600;
  color: var(--md-default-fg-color--light, #888);
  text-transform: uppercase;
  letter-spacing: 0.04em;
  margin-bottom: 0.6rem;
}
.chmod-direct-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
}
@media (max-width: 480px) { .chmod-direct-row { grid-template-columns: 1fr; } }
.chmod-direct-field { display: flex; flex-direction: column; gap: 0.3rem; }
.chmod-direct-field label { font-size: 0.8rem; color: var(--md-default-fg-color--light, #888); }
.chmod-direct-field input {
  height: 38px;
  padding: 0 0.7rem;
  border-radius: 6px;
  border: 1px solid var(--md-default-fg-color--lighter, #ccc);
  background: var(--md-default-bg-color, #fff);
  color: var(--md-default-fg-color, #222);
  font-family: var(--md-code-font, monospace);
  font-size: 0.95rem;
  outline: none;
  box-sizing: border-box;
}
.chmod-direct-field input:focus { border-color: var(--md-primary-fg-color, #3f51b5); }
.chmod-direct-error { font-size: 0.8rem; color: #c62828; margin-top: 0.4rem; min-height: 1rem; }

.chmod-presets { margin-bottom: 1rem; }
.chmod-presets-title {
  font-size: 0.75rem;
  font-weight: 500;
  text-transform: uppercase;
  letter-spacing: 0.04em;
  color: var(--md-default-fg-color--light, #888);
  margin-bottom: 0.5rem;
}
.chmod-presets-row { display: flex; flex-wrap: wrap; gap: 0.5rem; }
.chmod-preset-btn {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 0.45rem 0.9rem;
  border-radius: 8px;
  border: 1px solid var(--md-default-fg-color--lighter, #ddd);
  background: var(--md-default-bg-color, #fff);
  cursor: pointer;
  transition: border-color .12s, background .12s;
  gap: 0.1rem;
}
.chmod-preset-btn:hover { border-color: var(--md-primary-fg-color, #3f51b5); background: var(--md-code-bg-color, #f5f5f5); }
.chmod-preset-btn code { font-size: 1rem; font-weight: 700; color: var(--md-primary-fg-color, #3f51b5); }
.chmod-preset-btn span { font-size: 0.7rem; color: var(--md-default-fg-color--light, #888); white-space: nowrap; }

.chmod-ref {
  border: 1px solid var(--md-default-fg-color--lighter, #e0e0e0);
  border-radius: 8px;
  overflow: hidden;
}
.chmod-ref summary {
  padding: 0.7rem 1rem;
  cursor: pointer;
  font-size: 0.9rem;
  font-weight: 500;
  background: var(--md-code-bg-color, #f5f5f5);
  user-select: none;
  list-style: none;
}
.chmod-ref summary::-webkit-details-marker { display: none; }
.chmod-ref-content { padding: 0.75rem 1rem; overflow-x: auto; }
.chmod-ref-content table { width: 100%; border-collapse: collapse; font-size: 0.88rem; }
.chmod-ref-content th {
  text-align: left;
  padding: 0.4rem 0.75rem;
  background: var(--md-code-bg-color, #f5f5f5);
  border-bottom: 2px solid var(--md-default-fg-color--lighter, #ddd);
  font-size: 0.78rem;
  text-transform: uppercase;
  letter-spacing: 0.04em;
  color: var(--md-default-fg-color--light, #888);
}
.chmod-ref-content td {
  padding: 0.35rem 0.75rem;
  border-bottom: 1px solid var(--md-default-fg-color--lighter, #eee);
  font-family: var(--md-code-font, monospace);
}
.chmod-ref-content tr:last-child td { border-bottom: none; }
</style>

<script>
(function() {

function getOctet(r, w, x) {
  return (document.getElementById(r).checked ? 4 : 0)
       + (document.getElementById(w).checked ? 2 : 0)
       + (document.getElementById(x).checked ? 1 : 0);
}

function octetToSymbolic(n) {
  return (n & 4 ? 'r' : '-') + (n & 2 ? 'w' : '-') + (n & 1 ? 'x' : '-');
}

function setCheckboxes(prefix, n) {
  document.getElementById(prefix + 'r').checked = !!(n & 4);
  document.getElementById(prefix + 'w').checked = !!(n & 2);
  document.getElementById(prefix + 'x').checked = !!(n & 1);
}

function buildCommand(numeric) {
  const pathInput = document.getElementById('cmd-path-input').value.trim();
  const path = pathInput || '/шлях/до/файлу';
  const rec = document.getElementById('recursive').checked ? ' -R' : '';
  return 'chmod' + rec + ' ' + numeric + ' ' + path;
}

window.update = function() {
  const u = getOctet('ur','uw','ux');
  const g = getOctet('gr','gw','gx');
  const o = getOctet('or','ow','ox');

  document.getElementById('oct-owner').textContent = u;
  document.getElementById('oct-group').textContent = g;
  document.getElementById('oct-other').textContent = o;

  const numeric = '' + u + g + o;
  const symbolic = octetToSymbolic(u) + octetToSymbolic(g) + octetToSymbolic(o);

  document.getElementById('res-numeric').textContent = numeric;
  document.getElementById('res-symbolic').textContent = symbolic;
  document.getElementById('cmd-perm').textContent = numeric;
  document.getElementById('cmd-rec').textContent = document.getElementById('recursive').checked ? '-R' : '';

  const pathInput = document.getElementById('cmd-path-input').value.trim();
  document.getElementById('cmd-path-display').textContent = pathInput || '/шлях/до/файлу';

  const rec = document.getElementById('recursive').checked ? ' -R' : '';
  document.getElementById('cmd-path-display').previousSibling;

  document.getElementById('input-numeric').value = numeric;
  document.getElementById('input-symbolic').value = symbolic;
  document.getElementById('direct-error').textContent = '';
};

window.updatePath = function() {
  const pathInput = document.getElementById('cmd-path-input').value.trim();
  document.getElementById('cmd-path-display').textContent = pathInput || '/шлях/до/файлу';
};

window.copyCommand = function() {
  const numeric = document.getElementById('res-numeric').textContent;
  const cmd = buildCommand(numeric);
  navigator.clipboard.writeText(cmd).then(() => {
    const btn = document.getElementById('copy-btn');
    btn.classList.add('copied');
    btn.innerHTML = '<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><polyline points="20 6 9 17 4 12"/></svg> Скопійовано!';
    setTimeout(() => {
      btn.classList.remove('copied');
      btn.innerHTML = '<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="9" y="9" width="13" height="13" rx="2"/><path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"/></svg> Копіювати';
    }, 2000);
  });
};

window.applyPreset = function(u, g, o) {
  setCheckboxes('u', u);
  setCheckboxes('g', g);
  setCheckboxes('o', o);
  update();
};

window.fromNumeric = function() {
  const val = document.getElementById('input-numeric').value.trim();
  if (!/^[0-7]{3}$/.test(val)) {
    if (val.length === 3) document.getElementById('direct-error').textContent = 'Допустимі цифри: 0–7 (наприклад: 755)';
    return;
  }
  document.getElementById('direct-error').textContent = '';
  const digits = val.split('').map(Number);
  setCheckboxes('u', digits[0]);
  setCheckboxes('g', digits[1]);
  setCheckboxes('o', digits[2]);
  update();
  document.getElementById('input-symbolic').value = document.getElementById('res-symbolic').textContent;
};

window.fromSymbolic = function() {
  const val = document.getElementById('input-symbolic').value.trim();
  if (!/^[r\-][w\-][x\-][r\-][w\-][x\-][r\-][w\-][x\-]$/.test(val)) {
    if (val.length === 9) document.getElementById('direct-error').textContent = 'Формат: rwxrwxrwx (використовуй r, w, x або -)';
    return;
  }
  document.getElementById('direct-error').textContent = '';
  const charToVal = c => c !== '-' ? 1 : 0;
  const toOctet = (r,w,x) => charToVal(r)*4 + charToVal(w)*2 + charToVal(x);
  setCheckboxes('u', toOctet(val[0], val[1], val[2]));
  setCheckboxes('g', toOctet(val[3], val[4], val[5]));
  setCheckboxes('o', toOctet(val[6], val[7], val[8]));
  update();
  document.getElementById('input-numeric').value = document.getElementById('res-numeric').textContent;
};

update();

})();
</script>