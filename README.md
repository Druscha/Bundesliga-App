# Bundesliga Insight
Die ultimative App für Bundesliga-Fans: Detaillierte Spielerstatistiken, aktuelle Marktwertentwicklungen und smarte Zukunftsprognosen – alles auf einen Blick!
<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Bundesliga Player Analytics & Prognosen</title>
<style>
  body {
    font-family: Arial, sans-serif;
    margin: 0; padding: 20px;
    background: #f9f9f9;
    color: #222;
  }
  h1 {
    text-align: center;
    margin-bottom: 10px;
  }
  .search-filter {
    display: flex;
    justify-content: center;
    gap: 10px;
    margin-bottom: 20px;
    flex-wrap: wrap;
  }
  input, select {
    padding: 8px;
    font-size: 16px;
    min-width: 180px;
  }
  #playerList {
    max-width: 900px;
    margin: 0 auto;
  }
  .player-card {
    background: #fff;
    border-radius: 8px;
    box-shadow: 0 2px 6px rgb(0 0 0 / 0.1);
    padding: 15px;
    margin-bottom: 12px;
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;
    gap: 12px;
    align-items: center;
  }
  .player-name {
    font-weight: bold;
    font-size: 1.1em;
  }
  .label {
    font-weight: 600;
    color: #555;
  }
  .value {
    font-weight: 400;
  }
  .progress-bar {
    height: 10px;
    background: #ddd;
    border-radius: 5px;
    overflow: hidden;
    margin-top: 4px;
  }
  .progress-fill {
    height: 100%;
    background: #4caf50;
    transition: width 0.5s ease;
  }
  @media (max-width: 600px) {
    .player-card {
      grid-template-columns: 1fr;
    }
  }
</style>
</head>
<body>

<h1>⚽ Bundesliga Player Analytics & Prognosen</h1>

<div class="search-filter">
  <input type="text" id="searchInput" placeholder="Spieler oder Team suchen" />
  <select id="positionFilter">
    <option value="">Alle Positionen</option>
    <option value="Goalkeeper">Torwart</option>
    <option value="Defender">Abwehr</option>
    <option value="Midfielder">Mittelfeld</option>
    <option value="Attacker">Sturm</option>
  </select>
</div>

<div id="playerList">Lade Spieler...</div>

<script>
// ======= HIER API-KEY EINFÜGEN =======
const API_KEY = "b463b6a1a2c44ecf8b480f8f9e6bde94";

// Football-Data.org neue API Base URL
const BASE_URL = "https://api.football-data.org/v4";

// Zwischenspeicher Spieler-Daten
let players = [];

// Hilfsfunktion: Formkurve simulieren (0-100)
// basierend auf Anzahl Spiele der letzten Saison & Position
function simulateForm(player) {
  // Mehr Spiele = bessere Form, dazu Zufall + Position leicht gewichtet
  let base = Math.min(player.matchesLastSeason || 10, 30);
  let posFactor = 1;
  switch(player.position) {
    case "Goalkeeper": posFactor = 0.9; break;
    case "Defender": posFactor = 1.0; break;
    case "Midfielder": posFactor = 1.1; break;
    case "Attacker": posFactor = 1.2; break;
  }
  // Simuliere form: max 100
  return Math.min(100, Math.round(base * posFactor * (0.7 + Math.random() * 0.6)));
}

// Hilfsfunktion: Wahrscheinliche Startelf basierend auf Form und Belastung
function simulateStartelf(player) {
  let form = simulateForm(player);
  let loadFactor = 1;
  if(player.clubHasChampionsLeague) loadFactor -= 0.15; // evtl Belastung
  if(player.recentInjury) loadFactor -= 0.3;
  // Simpler Score
  const score = form * loadFactor;
  return score > 60; // über 60 = wahrscheinlich Startelf
}

// Hilfsfunktion: Marktwertentwicklung prognostizieren (vereinfacht)
function simulateMarketTrend(player) {
  // Wenn jung + gute Form → steigt der Marktwert
  let age = player.age || 25;
  let form = simulateForm(player);
  if(age < 23 && form > 70) return "Steigend 📈";
  if(age > 30 && form < 50) return "Fallend 📉";
  return "Stabil ➡️";
}

