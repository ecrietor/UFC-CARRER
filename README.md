import React, { useState, useEffect, useMemo, useRef } from "react";
import {
  Star, Swords, Trophy, Flame, Shield, Brain, Zap, Activity,
  ChevronRight, Award, Skull, RotateCcw, Users, Crown, Mic, VenetianMask,
  CircleDot, Lock, CheckCircle2, ArrowUpCircle, TrendingUp,
} from "lucide-react";
import {
  RadarChart, PolarGrid, PolarAngleAxis, Radar, ResponsiveContainer,
} from "recharts";
 
/* ============================== DESIGN TOKENS ============================== */
const C = {
  bg: "#0A0C10",
  bgGrad: "radial-gradient(1200px 600px at 50% -10%, #171B22 0%, #0A0C10 60%)",
  surface: "#14171D",
  surfaceRaised: "#1B1F27",
  border: "#272C35",
  borderSoft: "#1E222A",
  textPrimary: "#EDEFF3",
  textMuted: "#8B93A3",
  textFaint: "#5A6270",
  gold: "#C9A227",
  goldSoft: "#E8CB6A",
  crimson: "#B4342C",
  crimsonSoft: "#D66459",
  steel: "#4E9CC4",
  steelSoft: "#7FBEDE",
};
 
const FONT_CSS = `
@import url('https://fonts.googleapis.com/css2?family=Oswald:wght@400;500;600;700&family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500;700&display=swap');
.uf-display { font-family: 'Oswald', sans-serif; text-transform: uppercase; letter-spacing: 0.04em; }
.uf-body { font-family: 'Inter', sans-serif; }
.uf-mono { font-family: 'JetBrains Mono', monospace; }
.uf-card { transition: transform .18s ease, border-color .18s ease, box-shadow .18s ease; }
.uf-card:hover { transform: translateY(-2px); }
.uf-bar-fill { transition: width .6s cubic-bezier(.16,1,.3,1); }
.uf-fadein { animation: uf-fadein .35s ease both; }
@keyframes uf-fadein { from { opacity:0; transform: translateY(6px);} to {opacity:1; transform:translateY(0);} }
.uf-pulse { animation: uf-pulse 1.8s ease-in-out infinite; }
@keyframes uf-pulse { 0%,100% { opacity:1; } 50% { opacity:.55; } }
.uf-scrollbar::-webkit-scrollbar { width: 6px; }
.uf-scrollbar::-webkit-scrollbar-thumb { background: #2A2F39; border-radius: 4px; }
.uf-btn { transition: all .15s ease; }
.uf-btn:active { transform: scale(0.97); }
.uf-9 { font-size: 9px; }
.uf-10 { font-size: 10px; }
.uf-11 { font-size: 11px; }
.uf-track-lg { letter-spacing: 0.3em; }
@media (prefers-reduced-motion: reduce) {
  .uf-card, .uf-bar-fill, .uf-fadein, .uf-pulse, .uf-btn { animation: none !important; transition: none !important; }
}
`;
 
/* ============================== GAME DATA ============================== */
const ATTR_KEYS = ["stk", "sub", "qi", "tdd", "guarda", "cardio"];
const ATTR_LABELS = { stk: "STK", sub: "SUB", qi: "QI DE LUTA", tdd: "TDD", guarda: "GUARDA", cardio: "CARDIO" };
const ATTR_ICONS = { stk: Swords, sub: VenetianMask, qi: Brain, tdd: Shield, guarda: Shield, cardio: Activity };
const STAR_KEYS = ["chin", "ko"];
const STAR_LABELS = { chin: "QUEIXO", ko: "PODER DE NOCAUTE" };
 
const LEGENDS = [
  { name: "Georges St-Pierre", style: "Wrestling / Boxe", stk: 78, sub: 75, qi: 90, tdd: 96, guarda: 85, cardio: 92, chin: 88, ko: 70 },
  { name: "Khabib Nurmagomedov", style: "Sambo / Wrestling", stk: 65, sub: 90, qi: 88, tdd: 98, guarda: 82, cardio: 95, chin: 90, ko: 68 },
  { name: "Demian Maia", style: "Jiu-Jitsu", stk: 55, sub: 97, qi: 85, tdd: 70, guarda: 80, cardio: 78, chin: 75, ko: 40 },
  { name: "Fedor Emelianenko", style: "Sambo / Boxe", stk: 85, sub: 78, qi: 82, tdd: 70, guarda: 75, cardio: 80, chin: 94, ko: 96 },
  { name: "Chuck Liddell", style: "Kickboxing", stk: 88, sub: 45, qi: 78, tdd: 75, guarda: 65, cardio: 75, chin: 80, ko: 92 },
  { name: "Royce Gracie", style: "Jiu-Jitsu", stk: 40, sub: 95, qi: 90, tdd: 60, guarda: 88, cardio: 82, chin: 65, ko: 35 },
  { name: "Israel Adesanya", style: "Kickboxing", stk: 95, sub: 50, qi: 87, tdd: 60, guarda: 90, cardio: 85, chin: 78, ko: 82 },
  { name: "Amanda Nunes", style: "Muay Thai / Jiu-Jitsu", stk: 92, sub: 65, qi: 84, tdd: 78, guarda: 80, cardio: 87, chin: 85, ko: 93 },
  { name: "BJ Penn", style: "Jiu-Jitsu / Boxe", stk: 85, sub: 90, qi: 88, tdd: 65, guarda: 78, cardio: 70, chin: 82, ko: 75 },
  { name: "Daniel Cormier", style: "Wrestling", stk: 72, sub: 82, qi: 89, tdd: 94, guarda: 80, cardio: 93, chin: 87, ko: 78 },
  { name: "Conor McGregor", style: "Boxe / Karatê", stk: 90, sub: 50, qi: 85, tdd: 55, guarda: 72, cardio: 65, chin: 70, ko: 95 },
  { name: "Stipe Miocic", style: "Boxe / Wrestling", stk: 82, sub: 60, qi: 83, tdd: 80, guarda: 78, cardio: 96, chin: 90, ko: 84 },
];
 
const BOSSES = [
  {
    name: 'Charles "do Bronx" Oliveira',
    style: "Jiu-Jitsu / Boxe",
    testLabel: "Teste de Resiliência & SUB",
    testReq: { sub: 70, chin: 55 },
    ovr: 95, stk: 85, sub: 98, qi: 88, tdd: 68, guarda: 75, cardio: 88, chin: 78, ko: 80,
  },
  {
    name: "Ilia Topuria",
    style: "Muay Thai / Wrestling",
    testLabel: "Teste de Absorção de Dano & STK",
    testReq: { stk: 72, chin: 60 },
    ovr: 96, stk: 96, sub: 70, qi: 90, tdd: 78, guarda: 85, cardio: 88, chin: 92, ko: 94,
  },
  {
    name: "Arman Tsarukyan",
    style: "Wrestling / Sambo",
    testLabel: "Teste de TDD & Condicionamento",
    testReq: { tdd: 72, cardio: 72 },
    ovr: 94, stk: 82, sub: 75, qi: 87, tdd: 97, guarda: 82, cardio: 97, chin: 85, ko: 75,
  },
  {
    name: "Islam Makhachev",
    style: "Sambo / Wrestling",
    testLabel: "Teste Máximo de QI & Controle",
    testReq: { qi: 78, guarda: 70 },
    ovr: 98, stk: 78, sub: 88, qi: 99, tdd: 96, guarda: 95, cardio: 94, chin: 88, ko: 80,
  },
];
 
