import { useState, useEffect, useCallback, useRef } from "react";

// ─── FIREBASE CONFIG ──────────────────────────────────────────────────────────
import { initializeApp } from "firebase/app";
import { getDatabase, ref, get, set, onValue } from "firebase/database";

const firebaseConfig = {
  apiKey: "AIzaSyDnqkfkO7OIM2BhwuuzWWfAOwd0uwuBO5I",
  authDomain: "prode-mundial-2026-9599d.firebaseapp.com",
  databaseURL: "https://prode-mundial-2026-9599d-default-rtdb.firebaseio.com",
  projectId: "prode-mundial-2026-9599d",
  storageBucket: "prode-mundial-2026-9599d.firebasestorage.app",
  messagingSenderId: "798316010435",
  appId: "1:798316010435:web:cd3bbf3db54f73ed90fb76",
};

const firebaseApp = initializeApp(firebaseConfig, "eliminatoria");
const db = getDatabase(firebaseApp);

// ─── CONFIG ───────────────────────────────────────────────────────────────────
const ADMIN_PIN = "2026";
const DEADLINE_16VOS = new Date("2026-06-28T15:30:00Z"); // 28 jun 12:30hs ARG

// ─── MATCH DATA ───────────────────────────────────────────────────────────────
const FLAGS = {
  "Sudáfrica":"🇿🇦","Canadá":"🇨🇦","Brasil":"🇧🇷","Japón":"🇯🇵",
  "Alemania":"🇩🇪","Paraguay":"🇵🇾","Países Bajos":"🇳🇱","Marruecos":"🇲🇦",
  "Costa de Marfil":"🇨🇮","Noruega":"🇳🇴","Francia":"🇫🇷","Suecia":"🇸🇪",
  "México":"🇲🇽","Ecuador":"🇪🇨","Inglaterra":"🏴󠁧󠁢󠁥󠁮󠁧󠁿","Senegal":"🇸🇳",
  "Bélgica":"🇧🇪","Corea del Sur":"🇰🇷","EE.UU.":"🇺🇸","Bosnia y H.":"🇧🇦",
  "España":"🇪🇸","Austria":"🇦🇹","Argentina":"🇦🇷","Cabo Verde":"🇨🇻",
  "Colombia":"🇨🇴","Croacia":"🇭🇷","Suiza":"🇨🇭","Uruguay":"🇺🇾",
  "Portugal":"🇵🇹","Ghana":"🏴󠁧󠁢󠁷󠁬󠁳󠁿","Egipto":"🇪🇬","Irán":"🇮🇷",
};

const ROUNDS = [
  {
    id: "16vos",
    label: "Dieciseisavos",
    emoji: "⚔️",
    deadline: new Date("2026-06-28T15:30:00Z"), // 28 jun 12:30hs ARG (30 min antes de Sudáfrica vs Canadá)
    matches: [
      { id: "R1_01", h: "Sudáfrica",    a: "Canadá",         date: "28/06", venue: "Los Ángeles" },
      { id: "R1_02", h: "Brasil",        a: "Japón",          date: "29/06", venue: "Houston" },
      { id: "R1_03", h: "Alemania",      a: "Paraguay",       date: "29/06", venue: "Boston" },
      { id: "R1_04", h: "Países Bajos",  a: "Marruecos",      date: "29/06", venue: "Monterrey" },
      { id: "R1_05", h: "Costa de Marfil", a: "Noruega",      date: "30/06", venue: "Dallas" },
      { id: "R1_06", h: "Francia",       a: "Suecia",         date: "30/06", venue: "Nueva York" },
      { id: "R1_07", h: "México",        a: "Ecuador",        date: "30/06", venue: "Ciudad de México" },
      { id: "R1_08", h: "Inglaterra",    a: "Senegal",        date: "01/07", venue: "Guadalajara" },
      { id: "R1_09", h: "Bélgica",       a: "Corea del Sur",  date: "01/07", venue: "Seattle" },
      { id: "R1_10", h: "EE.UU.",        a: "Bosnia y H.",    date: "01/07", venue: "San Francisco" },
      { id: "R1_11", h: "España",        a: "Austria",        date: "02/07", venue: "Los Ángeles" },
      { id: "R1_12", h: "Argentina",     a: "Cabo Verde",     date: "03/07", venue: "Miami" },
      { id: "R1_13", h: "Colombia",      a: "Croacia",        date: "03/07", venue: "Kansas City" },
      { id: "R1_14", h: "Suiza",         a: "Uruguay",        date: "03/07", venue: "Atlanta" },
      { id: "R1_15", h: "Portugal",      a: "Ghana",          date: "02/07", venue: "Toronto" },
      { id: "R1_16", h: "Egipto",        a: "Irán",           date: "01/07", venue: "Vancouver" },
    ],
  },
  {
    id: "8vos",
    label: "Octavos",
    emoji: "🔥",
    deadline: new Date("2026-07-04T18:30:00Z"), // 04 jul 15:30hs ARG (30 min antes del primer octavo)
    matches: [
      { id: "R2_01", h: "Gan. P1", a: "Gan. P2", date: "04/07", venue: "TBD" },
      { id: "R2_02", h: "Gan. P3", a: "Gan. P4", date: "04/07", venue: "TBD" },
      { id: "R2_03", h: "Gan. P5", a: "Gan. P6", date: "05/07", venue: "TBD" },
      { id: "R2_04", h: "Gan. P7", a: "Gan. P8", date: "05/07", venue: "TBD" },
      { id: "R2_05", h: "Gan. P9", a: "Gan. P10", date: "06/07", venue: "TBD" },
      { id: "R2_06", h: "Gan. P11", a: "Gan. P12", date: "06/07", venue: "TBD" },
      { id: "R2_07", h: "Gan. P13", a: "Gan. P14", date: "07/07", venue: "TBD" },
      { id: "R2_08", h: "Gan. P15", a: "Gan. P16", date: "07/07", venue: "TBD" },
    ],
  },
  {
    id: "4tos",
    label: "Cuartos",
    emoji: "⚡",
    deadline: new Date("2026-07-09T18:30:00Z"), // 09 jul 15:30hs ARG (30 min antes del primer cuarto)
    matches: [
      { id: "R3_01", h: "TBD", a: "TBD", date: "09/07", venue: "TBD" },
      { id: "R3_02", h: "TBD", a: "TBD", date: "09/07", venue: "TBD" },
      { id: "R3_03", h: "TBD", a: "TBD", date: "10/07", venue: "TBD" },
      { id: "R3_04", h: "TBD", a: "TBD", date: "10/07", venue: "TBD" },
    ],
  },
  {
    id: "semis",
    label: "Semifinales",
    emoji: "🌟",
    deadline: new Date("2026-07-14T18:30:00Z"), // 14 jul 15:30hs ARG (30 min antes de la primera semi)
    matches: [
      { id: "R4_01", h: "TBD", a: "TBD", date: "14/07", venue: "TBD" },
      { id: "R4_02", h: "TBD", a: "TBD", date: "15/07", venue: "TBD" },
    ],
  },
  {
    id: "final",
    label: "Final",
    emoji: "🏆",
    deadline: new Date("2026-07-19T19:30:00Z"), // 19 jul 16:30hs ARG (30 min antes de la Final)
    matches: [
      { id: "R5_01", h: "TBD", a: "TBD", date: "19/07", venue: "MetLife Stadium" },
    ],
  },
];

