# Калькулятор підмереж IP
 
<div class="subnet-calc-wrapper">
 
<div class="calc-input-row">
  <div class="calc-field">
    <label>IP-адреса</label>
    <input type="text" id="ip-input" placeholder="192.168.1.0" oninput="calculate()" />
  </div>
  <div class="calc-field calc-field--slash">
    <label>Префікс</label>
    <div class="slash-wrap">
      <span>/</span>
      <input type="number" id="prefix-input" min="0" max="32" value="24" oninput="calculate()" />
    </div>
  </div>
  <div class="calc-field">
    <label>Маска підмережі</label>
    <input type="text" id="mask-input" placeholder="255.255.255.0" oninput="calculateFromMask()" />
  </div>
</div>
 
<div id="calc-error" class="calc-error" style="display:none"></div>
 
<div id="calc-results" class="calc-results">
 
<div class="results-grid">
  <div class="result-card">
    <div class="result-label">Адреса мережі</div>
    <div class="result-value" id="res-network">—</div>
    <div class="result-sub" id="res-network-bin">—</div>
  </div>
  <div class="result-card">
    <div class="result-label">Broadcast-адреса</div>
    <div class="result-value" id="res-broadcast">—</div>
    <div class="result-sub" id="res-broadcast-bin">—</div>
  </div>
  <div class="result-card">
    <div class="result-label">Перший хост</div>
    <div class="result-value" id="res-first">—</div>
  </div>
  <div class="result-card">
    <div class="result-label">Останній хост</div>
    <div class="result-value" id="res-last">—</div>
  </div>
  <div class="result-card">
    <div class="result-label">Кількість хостів</div>
    <div class="result-value" id="res-hosts">—</div>
    <div class="result-sub" id="res-total">—</div>
  </div>
  <div class="result-card">
    <div class="result-label">Wildcard-маска</div>
    <div class="result-value" id="res-wildcard">—</div>
  </div>
  <div class="result-card result-card--wide">
    <div class="result-label">Маска (двійкова)</div>
    <div class="result-value result-value--mono" id="res-mask-bin">—</div>
  </div>
  <div class="result-card result-card--wide">
    <div class="result-label">IP-адреса (двійкова)</div>
    <div class="result-value result-value--mono" id="res-ip-bin">—</div>
  </div>
</div>
 
<div class="cidr-strip">
  <div class="cidr-label">Швидкий вибір префіксу:</div>
  <div class="cidr-buttons" id="cidr-buttons"></div>
</div>
 
</div>
</div>
 