/* ============================== NACIONALIDADES ============================== */
const NATIONALITIES = [
  { code: "BR", name: "Brasil", flag: "🇧🇷", tier: 1 },
  { code: "US", name: "Estados Unidos", flag: "🇺🇸", tier: 1 },
  { code: "ES", name: "Espanha", flag: "🇪🇸", tier: 2 },
  { code: "MX", name: "México", flag: "🇲🇽", tier: 2 },
  { code: "GB", name: "Inglaterra", flag: "🇬🇧", tier: 2 },
  { code: "FR", name: "França", flag: "🇫🇷", tier: 2 },
  { code: "RU", name: "Rússia", flag: "🇷🇺", tier: 3 },
  { code: "IE", name: "Irlanda", flag: "🇮🇪", tier: 3 },
  { code: "NG", name: "Nigéria", flag: "🇳🇬", tier: 3 },
  { code: "CM", name: "Camarões", flag: "🇨🇲", tier: 3 },
  { code: "GE", name: "Geórgia", flag: "🇬🇪", tier: 3 },
  { code: "KZ", name: "Cazaquistão", flag: "🇰🇿", tier: 3 },
  { code: "AZ", name: "Azerbaijão", flag: "🇦🇿", tier: 3 },
  { code: "CA", name: "Canadá", flag: "🇨🇦", tier: 3 },
  { code: "AU", name: "Austrália", flag: "🇦🇺", tier: 3 },
  { code: "NZ", name: "Nova Zelândia", flag: "🇳🇿", tier: 3 },
  { code: "JP", name: "Japão", flag: "🇯🇵", tier: 3 },
  { code: "KR", name: "Coreia do Sul", flag: "🇰🇷", tier: 3 },
  { code: "CN", name: "China", flag: "🇨🇳", tier: 3 },
  { code: "UA", name: "Ucrânia", flag: "🇺🇦", tier: 3 },
  { code: "SE", name: "Suécia", flag: "🇸🇪", tier: 3 },
  { code: "NL", name: "Holanda", flag: "🇳🇱", tier: 3 },
  { code: "HR", name: "Croácia", flag: "🇭🇷", tier: 3 },
  { code: "RS", name: "Sérvia", flag: "🇷🇸", tier: 3 },
  { code: "PL", name: "Polônia", flag: "🇵🇱", tier: 3 },
  { code: "CU", name: "Cuba", flag: "🇨🇺", tier: 3 },
  { code: "CO", name: "Colômbia", flag: "🇨🇴", tier: 3 },
  { code: "AR", name: "Argentina", flag: "🇦🇷", tier: 3 },
  { code: "PE", name: "Peru", flag: "🇵🇪", tier: 3 },
  { code: "EC", name: "Equador", flag: "🇪🇨", tier: 3 },
  { code: "ZA", name: "África do Sul", flag: "🇿🇦", tier: 3 },
  { code: "MA", name: "Marrocos", flag: "🇲🇦", tier: 3 },
  { code: "JM", name: "Jamaica", flag: "🇯🇲", tier: 3 },
];
 
const TIER_INFO = {
  1: { label: "Tier 1 — Hype Máximo", desc: "1.5x Hype inicial · pressão psicológica escalonada em derrotas", multiplier: 1.5, color: C.gold },
  2: { label: "Tier 2 — Equilíbrio", desc: "1.2x Hype inicial · risco/retorno padrão", multiplier: 1.2, color: C.steelSoft },
  3: { label: "Tier 3 — Hype Orgânico", desc: "1.0x Hype inicial · hype só cresce com KO/SUB", multiplier: 1.0, color: C.textMuted },
};
 
const ARCHETYPES = [
  { id: "STK", name: "Striker", desc: "Bônus inicial em volume e precisão em pé.", icon: Swords, boost: { stk: 6, ko: 4 } },
  { id: "GRP", name: "Grappler", desc: "Bônus inicial em controle de solo e pressão isométrica.", icon: Shield, boost: { sub: 6, tdd: 4 } },
  { id: "ALL", name: "All-rounder", desc: "Perfil híbrido, atributos equilibrados.", icon: CircleDot, boost: { qi: 3, cardio: 3, guarda: 2 } },
];
 
const clamp = (n, min = 0, max = 99) => Math.max(min, Math.min(max, n));
const randInt = (min, max) => Math.floor(Math.random() * (max - min + 1)) + min;
const pick = (arr) => arr[randInt(0, arr.length - 1)];
 
function calcOVR(attrs) {
  const core = ATTR_KEYS.reduce((s, k) => s + (attrs[k] || 0), 0);
  const star = STAR_KEYS.reduce((s, k) => s + (attrs[k] || 0), 0);
  return clamp(Math.round((core + star * 0.75) / 7.5), 1, 99);
}
 
function tierLabel(ovr) {
  if (ovr >= 99) return { label: "O Fenômeno", sub: "G.O.A.T.", color: C.gold };
  if (ovr >= 90) return { label: "Lenda do Top 10", sub: "O Craque", color: C.goldSoft };
  if (ovr >= 80) return { label: "Contender", sub: "Ranqueado", color: C.steelSoft };
  return { label: "Card Preliminar", sub: "O Novato", color: C.textMuted };
}
 
function durabilityInfo(d) {
  if (d > 70) return { label: "Impecável", color: C.steelSoft };
  if (d > 40) return { label: "Estável", color: C.goldSoft };
  if (d > 15) return { label: "Desgastada", color: "#D68A3E" };
  return { label: "Crítica", color: C.crimson };
}
 
/* ============================== FIGHT SIM ============================== */
function simulateFight({ playerOVR, attrs, oppOVR, tempMod = 0 }) {
  const playerScore = playerOVR + tempMod + randInt(-8, 8);
  const oppScore = oppOVR + randInt(-8, 8);
  const win = playerScore > oppScore;
  const margin = Math.abs(playerScore - oppScore);
  let method;
  if (win) {
    const koThresh = (attrs.ko * 0.6 + attrs.stk * 0.4) * 0.55;
    const subThresh = koThresh + attrs.sub * 0.45;
    const roll = randInt(0, 100);
    method = roll < koThresh ? "Nocaute" : roll < subThresh ? "Finalização" : "Decisão";
  } else {
    const roll = randInt(0, 100);
    method = roll < 32 ? "Nocaute" : roll < 55 ? "Finalização" : "Decisão";
  }
  let bonus = null;
  if (win && (method === "Nocaute" || method === "Finalização") && Math.random() < 0.3) bonus = "Performance da Noite";
  else if (margin < 4 && Math.random() < 0.22) bonus = "Luta da Noite";
 
  let durabilityLoss = randInt(3, 9);
  if (!win) durabilityLoss += randInt(4, 8);
  if (method === "Nocaute" && !win) durabilityLoss += randInt(3, 6);
 
  return { win, method, bonus, durabilityLoss, margin };
}
 
/* ============================== SMALL UI PARTS ============================== */
function AttrBar({ label, value, donor, accent }) {
  return (
    <div className="mb-3">
      <div className="flex justify-between items-baseline mb-1">
        <span className="uf-display uf-11 tracking-wider" style={{ color: C.textMuted }}>{label}</span>
        <span className="uf-mono text-sm font-bold" style={{ color: C.textPrimary }}>{value}</span>
      </div>
      <div className="h-2 rounded-full overflow-hidden" style={{ background: C.borderSoft }}>
        <div className="h-full rounded-full uf-bar-fill" style={{ width: `${value}%`, background: `linear-gradient(90deg, ${accent}55, ${accent})` }} />
      </div>
      {donor && <div className="uf-mono uf-10 mt-1" style={{ color: C.textFaint }}>{donor}</div>}
    </div>
  );
}
 
