<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover" />
<meta name="theme-color" content="#14171A" />
<meta name="apple-mobile-web-app-capable" content="yes" />
<meta name="description" content="A weekly shopping list with a running total and budget tracker." />
<title>Shopping list</title>
<link rel="icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 32 32'><text y='26' font-size='26'>&#128722;</text></svg >" />
<script src="https://unpkg.com/react@18/umd/react.production.min.js" crossorigin></script>
<script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js" crossorigin></script>
<script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
<style>
@import url('https://fonts.googleapis.com/css2?family=Archivo:wght@500;700;800&family=Roboto+Mono:wght@400;500;700&display=swap');
html, body { margin:0; padding:0; background:#F7F6F2; }
.sl { --ink:#14171A; --paper:#F7F6F2; --rule:#CBC9C0; --muted:#7A786E; --till:#0B5D3B; --over:#A82D18;
  background:var(--paper); color:var(--ink); font-family:'Roboto Mono',ui-monospace,monospace;
  min-height:100vh; max-width:620px; margin:0 auto; padding:20px 14px 140px; box-sizing:border-box; }
.sl * { box-sizing:border-box; }
.sl-h { font-family:'Archivo',system-ui,sans-serif; font-weight:800; letter-spacing:-0.02em; }
.sl-eyebrow { font-size:11px; letter-spacing:.18em; text-transform:uppercase; color:var(--muted); }
.sl-title { font-size:30px; line-height:1; margin:6px 0 2px; }
.sl-rule { border:0; border-top:1px dashed var(--rule); margin:16px 0; }
.sl-cat { font-family:'Archivo',sans-serif; font-weight:700; font-size:12px; letter-spacing:.16em;
  text-transform:uppercase; color:var(--muted); margin:22px 0 6px; display:flex; justify-content:space-between; }
.sl-row { display:flex; align-items:center; gap:10px; padding:9px 0; border-bottom:1px dotted var(--rule); }
.sl-box { width:24px; height:24px; min-width:24px; border:1.5px solid var(--ink); background:transparent;
  cursor:pointer; display:flex; align-items:center; justify-content:center; font-size:14px; padding:0; color:var(--paper); }
.sl-box[data-on="1"] { background:var(--till); border-color:var(--till); }
.sl-box:focus-visible, .sl-in:focus-visible, .sl-x:focus-visible, .sl-add:focus-visible { outline:2px solid var(--till); outline-offset:2px; }
.sl-name { flex:1; min-width:0; }
.sl-in { width:100%; border:0; background:transparent; font:inherit; color:inherit; padding:2px 0; outline:none; }
.sl-in:focus { box-shadow:0 1px 0 0 var(--till); }
.sl-note { font-size:11px; color:var(--muted); }
.sl-price { width:78px; text-align:right; font-weight:500; }
.sl-price input { text-align:right; }
.sl-row[data-done="1"] .sl-name, .sl-row[data-done="1"] .sl-price { opacity:.42; text-decoration:line-through; }
.sl-x { background:none; border:0; color:var(--muted); cursor:pointer; font-size:18px; padding:0 4px; line-height:1; }
.sl-x:hover { color:var(--over); }
.sl-add { background:none; border:1px dashed var(--rule); color:var(--muted); font:inherit; font-size:12px;
  padding:9px 10px; width:100%; cursor:pointer; margin-top:8px; }
.sl-add:hover { border-color:var(--till); color:var(--till); }
.sl-bar { position:fixed; left:0; right:0; bottom:0; background:var(--ink); color:var(--paper);
  padding:12px 16px calc(16px + env(safe-area-inset-bottom)); }
.sl-bar-inner { max-width:620px; margin:0 auto; }
.sl-bar-top { display:flex; justify-content:space-between; align-items:baseline; }
.sl-total { font-family:'Archivo',sans-serif; font-weight:800; font-size:30px; letter-spacing:-0.02em; }
.sl-sub { font-size:11px; opacity:.65; letter-spacing:.06em; }
.sl-track { height:6px; background:rgba(247,246,242,.18); margin-top:10px; overflow:hidden; }
.sl-fill { height:100%; background:#4ADE80; transition:width .3s ease; }
.sl-fill[data-over="1"] { background:#F87171; }
.sl-budget { background:transparent; border:0; border-bottom:1px solid rgba(247,246,242,.4); color:var(--paper);
  font:inherit; font-size:13px; width:62px; text-align:right; outline:none; }
.sl-reset { background:none; border:0; color:var(--muted); font:inherit; font-size:11px; text-decoration:underline;
  cursor:pointer; margin-top:24px; padding:0; }
.sl-warn { font-size:11px; color:var(--over); margin-top:14px; }
@media (prefers-reduced-motion: reduce) { .sl-fill { transition:none; } }
</style>
</head>
<body>
<div id="root"></div>
<script type="text/babel" data-presets="react">
const { useState, useEffect, useRef } = React;

const DEFAULT = [
  { id:"meat", name:"Meat & eggs", items:[
    { id:"m1", name:"Beef mince", note:"1.2kg", price:15, done:false },
    { id:"m2", name:"Chicken thigh fillets", note:"1kg", price:12, done:false },
    { id:"m3", name:"Rump steak", note:"~600g", price:13, done:false },
    { id:"m4", name:"Middle bacon", note:"1kg", price:14, done:false },
    { id:"m5", name:"Eggs", note:"2 dozen", price:20, done:false },
  ]},
  { id:"dairy", name:"Dairy & fats", items:[
    { id:"d1", name:"Ghee", note:"375g", price:9, done:false },
    { id:"d2", name:"Mainland Vintage", note:"500g", price:12, done:false },
    { id:"d3", name:"Sour cream", note:"300g", price:3.5, done:false },
    { id:"d4", name:"Milk", note:"", price:0, done:false },
  ]},
  { id:"produce", name:"Produce", items:[
    { id:"p1", name:"Avocados", note:"x5", price:9, done:false },
    { id:"p2", name:"Brown onions", note:"1kg", price:3, done:false },
    { id:"p3", name:"Tomatoes", note:"", price:4, done:false },
    { id:"p4", name:"Capsicum", note:"x2", price:3, done:false },
    { id:"p5", name:"Carrots", note:"1kg", price:2, done:false },
    { id:"p6", name:"Garlic", note:"", price:1, done:false },
    { id:"p7", name:"Iceberg or cabbage", note:"", price:3, done:false },
  ]},
  { id:"pantry", name:"Pantry & carbs", items:[
    { id:"c1", name:"Bread", note:"2 loaves", price:6, done:false },
    { id:"c2", name:"Tortillas", note:"", price:4, done:false },
    { id:"c3", name:"Packet ramen", note:"5-pack", price:4, done:false },
    { id:"c4", name:"Japanese curry roux", note:"S&B / Golden Curry", price:5, done:false },
    { id:"c5", name:"Massaman paste", note:"", price:3, done:false },
    { id:"c6", name:"Coconut milk", note:"x2", price:3, done:false },
  ]},
];

const KEY = "shopping-list-v1";
const uid = () => Math.random().toString(36).slice(2, 9);
const money = (n) => "$" + (Math.round(n * 100) / 100).toFixed(2);

function load() {
  try {
    const raw = window.localStorage.getItem(KEY);
    if (!raw) return null;
    return JSON.parse(raw);
  } catch (e) {
    return null;
  }
}

function ShoppingList() {
  const saved = useRef(load());
  const [cats, setCats] = useState(() => (saved.current && saved.current.cats) || DEFAULT);
  const [budget, setBudget] = useState(() => (saved.current && saved.current.budget != null ? saved.current.budget : 150));
  const [canSave, setCanSave] = useState(true);

  useEffect(() => {
    const t = setTimeout(() => {
      try {
        window.localStorage.setItem(KEY, JSON.stringify({ cats, budget }));
      } catch (e) {
        setCanSave(false);
      }
    }, 400);
    return () => clearTimeout(t);
  }, [cats, budget]);

  const edit = (ci, ii, patch) =>
    setCats((cs) => cs.map((c, i) => i !== ci ? c : { ...c, items: c.items.map((it, j) => (j === ii ? { ...it, ...patch } : it)) }));
  const remove = (ci, ii) =>
    setCats((cs) => cs.map((c, i) => (i !== ci ? c : { ...c, items: c.items.filter((_, j) => j !== ii) })));
  const add = (ci) =>
    setCats((cs) => cs.map((c, i) => (i !== ci ? c : { ...c, items: [...c.items, { id: uid(), name: "", note: "", price: 0, done: false }] })));

  const all = cats.flatMap((c) => c.items);
  const planned = all.reduce((s, i) => s + (Number(i.price) || 0), 0);
  const inTrolley = all.filter((i) => i.done).reduce((s, i) => s + (Number(i.price) || 0), 0);
  const left = budget - planned;
  const pct = Math.min(100, budget > 0 ? (planned / budget) * 100 : 0);
  const over = planned > budget;

  return (
    <div className="sl">
      <div className="sl-eyebrow">Week of · two adults</div>
      <h1 className="sl-h sl-title">Shopping list</h1>
      <div className="sl-note">Tap any name or price to change it. Tick things off as they go in the trolley.</div>
      <hr className="sl-rule" />

      {cats.map((c, ci) => {
        const sub = c.items.reduce((s, i) => s + (Number(i.price) || 0), 0);
        return (
          <section key={c.id}>
            <div className="sl-cat"><span>{c.name}</span><span>{money(sub)}</span></div>
            {c.items.map((it, ii) => (
              <div className="sl-row" key={it.id} data-done={it.done ? "1" : "0"}>
                <button className="sl-box" data-on={it.done ? "1" : "0"} aria-pressed={it.done}
                  aria-label={"Mark " + (it.name || "item") + " as in trolley"}
                  onClick={() => edit(ci, ii, { done: !it.done })}>{it.done ? "✓" : ""}</button>
                <div className="sl-name">
                  <input className="sl-in" value={it.name} placeholder="Item name"
                    onChange={(e) => edit(ci, ii, { name: e.target.value })} />
                  <input className="sl-in sl-note" value={it.note} placeholder="size / note"
                    onChange={(e) => edit(ci, ii, { note: e.target.value })} />
                </div>
                <div className="sl-price">
                  <input className="sl-in" type="number" inputMode="decimal" step="0.5" min="0" value={it.price}
                    onChange={(e) => edit(ci, ii, { price: e.target.value === "" ? 0 : parseFloat(e.target.value) })} />
                </div>
                <button className="sl-x" aria-label="Remove item" onClick={() => remove(ci, ii)}>×</button>
              </div>
            ))}
            <button className="sl-add" onClick={() => add(ci)}>+ Add to {c.name.toLowerCase()}</button>
          </section>
        );
      })}

      <div>
        <button className="sl-reset" onClick={() => { setCats(DEFAULT); setBudget(150); }}>
          Start again from the original list
        </button>
      </div>

      {!canSave && <div className="sl-warn">This browser is blocking storage, so changes won't be here next visit.</div>}

      <div className="sl-bar">
        <div className="sl-bar-inner">
          <div className="sl-bar-top">
            <div>
              <div className="sl-total">{money(planned)}</div>
              <div className="sl-sub">{money(inTrolley)} in trolley · {all.filter((i) => i.done).length}/{all.length} items</div>
            </div>
            <div style={{ textAlign: "right" }}>
              <div className="sl-sub">
                Budget $<input className="sl-budget" type="number" inputMode="decimal" value={budget} aria-label="Budget"
                  onChange={(e) => setBudget(e.target.value === "" ? 0 : parseFloat(e.target.value))} />
              </div>
              <div className="sl-sub" style={{ marginTop: 4, opacity: 1, color: over ? "#F87171" : "#4ADE80" }}>
                {over ? money(-left) + " over" : money(left) + " left"}
              </div>
            </div>
          </div>
          <div className="sl-track"><div className="sl-fill" data-over={over ? "1" : "0"} style={{ width: pct + "%" }} /></div>
        </div>
      </div>
    </div>
  );
}

ReactDOM.createRoot(document.getElementById("root")).render(<ShoppingList />);
</script>
</body>
</html>
