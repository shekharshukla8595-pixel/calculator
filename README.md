# calculator
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Calculator</title>
  <style>
    :root {
      --bg: #0f172a;          /* slate-900 */
      --panel: #111827;       /* gray-900 */
      --panel-2: #1f2937;     /* gray-800 */
      --btn: #334155;         /* slate-700 */
      --btn-alt: #475569;     /* slate-600 */
      --accent: #22c55e;      /* green-500 */
      --danger: #ef4444;      /* red-500 */
      --text: #e5e7eb;        /* gray-200 */
      --muted: #94a3b8;       /* slate-400 */
      --shadow: 0 10px 30px rgba(0,0,0,.35);
      --radius: 20px;
    }

    * { box-sizing: border-box; }
    html, body { height: 100%; }
    body {
      margin: 0;
      font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial, "Apple Color Emoji", "Segoe UI Emoji";
      background: radial-gradient(1200px 700px at 80% -100px, #1e293b 0%, var(--bg) 50%);
      color: var(--text);
      display: grid;
      place-items: center;
      padding: 24px;
    }

    .wrap {
      width: min(420px, 95vw);
    }

    .calc {
      background: linear-gradient(180deg, var(--panel) 0%, var(--panel-2) 100%);
      border-radius: var(--radius);
      box-shadow: var(--shadow);
      overflow: hidden;
      border: 1px solid #1f2937;
    }

    .topbar {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 14px 18px;
      border-bottom: 1px solid #1f2937;
    }
    .title { font-size: 14px; color: var(--muted); letter-spacing: .08em; text-transform: uppercase; }
    .credits { font-size: 12px; color: var(--muted); }

    .display {
      padding: 10px;
      min-height: 120px;
      display: grid;
      align-content: end;
      gap: 8px;
      background: #0b1220;
      border-bottom: 1px solid #1f2937;
    }
    .expr {
      font-size: 40px;
      color: var(--muted);
      word-wrap: break-word;
      min-height: 28px;
    }
    .result {
      font-size: 44px;
      line-height: 1.1;
      font-weight: 700;
      word-wrap: break-word;
    }

    .keys {
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      gap: 10px;
      padding: 16px;
      background: linear-gradient(180deg, #0c1220, #0b0f1a);
    }

    button {
      appearance: none;
      border: none;
      padding: 18px 14px;
      border-radius: 16px;
      background: var(--btn);
      color: var(--text);
      font-size: 18px;
      font-weight: 600;
      cursor: pointer;
      transition: transform .06s ease, filter .2s ease, background .2s ease;
      box-shadow: inset 0 -2px 0 rgba(255,255,255,.04);
      user-select: none;
    }
    button:hover { filter: brightness(1.08); }
    button:active { transform: translateY(1px) scale(0.99); }

    .op { background: var(--btn-alt); }
    .accent { background: var(--accent); color: #062b18; }
    .danger { background: var(--danger); }
    .wide { grid-column: span 2; }

    .footer {
      text-align: center;
      color: var(--muted);
      font-size: 12px;
      margin-top: 12px;
    }
    .kbd { display:inline-block; padding:2px 6px; border:1px solid #334155; border-radius:6px; font-size:12px; color:#cbd5e1; background:#0b1220; }
  </style>
</head>
<body>
  <div class="wrap">
    <div class="calc" role="application" aria-label="Calculator">
      <div class="topbar">
        <div class="title">Calculator</div>
        <div class="credits">Keyboard: <span class="kbd">0–9</span> <span class="kbd">+ − × ÷</span> <span class="kbd">Enter</span> <span class="kbd">Backspace</span></div>
      </div>
      <div class="display" aria-live="polite">
        <div id="expr" class="expr"></div>
        <div id="result" class="result">0</div>
      </div>
      <div class="keys">
        <button class="danger" data-action="clear">AC</button>
        <button class="op" data-action="back">⌫</button>
        <button class="op" data-value="%">%</button>
        <button class="op" data-value="/">÷</button>

        <button data-value="7">7</button>
        <button data-value="8">8</button>
        <button data-value="9">9</button>
        <button class="op" data-value="*">×</button>

        <button data-value="4">4</button>
        <button data-value="5">5</button>
        <button data-value="6">6</button>
        <button class="op" data-value="-">−</button>

        <button data-value="1">1</button>
        <button data-value="2">2</button>
        <button data-value="3">3</button>
        <button class="op" data-value="+">+</button>

        <button class="wide" data-value="0">0</button>
        <button data-value=".">.</button>
        <button class="accent" data-action="equals">=</button>
      </div>
    </div>
   
  </div>

  <script>
    (function(){
      const exprEl = document.getElementById('expr');
      const resEl  = document.getElementById('result');

      let expr = '';
      let justEvaluated = false;

      function updateDisplay(){
        exprEl.textContent = expr || '';
      }

      function sanitizeExpression(raw){
        // Allow only digits, operators, parentheses, decimal point, and spaces
        const allowed = /[0-9+\-*/%.()\s]/g;
        const cleaned = (raw.match(allowed) || []).join('');
        return cleaned;
      }

      function evaluate(){
        if(!expr) return;
        try {
          const cleaned = sanitizeExpression(expr);
          // Prevent dangerous patterns (double operators at ends, etc.)
          if(!/^[0-9.(\s)]+([+\-*/%][0-9.(\s)]+)*$/.test(cleaned.replace(/\s+/g,''))){
            throw new Error('Malformed');
          }
          // Replace percentage: 50% => 0.5
          const pct = cleaned.replace(/(\d+(?:\.\d+)?)%/g, (_,n)=> String(parseFloat(n)/100));
          // Eval in strict isolated Function
          const result = Function('"use strict"; return (' + pct + ')')();
          if (typeof result !== 'number' || !isFinite(result)) throw new Error('Invalid');
          resEl.textContent = Number(result.toPrecision(12)).toString();
          justEvaluated = true;
        } catch(e){
          resEl.textContent = 'Error';
          justEvaluated = false;
        }
      }

      function input(val){
        if(justEvaluated && /[0-9.]/.test(val)){
          // start a new expression if a digit/dot after equals
          expr = '';
        }
        justEvaluated = false;
        expr += val;
        updateDisplay();
      }

      function back(){
        if (expr.length > 0) {
          expr = expr.slice(0,-1);
          updateDisplay();
        }
      }

      function clearAll(){
        expr = '';
        resEl.textContent = '0';
        justEvaluated = false;
        updateDisplay();
      }

      document.querySelector('.keys').addEventListener('click', (e)=>{
        const btn = e.target.closest('button');
        if(!btn) return;
        const val = btn.getAttribute('data-value');
        const action = btn.getAttribute('data-action');
        if(action === 'clear') return clearAll();
        if(action === 'back')  return back();
        if(action === 'equals') return evaluate();
        if(val) input(val);
      });

      // Keyboard support
      window.addEventListener('keydown', (e)=>{
        const k = e.key;
        if(/^[0-9]$/.test(k)) return input(k);
        if(k === '.') return input('.');
        if(k === '+') return input('+');
        if(k === '-') return input('-');
        if(k === '*' || k === 'x' || k === 'X') return input('*');
        if(k === '/' ) return input('/');
        if(k === '%') return input('%');
        if(k === '(' || k === ')') return input(k);
        if(k === 'Backspace') return back();
        if(k === 'Enter' || k === '=') return evaluate();
        if(k.toLowerCase() === 'c' && (e.ctrlKey || e.metaKey)) return clearAll();
      });

      updateDisplay();
    })();
  </script>
</body>
</html>