function StarStat({ label, value, donor }) {
  const stars = Math.round(value / 20);
  return (
    <div className="p-3 rounded-lg" style={{ background: C.surfaceRaised, border: `1px solid ${C.borderSoft}` }}>
      <div className="uf-display uf-11 tracking-wider mb-1" style={{ color: C.textMuted }}>{label}</div>
      <div className="flex items-center gap-1 mb-1">
        {[1, 2, 3, 4, 5].map((i) => (
          <Star key={i} size={16} fill={i <= stars ? C.gold : "none"} color={i <= stars ? C.gold : C.textFaint} strokeWidth={1.5} />
        ))}
        <span className="uf-mono text-sm font-bold ml-1" style={{ color: C.textPrimary }}>{value}</span>
      </div>
      {donor && <div className="uf-mono uf-10" style={{ color: C.textFaint }}>{donor}</div>}
    </div>
  );
}
 
function Badge({ children, color = C.textMuted, bg = "transparent", border }) {
  return (
    <span className="uf-display uf-10 px-2 py-1 rounded" style={{ color, background: bg, border: border ? `1px solid ${border}` : "none" }}>
      {children}
    </span>
  );
}
 
/* ============================== START SCREEN ============================== */
function StartScreen({ onStart }) {
  const [name, setName] = useState("");
  const [archetype, setArchetype] = useState("ALL");
  const [difficulty, setDifficulty] = useState("amador");
  const [nationality, setNationality] = useState(NATIONALITIES[0]);
 
  return (
    <div className="max-w-2xl mx-auto px-5 py-10 uf-fadein">
      <div className="text-center mb-10">
        <div className="uf-mono text-xs uf-track-lg mb-2" style={{ color: C.gold }}>SIMULADOR DE CARREIRA</div>
        <h1 className="uf-display text-5xl font-bold" style={{ color: C.textPrimary }}>UFC <span style={{ color: C.gold }}>FENÔMENO</span></h1>
        <p className="uf-body text-sm mt-3" style={{ color: C.textMuted }}>Herde atributos de lendas do octógono. Construa o G.O.A.T.</p>
      </div>
 
      <div className="mb-7">
        <label className="uf-display text-xs tracking-wider block mb-2" style={{ color: C.textMuted }}>Nome do Lutador</label>
        <input
          value={name}
          onChange={(e) => setName(e.target.value)}
          placeholder="Ex: Ricardo 'Fenômeno' Alves"
          className="uf-body w-full px-4 py-3 rounded-lg outline-none"
          style={{ background: C.surface, border: `1px solid ${C.border}`, color: C.textPrimary }}
        />
      </div>
 
      <div className="mb-7">
        <label className="uf-display text-xs tracking-wider block mb-3" style={{ color: C.textMuted }}>Nacionalidade</label>
        <div className="p-3 rounded-xl mb-3 flex items-center gap-3" style={{ background: C.surfaceRaised, border: `1px solid ${C.border}` }}>
          <span style={{ fontSize: 28, lineHeight: 1 }}>{nationality.flag}</span>
          <div>
            <div className="uf-display text-sm" style={{ color: C.textPrimary }}>{nationality.name}</div>
            <div className="uf-body uf-10 mt-0.5" style={{ color: TIER_INFO[nationality.tier].color }}>{TIER_INFO[nationality.tier].label}</div>
          </div>
        </div>
        <div className="uf-body uf-11 mb-2" style={{ color: C.textFaint }}>{TIER_INFO[nationality.tier].desc}</div>
        <div className="uf-scrollbar grid grid-cols-8 gap-1.5 max-h-32 overflow-y-auto p-2 rounded-lg" style={{ background: C.surface, border: `1px solid ${C.borderSoft}` }}>
          {NATIONALITIES.map((n) => {
            const active = nationality.code === n.code;
            return (
              <button
                key={n.code}
                type="button"
                onClick={() => setNationality(n)}
                title={`${n.name} · ${TIER_INFO[n.tier].label}`}
                className="uf-btn flex items-center justify-center rounded-md py-1.5"
                style={{
                  fontSize: 17,
                  background: active ? C.surfaceRaised : "transparent",
                  border: `1px solid ${active ? TIER_INFO[n.tier].color : "transparent"}`,
                }}
              >
                {n.flag}
              </button>
            );
          })}
        </div>
      </div>
 
      <div className="mb-7">
        <label className="uf-display text-xs tracking-wider block mb-3" style={{ color: C.textMuted }}>Arquétipo</label>
        <div className="grid grid-cols-3 gap-3">
          {ARCHETYPES.map((a) => {
            const Icon = a.icon;
            const active = archetype === a.id;
            return (
              <button
                key={a.id}
                onClick={() => setArchetype(a.id)}
                className="uf-card uf-btn text-left p-4 rounded-xl"
                style={{
                  background: active ? C.surfaceRaised : C.surface,
                  border: `1px solid ${active ? C.gold : C.border}`,
                }}
              >
                <Icon size={20} color={active ? C.gold : C.textMuted} />
                <div className="uf-display text-sm mt-2" style={{ color: C.textPrimary }}>{a.name}</div>
                <div className="uf-body uf-11 mt-1 leading-snug" style={{ color: C.textFaint }}>{a.desc}</div>
              </button>
            );
          })}
        </div>
      </div>
 
      <div className="mb-9">
        <label className="uf-display text-xs tracking-wider block mb-3" style={{ color: C.textMuted }}>Modo de Dificuldade</label>
        <div className="grid grid-cols-2 gap-3">
          {[
            { id: "amador", name: "Amador", desc: "Números das lendas visíveis. 1 reroll cego disponível." },
            { id: "pro", name: "Pro", desc: "Números ocultos. Exige conhecimento real dos atletas. Sem reroll." },
          ].map((d) => {
            const active = difficulty === d.id;
            return (
              <button
                key={d.id}
                onClick={() => setDifficulty(d.id)}
                className="uf-card uf-btn text-left p-4 rounded-xl"
                style={{ background: active ? C.surfaceRaised : C.surface, border: `1px solid ${active ? C.steel : C.border}` }}
              >
                <div className="uf-display text-sm" style={{ color: C.textPrimary }}>{d.name}</div>
                <div className="uf-body uf-11 mt-1 leading-snug" style={{ color: C.textFaint }}>{d.desc}</div>
              </button>
            );
          })}
        </div>
      </div>
 
      <button
        onClick={() => onStart({ name: name.trim() || "Lutador Anônimo", archetype, difficulty, nationality })}
        className="uf-btn uf-display w-full py-4 rounded-xl text-sm tracking-wider flex items-center justify-center gap-2"
        style={{ background: C.gold, color: "#0A0C10", fontWeight: 700 }}
      >
        Iniciar Draft de Lendas <ChevronRight size={18} />
      </button>
    </div>
  );
}
 
