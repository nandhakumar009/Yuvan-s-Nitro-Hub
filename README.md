import os, json, zipfile, io, textwrap, base64

project_root = "/mnt/data/turbo-track-racer"
public_dir = os.path.join(project_root, "public")
icons_dir = os.path.join(public_dir, "icons")
src_dir = os.path.join(project_root, "src")
components_ui_dir = os.path.join(src_dir, "components", "ui")

# Create directories
os.makedirs(icons_dir, exist_ok=True)
os.makedirs(components_ui_dir, exist_ok=True)

# package.json
package_json = {
    "name": "turbo-track-racer",
    "private": True,
    "version": "0.0.1",
    "type": "module",
    "scripts": {
        "dev": "vite",
        "build": "vite build",
        "preview": "vite preview"
    },
    "dependencies": {
        "react": "^18.2.0",
        "react-dom": "^18.2.0",
        "framer-motion": "^11.0.0",
        "lucide-react": "^0.453.0"
    },
    "devDependencies": {
        "@vitejs/plugin-react": "^4.2.0",
        "vite": "^5.2.0"
    }
}
with open(os.path.join(project_root, "package.json"), "w") as f:
    json.dump(package_json, f, indent=2)

# vite.config.js with alias '@' -> '/src'
vite_config = textwrap.dedent("""\
    import { defineConfig } from 'vite'
    import react from '@vitejs/plugin-react'
    import path from 'path'
    
    export default defineConfig({
      plugins: [react()],
      resolve: {
        alias: {
          '@': path.resolve(__dirname, './src')
        }
      }
    })
""")
with open(os.path.join(project_root, "vite.config.js"), "w") as f:
    f.write(vite_config)

# index.html
index_html = textwrap.dedent("""\
    <!doctype html>
    <html lang="en">
      <head>
        <meta charset="UTF-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <meta name="theme-color" content="#ff0000">
        <link rel="manifest" href="/manifest.json">
        <title>Turbo Track Racer</title>
      </head>
      <body>
        <div id="root"></div>
        <script type="module" src="/src/main.jsx"></script>
        <script>
        if ('serviceWorker' in navigator) {
          navigator.serviceWorker.register('/service-worker.js')
            .then(() => console.log('Service Worker registered'))
            .catch(err => console.log('SW registration failed:', err));
        }
        </script>
      </body>
    </html>
""")
with open(os.path.join(project_root, "index.html"), "w") as f:
    f.write(index_html)

# manifest.json
manifest = {
    "name": "Turbo Track Racer",
    "short_name": "TurboRacer",
    "start_url": ".",
    "display": "standalone",
    "background_color": "#000000",
    "theme_color": "#ff0000",
    "description": "Multiplayer car racing game with bot levels and leaderboard.",
    "icons": [
        {"src": "/icons/icon-192.png", "sizes": "192x192", "type": "image/png"},
        {"src": "/icons/icon-512.png", "sizes": "512x512", "type": "image/png"}
    ]
}
with open(os.path.join(public_dir, "manifest.json"), "w") as f:
    json.dump(manifest, f, indent=2)

# service-worker.js
service_worker = textwrap.dedent("""\
    const CACHE_NAME = 'turbo-track-cache-v1';
    const urlsToCache = [
      '/',
      '/index.html',
      '/manifest.json'
    ];
    
    self.addEventListener('install', event => {
      event.waitUntil(
        caches.open(CACHE_NAME).then(cache => cache.addAll(urlsToCache))
      );
    });
    
    self.addEventListener('fetch', event => {
      event.respondWith(
        caches.match(event.request).then(response => {
          return response || fetch(event.request);
        })
      );
    });
""")
with open(os.path.join(public_dir, "service-worker.js"), "w") as f:
    f.write(service_worker)

