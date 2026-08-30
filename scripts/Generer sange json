#!/usr/bin/env node
"use strict";

/**
 * Genererer data/sange.json ud fra data/sange.csv.
 *
 * Køres automatisk af GitHub Actions (.github/workflows/generer-sange-json.yml)
 * hver gang data/sange.csv ændrer sig. Kan også køres manuelt: `node scripts/generer-sange-json.js`.
 *
 * Vigtigt princip: data/sange.csv er kilden. data/sange.json er et GENERERET
 * resultat — ret aldrig i den i hånden (se data/LÆS-MIG.md). Sange der allerede
 * er slået op tidligere, slås IKKE op igen (så pipelinen forbliver hurtig og
 * skånsom mod Apples API, uanset hvor mange sange der er i puljen). Kun nye
 * rækker i CSV'en bliver slået op. Titel/kunstner/sværhed/kategori/noter
 * synkroniseres altid fra CSV'en; alt det der kræver et opslag (lyd, årstal,
 * genre, startpunkt) bevares fra sidste kørsel, medmindre sangen er helt ny.
 *
 * Robusthed (tilføjet 2026-08-30): Apple svarer nogle gange 403/429 hvis vi
 * spørger for hurtigt efter hinanden — det sker typisk et sted midt i en stor
 * bunke nye sange, ikke ved den første. Derfor: (1) en lille pause mellem hvert
 * opslag, (2) automatisk gentagelse med stigende ventetid hvis Apple svarer
 * 403/429, og (3) hvis ÉN sang alligevel fejler helt, springes den bare over
 * med en advarsel — resten af kørslen fortsætter. Resultatet gemmes desuden
 * løbende (for hver 20. sang), så en kørsel der af en eller anden grund
 * stopper undervejs ikke mister alt det den allerede nåede.
 *
 * Matchning mod Apple (rettet 2026-08-30): scriptet kræver nu at kunstneren
 * matcher PRÆCIST et af Apples søgeresultater, før det bruger klip/cover
 * derfra. Før kunne en tvetydig titel ende med at bruge Apples FØRSTE
 * søgeresultat uanset kunstner — det gav sange hvor lyden/coveret ikke
 * passede med den viste titel. Nu springes sangen i stedet over (med en
 * advarsel i loggen), hvis ingen af Apples resultater har den rigtige
 * kunstner.
 */

const fs = require("fs");
const path = require("path");
const os = require("os");
const { spawnSync } = require("child_process");

const ROD = path.join(__dirname, "..");
const CSV_STI = path.join(ROD, "data", "sange.csv");
const JSON_STI = path.join(ROD, "data", "sange.json");

// ---- CSV-parsing (understøtter citerede felter med komma/linjeskift) ----
function parseCsv(tekst) {
  const raekker = [];
  let felt = "";
  let raekke = [];
  let iCitat = false;
  // Fjern evt. UTF-8 BOM
  if (tekst.charCodeAt(0) === 0xfeff) tekst = tekst.slice(1);

  for (let i = 0; i < tekst.length; i++) {
    const c = tekst[i];
    if (iCitat) {
      if (c === '"') {
        if (tekst[i + 1] === '"') { felt += '"'; i++; }
        else { iCitat = false; }
      } else {
        felt += c;
      }
    } else if (c === '"') {
      iCitat = true;
    } else if (c === ",") {
      raekke.push(felt); felt = "";
    } else if (c === "\n" || c === "\r") {
      if (c === "\r" && tekst[i + 1] === "\n") i++;
      raekke.push(felt); felt = "";
      raekker.push(raekke); raekke = [];
    } else {
      felt += c;
    }
  }
  if (felt.length > 0 || raekke.length > 0) { raekke.push(felt); raekker.push(raekke); }

  const rensede = raekker
    .map(r => r.map(f => f.trim()))
    .filter(r => r.some(f => f.length > 0));
  if (rensede.length === 0) return [];

  const header = rensede[0].map(h => h.toLowerCase());
  return rensede.slice(1).map(r => {
    const obj = {};
    header.forEach((h, idx) => { obj[h] = (r[idx] || "").trim(); });
    return obj;
  });
}