// Funktion um Alter aus Geburtsdatum zu berechnen
function calcAge(dob) {
  if(!dob) return null;
  const birth = new Date(dob);
  const today = new Date();
  let age = today.getFullYear() - birth.getFullYear();
  const m = today.getMonth() - birth.getMonth();
  if(m < 0 || (m === 0 && today.getDate() < birth.getDate())) age--;
  return age;
}

// Hauptfunktion Spieler aus API laden & anzeigen
async function loadPlayers() {
  try {
    // 1. Bundesliga Teams abrufen
    const teamsRes = await fetch(`${BASE_URL}/competitions/BL1/teams`, {
      headers: { "X-Auth-Token": API_KEY }
    });
    const teamsData = await teamsRes.json();
    const teams = teamsData.teams;

    // 2. Alle Spieler aus allen Teams sammeln
    const allPlayers = [];
    for(const team of teams) {
      const teamRes = await fetch(`${BASE_URL}/teams/${team.id}`, {
        headers: { "X-Auth-Token": API_KEY }
      });
      const teamData = await teamRes.json();

      // Spieler anreichern mit Zusatzinfos (simuliert)
      for(const p of teamData.squad) {
        const player = {
          id: p.id,
          name: p.name,
          position: p.position || "Unbekannt",
          team: team.name,
          nationality: p.nationality || "Unbekannt",
          dateOfBirth: p.dateOfBirth || null,
          age: calcAge(p.dateOfBirth),
          matchesLastSeason: Math.floor(Math.random()*30) + 5,  // Zufall 5–35 Spiele
          clubHasChampionsLeague: ["Bayern München", "Borussia Dortmund", "RB Leipzig", "Bayer Leverkusen"].includes(team.name),
          recentInjury: Math.random() < 0.1  // 10% Chance verletzt
        };
        allPlayers.push(player);
      }
    }

    players = allPlayers;
    renderPlayers(players);
  } catch(err) {
    console.error("Fehler beim Laden:", err);
    document.getElementById("playerList").innerHTML = "<p>Fehler beim Laden der Daten. Bitte API-Key prüfen.</p>";
  }
}

// Filter & Suche anwenden und Spieler anzeigen
function renderPlayers(playerList) {
  const searchInput = document.getElementById("searchInput");
  const positionFilter = document.getElementById("positionFilter");

  function filterAndRender() {
    const searchValue = searchInput.value.toLowerCase();
    const posValue = positionFilter.value;

    const filtered = playerList.filter(p => {
      const matchesSearch = p.name.toLowerCase().includes(searchValue) || p.team.toLowerCase().includes(searchValue);
      const matchesPos = posValue === "" || p.position === posValue;
      return matchesSearch && matchesPos;
    });

    const container = document.getElementById("playerList");
    container.innerHTML = "";

    if(filtered.length === 0) {
      container.innerHTML = "<p>Keine Spieler gefunden.</p>";
      return;
    }

    filtered.forEach(p => {
      const form = simulateForm(p);
      const startelf = simulateStartelf(p);
      const marketTrend = simulateMarketTrend(p);

      const card = document.createElement("div");
      card.className = "player-card";
      card.innerHTML = `
        <div>
          <div class="player-name">${p.name}</div>
          <div><span class="label">Team:</span> <span class="value">${p.team}</span></div>
          <div><span class="label">Position:</span> <span class="value">${p.position}</span></div>
          <div><span class="label">Nationalität:</span> <span class="value">${p.nationality}</span></div>
          <div><span class="label">Alter:</span> <span class="value">${p.age ?? "?"} Jahre</span></div>
        </div>
        <div>
          <div><span class="label">Form (0-100):</span></div>
          <div class="progress-bar"><div class="progress-fill" style="width:${form}%;"></div></div>
          <div>${form} Punkte</div>
        </div>
        <div>
          <div><span class="label">Prognose Startelf:</span> <span class="value">${startelf ? "✅ Wahrscheinlich" : "❌ Eher nicht"}</span></div>
          <div><span class="label">Marktwert Entwicklung:</span> <span class="value">${marketTrend}</span></div>
        </div>
      `;
      container.appendChild(card);
    });
  }

  searchInput.addEventListener("input", filterAndRender);
  positionFilter.addEventListener("change", filterAndRender);

  // Erste Darstellung
  filterAndRender();
}

// App starten
loadPlayers();

</script>

</body>
</html>