const ALL_MATCHES = ROUNDS.flatMap(r => r.matches);

// ─── SCORING ──────────────────────────────────────────────────────────────────
function getOutcome(h, a) { return h > a ? "H" : a > h ? "A" : "D"; }
function calcPoints(pred, real) {
  if (pred?.h == null || pred?.h === "" || pred?.a == null || pred?.a === "") return null;
  if (real?.h == null || real?.h === "" || real?.a == null || real?.a === "") return null;
  const ph = parseInt(pred.h), pa = parseInt(pred.a), rh = parseInt(real.h), ra = parseInt(real.a);
  if (ph === rh && pa === ra) return 2;
  if (getOutcome(ph, pa) === getOutcome(rh, ra)) return 1;
  return 0;
}

// ─── FIREBASE HELPERS ─────────────────────────────────────────────────────────
async function fbGet(path) {
  try { const s = await get(ref(db, path)); return s.exists() ? s.val() : null; } catch { return null; }
}
async function fbSet(path, val) {
  try { await set(ref(db, path), val); } catch (e) { console.error(e); }
}

// ─── COUNTDOWN ────────────────────────────────────────────────────────────────
function useCountdown(deadline) {
  const [timeLeft, setTimeLeft] = useState(null);
  useEffect(() => {
    function calc() {
      const diff = deadline - new Date();
      if (diff <= 0) { setTimeLeft(null); return; }
      setTimeLeft({
        d: Math.floor(diff / 86400000),
        h: Math.floor((diff % 86400000) / 3600000),
        m: Math.floor((diff % 3600000) / 60000),
        s: Math.floor((diff % 60000) / 1000),
      });
    }
    calc();
    const id = setInterval(calc, 1000);
    return () => clearInterval(id);
  }, [deadline]);
  return timeLeft;
}

// ─── THEME ────────────────────────────────────────────────────────────────────
const T = {
  bg: "#05080F",
  surface: "#0A1020",
  card: "#0D1525",
  border: "rgba(255,255,255,.07)",
  gold: "#F5C842",
  silver: "#A8B8C8",
  teal: "#00D4A0",
  red: "#E63946",
  green: "#2DC653",
  muted: "#3A5570",
  text: "#C8D8E8",
};

// Round colors
const RC = {
  "16vos": "#00D4A0",
  "8vos":  "#F5C842",
  "4tos":  "#FF6B35",
  "semis": "#BD93F9",
  "final": "#FFD700",
};