/* ============================== DRAFT SCREEN ============================== */
function DraftScreen({ session, onComplete }) {
  const [pool, setPool] = useState(() => [...LEGENDS].sort(() => Math.random() - 0.5));
  const [current, setCurrent] = useState(0);
  const [filled, setFilled] = useState({});
  const [donors, setDonors] = useState({});
  const [rerolls, setRerolls] = useState(session.difficulty === "amador" ? 1 : 0);
  const [flash, setFlash] = useState(null);
 
  const legend = pool[current];
  const allKeys = [...ATTR_KEYS, ...STAR_KEYS];
  const remainingKeys = allKeys.filter((k) => !(k in filled));
  const slotsFilled = Object.keys(filled).length;
  const isPro = session.difficulty === "pro";
 
  function steal(key) {
    const val = legend[key];
    const newFilled = { ...filled, [key]: val };
    const newDonors = { ...donors, [key]: legend.name };
    setFilled(newFilled);
    setDonors(newDonors);
    setFlash({ key, val });
    setTimeout(() => setFlash(null), 550);
 
    if (Object.keys(newFilled).length >= 8) {
      setTimeout(() => onComplete({ attrs: newFilled, donors: newDonors }), 650);
    } else {
      setTimeout(() => setCurrent((c) => (c + 1) % pool.length), 400);
    }
  }
 
  function reroll() {
    if (rerolls <= 0) return;
    setRerolls((r) => r - 1);
    const discarded = pool[current];
    const others = pool.filter((_, i) => i !== current);
    const next = pick(others);
    setPool((p) => {
      const arr = p.filter((l) => l.name !== next.name && l.name !== discarded.name);
      const insertAt = Math.min(current, arr.length);
      arr.splice(insertAt, 0, next);
      return arr;
    });
  }
 
  return (
    <div className="max-w-2xl mx-auto px-5 py-8 uf-fadein">
      <div className="flex items-center justify-between mb-6">
        <div>
          <div className="uf-mono text-xs" style={{ color: C.gold }}>APRENDA COM AS LENDAS</div>
          <div className="uf-display text-lg" style={{ color: C.textPrimary }}>Slot {slotsFilled} de 8</div>
        </div>
        <div className="flex gap-1">
          {Array.from({ length: 8 }).map((_, i) => (
            <div key={i} className="w-5 h-1.5 rounded-full" style={{ background: i < slotsFilled ? C.gold : C.border }} />
          ))}
        </div>
      </div>
 
      {legend && (
        <div className="uf-card p-5 rounded-2xl mb-5" style={{ background: C.surface, border: `1px solid ${C.border}` }}>
          <div className="flex items-center justify-between mb-4">
            <div>
              <div className="uf-display text-xl" style={{ color: C.textPrimary }}>{legend.name}</div>
              <div className="uf-body text-xs" style={{ color: C.textFaint }}>{legend.style}</div>
            </div>
            <Badge color={C.gold} border={C.gold}>{isPro ? "MODO PRO" : "MODO AMADOR"}</Badge>
          </div>
 
          <div className="grid grid-cols-4 gap-2">
            {allKeys.map((k) => {
              const already = k in filled;
              const label = ATTR_LABELS[k] || STAR_LABELS[k];
              const value = legend[k];
              return (
                <button
                  key={k}
                  disabled={already}
                  onClick={() => steal(k)}
                  className="uf-btn uf-card p-3 rounded-lg text-center relative"
                  style={{
                    background: already ? C.borderSoft : C.surfaceRaised,
                    border: `1px solid ${already ? C.borderSoft : C.border}`,
                    opacity: already ? 0.35 : 1,
                    cursor: already ? "default" : "pointer",
                  }}
                >
                  <div className="uf-display uf-10 mb-1" style={{ color: C.textMuted }}>{label}</div>
                  <div className="uf-mono text-base font-bold" style={{ color: already ? C.textFaint : C.textPrimary }}>
                    {isPro && !already ? "??" : value}
                  </div>
                  {flash && flash.key === k && (
                    <div className="absolute inset-0 flex items-center justify-center rounded-lg uf-fadein" style={{ background: "#0A0C10CC" }}>
                      <span className="uf-mono text-sm font-bold" style={{ color: C.gold }}>+{flash.val}</span>
                    </div>
                  )}
                </button>
              );
            })}
          </div>
 
          {isPro && (
            <p className="uf-body uf-11 mt-3" style={{ color: C.textFaint }}>
              Números ocultos — use seu conhecimento sobre o estilo real do atleta para escolher onde roubar.
            </p>
          )}
        </div>
      )}
 
      <div className="flex items-center justify-between">
        <button
          onClick={reroll}
          disabled={rerolls <= 0}
          className="uf-btn uf-body text-xs px-4 py-2 rounded-lg flex items-center gap-2"
          style={{
            background: "transparent",
            border: `1px solid ${rerolls > 0 ? C.steel : C.border}`,
            color: rerolls > 0 ? C.steelSoft : C.textFaint,
            cursor: rerolls > 0 ? "pointer" : "not-allowed",
          }}
        >
          <RotateCcw size={14} /> Reroll cego ({rerolls} restante{rerolls === 1 ? "" : "s"})
        </button>
        <div className="uf-body uf-11" style={{ color: C.textFaint }}>
          Escolha 1 atributo por lenda para preencher sua ficha.
        </div>
      </div>
    </div>
  );
}
 
/* ============================== RADAR ============================== */
function AttrRadar({ attrs }) {
  const data = ATTR_KEYS.map((k) => ({ attr: ATTR_LABELS[k], value: attrs[k] || 0 }));
  return (
    <div style={{ width: "100%", height: 220 }}>
      <ResponsiveContainer>
        <RadarChart data={data} outerRadius="72%">
          <PolarGrid stroke={C.border} />
          <PolarAngleAxis dataKey="attr" tick={{ fill: C.textFaint, fontSize: 10, fontFamily: "Oswald" }} />
          <Radar dataKey="value" stroke={C.gold} fill={C.gold} fillOpacity={0.28} strokeWidth={2} />
        </RadarChart>
      </ResponsiveContainer>
    </div>
  );
}
 
/* ============================== FIGHTER SHEET TAB ============================== */
function FighterTab({ fighter }) {
  const ovr = calcOVR(fighter.attrs);
  const tier = tierLabel(ovr);
  const dur = durabilityInfo(fighter.durability);
  const craqueReqs = [
    { done: fighter.champTitles && Object.values(fighter.champTitles).some(Boolean), label: "Cinturão conquistado" },
    { done: fighter.top5Wins >= 9, label: `Vitórias vs Top 5 (${fighter.top5Wins}/9)` },
    { done: fighter.bonuses >= 1, label: `Bônus da Noite (${fighter.bonuses}/1)` },
    { done: fighter.bossesDefeated.length >= 1, label: `Portão vencido (${fighter.bossesDefeated.length}/1)` },
  ];
  const fenomenoReqs = [
    { done: fighter.titleDefenses >= 24, label: `Defesas de título (${fighter.titleDefenses}/24)` },
    { done: fighter.bonuses >= 8, label: `Bônus da Noite (${fighter.bonuses}/8)` },
    { done: fighter.champTitles && Object.values(fighter.champTitles).filter(Boolean).length >= 2, label: "Campeão Champ-Champ" },
    { done: fighter.bossesDefeated.length >= BOSSES.length, label: `Todos os portões (${fighter.bossesDefeated.length}/${BOSSES.length})` },
  ];
 
  return (
    <div className="uf-fadein">
      <div className="uf-card p-5 rounded-2xl mb-4" style={{ background: C.surface, border: `1px solid ${C.border}` }}>
        <AttrRadar attrs={fighter.attrs} />
      </div>
 
      <div className="grid grid-cols-1 gap-1 mb-5">
        {ATTR_KEYS.map((k) => (
          <AttrBar key={k} label={ATTR_LABELS[k]} value={fighter.attrs[k]} donor={fighter.donors[k]} accent={C.steel} />
        ))}
      </div>
 
      <div className="grid grid-cols-2 gap-3 mb-5">
        {STAR_KEYS.map((k) => (
          <StarStat key={k} label={STAR_LABELS[k]} value={fighter.attrs[k]} donor={fighter.donors[k]} />
        ))}
      </div>
 
      <div className="uf-card p-4 rounded-xl mb-3" style={{ background: C.surface, border: `1px solid ${C.border}` }}>
        <div className="flex justify-between items-center mb-2">
          <span className="uf-display text-xs tracking-wider" style={{ color: C.textMuted }}>RESILIÊNCIA DO CORPO</span>
          <Badge color={dur.color}>{dur.label}</Badge>
        </div>
        <div className="h-2 rounded-full overflow-hidden" style={{ background: C.borderSoft }}>
          <div className="h-full rounded-full uf-bar-fill" style={{ width: `${fighter.durability}%`, background: dur.color }} />
        </div>
      </div>
 
      <div className="uf-card p-4 rounded-xl mb-3" style={{ background: C.surface, border: `1px solid ${C.border}` }}>
        <div className="uf-display text-xs tracking-wider mb-3" style={{ color: C.textMuted }}>REQUISITOS — O CRAQUE (OVR 90)</div>
        {craqueReqs.map((r, i) => (
          <div key={i} className="flex items-center gap-2 mb-1.5">
            {r.done ? <CheckCircle2 size={14} color={C.goldSoft} /> : <Lock size={14} color={C.textFaint} />}
            <span className="uf-body text-xs" style={{ color: r.done ? C.textPrimary : C.textFaint }}>{r.label}</span>
          </div>
        ))}
      </div>
 
      <div className="uf-card p-4 rounded-xl" style={{ background: C.surface, border: `1px solid ${C.border}` }}>
        <div className="uf-display text-xs tracking-wider mb-3" style={{ color: C.textMuted }}>REQUISITOS — O FENÔMENO (OVR 99)</div>
        {fenomenoReqs.map((r, i) => (
          <div key={i} className="flex items-center gap-2 mb-1.5">
            {r.done ? <CheckCircle2 size={14} color={C.gold} /> : <Lock size={14} color={C.textFaint} />}
            <span className="uf-body text-xs" style={{ color: r.done ? C.textPrimary : C.textFaint }}>{r.label}</span>
          </div>
        ))}
      </div>
    </div>
  );
}
 
