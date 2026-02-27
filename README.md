<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>2026 MLB Monte Carlo Season Simulator</title>
<meta name="description" content="Simulating every game of the 2026 MLB season thousands of times using Monte Carlo probability, the Log5 formula, and real preseason projections." />
<link href="https://fonts.googleapis.com/css2?family=Archivo+Black&family=DM+Sans:wght@400;500;700&display=swap" rel="stylesheet" />
<style>
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
  body { background: #060d14; color: #c0d0e0; font-family: 'DM Sans', sans-serif; min-height: 100vh; }
  ::selection { background: #1a4a7a; }

  input[type=range] { -webkit-appearance: none; background: #1a2a3a; border-radius: 4px; height: 4px; outline: none; }
  input[type=range]::-webkit-slider-thumb { -webkit-appearance: none; width: 14px; height: 14px; border-radius: 50%; background: #c0d0e0; cursor: pointer; border: 2px solid #060d14; }
  select { font-family: 'DM Sans', sans-serif; }

  @keyframes fadeUp { from { opacity: 0; transform: translateY(12px); } to { opacity: 1; transform: translateY(0); } }
  @keyframes shimmer { 0% { background-position: -200% 0; } 100% { background-position: 200% 0; } }

  .header { background: linear-gradient(180deg, #0a1520 0%, #060d14 100%); border-bottom: 1px solid #1a2a3a; padding: 28px 20px 22px; text-align: center; }
  .header-sub { font-size: 10px; letter-spacing: 0.2em; text-transform: uppercase; color: #4a6a8a; margin-bottom: 8px; font-weight: 700; }
  .header h1 { font-family: 'Archivo Black', sans-serif; font-size: clamp(22px, 5vw, 36px); color: #e8ecf1; letter-spacing: -0.02em; line-height: 1.1; margin-bottom: 6px; }
  .header p { color: #5a7a9a; font-size: 13px; max-width: 500px; margin: 0 auto; }

  .container { max-width: 900px; margin: 0 auto; padding: 20px 16px; }

  .controls { display: flex; flex-wrap: wrap; gap: 10px; margin-bottom: 20px; align-items: center; }
  .controls label { font-size: 11px; color: #4a6a8a; font-weight: 700; text-transform: uppercase; letter-spacing: 0.06em; }
  .controls select { background: #0a1520; border: 1px solid #1a2a3a; color: #c0d0e0; padding: 6px 10px; border-radius: 8px; font-size: 13px; font-weight: 700; cursor: pointer; }

  .btn-run { background: linear-gradient(135deg, #1a5a3a 0%, #0f4c2a 100%); border: 1px solid #2a5a4a; color: #4ade80; padding: 8px 20px; border-radius: 8px; font-size: 13px; font-weight: 700; cursor: pointer; letter-spacing: 0.03em; font-family: 'DM Sans', sans-serif; }
  .btn-run:disabled { background: linear-gradient(90deg, #1a2a3a, #2a3a4a, #1a2a3a); background-size: 200% 100%; animation: shimmer 1.5s linear infinite; cursor: wait; }
  .btn-adjust { background: transparent; border: 1px solid #1a2a3a; color: #6b8299; padding: 8px 14px; border-radius: 8px; font-size: 12px; cursor: pointer; font-weight: 600; font-family: 'DM Sans', sans-serif; }
  .btn-adjust.active { background: #1a2a3a; }

  .view-btn { border: 1px solid #1a2a3a; padding: 6px 12px; border-radius: 6px; font-size: 11px; cursor: pointer; font-weight: 700; text-transform: uppercase; letter-spacing: 0.06em; font-family: 'DM Sans', sans-serif; }
  .view-btn.inactive { background: transparent; color: #4a6a8a; }
  .view-btn.active { background: #1a2a3a; color: #e8ecf1; }

  .panel { background: #0a1520; border-radius: 12px; border: 1px solid #1a2a3a; overflow: hidden; margin-bottom: 16px; }
  .panel-header { padding: 12px 14px; background: linear-gradient(135deg, #0f1d2d 0%, #132636 100%); border-bottom: 1px solid #1a2a3a; }
  .panel-title { font-family: 'Archivo Black', sans-serif; font-size: 14px; color: #c0d0e0; letter-spacing: 0.08em; text-transform: uppercase; }

  .expandable-btn { width: 100%; padding: 14px 18px; background: linear-gradient(135deg, #0f1d2d 0%, #132636 100%); border: none; color: #c0d0e0; font-family: 'Archivo Black', sans-serif; font-size: 13px; letter-spacing: 0.06em; text-transform: uppercase; cursor: pointer; display: flex; justify-content: space-between; align-items: center; }

  .col-headers { display: grid; grid-template-columns: 2fr 1.2fr 0.7fr 1fr 0.8fr 0.8fr 0.8fr; padding: 8px 14px; font-size: 10px; color: #4a6a8a; text-transform: uppercase; letter-spacing: 0.08em; font-weight: 700; border-bottom: 1px solid #1a2a3a; }

  .team-row { display: grid; grid-template-columns: 2fr 1.2fr 0.7fr 1fr 0.8fr 0.8fr 0.8fr; align-items: center; padding: 10px 14px; border-bottom: 1px solid #1a2a3a; font-size: 13px; transition: background 0.2s; cursor: default; }
  .team-row:hover { background: #0d1926; }

  .rank-badge { width: 22px; height: 22px; border-radius: 50%; display: inline-flex; align-items: center; justify-content: center; font-size: 9px; font-weight: 700; color: #fff; flex-shrink: 0; }
  .team-abbr { font-weight: 700; color: #e8ecf1; letter-spacing: -0.01em; }
  .team-name { color: #6b8299; margin-left: 8px; font-size: 11px; }
  .avg-wins { text-align: center; font-weight: 700; color: #e8ecf1; font-family: 'Archivo Black', sans-serif; font-size: 15px; }
  .ci-cell { text-align: center; color: #6b8299; font-size: 11px; }
  .ci-sigma { opacity: 0.6; }

  .playoff-badge { padding: 3px 8px; border-radius: 6px; font-weight: 700; font-size: 12px; }
  .playoff-high { background: #0f4c2a; color: #4ade80; }
  .playoff-mid { background: #2a3a1e; color: #a3c45a; }
  .playoff-low { background: #1a1a24; color: #6b8299; }

  .div-pct { text-align: center; color: #8ba3bd; font-weight: 500; }
  .ws-hot { font-weight: 700; color: #f0c040; font-size: 13px; }
  .ws-cold { font-weight: 700; color: #4a5a6a; font-size: 12px; }

  .tooltip-wrap { position: relative; display: inline-flex; align-items: center; gap: 3px; cursor: help; }
  .tooltip-icon { font-size: 8px; opacity: 0.6; line-height: 1; }
  .tooltip-box { position: absolute; top: calc(100% + 8px); left: 50%; transform: translateX(-50%); background: #162232; border: 1px solid #2a4a6a; border-radius: 8px; padding: 10px 13px; font-size: 11px; font-weight: 400; color: #a0b8d0; width: 220px; z-index: 100; text-transform: none; letter-spacing: 0; line-height: 1.5; box-shadow: 0 8px 24px rgba(0,0,0,0.5); pointer-events: none; display: none; }
  .tooltip-wrap:hover .tooltip-box { display: block; }
  .tooltip-arrow { position: absolute; top: -5px; left: 50%; transform: translateX(-50%) rotate(45deg); width: 8px; height: 8px; background: #162232; border-left: 1px solid #2a4a6a; border-top: 1px solid #2a4a6a; }

  .adj-panel { background: #0a1520; border-radius: 12px; border: 1px solid #1a2a3a; padding: 16px; margin-bottom: 20px; animation: fadeUp 0.3s ease; }
  .adj-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); gap: 2px 24px; }
  .adj-row { display: flex; align-items: center; gap: 8px; padding: 4px 0; }
  .adj-abbr { width: 34px; font-weight: 700; font-size: 11px; color: #8ba3bd; }
  .adj-val { width: 36px; text-align: right; font-weight: 700; font-size: 12px; }
  .adj-pos { color: #4ade80; }
  .adj-neg { color: #f87171; }
  .adj-zero { color: #4a5a6a; }

  .stat-card { display: flex; gap: 12px; padding: 10px 14px; background: #060d14; border-radius: 8px; border: 1px solid #1a2a3a; margin-bottom: 8px; }
  .stat-icon { font-size: 20px; flex-shrink: 0; }
  .stat-name { font-weight: 700; color: #e8ecf1; font-size: 13px; margin-bottom: 3px; }
  .stat-desc { color: #6b8299; font-size: 12px; line-height: 1.55; }

  .source-card { background: #060d14; border-radius: 8px; padding: 10px 12px; border: 1px solid #1a2a3a; }
  .source-name { font-weight: 700; color: #a0b8d0; font-size: 12px; }
  .source-desc { font-size: 11px; color: #4a6a8a; margin-top: 2px; }
  .source-url { font-size: 10px; color: #2a5a8a; margin-top: 4px; }

  .formula-box { background: #060d14; border-radius: 8px; padding: 12px 16px; font-family: monospace; font-size: 13px; color: #4ade80; margin: 8px 0; }
  .callout { background: #060d14; border-radius: 8px; padding: 12px 16px; font-size: 12px; color: #6b8299; border-left: 3px solid #f0c040; }
  .callout strong { color: #f0c040; }
  .disclaimer { background: #060d14; border-radius: 8px; padding: 10px 14px; border-left: 3px solid #4a6a8a; font-size: 11px; color: #4a6a8a; }
  .disclaimer strong { color: #6b8299; }

  .section-label { font-size: 10px; color: #4a6a8a; font-weight: 700; text-transform: uppercase; letter-spacing: 0.1em; margin-bottom: 12px; }

  .footer { text-align: center; padding: 32px 0 20px; font-size: 11px; color: #2a3a4a; line-height: 1.8; }

  .dist-bar { display: flex; align-items: flex-end; height: 40px; gap: 1px; opacity: 0.85; }
  .dist-tick { width: 3px; border-radius: 1px 1px 0 0; transition: height 0.3s ease; min-height: 1px; }

  .hidden { display: none; }
</style>
</head>
<body>

<!-- Header -->
<div class="header">
  <div class="header-sub" id="simCountLabel">Monte Carlo Simulation • 5,000 Iterations</div>
  <h1>2026 MLB Season Simulator</h1>
  <p>Simulating every game of the 2026 season using probability theory, the Log5 formula, and real preseason projections.</p>
</div>

<div class="container">

  <!-- Controls -->
  <div class="controls">
    <div style="display:flex;align-items:center;gap:8px">
      <label>Simulations:</label>
      <select id="simSelect">
        <option value="1000">1,000</option>
        <option value="5000" selected>5,000</option>
        <option value="10000">10,000</option>
        <option value="25000">25,000</option>
      </select>
    </div>
    <button class="btn-run" id="runBtn" onclick="handleRun()">▶ Run Simulation</button>
    <button class="btn-adjust" id="adjBtn" onclick="toggleAdjust()">⚙ Adjust Teams</button>
    <div style="margin-left:auto;display:flex;gap:4px">
      <button class="view-btn active" id="viewDiv" onclick="setView('division')">By Division</button>
      <button class="view-btn inactive" id="viewLeague" onclick="setView('league')">By League</button>
    </div>
  </div>

  <!-- Adjustment Panel (hidden by default) -->
  <div id="adjPanel" class="adj-panel hidden"></div>

  <!-- Stat Guide -->
  <div class="panel" style="margin-bottom:16px">
    <button class="expandable-btn" onclick="toggleSection('statGuide', this)">
      <span>📖 How to Read This Table — Stat Guide</span>
      <span style="font-size:18px;transition:transform 0.2s">▾</span>
    </button>
    <div id="statGuide" class="hidden" style="padding:16px 20px">
      <p style="color:#6b8299;font-size:12px;margin-bottom:14px;line-height:1.6">
        Every number on this page comes from running the full 162-game MLB season + playoffs <strong style="color:#c0d0e0">thousands of times</strong> using random simulation. Here's what each column means:
      </p>
      <div class="stat-card"><span class="stat-icon">📊</span><div><div class="stat-name">Win Distribution</div><div class="stat-desc">A histogram (bar chart) of every simulated win total. Each tiny bar is a possible outcome — taller bars happened more often. A wide spread means lots of uncertainty; a tight cluster means we're confident.</div></div></div>
      <div class="stat-card"><span class="stat-icon">🎯</span><div><div class="stat-name">Avg W (Average Wins)</div><div class="stat-desc">The expected value — add up all simulated win totals and divide by the number of simulations. This is the best single-number prediction for the team's season.</div></div></div>
      <div class="stat-card"><span class="stat-icon">📏</span><div><div class="stat-name">90% CI (Confidence Interval)</div><div class="stat-desc">The range of wins that covers 90% of all simulated outcomes (5th to 95th percentile). Example: 74–96 means in 90% of sims, the team won between 74 and 96 games. σ (sigma) is the standard deviation — how "spread out" results are.</div></div></div>
      <div class="stat-card"><span class="stat-icon">🏟</span><div><div class="stat-name">Playoff %</div><div class="stat-desc">How often this team made the playoffs across all simulations. MLB has 12 playoff teams (6 per league): 3 division winners + 3 wild cards.</div></div></div>
      <div class="stat-card"><span class="stat-icon">🏆</span><div><div class="stat-name">Div % (Division Winner)</div><div class="stat-desc">How often this team finished 1st in their division. This is always equal to or lower than Playoff %, because you can make the playoffs as a wild card without winning your division.</div></div></div>
      <div class="stat-card"><span class="stat-icon">💍</span><div><div class="stat-name">WS % (World Series Champion)</div><div class="stat-desc">How often this team won it all — the Wild Card Round, Division Series, Championship Series, AND World Series. Even great teams usually top out at 15–25% because the playoffs involve short series with lots of randomness.</div></div></div>
    </div>
  </div>

  <!-- Math Explainer -->
  <div class="panel" style="margin-bottom:20px">
    <button class="expandable-btn" onclick="toggleSection('mathExplainer', this)">
      <span>📐 The Math Behind This Simulator</span>
      <span style="font-size:18px;transition:transform 0.2s">▾</span>
    </button>
    <div id="mathExplainer" class="hidden" style="padding:18px 20px;color:#8ba3bd;font-size:13px;line-height:1.8">
      <div style="margin-bottom:16px">
        <span style="color:#e8ecf1;font-weight:700;font-size:14px">1. Win Probability (Bernoulli Trials)</span>
        <p style="margin:6px 0">Each of the 162 games is modeled as a <em>Bernoulli trial</em> — a random event with two outcomes (win or loss). If a team has a projected win rate of 0.556 (90 wins / 162 games), then each game is like flipping a weighted coin that lands "win" 55.6% of the time.</p>
      </div>
      <div style="margin-bottom:16px">
        <span style="color:#e8ecf1;font-weight:700;font-size:14px">2. Log5 Formula (Playoff Matchups)</span>
        <p style="margin:6px 0">To determine who wins each playoff game, we use Bill James' <em>Log5</em> formula:</p>
        <div class="formula-box">P(A beats B) = (pA − pA×pB) / (pA + pB − 2×pA×pB)</div>
        <p style="margin:6px 0">This elegantly adjusts each team's strength relative to their opponent. A .600 team vs a .400 team wins ~69.2% of the time, not 60%.</p>
      </div>
      <div style="margin-bottom:16px">
        <span style="color:#e8ecf1;font-weight:700;font-size:14px">3. Monte Carlo Method</span>
        <p style="margin:6px 0">We repeat the entire season + playoffs <strong>thousands</strong> of times, using random numbers each time. By counting outcomes across all simulations, we get probability estimates — like "the Dodgers won the World Series in 2,340 out of 10,000 sims = 23.4% WS odds." More simulations = more precise estimates.</p>
      </div>
      <div style="margin-bottom:16px">
        <span style="color:#e8ecf1;font-weight:700;font-size:14px">4. Standard Deviation & Confidence Intervals</span>
        <p style="margin:6px 0">The <em>standard deviation</em> (σ) tells us how much outcomes vary across simulations. The <em>90% confidence interval</em> (5th to 95th percentile) shows the range where 90% of simulated seasons land. A team projected for 85 wins with a CI of 74–96 means there's huge uncertainty — that's the beauty of simulation over a single number.</p>
      </div>
      <div class="callout">
        <strong>Why this matters:</strong> A single projected win total (like "Dodgers: 100 wins") hides all the uncertainty. Monte Carlo simulation reveals the <em>full probability landscape</em> — showing that even a 100-win projection has realistic paths to 88 wins or 110 wins.
      </div>
    </div>
  </div>

  <!-- Data Sources -->
  <div class="panel" style="padding:18px 20px;margin-bottom:20px">
    <div style="font-family:'Archivo Black',sans-serif;font-size:12px;color:#c0d0e0;letter-spacing:0.08em;text-transform:uppercase;margin-bottom:12px">📋 Data Sources & Methodology</div>
    <div style="color:#6b8299;font-size:12px;line-height:1.7">
      <p style="margin-bottom:10px"><strong style="color:#8ba3bd">Projected win totals</strong> are a composite average of multiple respected projection systems, sourced in February 2026:</p>
      <div style="display:grid;grid-template-columns:repeat(auto-fill,minmax(200px,1fr));gap:8px;margin-bottom:12px">
        <div class="source-card"><div class="source-name">FanGraphs Depth Charts</div><div class="source-desc">Combines Steamer + ZiPS projections with roster depth</div><div class="source-url">fangraphs.com</div></div>
        <div class="source-card"><div class="source-name">Baseball Prospectus PECOTA</div><div class="source-desc">Player-level projection system using comparable players</div><div class="source-url">baseballprospectus.com</div></div>
        <div class="source-card"><div class="source-name">Betting Market Consensus</div><div class="source-desc">Oddsmakers' over/under win totals (BetMGM, FanDuel, etc.)</div></div>
      </div>
      <p style="margin-bottom:8px"><strong style="color:#8ba3bd">Simulation method:</strong> Each team's projected wins are converted to a win probability (wins ÷ 162). Every game in the 162-game season is then simulated as a Bernoulli trial (weighted coin flip). Playoff matchups use the Log5 formula to calculate head-to-head probabilities. The entire season + playoffs are repeated thousands of times.</p>
      <div class="disclaimer"><strong>Disclaimer:</strong> This is an independent mathematical simulation for educational purposes. It is not affiliated with MLB, FanGraphs, or any sportsbook. Projections are estimates and should not be used for gambling decisions.</div>
    </div>
  </div>

  <!-- Results -->
  <div id="results"></div>

  <!-- Footer -->
  <div class="footer">
    <div style="color:#4a5a6a;margin-bottom:4px">Built with Monte Carlo simulation, Log5, and Bernoulli probability theory</div>
    <div>Data: FanGraphs Depth Charts (Steamer + ZiPS) • Baseball Prospectus PECOTA • Betting market consensus</div>
    <div style="margin-top:4px">Hover over any column header ⓘ for an explanation of that stat</div>
    <div style="margin-top:8px;color:#3a4a5a">© 2026 • MLB Monte Carlo Season Simulator • For educational purposes only</div>
  </div>
</div>

<script>
/*
 * ============================================================
 *  MLB 2026 MONTE CARLO SEASON SIMULATOR — Pure JavaScript
 * ============================================================
 *  Math concepts used:
 *    1. Win Probability  – Each game = Bernoulli trial (weighted coin flip)
 *    2. Log5 Formula     – Bill James' head-to-head win probability
 *    3. Monte Carlo Sim  – Repeat season thousands of times
 *    4. Expected Value   – Average projected wins across all sims
 *    5. Standard Deviation – Spread / uncertainty of outcomes
 *    6. Confidence Intervals – Range where 90% of outcomes fall
 * ============================================================
 */

// ── TEAM DATA ──
// Source: composite of FanGraphs Depth Charts, Baseball Prospectus PECOTA,
// and betting market consensus as of February 2026.
const TEAMS_DATA = {
  "AL East": [
    { name: "New York Yankees",    abbr: "NYY", wins: 91, color: "#003087" },
    { name: "Toronto Blue Jays",   abbr: "TOR", wins: 87, color: "#134A8E" },
    { name: "Baltimore Orioles",   abbr: "BAL", wins: 86, color: "#DF4601" },
    { name: "Tampa Bay Rays",      abbr: "TBR", wins: 86, color: "#092C5C" },
    { name: "Boston Red Sox",      abbr: "BOS", wins: 82, color: "#BD3039" },
  ],
  "AL Central": [
    { name: "Detroit Tigers",      abbr: "DET", wins: 84, color: "#0C2340" },
    { name: "Minnesota Twins",     abbr: "MIN", wins: 80, color: "#002B5C" },
    { name: "Cleveland Guardians", abbr: "CLE", wins: 78, color: "#00385D" },
    { name: "Kansas City Royals",  abbr: "KCR", wins: 76, color: "#004687" },
    { name: "Chicago White Sox",   abbr: "CHW", wins: 66, color: "#27251F" },
  ],
  "AL West": [
    { name: "Seattle Mariners",    abbr: "SEA", wins: 88, color: "#0C2C56" },
    { name: "Houston Astros",      abbr: "HOU", wins: 87, color: "#002D62" },
    { name: "Texas Rangers",       abbr: "TEX", wins: 83, color: "#003278" },
    { name: "Los Angeles Angels",  abbr: "LAA", wins: 76, color: "#BA0021" },
    { name: "Athletics",           abbr: "OAK", wins: 67, color: "#003831" },
  ],
  "NL East": [
    { name: "Atlanta Braves",        abbr: "ATL", wins: 92, color: "#CE1141" },
    { name: "New York Mets",         abbr: "NYM", wins: 87, color: "#002D72" },
    { name: "Philadelphia Phillies", abbr: "PHI", wins: 86, color: "#E81828" },
    { name: "Miami Marlins",         abbr: "MIA", wins: 76, color: "#00A3E0" },
    { name: "Washington Nationals",  abbr: "WSN", wins: 67, color: "#AB0003" },
  ],
  "NL Central": [
    { name: "Chicago Cubs",        abbr: "CHC", wins: 85, color: "#0E3386" },
    { name: "Milwaukee Brewers",   abbr: "MIL", wins: 82, color: "#FFC52F" },
    { name: "St. Louis Cardinals",  abbr: "STL", wins: 81, color: "#C41E3A" },
    { name: "Cincinnati Reds",     abbr: "CIN", wins: 79, color: "#C6011F" },
    { name: "Pittsburgh Pirates",  abbr: "PIT", wins: 74, color: "#27251F" },
  ],
  "NL West": [
    { name: "Los Angeles Dodgers",   abbr: "LAD", wins: 100, color: "#005A9C" },
    { name: "Arizona Diamondbacks",  abbr: "ARI", wins: 85, color: "#A71930" },
    { name: "San Francisco Giants",  abbr: "SFG", wins: 80, color: "#FD5A1E" },
    { name: "San Diego Padres",      abbr: "SDP", wins: 79, color: "#2F241D" },
    { name: "Colorado Rockies",      abbr: "COL", wins: 63, color: "#333366" },
  ],
};

const AL_DIVS = ["AL East", "AL Central", "AL West"];
const NL_DIVS = ["NL East", "NL Central", "NL West"];

const COLUMN_TOOLTIPS = {
  team: "Each team ranked by projected wins within their division. The colored circle shows their rank.",
  dist: "A mini bar chart showing how many wins the team got across all simulations. Wider spread = more uncertainty. Hover individual bars for exact percentages.",
  avgW: "The average (expected value) of wins across all simulations. This is the single best prediction for how many games this team will win.",
  ci: "The 90% Confidence Interval — in 90% of simulations, the team's wins fell within this range. σ (sigma) is the standard deviation, measuring how spread out the results are.",
  playoff: "% of simulations where this team made the postseason (3 division winners + 3 wild cards per league = 6 playoff teams per league).",
  div: "% of simulations where this team finished 1st in their division. Always ≤ Playoff % since wild card teams also make playoffs.",
  ws: "% of simulations where this team won the World Series. Even dominant teams rarely exceed ~25% because playoff series are short and unpredictable.",
};

// ── STATE ──
let adjustments = {};
let currentResults = null;
let viewMode = "division";

// ── MATH HELPERS ──

/** Log5: probability Team A (win% pA) beats Team B (win% pB). By Bill James. */
function log5(pA, pB) {
  return (pA - pA * pB) / (pA + pB - 2 * pA * pB);
}

/** Standard deviation: measures spread of results. Higher = more uncertainty. */
function standardDeviation(arr) {
  const n = arr.length;
  if (n === 0) return 0;
  const mean = arr.reduce((a, b) => a + b, 0) / n;
  const sqDiffs = arr.map(v => (v - mean) ** 2);
  return Math.sqrt(sqDiffs.reduce((a, b) => a + b, 0) / n);
}

/** Percentile: value below which p% of data falls. Used for confidence intervals. */
function percentile(arr, p) {
  const sorted = [...arr].sort((a, b) => a - b);
  const idx = (p / 100) * (sorted.length - 1);
  const lo = Math.floor(idx);
  const hi = Math.ceil(idx);
  if (lo === hi) return sorted[lo];
  return sorted[lo] + (sorted[hi] - sorted[lo]) * (idx - lo);
}

// ── SIMULATION ENGINE ──

function runSimulation(teamsData, numSims, adjustments) {
  const allTeams = [];
  const divisionMap = {};

  for (const [div, teams] of Object.entries(teamsData)) {
    for (const t of teams) {
      const adj = adjustments[t.abbr] || 0;
      const adjWins = Math.max(30, Math.min(130, t.wins + adj));
      const wp = adjWins / 162;
      allTeams.push({ ...t, adjWins, wp, division: div });
      if (!divisionMap[div]) divisionMap[div] = [];
      divisionMap[div].push(t.abbr);
    }
  }

  const results = {};
  for (const t of allTeams) {
    results[t.abbr] = { ...t, simWins: [], divisionWins: 0, playoffApps: 0, wsWins: 0 };
  }

  for (let sim = 0; sim < numSims; sim++) {
    const seasonWins = {};
    for (const t of allTeams) {
      let wins = 0;
      for (let g = 0; g < 162; g++) {
        if (Math.random() < t.wp) wins++;
      }
      seasonWins[t.abbr] = wins;
      results[t.abbr].simWins.push(wins);
    }

    const leagueResults = { AL: [], NL: [] };
    const divWinners = { AL: [], NL: [] };

    for (const [div, abbrs] of Object.entries(divisionMap)) {
      const league = div.startsWith("AL") ? "AL" : "NL";
      const sorted = [...abbrs].sort((a, b) => seasonWins[b] - seasonWins[a] + (Math.random() - 0.5) * 0.001);
      results[sorted[0]].divisionWins++;
      divWinners[league].push({ abbr: sorted[0], wins: seasonWins[sorted[0]] });
      for (const a of sorted) {
        leagueResults[league].push({ abbr: a, wins: seasonWins[a], divWinner: a === sorted[0] });
      }
    }

    for (const league of ["AL", "NL"]) {
      const dw = divWinners[league].sort((a, b) => b.wins - a.wins);
      const nonDW = leagueResults[league].filter(t => !t.divWinner).sort((a, b) => b.wins - a.wins + (Math.random() - 0.5) * 0.001);
      const wcTeams = nonDW.slice(0, 3);
      const playoffTeams = [...dw, ...wcTeams];
      for (const pt of playoffTeams) results[pt.abbr].playoffApps++;

      const seeds = playoffTeams.sort((a, b) => b.wins - a.wins);
      const wc1 = simSeries(seeds[2], seeds[5], allTeams, 3);
      const wc2 = simSeries(seeds[3], seeds[4], allTeams, 3);
      const ds1 = simSeries(seeds[0], wc2, allTeams, 5);
      const ds2 = simSeries(seeds[1], wc1, allTeams, 5);
      const cs = simSeries(ds1, ds2, allTeams, 7);
      leagueResults[league].pennant = cs;
    }

    const ws = simSeries(leagueResults.AL.pennant, leagueResults.NL.pennant, allTeams, 7);
    results[ws.abbr].wsWins++;
  }
  return results;
}

function simSeries(teamA, teamB, allTeams, numGames) {
  const pA = allTeams.find(t => t.abbr === teamA.abbr)?.wp || 0.5;
  const pB = allTeams.find(t => t.abbr === teamB.abbr)?.wp || 0.5;
  const prob = log5(pA, pB);
  const winsNeeded = Math.ceil(numGames / 2);
  let aW = 0, bW = 0;
  while (aW < winsNeeded && bW < winsNeeded) {
    if (Math.random() < prob) aW++; else bW++;
  }
  return aW >= winsNeeded ? teamA : teamB;
}

// ── RENDERING ──

function makeTooltipHTML(label, key, align) {
  return `<div class="tooltip-wrap" style="text-align:${align||'center'}">
    <span>${label}</span><span class="tooltip-icon">ⓘ</span>
    <div class="tooltip-box"><div class="tooltip-arrow"></div>${COLUMN_TOOLTIPS[key]}</div>
  </div>`;
}

function makeColumnHeaders() {
  return `<div class="col-headers">
    <div>${makeTooltipHTML('Team','team','left')}</div>
    <div style="text-align:center">${makeTooltipHTML('Win Dist.','dist')}</div>
    <div style="text-align:center">${makeTooltipHTML('Avg W','avgW')}</div>
    <div style="text-align:center">${makeTooltipHTML('90% CI','ci')}</div>
    <div style="text-align:center">${makeTooltipHTML('Playoff %','playoff')}</div>
    <div style="text-align:center">${makeTooltipHTML('Div %','div')}</div>
    <div style="text-align:center">${makeTooltipHTML('WS %','ws')}</div>
  </div>`;
}

function makeDistHTML(simWins, color, numSims) {
  if (!simWins || simWins.length === 0) return '';
  const min = Math.min(...simWins), max = Math.max(...simWins);
  const buckets = {};
  for (let w = min; w <= max; w++) buckets[w] = 0;
  for (const w of simWins) buckets[w]++;
  const maxCount = Math.max(...Object.values(buckets));
  const mean = simWins.reduce((a,b) => a+b, 0) / simWins.length;
  let html = '<div class="dist-bar">';
  for (let w = min; w <= max; w++) {
    const c = buckets[w];
    const pct = ((c / maxCount) * 100).toFixed(1);
    const bg = w >= mean ? color : color + '88';
    html += `<div class="dist-tick" title="${w} wins: ${((c/numSims)*100).toFixed(1)}%" style="height:${pct}%;background:${bg}"></div>`;
  }
  html += '</div>';
  return html;
}

function makeTeamRowHTML(team, numSims, rank) {
  const avg = (team.simWins.reduce((a,b) => a+b, 0) / team.simWins.length).toFixed(1);
  const sd = standardDeviation(team.simWins).toFixed(1);
  const p5 = percentile(team.simWins, 5);
  const p95 = percentile(team.simWins, 95);
  const pp = ((team.playoffApps / numSims) * 100).toFixed(1);
  const dp = ((team.divisionWins / numSims) * 100).toFixed(1);
  const wp = ((team.wsWins / numSims) * 100).toFixed(1);

  const ppClass = parseFloat(pp) > 75 ? 'playoff-high' : parseFloat(pp) > 40 ? 'playoff-mid' : 'playoff-low';
  const wsClass = parseFloat(wp) > 5 ? 'ws-hot' : 'ws-cold';

  return `<div class="team-row">
    <div style="display:flex;align-items:center;gap:10px">
      <span class="rank-badge" style="background:${team.color}">${rank}</span>
      <div><span class="team-abbr">${team.abbr}</span><span class="team-name">${team.name}</span></div>
    </div>
    ${makeDistHTML(team.simWins, team.color, numSims)}
    <div class="avg-wins">${avg}</div>
    <div class="ci-cell">${p5}–${p95} <span class="ci-sigma">(σ=${sd})</span></div>
    <div style="text-align:center"><span class="playoff-badge ${ppClass}">${pp}%</span></div>
    <div class="div-pct">${dp}%</div>
    <div style="text-align:center"><span class="${wsClass}">${wp}%</span></div>
  </div>`;
}

function renderResults() {
  if (!currentResults) return;
  const numSims = parseInt(document.getElementById('simSelect').value);
  const container = document.getElementById('results');
  let html = '<div style="animation:fadeUp 0.4s ease">';

  if (viewMode === 'division') {
    html += '<div class="section-label">American League</div>';
    for (const div of AL_DIVS) {
      html += renderDivision(div, numSims);
    }
    html += '<div class="section-label" style="margin-top:24px">National League</div>';
    for (const div of NL_DIVS) {
      html += renderDivision(div, numSims);
    }
  } else {
    const all = Object.values(currentResults).sort((a, b) => {
      const avgA = a.simWins.reduce((s,v) => s+v, 0) / a.simWins.length;
      const avgB = b.simWins.reduce((s,v) => s+v, 0) / b.simWins.length;
      return avgB - avgA;
    });
    html += `<div class="panel"><div class="panel-header"><span class="panel-title">All 30 Teams — Ranked by Projected Wins</span></div>`;
    html += makeColumnHeaders();
    all.forEach((t, i) => { html += makeTeamRowHTML(t, numSims, i+1); });
    html += '</div>';
  }

  html += '</div>';
  container.innerHTML = html;
}

function renderDivision(divName, numSims) {
  const teams = TEAMS_DATA[divName].map(t => currentResults[t.abbr]);
  const sorted = [...teams].sort((a, b) => {
    const avgA = a.simWins.reduce((s,v) => s+v, 0) / a.simWins.length;
    const avgB = b.simWins.reduce((s,v) => s+v, 0) / b.simWins.length;
    return avgB - avgA;
  });
  let html = `<div class="panel"><div class="panel-header"><span class="panel-title">${divName}</span></div>`;
  html += makeColumnHeaders();
  sorted.forEach((t, i) => { html += makeTeamRowHTML(t, numSims, i+1); });
  html += '</div>';
  return html;
}

// ── UI HANDLERS ──

function handleRun() {
  const btn = document.getElementById('runBtn');
  const numSims = parseInt(document.getElementById('simSelect').value);
  btn.disabled = true;
  btn.textContent = '⏳ Simulating...';
  document.getElementById('simCountLabel').textContent = `Monte Carlo Simulation • ${numSims.toLocaleString()} Iterations`;

  setTimeout(() => {
    currentResults = runSimulation(TEAMS_DATA, numSims, adjustments);
    renderResults();
    btn.disabled = false;
    btn.textContent = '▶ Run Simulation';
  }, 50);
}

function toggleAdjust() {
  const panel = document.getElementById('adjPanel');
  const btn = document.getElementById('adjBtn');
  if (panel.classList.contains('hidden')) {
    panel.classList.remove('hidden');
    btn.classList.add('active');
    buildAdjPanel();
  } else {
    panel.classList.add('hidden');
    btn.classList.remove('active');
  }
}

function buildAdjPanel() {
  const panel = document.getElementById('adjPanel');
  let html = '<div style="font-size:11px;color:#4a6a8a;font-weight:700;text-transform:uppercase;letter-spacing:0.06em;margin-bottom:12px">🎛 Win Adjustments — Slide to add/subtract projected wins, then re-run</div>';
  html += '<div class="adj-grid">';
  for (const [div, teams] of Object.entries(TEAMS_DATA)) {
    for (const t of teams) {
      const val = adjustments[t.abbr] || 0;
      const cls = val > 0 ? 'adj-pos' : val < 0 ? 'adj-neg' : 'adj-zero';
      html += `<div class="adj-row">
        <span class="adj-abbr">${t.abbr}</span>
        <input type="range" min="-15" max="15" value="${val}" style="flex:1;accent-color:${t.color};height:4px;cursor:pointer" oninput="updateAdj('${t.abbr}', this.value, this)">
        <span class="adj-val ${cls}" id="adj-${t.abbr}">${val > 0 ? '+'+val : val}</span>
      </div>`;
    }
  }
  html += '</div>';
  html += '<button onclick="resetAdj()" style="margin-top:10px;background:transparent;border:1px solid #1a2a3a;color:#6b8299;padding:5px 12px;border-radius:6px;font-size:11px;cursor:pointer;font-family:DM Sans,sans-serif">Reset All</button>';
  panel.innerHTML = html;
}

function updateAdj(abbr, val, input) {
  val = parseInt(val);
  adjustments[abbr] = val;
  const span = document.getElementById('adj-' + abbr);
  span.textContent = val > 0 ? '+' + val : val;
  span.className = 'adj-val ' + (val > 0 ? 'adj-pos' : val < 0 ? 'adj-neg' : 'adj-zero');
}

function resetAdj() {
  adjustments = {};
  buildAdjPanel();
}

function setView(mode) {
  viewMode = mode;
  document.getElementById('viewDiv').className = 'view-btn ' + (mode === 'division' ? 'active' : 'inactive');
  document.getElementById('viewLeague').className = 'view-btn ' + (mode === 'league' ? 'active' : 'inactive');
  renderResults();
}

function toggleSection(id, btn) {
  const el = document.getElementById(id);
  const arrow = btn.querySelector('span:last-child');
  if (el.classList.contains('hidden')) {
    el.classList.remove('hidden');
    arrow.style.transform = 'rotate(180deg)';
  } else {
    el.classList.add('hidden');
    arrow.style.transform = 'rotate(0)';
  }
}

// ── AUTO-RUN ON LOAD ──
window.addEventListener('DOMContentLoaded', () => { handleRun(); });
</script>
</body>
</html>