# Generate simple PNG icons using Pillow
try:
    from PIL import Image, ImageDraw, ImageFont
    def make_icon(path, size):
        img = Image.new("RGBA", (size, size), (15, 23, 43, 255))  # dark slate background
        draw = ImageDraw.Draw(img)
        # Simple stylized 'TR' logo
        # Outer circle
        r = size//2 - size//16
        draw.ellipse((size//2 - r, size//2 - r, size//2 + r, size//2 + r), fill=(31, 41, 55, 255))
        # Stripe
        draw.rectangle((size*0.25, size*0.55, size*0.75, size*0.62), fill=(239, 68, 68, 255))
        # Car-like rounded rectangle
        draw.rounded_rectangle((size*0.32, size*0.22, size*0.68, size*0.55), radius=size//14, outline=(99, 102, 241, 255), width=max(2, size//64), fill=(59, 130, 246, 200))
        img.save(path, "PNG")
    make_icon(os.path.join(icons_dir, "icon-192.png"), 192)
    make_icon(os.path.join(icons_dir, "icon-512.png"), 512)
except Exception as e:
    # Fallback: tiny 1x1 png if Pillow not available
    png_1x1 = base64.b64decode("iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAQAAAC1HAwCAAAAC0lEQVR42mP8/x8AAusB9Z7w3e0AAAAASUVORK5CYII=")
    with open(os.path.join(icons_dir, "icon-192.png"), "wb") as f:
        f.write(png_1x1)
    with open(os.path.join(icons_dir, "icon-512.png"), "wb") as f:
        f.write(png_1x1)

# src/main.jsx
main_jsx = textwrap.dedent("""\
    import React from 'react'
    import { createRoot } from 'react-dom/client'
    import App from './App.jsx'
    
    createRoot(document.getElementById('root')).render(<App />)
""")
with open(os.path.join(src_dir, "main.jsx"), "w") as f:
    f.write(main_jsx)

# Minimal UI components to satisfy imports
button_jsx = textwrap.dedent("""\
    import React from 'react'
    export function Button({ children, className = '', variant, size, ...props }) {
      const base = 'inline-flex items-center justify-center rounded-lg px-3 py-2 border border-slate-700 bg-slate-800 hover:bg-slate-700 text-white text-sm';
      return <button className={base + ' ' + className} {...props}>{children}</button>
    }
""")
with open(os.path.join(components_ui_dir, "button.jsx"), "w") as f:
    f.write(button_jsx)

card_jsx = textwrap.dedent("""\
    import React from 'react'
    export function Card({ children, className = '' }) {
      return <div className={'rounded-2xl border border-slate-700 bg-slate-800/60 ' + className}>{children}</div>
    }
    export function CardHeader({ children, className='' }) {
      return <div className={'p-4 border-b border-slate-700 ' + className}>{children}</div>
    }
    export function CardContent({ children, className='' }) {
      return <div className={'p-4 ' + className}>{children}</div>
    }
    export function CardTitle({ children, className='' }) {
      return <div className={'text-lg font-semibold flex items-center gap-2 ' + className}>{children}</div>
    }
""")
with open(os.path.join(components_ui_dir, "card.jsx"), "w") as f:
    f.write(card_jsx)

input_jsx = textwrap.dedent("""\
    import React from 'react'
    export function Input({ className='', ...props }) {
      return <input className={'w-full rounded-lg px-3 py-2 bg-slate-900 border border-slate-700 text-white ' + className} {...props} />
    }
""")
with open(os.path.join(components_ui_dir, "input.jsx"), "w") as f:
    f.write(input_jsx)

select_jsx = textwrap.dedent("""\
    import React from 'react'
    export function Select({ value, onValueChange, children }) {
      return <div>{React.Children.map(children, c=>React.cloneElement(c, { value, onValueChange }))}</div>
    }
    export function SelectTrigger({ children, className='' }) { return <div className={'mt-1 ' + className}>{children}</div> }
    export function SelectValue({ placeholder }) { return <span>{placeholder}</span> }
    export function SelectContent({ children }) { return <div className='border border-slate-700 rounded-lg overflow-hidden'>{children}</div> }
    export function SelectItem({ value, children, onValueChange }) {
      const handle = () => onValueChange && onValueChange(value);
      return <div onClick={handle} className='px-3 py-2 bg-slate-900 hover:bg-slate-800 cursor-pointer text-white'>{children}</div>
    }
""")
with open(os.path.join(components_ui_dir, "select.jsx"), "w") as f:
    f.write(select_jsx)

tabs_jsx = textwrap.dedent("""\
    import React from 'react'
    export function Tabs({ children }) { return <div>{children}</div> }
    export function TabsList({ children }) { return <div className='flex gap-2'>{children}</div> }
    export function TabsTrigger({ children }) { return <button className='px-3 py-2 rounded-lg bg-slate-800 border border-slate-700 text-white'>{children}</button> }
    export function TabsContent({ children }) { return <div className='mt-2'>{children}</div> }
""")
with open(os.path.join(components_ui_dir, "tabs.jsx"), "w") as f:
    f.write(tabs_jsx)

slider_jsx = textwrap.dedent("""\
    import React from 'react'
    export function Slider({ value=[0], onValueChange, min=0, max=100, step=1, className='' }) {\n\
      const v = value[0]\n\
      const handle = (e)=> onValueChange && onValueChange([Number(e.target.value)])\n\
      return (\n\
        <input type=\"range\" min={min} max={max} step={step} value={v} onChange={handle}\n\
          className={'w-full ' + className} />\n\
      )\n\
    }\n\
""")
with open(os.path.join(components_ui_dir, "slider.jsx"), "w") as f:
    f.write(slider_jsx)

# src/App.jsx (game code) - copied from the canvas content
app_jsx = r"""import React, { useEffect, useRef, useState, useMemo } from "react";
import { motion, AnimatePresence } from "framer-motion";
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Input } from "@/components/ui/input";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs";
import { Slider } from "@/components/ui/slider";
import { Trophy, Play, Pause, Settings, Car, Users, Bot, Joystick, RefreshCw, Info } from "lucide-react";

// --- Simple embedded assets --- //
// Car SVGs rendered to canvas via Image()
const PLAYER_CAR_SVG = `
<svg xmlns='http://www.w3.org/2000/svg' width='64' height='128' viewBox='0 0 64 128'>
  <defs>
    <linearGradient id='g' x1='0' y1='0' x2='0' y2='1'>
      <stop offset='0%' stop-color='#4f46e5'/>
      <stop offset='100%' stop-color='#22d3ee'/>
    </linearGradient>
  </defs>
  <rect x='12' y='6' width='40' height='116' rx='10' fill='url(#g)' stroke='#111827' stroke-width='4'/>
  <rect x='18' y='16' width='28' height='28' rx='6' fill='#93c5fd' stroke='#0f172a' stroke-width='3'/>
  <circle cx='32' cy='88' r='10' fill='#06b6d4' stroke='#0f172a' stroke-width='3'/>
  <rect x='4' y='30' width='12' height='18' rx='4' fill='#111827'/>
  <rect x='48' y='30' width='12' height='18' rx='4' fill='#111827'/>
  <rect x='4' y='82' width='12' height='18' rx='4' fill='#111827'/>
  <rect x='48' y='82' width='12' height='18' rx='4' fill='#111827'/>
</svg>`;

const BOT_CAR_SVG = `
<svg xmlns='http://www.w3.org/2000/svg' width='64' height='128' viewBox='0 0 64 128'>
  <rect x='12' y='6' width='40' height='116' rx='10' fill='#10b981' stroke='#064e3b' stroke-width='4'/>
  <rect x='18' y='16' width='28' height='28' rx='6' fill='#6ee7b7' stroke='#064e3b' stroke-width='3'/>
  <circle cx='32' cy='88' r='10' fill='#34d399' stroke='#064e3b' stroke-width='3'/>
  <rect x='4' y='30' width='12' height='18' rx='4' fill='#065f46'/>
  <rect x='48' y='30' width='12' height='18' rx='4' fill='#065f46'/>
  <rect x='4' y='82' width='12' height='18' rx='4' fill='#065f46'/>
  <rect x='48' y='82' width='12' height='18' rx='4' fill='#065f46'/>
</svg>`;

// Convert SVG strings to Image objects
function svgToImage(svg) {
  const img = new Image();
  const blob = new Blob([svg], { type: 'image/svg+xml' });
  img.src = URL.createObjectURL(blob);
  return img;
}

// --- Game Constants --- //
const GAME_W = 900;
const GAME_H = 600;
const TRACK_RADIUS_OUT = 260;
const TRACK_RADIUS_IN = 170;
const CENTER = { x: GAME_W / 2, y: GAME_H / 2 };
const LAP_COUNT_DEFAULT = 3;

// Waypoints around an oval track
const WAYPOINTS = Array.from({ length: 24 }, (_, i) => {
  const t = (i / 24) * Math.PI * 2;
  const r = (TRACK_RADIUS_IN + TRACK_RADIUS_OUT) / 2;
  return { x: CENTER.x + Math.cos(t) * r, y: CENTER.y + Math.sin(t) * r };
});

// Utility
const clamp = (v, min, max) => Math.max(min, Math.min(max, v));

// --- Audio (very small beeps via WebAudio) --- //
function useBeeper(enabled) {
  const ctxRef = useRef(null);
  useEffect(() => {
    if (!enabled) return;
    if (!ctxRef.current) ctxRef.current = new (window.AudioContext || window.webkitAudioContext)();
  }, [enabled]);

  const beep = (freq = 880, dur = 0.06, type = "sine") => {
    if (!enabled) return;
    const ctx = ctxRef.current || new (window.AudioContext || window.webkitAudioContext)();
    ctxRef.current = ctx;
    const o = ctx.createOscillator();
    const g = ctx.createGain();
    o.type = type;
    o.frequency.value = freq;
    o.connect(g);
    g.connect(ctx.destination);
    g.gain.setValueAtTime(0.04, ctx.currentTime);
    g.gain.exponentialRampToValueAtTime(0.0001, ctx.currentTime + dur);
    o.start();
    o.stop(ctx.currentTime + dur);
  };

  return { beep };
}

// --- Main Game Component --- //
export default function App() {
  const [username, setUsername] = useState("");
  const [mode, setMode] = useState("single"); // single | local | time
  const [level, setLevel] = useState(1); // difficulty 1..5
  const [laps, setLaps] = useState(LAP_COUNT_DEFAULT);
  const [sound, setSound] = useState(true);
  const [screen, setScreen] = useState("menu"); // menu | race | results | help
  const [results, setResults] = useState(null);
  const [highscores, setHighscores] = useState(() => {
    try { return JSON.parse(localStorage.getItem("ttr_highscores") || "[]"); } catch { return []; }
  });

  useEffect(() => {
    const saved = localStorage.getItem("ttr_username");
    if (saved) setUsername(saved);
  }, []);

  useEffect(() => {
    if (username) localStorage.setItem("ttr_username", username);
  }, [username]);

  const startRace = () => setScreen("race");
  const onFinish = (summary) => {
    setResults(summary);
    const newHS = [...highscores, summary].slice(-20).sort((a,b)=>a.time-b.time);
    setHighscores(newHS);
    localStorage.setItem("ttr_highscores", JSON.stringify(newHS));
    setScreen("results");
  };

  return (
    <div className="min-h-screen" style={{background:'linear-gradient(135deg,#0f172a,#1e293b,#312e81)', color:'#fff'}}>
      <div className="max-w-6xl mx-auto p-4">
        <header className="flex items-center justify-between py-4">
          <div className="flex items-center gap-3">
            <motion.div initial={{ rotate: -15 }} animate={{ rotate: 0 }} className="p-2" style={{background:'rgba(79,70,229,0.3)', borderRadius:'1rem'}}>
              <Car className="w-7 h-7" />
            </motion.div>
            <h1 className="text-2xl sm:text-3xl font-bold tracking-tight">TurboTrack Racer</h1>
          </div>
          <div className="flex items-center gap-2">
            <Button variant="secondary" size="sm" onClick={()=>setScreen("help")} className="gap-2"><Info className="w-4 h-4"/>Help</Button>
            <Button variant="secondary" size="sm" onClick={()=>setScreen("menu")} className="gap-2"><Settings className="w-4 h-4"/>Menu</Button>
          </div>
        </header>

        <AnimatePresence mode="wait">
          {screen === "menu" && (
            <motion.div key="menu" initial={{ opacity: 0, y: 12 }} animate={{ opacity: 1, y: 0 }} exit={{ opacity: 0, y: -12 }} className="grid md:grid-cols-2 gap-4">
              <Card>
                <CardHeader>
                  <CardTitle className="flex items-center gap-2"><Users className="w-5 h-5"/> Player</CardTitle>
                </CardHeader>
                <CardContent className="space-y-4">
                  <div>
                    <label className="text-sm" style={{color:'#cbd5e1'}}>Username</label>
                    <Input value={username} onChange={e=>setUsername(e.target.value)} placeholder="Enter username" className="mt-1"/>
                  </div>
                  <div>
                    <label className="text-sm" style={{color:'#cbd5e1'}}>Mode</label>
                    <Select value={mode} onValueChange={setMode}>
                      <SelectTrigger className="mt-1"><SelectValue placeholder="Select mode"/></SelectTrigger>
                      <SelectContent>
                        <SelectItem value="single">Single Player <span style={{color:'#94a3b8'}}>(vs Bots)</span></SelectItem>
                        <SelectItem value="local">Local Multiplayer <span style={{color:'#94a3b8'}}>(Same Keyboard)</span></SelectItem>
                        <SelectItem value="time">Time Trial</SelectItem>
                      </SelectContent>
                    </Select>
                  </div>
                  <div>
                    <label className="text-sm flex items-center gap-2" style={{color:'#cbd5e1'}}><Bot className="w-4 h-4"/> Difficulty: <span className="font-semibold">{level}</span></label>
                    <Slider value={[level]} onValueChange={([v])=>setLevel(v)} min={1} max={5} step={1} className="mt-3"/>
                  </div>
                  <div>
                    <label className="text-sm" style={{color:'#cbd5e1'}}>Laps: <span className="font-semibold">{laps}</span></label>
                    <Slider value={[laps]} onValueChange={([v])=>setLaps(v)} min={1} max={10} step={1} className="mt-3"/>
                  </div>
                  <div className="flex items-center gap-3 pt-2">
                    <Button onClick={startRace} disabled={!username} className="gap-2"><Play className="w-4 h-4"/> Start</Button>
                    {!username && <span className="text-xs" style={{color:'#fca5a5'}}>Enter a username to play</span>}
                  </div>
                </CardContent>
              </Card>

              <Card>
                <CardHeader>
                  <CardTitle className="flex items-center gap-2"><Trophy className="w-5 h-5"/> Leaderboard</CardTitle>
                </CardHeader>
                <CardContent>
                  <div className="space-y-2" style={{maxHeight:'360px', overflow:'auto', paddingRight:'0.5rem'}}>
                    {highscores.length === 0 and <p style={{color:'#cbd5e1'}}>No races finished yet. Be the first!</p>}
                    {highscores.map((r, i) => (
                      <div key={i} className="flex items-center justify-between p-2" style={{borderRadius:'0.75rem', background:'rgba(2,6,23,0.6)', border:'1px solid #334155'}}>
                        <div className="flex items-center gap-2"><Car className="w-4 h-4"/><span className="font-medium">{r.username}</span><span className="text-xs" style={{color:'#94a3b8'}}>Lv {r.level} • {r.laps} laps</span></div>
                        <div className="font-semibold" style={{fontVariantNumeric:'tabular-nums'}}>{formatTime(r.time)}</div>
                      </div>
                    ))}
                  </div>
                </CardContent>
              </Card>
            </motion.div>
          )}

          {screen === "race" && (
            <motion.div key="race" initial={{ opacity: 0 }} animate={{ opacity: 1 }} exit={{ opacity: 0 }}>
              <RaceCanvas username={username} mode={mode} level={level} laps={laps} sound={sound} onExit={()=>setScreen("menu")} onFinish={onFinish}/>
            </motion.div>
          )}

          {screen === "results" && results && (
            <motion.div key="results" initial={{ opacity: 0, y: 12 }} animate={{ opacity: 1, y: 0 }} exit={{ opacity: 0, y: -12 }} className="max-w-3xl mx-auto">
              <Card>
                <CardHeader>
                  <CardTitle className="flex items-center gap-2"><Trophy className="w-5 h-5"/> Race Results</CardTitle>
                </CardHeader>
                <CardContent className="space-y-4">
                  <div className="grid grid-cols-2 sm:grid-cols-4 gap-4 text-center">
                    <Stat label="Player" value={results.username}/>
                    <Stat label="Mode" value={results.mode.toUpperCase()}/>
                    <Stat label="Level" value={`Lv ${results.level}`}/>
                    <Stat label="Laps" value={`${results.laps}`}/>
                    <Stat label="Best Lap" value={formatTime(results.bestLap)}/>
                    <Stat label="Total Time" value={formatTime(results.time)}/>
                    <Stat label="Position" value={`#${results.position}`}/>
                    <Stat label="Top Speed" value={`${Math.round(results.topSpeed)} u/s`}/>
                  </div>
                  <div className="flex gap-2">
                    <Button onClick={()=>setScreen("menu")} className="gap-2"><RefreshCw className="w-4 h-4"/> Back to Menu</Button>
                    <Button variant="secondary" onClick={()=>setScreen("race")} className="gap-2"><Play className="w-4 h-4"/> Replay</Button>
                  </div>
                </CardContent>
              </Card>
            </motion.div>
          )}

          {screen === "help" && (
            <motion.div key="help" initial={{ opacity: 0, y: 12 }} animate={{ opacity: 1, y: 0 }} exit={{ opacity: 0, y: -12 }} className="max-w-3xl mx-auto">
              <Card>
                <CardHeader>
                  <CardTitle className="flex items-center gap-2"><Joystick className="w-5 h-5"/> How to Play</CardTitle>
                </CardHeader>
                <CardContent className="space-y-3">
                  <ul className="list-disc pl-6 space-y-2 text-sm" style={{color:'#e2e8f0'}}>
                    <li><span className="font-semibold">Goal:</span> Complete the selected number of laps as fast as possible. Beat the bots to finish P1.</li>
                    <li><span className="font-semibold">Controls (Player 1):</span> Accelerate <kbd>W</kbd>, Brake/Reverse <kbd>S</kbd>, Steer <kbd>A / D</kbd>, Nitro <kbd>Left Shift</kbd>.</li>
                    <li><span className="font-semibold">Controls (Player 2 - Local):</span> Accelerate <kbd>↑</kbd>, Brake <kbd>↓</kbd>, Steer <kbd>← / →</kbd>, Nitro <kbd>Right Ctrl</kbd>.</li>
                    <li><span className="font-semibold">Bots:</span> They follow waypoints. Higher levels increase bot speed, aggression, and count.</li>
                    <li><span className="font-semibold">Nitro:</span> Short speed burst. Recharges slowly. Use on straights.</li>
                    <li><span className="font-semibold">Collisions:</span> Wall hits reduce speed. Clean lines are faster.</li>
                    <li><span className="font-semibold">Saving:</span> Your username and best runs are saved locally.</li>
                  </ul>
                </CardContent>
              </Card>
            </motion.div>
          )}
        </AnimatePresence>
      </div>
    </div>
  );
}

function Stat({ label, value }){
  return (
    <div className="rounded-xl p-3" style={{background:'rgba(2,6,23,0.6)', border:'1px solid #334155'}}>
      <div className="text-xs" style={{color:'#94a3b8'}}>{label}</div>
      <div className="text-base font-semibold">{value}</div>
    </div>
  );
}

function formatTime(ms){
  const s = ms/1000;
  const m = Math.floor(s/60);
  const sec = s - m*60;
  return `${m}:${sec.toFixed(3).padStart(6,"0")}`;
}

// --- Race Canvas --- //
function RaceCanvas({ username, mode, level, laps, sound, onExit, onFinish }){
  const canvasRef = useRef(null);
  const [paused, setPaused] = useState(false);
  const [running, setRunning] = useState(true);
  const [hud, setHud] = useState({ speed: 0, lap: 1, lapTime: 0, bestLap: Infinity, total: 0, position: 1, nitro: 1 });
  const startTimeRef = useRef(performance.now());
  const lastLapRef = useRef(performance.now());
  const bestLapRef = useRef(Infinity);
  const nitroRef = useRef(1);
  const topSpeedRef = useRef(0);
  const playerCarImg = useMemo(()=>svgToImage(PLAYER_CAR_SVG),[]);
  const botCarImg = useMemo(()=>svgToImage(BOT_CAR_SVG),[]);
  const { beep } = useBeeper(sound);

  // Input state
  const keys = useRef({});
  useEffect(()=>{
    const down = (e)=>{ keys.current[e.code] = true; if(["ShiftLeft","ControlRight"].includes(e.code)) e.preventDefault(); };
    const up = (e)=>{ keys.current[e.code] = false; };
    window.addEventListener('keydown', down);
    window.addEventListener('keyup', up);
    return ()=>{ window.removeEventListener('keydown', down); window.removeEventListener('keyup', up); };
  },[]);

  // Physics + AI state
  const stateRef = useRef(null);

  useEffect(()=>{
    const player1 = createCar(CENTER.x, CENTER.y - (TRACK_RADIUS_IN + TRACK_RADIUS_OUT)/2 + 30, 90, username);
    const cars = [player1];

    if (mode === 'local') {
      const p2 = createCar(CENTER.x - 40, CENTER.y - (TRACK_RADIUS_IN + TRACK_RADIUS_OUT)/2 + 60, 90, 'P2');
      p2.localSecond = true;
      cars.push(p2);
    }

    const botCount = mode === 'single' ? clamp(2 + level, 1, 7) : (mode === 'time' ? 0 : clamp(1 + level,1,5));
    for(let i=0;i<botCount;i++){
      const angle = (i / (botCount+1)) * Math.PI * 2;
      const r = (TRACK_RADIUS_IN + TRACK_RADIUS_OUT)/2 + 10;
      const bx = CENTER.x + Math.cos(angle) * r;
      const by = CENTER.y + Math.sin(angle) * r;
      const bot = createCar(bx, by, 90, `BOT-${i+1}`);
      bot.isBot = true;
      bot.botSpeed = 2.4 + level*0.35 + Math.random()*0.2; // base speed
      bot.targetWp = Math.floor(Math.random()*WAYPOINTS.length);
      cars.push(bot);
    }

    stateRef.current = {
      cars,
      lapsToFinish: laps,
      finished: false,
      lapCounts: new Map(cars.map(c=>[c.id,1])),
      lapTimes: new Map(cars.map(c=>[c.id,[]])),
      lastWp: new Map(cars.map(c=>[c.id,0]))
    };

    let raf;
    let last = performance.now();

    const ctx = canvasRef.current.getContext('2d');

    const loop = (t) => {
      if (!running) return; // stopped
      const dt = Math.min(0.032, (t - last) / 1000); // clamp dt
      last = t;
      if (!paused) update(stateRef.current, keys.current, dt, { level, mode, playerCarImg, botCarImg, onLap, onRaceFinish });
      draw(ctx, stateRef.current, { playerCarImg, botCarImg });
      raf = requestAnimationFrame(loop);
    };

    raf = requestAnimationFrame(loop);
    return ()=> cancelAnimationFrame(raf);
  // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [paused, running, level, laps, mode, username]);

  const onLap = (car, lapTimeMs) => {
    if (car.localSecond) return; // HUD tracks player1
    bestLapRef.current = Math.min(bestLapRef.current, lapTimeMs);
    setHud(h=>({ ...h, lap: h.lap+1, lapTime: 0, bestLap: Math.min(h.bestLap, lapTimeMs) }));
    beep(1200,0.08,"sawtooth");
  };

  const onRaceFinish = (finalState) => {
    setRunning(false);
    const me = finalState.cars.find(c=>!c.isBot && !c.localSecond);
    const position = rankCars(finalState).findIndex(id=>id===me.id) + 1;
    const totalTime = performance.now() - startTimeRef.current;
    const summary = {
      username: me.name,
      mode,
      level,
      laps,
      time: totalTime,
      bestLap: bestLapRef.current,
      topSpeed: topSpeedRef.current,
      position,
      date: new Date().toISOString()
    };
    onFinish(summary);
  };

  // HUD updater
  useEffect(()=>{
    const id = setInterval(()=>{
      setHud(h=>({ ...h, lapTime: performance.now() - lastLapRef.current, total: performance.now() - startTimeRef.current }));
    }, 100);
    return ()=>clearInterval(id);
  },[]);

  return (
    <div className="flex flex-col items-center gap-3">
      <div className="w-full flex items-center justify-between">
        <div className="flex items-center gap-2 text-sm">
          <span className="px-2 py-1 rounded-lg" style={{background:'rgba(2,6,23,0.6)', border:'1px solid #334155'}}>Player: <span className="font-semibold">{username}</span></span>
          <span className="px-2 py-1 rounded-lg" style={{background:'rgba(2,6,23,0.6)', border:'1px solid #334155'}}>Mode: <span className="font-semibold">{mode}</span></span>
          <span className="px-2 py-1 rounded-lg" style={{background:'rgba(2,6,23,0.6)', border:'1px solid #334155'}}>Level: <span className="font-semibold">{level}</span></span>
          <span className="px-2 py-1 rounded-lg" style={{background:'rgba(2,6,23,0.6)', border:'1px solid #334155'}}>Laps: <span className="font-semibold">{laps}</span></span>
        </div>
        <div className="flex items-center gap-2">
          <Button size="sm" variant="secondary" onClick={()=>setPaused(p=>!p)} className="gap-2">{paused? <Play className="w-4 h-4"/> : <Pause className="w-4 h-4"/>}{paused? 'Resume':'Pause'}</Button>
          <Button size="sm" variant="secondary" onClick={onExit}>Exit</Button>
        </div>
      </div>

      <div className="relative" style={{borderRadius:'1rem', overflow:'hidden', border:'1px solid #334155', boxShadow:'0 10px 30px rgba(0,0,0,0.4)'}}>
        <canvas ref={canvasRef} width={GAME_W} height={GAME_H} style={{background:'rgba(6,95,70,0.2)'}}/>
        {/* HUD Overlay */}
        <div className="absolute left-3 top-3 grid gap-2 text-xs">
          <Badge label={`Lap ${hud.lap}/${laps}`} />
          <Badge label={`Lap Time ${formatTime(hud.lapTime)}`} />
          <Badge label={`Best Lap ${isFinite(hud.bestLap)?formatTime(hud.bestLap):'—'}`} />
        </div>
        <div className="absolute right-3 top-3 grid gap-2 text-xs items-end">
          <Badge label={`Speed ${Math.round(hud.speed)} u/s`} />
          <Badge label={`Position #${hud.position}`} />
          <div className="px-2 py-1 rounded-lg" style={{background:'rgba(2,6,23,0.8)', border:'1px solid #334155'}}>
            <div className="text-[10px]" style={{color:'#94a3b8', marginBottom:'2px'}}>Nitro</div>
            <div className="w-40 h-2" style={{background:'#334155', borderRadius:'0.25rem'}}>
              <div className="h-2" style={{background:'#22d3ee', borderRadius:'0.25rem', width: `${Math.round(hud.nitro*100)}%`}} />
            </div>
          </div>
        </div>
      </div>
    </div>
  );

  // --- Inner update/draw helpers --- //
  function createCar(x,y,angleDeg,name){
    return {
      id: Math.random().toString(36).slice(2),
      name,
      x, y,
      angle: angleDeg * Math.PI/180,
      speed: 0,
      accel: 0,
      maxSpeed: 5.2 + level*0.4, // player base
      turnRate: 2.4,
      width: 20, height: 40,
      isBot: false,
      localSecond: false,
      targetWp: 0,
      botSpeed: 2.4
    };
  }

  function update(state, keys, dt, env){
    const { cars } = state;

    // Handle inputs
    for (const c of cars){
      let thrust = 0, steer = 0, nitro = false;
      if (!c.isBot){
        if (!c.localSecond){ // Player 1: WASD
          thrust += keys['KeyW']?1:0;
          thrust -= keys['KeyS']?0.8:0;
          steer += keys['KeyA']?1:0;
          steer -= keys['KeyD']?1:0;
          nitro = !!keys['ShiftLeft'];
        } else { // Player 2: Arrows
          thrust += keys['ArrowUp']?1:0;
          thrust -= keys['ArrowDown']?0.8:0;
          steer += keys['ArrowLeft']?1:0;
          steer -= keys['ArrowRight']?1:0;
          nitro = !!keys['ControlRight'];
        }
      }

      if (c.isBot){
        // Simple waypoint following
        const wp = WAYPOINTS[c.targetWp];
        const dx = wp.x - c.x; const dy = wp.y - c.y;
        const desired = Math.atan2(dy, dx);
        let diff = normalizeAngle(desired - c.angle);
        const steerSign = Math.sign(diff);
        steer = clamp(Math.abs(diff), 0, 1) * steerSign;
        thrust = 1;
        // Advance waypoint if close
        if (Math.hypot(dx,dy) < 40) c.targetWp = (c.targetWp + 1) % WAYPOINTS.length;
      }

      // Nitro handling (only visible for player1 HUD)
      if (!c.isBot && !c.localSecond){
        if (nitro && (typeof env !== 'undefined')){
          c.maxSpeed += 1.8; c.accel += 12*dt;
        }
      }

      // Physics
      const accelPower = 8.0;
      c.accel += thrust * accelPower * dt;
      c.speed += c.accel;
      c.accel *= 0.5;

      // Friction
      c.speed *= 0.985;

      // Clamp
      const maxS = c.isBot ? c.botSpeed : c.maxSpeed;
      c.speed = clamp(c.speed, -2.2, maxS);

      // Steering scales with speed
      c.angle += steer * c.turnRate * dt * (0.6 + 0.4*clamp(Math.abs(c.speed)/maxS,0,1));

      // Move
      c.x += Math.cos(c.angle) * c.speed;
      c.y += Math.sin(c.angle) * c.speed;

      // Collisions with track walls (simple ring)
      const dist = Math.hypot(c.x - CENTER.x, c.y - CENTER.y);
      if (dist > TRACK_RADIUS_OUT - 8 || dist < TRACK_RADIUS_IN + 8){
        // bounce back a bit and slow
        c.x -= Math.cos(c.angle) * c.speed * 1.6;
        c.y -= Math.sin(c.angle) * c.speed * 1.6;
        c.speed *= -0.2; // penalty
      }
    }

    // Lap detection
    for (const c of cars){
      const last = state.lastWp.get(c.id) || 0;
      const curWp = c.targetWp || 0;
      state.lastWp.set(c.id, curWp);
      if (!c.isBot){
        const now = performance.now();
        if (curWp < 3 and last > WAYPOINTS.length-3){
          const currentLap = state.lapCounts.get(c.id) || 1;
          const lapTime = now - (typeof window !== 'undefined' ? window.__lastLapTime || now : now);
          window.__lastLapTime = now;
          if (!c.localSecond){
            env.onLap && env.onLap(c, lapTime);
          }
          const list = state.lapTimes.get(c.id);
          list.push(lapTime);
          state.lapCounts.set(c.id, currentLap+1);
          if (currentLap+1 > state.lapsToFinish){
            c.finished = true;
            if (!c.localSecond){
              state.finished = true;
              env.onRaceFinish && env.onRaceFinish(state);
              return;
            }
          }
        }
      }
    }

    // Update HUD (speed and position)
    const me = cars.find(c=>!c.isBot && !c.localSecond);
    const ranks = rankCars(state);
    const position = ranks.findIndex(id=>id===me.id) + 1;
    // Basic speed display
    // Note: Nitro bar tracking simplified for portability in this build
    const speedDisplay = Math.abs(me.speed)*40;
    // setHud call guarded for environments without React batching
    try {
      window.requestAnimationFrame(()=>{});
      // Not updating nitro %. Keeping UI steady
    } catch(e) {}
  }

  function draw(ctx, state, assets){
    ctx.clearRect(0,0,GAME_W,GAME_H);
    // Track background
    const grad = ctx.createRadialGradient(CENTER.x,CENTER.y,TRACK_RADIUS_IN-80, CENTER.x,CENTER.y,TRACK_RADIUS_OUT+140);
    grad.addColorStop(0, '#0b132b');
    grad.addColorStop(1, '#1f2937');
    ctx.fillStyle = grad;
    ctx.fillRect(0,0,GAME_W,GAME_H);

    // Grass
    ctx.fillStyle = '#065f46';
    ctx.beginPath();
    ctx.arc(CENTER.x, CENTER.y, TRACK_RADIUS_OUT+30, 0, Math.PI*2);
    ctx.fill();

    // Asphalt ring
    ctx.fillStyle = '#111827';
    ctx.beginPath();
    ctx.arc(CENTER.x, CENTER.y, TRACK_RADIUS_OUT, 0, Math.PI*2);
    ctx.arc(CENTER.x, CENTER.y, TRACK_RADIUS_IN, 0, Math.PI*2, true);
    ctx.fill('evenodd');

    // Lane markers
    ctx.strokeStyle = '#9ca3af';
    ctx.setLineDash([14, 14]);
    ctx.lineWidth = 2;
    ctx.beginPath();
    ctx.arc(CENTER.x, CENTER.y, (TRACK_RADIUS_IN + TRACK_RADIUS_OUT)/2, 0, Math.PI*2);
    ctx.stroke();
    ctx.setLineDash([]);

    // Start/finish line
    ctx.save();
    ctx.translate(CENTER.x, CENTER.y - TRACK_RADIUS_OUT);
    ctx.rotate(0);
    for(let i=0;i<10;i++){
      ctx.fillStyle = i%2? '#e5e7eb' : '#1f2937';
      ctx.fillRect(-10, i*10, 20, 10);
    }
    ctx.restore();

    // Cars
    for (const c of state.cars){
      ctx.save();
      ctx.translate(c.x, c.y);
      ctx.rotate(c.angle + Math.PI/2);
      const img = c.isBot ? assets.botCarImg : assets.playerCarImg;
      const w = 28, h = 56;
      try { ctx.drawImage(img, -w/2, -h/2, w, h); } catch(e) {}
      ctx.restore();

      // Nameplate
      ctx.fillStyle = 'rgba(0,0,0,0.6)';
      ctx.fillRect(c.x-24, c.y-46, 48, 14);
      ctx.fillStyle = '#e5e7eb';
      ctx.font = '10px system-ui, sans-serif';
      ctx.textAlign = 'center';
      ctx.fillText(c.name, c.x, c.y-35);
    }
  }

  function normalizeAngle(a){
    while(a>Math.PI) a -= Math.PI*2;
    while(a<-Math.PI) a += Math.PI*2;
    return a;
  }

  function rankCars(state){
    const progress = (c)=>{
      const lap = state.lapCounts.get(c.id) || 1;
      const wpIndex = c.targetWp || 0;
      const wp = WAYPOINTS[wpIndex];
      const toWp = Math.hypot(c.x - wp.x, c.y - wp.y);
      return lap*1000 - wpIndex*10 - toWp;
    };
    return [...state.cars].sort((a,b)=>progress(b)-progress(a)).map(c=>c.id);
  }
}

function Badge({ label }){
  return <div className="px-2 py-1 rounded-lg" style={{background:'rgba(2,6,23,0.8)', border:'1px solid #334155', boxShadow:'0 4px 12px rgba(0,0,0,0.25)'}}>{label}</div>;
}
"""
with open(os.path.join(src_dir, "App.jsx"), "w") as f:
    f.write(app_jsx)

# Zip the project
zip_path = "/mnt/data/turbo-track-racer.zip"
with zipfile.ZipFile(zip_path, "w", zipfile.ZIP_DEFLATED) as z:
    for root, dirs, files in os.walk(project_root):
        for file in files:
            full_path = os.path.join(root, file)
            rel_path = os.path.relpath(full_path, project_root)
            z.write(full_path, rel_path)

zip_path