<style>
.subnet-calc-wrapper {
  margin: 1.5rem 0;
  font-family: var(--md-text-font, sans-serif);
}
.calc-input-row {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
  margin-bottom: 1rem;
  align-items: flex-end;
}
.calc-field {
  display: flex;
  flex-direction: column;
  gap: 0.35rem;
  flex: 1;
  min-width: 160px;
}
.calc-field--slash { max-width: 120px; }
.calc-field label {
  font-size: 0.75rem;
  font-weight: 500;
  color: var(--md-default-fg-color--light, #666);
  text-transform: uppercase;
  letter-spacing: 0.04em;
}
.calc-field input {
  height: 42px;
  padding: 0 0.75rem;
  border-radius: 6px;
  border: 1px solid var(--md-default-fg-color--lighter, #ccc);
  background: var(--md-default-bg-color, #fff);
  color: var(--md-default-fg-color, #222);
  font-size: 0.95rem;
  font-family: var(--md-code-font, monospace);
  outline: none;
  transition: border-color .15s;
  width: 100%;
  box-sizing: border-box;
}
.calc-field input:focus {
  border-color: var(--md-primary-fg-color, #3f51b5);
}
.slash-wrap {
  display: flex;
  align-items: center;
  gap: 0.4rem;
}
.slash-wrap span {
  font-size: 1.4rem;
  color: var(--md-default-fg-color--light, #888);
  font-weight: 300;
  line-height: 42px;
}
.slash-wrap input { width: 70px; text-align: center; }
.calc-error {
  padding: 0.6rem 1rem;
  border-radius: 6px;
  background: var(--md-typeset-color, #f44336);
  color: #fff;
  font-size: 0.85rem;
  margin-bottom: 1rem;
}
.results-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 0.75rem;
  margin-bottom: 1rem;
}
.result-card {
  background: var(--md-code-bg-color, #f5f5f5);
  border-radius: 8px;
  padding: 0.75rem 1rem;
  border: 1px solid var(--md-default-fg-color--lighter, #e0e0e0);
}
.result-card--wide {
  grid-column: 1 / -1;
}
.result-label {
  font-size: 0.72rem;
  font-weight: 500;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: var(--md-default-fg-color--light, #888);
  margin-bottom: 0.3rem;
}
.result-value {
  font-size: 1.05rem;
  font-weight: 600;
  color: var(--md-default-fg-color, #222);
  font-family: var(--md-code-font, monospace);
}
.result-value--mono {
  font-size: 0.85rem;
  word-break: break-all;
  letter-spacing: 0.05em;
}
.result-sub {
  font-size: 0.72rem;
  color: var(--md-default-fg-color--light, #888);
  font-family: var(--md-code-font, monospace);
  margin-top: 0.2rem;
  word-break: break-all;
}
.cidr-strip {
  border-top: 1px solid var(--md-default-fg-color--lighter, #e0e0e0);
  padding-top: 0.85rem;
}
.cidr-label {
  font-size: 0.75rem;
  color: var(--md-default-fg-color--light, #888);
  margin-bottom: 0.5rem;
  font-weight: 500;
  text-transform: uppercase;
  letter-spacing: 0.04em;
}
.cidr-buttons {
  display: flex;
  flex-wrap: wrap;
  gap: 0.4rem;
}
.cidr-btn {
  padding: 0.25rem 0.6rem;
  border-radius: 5px;
  border: 1px solid var(--md-default-fg-color--lighter, #ccc);
  background: var(--md-default-bg-color, #fff);
  color: var(--md-default-fg-color, #333);
  font-size: 0.8rem;
  font-family: var(--md-code-font, monospace);
  cursor: pointer;
  transition: background .12s, border-color .12s;
}
.cidr-btn:hover, .cidr-btn.active {
  background: var(--md-primary-fg-color, #3f51b5);
  color: #fff;
  border-color: var(--md-primary-fg-color, #3f51b5);
}
@media (max-width: 600px) {
  .calc-input-row { flex-direction: column; }
  .calc-field--slash { max-width: 100%; }
}
</style>
 
<script>
(function() {
 
function ipToInt(ip) {
  const parts = ip.trim().split('.');
  if (parts.length !== 4) return null;
  let n = 0;
  for (let i = 0; i < 4; i++) {
    const b = parseInt(parts[i], 10);
    if (isNaN(b) || b < 0 || b > 255) return null;
    n = (n * 256) + b;
  }
  return n >>> 0;
}
 
function intToIp(n) {
  n = n >>> 0;
  return [(n >>> 24) & 255, (n >>> 16) & 255, (n >>> 8) & 255, n & 255].join('.');
}
 
function ipToBin(ip) {
  return ip.split('.').map(o => parseInt(o).toString(2).padStart(8, '0')).join('.');
}
 
function prefixToMask(prefix) {
  if (prefix === 0) return 0;
  return (0xFFFFFFFF << (32 - prefix)) >>> 0;
}
 
function maskToPrefix(mask) {
  let n = mask >>> 0;
  let count = 0;
  while (n & 0x80000000) { count++; n = (n << 1) >>> 0; }
  return count;
}
 
function showError(msg) {
  const el = document.getElementById('calc-error');
  el.textContent = msg;
  el.style.display = msg ? 'block' : 'none';
}
 
function setResult(id, val) {
  const el = document.getElementById(id);
  if (el) el.textContent = val;
}
 
function formatHostCount(n) {
  if (n >= 1000000) return (n / 1000000).toFixed(2) + ' млн';
  if (n >= 1000) return n.toLocaleString('uk-UA');
  return n.toString();
}
 
function compute(ipInt, prefix) {
  const mask = prefixToMask(prefix);
  const wildcard = (~mask) >>> 0;
  const network = (ipInt & mask) >>> 0;
  const broadcast = (network | wildcard) >>> 0;
  const firstHost = prefix < 31 ? network + 1 : network;
  const lastHost  = prefix < 31 ? broadcast - 1 : broadcast;
  const totalAddr = Math.pow(2, 32 - prefix);
  const usableHosts = prefix < 31 ? totalAddr - 2 : totalAddr;
 
  setResult('res-network',    intToIp(network) + '/' + prefix);
  setResult('res-network-bin', ipToBin(intToIp(network)));
  setResult('res-broadcast',  intToIp(broadcast));
  setResult('res-broadcast-bin', ipToBin(intToIp(broadcast)));
  setResult('res-first',      prefix <= 30 ? intToIp(firstHost) : '—');
  setResult('res-last',       prefix <= 30 ? intToIp(lastHost)  : '—');
  setResult('res-hosts',      prefix <= 30 ? formatHostCount(usableHosts) : '—');
  setResult('res-total',      'Всього адрес: ' + formatHostCount(totalAddr));
  setResult('res-wildcard',   intToIp(wildcard));
  setResult('res-mask-bin',   ipToBin(intToIp(mask)));
  setResult('res-ip-bin',     ipToBin(intToIp(ipInt)));
 
  document.querySelectorAll('.cidr-btn').forEach(b => {
    b.classList.toggle('active', parseInt(b.dataset.prefix) === prefix);
  });
 
  document.getElementById('mask-input').value = intToIp(mask);
  showError('');
}
 
window.calculate = function() {
  const ipRaw = document.getElementById('ip-input').value.trim();
  const prefixRaw = document.getElementById('prefix-input').value.trim();
  if (!ipRaw) return;
  const ipInt = ipToInt(ipRaw);
  if (ipInt === null) { showError('Невірний формат IP-адреси. Приклад: 192.168.1.0'); return; }
  const prefix = parseInt(prefixRaw, 10);
  if (isNaN(prefix) || prefix < 0 || prefix > 32) { showError('Префікс повинен бути від 0 до 32'); return; }
  compute(ipInt, prefix);
};
 
window.calculateFromMask = function() {
  const maskRaw = document.getElementById('mask-input').value.trim();
  if (!maskRaw) return;
  const maskInt = ipToInt(maskRaw);
  if (maskInt === null) { showError('Невірний формат маски. Приклад: 255.255.255.0'); return; }
  const prefix = maskToPrefix(maskInt);
  document.getElementById('prefix-input').value = prefix;
  calculate();
};
 
function buildCidrButtons() {
  const common = [8,16,24,25,26,27,28,29,30,32];
  const container = document.getElementById('cidr-buttons');
  common.forEach(p => {
    const mask = prefixToMask(p);
    const btn = document.createElement('button');
    btn.className = 'cidr-btn';
    btn.dataset.prefix = p;
    btn.title = intToIp(mask);
    btn.textContent = '/' + p;
    btn.onclick = () => {
      document.getElementById('prefix-input').value = p;
      calculate();
    };
    container.appendChild(btn);
  });
}
 
buildCidrButtons();
document.getElementById('ip-input').value = '192.168.1.0';
document.getElementById('prefix-input').value = '24';
calculate();
 
})();
</script>