/* ============================== CAREER TAB ============================== */
function CareerTab({ fighter, onFight, onQuickSim, onWeightChange, onChallengeBoss, onRetire, onRecover }) {
  const ovr = calcOVR(fighter.attrs);
  const canChangeWeight = fighter.weightClass === "Peso Pena" && fighter.record.wins >= 5;
  const isTier1 = fighter.nationality && fighter.nationality.tier === 1;
  const pressurePct = Math.min((fighter.pressureStacks || 0) * 20, 60);
  const recoveryLocked = fighter.recoveryUsedThisCycle || fighter.durability >= 100;
 
  return (
    <div className="uf-fadein">
      <div className="grid grid-cols-3 gap-2 mb-5">
        <div className="p-3 rounded-xl text-center" style={{ background: C.surface, border: `1px solid ${C.border}` }}>
          <div className="uf-mono text-xl font-bold" style={{ color: C.textPrimary }}>{fighter.record.wins}-{fighter.record.losses}</div>
          <div className="uf-display uf-10" style={{ color: C.textFaint }}>CARTEL</div>
        </div>
        <div className="p-3 rounded-xl text-center" style={{ background: C.surface, border: `1px solid ${C.border}` }}>
          <div className="uf-mono text-xl font-bold" style={{ color: C.crimsonSoft }}>{fighter.kos}</div>
          <div className="uf-display uf-10" style={{ color: C.textFaint }}>KOs</div>
        </div>
        <div className="p-3 rounded-xl text-center" style={{ background: C.surface, border: `1px solid ${C.border}` }}>
          <div className="uf-mono text-xl font-bold" style={{ color: C.steelSoft }}>{fighter.subs}</div>
          <div className="uf-display uf-10" style={{ color: C.textFaint }}>SUBs</div>
        </div>
      </div>
 
      <div className="uf-card p-4 rounded-xl mb-4" style={{ background: C.surface, border: `1px solid ${C.border}` }}>
        <div className="flex items-center gap-2 mb-1">
          <Flame size={14} color={C.gold} />
          <span className="uf-display text-xs tracking-wider" style={{ color: C.textMuted }}>HYPE: {fighter.hype}</span>
          {fighter.nationality && (
            <span className="uf-display uf-9" style={{ color: C.textFaint }}>
              {fighter.nationality.flag} {TIER_INFO[fighter.nationality.tier].label}
            </span>
          )}
        </div>
        <div className="h-1.5 rounded-full overflow-hidden mb-3" style={{ background: C.borderSoft }}>
          <div className="h-full rounded-full uf-bar-fill" style={{ width: `${fighter.hype}%`, background: `linear-gradient(90deg, ${C.crimson}, ${C.gold})` }} />
        </div>
        {isTier1 && pressurePct > 0 && (
          <div className="mb-3">
            <Badge color={C.crimsonSoft} border={C.crimson}>⚠ Pressão psicológica −{pressurePct}% na próxima luta</Badge>
          </div>
        )}
        <div className="grid grid-cols-2 gap-2">
          <button onClick={() => onFight(true)} className="uf-btn uf-display text-xs py-3 rounded-lg flex items-center justify-center gap-2" style={{ background: C.crimson, color: "#fff" }}>
            <Mic size={14} /> Provocar e Lutar
          </button>
          <button onClick={() => onFight(false)} className="uf-btn uf-display text-xs py-3 rounded-lg flex items-center justify-center gap-2" style={{ background: C.steel, color: "#0A0C10" }}>
            <Swords size={14} /> Lutar em Silêncio
          </button>
        </div>
        <button onClick={onQuickSim} className="uf-btn uf-body uf-11 w-full mt-2 py-2 rounded-lg" style={{ background: "transparent", border: `1px solid ${C.border}`, color: C.textFaint }}>
          Simular 5 lutas rápido
        </button>
      </div>
 
      <div className="uf-card p-4 rounded-xl mb-4" style={{ background: C.surface, border: `1px solid ${C.border}` }}>
        <div className="uf-display text-xs tracking-wider mb-1" style={{ color: C.textMuted }}>RECUPERAÇÃO & GESTÃO DE RISCO</div>
        <div className="uf-body uf-11 mb-3" style={{ color: C.textFaint }}>
          Restaura durabilidade, mas reduz a taxa de crescimento de atributos por treino.
        </div>
        {fighter.growthPenalty && (
          <div className="mb-3">
            <Badge color={C.crimsonSoft} border={C.crimson}>
              Crescimento −{Math.round((1 - fighter.growthPenalty.multiplier) * 100)}% · {fighter.growthPenalty.cyclesRemaining} luta(s) restante(s)
            </Badge>
          </div>
        )}
        <div className="grid grid-cols-2 gap-2">
          <button
            onClick={() => onRecover("elite")}
            disabled={recoveryLocked}
            className="uf-btn uf-display uf-10 py-3 px-2 rounded-lg leading-snug"
            style={{
              background: "transparent",
              border: `1px solid ${C.steel}`,
              color: recoveryLocked ? C.textFaint : C.steelSoft,
              opacity: recoveryLocked ? 0.5 : 1,
              cursor: recoveryLocked ? "not-allowed" : "pointer",
            }}
          >
            Fisioterapia de Elite
            <div className="uf-body uf-9 mt-1 normal-case tracking-normal" style={{ color: C.textFaint }}>+40 dur · −50% cresc. (2 lutas)</div>
          </button>
          <button
            onClick={() => onRecover("ativo")}
            disabled={recoveryLocked}
            className="uf-btn uf-display uf-10 py-3 px-2 rounded-lg leading-snug"
            style={{
              background: "transparent",
              border: `1px solid ${C.border}`,
              color: recoveryLocked ? C.textFaint : C.textMuted,
              opacity: recoveryLocked ? 0.5 : 1,
              cursor: recoveryLocked ? "not-allowed" : "pointer",
            }}
          >
            Descanso Ativo
            <div className="uf-body uf-9 mt-1 normal-case tracking-normal" style={{ color: C.textFaint }}>+15 dur · −10% cresc. (1 luta)</div>
          </button>
        </div>
        {fighter.recoveryUsedThisCycle && (
          <div className="uf-body uf-10 mt-2" style={{ color: C.textFaint }}>Já usou recuperação neste ciclo — disponível de novo após a próxima luta.</div>
        )}
      </div>
 
      {canChangeWeight && (
        <button onClick={onWeightChange} className="uf-btn uf-card w-full text-left p-4 rounded-xl mb-4 flex items-center justify-between" style={{ background: C.surface, border: `1px solid ${C.goldSoft}` }}>
          <div>
            <div className="uf-display text-xs tracking-wider" style={{ color: C.goldSoft }}>SUBIR DE CATEGORIA</div>
            <div className="uf-body uf-11 mt-1" style={{ color: C.textFaint }}>Peso Pena → Peso Leve (+STK, −CARDIO)</div>
          </div>
          <ArrowUpCircle size={20} color={C.goldSoft} />
        </button>
      )}
 
      <div className="mb-4">
        <div className="uf-display text-xs tracking-wider mb-2" style={{ color: C.textMuted }}>PORTÕES DE VALIDAÇÃO — GATEKEEPERS DE ELITE</div>
        <div className="space-y-2">
          {BOSSES.map((b) => {
            const defeated = fighter.bossesDefeated.includes(b.name);
            const testOk = Object.entries(b.testReq).every(([k, v]) => (fighter.attrs[k] || 0) >= v);
            const eligible = ovr >= b.ovr - 15 && testOk;
            return (
              <div key={b.name} className="uf-card p-3 rounded-xl flex items-center justify-between" style={{ background: C.surface, border: `1px solid ${defeated ? C.gold : C.border}` }}>
                <div>
                  <div className="uf-display text-sm" style={{ color: C.textPrimary }}>{b.name}</div>
                  <div className="uf-body uf-11" style={{ color: C.textFaint }}>{b.style} · OVR {b.ovr}</div>
                  <div className="uf-body uf-10 mt-0.5" style={{ color: testOk ? C.steelSoft : C.crimsonSoft }}>{b.testLabel}</div>
                </div>
                {defeated ? (
                  <Badge color={C.gold} border={C.gold}>VENCIDO</Badge>
                ) : (
                  <button
                    onClick={() => onChallengeBoss(b)}
                    disabled={!eligible}
                    className="uf-btn uf-display uf-10 px-3 py-2 rounded-lg"
                    style={{ background: eligible ? C.crimson : C.borderSoft, color: eligible ? "#fff" : C.textFaint, cursor: eligible ? "pointer" : "not-allowed" }}
                  >
                    Desafiar
                  </button>
                )}
              </div>
            );
          })}
        </div>
      </div>
 
      <div className="mb-4">
        <div className="uf-display text-xs tracking-wider mb-2" style={{ color: C.textMuted }}>ÚLTIMOS RESULTADOS</div>
        <div className="uf-scrollbar space-y-1 max-h-40 overflow-y-auto pr-1">
          {fighter.log.length === 0 && <div className="uf-body text-xs" style={{ color: C.textFaint }}>Nenhuma luta ainda.</div>}
          {fighter.log.slice().reverse().map((l, i) => (
            <div key={i} className="uf-mono uf-11 px-3 py-2 rounded-lg" style={{ background: C.surface, color: l.win ? C.steelSoft : C.crimsonSoft, border: `1px solid ${C.borderSoft}` }}>
              {l.text}
            </div>
          ))}
        </div>
      </div>
 
      <button onClick={onRetire} className="uf-btn uf-body text-xs w-full py-3 rounded-lg" style={{ background: "transparent", border: `1px solid ${C.crimson}`, color: C.crimsonSoft }}>
        Aposentar Agora
      </button>
    </div>
  );
}
 