// ─── MAIN APP ─────────────────────────────────────────────────────────────────
export default function App() {
  const [tab, setTab] = useState("ranking");
  const [activeRound, setActiveRound] = useState("16vos");
  const [user, setUser] = useState(null);
  const [participants, setParticipants] = useState({});
  const [myPreds, setMyPreds] = useState({});
  const [results, setResults] = useState({});
  const [allPreds, setAllPreds] = useState({});
  const [roundData, setRoundData] = useState(ROUNDS);
  const [loading, setLoading] = useState(true);
  const [saving, setSaving] = useState(false);
  const [saveMsg, setSaveMsg] = useState("");
  const [nameInput, setNameInput] = useState("");
  const [nameError, setNameError] = useState("");
  const [isAdmin, setIsAdmin] = useState(false);
  const [adminPin, setAdminPin] = useState("");
  const [adminPinErr, setAdminPinErr] = useState("");
  const [adminResults, setAdminResults] = useState({});
  const [adminRounds, setAdminRounds] = useState(ROUNDS);
  const [editingMatch, setEditingMatch] = useState(null);

  const currentRound = ROUNDS.find(r => r.id === activeRound);
  const isLocked = currentRound ? new Date() >= currentRound.deadline : true;
  const countdown = useCountdown(currentRound?.deadline || DEADLINE_16VOS);

  // ── Load ──
  useEffect(() => {
    (async () => {
      setLoading(true);
      const [parts, res, savedRounds] = await Promise.all([
        fbGet("elim/participants"),
        fbGet("elim/results"),
        fbGet("elim/rounds"),
      ]);
      setParticipants(parts || {});
      setResults(res || {});
      setAdminResults(res || {});
      if (savedRounds) {
        setRoundData(savedRounds);
        setAdminRounds(savedRounds);
      }
      try {
        const saved = localStorage.getItem("prode26elim:me");
        if (saved) {
          const u = JSON.parse(saved);
          setUser(u);
          const pr = await fbGet(`elim/predictions/${u.id}`);
          setMyPreds(pr || {});
        }
      } catch {}
      setLoading(false);
    })();
  }, []);

  useEffect(() => {
    const unsub = onValue(ref(db, "elim/results"), snap => {
      const val = snap.val() || {};
      setResults(val);
      setAdminResults(val);
    });
    return () => unsub();
  }, []);

  useEffect(() => {
    const unsub = onValue(ref(db, "elim/participants"), snap => {
      setParticipants(snap.val() || {});
    });
    return () => unsub();
  }, []);

  useEffect(() => {
    const unsub = onValue(ref(db, "elim/rounds"), snap => {
      if (snap.val()) {
        setRoundData(snap.val());
        setAdminRounds(snap.val());
      }
    });
    return () => unsub();
  }, []);

  useEffect(() => {
    if (!Object.keys(participants).length) return;
    (async () => {
      const map = {};
      await Promise.all(Object.keys(participants).map(async uid => {
        map[uid] = (await fbGet(`elim/predictions/${uid}`)) || {};
      }));
      setAllPreds(map);
    })();
  }, [participants, results]);

  // ── Register ──
  async function register() {
    const name = nameInput.trim();
    if (!name || name.length < 2) return setNameError("Ingresá tu nombre (mínimo 2 letras)");
    setNameError("");
    const existing = Object.entries(participants).find(([, n]) => n.toLowerCase() === name.toLowerCase());
    let uid;
    if (existing) {
      uid = existing[0];
    } else {
      uid = "u_" + Date.now() + "_" + Math.random().toString(36).slice(2, 6);
      await fbSet(`elim/participants/${uid}`, name);
    }
    const u = { id: uid, name: existing ? existing[1] : name };
    setUser(u);
    const pr = await fbGet(`elim/predictions/${uid}`);
    setMyPreds(pr || {});
    try { localStorage.setItem("prode26elim:me", JSON.stringify(u)); } catch {}
    setTab("prode");
  }

  // ── Save predictions ──
  async function savePreds() {
    if (!user || isLocked) return;
    const rMatches = currentRound?.matches || [];
    const filled = rMatches.filter(m => myPreds[m.id]?.h !== "" && myPreds[m.id]?.h != null && myPreds[m.id]?.a !== "" && myPreds[m.id]?.a != null).length;
    if (filled < rMatches.length) {
      setSaveMsg(`⚠️ Faltan ${rMatches.length - filled} partido(s) por completar en esta ronda.`);
      setTimeout(() => setSaveMsg(""), 4000);
      return;
    }
    setSaving(true);
    await fbSet(`elim/predictions/${user.id}`, myPreds);
    setAllPreds(p => ({ ...p, [user.id]: myPreds }));
    setSaving(false);
    setSaveMsg("✅ ¡Predicciones guardadas!");
    setTimeout(() => setSaveMsg(""), 3000);
  }

  // ── Ranking ──
  function getRanking() {
    const sc = {};
    Object.entries(participants).forEach(([uid, name]) => {
      sc[uid] = { name, pts: 0, exact: 0, winner: 0, wrong: 0, played: 0 };
    });
    Object.entries(results).forEach(([mid, real]) => {
      if (real?.h == null || real?.h === "") return;
      Object.keys(participants).forEach(uid => {
        const pred = (allPreds[uid] || {})[mid];
        if (!pred) return;
        const pts = calcPoints(pred, real);
        if (pts === null) return;
        sc[uid].played++;
        sc[uid].pts += pts;
        if (pts === 2) sc[uid].exact++;
        else if (pts === 1) sc[uid].winner++;
        else sc[uid].wrong++;
      });
    });
    return Object.entries(sc).map(([uid, s]) => ({ uid, ...s }))
      .sort((a, b) => b.pts - a.pts);
  }

  const rankingRaw = getRanking();
  let pos = 1;
  const ranking = rankingRaw.map((p, i, arr) => {
    if (i > 0 && p.pts < arr[i - 1].pts) pos++;
    return { ...p, pos };
  });
  const totalPlayed = Object.values(results).filter(r => r?.h != null && r?.h !== "").length;

  const getFlag = (team) => FLAGS[team] || "🏳️";
  const fmtTs = (iso) => { try { return new Date(iso).toLocaleString("es-AR", { day: "2-digit", month: "2-digit", hour: "2-digit", minute: "2-digit" }); } catch { return ""; } };

  // ─── LOADING ──────────────────────────────────────────────────────────────
  if (loading) return (
    <div style={{ minHeight: "100vh", background: T.bg, display: "flex", flexDirection: "column", alignItems: "center", justifyContent: "center", gap: 14 }}>
      <span style={{ fontSize: "3rem", animation: "spin 1s linear infinite", display: "block" }}>⚽</span>
      <span style={{ color: T.muted, letterSpacing: 3, fontSize: ".82rem", textTransform: "uppercase" }}>Cargando...</span>
      <style>{`@keyframes spin{to{transform:rotate(360deg)}}`}</style>
    </div>
  );

  // ─── RENDER ───────────────────────────────────────────────────────────────
  return (
    <div style={{ minHeight: "100vh", background: T.bg, color: T.text, fontFamily: "'Segoe UI',system-ui,sans-serif", overflowX: "hidden" }}>
      <style>{`
        *{box-sizing:border-box;margin:0;padding:0}
        input[type=number]::-webkit-inner-spin-button,input[type=number]::-webkit-outer-spin-button{-webkit-appearance:none}
        input[type=number]{-moz-appearance:textfield}
        @keyframes spin{to{transform:rotate(360deg)}}
        @keyframes fadeUp{from{opacity:0;transform:translateY(10px)}to{opacity:1;transform:translateY(0)}}
        @keyframes pulse{0%,100%{transform:scale(1)}50%{transform:scale(1.05)}}
        @keyframes shimmer{0%{background-position:-200% center}100%{background-position:200% center}}
        .hov:hover{opacity:.82!important;transition:opacity .15s}
        .mrow:hover{background:rgba(255,255,255,.015)!important}
        input:focus{outline:none;border-color:rgba(245,200,66,.4)!important;box-shadow:0 0 0 2px rgba(245,200,66,.08)!important}
        ::-webkit-scrollbar{width:4px;height:4px}
        ::-webkit-scrollbar-track{background:transparent}
        ::-webkit-scrollbar-thumb{background:#1a2a3a;border-radius:2px}
      `}</style>

      {/* ── HEADER ── */}
      <header style={{ background: "linear-gradient(160deg,#020508,#061220,#020508)", padding: "28px 20px 20px", textAlign: "center", borderBottom: `1px solid ${T.border}`, position: "relative", overflow: "hidden" }}>
        <div style={{ position: "absolute", inset: 0, background: "radial-gradient(ellipse 70% 50% at 50% 0%,rgba(0,212,160,.08),transparent)", pointerEvents: "none" }} />
        <div style={{ position: "absolute", inset: 0, background: "radial-gradient(ellipse 40% 30% at 80% 80%,rgba(245,200,66,.04),transparent)", pointerEvents: "none" }} />

        {/* Trophy animation */}
        <div style={{ fontSize: "3.2rem", filter: "drop-shadow(0 0 24px rgba(245,200,66,.5))", animation: "pulse 3s ease-in-out infinite", marginBottom: 6 }}>🏆</div>

        <div style={{ fontSize: ".65rem", letterSpacing: 8, textTransform: "uppercase", color: T.muted, marginBottom: 4 }}>Copa Mundial FIFA 2026</div>
        <div style={{
          fontWeight: 900, fontSize: "clamp(2rem,8vw,4rem)", letterSpacing: 4, lineHeight: .95,
          background: "linear-gradient(135deg,#00D4A0 0%,#F5C842 50%,#FF6B35 100%)",
          backgroundSize: "200% auto",
          WebkitBackgroundClip: "text", WebkitTextFillColor: "transparent",
          animation: "shimmer 4s linear infinite",
          marginBottom: 4,
        }}>FASE ELIMINATORIA</div>
        <div style={{ fontSize: ".7rem", color: "#1a3a4a", letterSpacing: 2, marginBottom: 12 }}>28 Jun – 19 Jul · Dieciseisavos → Final</div>

        {/* Round pills */}
        <div style={{ display: "flex", gap: 5, justifyContent: "center", flexWrap: "wrap", marginBottom: 12 }}>
          {ROUNDS.map(r => (
            <span key={r.id} style={{
              background: activeRound === r.id ? `${RC[r.id]}20` : "rgba(255,255,255,.03)",
              border: `1px solid ${activeRound === r.id ? RC[r.id] : T.border}`,
              borderRadius: 20, padding: "3px 10px", fontSize: ".65rem",
              color: activeRound === r.id ? RC[r.id] : T.muted,
              cursor: "pointer", fontWeight: 700, letterSpacing: 1,
              transition: "all .2s",
            }} onClick={() => setActiveRound(r.id)}>
              {r.emoji} {r.label}
            </span>
          ))}
        </div>

        {/* Countdown */}
        {!isLocked && countdown && (
          <div style={{ display: "inline-flex", gap: 6, background: `${RC[activeRound]}10`, border: `1px solid ${RC[activeRound]}30`, borderRadius: 12, padding: "8px 14px", marginBottom: 8 }}>
            <span style={{ fontSize: ".6rem", color: RC[activeRound], letterSpacing: 2, textTransform: "uppercase", marginRight: 4, alignSelf: "center" }}>⏱ Cierre:</span>
            {[["d", "días"], ["h", "hs"], ["m", "min"], ["s", "seg"]].map(([k, label]) => (
              <div key={k} style={{ textAlign: "center", minWidth: 30 }}>
                <div style={{ fontWeight: 900, fontSize: "1.1rem", color: RC[activeRound], lineHeight: 1 }}>{String(countdown[k]).padStart(2, "0")}</div>
                <div style={{ fontSize: ".5rem", color: T.muted, letterSpacing: 1 }}>{label}</div>
              </div>
            ))}
          </div>
        )}
        {isLocked && (
          <div style={{ display: "inline-block", background: "rgba(230,57,70,.1)", border: "1px solid rgba(230,57,70,.3)", borderRadius: 10, padding: "5px 14px", fontSize: ".75rem", color: T.red, fontWeight: 700, marginBottom: 8 }}>
            🔒 Predicciones cerradas para {currentRound?.label}
          </div>
        )}

        {user && (
          <div style={{ marginTop: 10, display: "flex", alignItems: "center", justifyContent: "center", gap: 7 }}>
            <span style={{ background: "rgba(245,200,66,.1)", border: "1px solid rgba(245,200,66,.2)", borderRadius: 20, padding: "4px 13px", fontSize: ".8rem", color: T.gold, fontWeight: 700 }}>
              ⚽ {user.name}
            </span>
            <button className="hov" onClick={() => { setUser(null); setMyPreds({}); try { localStorage.removeItem("prode26elim:me"); } catch {} }}
              style={{ background: "rgba(255,255,255,.04)", color: T.muted, border: `1px solid ${T.border}`, borderRadius: 8, padding: "4px 11px", fontSize: ".75rem", cursor: "pointer", fontWeight: 600 }}>
              Salir
            </button>
          </div>
        )}
      </header>

      {/* ── LEGEND ── */}
      <div style={{ background: "#060D18", borderBottom: `1px solid ${T.border}`, padding: "7px 14px", display: "flex", gap: 7, flexWrap: "wrap", justifyContent: "center", alignItems: "center" }}>
        <span style={{ fontSize: ".65rem", color: T.muted, letterSpacing: 2, textTransform: "uppercase", fontWeight: 700, marginRight: 2 }}>Puntos:</span>
        {[[2, T.gold, "Exacto"], [1, T.green, "Ganador"], [0, T.red, "Error"]].map(([pts, bg, label]) => (
          <div key={pts} style={{ display: "flex", alignItems: "center", gap: 5, background: "rgba(255,255,255,.02)", border: `1px solid ${T.border}`, borderRadius: 8, padding: "3px 9px" }}>
            <span style={{ background: bg, color: "#05080F", borderRadius: 5, padding: "1px 7px", fontWeight: 900, fontSize: ".78rem", minWidth: 20, textAlign: "center" }}>{pts}</span>
            <span style={{ fontSize: ".68rem", color: "#4a6a8a" }}>{label}</span>
          </div>
        ))}
      </div>

      {/* ── TABS ── */}
      <div style={{ display: "flex", background: "#060D18", borderBottom: `2px solid ${T.border}`, overflowX: "auto" }}>
        {[["ranking", "🏆", "Ranking"], ["prode", "⚽", "Mi Prode"], ["resultados", "📊", "Resultados"], ["admin", "🔐", "Admin"]].map(([id, ic, lb]) => (
          <button key={id} onClick={() => setTab(id)}
            style={{
              padding: "11px 17px", border: "none", background: "none", cursor: "pointer", whiteSpace: "nowrap",
              fontWeight: 700, fontSize: ".8rem", letterSpacing: 2, textTransform: "uppercase",
              color: tab === id ? T.gold : "#2a4a5a",
              borderBottom: tab === id ? `2px solid ${T.gold}` : "2px solid transparent", marginBottom: -2,
              transition: "color .15s",
            }}>
            {ic} {lb}
          </button>
        ))}
      </div>

      {/* ── CONTENT ── */}
      <div style={{ maxWidth: 900, margin: "0 auto", padding: "16px 13px 60px" }}>

        {/* ── RANKING ── */}
        {tab === "ranking" && (
          <div style={{ animation: "fadeUp .25s ease" }}>
            <div style={{ marginBottom: 14 }}>
              <div style={{ fontWeight: 900, fontSize: "1.3rem", letterSpacing: 2, marginBottom: 3 }}>RANKING GENERAL</div>
              <div style={{ fontSize: ".7rem", color: T.muted }}>{totalPlayed} partido(s) jugado(s) · {Object.keys(participants).length} participante(s)</div>
            </div>

            {ranking.length === 0 ? (
              <div style={{ textAlign: "center", padding: "50px 20px", color: T.muted }}>
                <div style={{ fontSize: "3rem", marginBottom: 10 }}>🏟️</div>
                <div style={{ fontSize: ".9rem", marginBottom: 16 }}>Nadie se registró todavía</div>
                <button className="hov" onClick={() => setTab("prode")}
                  style={{ background: `linear-gradient(135deg,${T.teal},#00a07a)`, color: "#05080F", border: "none", borderRadius: 9, padding: "10px 22px", fontWeight: 800, fontSize: ".9rem", cursor: "pointer" }}>
                  Registrarme ⚽
                </button>
              </div>
            ) : (
              <div style={{ background: T.card, border: `1px solid ${T.border}`, borderRadius: 14, overflow: "hidden" }}>
                {ranking.map((p, i) => (
                  <div key={p.uid} className="mrow" style={{
                    display: "flex", alignItems: "center", gap: 11, padding: "13px 17px",
                    borderBottom: `1px solid ${T.border}`,
                    background: p.pos === 1 ? "rgba(245,200,66,.05)" : p.pos === 2 ? "rgba(168,184,200,.03)" : p.pos === 3 ? "rgba(205,127,50,.03)" : "transparent",
                  }}>
                    <div style={{
                      width: 32, height: 32, borderRadius: 9, flexShrink: 0, display: "flex", alignItems: "center", justifyContent: "center", fontWeight: 900, fontSize: ".9rem",
                      background: p.pos === 1 ? "linear-gradient(135deg,#F5C842,#E5A820)" : p.pos === 2 ? "linear-gradient(135deg,#A8B8C8,#788898)" : p.pos === 3 ? "linear-gradient(135deg,#CD7F32,#9A5420)" : "rgba(255,255,255,.05)",
                      color: p.pos <= 3 ? "#05080F" : "#2a4a5a",
                    }}>{p.pos}</div>
                    <div style={{ flex: 1, minWidth: 0 }}>
                      <div style={{ fontWeight: 700, fontSize: ".96rem", overflow: "hidden", textOverflow: "ellipsis", whiteSpace: "nowrap" }}>
                        {p.name}
                        {p.uid === user?.id && <span style={{ fontSize: ".65rem", color: T.gold, marginLeft: 5 }}>(vos)</span>}
                      </div>
                      <div style={{ display: "flex", gap: 9, marginTop: 3, flexWrap: "wrap" }}>
                        <span style={{ fontSize: ".67rem", color: T.muted }}>{p.played} jugados</span>
                        <span style={{ fontSize: ".67rem", color: T.gold }}>✦ {p.exact} exactos</span>
                        <span style={{ fontSize: ".67rem", color: T.green }}>✓ {p.winner} ganadores</span>
                        <span style={{ fontSize: ".67rem", color: T.red }}>✗ {p.wrong} errores</span>
                      </div>
                    </div>
                    <div style={{ textAlign: "right", flexShrink: 0 }}>
                      <div style={{ fontWeight: 900, fontSize: "1.5rem", color: T.gold, lineHeight: 1 }}>{p.pts}</div>
                      <div style={{ fontSize: ".6rem", color: T.muted, letterSpacing: 1 }}>pts</div>
                    </div>
                  </div>
                ))}
              </div>
            )}

            {/* Últimos resultados */}
            {totalPlayed > 0 && (
              <div style={{ marginTop: 20 }}>
                <div style={{ fontWeight: 700, fontSize: ".72rem", letterSpacing: 2, color: T.muted, textTransform: "uppercase", marginBottom: 9 }}>Últimos resultados</div>
                <div style={{ background: T.card, border: `1px solid ${T.border}`, borderRadius: 12, overflow: "hidden" }}>
                  {ALL_MATCHES.filter(m => results[m.id]?.h != null && results[m.id]?.h !== "").slice(-6).reverse().map(m => (
                    <div key={m.id} className="mrow" style={{ display: "flex", alignItems: "center", gap: 8, padding: "8px 15px", borderBottom: `1px solid rgba(255,255,255,.02)`, fontSize: ".8rem" }}>
                      <span style={{ flex: 1, textAlign: "right", color: "#5a7a9a" }}>{getFlag(m.h)} {m.h}</span>
                      <span style={{ fontWeight: 900, color: T.gold, minWidth: 46, textAlign: "center", letterSpacing: 2 }}>{results[m.id].h}–{results[m.id].a}</span>
                      <span style={{ flex: 1, color: "#5a7a9a" }}>{getFlag(m.a)} {m.a}</span>
                    </div>
                  ))}
                </div>
              </div>
            )}
          </div>
        )}

        {/* ── MI PRODE ── */}
        {tab === "prode" && (
          <div style={{ animation: "fadeUp .25s ease" }}>
            {!user ? (
              <div style={{ maxWidth: 360, margin: "36px auto", textAlign: "center" }}>
                <div style={{ fontSize: "2.8rem", marginBottom: 12 }}>⚽</div>
                <div style={{ fontWeight: 900, fontSize: "1.2rem", letterSpacing: 2, marginBottom: 6 }}>INGRESÁ AL PRODE</div>
                <div style={{ color: T.muted, fontSize: ".83rem", marginBottom: 20, lineHeight: 1.6 }}>
                  Escribí tu nombre para registrarte.<br />Si ya participaste, usá el mismo nombre exacto.
                </div>
                <input value={nameInput} onChange={e => { setNameInput(e.target.value); setNameError(""); }}
                  onKeyDown={e => e.key === "Enter" && register()}
                  placeholder="Tu nombre..." autoFocus
                  style={{ background: "rgba(255,255,255,.06)", border: `1px solid ${T.border}`, borderRadius: 9, color: "white", padding: "11px 14px", fontSize: "1rem", width: "100%", marginBottom: 6, textAlign: "center", fontFamily: "inherit" }} />
                {nameError && <div style={{ color: T.red, fontSize: ".78rem", marginBottom: 7 }}>{nameError}</div>}
                <button className="hov" onClick={register}
                  style={{ background: `linear-gradient(135deg,${T.gold},#E5A820)`, color: "#05080F", border: "none", borderRadius: 9, padding: "11px", fontWeight: 800, fontSize: ".9rem", cursor: "pointer", width: "100%", letterSpacing: 1 }}>
                  Entrar 🚀
                </button>
              </div>
            ) : (
              <div>
                {/* Round selector */}
                <div style={{ display: "flex", gap: 6, marginBottom: 14, overflowX: "auto", paddingBottom: 4 }}>
                  {ROUNDS.map(r => {
                    const rLocked = new Date() >= r.deadline;
                    const rMatches = r.matches;
                    const rFilled = rMatches.filter(m => myPreds[m.id]?.h !== "" && myPreds[m.id]?.h != null && myPreds[m.id]?.a !== "" && myPreds[m.id]?.a != null).length;
                    return (
                      <button key={r.id} onClick={() => setActiveRound(r.id)}
                        style={{
                          background: activeRound === r.id ? `${RC[r.id]}18` : "rgba(255,255,255,.03)",
                          border: `1px solid ${activeRound === r.id ? RC[r.id] : T.border}`,
                          borderRadius: 10, padding: "7px 13px", cursor: "pointer", flexShrink: 0,
                          color: activeRound === r.id ? RC[r.id] : T.muted, fontWeight: 700, fontSize: ".75rem",
                          transition: "all .2s",
                        }}>
                        {r.emoji} {r.label}
                        {!rLocked && <span style={{ fontSize: ".6rem", marginLeft: 4, opacity: .7 }}>{rFilled}/{rMatches.length}</span>}
                        {rLocked && <span style={{ fontSize: ".6rem", marginLeft: 4 }}>🔒</span>}
                      </button>
                    );
                  })}
                </div>

                {/* Header */}
                <div style={{ display: "flex", alignItems: "center", justifyContent: "space-between", marginBottom: 11, flexWrap: "wrap", gap: 8 }}>
                  <div>
                    <div style={{ fontWeight: 900, fontSize: "1.1rem", letterSpacing: 2, color: RC[activeRound] }}>{currentRound?.emoji} {currentRound?.label?.toUpperCase()}</div>
                    <div style={{ fontSize: ".7rem", color: T.muted, marginTop: 2 }}>
                      {isLocked ? "🔒 Predicciones cerradas para esta ronda" : `Cargá tus predicciones antes del cierre`}
                    </div>
                  </div>
                  {!isLocked && (
                    <button className="hov" onClick={savePreds} disabled={saving}
                      style={{ background: `linear-gradient(135deg,${T.green},#1a9e3f)`, color: "white", border: "none", borderRadius: 9, padding: "9px 18px", fontWeight: 800, fontSize: ".85rem", cursor: "pointer", opacity: saving ? .5 : 1, display: "flex", alignItems: "center", gap: 5 }}>
                      {saving ? "Guardando..." : "💾 Guardar"}
                    </button>
                  )}
                </div>

                {saveMsg && (
                  <div style={{ background: saveMsg.startsWith("✅") ? "rgba(45,198,83,.1)" : "rgba(230,57,70,.1)", border: `1px solid ${saveMsg.startsWith("✅") ? "rgba(45,198,83,.25)" : "rgba(230,57,70,.25)"}`, borderRadius: 9, padding: "10px 14px", fontSize: ".83rem", color: saveMsg.startsWith("✅") ? T.green : T.red, marginBottom: 12, fontWeight: 600 }}>
                    {saveMsg}
                  </div>
                )}

                {/* Matches */}
                <div style={{ background: T.card, border: `1px solid ${T.border}`, borderRadius: 14, overflow: "hidden" }}>
                  <div style={{ background: "rgba(0,0,0,.3)", borderBottom: `1px solid ${T.border}`, padding: "9px 14px", display: "flex", alignItems: "center", gap: 8 }}>
                    <span style={{ width: 8, height: 8, borderRadius: "50%", background: RC[activeRound], flexShrink: 0 }} />
                    <span style={{ fontWeight: 900, fontSize: ".85rem", letterSpacing: 3, color: RC[activeRound] }}>{currentRound?.label?.toUpperCase()}</span>
                    <span style={{ fontSize: ".65rem", color: T.muted }}>· {currentRound?.matches.length} partidos</span>
                  </div>
                  {currentRound?.matches.map(m => {
                    const pred = myPreds[m.id] || { h: "", a: "" };
                    const real = results[m.id];
                    const played = real?.h != null && real?.h !== "";
                    const locked = isLocked || played;
                    const pts = played ? calcPoints(pred, real) : null;
                    const isTBD = m.h.startsWith("Gan.") || m.h === "TBD" || m.a.startsWith("Gan.") || m.a === "TBD";
                    return (
                      <div key={m.id} className="mrow" style={{ padding: "8px 13px", borderBottom: `1px solid rgba(255,255,255,.025)` }}>
                        <div style={{ fontSize: ".58rem", color: "#1a3040", textAlign: "center", marginBottom: 4, letterSpacing: 1 }}>{m.date} · {m.venue}</div>
                        <div style={{ display: "grid", gridTemplateColumns: "1fr auto 1fr", alignItems: "center", gap: 4 }}>
                          <div style={{ display: "flex", alignItems: "center", gap: 5, justifyContent: "flex-end", flexDirection: "row-reverse" }}>
                            <span style={{ fontSize: ".8rem", fontWeight: 600, color: "#7a9ab8", overflow: "hidden", textOverflow: "ellipsis", whiteSpace: "nowrap", maxWidth: 90 }}>{m.h}</span>
                            {!isTBD && <span style={{ fontSize: "1rem", flexShrink: 0 }}>{getFlag(m.h)}</span>}
                          </div>
                          <div style={{ display: "flex", alignItems: "center", gap: 3, justifyContent: "center", flexShrink: 0 }}>
                            {pts !== null && (
                              <div style={{ width: 20, height: 20, borderRadius: 5, display: "flex", alignItems: "center", justifyContent: "center", fontWeight: 900, fontSize: ".72rem", marginRight: 2, background: pts === 2 ? T.gold : pts === 1 ? T.green : T.red, color: pts > 0 ? "#05080F" : "white" }}>{pts}</div>
                            )}
                            {isTBD ? (
                              <span style={{ fontSize: ".7rem", color: T.muted, padding: "0 8px" }}>Por definir</span>
                            ) : (
                              <>
                                <input type="number" min="0" max="99" placeholder="–" value={pred.h} disabled={locked}
                                  onChange={e => !locked && setMyPreds(p => ({ ...p, [m.id]: { ...pred, h: e.target.value } }))}
                                  style={{ width: 34, height: 34, background: locked ? "rgba(255,255,255,.02)" : "#0D1E30", border: `1.5px solid ${locked ? "rgba(255,255,255,.04)" : "rgba(255,255,255,.12)"}`, borderRadius: 7, color: locked ? "#1a3a4a" : T.gold, fontSize: "1rem", fontWeight: 900, textAlign: "center", cursor: locked ? "default" : "pointer" }} />
                                <span style={{ color: "#0a1a25", fontWeight: 900, fontSize: ".9rem", margin: "0 1px" }}>:</span>
                                <input type="number" min="0" max="99" placeholder="–" value={pred.a} disabled={locked}
                                  onChange={e => !locked && setMyPreds(p => ({ ...p, [m.id]: { ...pred, a: e.target.value } }))}
                                  style={{ width: 34, height: 34, background: locked ? "rgba(255,255,255,.02)" : "#0D1E30", border: `1.5px solid ${locked ? "rgba(255,255,255,.04)" : "rgba(255,255,255,.12)"}`, borderRadius: 7, color: locked ? "#1a3a4a" : T.gold, fontSize: "1rem", fontWeight: 900, textAlign: "center", cursor: locked ? "default" : "pointer" }} />
                              </>
                            )}
                            {played && <span style={{ fontSize: ".66rem", color: "#1a4030", marginLeft: 3, fontWeight: 700 }}>({real.h}–{real.a})</span>}
                          </div>
                          <div style={{ display: "flex", alignItems: "center", gap: 5 }}>
                            {!isTBD && <span style={{ fontSize: "1rem", flexShrink: 0 }}>{getFlag(m.a)}</span>}
                            <span style={{ fontSize: ".8rem", fontWeight: 600, color: "#7a9ab8", overflow: "hidden", textOverflow: "ellipsis", whiteSpace: "nowrap", maxWidth: 90 }}>{m.a}</span>
                          </div>
                        </div>
                      </div>
                    );
                  })}
                </div>

                {!isLocked && (
                  <button className="hov" onClick={savePreds} disabled={saving}
                    style={{ background: `linear-gradient(135deg,${T.green},#1a9e3f)`, color: "white", border: "none", borderRadius: 9, padding: "12px", fontWeight: 800, fontSize: ".9rem", cursor: "pointer", width: "100%", marginTop: 10, opacity: saving ? .5 : 1 }}>
                    {saving ? "Guardando..." : "💾 Guardar predicciones"}
                  </button>
                )}
              </div>
            )}
          </div>
        )}

        {/* ── RESULTADOS ── */}
        {tab === "resultados" && (
          <div style={{ animation: "fadeUp .25s ease" }}>
            <div style={{ fontWeight: 900, fontSize: "1.15rem", letterSpacing: 2, marginBottom: 4 }}>RESULTADOS</div>
            <div style={{ fontSize: ".7rem", color: T.muted, marginBottom: 14 }}>{totalPlayed} partido(s) jugado(s)</div>

            {ROUNDS.map(r => {
              const hasResults = r.matches.some(m => results[m.id]?.h != null && results[m.id]?.h !== "");
              return (
                <div key={r.id} style={{ background: T.card, border: `1px solid ${T.border}`, borderRadius: 14, overflow: "hidden", marginBottom: 12, opacity: hasResults ? 1 : .5 }}>
                  <div style={{ background: "rgba(0,0,0,.3)", borderBottom: `1px solid ${T.border}`, padding: "9px 14px", display: "flex", alignItems: "center", gap: 8 }}>
                    <span style={{ fontWeight: 900, fontSize: ".85rem", letterSpacing: 2, color: RC[r.id] }}>{r.emoji} {r.label.toUpperCase()}</span>
                    {!hasResults && <span style={{ fontSize: ".65rem", color: T.muted }}>· Sin resultados aún</span>}
                  </div>
                  {r.matches.map(m => {
                    const res2 = results[m.id];
                    const played = res2?.h != null && res2?.h !== "";
                    return (
                      <div key={m.id} className="mrow" style={{ padding: "7px 13px", borderBottom: `1px solid rgba(255,255,255,.02)` }}>
                        <div style={{ fontSize: ".58rem", color: "#1a3040", textAlign: "center", marginBottom: 3 }}>{m.date} · {m.venue}</div>
                        <div style={{ display: "grid", gridTemplateColumns: "1fr auto 1fr", alignItems: "center", gap: 4 }}>
                          <div style={{ display: "flex", alignItems: "center", gap: 5, justifyContent: "flex-end", flexDirection: "row-reverse" }}>
                            <span style={{ fontSize: ".8rem", fontWeight: played && parseInt(res2.h) > parseInt(res2.a) ? 700 : 400, color: played && parseInt(res2.h) > parseInt(res2.a) ? "#ddeeff" : "#5a7a9a", overflow: "hidden", textOverflow: "ellipsis", whiteSpace: "nowrap", maxWidth: 90 }}>{m.h}</span>
                            {!m.h.startsWith("Gan.") && m.h !== "TBD" && <span>{getFlag(m.h)}</span>}
                          </div>
                          <div style={{ textAlign: "center", minWidth: 52 }}>
                            {played
                              ? <span style={{ fontWeight: 900, fontSize: "1.1rem", color: T.gold, letterSpacing: 3 }}>{res2.h}–{res2.a}</span>
                              : <span style={{ color: "#0a1a25", fontSize: ".75rem" }}>vs</span>}
                          </div>
                          <div style={{ display: "flex", alignItems: "center", gap: 5 }}>
                            {!m.a.startsWith("Gan.") && m.a !== "TBD" && <span>{getFlag(m.a)}</span>}
                            <span style={{ fontSize: ".8rem", fontWeight: played && parseInt(res2.a) > parseInt(res2.h) ? 700 : 400, color: played && parseInt(res2.a) > parseInt(res2.h) ? "#ddeeff" : "#5a7a9a", overflow: "hidden", textOverflow: "ellipsis", whiteSpace: "nowrap", maxWidth: 90 }}>{m.a}</span>
                          </div>
                        </div>
                      </div>
                    );
                  })}
                </div>
              );
            })}
          </div>
        )}

        {/* ── ADMIN ── */}
        {tab === "admin" && (
          <div style={{ animation: "fadeUp .25s ease" }}>
            {!isAdmin ? (
              <div style={{ maxWidth: 290, margin: "36px auto", textAlign: "center" }}>
                <div style={{ fontSize: "2.2rem", marginBottom: 11 }}>🔐</div>
                <div style={{ fontWeight: 900, fontSize: "1.1rem", letterSpacing: 2, marginBottom: 16 }}>PANEL ADMIN</div>
                <input type="password" placeholder="PIN" value={adminPin}
                  onChange={e => { setAdminPin(e.target.value); setAdminPinErr(""); }}
                  onKeyDown={e => { if (e.key === "Enter") { if (adminPin === ADMIN_PIN) setIsAdmin(true); else setAdminPinErr("PIN incorrecto"); } }}
                  style={{ background: "rgba(255,255,255,.06)", border: `1px solid ${T.border}`, borderRadius: 9, color: "white", padding: "10px 14px", fontSize: "1.1rem", width: "100%", textAlign: "center", letterSpacing: 6, marginBottom: 6, fontFamily: "inherit" }} />
                {adminPinErr && <div style={{ color: T.red, fontSize: ".78rem", marginBottom: 6 }}>{adminPinErr}</div>}
                <button className="hov" onClick={() => { if (adminPin === ADMIN_PIN) setIsAdmin(true); else setAdminPinErr("PIN incorrecto"); }}
                  style={{ background: `linear-gradient(135deg,${T.gold},#E5A820)`, color: "#05080F", border: "none", borderRadius: 9, padding: "10px", fontWeight: 800, fontSize: ".88rem", cursor: "pointer", width: "100%", letterSpacing: 1 }}>
                  Ingresar
                </button>
              </div>
            ) : (
              <div>
                <div style={{ display: "flex", alignItems: "center", justifyContent: "space-between", marginBottom: 14, flexWrap: "wrap", gap: 8 }}>
                  <div>
                    <div style={{ fontWeight: 900, fontSize: "1.1rem", letterSpacing: 2, color: T.teal }}>✅ MODO ADMIN</div>
                    <div style={{ fontSize: ".7rem", color: T.muted, marginTop: 2 }}>Cargá resultados y editá equipos de rondas futuras</div>
                  </div>
                  <div style={{ display: "flex", gap: 7 }}>
                    <button className="hov" onClick={async () => { await fbSet("elim/results", adminResults); alert("✅ Resultados guardados."); }}
                      style={{ background: `linear-gradient(135deg,${T.green},#1a9e3f)`, color: "white", border: "none", borderRadius: 9, padding: "8px 16px", fontWeight: 800, fontSize: ".85rem", cursor: "pointer" }}>
                      💾 Guardar
                    </button>
                    <button className="hov" onClick={() => setIsAdmin(false)}
                      style={{ background: "rgba(255,255,255,.04)", color: T.muted, border: `1px solid ${T.border}`, borderRadius: 9, padding: "8px 14px", fontWeight: 600, fontSize: ".85rem", cursor: "pointer" }}>
                      Salir
                    </button>
                  </div>
                </div>

                {/* Participantes */}
                <div style={{ background: T.card, border: `1px solid ${T.border}`, borderRadius: 12, marginBottom: 13, overflow: "hidden" }}>
                  <div style={{ borderBottom: `1px solid ${T.border}`, padding: "8px 14px", fontSize: ".7rem", color: T.muted, letterSpacing: 2, textTransform: "uppercase", fontWeight: 700 }}>
                    👥 Participantes ({Object.keys(participants).length})
                  </div>
                  <div style={{ padding: "10px 14px", display: "flex", gap: 6, flexWrap: "wrap" }}>
                    {Object.values(participants).length === 0
                      ? <span style={{ color: "#0a1a25", fontSize: ".8rem" }}>Ninguno todavía</span>
                      : Object.values(participants).map((n, i) => (
                        <span key={i} style={{ background: "rgba(255,255,255,.03)", border: `1px solid ${T.border}`, borderRadius: 20, padding: "3px 10px", fontSize: ".78rem", color: "#4a6a8a" }}>{n}</span>
                      ))}
                  </div>
                </div>

                {/* Results per round */}
                {ROUNDS.map(r => (
                  <div key={r.id} style={{ background: T.card, border: `1px solid ${T.border}`, borderRadius: 13, overflow: "hidden", marginBottom: 12 }}>
                    <div style={{ background: "rgba(0,0,0,.3)", borderBottom: `1px solid ${T.border}`, padding: "9px 14px" }}>
                      <span style={{ fontWeight: 900, fontSize: ".85rem", letterSpacing: 2, color: RC[r.id] }}>{r.emoji} {r.label.toUpperCase()}</span>
                    </div>
                    {r.matches.map(m => {
                      const res2 = adminResults[m.id] || { h: "", a: "" };
                      const isTBD = m.h.startsWith("Gan.") || m.h === "TBD" || m.a.startsWith("Gan.") || m.a === "TBD";
                      return (
                        <div key={m.id} className="mrow" style={{ padding: "7px 13px", borderBottom: `1px solid rgba(255,255,255,.02)` }}>
                          <div style={{ fontSize: ".58rem", color: "#1a3040", textAlign: "center", marginBottom: 3 }}>{m.date} · {m.venue}</div>
                          <div style={{ display: "grid", gridTemplateColumns: "1fr auto 1fr", alignItems: "center", gap: 4 }}>
                            <div style={{ display: "flex", alignItems: "center", gap: 5, justifyContent: "flex-end", flexDirection: "row-reverse" }}>
                              <span style={{ fontSize: ".78rem", color: "#7a9ab8", overflow: "hidden", textOverflow: "ellipsis", whiteSpace: "nowrap", maxWidth: 90 }}>{m.h}</span>
                              {!isTBD && <span>{getFlag(m.h)}</span>}
                            </div>
                            <div style={{ display: "flex", alignItems: "center", gap: 3, justifyContent: "center", flexShrink: 0 }}>
                              <input type="number" min="0" max="99" placeholder="–" value={res2.h}
                                onChange={e => setAdminResults(p => ({ ...p, [m.id]: { ...res2, h: e.target.value } }))}
                                style={{ width: 34, height: 34, background: "#060F18", border: `1.5px solid rgba(0,212,160,.2)`, borderRadius: 7, color: T.teal, fontSize: "1rem", fontWeight: 900, textAlign: "center" }} />
                              <span style={{ color: "#0a1a25", fontWeight: 900, fontSize: ".9rem", margin: "0 1px" }}>:</span>
                              <input type="number" min="0" max="99" placeholder="–" value={res2.a}
                                onChange={e => setAdminResults(p => ({ ...p, [m.id]: { ...res2, a: e.target.value } }))}
                                style={{ width: 34, height: 34, background: "#060F18", border: `1.5px solid rgba(0,212,160,.2)`, borderRadius: 7, color: T.teal, fontSize: "1rem", fontWeight: 900, textAlign: "center" }} />
                            </div>
                            <div style={{ display: "flex", alignItems: "center", gap: 5 }}>
                              {!isTBD && <span>{getFlag(m.a)}</span>}
                              <span style={{ fontSize: ".78rem", color: "#7a9ab8", overflow: "hidden", textOverflow: "ellipsis", whiteSpace: "nowrap", maxWidth: 90 }}>{m.a}</span>
                            </div>
                          </div>
                        </div>
                      );
                    })}
                  </div>
                ))}

                <button className="hov" onClick={async () => { await fbSet("elim/results", adminResults); alert("✅ Resultados guardados."); }}
                  style={{ background: `linear-gradient(135deg,${T.green},#1a9e3f)`, color: "white", border: "none", borderRadius: 9, padding: "12px", fontWeight: 800, fontSize: ".9rem", cursor: "pointer", width: "100%", marginTop: 4 }}>
                  💾 Guardar todos los resultados
                </button>
              </div>
            )}
          </div>
        )}

      </div>
    </div>
  );
}