// ---- Normalisering til sammenligning/matching (samme princip som i spillet selv) ----
function normaliser(s) {
  return (s || "")
    .toLowerCase()
    .normalize("NFC")
    .replace(/[^\p{L}\p{N}\s]/gu, "")
    .replace(/\s+/g, " ")
    .trim();
}

function noegle(row) {
  return `${normaliser(row.kunstner)}||${normaliser(row.titel)}`;
}

function slugify(s) {
  return (s || "")
    .toLowerCase()
    .replace(/æ/g, "ae").replace(/ø/g, "oe").replace(/å/g, "aa")
    .normalize("NFD").replace(/[̀-ͯ]/g, "") // fjern resterende accenter
    .replace(/[^a-z0-9]+/g, "-")
    .replace(/^-+|-+$/g, "");
}

function lavId(row) {
  return `${slugify(row.kunstner)}-${slugify(row.titel)}`;
}

function decadeFraAar(aar) {
  if (!aar || !Number.isFinite(aar)) return null;
  const dekadeStart = aar - (aar % 10);
  const to = String(dekadeStart % 100).padStart(2, "0");
  return `${to}er`;
}

// Best-effort alternative stavemåder — Emil kan altid tilføje flere direkte i
// sange.json bagefter; de bliver ikke overskrevet ved senere kørsler (se
// bevar-logik nedenfor).
function foreslaaAlternativer(navn) {
  if (!navn) return [];
  const uden = navn.replace(/['’]/g, "").trim();
  return uden !== navn ? [uden] : [];
}

function vent(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

// ---- iTunes-opslag (almindeligt fetch — dette script kører server-side i
// GitHub Actions, IKKE i en browser, så CORS/JSONP er ikke et problem her).
// Gentager automatisk hvis Apple svarer 403/429 (for mange opslag for hurtigt). ----
async function slaaOpHosItunes(row) {
  const term = `${row.titel} ${row.kunstner}`;
  const url = `https://itunes.apple.com/search?term=${encodeURIComponent(term)}&country=DK&media=music&entity=song&limit=5`;

  const MAX_FORSOEG = 4;
  let sidsteFejl = null;
  for (let forsoeg = 1; forsoeg <= MAX_FORSOEG; forsoeg++) {
    try {
      const svar = await fetch(url);
      if (svar.status === 403 || svar.status === 429) {
        const ventetidMs = 2000 * forsoeg; // 2s, 4s, 6s, 8s
        console.warn(`  Apple svarede ${svar.status} (for mange opslag for hurtigt) — venter ${ventetidMs / 1000}s og prøver igen (forsøg ${forsoeg}/${MAX_FORSOEG})`);
        await vent(ventetidMs);
        continue;
      }
      if (!svar.ok) throw new Error(`iTunes svarede ${svar.status}`);
      const data = await svar.json();
      const resultater = Array.isArray(data.results) ? data.results : [];
      if (resultater.length === 0) return null;

      const oenskTitel = normaliser(row.titel);
      const oenskKunstner = normaliser(row.kunstner);

      // Præcist match på både titel og kunstner er bedst.
      const eksakt = resultater.find(r =>
        normaliser(r.trackName) === oenskTitel && normaliser(r.artistName) === oenskKunstner
      );
      if (eksakt) return eksakt;

      // Fejl fundet 2026-08-30: her stod der før `return eksakt || resultater[0]`
      // — dvs. hvis intet resultat matchede PRÆCIST, tog scriptet bare det
      // FØRSTE resultat Apple sendte tilbage, uanset hvor forskelligt det var.
      // For korte/tvetydige titler (fx forkortelser) kan Apples søgning sagtens
      // ramme en helt anden sang af en helt anden kunstner — det gav sange hvor
      // klippet/coveret slet ikke passede med den viste titel. Rettelsen: kunstneren
      // skal matche PRÆCIST (det er den vigtigste beskyttelse mod en forkert
      // sang), men titlen må godt afvige lidt — Apple pynter tit selv på titlen
      // ("(Live)", "Remastered 2013", "feat. …").
      const sammeKunstner = resultater.find(r => normaliser(r.artistName) === oenskKunstner);
      if (sammeKunstner) return sammeKunstner;

      // Ingen af Apples resultater har den rigtige kunstner — spring hellere
      // sangen over (se genererNytOpslag) end at gætte blindt og vise en helt
      // forkert sang.
      return null;
    } catch (e) {
      sidsteFejl = e;
      const ventetidMs = 1500 * forsoeg;
      console.warn(`  Netværksfejl ved opslag (${e.message}) — venter ${ventetidMs / 1000}s og prøver igen (forsøg ${forsoeg}/${MAX_FORSOEG})`);
      await vent(ventetidMs);
    }
  }
  // Alle forsøg mislykkedes — spring denne sang over i stedet for at stoppe hele kørslen.
  console.warn(`  ⚠️  Kunne ikke slå "${row.titel}" — ${row.kunstner} op efter ${MAX_FORSOEG} forsøg (${sidsteFejl ? sidsteFejl.message : "ukendt fejl"}). Springes over — prøv igen senere ved at køre robotten manuelt.`);
  return null;
}

// ---- Startpunkt: spring evt. stilhed i starten af klippet over ----
async function findStartpunktMs(previewUrl) {
  const tmpFil = path.join(os.tmpdir(), `preview-${Date.now()}-${Math.random().toString(36).slice(2)}.m4a`);
  try {
    const svar = await fetch(previewUrl);
    if (!svar.ok) return 0;
    const buffer = Buffer.from(await svar.arrayBuffer());
    fs.writeFileSync(tmpFil, buffer);

    // ffmpeg skriver silencedetect-resultatet til stderr, uanset om
    // kommandoen i øvrigt lykkes eller fejler.
    const proces = spawnSync("ffmpeg", [
      "-i", tmpFil,
      "-af", "silencedetect=noise=-35dB:d=0.2",
      "-f", "null", "-",
    ]);
    const stderrTekst = (proces.stderr || "").toString();

    const startMatch = stderrTekst.match(/silence_start:\s*(-?\d+(\.\d+)?)/);
    const endMatch = stderrTekst.match(/silence_end:\s*(-?\d+(\.\d+)?)/);
    if (startMatch && endMatch) {
      const silenceStart = parseFloat(startMatch[1]);
      const silenceEnd = parseFloat(endMatch[1]);
      // Kun relevant hvis stilheden sidder helt i starten af klippet.
      if (silenceStart <= 0.15 && silenceEnd > 0) {
        const sekunder = Math.min(silenceEnd, 10); // sæt et loft, for en sikkerheds skyld
        return Math.round(sekunder * 10) * 100; // afrund til nærmeste 0,1s, i ms
      }
    }
    return 0;
  } catch (e) {
    console.warn(`  Kunne ikke analysere startpunkt (${e.message}) — bruger 0`);
    return 0;
  } finally {
    if (fs.existsSync(tmpFil)) fs.unlinkSync(tmpFil);
  }
}

async function genererNytOpslag(row) {
  console.log(`  Slår op hos Apple: "${row.titel}" — ${row.kunstner}`);
  const fund = await slaaOpHosItunes(row);
  if (!fund) {
    console.warn(`  ⚠️  Ingen match hos Apple Music for "${row.titel}" — ${row.kunstner}. Springes over — tjek stavning i sange.csv.`);
    return null;
  }

  const aar = fund.releaseDate ? new Date(fund.releaseDate).getUTCFullYear() : null;
  const startpunktMs = fund.previewUrl ? await findStartpunktMs(fund.previewUrl) : 0;

  return {
    id: lavId(row),
    titel: row.titel,
    kunstner: row.kunstner,
    aar: aar,
    aarti: decadeFraAar(aar),
    genre: fund.primaryGenreName ? [fund.primaryGenreName] : [],
    svaerhed: Number(row.svaerhed) || null,
    kategori: row.kategori || null,
    alternative_titler: foreslaaAlternativer(row.titel),
    alternative_kunstnere: foreslaaAlternativer(row.kunstner),
    lyd_kilde: "itunes",
    lyd_id: fund.trackId ? String(fund.trackId) : null,
    lyd_url: fund.previewUrl || null,
    start_offset_ms: startpunktMs,
    cover_url: fund.artworkUrl100 || null,
    link_ud: fund.trackViewUrl || null,
    noter: row.noter || "",
  };
}

function gemResultat(resultat) {
  fs.mkdirSync(path.dirname(JSON_STI), { recursive: true });
  fs.writeFileSync(JSON_STI, JSON.stringify(resultat, null, 2) + "\n", "utf8");
}

async function main() {
  if (!fs.existsSync(CSV_STI)) {
    console.error(`Finder ikke ${CSV_STI}`);
    process.exit(1);
  }
  const csvRaekker = parseCsv(fs.readFileSync(CSV_STI, "utf8"))
    .filter(r => r.titel && r.kunstner);

  let eksisterende = [];
  if (fs.existsSync(JSON_STI)) {
    try {
      const parsed = JSON.parse(fs.readFileSync(JSON_STI, "utf8"));
      if (Array.isArray(parsed)) eksisterende = parsed;
    } catch (e) {
      console.warn(`Kunne ikke læse eksisterende ${JSON_STI} (${e.message}) — starter forfra.`);
    }
  }
  const cache = new Map();
  for (const s of eksisterende) {
    if (s && s.titel && s.kunstner && s.lyd_url) {
      cache.set(`${normaliser(s.kunstner)}||${normaliser(s.titel)}`, s);
    }
  }

  console.log(`Fandt ${csvRaekker.length} sange i sange.csv, ${cache.size} allerede slået op tidligere.`);

  const resultat = [];
  let nyeOpslagSidenGem = 0;
  let sprungetOver = 0;
  for (const row of csvRaekker) {
    const k = noegle(row);
    const cached = cache.get(k);
    if (cached) {
      // Genbrug alt det dyre (lyd, årstal, genre, startpunkt) — synkronisér
      // kun de felter Emil selv redigerer direkte i CSV'en.
      resultat.push({
        ...cached,
        titel: row.titel,
        kunstner: row.kunstner,
        svaerhed: Number(row.svaerhed) || cached.svaerhed || null,
        noter: row.noter || cached.noter || "",
        kategori: row.kategori || cached.kategori || null,
      });
      continue;
    }

    let nyt = null;
    try {
      nyt = await genererNytOpslag(row);
    } catch (e) {
      // Skulle ikke kunne ske længere (slaaOpHosItunes fanger selv sine fejl),
      // men hvis noget uventet alligevel går galt for denne ene sang, skal
      // det ikke vælte resten af kørslen.
      console.warn(`  ⚠️  Uventet fejl ved "${row.titel}" — ${row.kunstner} (${e.message}). Springes over.`);
    }

    if (nyt) {
      resultat.push(nyt);
    } else {
      sprungetOver++;
    }

    // Lille pause mellem hvert nyt opslag, så vi ikke banker Apples API for hurtigt.
    await vent(350);

    // Gem løbende for hver 20. nye sang, så vi ikke mister alt hvis kørslen
    // af en eller anden grund stopper undervejs (fx GitHub Actions der løber
    // tør for tid). Sange der allerede er i cachen tæller ikke med her, kun
    // reelt nye opslag.
    nyeOpslagSidenGem++;
    if (nyeOpslagSidenGem >= 20) {
      gemResultat(resultat);
      nyeOpslagSidenGem = 0;
      console.log(`  … gemt løbende, ${resultat.length} sange indtil videre.`);
    }
  }

  gemResultat(resultat);
  console.log(`Skrev ${resultat.length} sange til ${JSON_STI}.`);
  if (sprungetOver > 0) {
    console.log(`${sprungetOver} sange blev sprunget over (ingen match hos Apple, eller opslaget blev ved med at fejle) — se advarslerne ovenfor.`);
  }
}

if (require.main === module) {
  main().catch(e => {
    console.error("Uventet fejl i generator:", e);
    process.exit(1);
  });
}

module.exports = { parseCsv, normaliser, slugify, lavId, decadeFraAar, foreslaaAlternativer, findStartpunktMs, genererNytOpslag, main };