/* ============================== HALL OF FAME TAB ============================== */
function HallOfFameTab() {
  const [entries, setEntries] = useState(null);
  const [error, setError] = useState(false);
 
  useEffect(() => {
    (async () => {
      try {
        const list = await window.storage.list("career:", false);
        const keys = (list && list.keys) || [];
        const results = [];
        for (const k of keys) {
          try {
            const r = await window.storage.get(k, false);
            if (r && r.value) results.push(JSON.parse(r.value));
          } catch (e) { /* skip missing key */ }
        }
        results.sort((a, b) => b.finalOVR - a.finalOVR);
        setEntries(results);
      } catch (e) {
        setError(true);
        setEntries([]);
      }
    })();
  }, []);
 
  return (
    <div className="uf-fadein">
      <div className="uf-display text-xs tracking-wider mb-3" style={{ color: C.textMuted }}>HALL DA FAMA (SALVO NESTE DISPOSITIVO)</div>
      {entries === null && <div className="uf-body text-sm" style={{ color: C.textFaint }}>Carregando…</div>}
      {entries && entries.length === 0 && (
        <div className="p-6 rounded-xl text-center" style={{ background: C.surface, border: `1px solid ${C.border}` }}>
          <Trophy size={24} color={C.textFaint} className="mx-auto mb-2" />
          <div className="uf-body text-xs" style={{ color: C.textFaint }}>Nenhuma carreira encerrada ainda. Aposente um lutador para entrar no Hall da Fama.</div>
        </div>
      )}
      <div className="space-y-2">
        {entries && entries.map((e, i) => (
          <div key={i} className="uf-card p-4 rounded-xl flex items-center justify-between" style={{ background: C.surface, border: `1px solid ${i === 0 ? C.gold : C.border}` }}>
            <div>
              <div className="uf-display text-sm flex items-center gap-2" style={{ color: C.textPrimary }}>
                {i === 0 && <Crown size={14} color={C.gold} />} {e.nationality?.flag} {e.fighterName}
              </div>
              <div className="uf-body uf-11" style={{ color: C.textFaint }}>{e.tierLabel} · {e.record.wins}-{e.record.losses} · {e.weightClass}</div>
            </div>
            <div className="uf-mono text-lg font-bold" style={{ color: C.gold }}>{e.finalOVR}</div>
          </div>
        ))}
      </div>
    </div>
  );
}
 
/* ============================== RETIRED SCREEN ============================== */
function RetiredScreen({ fighter, reason, onNewCareer }) {
  const ovr = calcOVR(fighter.attrs);
  const tier = tierLabel(ovr);
  const stats = [
    { label: "Vitórias por KO (carreira)", value: fighter.kos, equiv: "Gols na temporada" },
    { label: "Lutas na Carreira", value: fighter.record.wins + fighter.record.losses, equiv: "Jogos na carreira" },
    { label: "Vitórias por via rápida (KO/SUB)", value: fighter.kos + fighter.subs, equiv: "Gols na carreira" },
    { label: "Vitórias por Decisão", value: fighter.decisions, equiv: "Assistências" },
    { label: "Cinturões em Divisões Diferentes", value: Object.values(fighter.champTitles || {}).filter(Boolean).length, equiv: "Copas do Mundo" },
    { label: "Bônus da Noite", value: fighter.bonuses, equiv: "Bolas de Ouro" },
    { label: "Vitórias vs Hall of Famers", value: fighter.bossesDefeated.length, equiv: "Jogos pela seleção" },
    { label: "Defesas de Cinturão", value: fighter.titleDefenses, equiv: "Títulos de liga" },
  ];
 
  useEffect(() => {
    (async () => {
      try {
        const record = {
          fighterName: fighter.name,
          weightClass: fighter.weightClass,
          finalOVR: ovr,
          tierLabel: tier.label,
          record: fighter.record,
          kos: fighter.kos,
          subs: fighter.subs,
          titleDefenses: fighter.titleDefenses,
          bonuses: fighter.bonuses,
          bossesDefeated: fighter.bossesDefeated,
          nationality: fighter.nationality,
          retiredAt: new Date().toISOString(),
        };
        await window.storage.set(`career:${Date.now()}`, JSON.stringify(record), false);
      } catch (e) { /* storage unavailable, ignore */ }
      // eslint-disable-next-line react-hooks/exhaustive-deps
    })();
  }, []);
 
  return (
    <div className="max-w-2xl mx-auto px-5 py-10 uf-fadein">
      <div className="text-center mb-8">
        <Skull size={28} color={C.crimsonSoft} className="mx-auto mb-3" />
        <div className="uf-mono text-xs uf-track-lg" style={{ color: C.crimsonSoft }}>{reason}</div>
        <h1 className="uf-display text-4xl font-bold mt-2 flex items-center justify-center gap-2" style={{ color: C.textPrimary }}>
          {fighter.nationality?.flag} {fighter.name}
        </h1>
        <div className="uf-display text-sm mt-1" style={{ color: tier.color }}>{tier.label} — {tier.sub} · OVR {ovr}</div>
      </div>
 
      <div className="grid grid-cols-1 gap-2 mb-8">
        {stats.map((s, i) => (
          <div key={i} className="uf-card p-4 rounded-xl flex items-center justify-between" style={{ background: C.surface, border: `1px solid ${C.border}` }}>
            <div>
              <div className="uf-body text-sm" style={{ color: C.textPrimary }}>{s.label}</div>
              <div className="uf-body uf-10" style={{ color: C.textFaint }}>Equivalente a: {s.equiv}</div>
            </div>
            <div className="uf-mono text-xl font-bold" style={{ color: C.gold }}>{s.value}</div>
          </div>
        ))}
      </div>
 
      <button onClick={onNewCareer} className="uf-btn uf-display w-full py-4 rounded-xl text-sm tracking-wider" style={{ background: C.gold, color: "#0A0C10", fontWeight: 700 }}>
        Iniciar Nova Carreira
      </button>
    </div>
  );
}
 
/* ============================== HUB ============================== */
function Hub({ fighter, dispatch, onRetireFinal }) {
  const [tab, setTab] = useState("ficha");
  const ovr = calcOVR(fighter.attrs);
  const tier = tierLabel(ovr);
 
  function resolveFight(oppOVR, trashTalk, isBoss = false, bossName = null) {
    let tempMod = 0;
    let hypeDelta = 0;
    let talkMsg = "";
    if (trashTalk) {
      const successChance = clamp(50 + (fighter.attrs.qi - 70) * 0.5 + fighter.hype * 0.1, 10, 90);
      const success = randInt(0, 100) < successChance;
      if (success) { tempMod = 4; hypeDelta = 15; talkMsg = "Provocação certeira. "; }
      else { tempMod = -6; hypeDelta = -20; talkMsg = "A provocação se voltou contra você. "; }
    }
 
    // Pressão psicológica (Tier 1): escalona 20% por derrota consecutiva, até 60%.
    const tier = fighter.nationality ? fighter.nationality.tier : 3;
    let pressureApplied = false;
    if (tier === 1 && fighter.pressureStacks > 0) {
      const pct = Math.min(fighter.pressureStacks * 0.2, 0.6);
      tempMod -= Math.round(ovr * pct);
      pressureApplied = true;
    }
 
    const result = simulateFight({ playerOVR: ovr, attrs: fighter.attrs, oppOVR, tempMod });
    dispatch({ type: "APPLY_FIGHT", result, hypeDelta, talkMsg, isBoss, bossName, pressureApplied });
  }
 
  function onFight(trashTalk) {
    const oppOVR = clamp(ovr + randInt(-10, 6), 40, 96);
    resolveFight(oppOVR, trashTalk, false);
  }
 
  function onQuickSim() {
    dispatch({ type: "QUICK_SIM_START" });
  }
 
  function onChallengeBoss(boss) {
    resolveFight(boss.ovr, false, true, boss.name);
  }
 
  function onRecover(kind) {
    dispatch({ type: "RECOVER", kind });
  }
 
  const tabs = [
    { id: "ficha", label: "Ficha", icon: Users },
    { id: "carreira", label: "Carreira", icon: Trophy },
    { id: "fama", label: "Hall da Fama", icon: Award },
  ];
 
  return (
    <div className="max-w-2xl mx-auto px-5 py-6 pb-16">
      <div className="uf-card p-4 rounded-2xl mb-5 flex items-center justify-between" style={{ background: `linear-gradient(135deg, ${C.surfaceRaised}, ${C.surface})`, border: `1px solid ${C.border}` }}>
        <div>
          <div className="uf-display text-lg leading-tight flex items-center gap-2" style={{ color: C.textPrimary }}>
            {fighter.nationality?.flag} {fighter.name}
          </div>
          <div className="flex items-center gap-2 mt-1">
            <Badge color={C.textMuted} border={C.border}>{fighter.weightClass}</Badge>
            <Badge color={tier.color} border={tier.color}>{tier.label}</Badge>
          </div>
        </div>
        <div className="text-right">
          <div className="uf-mono text-3xl font-bold" style={{ color: C.gold }}>{ovr}</div>
          <div className="uf-display uf-9" style={{ color: C.textFaint }}>OVR</div>
        </div>
      </div>
 
      <div className="flex gap-1 mb-5 p-1 rounded-xl" style={{ background: C.surface, border: `1px solid ${C.borderSoft}` }}>
        {tabs.map((t) => {
          const Icon = t.icon;
          const active = tab === t.id;
          return (
            <button
              key={t.id}
              onClick={() => setTab(t.id)}
              className="uf-btn flex-1 uf-display uf-11 py-2.5 rounded-lg flex items-center justify-center gap-1.5"
              style={{ background: active ? C.surfaceRaised : "transparent", color: active ? C.gold : C.textFaint }}
            >
              <Icon size={13} /> {t.label}
            </button>
          );
        })}
      </div>
 
      {tab === "ficha" && <FighterTab fighter={fighter} />}
      {tab === "carreira" && (
        <CareerTab
          fighter={fighter}
          onFight={onFight}
          onQuickSim={onQuickSim}
          onWeightChange={() => dispatch({ type: "WEIGHT_CHANGE" })}
          onChallengeBoss={onChallengeBoss}
          onRetire={onRetireFinal}
          onRecover={onRecover}
        />
      )}
      {tab === "fama" && <HallOfFameTab />}
    </div>
  );
}
 
/* ============================== APP ============================== */
function makeFighter({ name, archetype, difficulty, attrs, donors, nationality }) {
  const boost = ARCHETYPES.find((a) => a.id === archetype)?.boost || {};
  const boosted = { ...attrs };
  Object.entries(boost).forEach(([k, v]) => { boosted[k] = clamp((boosted[k] || 0) + v); });
  const nat = nationality || NATIONALITIES[0];
  const multiplier = TIER_INFO[nat.tier].multiplier;
  return {
    name, archetype, difficulty, attrs: boosted, donors,
    nationality: nat,
    weightClass: "Peso Pena",
    durability: 100,
    record: { wins: 0, losses: 0 },
    kos: 0, subs: 0, decisions: 0,
    titleDefenses: 0, bonuses: 0, top5Wins: 0,
    champTitles: {}, consecutiveWins: 0,
    bossesDefeated: [], hype: clamp(Math.round(20 * multiplier), 0, 100),
    pressureStacks: 0,
    growthPenalty: null,
    recoveryUsedThisCycle: false,
    log: [],
  };
}
 
function fighterReducer(fighter, action) {
  if (action.type === "WEIGHT_CHANGE") {
    return {
      ...fighter,
      weightClass: "Peso Leve",
      attrs: { ...fighter.attrs, stk: clamp(fighter.attrs.stk + 4), cardio: clamp(fighter.attrs.cardio - 4) },
      log: [...fighter.log, { win: true, text: `⬆ Subiu para Peso Leve. Ganhou massa: +STK, −CARDIO.` }],
    };
  }
 
  if (action.type === "RECOVER") {
    if (fighter.recoveryUsedThisCycle) return fighter;
    const isElite = action.kind === "elite";
    const restore = isElite ? 40 : 15;
    const penalty = isElite ? { multiplier: 0.5, cyclesRemaining: 2 } : { multiplier: 0.9, cyclesRemaining: 1 };
    const label = isElite ? "Fisioterapia de Elite" : "Descanso Ativo";
    return {
      ...fighter,
      durability: clamp(fighter.durability + restore, 0, 100),
      growthPenalty: penalty,
      recoveryUsedThisCycle: true,
      log: [...fighter.log, { win: true, text: `🩹 ${label}: +${restore} durabilidade · crescimento −${Math.round((1 - penalty.multiplier) * 100)}% por ${penalty.cyclesRemaining} luta(s)` }],
    };
  }
 
  if (action.type === "APPLY_FIGHT") {
    const { result, hypeDelta, talkMsg, isBoss, bossName, pressureApplied } = action;
    const rank = randInt(1, 15);
    let f = { ...fighter };
    const tier = f.nationality ? f.nationality.tier : 3;
 
    f.durability = clamp(f.durability - result.durabilityLoss, 0, 100);
 
    const wasChamp = !!(f.champTitles && f.champTitles[f.weightClass]);
 
    if (result.win) {
      const isFinish = result.method === "Nocaute" || result.method === "Finalização";
      const winHypeGain = tier === 3 ? (isFinish ? 8 : 0) : 5;
      f.hype = clamp(f.hype + hypeDelta + winHypeGain, 0, 100);
      f.pressureStacks = 0;
 
      f.consecutiveWins = (f.consecutiveWins || 0) + 1;
      f.record = { ...f.record, wins: f.record.wins + 1 };
      if (rank <= 5) f.top5Wins = (f.top5Wins || 0) + 1;
      if (result.method === "Nocaute") f.kos += 1;
      else if (result.method === "Finalização") f.subs += 1;
      else f.decisions += 1;
      if (result.bonus === "Performance da Noite" || result.bonus === "Luta da Noite") f.bonuses += 1;
 
      if (isBoss) {
        f.bossesDefeated = f.bossesDefeated.includes(bossName) ? f.bossesDefeated : [...f.bossesDefeated, bossName];
      }
 
      if (wasChamp) {
        f.titleDefenses = (f.titleDefenses || 0) + 1;
      } else if (f.consecutiveWins >= 5 && calcOVR(f.attrs) >= 85) {
        f.champTitles = { ...f.champTitles, [f.weightClass]: true };
      }
 
      // Treino / crescimento orgânico de OVR — reduzido enquanto houver penalidade de recuperação ativa.
      const growthRate = f.growthPenalty && f.growthPenalty.cyclesRemaining > 0 ? f.growthPenalty.multiplier : 1;
      let growthTxt = "";
      if (Math.random() < 0.55 * growthRate) {
        const growable = [...ATTR_KEYS, ...STAR_KEYS].filter((k) => (f.attrs[k] || 0) < 99);
        if (growable.length) {
          const gk = pick(growable);
          f.attrs = { ...f.attrs, [gk]: clamp((f.attrs[gk] || 0) + 1) };
          growthTxt = ` · Treino: +1 ${ATTR_LABELS[gk] || STAR_LABELS[gk]}`;
        }
      }
 
      const oppLabel = isBoss ? bossName : `Contender #${rank}`;
      const bonusTxt = result.bonus ? ` · Bônus: ${result.bonus}` : "";
      const pressureTxt = pressureApplied ? " · Sob pressão psicológica" : "";
      f.log = [...f.log, { win: true, text: `${talkMsg}V (${result.method}) vs ${oppLabel}${bonusTxt}${pressureTxt}${growthTxt}` }];
    } else {
      f.hype = clamp(f.hype + hypeDelta - 10, 0, 100);
      f.consecutiveWins = 0;
      f.record = { ...f.record, losses: f.record.losses + 1 };
      if (tier === 1) f.pressureStacks = (f.pressureStacks || 0) + 1;
 
      const oppLabel = isBoss ? bossName : `Contender #${rank}`;
      let beltTxt = "";
      if (wasChamp) {
        f.champTitles = { ...f.champTitles, [f.weightClass]: false };
        beltTxt = " · Cinturão perdido!";
      }
      const pressureTxt = pressureApplied ? " · Sob pressão psicológica" : "";
      f.log = [...f.log, { win: false, text: `${talkMsg}D (${result.method}) vs ${oppLabel}${beltTxt}${pressureTxt}` }];
    }
 
    // Avança o ciclo de recuperação/crescimento a cada luta.
    f.recoveryUsedThisCycle = false;
    if (f.growthPenalty) {
      const remaining = f.growthPenalty.cyclesRemaining - 1;
      f.growthPenalty = remaining > 0 ? { ...f.growthPenalty, cyclesRemaining: remaining } : null;
    }
 
    return f;
  }
  return fighter;
}
 
export default function App() {
  const [stage, setStage] = useState("start"); // start | draft | hub | retired
  const [session, setSession] = useState(null);
  const [fighter, setFighter] = useState(null);
  const [retireReason, setRetireReason] = useState("");
  const quickSimRef = useRef(false);
 
  function dispatch(action) {
    setFighter((f) => {
      const next = fighterReducer(f, action);
      if (next.durability <= 0 && f.durability > 0) {
        setTimeout(() => finishCareer(next, "APOSENTADORIA FORÇADA — CORPO NO LIMITE"), 500);
      }
      return next;
    });
  }
 
  function finishCareer(f, reason) {
    setRetireReason(reason);
    setStage("retired");
  }
 
  function onStart(sess) {
    setSession(sess);
    setStage("draft");
  }
 
  function onDraftComplete({ attrs, donors }) {
    const f = makeFighter({ ...session, attrs, donors });
    setFighter(f);
    setStage("hub");
  }
 
  function onQuickSim() {
    let f = fighter;
    for (let i = 0; i < 5; i++) {
      if (f.durability <= 0) break;
      const ovr = calcOVR(f.attrs);
      const oppOVR = clamp(ovr + randInt(-10, 6), 40, 96);
      const tier = f.nationality ? f.nationality.tier : 3;
      let tempMod = 0;
      let pressureApplied = false;
      if (tier === 1 && f.pressureStacks > 0) {
        const pct = Math.min(f.pressureStacks * 0.2, 0.6);
        tempMod -= Math.round(ovr * pct);
        pressureApplied = true;
      }
      const result = simulateFight({ playerOVR: ovr, attrs: f.attrs, oppOVR, tempMod });
      f = fighterReducer(f, { type: "APPLY_FIGHT", result, hypeDelta: 0, talkMsg: "", isBoss: false, bossName: null, pressureApplied });
    }
    setFighter(f);
    if (f.durability <= 0) {
      setTimeout(() => finishCareer(f, "APOSENTADORIA FORÇADA — CORPO NO LIMITE"), 300);
    }
  }
 
  function onRetireFinal() {
    finishCareer(fighter, "APOSENTADORIA VOLUNTÁRIA");
  }
 
  function onNewCareer() {
    setStage("start");
    setSession(null);
    setFighter(null);
  }
 
  return (
    <div className="min-h-screen uf-body" style={{ background: C.bgGrad, backgroundColor: C.bg }}>
      <style>{FONT_CSS}</style>
      {stage === "start" && <StartScreen onStart={onStart} />}
      {stage === "draft" && <DraftScreen session={session} onComplete={onDraftComplete} />}
      {stage === "hub" && fighter && (
        <Hub
          fighter={fighter}
          dispatch={(action) => {
            if (action.type === "QUICK_SIM_START") { onQuickSim(); return; }
            dispatch(action);
          }}
          onRetireFinal={onRetireFinal}
        />
      )}
      {stage === "retired" && fighter && <RetiredScreen fighter={fighter} reason={retireReason} onNewCareer={onNewCareer} />}
    </div>
  );
}
 
