/*

- X-SIM v5.1 — Fastbreak Release
- 105 powers, 34 combos, 12 materials, 10 environments.
- v5.1: Damage queue, particle cap, sparse zones, effect pools, toast batching.
- Weather system, terrain, joints, scenarios, AI behaviors.
- Dynamic lighting, zone VFX, procedural audio.
  */
  import { useState, useCallback, useRef, useMemo, useEffect } from “react”;

// ─── POWER DEFINITIONS (DC + Marvel) ───
const POWERS = [
// ═══ ENERGY ═══
{ id:“optic_blast”, name:“Optic Blast”, char:“Cyclops”, icon:“👁️”, cat:“Energy”, color:”#dc2626”, desc:“Ruby-red concussive beam”, mode:“click”, dmg:35 },
{ id:“kinetic_charge”, name:“Kinetic Charge”, char:“Gambit”, icon:“🃏”, cat:“Energy”, color:”#d946ef”, desc:“Hold to charge, release to detonate”, mode:“hold”, dmg:30 },
{ id:“plasmoids”, name:“Plasmoids”, char:“Jubilee”, icon:“🎆”, cat:“Energy”, color:”#f472b6”, desc:“Sparkler burst on click”, mode:“click”, dmg:15 },
{ id:“plasma_rings”, name:“Plasma Rings”, char:“Havok”, icon:“💠”, cat:“Energy”, color:”#60a5fa”, desc:“Concentric plasma discharge”, mode:“click”, dmg:30 },
{ id:“sonic_scream”, name:“Sonic Scream”, char:“Banshee”, icon:“🗣️”, cat:“Energy”, color:”#86efac”, desc:“Expanding sonic wave”, mode:“click”, dmg:20 },
{ id:“solar_blast”, name:“Solar Blast”, char:“Sunspot”, icon:“☀️”, cat:“Energy”, color:”#f59e0b”, desc:“Solar corona burst”, mode:“click”, dmg:25 },
{ id:“energy_absorb”, name:“Energy Absorb”, char:“Bishop”, icon:“⚡”, cat:“Energy”, color:”#fbbf24”, desc:“Absorb & redirect energy”, mode:“click”, dmg:15 },
{ id:“heat_vision”, name:“Heat Vision”, char:“Superman”, icon:“🔴”, cat:“Energy”, color:”#ef4444”, desc:“Dual heat beams from eyes”, mode:“click”, dmg:40 },
{ id:“starbolts”, name:“Starbolts”, char:“Starfire”, icon:“💚”, cat:“Energy”, color:”#22c55e”, desc:“Green energy barrage”, mode:“click”, dmg:25 },
{ id:“photon_blast”, name:“Photon Blast”, char:“Captain Marvel”, icon:“⭐”, cat:“Energy”, color:”#fbbf24”, desc:“Golden photon energy wave”, mode:“click”, dmg:35 },
{ id:“omega_beam”, name:“Omega Beams”, char:“Darkseid”, icon:“Ω”, cat:“Energy”, color:”#dc2626”, desc:“Homing zigzag beams”, mode:“click”, dmg:50 },
{ id:“power_cosmic”, name:“Power Cosmic”, char:“Silver Surfer”, icon:“🏄”, cat:“Energy”, color:”#c4b5fd”, desc:“Cosmic energy nova”, mode:“click”, dmg:40 },
// ═══ ELEMENTAL ═══
{ id:“weather”, name:“Weather”, char:“Storm”, icon:“🌩️”, cat:“Elemental”, color:”#93c5fd”, desc:“Hold to build storm”, mode:“hold”, dmg:30 },
{ id:“cryokinesis”, name:“Cryokinesis”, char:“Iceman”, icon:“❄️”, cat:“Elemental”, color:”#67e8f9”, desc:“Freeze objects, drag for frost trail”, mode:“click”, dmg:10 },
{ id:“pyrokinesis”, name:“Pyrokinesis”, char:“Pyro”, icon:“🔥”, cat:“Elemental”, color:”#fb923c”, desc:“Ignite & control fire”, mode:“click”, dmg:20 },
{ id:“magma”, name:“Magma Flow”, char:“Magma”, icon:“🌋”, cat:“Elemental”, color:”#ef4444”, desc:“Lava eruption from ground”, mode:“click”, dmg:25 },
{ id:“plant_growth”, name:“Plant Growth”, char:“Poison Ivy”, icon:“🌿”, cat:“Elemental”, color:”#4ade80”, desc:“Vines sprout from click”, mode:“click”, dmg:5 },
{ id:“sandstorm”, name:“Sandstorm”, char:“Dust”, icon:“🏜️”, cat:“Elemental”, color:”#d4a373”, desc:“Erosion cloud on objects”, mode:“click”, dmg:15 },
{ id:“freeze_breath”, name:“Freeze Breath”, char:“Killer Frost”, icon:“🥶”, cat:“Elemental”, color:”#a5f3fc”, desc:“Directional ice cone”, mode:“click”, dmg:20 },
{ id:“hydrokinesis”, name:“Hydrokinesis”, char:“Mera”, icon:“🌊”, cat:“Elemental”, color:”#38bdf8”, desc:“Water push and pull”, mode:“click”, dmg:15 },
{ id:“earth_control”, name:“Earth Control”, char:“Terra”, icon:“⛰️”, cat:“Elemental”, color:”#a8a29e”, desc:“Rocks erupt from ground”, mode:“click”, dmg:30 },
{ id:“lightning_summon”, name:“Lightning”, char:“Shazam”, icon:“⚡”, cat:“Elemental”, color:”#fbbf24”, desc:“Call down the lightning!”, mode:“click”, dmg:35 },
{ id:“hellfire”, name:“Hellfire”, char:“Ghost Rider”, icon:“💀”, cat:“Elemental”, color:”#f97316”, desc:“Supernatural fire ignores armor”, mode:“click”, dmg:30 },
// ═══ KINETIC ═══
{ id:“telekinesis”, name:“Telekinesis”, char:“Jean Grey”, icon:“🧠”, cat:“Kinetic”, color:”#fbbf24”, desc:“Lift & throw any object”, mode:“drag”, dmg:10 },
{ id:“magnetism”, name:“Magnetism”, char:“Magneto”, icon:“🧲”, cat:“Kinetic”, color:”#a855f7”, desc:“Control metal + field lines”, mode:“drag”, material:“metal”, dmg:15 },
{ id:“magnetism_g”, name:“Magnetism”, char:“Polaris”, icon:“🧲”, cat:“Kinetic”, color:”#22c55e”, desc:“Green magnetic control”, mode:“drag”, material:“metal”, dmg:15 },
{ id:“super_strength”, name:“Super Strength”, char:“Colossus”, icon:“💪”, cat:“Kinetic”, color:”#94a3b8”, desc:“Fling with massive force”, mode:“drag”, dmg:25 },
{ id:“repulsion”, name:“Repulsion”, char:“Blob”, icon:“🛡️”, cat:“Kinetic”, color:”#f87171”, desc:“Push everything away”, mode:“click”, dmg:20 },
{ id:“seismic”, name:“Seismic Wave”, char:“Avalanche”, icon:“🫨”, cat:“Kinetic”, color:”#78716c”, desc:“Shockwave shakes scene”, mode:“click”, dmg:25 },
{ id:“cannonball”, name:“Blast Field”, char:“Cannonball”, icon:“🚀”, cat:“Kinetic”, color:”#c084fc”, desc:“Propulsive impact blast”, mode:“click”, dmg:30 },
{ id:“thunder_clap”, name:“Thunder Clap”, char:“Hulk”, icon:“👏”, cat:“Kinetic”, color:”#86efac”, desc:“Massive shockwave AOE”, mode:“click”, dmg:40 },
{ id:“shield_throw”, name:“Shield Throw”, char:“Captain America”, icon:“🛡️”, cat:“Kinetic”, color:”#3b82f6”, desc:“Bouncing vibranium shield”, mode:“click”, dmg:30 },
{ id:“mjolnir”, name:“Mjolnir”, char:“Thor”, icon:“🔨”, cat:“Kinetic”, color:”#60a5fa”, desc:“Hammer strike + lightning”, mode:“click”, dmg:40 },
{ id:“lasso”, name:“Lasso of Truth”, char:“Wonder Woman”, icon:“🪢”, cat:“Kinetic”, color:”#fbbf24”, desc:“Pull objects together”, mode:“click”, dmg:15 },
{ id:“vibranium_pulse”, name:“Vibranium Pulse”, char:“Black Panther”, icon:“🐾”, cat:“Kinetic”, color:”#a855f7”, desc:“Stored kinetic release”, mode:“click”, dmg:35 },
// ═══ PHYSICAL ═══
{ id:“claws”, name:“Adamantium Claws”, char:“Wolverine”, icon:“🐺”, cat:“Physical”, color:”#e5e7eb”, desc:“Slash objects apart”, mode:“click”, dmg:45 },
{ id:“claws_x23”, name:“Claws + Kicks”, char:“X-23”, icon:“🗡️”, cat:“Physical”, color:”#d1d5db”, desc:“Dual slash pattern”, mode:“click”, dmg:50 },
{ id:“diamond_form”, name:“Diamond Form”, char:“Emma Frost”, icon:“💎”, cat:“Physical”, color:”#e0f2fe”, desc:“Crystallize objects”, mode:“click”, dmg:5 },
{ id:“organic_steel”, name:“Organic Steel”, char:“Colossus”, icon:“🪨”, cat:“Physical”, color:”#9ca3af”, desc:“Turn objects to chrome”, mode:“click”, dmg:10 },
{ id:“armor”, name:“Psionic Armor”, char:“Armor”, icon:“🛡️”, cat:“Physical”, color:”#f87171”, desc:“Protective exo-shell”, mode:“click”, dmg:0 },
{ id:“duplication”, name:“Duplication”, char:“Multiple Man”, icon:“👥”, cat:“Physical”, color:”#86efac”, desc:“Clone objects on click”, mode:“click”, dmg:0 },
{ id:“reactive”, name:“Reactive Evolve”, char:“Darwin”, icon:“🧬”, cat:“Physical”, color:”#a3e635”, desc:“Objects adapt to threats”, mode:“click”, dmg:5 },
{ id:“web_shot”, name:“Web-Slinging”, char:“Spider-Man”, icon:“🕷️”, cat:“Physical”, color:”#e5e7eb”, desc:“Web restraint on objects”, mode:“click”, dmg:10 },
{ id:“symbiote”, name:“Symbiote”, char:“Venom”, icon:“🖤”, cat:“Physical”, color:”#1e1b4b”, desc:“Tendrils grab + corrupt”, mode:“click”, dmg:30 },
{ id:“hulk_smash”, name:“Hulk Smash”, char:“Hulk”, icon:“👊”, cat:“Physical”, color:”#22c55e”, desc:“Ground pound destruction”, mode:“click”, dmg:50 },
{ id:“martial_arts”, name:“Martial Arts”, char:“Shang-Chi”, icon:“🥋”, cat:“Physical”, color:”#f59e0b”, desc:“Rapid multi-hit combo”, mode:“click”, dmg:40 },
{ id:“regeneration”, name:“Regeneration”, char:“Deadpool”, icon:“❤️‍🩹”, cat:“Physical”, color:”#ef4444”, desc:“Heal objects over time”, mode:“click”, dmg:0 },
// ═══ PSYCHIC ═══
{ id:“psi_blade”, name:“Psychic Blade”, char:“Psylocke”, icon:“🔮”, cat:“Psychic”, color:”#e879f9”, desc:“Pink-purple psi-slash”, mode:“click”, dmg:35 },
{ id:“telepathy”, name:“Telepathy”, char:“Prof X”, icon:“🧠”, cat:“Psychic”, color:”#818cf8”, desc:“Reveal object details”, mode:“click”, dmg:0 },
{ id:“phoenix”, name:“Phoenix Force”, char:“Phoenix”, icon:“🦅”, cat:“Psychic”, color:”#f59e0b”, desc:“Escalates with repeated use!”, mode:“click”, dmg:30 },
{ id:“hex_bolt”, name:“Hex Bolts”, char:“Scarlet Witch”, icon:“🔴”, cat:“Psychic”, color:”#dc2626”, desc:“Creates reality warp zones”, mode:“click”, dmg:20 },
{ id:“illusion”, name:“Illusion”, char:“Mastermind”, icon:“🪞”, cat:“Psychic”, color:”#c4b5fd”, desc:“Ghost copies appear”, mode:“click”, dmg:0 },
{ id:“mind_control”, name:“Mind Control”, char:“Shadow King”, icon:“🎯”, cat:“Psychic”, color:”#7c3aed”, desc:“Objects follow cursor”, mode:“hold”, dmg:5 },
{ id:“probability”, name:“Luck Field”, char:“Domino”, icon:“🎲”, cat:“Psychic”, color:”#34d399”, desc:“Random lucky effects”, mode:“click”, dmg:10 },
{ id:“penance_stare”, name:“Penance Stare”, char:“Ghost Rider”, icon:“💀”, cat:“Psychic”, color:”#f97316”, desc:“Massive single-target damage”, mode:“click”, dmg:60 },
{ id:“empathy”, name:“Empathy”, char:“Raven”, icon:“🕊️”, cat:“Psychic”, color:”#6366f1”, desc:“Transfer statuses between objects”, mode:“click”, dmg:5 },
{ id:“astral_project”, name:“Astral Form”, char:“Dr. Strange”, icon:“👻”, cat:“Psychic”, color:”#e879f9”, desc:“Ghost copy attacks”, mode:“click”, dmg:20 },
// ═══ SPATIAL ═══
{ id:“teleport”, name:“Teleportation”, char:“Nightcrawler”, icon:“💨”, cat:“Spatial”, color:”#6366f1”, desc:“BAMF! Leaves toxic smoke”, mode:“two_click”, dmg:10 },
{ id:“portal”, name:“Stepping Disc”, char:“Magik”, icon:“🌀”, cat:“Spatial”, color:”#fde68a”, desc:“Link two portals”, mode:“two_click”, dmg:5 },
{ id:“super_speed”, name:“Super Speed”, char:“Quicksilver”, icon:“⚡”, cat:“Spatial”, color:”#e5e7eb”, desc:“Trail of afterimages”, mode:“drag”, dmg:15 },
{ id:“phasing”, name:“Phasing”, char:“Shadowcat”, icon:“👻”, cat:“Spatial”, color:”#a5b4fc”, desc:“Objects go intangible”, mode:“click”, dmg:0 },
{ id:“size_shift”, name:“Size Shift”, char:“Apocalypse”, icon:“🔍”, cat:“Spatial”, color:”#fb7185”, desc:“Grow/shrink objects”, mode:“click”, dmg:10 },
{ id:“time_slow”, name:“Time Slow”, char:“Tempo”, icon:“⏳”, cat:“Spatial”, color:”#a78bfa”, desc:“Hold to slow time”, mode:“hold”, dmg:0 },
{ id:“time_reverse”, name:“Time Reverse”, char:“Tempus”, icon:“⏪”, cat:“Spatial”, color:”#60a5fa”, desc:“Reset all objects”, mode:“click”, dmg:0 },
{ id:“gravity”, name:“Gravity Well”, char:“Exodus”, icon:“⬇️”, cat:“Spatial”, color:”#dc2626”, desc:“Pull objects to point”, mode:“hold”, dmg:10 },
{ id:“speed_force”, name:“Speed Force”, char:“The Flash”, icon:“⚡”, cat:“Spatial”, color:”#ef4444”, desc:“Speed zone + lightning trail”, mode:“drag”, dmg:20 },
{ id:“blink_slash”, name:“Blink Slash”, char:“Blink”, icon:“🔪”, cat:“Spatial”, color:”#ec4899”, desc:“Portal slices through objects”, mode:“click”, dmg:35 },
// ═══ MYSTIC ═══
{ id:“mystic_shield”, name:“Mystic Shield”, char:“Dr. Strange”, icon:“🛡️”, cat:“Mystic”, color:”#f59e0b”, desc:“Mandala barrier + reflect”, mode:“click”, dmg:10 },
{ id:“chaos_magic”, name:“Chaos Magic”, char:“Scarlet Witch”, icon:“❌”, cat:“Mystic”, color:”#dc2626”, desc:“Reality rewrite — anything goes”, mode:“click”, dmg:30 },
{ id:“order_magic”, name:“Order Magic”, char:“Dr. Fate”, icon:“☥”, cat:“Mystic”, color:”#fbbf24”, desc:“Golden ankh constructs”, mode:“click”, dmg:25 },
{ id:“sorcery”, name:“Sorcery”, char:“Zatanna”, icon:“🎩”, cat:“Mystic”, color:”#c084fc”, desc:“Reverse-spell random effects”, mode:“click”, dmg:20 },
{ id:“hellfire_chains”, name:“Hellfire Chains”, char:“Ghost Rider”, icon:“⛓️”, cat:“Mystic”, color:”#f97316”, desc:“Burning chains restrain + damage”, mode:“click”, dmg:30 },
{ id:“blood_magic”, name:“Blood Magic”, char:“Blade”, icon:“🩸”, cat:“Mystic”, color:”#b91c1c”, desc:“Life drain to heal others”, mode:“click”, dmg:25 },
{ id:“rune_magic”, name:“Rune Magic”, char:“Loki”, icon:“ᚱ”, cat:“Mystic”, color:”#22d3ee”, desc:“Area buff and debuff runes”, mode:“click”, dmg:15 },
{ id:“eldritch_blast”, name:“Eldritch Blast”, char:“Dormammu”, icon:“🌑”, cat:“Mystic”, color:”#7c3aed”, desc:“Dark dimension energy beam”, mode:“click”, dmg:40 },
// ═══ COSMIC ═══
{ id:“infinity_snap”, name:“Infinity Snap”, char:“Thanos”, icon:“🫰”, cat:“Cosmic”, color:”#c084fc”, desc:“Random half of objects destroyed”, mode:“click”, dmg:999 },
{ id:“nova_force”, name:“Nova Force”, char:“Nova”, icon:“💫”, cat:“Cosmic”, color:”#fbbf24”, desc:“Golden energy burst”, mode:“click”, dmg:35 },
{ id:“binary_mode”, name:“Binary Mode”, char:“Captain Marvel”, icon:“🔥”, cat:“Cosmic”, color:”#f59e0b”, desc:“Sustained damage aura”, mode:“hold”, dmg:20 },
{ id:“lantern_construct”, name:“Hard Light”, char:“Green Lantern”, icon:“💚”, cat:“Cosmic”, color:”#22c55e”, desc:“Construct barrier wall”, mode:“click”, dmg:10 },
{ id:“anti_life”, name:“Anti-Life”, char:“Darkseid”, icon:“☠️”, cat:“Cosmic”, color:”#4b0082”, desc:“Corruption spreading zone”, mode:“click”, dmg:25 },
{ id:“starheart”, name:“Starheart”, char:“Alan Scott”, icon:“💚”, cat:“Cosmic”, color:”#4ade80”, desc:“Green mystical flame”, mode:“click”, dmg:30 },
{ id:“celestial_beam”, name:“Celestial Beam”, char:“Celestials”, icon:“👁️”, cat:“Cosmic”, color:”#fde68a”, desc:“Massive cosmic judgment beam”, mode:“click”, dmg:60 },
{ id:“bifrost”, name:“Bifrost”, char:“Heimdall”, icon:“🌈”, cat:“Cosmic”, color:”#c084fc”, desc:“Rainbow bridge teleport”, mode:“two_click”, dmg:15 },
// ═══ TECH ═══
{ id:“repulsor”, name:“Repulsor Beam”, char:“Iron Man”, icon:“🤖”, cat:“Tech”, color:”#38bdf8”, desc:“Tech energy beam”, mode:“click”, dmg:30 },
{ id:“batarang”, name:“Batarang”, char:“Batman”, icon:“🦇”, cat:“Tech”, color:”#6b7280”, desc:“Bouncing precision projectile”, mode:“click”, dmg:20 },
{ id:“sonic_cannon”, name:“Sonic Cannon”, char:“Cyborg”, icon:“📡”, cat:“Tech”, color:”#60a5fa”, desc:“Wide directional sonic wave”, mode:“click”, dmg:25 },
{ id:“nano_swarm”, name:“Nano Swarm”, char:“Iron Man”, icon:“🔬”, cat:“Tech”, color:”#ef4444”, desc:“Nanobots dissolve + rebuild”, mode:“click”, dmg:20 },
{ id:“technopathy”, name:“Technopathy”, char:“Brainiac”, icon:“🧠”, cat:“Tech”, color:”#22c55e”, desc:“Convert objects to tech/metal”, mode:“click”, dmg:10 },
{ id:“boom_tube”, name:“Boom Tube”, char:“Motherbox”, icon:“🕳️”, cat:“Tech”, color:”#f59e0b”, desc:“Instant Apokolips teleport”, mode:“two_click”, dmg:10 },
{ id:“power_ring”, name:“Power Ring”, char:“Green Lantern”, icon:“💍”, cat:“Tech”, color:”#22c55e”, desc:“Construct cage around object”, mode:“click”, dmg:5 },
{ id:“web_bomb”, name:“Web Bomb”, char:“Spider-Man”, icon:“🕸️”, cat:“Tech”, color:”#d4d4d8”, desc:“Area web explosion”, mode:“click”, dmg:10 },
// ═══ v5.0 NEW POWERS ═══
{ id:“gravity_well”, name:“Gravity Well”, char:“Graviton”, icon:“🌀”, cat:“Cosmic”, color:”#6366f1”, desc:“Create localized gravity pull”, mode:“click”, dmg:5 },
{ id:“time_dilation”, name:“Time Dilation”, char:“Tempo”, icon:“⏳”, cat:“Cosmic”, color:”#fbbf24”, desc:“Slow/speed time in area”, mode:“click”, dmg:0 },
{ id:“transmutation”, name:“Transmutation”, char:“Alchemy”, icon:“🔄”, cat:“Mystic”, color:”#f59e0b”, desc:“Cycle object material”, mode:“click”, dmg:0 },
{ id:“vine_growth”, name:“Vine Growth”, char:“Poison Ivy”, icon:“🌿”, cat:“Elemental”, color:”#22c55e”, desc:“Spawn connected vine chain”, mode:“click”, dmg:10 },
{ id:“sand_control”, name:“Sand Control”, char:“Gaara”, icon:“🏜️”, cat:“Elemental”, color:”#d4b483”, desc:“Create sand barriers”, mode:“click”, dmg:15 },
{ id:“crystal_shield”, name:“Crystal Shield”, char:“Diamondback”, icon:“💎”, cat:“Tech”, color:”#e1bee7”, desc:“Spawn crystal barrier”, mode:“click”, dmg:0 },
{ id:“rubber_bounce”, name:“Rubber Bounce”, char:“Mr. Fantastic”, icon:“🟤”, cat:“Physical”, color:”#2d2d2d”, desc:“Make object super bouncy”, mode:“click”, dmg:0 },
// ═══ DARK ═══
{ id:“shadow_form”, name:“Shadow Form”, char:“Obsidian”, icon:“🌑”, cat:“Dark”, color:”#1e1b4b”, desc:“Darkness zone saps energy”, mode:“click”, dmg:15 },
{ id:“soul_self”, name:“Soul Self”, char:“Raven”, icon:“🐦‍⬛”, cat:“Dark”, color:”#6366f1”, desc:“Shadow raven attacks”, mode:“click”, dmg:30 },
{ id:“necroplasm”, name:“Necroplasm”, char:“Spawn”, icon:“💀”, cat:“Dark”, color:”#16a34a”, desc:“Supernatural dark fire”, mode:“click”, dmg:35 },
{ id:“void_touch”, name:“Void Touch”, char:“Sentry”, icon:“⚫”, cat:“Dark”, color:”#fbbf24”, desc:“Anti-matter disintegration”, mode:“click”, dmg:45 },
{ id:“fear_toxin”, name:“Fear Toxin”, char:“Scarecrow”, icon:“🎃”, cat:“Dark”, color:”#84cc16”, desc:“Confusion + panic zone”, mode:“click”, dmg:10 },
{ id:“darkforce”, name:“Darkforce”, char:“Darkstar”, icon:“✨”, cat:“Dark”, color:”#7c3aed”, desc:“Dark dimension energy blast”, mode:“click”, dmg:30 },
{ id:“black_lantern”, name:“Black Lantern”, char:“Black Hand”, icon:“🖐️”, cat:“Dark”, color:”#111”, desc:“Death corruption ring”, mode:“click”, dmg:35 },
];

const CATEGORIES = [“Energy”,“Elemental”,“Kinetic”,“Physical”,“Psychic”,“Spatial”,“Mystic”,“Cosmic”,“Tech”,“Dark”];
const CAT_COLORS = { Energy:”#f59e0b”, Elemental:”#22c55e”, Kinetic:”#ef4444”, Physical:”#94a3b8”, Psychic:”#a855f7”, Spatial:”#3b82f6”, Mystic:”#c084fc”, Cosmic:”#fde68a”, Tech:”#38bdf8”, Dark:”#6366f1” };
const CAT_ICONS = { Energy:“⚡”, Elemental:“🌍”, Kinetic:“💥”, Physical:“🦾”, Psychic:“🧠”, Spatial:“🌀”, Mystic:“✨”, Cosmic:“🌌”, Tech:“🔧”, Dark:“🌑” };

const MATERIALS = [“metal”,“wood”,“glass”,“stone”,“water”,“organic”,“energy”,“explosive”,“rubber”,“ice”,“sand”,“crystal”];
const MAT_COLORS = { metal:”#8b9daf”, wood:”#a0845e”, glass:”#c2e0f4”, stone:”#8a8279”, water:”#4facd4”, organic:”#6db86d”, energy:”#e8c44a”, explosive:”#d95050”, rubber:”#2d2d2d”, ice:”#b3e5fc”, sand:”#d4b483”, crystal:”#e1bee7” };
const MAT_EMOJI = { metal:“⚙️”, wood:“🪵”, glass:“🔮”, stone:“🪨”, water:“💧”, organic:“🌱”, energy:“⚡”, explosive:“💣”, rubber:“🟤”, ice:“🧊”, sand:“⏳”, crystal:“💎” };
const MAT_HP = { metal:150, wood:80, glass:40, stone:200, water:60, organic:70, energy:120, explosive:50, rubber:100, ice:35, sand:50, crystal:60 };
const MAT_FRAGILE = { glass:true, explosive:true, ice:true, crystal:true };
const MAT_PROPS = {
metal:     { density:3.0, conductivity:0.9, flammability:0,   reflectivity:0.6, meltPoint:500, freezePoint:-50 },
wood:      { density:1.0, conductivity:0.1, flammability:0.8, reflectivity:0.1, meltPoint:null, freezePoint:-20 },
glass:     { density:1.5, conductivity:0.3, flammability:0,   reflectivity:0.8, meltPoint:300,  freezePoint:-30 },
stone:     { density:4.0, conductivity:0.2, flammability:0,   reflectivity:0.2, meltPoint:800,  freezePoint:-100 },
water:     { density:1.0, conductivity:0.7, flammability:0,   reflectivity:0.3, meltPoint:null, freezePoint:0 },
organic:   { density:0.8, conductivity:0.2, flammability:0.6, reflectivity:0.1, meltPoint:null, freezePoint:-10 },
energy:    { density:0.3, conductivity:1.0, flammability:0,   reflectivity:0.9, meltPoint:null, freezePoint:null },
explosive: { density:1.2, conductivity:0.4, flammability:1.0, reflectivity:0.1, meltPoint:null, freezePoint:-40 },
rubber:    { density:0.6, conductivity:0.01, flammability:0.1, reflectivity:0,   meltPoint:null, freezePoint:-60, bounce:0.9, friction:0.8 },
ice:       { density:0.9, conductivity:0.2,  flammability:0,   reflectivity:0.4, meltPoint:30,   freezePoint:null, bounce:0.15, friction:0.05 },
sand:      { density:1.5, conductivity:0.05, flammability:0,   reflectivity:0,   meltPoint:1700, freezePoint:null, bounce:0.05, friction:0.9 },
crystal:   { density:2.5, conductivity:0.3,  flammability:0,   reflectivity:0.8, meltPoint:null, freezePoint:null, bounce:0.3, friction:0.3 },
};
// Derived rules from properties
const MAT_RULES = {
burns: Object.fromEntries(MATERIALS.filter(m=>(MAT_PROPS[m]?.flammability||0)>0.3).map(m=>[m,1])),
conducts: Object.fromEntries(MATERIALS.filter(m=>(MAT_PROPS[m]?.conductivity||0)>0.5).map(m=>[m,1])),
freezes: Object.fromEntries(MATERIALS.filter(m=>MAT_PROPS[m]?.freezePoint!=null).map(m=>[m,1])),
shatters: { glass:1, stone:1 },
magnetizes: { metal:1 },
bounces: { rubber:1 },
melts: { ice:1 },
vitrifies: { sand:1 },
reflects_beam: { crystal:1 },
};

// ─── COMBOS (28) ───
const COMBOS = [
// Original 14
{ a:“kinetic_charge”,b:“claws”,name:“Charged Claws”,desc:“Explosive slashing”,color:”#d946ef”,effect:“charged_slash”,dmg:70 },
{ a:“pyrokinesis”,b:“telekinesis”,name:“Fire Tornado”,desc:“Flaming vortex”,color:”#f97316”,effect:“fire_tornado”,dmg:55 },
{ a:“cryokinesis”,b:“weather”,name:“Blizzard”,desc:“Ice storm barrage”,color:”#67e8f9”,effect:“blizzard”,dmg:50 },
{ a:“optic_blast”,b:“diamond_form”,name:“Prism Scatter”,desc:“Refracted beam spray”,color:”#dc2626”,effect:“prism”,dmg:60 },
{ a:“magnetism”,b:“super_strength”,name:“Magnetic Railgun”,desc:“Launch metal at speed”,color:”#a855f7”,effect:“railgun”,dmg:65 },
{ a:“teleport”,b:“claws”,name:“Bamf Blitz”,desc:“Teleporting slash combos”,color:”#6366f1”,effect:“bamf_blitz”,dmg:75 },
{ a:“sonic_scream”,b:“plasmoids”,name:“Sonic Laser”,desc:“Sound→light barrage”,color:”#86efac”,effect:“sonic_laser”,dmg:50 },
{ a:“pyrokinesis”,b:“cryokinesis”,name:“Thermal Shock”,desc:“Extreme temp shatter”,color:”#fb923c”,effect:“thermal_shock”,dmg:60 },
{ a:“phoenix”,b:“telekinesis”,name:“Phoenix Storm”,desc:“Golden destruction wave”,color:”#f59e0b”,effect:“phoenix_storm”,dmg:80 },
{ a:“hex_bolt”,b:“probability”,name:“Chaos Cascade”,desc:“Everything goes haywire”,color:”#dc2626”,effect:“chaos”,dmg:55 },
{ a:“magnetism”,b:“kinetic_charge”,name:“Metal Barrage”,desc:“Charge all metal at once”,color:”#c084fc”,effect:“metal_barrage”,dmg:60 },
{ a:“phasing”,b:“weather”,name:“Phase Lightning”,desc:“Pass through then shock”,color:”#a5b4fc”,effect:“phase_lightning”,dmg:55 },
{ a:“energy_absorb”,b:“optic_blast”,name:“Overcharged Blast”,desc:“Absorbed → mega beam”,color:”#fbbf24”,effect:“overcharged”,dmg:70 },
{ a:“super_strength”,b:“cannonball”,name:“Fastball Special”,desc:“The classic combo!”,color:”#94a3b8”,effect:“fastball”,dmg:65 },
// New 14 — DC + Marvel cross-universe
{ a:“heat_vision”,b:“freeze_breath”,name:“Thermal Vision”,desc:“Flash-freeze shatter”,color:”#a5f3fc”,effect:“thermal_vision”,dmg:65 },
{ a:“mjolnir”,b:“lightning_summon”,name:“Thunder God”,desc:“Divine lightning storm”,color:”#fbbf24”,effect:“thunder_god”,dmg:75 },
{ a:“web_shot”,b:“symbiote”,name:“Web of Shadows”,desc:“Dark web trap”,color:”#1e1b4b”,effect:“web_shadows”,dmg:55 },
{ a:“hulk_smash”,b:“thunder_clap”,name:“World Breaker”,desc:“Absolute destruction”,color:”#22c55e”,effect:“world_breaker”,dmg:90 },
{ a:“mystic_shield”,b:“chaos_magic”,name:“Nexus Event”,desc:“Reality collapses”,color:”#dc2626”,effect:“nexus_event”,dmg:70 },
{ a:“speed_force”,b:“lightning_summon”,name:“Speed Lightning”,desc:“Chain lightning at Mach 10”,color:”#ef4444”,effect:“speed_lightning”,dmg:60 },
{ a:“hellfire”,b:“hellfire_chains”,name:“Spirit of Vengeance”,desc:“Full Ghost Rider mode”,color:”#f97316”,effect:“spirit_vengeance”,dmg:70 },
{ a:“shield_throw”,b:“mjolnir”,name:“Worthy Shield”,desc:“Shield + lightning redirect”,color:”#3b82f6”,effect:“worthy_shield”,dmg:65 },
{ a:“repulsor”,b:“nano_swarm”,name:“Full Arsenal”,desc:“Iron Man at maximum”,color:”#ef4444”,effect:“full_arsenal”,dmg:60 },
{ a:“symbiote”,b:“void_touch”,name:“Maximum Carnage”,desc:“Symbiote explosion”,color:”#dc2626”,effect:“max_carnage”,dmg:75 },
{ a:“omega_beam”,b:“anti_life”,name:“Apokolips Rising”,desc:“Darkseid unleashed”,color:”#4b0082”,effect:“apokolips”,dmg:85 },
{ a:“infinity_snap”,b:“celestial_beam”,name:“Cosmic Judgment”,desc:“Universal reset”,color:”#fde68a”,effect:“cosmic_judgment”,dmg:999 },
{ a:“batarang”,b:“fear_toxin”,name:“Dark Knight”,desc:“Terror + precision”,color:”#6b7280”,effect:“dark_knight”,dmg:50 },
{ a:“soul_self”,b:“empathy”,name:“Azarath Metrion”,desc:“Raven unleashed”,color:”#6366f1”,effect:“azarath”,dmg:65 },
// ═══ v5.0 COMBOS ═══
{ a:“gravity_well”,b:“magnetism”,name:“Singularity”,desc:“Gravitomagnetic collapse”,color:”#4338ca”,effect:“singularity”,dmg:70 },
{ a:“cryokinesis”,b:“sand_control”,name:“Permafrost”,desc:“Frozen sand barrier”,color:”#a5f3fc”,effect:“permafrost”,dmg:30 },
{ a:“vine_growth”,b:“pyrokinesis”,name:“Wildfire”,desc:“Burning vine spread”,color:”#f97316”,effect:“wildfire”,dmg:55 },
{ a:“crystal_shield”,b:“optic_blast”,name:“Prism Burst”,desc:“Refracted beam spray”,color:”#e879f9”,effect:“prism_burst”,dmg:50 },
{ a:“time_dilation”,b:“super_speed”,name:“Time Stop”,desc:“Freeze all objects in area”,color:”#fbbf24”,effect:“time_stop”,dmg:0 },
{ a:“duplication”,b:“telekinesis”,name:“Clone Army”,desc:“Duplicate and scatter”,color:”#8b5cf6”,effect:“clone_army”,dmg:40 },
];

// ─── ZONE TYPES ───
const ZONE_TYPES = {
fire:   { color:”#f97316”, decay:0.0015, statusType:“burning” },
ice:    { color:”#67e8f9”, decay:0.001,  statusType:“frozen” },
electric:{ color:”#38bdf8”, decay:0.003, statusType:“electrified” },
toxic:  { color:”#a855f7”, decay:0.002,  statusType:“corroding” },
void:   { color:”#dc2626”, decay:0.0025, statusType:null },
magnetic:{ color:”#c084fc”, decay:0.0012, statusType:null },
gravity:{ color:”#6366f1”, decay:0.002, statusType:null },
time:{ color:”#fbbf24”, decay:0.003, statusType:null },
};

// ─── ENVIRONMENTS (v4.0) ───
const ENVIRONMENTS = {
danger_room: { name:“Danger Room”, icon:“🏟️”, gravity:0.4, friction:0.92, powerMods:{}, ambientColor:”#1a1a3e”, floor:true, weather:[“clear”,“storm”] },
city:        { name:“City”,        icon:“🏢”, gravity:0.4, friction:0.90, powerMods:{Elemental:1.2}, ambientColor:”#1a1410”, floor:true, weather:[“clear”,“rain”,“storm”,“wind”] },
space:       { name:“Space”,       icon:“🚀”, gravity:0.02, friction:0.99, powerMods:{Energy:1.3,Cosmic:1.3,Elemental:0.5}, ambientColor:”#050510”, floor:false, weather:[“clear”,“meteor_shower”] },
underwater:  { name:“Underwater”,  icon:“🌊”, gravity:0.08, friction:0.80, powerMods:{Elemental:0.4,Energy:0.7}, ambientColor:”#0a1a2a”, floor:true, weather:[“clear”,“rain”,“storm”] },
volcano:     { name:“Volcano”,     icon:“🌋”, gravity:0.4, friction:0.92, powerMods:{Elemental:1.5}, ambientColor:”#1a0a05”, floor:true, weather:[“clear”,“sandstorm”] },
arctic:      { name:“Arctic”,      icon:“🧊”, gravity:0.35, friction:0.2, powerMods:{Elemental:1.4,Energy:0.9}, ambientColor:”#e0f0ff”, floor:true, weather:[“snow”,“wind”,“clear”] },
forest:      { name:“Forest”,      icon:“🌲”, gravity:0.4, friction:0.85, powerMods:{Elemental:1.2,Mystic:1.3,Physical:1.1}, ambientColor:”#0a1f0a”, floor:true, weather:[“rain”,“clear”,“wind”] },
laboratory:  { name:“Laboratory”,  icon:“🔬”, gravity:0.4, friction:0.6, powerMods:{Energy:1.3,Tech:1.5}, ambientColor:”#f0f0f5”, floor:true, weather:[“clear”] },
astral_plane:{ name:“Astral Plane”, icon:“✨”, gravity:0.05, friction:0.3, powerMods:{Psychic:2.0,Mystic:1.8,Cosmic:1.5,Physical:0.6}, ambientColor:”#1a0033”, floor:false, weather:[“clear”] },
desert:      { name:“Desert”,      icon:“🏜️”, gravity:0.45, friction:0.5, powerMods:{Elemental:0.8,Energy:1.2}, ambientColor:”#3d2b1f”, floor:true, weather:[“sandstorm”,“clear”,“wind”] },
};
const ENV_KEYS = Object.keys(ENVIRONMENTS);

// ─── OBJECTS ───
const ORIGINAL_IDS = new Set([1,2,3,4,5,6,7,8,9,10,11,12,13,14,15]);
const createObjects = () => [
{ id:1,x:80,y:120,w:55,h:55,mat:“metal”,label:“Crate”,shape:“rect” },
{ id:2,x:250,y:280,w:50,h:50,mat:“stone”,label:“Boulder”,shape:“rect” },
{ id:3,x:420,y:100,w:40,h:70,mat:“wood”,label:“Tree”,shape:“rect” },
{ id:4,x:160,y:380,w:40,h:40,mat:“glass”,label:“Orb”,shape:“circle” },
{ id:5,x:560,y:320,w:80,h:45,mat:“metal”,label:“Car”,shape:“rect” },
{ id:6,x:340,y:220,w:30,h:55,mat:“organic”,label:“Person”,shape:“rect” },
{ id:7,x:80,y:300,w:45,h:45,mat:“energy”,label:“Crystal”,shape:“diamond” },
{ id:8,x:480,y:420,w:65,h:30,mat:“water”,label:“Pool”,shape:“rect” },
{ id:9,x:300,y:80,w:85,h:60,mat:“stone”,label:“Wall”,shape:“rect” },
{ id:10,x:620,y:130,w:40,h:40,mat:“glass”,label:“Gem”,shape:“diamond” },
{ id:11,x:500,y:200,w:55,h:35,mat:“metal”,label:“Drone”,shape:“rect” },
{ id:12,x:140,y:200,w:45,h:60,mat:“organic”,label:“Statue”,shape:“rect” },
{ id:13,x:680,y:400,w:35,h:35,mat:“explosive”,label:“Barrel”,shape:“circle” },
{ id:14,x:380,y:400,w:50,h:50,mat:“wood”,label:“Chest”,shape:“rect” },
{ id:15,x:200,y:480,w:60,h:25,mat:“metal”,label:“Beam”,shape:“rect” },
].map(o => ({ …o, origX:o.x,origY:o.y,origMat:o.mat,scale:1,rotation:0,opacity:1,effects:[],statuses:[],vx:0,vy:0,hp:MAT_HP[o.mat],maxHp:MAT_HP[o.mat],fragile:!!MAT_FRAGILE[o.mat],temp:20,mass:o.w*o.h*(MAT_PROPS[o.mat]?.density||1)*0.01,resting:0 }));

const MAX_PARTICLES_DESKTOP = 2000;
const MAX_PARTICLES_MOBILE = 1000;
const PARTICLE_CAP = 200;
const EFFECT_CAP = 30;
const IMPACT_CAP = 20;
const ZONE_COLS = 20;
const ZONE_ROWS = 14;
const createZoneGrid = () => Array.from({length:ZONE_ROWS}, () => Array(ZONE_COLS).fill(null));

// ─── WEATHER SYSTEM (v5.0) ───
const WEATHER_TYPES = {
clear:{ particles:0, force:{x:0,y:0}, tempMod:0, visMod:1 },
rain:{ particles:15, force:{x:0,y:0}, tempMod:-5, visMod:0.85, particleColor:”#60a5fa”, particleDy:12 },
snow:{ particles:8, force:{x:0,y:0}, tempMod:-15, visMod:0.9, particleColor:”#e0f0ff”, particleDy:1.5 },
wind:{ particles:3, force:{x:2.5,y:0}, tempMod:0, visMod:0.95, particleColor:”#d1d5db”, particleDy:0 },
storm:{ particles:20, force:{x:1.5,y:0}, tempMod:-8, visMod:0.7, particleColor:”#60a5fa”, particleDy:14 },
sandstorm:{ particles:18, force:{x:3,y:0}, tempMod:10, visMod:0.5, particleColor:”#d4b483”, particleDy:1 },
meteor_shower:{ particles:0, force:{x:0,y:0}, tempMod:0, visMod:0.9 },
};

// ─── TERRAIN TYPES (v5.0) ───
const TERRAIN_COLORS = { wood:”#8B6914”, metal:”#5a6a7a”, stone:”#6a625a”, ice:”#a0d8ef”, sand:”#c4a35a” };

// ─── SCENARIO DATA (v5.0) ───
const SCENARIOS = [
{ id:“topple_tower”,name:“Topple the Tower”,desc:“Destroy all 8 blocks with a single power use”,env:“danger_room”,goal:{type:“destroy_all”,par_time:15,par_powers:1},
objects:[…Array(8)].map((*,i)=>({x:200,y:350-i*45,w:60,h:40,mat:i%2?“wood”:“stone”,label:i%2?“Plank”:“Rock”,shape:“rect”})) },
{ id:“deep_freeze”,name:“Deep Freeze”,desc:“Freeze all water objects within 30s”,env:“arctic”,goal:{type:“freeze_all”,material:“water”,par_time:30,par_powers:5},
objects:[…Array(10)].map((*,i)=>({x:50+i*42,y:200+Math.sin(i)*60,w:35,h:35,mat:“water”,label:“Drop”,shape:“circle”})) },
{ id:“chain_master”,name:“Chain Master”,desc:“Trigger a 5-link explosive chain”,env:“danger_room”,goal:{type:“chain_reaction”,count:5,par_time:20,par_powers:1},
objects:[…Array(6)].map((_,i)=>({x:60+i*70,y:350,w:40,h:40,mat:“explosive”,label:“Bomb”,shape:“rect”})) },
{ id:“protect_crystal”,name:“Protect the Crystal”,desc:“Keep the glass object alive for 30s”,env:“volcano”,goal:{type:“protect”,material:“glass”,time:30,par_powers:10},
objects:[{x:220,y:300,w:50,h:50,mat:“glass”,label:“Crystal”,shape:“diamond”},…[…Array(4)].map((_,i)=>({x:80+i*100,y:50,w:40,h:40,mat:“stone”,label:“Rock”,shape:“circle”}))] },
{ id:“magneto_sort”,name:“Magneto’s Domain”,desc:“Move all metal to the right half using only magnetism”,env:“danger_room”,goal:{type:“reach_target”,material:“metal”,zone:{x:250,w:250},par_time:45,par_powers:10},
objects:[…Array(8)].map((*,i)=>({x:30+Math.random()*180,y:100+i*35,w:35,h:35,mat:i%2?“metal”:“wood”,label:i%2?“Block”:“Plank”,shape:“rect”})) },
{ id:“elemental_mastery”,name:“Elemental Mastery”,desc:“Create all 6 zone types in 60s”,env:“danger_room”,goal:{type:“zone_count”,count:6,par_time:60,par_powers:12},
objects:[…Array(6)].map((*,i)=>({x:40+i*70,y:300,w:40,h:40,mat:[“metal”,“wood”,“glass”,“stone”,“water”,“organic”][i],label:“Target”,shape:“rect”})) },
{ id:“rube_goldberg”,name:“Rube Goldberg”,desc:“Chain reaction through 5 different materials”,env:“danger_room”,goal:{type:“material_chain”,count:5,par_time:30,par_powers:1},
objects:[{x:40,y:300,w:50,h:50,mat:“explosive”,label:“Start”,shape:“rect”},{x:120,y:280,w:40,h:40,mat:“wood”,label:“Plank”,shape:“rect”},{x:200,y:260,w:40,h:40,mat:“glass”,label:“Pane”,shape:“rect”},{x:280,y:240,w:40,h:40,mat:“organic”,label:“Blob”,shape:“circle”},{x:360,y:300,w:40,h:40,mat:“metal”,label:“Block”,shape:“rect”}] },
{ id:“combo_king”,name:“Combo King”,desc:“Discover 10 combos in 3 minutes”,env:“laboratory”,goal:{type:“combo_count”,count:10,par_time:180,par_powers:30},
objects:[…Array(10)].map((_,i)=>({x:30+i*44,y:300,w:35,h:35,mat:[“metal”,“wood”,“glass”,“stone”,“water”,“metal”,“organic”,“energy”,“stone”,“wood”][i],label:“Target”,shape:“rect”})) },
];

// ─── BEHAVIOR TYPES (v5.0) ───
const BEHAVIOR_TYPES = {
sentinel:{ icon:“🛡️”, color:”#3b82f6”, speed:1.5 },
seeker:{ icon:“🎯”, color:”#ef4444”, speed:2.5 },
orbiter:{ icon:“🔄”, color:”#a855f7”, speed:2.0 },
repeller:{ icon:“💨”, color:”#22c55e”, speed:0, range:100 },
mimic:{ icon:“🪞”, color:”#f59e0b”, speed:1.0, delay:200 },
};

// ─── SPATIAL GRID (v4.5) ───
class SpatialGrid {
constructor(cellSize=80) { this.cs=cellSize; this.cells={}; }
_key(x,y) { return `${Math.floor(x/this.cs)},${Math.floor(y/this.cs)}`; }
clear() { this.cells={}; }
insert(obj) {
const x0=Math.floor(obj.x/this.cs), y0=Math.floor(obj.y/this.cs);
const x1=Math.floor((obj.x+obj.w)/this.cs), y1=Math.floor((obj.y+obj.h)/this.cs);
for(let x=x0;x<=x1;x++) for(let y=y0;y<=y1;y++){
const k=`${x},${y}`; if(!this.cells[k]) this.cells[k]=[]; this.cells[k].push(obj);
}
}
query(cx,cy,r) {
const seen=new Set(), out=[];
const x0=Math.floor((cx-r)/this.cs), x1=Math.floor((cx+r)/this.cs);
const y0=Math.floor((cy-r)/this.cs), y1=Math.floor((cy+r)/this.cs);
for(let x=x0;x<=x1;x++) for(let y=y0;y<=y1;y++){
const c=this.cells[`${x},${y}`]; if(!c) continue;
for(const o of c) if(!seen.has(o.id)){seen.add(o.id);out.push(o);}
}
return out;
}
rebuild(objs) { this.clear(); for(const o of objs) this.insert(o); }
}
const spatialGrid = new SpatialGrid(80);

// ─── COMPONENT ───
export default function XSim() {
const [powers, setPowers] = useState([]);
const [objects, setObjects] = useState(createObjects);
const [catFilter, setCatFilter] = useState(null);
const [powerSearch, setPowerSearch] = useState(””);
const [dragging, setDragging] = useState(null);
const [dragOff, setDragOff] = useState({x:0,y:0});
const [twoClickSrc, setTwoClickSrc] = useState(null);
const [holdIv, setHoldIv] = useState(null);
const holdIvRef = useRef(null);
const [shake, setShake] = useState(0);
const shakeRef = useRef(0);
const [showInfo, setShowInfo] = useState(null);
const [showPanel, setShowPanel] = useState(true);
const [showDiscovery, setShowDiscovery] = useState(false);
const [env, setEnv] = useState(“danger_room”);
const [buildMode, setBuildMode] = useState(false);
const [buildTool, setBuildTool] = useState(“object”); // “object”|“terrain”|“joint”

// ─── v5.0 STATE ───
const [weather, setWeather] = useState(“clear”);
const weatherRef = useRef(“clear”);
const weatherTimerRef = useRef(null);
const [scenario, setScenario] = useState(null);
const scenarioRef = useRef(null);
const scenarioStartRef = useRef(0);
const [scenarioResult, setScenarioResult] = useState(null);
const terrainRef = useRef([]);
const jointsRef = useRef([]);
const nextJointId = useRef(1);
const [jointMode, setJointMode] = useState(null); // null | {objA, anchorA}
const weatherParticlesRef = useRef([]);
const lightSourcesRef = useRef([]);
const [buildMat, setBuildMat] = useState(“metal”);
const [buildShape, setBuildShape] = useState(“rect”);
const [onboardStep, setOnboardStep] = useState(0);
const onboardStepRef = useRef(0);
useEffect(() => { onboardStepRef.current = onboardStep; }, [onboardStep]);
const [replayMode, setReplayMode] = useState(null);
const snapshotsRef = useRef([]);
const snapshotMaxRef = useRef(50);
const lastSnapshotRef = useRef(0);
const shownTipsRef = useRef(new Set());
const usedCatsRef = useRef(new Set());
const hintedCombosRef = useRef(new Set());
const sessionStartRef = useRef(Date.now());
const envRef = useRef(ENVIRONMENTS.danger_room);
const [comboFlash, setComboFlash] = useState(null);
const [discovered, setDiscovered] = useState(new Set());
const [toasts, setToasts] = useState([]);
const toastQueueRef = useRef([]); // v5.1 toast batching
const toastFlushTimerRef = useRef(null);
const [tick, setTick] = useState(0); // force re-render for HP bars etc

// Refs
const sceneRef = useRef(null);
const canvasRef = useRef(null);
const [bounds, setBounds] = useState({w:800,h:560});
const isMobile = bounds.w < 768;
const animRef = useRef(null);
const pKeyRef = useRef(0);
const pointerRef = useRef({x:0,y:0});
const effectIdRef = useRef(0);
const powersRef = useRef([]);
const objectsRef = useRef([]);
const particlesRef = useRef([]);
const beamsRef = useRef([]);
const groundFxRef = useRef([]);
const zonesRef = useRef(createZoneGrid());
const activeZonesRef = useRef(new Set()); // v5.1 sparse zone tracking
const lastStatusTickRef = useRef(0);
const lastAmbientRef = useRef(0);
const lastCollisionAudioRef = useRef(0);
const collDiscRef = useRef(new Set());
const discoveredRef = useRef(new Set());
const activeEffectsRef = useRef([]);
// v5.1 — Damage queue: accumulate damage, flush once per frame
const damageQueueRef = useRef([]);
const lastPwrCollRef = useRef(0);
const pwrCollCooldowns = useRef({});
const phoenixRef = useRef({ level:1, lastUsed:0 });
const holdStartRef = useRef(0);
const stormStageRef = useRef(0);
const impactsRef = useRef([]);
const postFxRef = useRef([]);
const floatingDmgRef = useRef([]);

// ═══════════════════════════════════════════════════════
// ─── AUDIO ENGINE (v3.7 — procedural Web Audio synthesis) ───
// ═══════════════════════════════════════════════════════
const audioCtxRef = useRef(null);
const masterGainRef = useRef(null);
const sfxGainRef = useRef(null);
const uiGainRef = useRef(null);
const ambiGainRef = useRef(null);
const noiseBufferRef = useRef(null);
const [muted, setMuted] = useState(false);
const [volume, setVolume] = useState(0.5);
const replayModeRef = useRef(null);
const mutedRef = useRef(false);

// ─── Ref sync effects ───
useEffect(() => { holdIvRef.current = holdIv; }, [holdIv]);
useEffect(() => { powersRef.current = powers; }, [powers]);
useEffect(() => { objectsRef.current = objects; }, [objects]);
useEffect(() => { replayModeRef.current = replayMode; }, [replayMode]);
useEffect(() => { mutedRef.current = muted; }, [muted]);
useEffect(() => { envRef.current = ENVIRONMENTS[env]; weatherParticlesRef.current=[]; }, [env]);
useEffect(() => { weatherRef.current = weather; }, [weather]);
const nextPKey = useCallback(() => ++pKeyRef.current, []);
const nextEffectId = useCallback(() => ++effectIdRef.current, []);

const initAudio = useCallback(() => {
if (audioCtxRef.current) return;
try {
const ctx = new (window.AudioContext || window.webkitAudioContext)();
audioCtxRef.current = ctx;
const master = ctx.createGain(); master.gain.value = 0.5; master.connect(ctx.destination); masterGainRef.current = master;
const sfx = ctx.createGain(); sfx.gain.value = 1; sfx.connect(master); sfxGainRef.current = sfx;
const ui = ctx.createGain(); ui.gain.value = 0.6; ui.connect(master); uiGainRef.current = ui;
const ambi = ctx.createGain(); ambi.gain.value = 0.3; ambi.connect(master); ambiGainRef.current = ambi;
const bufSize = ctx.sampleRate * 2;
const buf = ctx.createBuffer(1, bufSize, ctx.sampleRate);
const d = buf.getChannelData(0);
for (let i = 0; i < bufSize; i++) d[i] = Math.random() * 2 - 1;
noiseBufferRef.current = buf;
} catch(e) { /* audio unsupported */ }
}, []);

// Synthesis primitives
const playTone = useCallback((freq, dur, type=“sine”, env={a:0.01,d:0.05,s:0.3,r:0.1}, ch=“sfx”, detune=0) => {
const ctx = audioCtxRef.current; if (!ctx||muted) return;
const gn = ch===“ui”?uiGainRef.current:ch===“ambi”?ambiGainRef.current:sfxGainRef.current; if(!gn) return;
const t=ctx.currentTime, o=ctx.createOscillator(), g=ctx.createGain();
o.type=type; o.frequency.value=freq; if(detune) o.detune.value=detune;
g.gain.setValueAtTime(0,t);
g.gain.linearRampToValueAtTime(env.s||0.3,t+(env.a||0.01));
g.gain.setValueAtTime(env.s||0.3,t+(env.a||0.01)+(env.d||0.05));
g.gain.linearRampToValueAtTime(env.s||0.3,t+dur-(env.r||0.1));
g.gain.linearRampToValueAtTime(0,t+dur);
o.connect(g); g.connect(gn); o.start(t); o.stop(t+dur+0.05);
}, [muted]);

const playNoise = useCallback((dur, fFreq=2000, fQ=1, vol=0.3, ch=“sfx”) => {
const ctx=audioCtxRef.current; if(!ctx||muted) return;
const gn=ch===“ui”?uiGainRef.current:ch===“ambi”?ambiGainRef.current:sfxGainRef.current;
if(!gn||!noiseBufferRef.current) return;
const t=ctx.currentTime, s=ctx.createBufferSource(); s.buffer=noiseBufferRef.current;
const f=ctx.createBiquadFilter(); f.type=“bandpass”; f.frequency.value=fFreq; f.Q.value=fQ;
const g=ctx.createGain(); g.gain.setValueAtTime(vol,t); g.gain.linearRampToValueAtTime(0,t+dur);
s.connect(f); f.connect(g); g.connect(gn); s.start(t); s.stop(t+dur+0.05);
}, [muted]);

const playSweep = useCallback((sf, ef, dur, type=“sawtooth”, vol=0.2, ch=“sfx”) => {
const ctx=audioCtxRef.current; if(!ctx||muted) return;
const gn=ch===“ui”?uiGainRef.current:ch===“ambi”?ambiGainRef.current:sfxGainRef.current; if(!gn) return;
const t=ctx.currentTime, o=ctx.createOscillator(), g=ctx.createGain();
o.type=type; o.frequency.setValueAtTime(sf,t); o.frequency.exponentialRampToValueAtTime(Math.max(20,ef),t+dur);
g.gain.setValueAtTime(vol,t); g.gain.linearRampToValueAtTime(0,t+dur);
o.connect(g); g.connect(gn); o.start(t); o.stop(t+dur+0.05);
}, [muted]);

const playChord = useCallback((freqs, dur, type=“triangle”, vol=0.1) => {
freqs.forEach((f,i) => playTone(f,dur,type,{a:0.02,d:0.1,s:vol,r:dur*0.3},“sfx”,i*3));
}, [playTone]);

useEffect(() => { if(masterGainRef.current) masterGainRef.current.gain.value = muted?0:volume; }, [muted,volume]);
useEffect(() => { const h=e=>{if(e.key===‘m’||e.key===‘M’)setMuted(p=>!p);}; window.addEventListener(‘keydown’,h); return()=>window.removeEventListener(‘keydown’,h); }, []);

// ─── POWER SOUND MAP ───
const PS = useMemo(() => ({
optic_blast:“B”,heat_vision:“B”,repulsor:“B”,celestial_beam:“BH”,eldritch_blast:“BD”,omega_beam:“BH”,
starbolts:“E”,photon_blast:“EH”,power_cosmic:“EH”,nova_force:“EH”,plasmoids:“ES”,plasma_rings:“E”,
sonic_scream:“SO”,sonic_cannon:“SO”,
pyrokinesis:“F”,magma:“FH”,hellfire:“FD”,solar_blast:“FH”,necroplasm:“FD”,starheart:“F”,binary_mode:“F”,
cryokinesis:“I”,freeze_breath:“IH”,hydrokinesis:“W”,
weather:“EB”,lightning_summon:“EL”,mjolnir:“ELH”,speed_force:“ELF”,kinetic_charge:“CH”,
hulk_smash:“IM”,thunder_clap:“IM”,seismic:“IH”,cannonball:“IH”,vibranium_pulse:“IH”,
shield_throw:“IB”,repulsion:“IP”,earth_control:“IH”,
claws:“S”,claws_x23:“SM”,blink_slash:“SP”,martial_arts:“SM”,psi_blade:“SY”,batarang:“ST”,
phoenix:“PE”,hex_bolt:“PW”,telepathy:“PS”,chaos_magic:“PW”,soul_self:“PD”,astral_project:“PG”,
mind_control:“PH”,probability:“PL”,illusion:“PG”,empathy:“PS”,penance_stare:“PD”,sorcery:“PW”,
shadow_form:“DS”,void_touch:“DS”,anti_life:“DH”,black_lantern:“DH”,fear_toxin:“DI”,darkforce:“DB”,symbiote:“DS”,
nano_swarm:“TS”,technopathy:“TC”,web_bomb:“TP”,power_ring:“TH”,boom_tube:“PB”,web_shot:“TP”,
teleport:“PA”,portal:“PI”,bifrost:“PR”,super_speed:“WH”,phasing:“WH”,size_shift:“MO”,
time_slow:“TL”,time_reverse:“TR”,gravity:“GR”,
diamond_form:“AC”,organic_steel:“AM”,armor:“AH”,mystic_shield:“AG”,lantern_construct:“AH”,order_magic:“AG”,
plant_growth:“HL”,regeneration:“HL”,rune_magic:“HR”,blood_magic:“DR”,
duplication:“MO”,reactive:“MO”,sandstorm:“ER”,energy_absorb:“DR”,lasso:“WP”,
magnetism:“MG”,magnetism_g:“MG”,gravity_well:“PH”,time_dilation:“PH”,transmutation:“PH”,vine_growth:“CK”,sand_control:“CK”,crystal_shield:“IP”,rubber_bounce:“IP”,super_strength:“IP”,telekinesis:“PH”,hellfire_chains:“FC”,
infinity_snap:“SN”,
}), []);

const playPowerSound = useCallback((pid) => {
const ctx=audioCtxRef.current; if(!ctx||muted) return;
const s=PS[pid]||“E”;
switch(s) {
case “B”: playSweep(1200,300,0.25,“sawtooth”,0.15); playNoise(0.15,3000,2,0.1); break;
case “BH”: playSweep(800,150,0.4,“sawtooth”,0.2); playNoise(0.25,2000,1,0.15); playTone(80,0.3,“sine”,{a:0.01,d:0.1,s:0.15,r:0.15}); break;
case “BD”: playSweep(600,80,0.5,“square”,0.12); playNoise(0.3,800,3,0.1); break;
case “E”: playTone(600,0.15,“sine”,{a:0.005,d:0.05,s:0.25,r:0.08}); playNoise(0.08,4000,2,0.1); break;
case “EH”: playTone(400,0.25,“sine”,{a:0.01,d:0.08,s:0.3,r:0.1}); playNoise(0.15,3000,1,0.15); playTone(60,0.3,“sine”,{a:0.01,d:0.1,s:0.12,r:0.15}); break;
case “ES”: for(let i=0;i<3;i++) setTimeout(()=>playTone(800+i*200,0.08,“sine”,{a:0.003,d:0.02,s:0.2,r:0.04}),i*40); break;
case “SO”: playSweep(2000,400,0.3,“sine”,0.12); playNoise(0.2,1500,0.5,0.15); break;
case “F”: playNoise(0.3,1200,1,0.15); playTone(120,0.2,“sawtooth”,{a:0.02,d:0.05,s:0.08,r:0.12}); break;
case “FH”: playNoise(0.5,800,0.8,0.2); playTone(80,0.4,“sawtooth”,{a:0.02,d:0.1,s:0.12,r:0.2}); break;
case “FD”: playNoise(0.4,600,1,0.15); playTone(55,0.4,“square”,{a:0.03,d:0.1,s:0.1,r:0.2}); break;
case “FC”: playNoise(0.3,1000,1,0.12); playTone(2000,0.05,“sine”,{a:0.002,d:0.01,s:0.3,r:0.02}); break;
case “I”: playTone(3000,0.15,“sine”,{a:0.005,d:0.05,s:0.15,r:0.08}); playTone(3200,0.12,“sine”,{a:0.008,d:0.04,s:0.1,r:0.06},“sfx”,15); break;
case “IH”: playTone(2500,0.2,“sine”,{a:0.005,d:0.08,s:0.15,r:0.1}); playNoise(0.15,7000,3,0.1); break;
case “W”: playNoise(0.25,2000,0.5,0.12); playSweep(400,200,0.2,“sine”,0.08); break;
case “EL”: playNoise(0.08,8000,5,0.25); playTone(2000,0.04,“square”,{a:0.001,d:0.01,s:0.3,r:0.02}); break;
case “ELH”: playNoise(0.1,8000,5,0.3); playTone(1500,0.06,“square”,{a:0.001,d:0.02,s:0.35,r:0.02}); playTone(70,0.3,“sine”,{a:0.01,d:0.1,s:0.1,r:0.15}); break;
case “ELF”: for(let i=0;i<3;i++) setTimeout(()=>playNoise(0.03,6000+i*1000,5,0.15),i*30); break;
case “EB”: playSweep(200,2000,0.5,“sawtooth”,0.06); break;
case “CH”: playSweep(200,800,0.3,“sine”,0.1); break;
case “IP”: playTone(80,0.15,“sine”,{a:0.003,d:0.05,s:0.3,r:0.08}); playNoise(0.08,1500,1,0.15); break;
case “IM”: playTone(35,0.4,“sine”,{a:0.005,d:0.15,s:0.4,r:0.2}); playNoise(0.2,800,0.5,0.25); break;
case “IB”: playTone(100,0.1,“sine”,{a:0.003,d:0.03,s:0.25,r:0.05}); setTimeout(()=>playTone(120,0.08,“sine”,{a:0.003,d:0.02,s:0.2,r:0.04}),120); break;
case “S”: playNoise(0.06,3000,3,0.2); playTone(1200,0.04,“sawtooth”,{a:0.002,d:0.01,s:0.15,r:0.02}); break;
case “SM”: for(let i=0;i<3;i++) setTimeout(()=>playNoise(0.05,3000+i*500,3,0.15),i*100); break;
case “SY”: playNoise(0.05,3500,3,0.15); playTone(600,0.1,“triangle”,{a:0.005,d:0.03,s:0.1,r:0.05}); break;
case “SP”: playSweep(2000,800,0.08,“sine”,0.12); playNoise(0.05,4000,3,0.15); break;
case “ST”: playSweep(1500,600,0.15,“sine”,0.1); break;
case “PE”: { const lv=phoenixRef.current?.level||1; playChord([330,440,550].slice(0,Math.min(3,lv)),0.3+lv*0.1,“triangle”,0.06+lv*0.02); if(lv>=3)playNoise(0.3,600,1,0.08); if(lv>=5)playTone(40,0.5,“sine”,{a:0.05,d:0.2,s:0.15,r:0.2}); break; }
case “PW”: playChord([220,277,330],0.3,“triangle”,0.08); break;
case “PS”: playTone(600,0.15,“triangle”,{a:0.01,d:0.05,s:0.1,r:0.06}); setTimeout(()=>playTone(800,0.1,“triangle”,{a:0.01,d:0.03,s:0.08,r:0.04}),80); break;
case “PD”: playChord([165,196,247],0.4,“triangle”,0.08); break;
case “PG”: playSweep(600,300,0.3,“triangle”,0.06); break;
case “PH”: playTone(300,0.15,“triangle”,{a:0.01,d:0.05,s:0.08,r:0.06}); break;
case “PL”: playTone(500+Math.random()*500,0.1,“sine”,{a:0.005,d:0.03,s:0.15,r:0.04}); break;
case “DS”: playTone(45,0.5,“square”,{a:0.15,d:0.1,s:0.1,r:0.2}); playNoise(0.4,400,2,0.06); break;
case “DH”: playTone(35,0.6,“square”,{a:0.2,d:0.15,s:0.12,r:0.2}); playNoise(0.5,300,1.5,0.08); break;
case “DB”: playTone(50,0.3,“square”,{a:0.01,d:0.1,s:0.15,r:0.15}); playNoise(0.15,1000,2,0.12); break;
case “DI”: playNoise(0.4,3000,4,0.1); break;
case “TS”: for(let i=0;i<5;i++) setTimeout(()=>playTone(800+Math.random()*400,0.04,“square”,{a:0.002,d:0.01,s:0.1,r:0.02}),i*30); break;
case “TC”: playTone(1000,0.03,“square”,{a:0.002,d:0.01,s:0.2,r:0.01}); break;
case “TP”: playTone(400,0.08,“sine”,{a:0.003,d:0.03,s:0.2,r:0.04}); playNoise(0.06,5000,3,0.08); break;
case “TH”: playTone(220,0.2,“square”,{a:0.01,d:0.05,s:0.08,r:0.1}); break;
case “TL”: playSweep(500,100,0.4,“sine”,0.06); break;
case “TR”: playSweep(200,1200,0.4,“sine”,0.08); break;
case “PA”: playSweep(200,1500,0.12,“sine”,0.15); playNoise(0.1,2000,2,0.12); break;
case “PI”: playSweep(600,1200,0.2,“sine”,0.08); break;
case “PR”: for(let i=0;i<4;i++) setTimeout(()=>playTone(400+i*150,0.12,“sine”,{a:0.01,d:0.04,s:0.1,r:0.05}),i*50); break;
case “PB”: playTone(80,0.15,“sine”,{a:0.005,d:0.05,s:0.2,r:0.08}); playSweep(400,1000,0.1,“sine”,0.1); break;
case “WH”: playNoise(0.12,2000,1,0.1); break;
case “MO”: playSweep(300,600,0.15,“triangle”,0.08); break;
case “GR”: playSweep(400,60,0.3,“sine”,0.1); break;
case “AC”: playTone(4000,0.08,“sine”,{a:0.003,d:0.02,s:0.2,r:0.04}); playTone(4200,0.06,“sine”,{a:0.005,d:0.02,s:0.15,r:0.03},“sfx”,12); break;
case “AM”: playTone(2000,0.1,“sine”,{a:0.003,d:0.03,s:0.2,r:0.05}); break;
case “AH”: playTone(300,0.2,“sine”,{a:0.02,d:0.05,s:0.1,r:0.1}); break;
case “AG”: playTone(800,0.12,“sine”,{a:0.01,d:0.04,s:0.12,r:0.05}); playTone(1200,0.1,“sine”,{a:0.015,d:0.03,s:0.08,r:0.04}); break;
case “HL”: playTone(523,0.12,“sine”,{a:0.01,d:0.03,s:0.12,r:0.05}); setTimeout(()=>playTone(659,0.1,“sine”,{a:0.01,d:0.03,s:0.1,r:0.04}),80); setTimeout(()=>playTone(784,0.08,“sine”,{a:0.01,d:0.03,s:0.08,r:0.03}),150); break;
case “HR”: playTone(440,0.15,“triangle”,{a:0.01,d:0.05,s:0.08,r:0.06}); break;
case “DR”: playSweep(600,200,0.25,“sine”,0.1); break;
case “ER”: playNoise(0.3,2000,1,0.1); break;
case “WP”: playNoise(0.05,4000,4,0.2); playSweep(800,1500,0.06,“sine”,0.12); break;
case “MG”: playTone(120,0.25,“sine”,{a:0.02,d:0.08,s:0.1,r:0.12}); break;
case “SN”: setTimeout(()=>playTone(2000,0.02,“sine”,{a:0.001,d:0.005,s:0.4,r:0.01}),100); setTimeout(()=>{playNoise(0.4,1500,0.5,0.2);playTone(50,0.5,“sine”,{a:0.05,d:0.15,s:0.15,r:0.25});},600); break;
default: playTone(500,0.1,“sine”,{a:0.005,d:0.03,s:0.2,r:0.05}); break;
}
}, [muted,PS,playTone,playNoise,playSweep,playChord]);

// ─── COMBO SOUNDS ───
const playComboSound = useCallback((fx) => {
if(!audioCtxRef.current||muted) return;
switch(fx) {
case “world_breaker”: playTone(30,0.5,“sine”,{a:0.01,d:0.2,s:0.4,r:0.25}); setTimeout(()=>playNoise(0.3,800,0.5,0.25),150); break;
case “phoenix_storm”: playNoise(0.6,1000,0.8,0.15); playChord([220,277,330,440],0.5,“triangle”,0.08); break;
case “apokolips”: playTone(30,0.8,“square”,{a:0.3,d:0.2,s:0.12,r:0.25}); setTimeout(()=>playNoise(0.1,6000,4,0.3),500); break;
case “cosmic_judgment”: playSweep(200,3000,0.4,“sine”,0.1); setTimeout(()=>{playNoise(0.5,2000,0.5,0.3);playTone(30,0.5,“sine”,{a:0.01,d:0.2,s:0.3,r:0.25});},500); break;
case “thunder_god”: for(let i=0;i<4;i++) setTimeout(()=>{playNoise(0.08,8000,5,0.25);playTone(1500,0.05,“square”,{a:0.001,d:0.02,s:0.3,r:0.02});},i*150); break;
case “spirit_vengeance”: playNoise(0.4,1000,1,0.12); playTone(2000,0.04,“sine”,{a:0.002,d:0.01,s:0.25,r:0.02}); break;
case “nexus_event”: for(let i=0;i<6;i++) setTimeout(()=>playTone(300+Math.random()*400,0.02,“square”,{a:0.001,d:0.005,s:0.2,r:0.01}),i*25); break;
default: playTone(400,0.15,“sine”,{a:0.005,d:0.05,s:0.2,r:0.08}); playNoise(0.1,2000,1,0.12); break;
}
setTimeout(()=>playTone(880,0.08,“sine”,{a:0.005,d:0.02,s:0.1,r:0.03},“ui”),50);
}, [muted,playTone,playNoise,playSweep,playChord]);

// ─── INTERACTION SOUNDS ───
const playImpact = useCallback((dmg) => {
if(!audioCtxRef.current||muted) return;
if(dmg<15) playTone(200,0.06,“sine”,{a:0.003,d:0.02,s:0.12,r:0.03});
else if(dmg<35) { playTone(120,0.1,“sine”,{a:0.005,d:0.03,s:0.2,r:0.05}); playNoise(0.06,2000,2,0.08); }
else { playTone(70,0.15,“sine”,{a:0.005,d:0.05,s:0.3,r:0.08}); playNoise(0.1,1500,1,0.12); }
}, [muted,playTone,playNoise]);

const playDestroy = useCallback((mat) => {
if(!audioCtxRef.current||muted) return;
const m={glass:()=>{for(let i=0;i<4;i++) setTimeout(()=>playTone(3000+Math.random()*2000,0.05,“sine”,{a:0.002,d:0.01,s:0.15,r:0.03}),i*20);playNoise(0.1,6000,3,0.12);},
wood:()=>{playNoise(0.08,1500,2,0.2);playTone(200,0.1,“sine”,{a:0.003,d:0.03,s:0.15,r:0.05});},
metal:()=>{playTone(1500,0.2,“sine”,{a:0.003,d:0.08,s:0.15,r:0.1});playTone(1200,0.15,“sine”,{a:0.005,d:0.06,s:0.1,r:0.08},“sfx”,-10);},
stone:()=>{playNoise(0.15,800,0.8,0.2);playTone(100,0.15,“sine”,{a:0.005,d:0.05,s:0.15,r:0.08});},
explosive:()=>{playTone(50,0.3,“sine”,{a:0.005,d:0.1,s:0.35,r:0.15});playNoise(0.2,1000,0.5,0.25);},
water:()=>playNoise(0.15,3000,0.5,0.12),organic:()=>playNoise(0.1,1500,1,0.1),energy:()=>playSweep(1000,200,0.2,“sine”,0.12)};
(m[mat]||m.wood)();
}, [muted,playTone,playNoise,playSweep]);

const playStatus = useCallback((st) => {
if(!audioCtxRef.current||muted) return;
const m={burning:()=>playNoise(0.1,1200,1,0.06),frozen:()=>playTone(3000,0.08,“sine”,{a:0.005,d:0.02,s:0.1,r:0.04}),
electrified:()=>playNoise(0.04,6000,5,0.1),corroding:()=>playNoise(0.15,2500,3,0.05),phased:()=>playSweep(800,400,0.1,“sine”,0.06),
armored:()=>playTone(1500,0.06,“sine”,{a:0.003,d:0.02,s:0.15,r:0.03}),charged:()=>playSweep(200,600,0.1,“sine”,0.06)};
(m[st]||(() => {}))();
}, [muted,playTone,playNoise,playSweep]);

const sfxUI = useCallback((freq=800) => playTone(freq,0.03,“sine”,{a:0.003,d:0.01,s:0.4,r:0.01},“ui”), [playTone]);
const sfxChime = useCallback(() => { if(!audioCtxRef.current||muted) return; playTone(523,0.1,“triangle”,{a:0.005,d:0.03,s:0.12,r:0.04},“ui”); setTimeout(()=>playTone(659,0.08,“triangle”,{a:0.005,d:0.03,s:0.1,r:0.03},“ui”),80); setTimeout(()=>playTone(784,0.08,“triangle”,{a:0.005,d:0.03,s:0.08,r:0.03},“ui”),150); }, [muted,playTone]);
const sfxReset = useCallback(() => { if(!audioCtxRef.current||muted) return; playSweep(800,200,0.3,“sine”,0.08,“ui”); }, [muted,playSweep]);
const sfxPickup = useCallback(() => playTone(120,0.06,“sine”,{a:0.003,d:0.02,s:0.15,r:0.03}), [playTone]);
const sfxDrop = useCallback(() => playTone(90,0.08,“sine”,{a:0.003,d:0.03,s:0.12,r:0.04}), [playTone]);
// ═══════════════════════════════════════════════════════

const activeCombo = useMemo(() => {
if (powers.length !== 2) return null;
return COMBOS.find(c => (c.a===powers[0]&&c.b===powers[1])||(c.a===powers[1]&&c.b===powers[0])) || null;
}, [powers]);

const activePower = useMemo(() => powers.length > 0 ? POWERS.find(p => p.id === powers[powers.length-1]) : null, [powers]);

// ─── AMBIENT AUDIO PER ENVIRONMENT (v4.5) ───
const ambientOscRef = useRef(null);
const ambientNoiseRef = useRef(null);

const startAmbientForEnv = useCallback((envKey) => {
const ctx = audioCtxRef.current; if(!ctx) return;
// Stop previous
try { ambientOscRef.current?.stop(); } catch(e){}
try { ambientNoiseRef.current?.stop(); } catch(e){}
const gain = ambiGainRef.current; if(!gain) return;

```
const configs = {
  danger_room: { freq:80, type:'sine', vol:0.02, noiseVol:0.01 },
  city: { freq:0, type:'sine', vol:0, noiseVol:0.03 },
  space: { freq:30, type:'sine', vol:0.03, noiseVol:0.005 },
  underwater: { freq:0, type:'sine', vol:0, noiseVol:0.02 },
  volcano: { freq:40, type:'sine', vol:0.04, noiseVol:0.03 },
  arctic: { freq:0, type:'sine', vol:0, noiseVol:0.01 },
  forest: { freq:0, type:'sine', vol:0, noiseVol:0.025 },
  laboratory: { freq:120, type:'sine', vol:0.01, noiseVol:0.005 },
  astral_plane: { freq:50, type:'triangle', vol:0.025, noiseVol:0.015 },
  desert: { freq:0, type:'sine', vol:0, noiseVol:0.02 },
};
const cfg = configs[envKey] || configs.danger_room;

if(cfg.freq > 0){
  const osc = ctx.createOscillator();
  const g = ctx.createGain();
  osc.type = cfg.type;
  osc.frequency.value = cfg.freq;
  g.gain.value = 0;
  g.gain.linearRampToValueAtTime(cfg.vol, ctx.currentTime+0.3);
  osc.connect(g).connect(gain);
  osc.start();
  ambientOscRef.current = osc;
}
if(cfg.noiseVol > 0 && noiseBufferRef.current){
  const src = ctx.createBufferSource();
  src.buffer = noiseBufferRef.current;
  src.loop = true;
  const g = ctx.createGain();
  const filt = ctx.createBiquadFilter();
  filt.type = 'bandpass';
  filt.frequency.value = envKey==='city'?400:envKey==='underwater'?200:600;
  filt.Q.value = 0.5;
  g.gain.value = 0;
  g.gain.linearRampToValueAtTime(cfg.noiseVol, ctx.currentTime+0.3);
  src.connect(filt).connect(g).connect(gain);
  src.start();
  ambientNoiseRef.current = src;
}
```

}, []);

useEffect(() => { startAmbientForEnv(env); }, [env, startAmbientForEnv]);

// ─── CANVAS PARTICLE SYSTEM (v3.1 — typed particles) ───
const _push = useCallback((arr, items) => {
arr.push(…items);
const _mp = (typeof window!==“undefined”&&window.innerWidth<768) ? MAX_PARTICLES_MOBILE : MAX_PARTICLES_DESKTOP;
if (arr.length > _mp) arr.splice(0, arr.length - _mp);
}, []);

const spawn = useCallback((x,y,color,count=12,spread=60,life=800,sizeRange=[3,8],type=“circle”,extra=null) => {
const arr = particlesRef.current, now=Date.now(), items=[];
for (let i=0;i<count;i++) {
const a=(Math.PI*2*i)/count+(Math.random()-0.5)*0.6;
const d=spread*(0.4+Math.random()*0.6);
items.push({x,y,dx:Math.cos(a)*d,dy:Math.sin(a)*d,color,life,size:sizeRange[0]+Math.random()*(sizeRange[1]-sizeRange[0]),born:now,type,extra:extra?{…extra}:null});
}
_push(arr, items);
}, [_push]);

const spawnFire = useCallback((x,y,count=12,spread=40,life=900) => {
const arr=particlesRef.current, now=Date.now(), items=[];
for(let i=0;i<count;i++){
const a=-Math.PI/2+(Math.random()-0.5)*1.2;
const d=spread*(0.3+Math.random()*0.7);
items.push({x:x+(Math.random()-0.5)*10,y,dx:Math.cos(a)*d*0.4,dy:Math.sin(a)*d,color:”#fb923c”,life:life*(0.6+Math.random()*0.4),size:4+Math.random()*6,born:now,type:“fire”,extra:null});
}
_push(arr, items);
}, [_push]);

const spawnIce = useCallback((x,y,count=8,spread=30) => {
const arr=particlesRef.current, now=Date.now(), items=[];
for(let i=0;i<count;i++){
const a=(Math.PI*2*i)/count+(Math.random()-0.5)*0.4;
const d=spread*(0.4+Math.random()*0.6);
items.push({x,y,dx:Math.cos(a)*d,dy:Math.sin(a)*d+0.5,color:”#67e8f9”,life:800+Math.random()*400,size:3+Math.random()*4,born:now,type:“ice”,extra:{sides:3+Math.floor(Math.random()*3),spin:(Math.random()-0.5)*4}});
}
_push(arr, items);
}, [_push]);

const spawnLightning = useCallback((x1,y1,x2,y2,color=”#93c5fd”) => {
const arr=particlesRef.current, now=Date.now();
const dx=x2-x1, dy=y2-y1, len=Math.sqrt(dx*dx+dy*dy)||1;
const segs=6+Math.floor(len/30), nx=-dy/len, ny=dx/len;
const pts=[{x:x1,y:y1}];
for(let i=1;i<segs;i++){const t=i/segs;pts.push({x:x1+dx*t+nx*(Math.random()-0.5)*25,y:y1+dy*t+ny*(Math.random()-0.5)*25});}
pts.push({x:x2,y:y2});
arr.push({x:x1,y:y1,dx:0,dy:0,color,life:180,size:3,born:now,type:“lightning”,extra:{points:pts,width:3}});
if(segs>4){
const bi=2+Math.floor(Math.random()*(segs-3)), bp=pts[bi];
const ba=Math.atan2(dy,dx)+(Math.random()>0.5?0.5:-0.5), blen=len*0.3*(0.5+Math.random()*0.5);
const bpts=[{x:bp.x,y:bp.y}];
for(let j=1;j<=3;j++){const t=j/3;bpts.push({x:bp.x+Math.cos(ba)*blen*t+(Math.random()-0.5)*12,y:bp.y+Math.sin(ba)*blen*t+(Math.random()-0.5)*12});}
arr.push({x:bp.x,y:bp.y,dx:0,dy:0,color,life:150,size:2,born:now,type:“lightning”,extra:{points:bpts,width:1.5}});
}
for(let i=0;i<4;i++){const a=Math.random()*Math.PI*2,d=8+Math.random()*15;arr.push({x:x2,y:y2,dx:Math.cos(a)*d,dy:Math.sin(a)*d,color:”#fff”,life:150,size:2,born:now,type:“spark”,extra:{trail:[]}});}
}, []);

const spawnSlash = useCallback((x,y,angle,color=”#e5e7eb”,len=60) => {
const arr=particlesRef.current, now=Date.now();
arr.push({x,y,dx:0,dy:0,color,life:180,size:len,born:now,type:“slash”,extra:{angle,len,width:4}});
for(let i=0;i<6;i++){const a=angle+(Math.random()-0.5)*1.5,d=20+Math.random()*30;arr.push({x,y,dx:Math.cos(a)*d,dy:Math.sin(a)*d,color:”#fbbf24”,life:300,size:1.5,born:now,type:“spark”,extra:{trail:[]}});}
}, []);

const spawnSmoke = useCallback((x,y,color=”#6366f1”,count=8,spread=25) => {
const arr=particlesRef.current, now=Date.now(), items=[];
for(let i=0;i<count;i++){const a=(Math.PI*2*i)/count+(Math.random()-0.5)*0.8,d=spread*(0.3+Math.random()*0.7);items.push({x,y,dx:Math.cos(a)*d*0.3,dy:-Math.abs(Math.sin(a)*d*0.3)-1,color,life:1200+Math.random()*600,size:8+Math.random()*10,born:now,type:“smoke”,extra:null});}
_push(arr, items);
}, [_push]);

const spawnPsi = useCallback((x,y,color=”#e879f9”,count=6,spread=40) => {
const arr=particlesRef.current, now=Date.now(), items=[];
for(let i=0;i<count;i++){const a=(Math.PI*2*i)/count+(Math.random()-0.5)*0.5,d=spread*(0.4+Math.random()*0.6);items.push({x,y,dx:Math.cos(a)*d,dy:Math.sin(a)*d,color,life:700+Math.random()*300,size:3+Math.random()*3,born:now,type:“psi”,extra:{freq:2+Math.random()*3,amp:4+Math.random()*6,phase:Math.random()*Math.PI*2}});}
_push(arr, items);
}, [_push]);

const _genLightningPts = (x1,y1,x2,y2) => {
const dx=x2-x1,dy=y2-y1,len=Math.sqrt(dx*dx+dy*dy)||1,segs=5+Math.floor(len/40),nx=-dy/len,ny=dx/len;
const pts=[{x:x1,y:y1}];
for(let i=1;i<segs;i++){const t=i/segs;pts.push({x:x1+dx*t+nx*(Math.random()-0.5)*20,y:y1+dy*t+ny*(Math.random()-0.5)*20});}
pts.push({x:x2,y:y2}); return pts;
};

const spawnBeamFx = useCallback((x1,y1,x2,y2,color,width=4,style=“energy”) => {
const life=style===“frost”?600:style===“lightning”?250:400;
beamsRef.current.push({x1,y1,x2,y2,color,width,born:Date.now(),life,style,extra:style===“lightning”?_genLightningPts(x1,y1,x2,y2):null});
if(activeEffectsRef.current.length>=EFFECT_CAP)activeEffectsRef.current.shift();
activeEffectsRef.current.push({type:“beam”,x1,y1,x2,y2,color,element:style===“frost”?“ice”:style===“lightning”?“electric”:style===“psi”?“psychic”:“energy”,born:Date.now(),life,id:Date.now()+Math.random()});
const type=style===“lightning”?“spark”:style===“psi”?“psi”:style===“frost”?“ice”:“circle”;
const arr=particlesRef.current, now=Date.now();
for(let i=0;i<6;i++){const t=i/5;arr.push({x:x1+(x2-x1)*t,y:y1+(y2-y1)*t,dx:(Math.random()-0.5)*8,dy:(Math.random()-0.5)*8,color,life:300,size:2,born:now,type,extra:type===“spark”?{trail:[]}:type===“ice”?{sides:4,spin:2}:type===“psi”?{freq:3,amp:4,phase:Math.random()*6}:null});}
}, []);

const spawnRing = useCallback((x,y,color,radius=100,orbit=false) => {
const arr=particlesRef.current, now=Date.now(), count=orbit?20:24;
for(let i=0;i<count;i++){const a=(Math.PI*2*i)/count;
if(orbit){arr.push({x,y,dx:0,dy:0,color,life:800,size:4,born:now,type:“ring”,extra:{cx:x,cy:y,radius,angle:a,speed:2+Math.random()*1.5,arcLen:0.3+Math.random()*0.2}});}
else{arr.push({x,y,dx:Math.cos(a)*radius,dy:Math.sin(a)*radius,color,life:600,size:4,born:now,type:“circle”,extra:null});}
}
}, []);

const spawnGroundFx = useCallback((x,y,color,radius=20,life=3000) => { groundFxRef.current.push({x,y,color,radius,life,born:Date.now()}); }, []);
const doShake = useCallback((intensity=0.6) => { shakeRef.current=intensity; setShake(intensity); }, []);

// Impact / post-effect helpers
const spawnShockwave = useCallback((x,y,maxR=120,color=”#fff”) => { impactsRef.current.push({type:“shockwave”,x,y,maxR,color,born:Date.now(),life:350}); }, []);
const spawnSlashImpact = useCallback((x,y,angle,color=”#e5e7eb”,count=1) => { for(let i=0;i<count;i++) impactsRef.current.push({type:“slashArc”,x,y,angle:angle+(i*1.2),color,born:Date.now(),life:200}); }, []);
const spawnHitFlash = useCallback((x,y,dmg=20) => { impactsRef.current.push({type:“flash”,x,y,radius:Math.min(40,8+dmg*0.5),born:Date.now(),life:80}); }, []);
const spawnDmgNumber = useCallback((x,y,dmg,color=”#fff”) => { if(dmg>15) floatingDmgRef.current.push({x,y:y-10,dmg:Math.round(dmg),color,born:Date.now()}); }, []);
const spawnPostFx = useCallback((type,config={}) => { if(postFxRef.current.length<3) postFxRef.current.push({type,born:Date.now(),life:config.life||200,…config}); }, []);

// ─── CANVAS DRAW (v3.1 — 10-layer VFX pipeline) ───
const drawCanvas = useCallback((ctx, w, h, now, activePwrs, objs) => {
ctx.clearRect(0,0,w,h);

```
// === L0: Environment background ===
const ek = Object.keys(ENVIRONMENTS).find(k=>ENVIRONMENTS[k]===envRef.current) || "danger_room";
ctx.save(); ctx.globalAlpha = 0.4;
if(ek==="danger_room") {
  ctx.strokeStyle="#3b82f640"; ctx.lineWidth=0.5;
  const gs=40,off=(now*0.01)%gs;
  for(let x=off;x<w;x+=gs){ctx.beginPath();ctx.moveTo(x,0);ctx.lineTo(x,h);ctx.stroke();}
  for(let y=off;y<h;y+=gs){ctx.beginPath();ctx.moveTo(0,y);ctx.lineTo(w,y);ctx.stroke();}
} else if(ek==="city") {
  ctx.fillStyle="#0a0a0f";
  [80,120,95,140,70,110,130,85,100,115,90].forEach((bht,i)=>{ctx.fillRect(i*(w/11),h-bht,w/11-3,bht);});
  ctx.fillStyle="#f59e0b20";
  for(let i=0;i<40;i++){ctx.fillRect((i*73+now*0.002)%w,h-30-Math.random()*120,2,2);}
} else if(ek==="space") {
  for(let i=0;i<80;i++){
    ctx.globalAlpha=0.3+Math.sin(now*0.002+i)*0.2;
    ctx.fillStyle="#fff";ctx.fillRect((i*137+now*0.003)%w,(i*89+now*0.001)%h,1+((i%3===0)?1:0),1);
  }
  ctx.globalAlpha=0.06;const grd=ctx.createRadialGradient(w*0.8,h*0.3,10,w*0.8,h*0.3,150);
  grd.addColorStop(0,"#7c3aed");grd.addColorStop(1,"transparent");ctx.fillStyle=grd;ctx.fillRect(0,0,w,h);
} else if(ek==="underwater") {
  for(let i=0;i<6;i++){ctx.globalAlpha=0.04;ctx.fillStyle="#38bdf8";ctx.fillRect(Math.sin(now*0.001+i*2)*w*0.3+w*0.5-100,i*(h/6),200,h/6);}
  for(let i=0;i<15;i++){ctx.globalAlpha=0.15;ctx.fillStyle="#93c5fd";ctx.beginPath();ctx.arc((i*97+now*0.02)%w,h-((now*0.03+i*50)%h),1+i%3,0,Math.PI*2);ctx.fill();}
} else if(ek==="volcano") {
  ctx.globalAlpha=0.08;const lvGrd=ctx.createLinearGradient(0,h,0,h*0.6);
  lvGrd.addColorStop(0,"#dc2626");lvGrd.addColorStop(1,"transparent");ctx.fillStyle=lvGrd;ctx.fillRect(0,0,w,h);
  for(let i=0;i<10;i++){ctx.globalAlpha=0.2;ctx.fillStyle="#f97316";ctx.fillRect((i*131+now*0.01)%w,h-((now*0.02+i*40)%(h*0.6)),1,1);}
}
ctx.restore();

// === L1: Ground effects ===
const gfx=groundFxRef.current;
for(let i=gfx.length-1;i>=0;i--){const g=gfx[i],age=(now-g.born)/g.life;if(age>=1){gfx.splice(i,1);continue;}ctx.globalCompositeOperation="lighter";ctx.globalAlpha=(1-age)*0.3;const gr=ctx.createRadialGradient(g.x,g.y,0,g.x,g.y,g.radius);gr.addColorStop(0,g.color);gr.addColorStop(1,"transparent");ctx.fillStyle=gr;ctx.beginPath();ctx.arc(g.x,g.y,g.radius,0,Math.PI*2);ctx.fill();}

// === L2: Zones ===
const zones=zonesRef.current, cw=w/ZONE_COLS, ch=h/ZONE_ROWS;
ctx.globalCompositeOperation="lighter";
for(let r=0;r<ZONE_ROWS;r++) for(let c=0;c<ZONE_COLS;c++){const z=zones[r][c];if(!z)continue;const zt=ZONE_TYPES[z.type],zx=c*cw,zy=r*ch;ctx.globalAlpha=z.intensity*0.25;ctx.fillStyle=zt.color;ctx.fillRect(zx,zy,cw,ch);if(z.type==="fire"){ctx.globalAlpha=z.intensity*0.15*(0.7+Math.random()*0.3);ctx.fillStyle="#fbbf24";ctx.fillRect(zx+Math.random()*cw*0.5,zy,cw*0.5,ch);}else if(z.type==="electric"&&Math.random()<0.3){ctx.globalAlpha=z.intensity*0.3;ctx.strokeStyle="#fff";ctx.lineWidth=1;ctx.beginPath();ctx.moveTo(zx+Math.random()*cw,zy);ctx.lineTo(zx+Math.random()*cw,zy+ch);ctx.stroke();}else if(z.type==="gravity"){ctx.globalAlpha=z.intensity*0.12;ctx.strokeStyle="#6366f1";ctx.lineWidth=0.8;const gcx=zx+cw/2,gcy=zy+ch/2;for(let gi=0;gi<3;gi++){const gr=8+gi*8+Math.sin(now*0.003+gi)*3;ctx.beginPath();ctx.arc(gcx,gcy,gr,0,Math.PI*2);ctx.stroke();}}else if(z.type==="time"){ctx.globalAlpha=z.intensity*0.15;ctx.strokeStyle="#fbbf24";ctx.lineWidth=1;const tcx=zx+cw/2,tcy=zy+ch/2;ctx.beginPath();ctx.arc(tcx,tcy,12,0,Math.PI*2);ctx.stroke();const ha=now*0.001;ctx.beginPath();ctx.moveTo(tcx,tcy);ctx.lineTo(tcx+Math.cos(ha)*8,tcy+Math.sin(ha)*8);ctx.stroke();}else if(z.type==="magnetic"){ctx.globalAlpha=z.intensity*0.1;ctx.strokeStyle="#c084fc";ctx.lineWidth=0.5;const mcx=zx+cw/2,mcy=zy+ch/2;for(let mi=0;mi<4;mi++){const ma=mi*Math.PI/2+now*0.002;ctx.beginPath();ctx.moveTo(mcx,mcy);ctx.quadraticCurveTo(mcx+Math.cos(ma)*15,mcy+Math.sin(ma)*15,mcx+Math.cos(ma+0.5)*25,mcy+Math.sin(ma+0.5)*25);ctx.stroke();}}}

// === L3: Status overlays ===
if(objs){ctx.globalCompositeOperation="lighter";for(const o of objs){if(o.statuses.length===0)continue;const ox=o.x+o.w/2,oy=o.y+o.h/2,hw=o.w*o.scale/2,hh=o.h*o.scale/2;for(const s of o.statuses){
  if(s.type==="burning"){ctx.globalAlpha=0.25;const fg=ctx.createRadialGradient(ox,oy+hh,0,ox,oy+hh,hw*1.5);fg.addColorStop(0,"#f97316");fg.addColorStop(1,"transparent");ctx.fillStyle=fg;ctx.beginPath();ctx.arc(ox,oy+hh,hw*1.5,0,Math.PI*2);ctx.fill();for(let f=0;f<3;f++){const fx=ox+(Math.random()-0.5)*o.w*0.6,fy=oy-hh,fh=8+Math.random()*12,ph=now*0.008+f*2;ctx.globalAlpha=0.4+Math.sin(ph)*0.2;const gd=ctx.createRadialGradient(fx,fy,0,fx,fy-fh,fh);gd.addColorStop(0,"#fbbf24");gd.addColorStop(0.4,"#fb923c");gd.addColorStop(1,"transparent");ctx.fillStyle=gd;ctx.beginPath();ctx.arc(fx,fy-fh*0.5,fh,0,Math.PI*2);ctx.fill();}}
  if(s.type==="frozen"){ctx.globalAlpha=0.2;const ig=ctx.createRadialGradient(ox,oy,0,ox,oy,Math.max(hw,hh)*1.8);ig.addColorStop(0,"#a5f3fc");ig.addColorStop(1,"transparent");ctx.fillStyle=ig;ctx.beginPath();ctx.arc(ox,oy,Math.max(hw,hh)*1.8,0,Math.PI*2);ctx.fill();ctx.globalAlpha=0.5;ctx.strokeStyle="#e0f2fe";ctx.lineWidth=1.5;[[ox-hw,oy-hh],[ox+hw,oy-hh],[ox+hw,oy+hh],[ox-hw,oy+hh]].forEach(([cx,cy],ci)=>{const ca=Math.PI/4+ci*Math.PI/2+Math.sin(now*0.002+ci)*0.2,cl=6+Math.sin(now*0.003+ci*1.5)*3;ctx.beginPath();ctx.moveTo(cx,cy);ctx.lineTo(cx+Math.cos(ca)*cl,cy+Math.sin(ca)*cl);ctx.stroke();const sba=ca+(ci%2===0?0.5:-0.5);ctx.beginPath();ctx.moveTo(cx+Math.cos(ca)*cl*0.5,cy+Math.sin(ca)*cl*0.5);ctx.lineTo(cx+Math.cos(ca)*cl*0.5+Math.cos(sba)*cl*0.4,cy+Math.sin(ca)*cl*0.5+Math.sin(sba)*cl*0.4);ctx.stroke();});}
  if(s.type==="electrified"){ctx.globalAlpha=0.6;ctx.strokeStyle="#a5f3fc";ctx.lineWidth=1.5;for(let a=0;a<2;a++){const ea=Math.random()*Math.PI*2,er=Math.max(hw,hh),sx=ox+Math.cos(ea)*er*0.8,sy=oy+Math.sin(ea)*er*0.8,el=15+Math.random()*20;ctx.beginPath();ctx.moveTo(sx,sy);ctx.lineTo(sx+Math.cos(ea)*el*0.5+(Math.random()-0.5)*10,sy+Math.sin(ea)*el*0.5+(Math.random()-0.5)*10);ctx.lineTo(sx+Math.cos(ea)*el+(Math.random()-0.5)*8,sy+Math.sin(ea)*el+(Math.random()-0.5)*8);ctx.stroke();}if(Math.sin(now*0.02)>0.8){ctx.globalAlpha=0.15;ctx.fillStyle="#fff";ctx.beginPath();ctx.arc(ox,oy,Math.max(hw,hh)*1.3,0,Math.PI*2);ctx.fill();}}
  if(s.type==="phased"){ctx.globalCompositeOperation="source-over";ctx.globalAlpha=0.08;ctx.fillStyle="#a5b4fc";for(let sl=0;sl<Math.floor(o.h/3);sl++){if(sl%2===0)ctx.fillRect(o.x-2,o.y+sl*3,o.w+4,1.5);}ctx.globalAlpha=0.06;ctx.fillRect(o.x+3+Math.sin(now*0.005)*2,o.y+2,o.w,o.h);ctx.globalCompositeOperation="lighter";}
  if(s.type==="charged"){const urg=1-s.ticksLeft/6;ctx.globalAlpha=0.15+urg*0.25;const cg=ctx.createRadialGradient(ox,oy,0,ox,oy,Math.max(hw,hh)*(1.5+urg*0.5));cg.addColorStop(0,"#d946ef");cg.addColorStop(1,"transparent");ctx.fillStyle=cg;ctx.beginPath();ctx.arc(ox,oy,Math.max(hw,hh)*(1.5+urg*0.5),0,Math.PI*2);ctx.fill();ctx.strokeStyle="#d946ef";ctx.lineWidth=1;ctx.globalAlpha=0.4+urg*0.3;for(let cl=0;cl<3;cl++){const cla=Math.random()*Math.PI*2,cll=10+urg*15;ctx.beginPath();ctx.moveTo(ox,oy);ctx.lineTo(ox+Math.cos(cla)*cll,oy+Math.sin(cla)*cll);ctx.stroke();}}
  if(s.type==="armored"){ctx.globalAlpha=0.12;ctx.strokeStyle="#f87171";ctx.lineWidth=0.5;const gs=8;for(let gx=o.x-2;gx<o.x+o.w+2;gx+=gs)for(let gy=o.y-2;gy<o.y+o.h+2;gy+=gs*0.866){const off=(Math.floor((gy-o.y)/(gs*0.866))%2)*gs/2;ctx.beginPath();ctx.arc(gx+off,gy,gs/2,0,Math.PI*2);ctx.stroke();}}
  if(s.type==="corroding"){ctx.globalAlpha=0.15;const dg=ctx.createRadialGradient(ox,oy+hh+5,0,ox,oy+hh+5,hw);dg.addColorStop(0,"#8a7a50");dg.addColorStop(1,"transparent");ctx.fillStyle=dg;ctx.beginPath();ctx.arc(ox,oy+hh+5,hw,0,Math.PI*2);ctx.fill();}
}}}

// === L4: Beams (styled) ===
ctx.globalCompositeOperation="lighter";
const bms=beamsRef.current;
for(let i=bms.length-1;i>=0;i--){const b=bms[i],age=(now-b.born)/b.life;if(age>=1){bms.splice(i,1);continue;}const fd=1-age;
  if(b.style==="lightning"){
    const pts=b.extra||[{x:b.x1,y:b.y1},{x:b.x2,y:b.y2}];
    ctx.globalAlpha=fd*0.4;ctx.strokeStyle=b.color;ctx.lineWidth=b.width*3;ctx.lineCap="round";ctx.lineJoin="round";ctx.beginPath();pts.forEach((p,pi)=>{const jx=pi>0&&pi<pts.length-1?(Math.random()-0.5)*4:0,jy=pi>0&&pi<pts.length-1?(Math.random()-0.5)*4:0;pi===0?ctx.moveTo(p.x+jx,p.y+jy):ctx.lineTo(p.x+jx,p.y+jy);});ctx.stroke();
    ctx.globalAlpha=fd*0.95;ctx.strokeStyle="#fff";ctx.lineWidth=Math.max(1,b.width*0.8);ctx.beginPath();pts.forEach((p,pi)=>{pi===0?ctx.moveTo(p.x,p.y):ctx.lineTo(p.x,p.y);});ctx.stroke();
  } else if(b.style==="slash"){
    const mx=(b.x1+b.x2)/2,my=(b.y1+b.y2)/2,dx=b.x2-b.x1,dy=b.y2-b.y1,cpx=mx+(-dy)*0.4,cpy=my+(dx)*0.4;
    ctx.globalAlpha=fd*0.5;ctx.strokeStyle=b.color;ctx.lineWidth=b.width*2.5;ctx.lineCap="round";ctx.beginPath();ctx.moveTo(b.x1,b.y1);ctx.quadraticCurveTo(cpx,cpy,b.x2,b.y2);ctx.stroke();
    ctx.globalAlpha=fd*0.95;ctx.strokeStyle="#fff";ctx.lineWidth=Math.max(1,b.width*0.5);ctx.beginPath();ctx.moveTo(b.x1,b.y1);ctx.quadraticCurveTo(cpx,cpy,b.x2,b.y2);ctx.stroke();
  } else if(b.style==="rainbow"){
    const dx=b.x2-b.x1,dy=b.y2-b.y1,len=Math.sqrt(dx*dx+dy*dy)||1,nx=-dy/len,ny=dx/len;
    ["#ef4444","#f97316","#fbbf24","#22c55e","#3b82f6","#a855f7"].forEach((rc,ri)=>{const off=(ri-2.5)*2;ctx.globalAlpha=fd*0.6;ctx.strokeStyle=rc;ctx.lineWidth=1.5;ctx.lineCap="round";ctx.beginPath();ctx.moveTo(b.x1+nx*off,b.y1+ny*off);ctx.lineTo(b.x2+nx*off,b.y2+ny*off);ctx.stroke();});
    ctx.globalAlpha=fd*0.5;ctx.strokeStyle="#fff";ctx.lineWidth=1;ctx.beginPath();ctx.moveTo(b.x1,b.y1);ctx.lineTo(b.x2,b.y2);ctx.stroke();
  } else if(b.style==="psi"){
    const dx=b.x2-b.x1,dy=b.y2-b.y1,len=Math.sqrt(dx*dx+dy*dy)||1,nx=-dy/len,ny=dx/len;
    for(let wv=0;wv<3;wv++){ctx.globalAlpha=fd*(0.3-wv*0.08);ctx.strokeStyle=b.color;ctx.lineWidth=2-wv*0.4;ctx.lineCap="round";ctx.beginPath();for(let t=0;t<=1;t+=0.02){const px=b.x1+dx*t,py=b.y1+dy*t,wave=Math.sin(t*12+now*0.005+wv*2)*(6+wv*3);t===0?ctx.moveTo(px+nx*wave,py+ny*wave):ctx.lineTo(px+nx*wave,py+ny*wave);}ctx.stroke();}
  } else if(b.style==="frost"){
    ctx.globalAlpha=fd*0.3;ctx.strokeStyle="#a5f3fc";ctx.lineWidth=b.width*3;ctx.lineCap="round";ctx.beginPath();ctx.moveTo(b.x1,b.y1);ctx.lineTo(b.x2,b.y2);ctx.stroke();
    ctx.globalAlpha=fd*0.9;ctx.strokeStyle="#e0f2fe";ctx.lineWidth=b.width*0.8;ctx.beginPath();ctx.moveTo(b.x1,b.y1);ctx.lineTo(b.x2,b.y2);ctx.stroke();
    const bdx=b.x2-b.x1,bdy=b.y2-b.y1;for(let br=0;br<3;br++){const t=0.25+br*0.25,bx=b.x1+bdx*t,by=b.y1+bdy*t,ba=Math.atan2(bdy,bdx)+(br%2===0?0.7:-0.7),bl=15+br*5;ctx.globalAlpha=fd*0.5;ctx.lineWidth=1;ctx.beginPath();ctx.moveTo(bx,by);ctx.lineTo(bx+Math.cos(ba)*bl,by+Math.sin(ba)*bl);ctx.stroke();}
  } else {
    // "energy" default — halo + core + crackle
    ctx.globalAlpha=fd*0.2;ctx.strokeStyle=b.color;ctx.lineWidth=b.width*4;ctx.lineCap="round";ctx.beginPath();ctx.moveTo(b.x1,b.y1);ctx.lineTo(b.x2,b.y2);ctx.stroke();
    ctx.globalAlpha=fd*0.8*(0.8+Math.sin(now*0.02)*0.2);ctx.strokeStyle=b.color;ctx.lineWidth=b.width;ctx.beginPath();ctx.moveTo(b.x1,b.y1);ctx.lineTo(b.x2,b.y2);ctx.stroke();
    ctx.globalAlpha=fd*0.9;ctx.strokeStyle="#fff";ctx.lineWidth=Math.max(1,b.width*0.3);ctx.beginPath();ctx.moveTo(b.x1,b.y1);ctx.lineTo(b.x2,b.y2);ctx.stroke();
    const edx=b.x2-b.x1,edy=b.y2-b.y1,elen=Math.sqrt(edx*edx+edy*edy)||1,enx=-edy/elen,eny=edx/elen;ctx.globalAlpha=fd*0.4;ctx.strokeStyle=b.color;ctx.lineWidth=1;for(let cr=0;cr<4;cr++){const ct=Math.random(),cpx=b.x1+edx*ct,cpy=b.y1+edy*ct,cd=(Math.random()>0.5?1:-1)*(3+Math.random()*8);ctx.beginPath();ctx.moveTo(cpx,cpy);ctx.lineTo(cpx+enx*cd,cpy+eny*cd);ctx.stroke();}
  }
}

// === L5: Particles (type-switched) ===
ctx.globalCompositeOperation="lighter";
// v5.1 — Particle cap: evict oldest when over limit
const parts=particlesRef.current;
if(parts.length>PARTICLE_CAP){parts.splice(0,parts.length-PARTICLE_CAP);}
for(let i=parts.length-1;i>=0;i--){const p=parts[i],age=(now-p.born)/p.life;if(age>=1){parts.splice(i,1);continue;}const px=p.x+p.dx*age,py=p.y+p.dy*age,fd=(1-age);
  switch(p.type){
    case "fire":{const sz=p.size*(1-age*0.3)*(0.8+Math.random()*0.4),fy=py-sz*0.6;ctx.globalAlpha=fd*fd*0.7;const fg=ctx.createRadialGradient(px,fy+sz*0.3,0,px,fy,sz*1.8);if(age<0.3){fg.addColorStop(0,"#fef3c7");fg.addColorStop(0.3,"#fbbf24");fg.addColorStop(0.7,"#fb923c");fg.addColorStop(1,"transparent");}else if(age<0.7){fg.addColorStop(0,"#fbbf24");fg.addColorStop(0.3,"#fb923c");fg.addColorStop(0.7,"#dc2626");fg.addColorStop(1,"transparent");}else{fg.addColorStop(0,"#fb923c80");fg.addColorStop(0.4,"#dc262640");fg.addColorStop(1,"transparent");}ctx.fillStyle=fg;ctx.beginPath();ctx.arc(px,fy,sz*1.8,0,Math.PI*2);ctx.fill();break;}
    case "ice":{const sz=p.size*(0.8+Math.sin(now*0.005+i)*0.2),sides=p.extra?.sides||5,spin=(p.extra?.spin||0)*age*2;ctx.globalAlpha=fd*0.3;ctx.fillStyle=p.color+"40";ctx.beginPath();ctx.arc(px,py,sz*2,0,Math.PI*2);ctx.fill();ctx.globalAlpha=fd*0.7;ctx.fillStyle=p.color;ctx.beginPath();for(let s=0;s<=sides;s++){const a=(Math.PI*2*s)/sides+spin;ctx.lineTo(px+Math.cos(a)*sz,py+Math.sin(a)*sz);}ctx.fill();ctx.globalAlpha=fd*0.9;ctx.strokeStyle="#e0f2fe";ctx.lineWidth=0.8;ctx.beginPath();for(let s=0;s<=sides;s++){const a=(Math.PI*2*s)/sides+spin;ctx.lineTo(px+Math.cos(a)*sz,py+Math.sin(a)*sz);}ctx.stroke();const specA=(Math.PI*2*Math.floor(now*0.003%sides))/sides+spin;ctx.globalAlpha=fd*0.5;ctx.fillStyle="#fff";ctx.beginPath();ctx.arc(px+Math.cos(specA)*sz,py+Math.sin(specA)*sz,1.5,0,Math.PI*2);ctx.fill();break;}
    case "lightning":{const pts=p.extra?.points;if(!pts)break;const w=p.extra?.width||2;ctx.globalAlpha=fd*0.4;ctx.strokeStyle=p.color;ctx.lineWidth=w*3;ctx.lineCap="round";ctx.lineJoin="round";ctx.beginPath();pts.forEach((pt,pi)=>{const jx=pi>0&&pi<pts.length-1?(Math.random()-0.5)*3:0;pi===0?ctx.moveTo(pt.x+jx,pt.y+(Math.random()-0.5)*3):ctx.lineTo(pt.x+jx,pt.y+(Math.random()-0.5)*3);});ctx.stroke();ctx.globalAlpha=fd*0.95;ctx.strokeStyle="#fff";ctx.lineWidth=Math.max(0.5,w*0.5);ctx.beginPath();pts.forEach((pt,pi)=>{pi===0?ctx.moveTo(pt.x,pt.y):ctx.lineTo(pt.x,pt.y);});ctx.stroke();break;}
    case "spark":{const trail=p.extra?.trail;if(trail){trail.push({x:px,y:py});if(trail.length>4)trail.shift();}ctx.globalAlpha=fd*0.9;ctx.fillStyle="#fff";ctx.beginPath();ctx.arc(px,py,p.size,0,Math.PI*2);ctx.fill();if(trail){for(let t=0;t<trail.length;t++){ctx.globalAlpha=fd*(t/trail.length)*0.4;ctx.fillStyle=p.color;ctx.beginPath();ctx.arc(trail[t].x,trail[t].y,p.size*(t/trail.length),0,Math.PI*2);ctx.fill();}}break;}
    case "smoke":{const expand=1+age*0.8,sz=p.size*expand;ctx.globalAlpha=fd*fd*0.12;ctx.globalCompositeOperation="source-over";const sg=ctx.createRadialGradient(px,py,0,px,py,sz);sg.addColorStop(0,p.color+"40");sg.addColorStop(1,"transparent");ctx.fillStyle=sg;ctx.beginPath();ctx.arc(px,py,sz,0,Math.PI*2);ctx.fill();ctx.globalCompositeOperation="lighter";break;}
    case "slash":{const ang=p.extra?.angle||0,len=p.extra?.len||60,w=p.extra?.width||4;for(let m=0;m<3;m++){const t=1-age*2;if(t<=0)break;ctx.globalAlpha=fd*(1-m*0.3)*t;ctx.strokeStyle=m===0?"#fff":p.color;ctx.lineWidth=w*(1-m*0.2)*(1-age);ctx.lineCap="round";ctx.beginPath();ctx.arc(px-m*2,py-m*2,len*0.5,ang-0.6,ang+0.6);ctx.stroke();}break;}
    case "psi":{const freq=p.extra?.freq||3,amp=p.extra?.amp||5,phase=p.extra?.phase||0,dx=p.dx,dy=p.dy,len=Math.sqrt(dx*dx+dy*dy)||1,nnx=-dy/len,nny=dx/len;ctx.globalAlpha=fd*0.5;ctx.strokeStyle=p.color;ctx.lineWidth=2;ctx.lineCap="round";ctx.beginPath();for(let t=0;t<=1;t+=0.05){const tx=p.x+dx*age*t,ty=p.y+dy*age*t,wave=Math.sin(t*freq*Math.PI+phase+now*0.004)*amp*(1-t);t===0?ctx.moveTo(tx+nnx*wave,ty+nny*wave):ctx.lineTo(tx+nnx*wave,ty+nny*wave);}ctx.stroke();ctx.globalAlpha=fd*0.3;ctx.fillStyle=p.color;ctx.beginPath();ctx.arc(px,py,p.size*1.5,0,Math.PI*2);ctx.fill();break;}
    case "shard":{const sides=3+Math.floor(p.extra?.sides||0),spin=(p.extra?.spin||2)*age*4,sz=p.size;ctx.globalAlpha=fd*0.7;ctx.fillStyle=p.color;ctx.strokeStyle="#fff";ctx.lineWidth=0.5;ctx.save();ctx.translate(px,py);ctx.rotate(spin);ctx.beginPath();for(let s=0;s<=sides;s++){const a=(Math.PI*2*s)/sides;ctx.lineTo(Math.cos(a)*sz,Math.sin(a)*sz);}ctx.fill();ctx.stroke();ctx.restore();if(Math.abs(spin%(Math.PI*2))<0.3){ctx.globalAlpha=fd*0.5;ctx.fillStyle="#fff";ctx.beginPath();ctx.arc(px,py,sz*0.5,0,Math.PI*2);ctx.fill();}break;}
    case "ember":{const drift=Math.sin(now*0.002+i*1.3)*3;ctx.globalAlpha=fd*0.6*(0.5+Math.sin(now*0.004+i)*0.5);const eg=ctx.createRadialGradient(px+drift,py,0,px+drift,py,p.size*3);eg.addColorStop(0,"#fbbf24");eg.addColorStop(0.3,p.color);eg.addColorStop(1,"transparent");ctx.fillStyle=eg;ctx.beginPath();ctx.arc(px+drift,py,p.size*3,0,Math.PI*2);ctx.fill();break;}
    case "ring":{const ex=p.extra;if(!ex)break;const angle=ex.angle+ex.speed*age,rx=ex.cx+Math.cos(angle)*ex.radius*(1+age*0.3),ry=ex.cy+Math.sin(angle)*ex.radius*(1+age*0.3);ctx.globalAlpha=fd*0.6;ctx.strokeStyle=p.color;ctx.lineWidth=3*(1-age*0.5);ctx.lineCap="round";ctx.beginPath();ctx.arc(ex.cx,ex.cy,ex.radius*(1+age*0.3),angle-ex.arcLen/2,angle+ex.arcLen/2);ctx.stroke();ctx.globalAlpha=fd*0.8;ctx.fillStyle="#fff";ctx.beginPath();ctx.arc(rx,ry,2,0,Math.PI*2);ctx.fill();break;}
    case "sonic":{const r=p.size*(1+age*8);for(let a=0;a<3;a++){ctx.globalAlpha=fd*(1-a*0.25)*0.4;ctx.strokeStyle=p.color;ctx.lineWidth=2-a*0.5;ctx.lineCap="round";ctx.beginPath();ctx.arc(p.x,p.y,r+a*4,-0.5+age,0.5+age);ctx.stroke();}break;}
    default:{ctx.globalAlpha=fd*fd*0.15;const hg=ctx.createRadialGradient(px,py,0,px,py,p.size*4);hg.addColorStop(0,p.color);hg.addColorStop(1,"transparent");ctx.fillStyle=hg;ctx.beginPath();ctx.arc(px,py,p.size*4,0,Math.PI*2);ctx.fill();ctx.globalAlpha=fd*fd*0.7;const cg=ctx.createRadialGradient(px,py,0,px,py,p.size*1.2);cg.addColorStop(0,"#fff");cg.addColorStop(0.3,p.color);cg.addColorStop(1,"transparent");ctx.fillStyle=cg;ctx.beginPath();ctx.arc(px,py,p.size*1.2,0,Math.PI*2);ctx.fill();}
  }
}

// === L6: Impacts ===
ctx.globalCompositeOperation="lighter";
// v5.1 — Impact cap
const imps=impactsRef.current;
if(imps.length>IMPACT_CAP){imps.splice(0,imps.length-IMPACT_CAP);}
for(let i=imps.length-1;i>=0;i--){const imp=imps[i],age=(now-imp.born)/imp.life;if(age>=1){imps.splice(i,1);continue;}const fd=1-age;
  if(imp.type==="shockwave"){const r=imp.maxR*age;ctx.globalAlpha=fd*fd*0.5;ctx.strokeStyle=imp.color;ctx.lineWidth=3*(1-age);ctx.beginPath();ctx.arc(imp.x,imp.y,r,0,Math.PI*2);ctx.stroke();ctx.globalAlpha=fd*0.3;ctx.strokeStyle="#fff";ctx.lineWidth=1;ctx.beginPath();ctx.arc(imp.x,imp.y,r*0.8,0,Math.PI*2);ctx.stroke();}
  if(imp.type==="slashArc"){for(let m=0;m<3;m++){ctx.globalAlpha=fd*(1-m*0.3);ctx.strokeStyle=m===0?"#fff":imp.color;ctx.lineWidth=(4-m*1.2)*(1-age);ctx.lineCap="round";ctx.beginPath();ctx.arc(imp.x+m*1.5,imp.y+m*1.5,35,imp.angle-0.7,imp.angle+0.7);ctx.stroke();}}
  if(imp.type==="flash"){ctx.globalAlpha=fd*0.6;const fg=ctx.createRadialGradient(imp.x,imp.y,0,imp.x,imp.y,imp.radius);fg.addColorStop(0,"#fff");fg.addColorStop(0.5,"#ffffff80");fg.addColorStop(1,"transparent");ctx.fillStyle=fg;ctx.beginPath();ctx.arc(imp.x,imp.y,imp.radius,0,Math.PI*2);ctx.fill();}
}

// === L7: Magneto field lines ===
ctx.globalCompositeOperation="source-over";ctx.globalAlpha=1;
if(activePwrs&&(activePwrs.includes("magnetism")||activePwrs.includes("magnetism_g"))){const mColor=activePwrs.includes("magnetism")?"#a855f7":"#22c55e";const metals=(objs||[]).filter(o=>o.mat==="metal");const mp={x:w/2,y:h/2};ctx.strokeStyle=mColor;ctx.lineWidth=1;ctx.setLineDash([4,4]);metals.forEach(m=>{const mx=m.x+m.w/2,my=m.y+m.h/2;ctx.globalAlpha=0.3;ctx.beginPath();ctx.moveTo(mx,my);ctx.quadraticCurveTo((mx+mp.x)/2+Math.sin(now*0.003+mx)*15,(my+mp.y)/2+Math.cos(now*0.003+my)*15,mp.x,mp.y);ctx.stroke();metals.forEach(m2=>{if(m2.id<=m.id)return;const d=Math.sqrt((m2.x-m.x)**2+(m2.y-m.y)**2);if(d<250){ctx.globalAlpha=0.15*(1-d/250);ctx.beginPath();ctx.moveTo(mx,my);ctx.lineTo(m2.x+m2.w/2,m2.y+m2.h/2);ctx.stroke();}});});ctx.setLineDash([]);}

// === L8: Post effects ===
const pfx=postFxRef.current;
for(let i=pfx.length-1;i>=0;i--){const p=pfx[i],age=(now-p.born)/p.life;if(age>=1){pfx.splice(i,1);continue;}const fd=1-age;
  if(p.type==="tint"){ctx.globalCompositeOperation="source-over";ctx.globalAlpha=fd*0.12;ctx.fillStyle=p.color||"#f59e0b";ctx.fillRect(0,0,w,h);}
  if(p.type==="vignette"){ctx.globalCompositeOperation="source-over";ctx.globalAlpha=fd*0.3;const vg=ctx.createRadialGradient(w/2,h/2,w*0.3,w/2,h/2,w*0.7);vg.addColorStop(0,"transparent");vg.addColorStop(1,"#000");ctx.fillStyle=vg;ctx.fillRect(0,0,w,h);}
}

// === L9: Floating damage numbers ===
ctx.globalCompositeOperation="source-over";
const fdmg=floatingDmgRef.current;
for(let i=fdmg.length-1;i>=0;i--){const d=fdmg[i],age=(now-d.born)/800;if(age>=1){fdmg.splice(i,1);continue;}ctx.globalAlpha=(1-age)*(1-age);ctx.fillStyle=d.color;ctx.font=`bold ${Math.max(8,12-age*4)}px sans-serif`;ctx.textAlign="center";ctx.fillText(d.dmg,d.x,d.y-age*30);}

// === L10: Phoenix aura ===
const pxLvl=phoenixRef.current?.level||1;
if(pxLvl>=3){ctx.globalCompositeOperation="source-over";ctx.globalAlpha=(pxLvl-2)*0.03;ctx.fillStyle=pxLvl>=5?"#dc2626":"#f59e0b";ctx.fillRect(0,0,w,h);}

// === v5.0 RENDER LAYERS ===

// Weather particles
ctx.globalCompositeOperation="source-over";
const _wp=weatherParticlesRef.current;
for(let i=0;i<_wp.length;i++){
  const p=_wp[i]; const age=(now-p.born)/p.life;
  ctx.globalAlpha=Math.max(0,(1-age)*0.6);
  ctx.fillStyle=p.color;
  if(weatherRef.current==="snow"){ctx.beginPath();ctx.arc(p.x,p.y,p.size,0,Math.PI*2);ctx.fill();}
  else{ctx.fillRect(p.x,p.y,p.size,p.size*3);}
}

// Terrain
const _terrain=terrainRef.current;
for(const t of _terrain){
  ctx.globalAlpha=0.85;
  ctx.fillStyle=TERRAIN_COLORS[t.material]||"#555";
  if(t.type==="ramp"){
    ctx.beginPath();
    if(t.angle>0){ctx.moveTo(t.x,t.y+t.h);ctx.lineTo(t.x+t.w,t.y+t.h);ctx.lineTo(t.x+t.w,t.y);}
    else{ctx.moveTo(t.x,t.y+t.h);ctx.lineTo(t.x+t.w,t.y+t.h);ctx.lineTo(t.x,t.y);}
    ctx.closePath();ctx.fill();
  } else {
    ctx.fillRect(t.x,t.y,t.w,t.h);
    if(t.type==="platform"){ctx.fillStyle="#fff";ctx.globalAlpha=0.2;ctx.fillRect(t.x,t.y,t.w,3);}
  }
}

// Joints
const _joints=jointsRef.current;
ctx.globalAlpha=0.7;ctx.globalCompositeOperation="source-over";
for(const j of _joints){
  const a=(objs||[]).find(o=>o.id===j.objA), b=(objs||[]).find(o=>o.id===j.objB);
  if(!a||!b) continue;
  const ax=a.x+a.w/2, ay=a.y+a.h/2, bx=b.x+b.w/2, by=b.y+b.h/2;
  ctx.strokeStyle=j.type==="rope"?"#a0845e":j.type==="spring"?"#22c55e":j.type==="rigid"?"#8b9daf":"#fbbf24";
  ctx.lineWidth=j.type==="rigid"?3:j.type==="spring"?2:1.5;
  if(j.type==="spring"){ctx.setLineDash([4,4]);ctx.beginPath();ctx.moveTo(ax,ay);ctx.lineTo(bx,by);ctx.stroke();ctx.setLineDash([]);}
  else if(j.type==="rope"){
    const sag=Math.min(30,Math.sqrt((bx-ax)**2+(by-ay)**2)*0.15);
    ctx.beginPath();ctx.moveTo(ax,ay);ctx.quadraticCurveTo((ax+bx)/2,(ay+by)/2+sag,bx,by);ctx.stroke();
  } else {ctx.beginPath();ctx.moveTo(ax,ay);ctx.lineTo(bx,by);ctx.stroke();}
  if(j.type==="hinge"){ctx.beginPath();ctx.arc(ax,ay,4,0,Math.PI*2);ctx.fillStyle="#fbbf24";ctx.fill();}
}

// Behavior indicators
if(objs) for(const o of objs){
  if(!o.behavior) continue;
  const bt=BEHAVIOR_TYPES[o.behavior]; if(!bt) continue;
  ctx.globalAlpha=0.5;ctx.font="10px sans-serif";ctx.textAlign="center";
  ctx.fillStyle=bt.color;ctx.fillText(bt.icon,o.x+o.w/2,o.y-4);
}

// Dynamic lighting
ctx.globalCompositeOperation="screen";
if(objs) for(const o of objs){
  let lc=null, lr=0;
  if(o.statuses.some(s=>s.type==="burning")){lc="rgba(255,150,50,0.07)";lr=80;}
  else if(o.mat==="energy"){lc="rgba(232,196,74,0.04)";lr=60;}
  if(lc){ctx.globalAlpha=1;const grd=ctx.createRadialGradient(o.x+o.w/2,o.y+o.h/2,0,o.x+o.w/2,o.y+o.h/2,lr);grd.addColorStop(0,lc);grd.addColorStop(1,"rgba(0,0,0,0)");ctx.fillStyle=grd;ctx.fillRect(o.x+o.w/2-lr,o.y+o.h/2-lr,lr*2,lr*2);}
}
ctx.globalCompositeOperation="source-over";

// Material decorations (v5.0)
ctx.globalCompositeOperation="source-over";
if(objs) for(const o of objs){
  if(o.mat==="rubber"){ctx.globalAlpha=0.3;ctx.strokeStyle="#555";ctx.lineWidth=2;ctx.beginPath();ctx.roundRect?.(o.x+2,o.y+2,o.w-4,o.h-4,6);ctx.stroke();}
  if(o.mat==="crystal"){ctx.globalAlpha=0.25;for(let f=0;f<3;f++){ctx.strokeStyle=["#f0f","#0ff","#ff0"][f];ctx.lineWidth=0.5;ctx.beginPath();ctx.moveTo(o.x+o.w*0.3+f*o.w*0.2,o.y+o.h*0.2);ctx.lineTo(o.x+o.w*0.3+f*o.w*0.2+4,o.y+o.h*0.7);ctx.stroke();}}
  if(o.mat==="ice"){ctx.globalAlpha=0.3;ctx.fillStyle="#e0f0ff";ctx.fillRect(o.x+o.w*0.35,o.y+o.h*0.15,2,o.h*0.5);ctx.fillRect(o.x+o.w*0.55,o.y+o.h*0.25,1.5,o.h*0.4);}
  if(o.mat==="sand"){ctx.globalAlpha=0.25;ctx.fillStyle="#b89a60";for(let g=0;g<4;g++)ctx.fillRect(o.x+5+g*8,o.y+3+g*6,1.5,1.5);}
}

// Weather overlay
const _wType=weatherRef.current;
if(_wType==="storm"||_wType==="sandstorm"){
  ctx.globalAlpha=_wType==="storm"?0.05:0.1;
  ctx.fillStyle=_wType==="storm"?"#1e3a5f":"#d4b483";
  ctx.fillRect(0,0,w,h);
}
if(_wType==="storm"&&Math.random()<0.004){
  ctx.globalAlpha=0.25;ctx.fillStyle="#fff";ctx.fillRect(0,0,w,h);
}

// Scenario goal zone
const _sc=scenarioRef.current;
if(_sc?.goal?.zone){
  const z=_sc.goal.zone;
  ctx.globalAlpha=0.08;ctx.fillStyle="#22c55e";ctx.fillRect(z.x,0,z.w,h);
  ctx.globalAlpha=0.4;ctx.strokeStyle="#22c55e";ctx.lineWidth=2;ctx.setLineDash([8,4]);
  ctx.strokeRect(z.x,0,z.w,h);ctx.setLineDash([]);
}

ctx.globalAlpha=1;ctx.globalCompositeOperation="source-over";
```

}, []);
// ─── SCENE HELPERS ───
const getPos = useCallback((e) => {
const r=sceneRef.current?.getBoundingClientRect();
if(!r)return{x:0,y:0};
const t=e.touches?e.touches[0]:e;
return{x:t.clientX-r.left,y:t.clientY-r.top};
}, []);

const findObj = useCallback((x,y) => {
for(let i=objects.length-1;i>=0;i–) {
const o=objects[i], hw=(o.w*o.scale)/2, hh=(o.h*o.scale)/2;
const cx=o.x+o.w/2, cy=o.y+o.h/2;
if(x>=cx-hw-8&&x<=cx+hw+8&&y>=cy-hh-8&&y<=cy+hh+8)return o;
}
return null;
}, [objects]);

const togglePower = useCallback((id) => {
setBuildMode(false); // exit build mode when selecting power
if(onboardStep===1) setOnboardStep(2);
setPowers(prev => {
if(prev.includes(id)){sfxUI(500);return prev.filter(p=>p!==id);}
sfxUI(800);
if(prev.length>=2)return[prev[1],id];
return[…prev,id];
});
setTwoClickSrc(null);
}, [sfxUI]);

// ─── DISCOVERY SYSTEM ───
const recordDiscovery = useCallback((key) => {
if(onboardStepRef.current===2) setOnboardStep(3);
setDiscovered(prev => {
if (prev.has(key)) return prev;
const next = new Set(prev);
next.add(key);
discoveredRef.current = next;
// v5.1 — Batch discovery toasts
const label = key.replace(/^p:/,’’).replace(/+m:/,’ × ’).replace(/^combo:/,’⚡ ’).replace(/^destroy:/,’💥 ’).replace(/^status:/,’🔄 ‘);
toastQueueRef.current.push(label);
if (!toastFlushTimerRef.current) {
toastFlushTimerRef.current = setTimeout(() => {
const q = toastQueueRef.current.splice(0, toastQueueRef.current.length);
if (q.length === 0) { toastFlushTimerRef.current = null; return; }
const text = q.length <= 2 ? q.join(’ • ‘) : q.slice(0, 2).join(’ • ’) + ` +${q.length - 2} more`;
setToasts(t => […t.slice(-2), { id: Date.now() + Math.random(), text, born: Date.now() }]);
toastFlushTimerRef.current = null;
}, 300);
}
if(!mutedRef.current) sfxChime();
return next;
});
}, [sfxChime]);

// ─── ZONE SYSTEM ───
const createZone = useCallback((x, y, type, radius=2, intensity=0.8) => {
const zones = zonesRef.current;
const cw = bounds.w/ZONE_COLS, ch = bounds.h/ZONE_ROWS;
const cc = Math.floor(x/cw), cr = Math.floor(y/ch);
for (let dr=-radius;dr<=radius;dr++) for (let dc=-radius;dc<=radius;dc++) {
const r=cr+dr, c=cc+dc;
if (r<0||r>=ZONE_ROWS||c<0||c>=ZONE_COLS) continue;
const dist = Math.sqrt(dr*dr+dc*dc);
if (dist > radius) continue;
const falloff = 1 - dist/radius;
const existing = zones[r][c];
// Fire + ice cancel
if (existing && ((existing.type===“fire”&&type===“ice”)||(existing.type===“ice”&&type===“fire”))) {
zones[r][c] = null;
spawn(c*cw+cw/2, r*ch+ch/2, “#fff”, 4, 20, 300, [2,4]);
continue;
}
zones[r][c] = { type, intensity: Math.min(1, intensity*falloff + (existing?.type===type ? existing.intensity*0.3 : 0)), source:type }; activeZonesRef.current.add(r*ZONE_COLS+c);
}
// Register zone as active effect for power-on-power interactions
if(activeEffectsRef.current.length>=EFFECT_CAP)activeEffectsRef.current.shift();
activeEffectsRef.current.push({type:“zone”,cx:x,cy:y,radius:radius*Math.max(bounds.w/ZONE_COLS,bounds.h/ZONE_ROWS),element:type,born:Date.now(),life:2000,id:Date.now()+Math.random()});
}, [bounds, spawn]);

// ─── WEATHER ENGINE (v5.0) ───
const startWeatherCycle = useCallback(() => {
if(weatherTimerRef.current) clearInterval(weatherTimerRef.current);
const cycle = () => {
const E = envRef.current;
const options = E?.weather || [“clear”];
const pick = options[Math.floor(Math.random()*options.length)];
setWeather(pick);
};
cycle();
weatherTimerRef.current = setInterval(cycle, 35000 + Math.random()*25000);
}, []);

useEffect(() => { startWeatherCycle(); return () => { if(weatherTimerRef.current) clearInterval(weatherTimerRef.current); }; }, [env, startWeatherCycle]);

const tickWeather = useCallback((now, bw, bh) => {
const w = weatherRef.current;
const wt = WEATHER_TYPES[w]; if(!wt || wt.particles===0) return;
const arr = weatherParticlesRef.current;
// Spawn
for(let i=0;i<wt.particles;i++){
if(arr.length > 600) break;
const x = Math.random()*bw + (wt.force?.x||0)*-50;
const y = w===“sandstorm”?Math.random()*bh:-10;
arr.push({x, y, dx:(wt.force?.x||0)+(Math.random()-0.5)*2, dy:wt.particleDy||2, color:wt.particleColor||”#fff”, life:w===“snow”?4000:1500, born:now, size:w===“snow”?2:1});
}
// Update + cleanup
for(let i=arr.length-1;i>=0;i–){
const p=arr[i];
p.x+=p.dx; p.y+=p.dy;
if(w===“snow”) { p.x += Math.sin(now*0.002+p.x*0.01)*0.5; p.dy*=0.998; }
if(p.y>bh+10||p.x<-20||p.x>bw+20||now-p.born>p.life) arr.splice(i,1);
}
}, []);

const tickMeteors = useCallback((now, bw, bh) => {
if(weatherRef.current!==“meteor_shower”) return;
if(Math.random()>0.008) return; // ~0.5/sec
const mx=Math.random()*bw, my=-30;
const id=Date.now()+Math.random();
setObjects(prev=>{
if(prev.length>=50)return prev;
return[…prev,{id,x:mx,y:my,w:25,h:25,mat:“stone”,label:“Meteor”,shape:“circle”,origX:mx,origY:my,origMat:“stone”,scale:1,rotation:0,opacity:1,effects:[],statuses:[{type:“burning”,ticksLeft:8}],vx:(Math.random()-0.5)*4,vy:8+Math.random()*6,hp:30,maxHp:30,fragile:true,temp:800,mass:25*25*2.2*0.01,resting:0,trailUntil:now+3000}];
});
}, []);

// ─── TERRAIN COLLISION (v5.0) ───
const checkTerrainCollision = useCallback((o) => {
const terrain = terrainRef.current;
let nx=o.x,ny=o.y,nvx=o.vx,nvy=o.vy;
for(const t of terrain){
if(t.type===“platform”){
if(o.y+o.h<=t.y+4 && o.y+o.h+nvy>=t.y && o.x+o.w>t.x && o.x<t.x+t.w){
ny=t.y-o.h; nvy=Math.abs(nvy)>1?-nvy*(MAT_PROPS[o.mat]?.bounce||0.2):0;
nvx*=(t.friction||0.85);
}
} else if(t.type===“wall”){
const ox=Math.min(nx+o.w,t.x+t.w)-Math.max(nx,t.x);
const oy=Math.min(ny+o.h,t.y+t.h)-Math.max(ny,t.y);
if(ox>0&&oy>0){
if(ox<oy){ if(nx<t.x) nx=t.x-o.w; else nx=t.x+t.w; nvx=-nvx*0.3; }
else { if(ny<t.y) { ny=t.y-o.h; nvy=Math.abs(nvy)>1?-nvy*0.2:0; } else { ny=t.y+t.h; nvy=Math.abs(nvy)*0.5; } }
}
} else if(t.type===“ramp”){
const rx=nx+o.w/2-t.x, ry=ny+o.h-t.y;
if(rx>=0&&rx<=t.w&&ry>=0){
const rampY=t.h*(1-rx/t.w)*(t.angle>0?1:-1);
const surfaceY=t.y+(t.angle>0?t.h-rampY:rampY);
if(ny+o.h>surfaceY&&ny+o.h<surfaceY+10){
ny=surfaceY-o.h; nvy=0; nvx*=0.92;
if(t.angle!==0) nvx+=t.angle>0?0.3:-0.3;
}
}
}
}
return {x:nx,y:ny,vx:nvx,vy:nvy};
}, []);

// ─── JOINT PHYSICS (v5.0) ───
const tickJoints = useCallback(() => {
const joints = jointsRef.current;
if(joints.length===0) return;
setObjects(prev => {
const objs=[…prev.map(o=>({…o}))];
const byId=new Map(objs.map(o=>[o.id,o]));
for(const j of joints){
const a=byId.get(j.objA), b=byId.get(j.objB);
if(!a||!b){ j._dead=true; continue; }
const acx=a.x+a.w/2, acy=a.y+a.h/2, bcx=b.x+b.w/2, bcy=b.y+b.h/2;
const dx=bcx-acx, dy=bcy-acy, dist=Math.sqrt(dx*dx+dy*dy)||1;
const nx=dx/dist, ny=dy/dist;
if(j.type===“rope”&&dist<=j.length) continue; // slack rope
const diff=dist-j.length;
const stiff=j.type===“spring”?0.15:j.type===“rope”?0.5:0.9;
const correction=diff*stiff*0.5;
const ma=a.mass||1, mb=b.mass||1, tm=ma+mb;
a.x+=nx*correction*(mb/tm); a.y+=ny*correction*(mb/tm);
b.x-=nx*correction*(ma/tm); b.y-=ny*correction*(ma/tm);
// Velocity damping
const relVn=(a.vx-b.vx)*nx+(a.vy-b.vy)*ny;
const damp=(j.damping||0.1)*relVn*0.5;
a.vx-=nx*damp*(mb/tm); a.vy-=ny*damp*(mb/tm);
b.vx+=nx*damp*(ma/tm); b.vy+=ny*damp*(ma/tm);
// Rigid body: also constrain angular offset
if(j.type===“rigid”){
const avgVx=(a.vx*ma+b.vx*mb)/tm, avgVy=(a.vy*ma+b.vy*mb)/tm;
a.vx=a.vx*0.8+avgVx*0.2; a.vy=a.vy*0.8+avgVy*0.2;
b.vx=b.vx*0.8+avgVx*0.2; b.vy=b.vy*0.8+avgVy*0.2;
}
// Breaking force check
const force=Math.abs(diff)*stiff*50;
if(j.breakForce && force>j.breakForce) j._dead=true;
}
// Remove dead joints
jointsRef.current=joints.filter(j=>!j._dead);
return objs;
});
}, []);

// ─── BEHAVIOR TICK (v5.0) ───
const tickBehaviors = useCallback((now) => {
setObjects(prev => {
let changed=false;
const next=prev.map(o => {
if(!o.behavior) return o;
const bt=BEHAVIOR_TYPES[o.behavior];
if(!bt) return o;
let nvx=o.vx, nvy=o.vy;
if(o.behavior===“sentinel”){
const cx=o.origX||o.x, range=80;
const target=cx+Math.sin(now*0.002*(bt.speed||1))*range;
nvx+=(target-o.x)*0.05;
// Push intruders
prev.forEach(other=>{
if(other.id===o.id||other.behavior) return;
const d=Math.sqrt((other.x-o.x)**2+(other.y-o.y)**2);
if(d<60&&d>1){const f=3/d;nvx+=(o.x-other.x)*f*0.01;} // slight nudge
});
changed=true;
} else if(o.behavior===“seeker”){
let nearest=null, nd=Infinity;
prev.forEach(other=>{
if(other.id===o.id||other.behavior===“seeker”) return;
const d=(other.x-o.x)**2+(other.y-o.y)**2;
if(d<nd){nd=d;nearest=other;}
});
if(nearest){
const dx=nearest.x-o.x, dy=nearest.y-o.y, dist=Math.sqrt(dx*dx+dy*dy)||1;
nvx+=(dx/dist)*bt.speed*0.3; nvy+=(dy/dist)*bt.speed*0.3;
changed=true;
}
} else if(o.behavior===“orbiter”){
const cx=o.origX||o.x, cy=o.origY||o.y, r=60;
const angle=now*0.003*bt.speed;
const tx=cx+Math.cos(angle)*r, ty=cy+Math.sin(angle)*r;
nvx+=(tx-o.x)*0.08; nvy+=(ty-o.y)*0.08;
changed=true;
} else if(o.behavior===“repeller”){
// Forces accumulated via repelForces map below
} else if(o.behavior===“mimic”){
let nearest=null, nd=Infinity;
prev.forEach(other=>{
if(other.id===o.id) return;
const d=(other.x-o.x)**2+(other.y-o.y)**2;
if(d<nd){nd=d;nearest=other;}
});
if(nearest){ nvx=nearest.vx*0.8; nvy=nearest.vy*0.8; changed=true; }
}
if(nvx!==o.vx||nvy!==o.vy) return{…o,vx:nvx,vy:nvy};
return o;
});
// Repeller pass — accumulate forces immutably
const repellers=next.filter(o=>o.behavior===“repeller”);
if(repellers.length>0){
const forces=new Map();
for(const rep of repellers){
const bt=BEHAVIOR_TYPES.repeller;
const range=bt?.range||100, r2=range*range;
for(const other of next){
if(other.id===rep.id) continue;
const dx=other.x-rep.x, dy=other.y-rep.y, dd=dx*dx+dy*dy;
if(dd<r2&&dd>1){
const dist=Math.sqrt(dd);
const f=forces.get(other.id)||{dvx:0,dvy:0};
f.dvx+=(dx/dist)*1.5; f.dvy+=(dy/dist)*1.5;
forces.set(other.id,f);
}
}
}
if(forces.size>0){
changed=true;
return next.map(o=>{const f=forces.get(o.id);return f?{…o,vx:o.vx+f.dvx,vy:o.vy+f.dvy}:o;});
}
}
return changed?next:prev;
});
}, []);

// ─── SCENARIO ENGINE (v5.0) ───
const checkScenarioGoal = useCallback(() => {
const sc=scenarioRef.current; if(!sc) return;
const objs=objectsRef.current;
const goal=sc.goal;
let complete=false;
if(goal.type===“destroy_all”) complete=objs.filter(o=>o.label!==“Debris”).length===0;
else if(goal.type===“freeze_all”) complete=objs.filter(o=>o.mat===goal.material).every(o=>o.statuses.some(s=>s.type===“frozen”));
else if(goal.type===“chain_reaction”){ const chainMax=objs.reduce((m,o)=>Math.max(m,o._chainIdx||0),0); complete=chainMax>=goal.count; }
else if(goal.type===“protect”){ const target=objs.find(o=>o.mat===goal.material); complete=!target?false:((Date.now()-scenarioStartRef.current)/1000>=goal.time); if(!target&&!complete) setScenarioResult({win:false,reason:“Target destroyed!”}); }
else if(goal.type===“zone_count”){ const zoneTypes=new Set(); const zones=zonesRef.current; for(let r=0;r<zones.length;r++) for(let c=0;c<zones[r].length;c++) if(zones[r][c]) zoneTypes.add(zones[r][c].type); complete=zoneTypes.size>=goal.count; }
else if(goal.type===“material_chain”){ const destroyed=[…discoveredRef.current].filter(k=>k.startsWith(“destroy:”)); const mats=new Set(destroyed.map(k=>k.split(”+m:”)[1]).filter(Boolean)); complete=mats.size>=goal.count; }
else if(goal.type===“combo_count”){ const combos=[…discoveredRef.current].filter(k=>k.startsWith(“combo:”)); complete=combos.length>=goal.count; }
else if(goal.type===“reach_target”){ const targets=objs.filter(o=>o.mat===goal.material); complete=targets.length>0&&targets.every(o=>o.x>=goal.zone.x&&o.x+o.w<=goal.zone.x+goal.zone.w); }
if(complete && !scenarioRef.current._done){
scenarioRef.current._done=true;
const elapsed=(Date.now()-scenarioStartRef.current)/1000;
const stars=elapsed<=goal.par_time?3:elapsed<=goal.par_time*1.5?2:1;
setScenarioResult({win:true,time:elapsed,stars});
}
}, []);

const startScenario = useCallback((sc) => {
const fullObjs=sc.objects.map((o,i)=>({
…o, id:Date.now()+i+Math.random(), origX:o.x, origY:o.y, origMat:o.mat,
scale:1, rotation:0, opacity:1, effects:[], statuses:o.statuses||[], vx:0, vy:0,
hp:MAT_HP[o.mat]||100, maxHp:MAT_HP[o.mat]||100, fragile:!!MAT_FRAGILE[o.mat],
temp:20, mass:(o.w||40)*(o.h||40)*(MAT_PROPS[o.mat]?.density||1)*0.01, resting:0
}));
setObjects(fullObjs);
if(sc.env) setEnv(sc.env);
setScenario(sc);
scenarioRef.current={…sc,_done:false};
scenarioStartRef.current=Date.now();
setScenarioResult(null);
terrainRef.current=sc.terrain||[];
jointsRef.current=sc.joints||[];
}, []);

// ─── EFFECT HELPERS ───
const addEffect = useCallback((objId, effect) => {
const eid = nextEffectId();
const tagged = {…effect, _eid:eid};
setObjects(prev => prev.map(o => o.id===objId ? {…o, effects:[…o.effects,tagged]} : o));
setTimeout(() => setObjects(prev => prev.map(o => o.id===objId ? {…o, effects:o.effects.filter(e=>e._eid!==eid)} : o)), effect.dur||1000);
}, [nextEffectId]);

const applyStatus = useCallback((objId, type, ticks=6) => {
playStatus(type);
setObjects(prev => prev.map(o => {
if (o.id!==objId) return o;
// Status interactions
if (type===“burning” && o.statuses.some(s=>s.type===“frozen”)) return {…o, statuses:o.statuses.filter(s=>s.type!==“frozen”)};
if (type===“frozen” && o.statuses.some(s=>s.type===“burning”)) return {…o, statuses:o.statuses.filter(s=>s.type!==“burning”)};
if (o.statuses.some(s=>s.type===“phased”) && type!==“phased”) return o; // phased blocks new statuses
const existing = o.statuses.find(s=>s.type===type);
if (existing) return {…o, statuses:o.statuses.map(s=>s.type===type?{…s,ticksLeft:Math.max(s.ticksLeft,ticks)}:s)};
return {…o, statuses:[…o.statuses,{type,ticksLeft:ticks}]};
}));
}, [playStatus]);

// ─── DAMAGE & DESTRUCTION ───
const dealDamage = useCallback((objId, amount) => {
// v5.1 — Queue damage instead of immediate setObjects
damageQueueRef.current.push({ id: objId, amount });
}, []); // v5.1 — only pushes to ref, no deps needed

// v5.1 — Flush damage queue: apply all queued damage in single setObjects
const flushDamageQueue = useCallback(() => {
const queue = damageQueueRef.current;
if (queue.length === 0) return;
const batch = queue.splice(0, queue.length);
setObjects(prev => {
let next = […prev];
const debris = [];
const dmgMap = new Map();
for (const { id, amount } of batch) {
dmgMap.set(id, (dmgMap.get(id) || 0) + amount);
}
for (const [objId, totalAmt] of dmgMap) {
const idx = next.findIndex(o => o.id === objId);
if (idx === -1) continue;
const target = next[idx];
let dmg = totalAmt;
if (target.statuses.some(s => s.type === “armored”)) dmg *= 0.5;
if (target.statuses.some(s => s.type === “phased”)) dmg = 0;
if (target.statuses.some(s => s.type === “frozen”)) dmg *= 2;
const cx = target.x + target.w / 2, cy = target.y + target.h / 2;
if (dmg > 0) { spawnHitFlash(cx, cy, dmg); spawnDmgNumber(cx, cy, dmg); playImpact(dmg); }
const newHp = Math.max(0, target.hp - dmg);
if (newHp <= 0) {
playDestroy(target.mat);
const matColor = MAT_COLORS[target.mat] || “#fff”;
const arr = particlesRef.current, now = Date.now();
for (let s = 0; s < 8; s++) {
const a = (Math.PI * 2 * s) / 8 + (Math.random() - 0.5) * 0.4, d = 30 + Math.random() * 40;
if (arr.length < PARTICLE_CAP) arr.push({ x: cx, y: cy, dx: Math.cos(a) * d, dy: Math.sin(a) * d, color: matColor, life: 500, size: 3 + Math.random() * 4, born: now, type: “shard”, extra: { sides: 3 + Math.floor(Math.random() * 2), spin: (Math.random() - 0.5) * 5 } });
}
spawnShockwave(cx, cy, target.w * 1.5, matColor);
spawnGroundFx(cx, cy, matColor, target.w * 0.8, 5000);
doShake(0.4);
recordDiscovery(“destroy:” + target.mat);
if (!shownTipsRef.current.has(‘destroy_tip’)) { shownTipsRef.current.add(‘destroy_tip’); setToasts(t => […t, { id: Date.now(), text: ‘💡 Objects have HP — keep hitting to destroy them!’, born: Date.now() }]); }
const debrisCount = target.fragile ? 3 : 2;
for (let i = 0; i < debrisCount; i++) {
const angle = (Math.PI * 2 * i) / debrisCount + (Math.random() - 0.5) * 0.5;
debris.push({
id: Date.now() + i + Math.random(), x: cx + (Math.cos(angle) * 30) - 10, y: cy + (Math.sin(angle) * 30) - 10,
w: Math.max(15, target.w * 0.35), h: Math.max(15, target.h * 0.35), mat: target.mat, label: “Debris”, shape: “rect”,
origX: cx, origY: cy, origMat: target.mat, scale: 0.8, rotation: Math.random() * 360, opacity: 0.8, effects: [], statuses: [],
vx: Math.cos(angle) * 8 + Math.random() * 4, vy: Math.sin(angle) * 8 - 3, hp: target.maxHp * 0.2, maxHp: target.maxHp * 0.2, fragile: true,
temp: 20, mass: Math.max(15, target.w * 0.35) * Math.max(15, target.h * 0.35) * (MAT_PROPS[target.mat]?.density || 1) * 0.01 * 0.3, resting: 0,
trailUntil: Date.now() + 500,
});
}
if (target.mat === “explosive”) {
const _cx = cx, _cy = cy;
setTimeout(() => {
spawnFire(_cx, _cy, 30, 120, 600);
spawnShockwave(_cx, _cy, 180, “#ef4444”);
spawnPostFx(“vignette”, { life: 300 });
doShake(0.8);
createZone(_cx, _cy, “fire”, 2, 1);
setObjects(p => p.map(o => {
const dx = (o.x + o.w / 2) - _cx, dy = (o.y + o.h / 2) - _cy, dist = Math.sqrt(dx * dx + dy * dy) || 1;
if (dist < 180) return { …o, vx: o.vx + (dx / dist) * 20, vy: o.vy + (dy / dist) * 20, hp: Math.max(0, o.hp - 40) };
return o;
}));
}, 100);
}
next.splice(idx, 1);
} else {
next[idx] = { …target, hp: newHp };
}
}
if (debris.length > 0 && next.length + debris.length <= 50) {
next.push(…debris);
}
return next;
});
}, [spawnHitFlash, spawnDmgNumber, playImpact, playDestroy, spawnShockwave, spawnGroundFx, doShake, recordDiscovery, spawnFire, spawnPostFx, createZone]);

// ─── BEAM REFLECTION HELPER (v4.0) ───
const tryReflect = useCallback((obj, powerId, pos) => {
if(!obj) return;
const refl = MAT_PROPS[obj.mat]?.reflectivity || 0;
if(refl < 0.5 || Math.random() > refl) return;
const pw = POWERS.find(p=>p.id===powerId); if(!pw) return;
const cx=obj.x+obj.w/2, cy=obj.y+obj.h/2;
const dx=cx-pos.x, dy=cy-pos.y, len=Math.sqrt(dx*dx+dy*dy)||1;
const rx=cx+dx/len*120, ry=cy+dy/len*120;
spawnBeamFx(cx,cy,rx,ry,pw.color,200);
const targets = (objectsRef.current||[]).filter(o=>o.id!==obj.id&&Math.sqrt((o.x+o.w/2-rx)**2+(o.y+o.h/2-ry)**2)<50);
targets.forEach(t => dealDamage(t.id, Math.round(pw.dmg * refl * 0.5)));
if(targets.length > 0) recordDiscovery(`reflect:${powerId}+${obj.mat}`);
spawn(cx,cy,”#fff”,4,15,200,[1,3],“spark”,{trail:[]});
}, [spawn,spawnBeamFx,dealDamage,recordDiscovery]);

// ─── APPLY SINGLE POWER (v3.5 — DC + Marvel) ───
const applyPower = useCallback((powerId, obj, pos) => {
const pw = POWERS.find(p=>p.id===powerId);
if(!pw) return;
const c = pw.color;
const cx = obj?obj.x+obj.w/2:pos.x, cy = obj?obj.y+obj.h/2:pos.y;

```
// Discovery
if (obj) recordDiscovery(`p:${powerId}+m:${obj.mat}`);
playPowerSound(powerId);

// Environment power modifier
const envMod = ENVIRONMENTS[env]?.powerMods?.[pw.cat] || 1;
const eDmg = Math.round(pw.dmg * envMod);
const envDiscKey = 'env:'+env+'+'+pw.cat;
if(!discoveredRef.current.has(envDiscKey)) recordDiscovery(envDiscKey);

switch(powerId) {
  // ═══ ENERGY ═══
  case "optic_blast":
    if(obj) {
      tryReflect(obj,powerId,pos);
      spawnBeamFx(0,bounds.h*0.4,cx,cy,"#dc2626",6,"energy");
      spawn(cx,cy,"#ef4444",15,40,500,[3,7]); spawnHitFlash(cx,cy,pw.dmg);
      addEffect(obj.id,{type:"hit",dur:400});
      dealDamage(obj.id,eDmg);
      setObjects(p=>p.map(o=>o.id===obj.id?{...o,vx:o.vx+cx*0.02,vy:o.vy+(cy-bounds.h*0.4)*0.015}:o));
      doShake(0.3);
    }
    break;
  case "plasmoids":
    spawn(pos.x,pos.y,"#ec4899",10,60,700,[3,7]);
    spawn(pos.x,pos.y,"#fbbf24",8,45,800,[2,5]);
    spawn(pos.x,pos.y,"#a78bfa",6,50,600,[3,6]);
    if(obj) dealDamage(obj.id,eDmg);
    break;
  case "plasma_rings":
    spawnRing(pos.x,pos.y,"#60a5fa",120);
    spawnRing(pos.x,pos.y,"#93c5fd",80);
    if(obj){addEffect(obj.id,{type:"burn",dur:1200});doShake(0.2);dealDamage(obj.id,eDmg);applyStatus(obj.id,"burning",4);}
    setObjects(p=>p.map(o=>{const dx=(o.x+o.w/2)-pos.x,dy=(o.y+o.h/2)-pos.y,dist=Math.sqrt(dx*dx+dy*dy)||1;if(dist<140)return{...o,vx:o.vx+(dx/dist)*6,vy:o.vy+(dy/dist)*6};return o;}));
    break;
  case "sonic_scream":
    spawnRing(pos.x,pos.y,"#86efac",160);
    spawn(pos.x,pos.y,"#bbf7d0",8,100,500,[4,8]);
    doShake(0.3);
    setObjects(p=>p.map(o=>{const dx=(o.x+o.w/2)-pos.x,dist=Math.abs(dx)||1;if(dist<200){const push=(dx>0?1:-1)*Math.min(12,1500/dist);if(dist<80)dealDamage(o.id,pw.dmg);return{...o,vx:o.vx+push};}return o;}));
    break;
  case "solar_blast":
    spawnFire(pos.x,pos.y,20,80,800);
    spawnRing(pos.x,pos.y,"#fbbf24",100,true); spawnShockwave(pos.x,pos.y,100,"#fbbf24");
    doShake(0.3);
    createZone(pos.x,pos.y,"fire",1,0.7);
    if(obj){dealDamage(obj.id,eDmg);if(MAT_RULES.burns[obj.mat])applyStatus(obj.id,"burning",5);}
    break;
  case "energy_absorb":
    if(obj){
      spawnBeamFx(cx,cy,pos.x,pos.y,"#fbbf24",3,"psi");
      addEffect(obj.id,{type:"drain",dur:1000});
      dealDamage(obj.id,eDmg);
      setObjects(p=>p.map(o=>o.id===obj.id?{...o,scale:Math.max(0.3,o.scale-0.15)}:o));
    }
    break;
  case "heat_vision":
    if(obj){
      tryReflect(obj,powerId,pos);
      spawnBeamFx(cx-40,0,cx-5,cy,"#ef4444",4,"energy");
      spawnBeamFx(cx+40,0,cx+5,cy,"#ef4444",4,"energy");
      spawnFire(cx,cy,10,25,600); spawnHitFlash(cx,cy,pw.dmg);
      addEffect(obj.id,{type:"burn",dur:1500});
      dealDamage(obj.id,eDmg); doShake(0.3);
      if(MAT_RULES.burns[obj.mat]) applyStatus(obj.id,"burning",5);
      createZone(cx,cy,"fire",1,0.5);
    } else {
      spawnBeamFx(pos.x-40,0,pos.x,pos.y,"#ef4444",4,"energy");
      spawnBeamFx(pos.x+40,0,pos.x,pos.y,"#ef4444",4,"energy");
      spawnFire(pos.x,pos.y,6,20,400);
    }
    break;
  case "starbolts":
    for(let i=0;i<5;i++){
      const a=(Math.PI*2*i)/5,ex=pos.x+Math.cos(a)*120,ey=pos.y+Math.sin(a)*120;
      setTimeout(()=>{spawn(ex,ey,"#22c55e",6,25,400,[3,6]);spawnShockwave(ex,ey,40,"#22c55e");},i*80);
    }
    doShake(0.2);
    setObjects(p=>p.map(o=>{const dist=Math.sqrt((o.x+o.w/2-pos.x)**2+(o.y+o.h/2-pos.y)**2);if(dist<140){dealDamage(o.id,pw.dmg);return{...o,vx:o.vx+(Math.random()-0.5)*6};}return o;}));
    break;
  case "photon_blast":
    spawnRing(pos.x,pos.y,"#fbbf24",140,true);
    spawnShockwave(pos.x,pos.y,140,"#fbbf24");
    spawn(pos.x,pos.y,"#fde68a",20,80,700,[4,9]);
    doShake(0.4); spawnPostFx("tint",{color:"#fbbf24",life:200});
    setObjects(p=>p.map(o=>{const dx=(o.x+o.w/2)-pos.x,dy=(o.y+o.h/2)-pos.y,dist=Math.sqrt(dx*dx+dy*dy)||1;if(dist<160){dealDamage(o.id,pw.dmg);return{...o,vx:o.vx+(dx/dist)*10,vy:o.vy+(dy/dist)*10};}return o;}));
    break;
  case "omega_beam": {
    if(obj){
      // Zigzag beam path
      const pts=[];
      const dx=cx-50,dy=cy;
      for(let i=0;i<=6;i++){const t=i/6;pts.push({x:50+dx*t+(i%2===0?40:-40)*(i>0&&i<6?1:0),y:bounds.h*0.3+dy*t*0.5});}
      const arr=particlesRef.current,now=Date.now();
      arr.push({x:50,y:bounds.h*0.3,dx:0,dy:0,color:"#dc2626",life:300,size:4,born:now,type:"lightning",extra:{points:pts,width:5}});
      spawnFire(cx,cy,8,20,400); spawnHitFlash(cx,cy,pw.dmg);
      dealDamage(obj.id,eDmg); doShake(0.5);
      addEffect(obj.id,{type:"hit",dur:500});
      setObjects(p=>p.map(o=>o.id===obj.id?{...o,vx:o.vx+8,vy:o.vy-3}:o));
    }
    break;
  }
  case "power_cosmic":
    spawnRing(pos.x,pos.y,"#c4b5fd",180,true);
    spawnRing(pos.x,pos.y,"#818cf8",140,true);
    spawn(pos.x,pos.y,"#e0e7ff",25,100,900,[4,10]);
    spawnShockwave(pos.x,pos.y,180,"#c4b5fd"); doShake(0.5);
    spawnPostFx("tint",{color:"#818cf8",life:300});
    setObjects(p=>p.map(o=>{const dx=(o.x+o.w/2)-pos.x,dy=(o.y+o.h/2)-pos.y,dist=Math.sqrt(dx*dx+dy*dy)||1;if(dist<200){dealDamage(o.id,pw.dmg);return{...o,vx:o.vx+(dx/dist)*12,vy:o.vy+(dy/dist)*12};}return o;}));
    break;

  // ═══ ELEMENTAL ═══
  case "cryokinesis":
    if(obj){
      // Water→ice transformation
      if(obj.mat==='water'){setObjects(p=>p.map(o=>o.id===obj.id?{...o,mat:'glass',label:'Frozen',hp:MAT_HP.glass,maxHp:MAT_HP.glass,temp:-50,mass:o.w*o.h*MAT_PROPS.glass.density*0.01}:o));recordDiscovery('transform:water+frozen');spawnIce(cx,cy,10,20);}
      addEffect(obj.id,{type:"freeze",dur:3000});
      spawnIce(cx,cy,12,30);
      applyStatus(obj.id,"frozen",8);
      dealDamage(obj.id,eDmg);
      createZone(cx,cy,"ice",1,0.6);
      spawnGroundFx(cx,cy,"#67e8f9",25,4000);
      if(obj.mat==="water")setObjects(p=>p.map(o=>o.id===obj.id?{...o,mat:"glass",label:"Ice"}:o));
    }
    break;
  case "pyrokinesis":
    if(obj){
      // Water→steam transformation
      if(obj.mat==='water'){spawnSmoke(cx,cy,'#d1d5db',15,40);recordDiscovery('transform:water+steam');setObjects(p=>p.map(o=>o.id===obj.id?{...o,hp:Math.max(0,o.hp-30)}:o));}
      if(MAT_RULES.burns[obj.mat]){
        addEffect(obj.id,{type:"burn",dur:2500});
        spawnFire(cx,cy-10,15,30,900);
        applyStatus(obj.id,"burning",6);
        dealDamage(obj.id,eDmg);
        createZone(cx,cy,"fire",1,0.8);
        spawnGroundFx(cx,cy,"#f97316",20,3000);
      } else spawn(cx,cy,"#fb923c",6,20,400,[2,4]);
    }
    break;
  case "magma":
    spawnFire(pos.x,pos.y,15,60,900);
    spawnFire(pos.x,pos.y+20,10,40,700);
    doShake(0.4);
    createZone(pos.x,pos.y,"fire",2,1);
    setObjects(p=>p.map(o=>{const dist=Math.sqrt((o.x+o.w/2-pos.x)**2+(o.y+o.h/2-pos.y)**2);if(dist<100){applyStatus(o.id,"burning",5);dealDamage(o.id,pw.dmg);return{...o,vy:o.vy-5};}return o;}));
    break;
  case "plant_growth":
    spawn(pos.x,pos.y,"#22c55e",18,60,1200,[3,8]);
    spawn(pos.x,pos.y,"#4ade80",12,40,1000,[2,5]);
    if(obj){setObjects(p=>p.map(o=>o.id===obj.id?{...o,hp:Math.min(o.maxHp,o.hp+20)}:o));spawn(cx,cy,"#86efac",6,15,400,[2,4]);}
    break;
  case "sandstorm":
    if(obj){
      spawnSmoke(cx,cy,"#d4a373",20,45);
      addEffect(obj.id,{type:"dissolve",dur:1500});
      applyStatus(obj.id,"corroding",5);
      dealDamage(obj.id,eDmg);
    }
    break;
  case "freeze_breath":
    spawnIce(pos.x,pos.y,20,60);
    spawnBeamFx(pos.x-80,pos.y,pos.x+80,pos.y,"#a5f3fc",8,"frost");
    createZone(pos.x,pos.y,"ice",2,0.8); doShake(0.2);
    setObjects(p=>p.map(o=>{const dist=Math.sqrt((o.x+o.w/2-pos.x)**2+(o.y+o.h/2-pos.y)**2);if(dist<120){applyStatus(o.id,"frozen",6);dealDamage(o.id,pw.dmg);return{...o,vx:o.vx*0.2,vy:o.vy*0.2};}return o;}));
    break;
  case "hydrokinesis":
    spawn(pos.x,pos.y,"#38bdf8",20,80,800,[4,8],"circle");
    spawnRing(pos.x,pos.y,"#38bdf8",100);
    if(obj){
      dealDamage(obj.id,eDmg);
      setObjects(p=>p.map(o=>{const dx=(o.x+o.w/2)-pos.x,dy=(o.y+o.h/2)-pos.y,dist=Math.sqrt(dx*dx+dy*dy)||1;if(dist<130)return{...o,vx:o.vx+(dx/dist)*8,vy:o.vy+(dy/dist)*8};return o;}));
      if(MAT_RULES.conducts[obj.mat]) applyStatus(obj.id,"electrified",2);
    }
    createZone(pos.x,pos.y,"ice",1,0.4); // water cools area
    break;
  case "earth_control":
    doShake(0.6);
    spawn(pos.x,pos.y,"#78716c",15,50,700,[5,12],"shard",{sides:4,spin:1});
    spawnShockwave(pos.x,pos.y,120,"#a8a29e");
    setObjects(p=>p.map(o=>{const dist=Math.sqrt((o.x+o.w/2-pos.x)**2+(o.y+o.h/2-pos.y)**2);if(dist<130){dealDamage(o.id,pw.dmg);return{...o,vy:o.vy-8+Math.random()*-6};}return o;}));
    break;
  case "lightning_summon":
    spawnLightning(pos.x+(Math.random()-0.5)*30,0,pos.x,pos.y,"#fbbf24");
    spawnBeamFx(pos.x,0,pos.x,pos.y,"#fbbf24",6,"lightning");
    spawnShockwave(pos.x,pos.y,100,"#fbbf24"); doShake(0.5);
    createZone(pos.x,pos.y,"electric",2,0.8);
    if(obj){dealDamage(obj.id,eDmg);applyStatus(obj.id,"electrified",4);addEffect(obj.id,{type:"shock",dur:800});}
    setObjects(p=>p.map(o=>{const dist=Math.sqrt((o.x+o.w/2-pos.x)**2+(o.y+o.h/2-pos.y)**2);if(dist<100)return{...o,vx:o.vx+(Math.random()-0.5)*10,vy:o.vy-3};return o;}));
    break;
  case "hellfire":
    spawnFire(pos.x,pos.y,20,50,1000);
    spawn(pos.x,pos.y,"#22c55e",6,30,600,[2,5]); // green undertone
    if(obj){
      dealDamage(obj.id,eDmg); applyStatus(obj.id,"burning",6);
      addEffect(obj.id,{type:"burn",dur:2000});
      // Ignores armor
      setObjects(p=>p.map(o=>o.id===obj.id?{...o,statuses:o.statuses.filter(s=>s.type!=="armored")}:o));
    }
    createZone(pos.x,pos.y,"fire",1,0.9);
    break;

  // ═══ KINETIC ═══
  case "repulsion":
    spawnRing(pos.x,pos.y,c,130);doShake(0.3);
    setObjects(p=>p.map(o=>{const dx=(o.x+o.w/2)-pos.x,dy=(o.y+o.h/2)-pos.y,dist=Math.sqrt(dx*dx+dy*dy)||1;return{...o,vx:o.vx+(dx/dist)*Math.min(18,4000/dist),vy:o.vy+(dy/dist)*Math.min(18,4000/dist)};}));
    break;
  case "seismic":
    doShake(0.8);
    spawnShockwave(pos.x,pos.y,150,"#78716c"); spawnSmoke(pos.x,pos.y,"#78716c",10,30);
    setObjects(p=>p.map(o=>({...o,vx:o.vx+(Math.random()-0.5)*15,vy:o.vy+(Math.random()-0.5)*8})));
    break;
  case "cannonball":
    if(obj){
      spawn(cx,cy,"#c084fc",15,70,600,[4,9]); spawnShockwave(cx,cy,100,"#a855f7");
      spawnRing(cx,cy,"#a855f7",80,true);
      doShake(0.4);
      dealDamage(obj.id,eDmg);
      setObjects(p=>p.map(o=>{const dx=(o.x+o.w/2)-cx,dy=(o.y+o.h/2)-cy,dist=Math.sqrt(dx*dx+dy*dy)||1;if(dist<140)return{...o,vx:o.vx+(dx/dist)*12,vy:o.vy+(dy/dist)*12};return o;}));
    }
    break;
  case "thunder_clap":
    doShake(0.9);
    spawnShockwave(pos.x,pos.y,200,"#86efac");
    spawnRing(pos.x,pos.y,"#22c55e",200);
    spawn(pos.x,pos.y,"#bbf7d0",20,120,600,[3,8]);
    setObjects(p=>p.map(o=>{const dx=(o.x+o.w/2)-pos.x,dy=(o.y+o.h/2)-pos.y,dist=Math.sqrt(dx*dx+dy*dy)||1;if(dist<220){dealDamage(o.id,pw.dmg);return{...o,vx:o.vx+(dx/dist)*Math.min(20,5000/dist),vy:o.vy+(dy/dist)*Math.min(20,5000/dist)};}return o;}));
    break;
  case "shield_throw": {
    // Bouncing projectile — hits up to 3 objects
    const targets=(objectsRef.current||[]).sort((a,b)=>{const da=Math.sqrt((a.x+a.w/2-pos.x)**2+(a.y+a.h/2-pos.y)**2);const db=Math.sqrt((b.x+b.w/2-pos.x)**2+(b.y+b.h/2-pos.y)**2);return da-db;}).slice(0,3);
    targets.forEach((t,i)=>{
      const tx=t.x+t.w/2,ty=t.y+t.h/2;
      setTimeout(()=>{
        const prev=i>0?targets[i-1]:null;
        const fx=prev?prev.x+prev.w/2:pos.x, fy=prev?prev.y+prev.h/2:pos.y;
        spawnBeamFx(fx,fy,tx,ty,"#3b82f6",3,"energy");
        spawnHitFlash(tx,ty,pw.dmg); dealDamage(t.id,eDmg);
        addEffect(t.id,{type:"hit",dur:400});
        setObjects(p=>p.map(o=>o.id===t.id?{...o,vx:o.vx+(Math.random()-0.5)*6}:o));
      },i*200);
    });
    doShake(0.2);
    break;
  }
  case "mjolnir":
    if(obj){
      spawnLightning(cx,0,cx,cy,"#60a5fa");
      spawnBeamFx(cx,0,cx,cy,"#60a5fa",5,"lightning");
      spawnShockwave(cx,cy,120,"#93c5fd");
      spawn(cx,cy,"#bfdbfe",15,50,600,[4,8]);
      doShake(0.6);
      dealDamage(obj.id,eDmg); applyStatus(obj.id,"electrified",4);
      addEffect(obj.id,{type:"shock",dur:1000});
      createZone(cx,cy,"electric",1,0.7);
      setObjects(p=>p.map(o=>{const dx=(o.x+o.w/2)-cx,dy=(o.y+o.h/2)-cy,dist=Math.sqrt(dx*dx+dy*dy)||1;if(dist<140)return{...o,vx:o.vx+(dx/dist)*10,vy:o.vy+(dy/dist)*10};return o;}));
    }
    break;
  case "lasso":
    if(obj){
      spawnBeamFx(pos.x,pos.y,cx,cy,"#fbbf24",3,"energy");
      spawn(cx,cy,"#fbbf24",6,15,400,[2,4]);
      setObjects(p=>p.map(o=>o.id===obj.id?{...o,x:pos.x-o.w/2,y:pos.y-o.h/2,vx:0,vy:0}:o));
      dealDamage(obj.id,eDmg);
    }
    break;
  case "vibranium_pulse":
    spawnRing(pos.x,pos.y,"#a855f7",130,true);
    spawnShockwave(pos.x,pos.y,130,"#a855f7");
    spawn(pos.x,pos.y,"#c084fc",15,60,500,[3,7]);
    doShake(0.4); spawnPostFx("tint",{color:"#a855f7",life:150});
    setObjects(p=>p.map(o=>{const dx=(o.x+o.w/2)-pos.x,dy=(o.y+o.h/2)-pos.y,dist=Math.sqrt(dx*dx+dy*dy)||1;if(dist<150){dealDamage(o.id,pw.dmg);return{...o,vx:o.vx+(dx/dist)*12,vy:o.vy+(dy/dist)*12};}return o;}));
    break;

  // ═══ PHYSICAL ═══
  case "claws":
    if(obj){
      addEffect(obj.id,{type:"slash",dur:500});
      spawnSlash(cx,cy,Math.random()*Math.PI*2,"#e5e7eb",50); spawnSlashImpact(cx,cy,Math.random()*Math.PI*2,"#e5e7eb");
      dealDamage(obj.id,eDmg);
      setObjects(p=>p.map(o=>o.id===obj.id?{...o,rotation:o.rotation+(Math.random()-0.5)*20}:o));
    }
    break;
  case "claws_x23":
    if(obj){
      addEffect(obj.id,{type:"slash",dur:300});
      setTimeout(()=>addEffect(obj.id,{type:"slash",dur:300}),200);
      setTimeout(()=>{addEffect(obj.id,{type:"hit",dur:300});spawn(cx,cy,"#d1d5db",10,30,400,[2,5]);},400);
      dealDamage(obj.id,eDmg);
    }
    break;
  case "diamond_form":
    if(obj){
      addEffect(obj.id,{type:"diamond",dur:3000});
      spawn(cx,cy,"#e0f2fe",10,25,500,[3,6]);
      applyStatus(obj.id,"armored",10);
    }
    break;
  case "organic_steel":
    if(obj){
      setObjects(p=>p.map(o=>o.id===obj.id?{...o,mat:"metal",hp:Math.min(o.hp+30,MAT_HP.metal),maxHp:MAT_HP.metal}:o));
      addEffect(obj.id,{type:"steel",dur:2000});
      spawn(cx,cy,"#9ca3af",8,20,400,[2,5]);
    }
    break;
  case "armor":
    if(obj){addEffect(obj.id,{type:"armor",dur:3000});spawnRing(cx,cy,"#f87171",50);applyStatus(obj.id,"armored",10);}
    break;
  case "duplication":
    if(obj){
      const clone={...obj,id:Date.now()+Math.random(),x:obj.x+25+Math.random()*20,y:obj.y+25+Math.random()*20,origX:obj.x+25,origY:obj.y+25,effects:[],statuses:[],vx:0,vy:0,hp:obj.hp,maxHp:obj.maxHp};
      setObjects(p=>[...p,clone]);
      spawn(clone.x+clone.w/2,clone.y+clone.h/2,"#86efac",8,25,400,[2,5]);
    }
    break;
  case "reactive": {
    if(obj){
      const newMats=MATERIALS.filter(m=>m!==obj.mat);
      const newMat=newMats[Math.floor(Math.random()*newMats.length)];
      setObjects(p=>p.map(o=>o.id===obj.id?{...o,mat:newMat,scale:Math.min(2,o.scale+0.15),hp:Math.min(o.hp+15,MAT_HP[newMat]),maxHp:MAT_HP[newMat]}:o));
      addEffect(obj.id,{type:"evolve",dur:800});
      spawn(cx,cy,"#a3e635",10,30,600,[3,6]);
    }
    break;
  }
  case "web_shot":
    if(obj){
      spawnBeamFx(pos.x,pos.y,cx,cy,"#e5e7eb",2,"energy");
      spawn(cx,cy,"#d4d4d8",10,20,800,[2,5]);
      addEffect(obj.id,{type:"freeze",dur:2000}); // web restraint
      setObjects(p=>p.map(o=>o.id===obj.id?{...o,vx:0,vy:0}:o));
      dealDamage(obj.id,eDmg);
    }
    break;
  case "symbiote":
    if(obj){
      spawn(cx,cy,"#1e1b4b",12,30,800,[3,8]);
      spawn(cx,cy,"#4c1d95",6,40,600,[2,5],"psi",{freq:1.5,amp:8,phase:0});
      addEffect(obj.id,{type:"dissolve",dur:1500});
      dealDamage(obj.id,eDmg);
      applyStatus(obj.id,"corroding",4);
      setObjects(p=>p.map(o=>o.id===obj.id?{...o,mat:o.mat==="organic"?"energy":o.mat}:o));
    }
    break;
  case "hulk_smash":
    doShake(1);
    spawnShockwave(pos.x,pos.y,180,"#22c55e");
    spawn(pos.x,pos.y,"#86efac",20,80,600,[5,12],"shard",{sides:3,spin:2});
    spawnPostFx("tint",{color:"#22c55e",life:150});
    setObjects(p=>p.map(o=>{const dx=(o.x+o.w/2)-pos.x,dy=(o.y+o.h/2)-pos.y,dist=Math.sqrt(dx*dx+dy*dy)||1;if(dist<200){dealDamage(o.id,pw.dmg);return{...o,vx:o.vx+(dx/dist)*Math.min(20,6000/dist),vy:o.vy-8};}return o;}));
    break;
  case "martial_arts":
    if(obj){
      for(let i=0;i<4;i++){
        setTimeout(()=>{
          spawnSlash(cx+(Math.random()-0.5)*20,cy+(Math.random()-0.5)*20,Math.random()*Math.PI*2,"#f59e0b",40);
          spawnSlashImpact(cx,cy,Math.random()*Math.PI*2,"#fbbf24");
          addEffect(obj.id,{type:"slash",dur:200});
          spawnHitFlash(cx,cy,10);
          dealDamage(obj.id,Math.floor(pw.dmg/4));
        },i*120);
      }
      doShake(0.3);
    }
    break;
  case "regeneration":
    if(obj){
      setObjects(p=>p.map(o=>o.id===obj.id?{...o,hp:Math.min(o.maxHp,o.hp+40),statuses:[]}:o));
      spawn(cx,cy,"#ef4444",12,20,600,[3,6]);
      spawn(cx,cy,"#fbbf24",6,15,400,[2,4]);
      addEffect(obj.id,{type:"evolve",dur:600});
    }
    break;

  // ═══ PSYCHIC ═══
  case "psi_blade":
    if(obj){
      spawnBeamFx(pos.x,pos.y,cx,cy,"#e879f9",4,"psi");
      addEffect(obj.id,{type:"slash",dur:500});
      spawnPsi(cx,cy,"#e879f9",10,35); spawnSlashImpact(cx,cy,Math.atan2(cy-pos.y,cx-pos.x),"#e879f9");
      dealDamage(obj.id,eDmg);
      setObjects(p=>p.map(o=>o.id===obj.id?{...o,vx:o.vx+(Math.random()-0.5)*8,rotation:o.rotation+15}:o));
    }
    break;
  case "telepathy":
    if(obj){setShowInfo(obj.id);setTimeout(()=>setShowInfo(null),2500);spawnPsi(cx,cy,"#818cf8",8,20);addEffect(obj.id,{type:"scan",dur:2500});}
    break;
  case "phoenix": {
    const pRef = phoenixRef.current;
    const now = Date.now();
    if (now - pRef.lastUsed < 30000) pRef.level = Math.min(5, pRef.level+1);
    else pRef.level = 1;
    pRef.lastUsed = now;
    const lvl = pRef.level;
    const radius = 120 + lvl*40;
    const dmgMult = lvl;
    spawnRing(pos.x,pos.y,"#f59e0b",radius,true);
    if(lvl>=2) spawnRing(pos.x,pos.y,"#ef4444",radius*0.8,true);
    spawnFire(pos.x,pos.y,20+lvl*8,radius,1200);
    doShake(0.3+lvl*0.15); spawnPostFx("tint",{color:"#f59e0b",life:300+lvl*100});
    if(lvl>=3) createZone(pos.x,pos.y,"void",lvl-1,0.8);
    setObjects(p=>p.map(o=>{
      const dx=(o.x+o.w/2)-pos.x,dy=(o.y+o.h/2)-pos.y,dist=Math.sqrt(dx*dx+dy*dy)||1;
      if(dist<radius){
        applyStatus(o.id,"burning",3+lvl);
        dealDamage(o.id,pw.dmg*dmgMult);
        const force = Math.min(10+lvl*4,5000/dist);
        return{...o,vx:o.vx+(dx/dist)*force,vy:o.vy+(dy/dist)*force};
      }
      return o;
    }));
    if(lvl>=4) setObjects(p=>p.map(o=>{const dx=(o.x+o.w/2)-pos.x,dy=(o.y+o.h/2)-pos.y,dist=Math.sqrt(dx*dx+dy*dy)||1;return{...o,vx:o.vx-(dx/dist)*5,vy:o.vy-(dy/dist)*5};}));
    if(lvl===5) setTimeout(()=>{spawnRing(pos.x,pos.y,"#dc2626",300,true);spawnShockwave(pos.x,pos.y,300,"#dc2626");spawnPostFx("vignette",{life:400});doShake(1);setObjects(p=>p.map(o=>{dealDamage(o.id,o.maxHp*0.8);return{...o,vx:o.vx+(Math.random()-0.5)*25,vy:o.vy+(Math.random()-0.5)*25};}));},600);
    break;
  }
  case "hex_bolt": {
    spawnPsi(cx||pos.x,cy||pos.y,"#dc2626",15,40); spawnPostFx("tint",{color:"#dc2626",life:150});
    createZone(pos.x,pos.y,"void",2,0.9);
    if(obj){
      addEffect(obj.id,{type:"hex",dur:1500});
      dealDamage(obj.id,eDmg);
      const chaos=Math.random();
      setObjects(p=>p.map(o=>{
        if(o.id!==obj.id)return o;
        if(chaos<0.33)return{...o,scale:o.scale>1?0.5:1.8};
        if(chaos<0.66)return{...o,rotation:o.rotation+180,vy:o.vy-10};
        const newMats=MATERIALS.filter(m=>m!==o.mat);
        return{...o,mat:newMats[Math.floor(Math.random()*newMats.length)]};
      }));
    }
    break;
  }
  case "illusion":
    if(obj){
      const ghosts=[];
      for(let i=0;i<3;i++) ghosts.push({...obj,id:Date.now()+i+Math.random(),x:obj.x+(Math.random()-0.5)*80,y:obj.y+(Math.random()-0.5)*80,opacity:0.3,effects:[],statuses:[],vx:0,vy:0,hp:1,maxHp:1});
      const ghostIds=ghosts.map(g=>g.id);
      setObjects(p=>[...p,...ghosts]);
      setTimeout(()=>setObjects(p=>p.filter(o=>!ghostIds.includes(o.id))),2000);
      spawn(cx,cy,"#c4b5fd",8,30,500,[3,6],"psi",{freq:2,amp:5,phase:0});
    }
    break;
  case "probability": {
    const eligible=POWERS.filter(p=>p.id!=="probability");
    const rp=eligible[Math.floor(Math.random()*eligible.length)];
    spawn(pos.x,pos.y,"#34d399",8,25,400,[3,6]);
    setTimeout(()=>applyPower(rp.id,obj,pos),150);
    break;
  }
  case "penance_stare":
    if(obj){
      spawnBeamFx(pos.x,pos.y,cx,cy,"#f97316",6,"psi");
      spawnFire(cx,cy,8,15,500);
      spawnPostFx("vignette",{life:300});
      addEffect(obj.id,{type:"hex",dur:800});
      dealDamage(obj.id,eDmg); doShake(0.3);
    }
    break;
  case "empathy":
    if(obj&&obj.statuses.length>0){
      const status=obj.statuses[0].type;
      setObjects(p=>p.map(o=>o.id===obj.id?{...o,statuses:[]}:o));
      spawnPsi(cx,cy,"#6366f1",8,30);
      // Spread to nearby
      setObjects(p=>p.map(o=>{
        if(o.id===obj.id)return o;
        const dist=Math.sqrt((o.x+o.w/2-cx)**2+(o.y+o.h/2-cy)**2);
        if(dist<120){applyStatus(o.id,status,4);return o;}
        return o;
      }));
    } else if(obj){
      spawnPsi(cx,cy,"#6366f1",6,20); dealDamage(obj.id,eDmg);
    }
    break;
  case "astral_project":
    if(obj){
      const ghost={...obj,id:Date.now()+Math.random(),x:obj.x+40,y:obj.y-30,opacity:0.35,effects:[],statuses:[],vx:4,vy:-2,hp:1,maxHp:1};
      setObjects(p=>[...p,ghost]);
      spawn(cx,cy,"#e879f9",8,30,600,[3,6],"psi",{freq:2,amp:6,phase:0});
      setTimeout(()=>{
        spawn(ghost.x+ghost.w/2,ghost.y+ghost.h/2,"#e879f9",12,40,500,[3,7]);
        setObjects(p=>{
          const nearObjs=p.filter(o=>o.id!==ghost.id&&Math.sqrt((o.x-ghost.x)**2+(o.y-ghost.y)**2)<80);
          nearObjs.forEach(no=>dealDamage(no.id,pw.dmg));
          return p.filter(o=>o.id!==ghost.id);
        });
      },800);
    }
    break;

  // ═══ SPATIAL ═══
  case "time_reverse":
    setObjects(p=>p.filter(o=>ORIGINAL_IDS.has(o.id)).map(o=>({...o,x:o.origX,y:o.origY,mat:o.origMat,scale:1,opacity:1,rotation:0,vx:0,vy:0,effects:[],statuses:[],hp:MAT_HP[o.origMat],maxHp:MAT_HP[o.origMat],temp:20,mass:o.w*o.h*(MAT_PROPS[o.origMat]?.density||1)*0.01,resting:0})));
    spawnRing(bounds.w/2,bounds.h/2,"#60a5fa",200);
    zonesRef.current = createZoneGrid(); activeZonesRef.current.clear();
    groundFxRef.current = [];
    phoenixRef.current = {level:1,lastUsed:0};
    break;
  case "phasing":
    if(obj){
      setObjects(p=>p.map(o=>o.id===obj.id?{...o,opacity:o.opacity<0.5?1:0.25}:o));
      addEffect(obj.id,{type:"phase",dur:2000});
      applyStatus(obj.id,"phased",6);
      spawnSmoke(cx,cy,"#a5b4fc",6,15);
    }
    break;
  case "size_shift":
    if(obj){
      setObjects(p=>p.map(o=>o.id===obj.id?{...o,scale:o.scale>1.3?0.5:o.scale+0.4}:o));
      spawn(cx,cy,"#fb7185",8,25,400,[3,6]);
      dealDamage(obj.id,eDmg);
    }
    break;
  case "speed_force":
    // Creates speed zone + lightning trail
    createZone(pos.x,pos.y,"electric",1,0.5);
    spawnLightning(pos.x-60,pos.y,pos.x+60,pos.y,"#ef4444");
    spawn(pos.x,pos.y,"#fbbf24",8,40,300,[2,4],"spark",{trail:[]});
    if(obj){
      dealDamage(obj.id,eDmg);
      setObjects(p=>p.map(o=>o.id===obj.id?{...o,vx:o.vx+(pos.x>cx?-15:15)}:o));
    }
    break;
  case "blink_slash":
    if(obj){
      spawnSmoke(cx-30,cy,"#ec4899",4,10);
      spawnSlash(cx,cy,Math.atan2(cy-pos.y,cx-pos.x),"#ec4899",60);
      spawnSlashImpact(cx,cy,Math.atan2(cy-pos.y,cx-pos.x),"#ec4899",2);
      spawnSmoke(cx+30,cy,"#ec4899",4,10);
      dealDamage(obj.id,eDmg);
      addEffect(obj.id,{type:"slash",dur:400});
      setObjects(p=>p.map(o=>o.id===obj.id?{...o,rotation:o.rotation+25}:o));
    }
    break;

  // ═══ MYSTIC ═══
  case "mystic_shield":
    spawnRing(pos.x,pos.y,"#f59e0b",80,true);
    spawnRing(pos.x,pos.y,"#fbbf24",60,true);
    spawn(pos.x,pos.y,"#fde68a",10,30,800,[2,5]);
    if(obj){addEffect(obj.id,{type:"armor",dur:3000});applyStatus(obj.id,"armored",8);}
    // Reflects damage to nearby
    setObjects(p=>p.map(o=>{const dist=Math.sqrt((o.x+o.w/2-pos.x)**2+(o.y+o.h/2-pos.y)**2);if(dist<100&&dist>30){dealDamage(o.id,pw.dmg);return{...o,vx:o.vx+(Math.random()-0.5)*6};}return o;}));
    break;
  case "chaos_magic":
    spawnPsi(pos.x,pos.y,"#dc2626",20,60);
    spawnPostFx("tint",{color:"#dc2626",life:250});
    doShake(0.5);
    createZone(pos.x,pos.y,"void",2,1);
    setObjects(p=>p.map(o=>{
      const dist=Math.sqrt((o.x+o.w/2-pos.x)**2+(o.y+o.h/2-pos.y)**2);
      if(dist<150){
        const r=Math.random();
        dealDamage(o.id,pw.dmg);
        if(r<0.25) return{...o,scale:0.3+Math.random()*1.7};
        if(r<0.5) return{...o,mat:MATERIALS[Math.floor(Math.random()*MATERIALS.length)]};
        if(r<0.75) return{...o,vx:(Math.random()-0.5)*20,vy:(Math.random()-0.5)*20,rotation:Math.random()*360};
        return{...o,hp:Math.min(o.maxHp,o.hp+50)};
      }
      return o;
    }));
    break;
  case "order_magic":
    spawn(pos.x,pos.y,"#fbbf24",12,40,700,[3,7]);
    spawnRing(pos.x,pos.y,"#fde68a",100,true);
    if(obj){
      addEffect(obj.id,{type:"armor",dur:2500});
      applyStatus(obj.id,"armored",6);
      spawnBeamFx(pos.x,pos.y-50,cx,cy,"#fbbf24",4,"energy");
      dealDamage(obj.id,eDmg);
    }
    break;
  case "sorcery": {
    // Random effect from pool
    const effects=["burn","freeze","shock","hex","slash","diamond"];
    const pick=effects[Math.floor(Math.random()*effects.length)];
    spawn(pos.x,pos.y,"#c084fc",12,35,600,[3,7]);
    if(obj){addEffect(obj.id,{type:pick,dur:1500});dealDamage(obj.id,eDmg);}
    break;
  }
  // ═══ v5.0 POWERS ═══
  case "gravity_well":
    createZone(pos.x,pos.y,"gravity",3,0.9);
    spawnRing(pos.x,pos.y,"#6366f1",120,true); spawn(pos.x,pos.y,"#6366f1",15,40,600,[3,6]);
    doShake(0.3);
    setObjects(p=>p.map(o=>{const dx=pos.x-(o.x+o.w/2),dy=pos.y-(o.y+o.h/2),dist=Math.sqrt(dx*dx+dy*dy)||1;if(dist<200){const f=6*(1-dist/200);return{...o,vx:o.vx+(dx/dist)*f,vy:o.vy+(dy/dist)*f};}return o;}));
    break;
  case "time_dilation":
    createZone(pos.x,pos.y,"time",2,0.8);
    spawnRing(pos.x,pos.y,"#fbbf24",80,false); spawn(pos.x,pos.y,"#fbbf24",10,30,800,[2,5]);
    break;
  case "transmutation":
    if(obj){
      const matOrder=["metal","wood","glass","stone","water","organic","energy","rubber","ice","sand","crystal"];
      const idx2=matOrder.indexOf(obj.mat);
      const nextMat=matOrder[(idx2+1)%matOrder.length];
      setObjects(p=>p.map(o=>o.id===obj.id?{...o,mat:nextMat,label:nextMat[0].toUpperCase()+nextMat.slice(1),hp:MAT_HP[nextMat]||100,maxHp:MAT_HP[nextMat]||100,fragile:!!MAT_FRAGILE[nextMat],mass:o.w*o.h*(MAT_PROPS[nextMat]?.density||1)*0.01}:o));
      spawnRing(cx,cy,"#f59e0b",50,true); spawn(cx,cy,"#f59e0b",8,20,400,[2,4]);
    }
    break;
  case "duplication":
    if(obj){
      const clone={...obj,id:Date.now()+Math.random(),x:obj.x+(Math.random()-0.5)*30,y:obj.y-20,vx:(Math.random()-0.5)*4,vy:-3,statuses:[...obj.statuses],effects:[]};
      setObjects(p=>p.length<50?[...p,clone]:p);
      spawn(cx,cy,"#8b5cf6",10,25,400,[2,5]); spawnRing(cx,cy,"#8b5cf6",40,false);
    }
    break;
  case "vine_growth": {
    const vc=4+Math.floor(Math.random()*3); const vines=[];
    for(let vi=0;vi<vc;vi++){const vx2=pos.x+vi*25-vc*12,vy2=pos.y-vi*20;
      vines.push({id:Date.now()+vi+Math.random(),x:vx2,y:vy2,w:20,h:20,mat:"organic",label:"Vine",shape:"circle",origX:vx2,origY:vy2,origMat:"organic",scale:1,rotation:0,opacity:1,effects:[],statuses:[],vx:(Math.random()-0.5)*2,vy:-2,hp:40,maxHp:40,fragile:false,temp:20,mass:20*20*0.7*0.01,resting:0});}
    setObjects(p=>p.length+vines.length<=50?[...p,...vines]:p);
    for(let vi=1;vi<vines.length;vi++){jointsRef.current.push({id:nextJointId.current++,type:"rope",objA:vines[vi-1].id,objB:vines[vi].id,length:30,damping:0.2,breakForce:200});}
    spawn(pos.x,pos.y,"#22c55e",12,30,500,[2,5]); break;
  }
  case "sand_control": {
    const sands=[];
    for(let si=0;si<5;si++){const sx=pos.x-50+si*25;sands.push({id:Date.now()+si+Math.random(),x:sx,y:pos.y,w:25,h:25,mat:"sand",label:"Sand",shape:"rect",origX:sx,origY:pos.y,origMat:"sand",scale:1,rotation:0,opacity:1,effects:[],statuses:[],vx:0,vy:0,hp:50,maxHp:50,fragile:false,temp:20,mass:25*25*1.5*0.01,resting:0});}
    setObjects(p=>p.length+sands.length<=50?[...p,...sands]:p);
    spawn(pos.x,pos.y,"#d4b483",15,40,500,[3,6]); break;
  }
  case "crystal_shield": {
    const crystals=[];
    for(let ci=0;ci<3;ci++){const cx2=pos.x-30+ci*30;crystals.push({id:Date.now()+ci+Math.random(),x:cx2,y:pos.y,w:20,h:50,mat:"crystal",label:"Crystal",shape:"diamond",origX:cx2,origY:pos.y,origMat:"crystal",scale:1,rotation:0,opacity:1,effects:[],statuses:[],vx:0,vy:0,hp:60,maxHp:60,fragile:true,temp:20,mass:20*50*2.5*0.01,resting:0});}
    setObjects(p=>p.length+crystals.length<=50?[...p,...crystals]:p);
    spawnRing(pos.x,pos.y,"#e1bee7",60,true); spawn(pos.x,pos.y,"#e1bee7",10,25,400,[2,5]); break;
  }
  case "rubber_bounce":
    if(obj){
      const _origMat=obj.mat;
      setObjects(p=>p.map(o=>o.id===obj.id?{...o,mat:"rubber",label:"Bouncy"}:o));
      spawn(cx,cy,"#2d2d2d",8,20,300,[2,4]); spawnRing(cx,cy,"#2d2d2d",30,false);
      setTimeout(()=>setObjects(p=>p.map(o=>o.id===obj.id&&o.mat==="rubber"?{...o,mat:_origMat,label:_origMat}:o)),5000);
    }
    break;
  case "hellfire_chains":
    if(obj){
      spawnBeamFx(pos.x,pos.y,cx,cy,"#f97316",3,"energy");
      spawnFire(cx,cy,8,20,600);
      addEffect(obj.id,{type:"burn",dur:2000});
      applyStatus(obj.id,"burning",5);
      setObjects(p=>p.map(o=>o.id===obj.id?{...o,vx:0,vy:0,statuses:o.statuses.filter(s=>s.type!=="armored")}:o));
      dealDamage(obj.id,eDmg);
    }
    break;
  case "blood_magic":
    if(obj){
      addEffect(obj.id,{type:"drain",dur:1200});
      spawn(cx,cy,"#b91c1c",10,25,600,[3,6]);
      dealDamage(obj.id,eDmg);
      // Heal nearby objects
      setObjects(p=>p.map(o=>{if(o.id===obj.id)return o;const dist=Math.sqrt((o.x+o.w/2-cx)**2+(o.y+o.h/2-cy)**2);if(dist<100)return{...o,hp:Math.min(o.maxHp,o.hp+15)};return o;}));
    }
    break;
  case "rune_magic":
    spawnRing(pos.x,pos.y,"#22d3ee",80,true);
    spawn(pos.x,pos.y,"#67e8f9",10,40,1000,[3,6]);
    createZone(pos.x,pos.y,"ice",1,0.5);
    // Buff objects inside, debuff outside
    setObjects(p=>p.map(o=>{const dist=Math.sqrt((o.x+o.w/2-pos.x)**2+(o.y+o.h/2-pos.y)**2);if(dist<80){applyStatus(o.id,"armored",4);return{...o,hp:Math.min(o.maxHp,o.hp+10)};}if(dist<160){applyStatus(o.id,"corroding",3);dealDamage(o.id,pw.dmg);}return o;}));
    break;
  case "eldritch_blast":
    spawnBeamFx(0,0,pos.x,pos.y,"#7c3aed",8,"psi");
    spawnPsi(pos.x,pos.y,"#7c3aed",15,50);
    spawnShockwave(pos.x,pos.y,100,"#7c3aed");
    spawnPostFx("vignette",{life:200}); doShake(0.4);
    if(obj){dealDamage(obj.id,eDmg);addEffect(obj.id,{type:"hex",dur:800});}
    setObjects(p=>p.map(o=>{const dist=Math.sqrt((o.x+o.w/2-pos.x)**2+(o.y+o.h/2-pos.y)**2);if(dist<120){dealDamage(o.id,Math.floor(pw.dmg*0.5));return{...o,vx:o.vx+(Math.random()-0.5)*8};}return o;}));
    break;

  // ═══ COSMIC ═══
  case "infinity_snap": {
    doShake(1);spawnPostFx("vignette",{life:600});spawnPostFx("tint",{color:"#c084fc",life:500});
    spawn(pos.x,pos.y,"#c084fc",30,120,1000,[4,12]);
    spawnRing(pos.x,pos.y,"#fde68a",250,true);
    setObjects(p=>{
      const shuffled=[...p].sort(()=>Math.random()-0.5);
      const half=shuffled.slice(0,Math.ceil(shuffled.length/2));
      const ids=new Set(half.map(o=>o.id));
      half.forEach(o=>{spawn(o.x+o.w/2,o.y+o.h/2,"#c084fc",15,40,800,[3,8]);spawnSmoke(o.x+o.w/2,o.y+o.h/2,"#a855f7",6,20);});
      return p.filter(o=>!ids.has(o.id));
    });
    break;
  }
  case "nova_force":
    spawnRing(pos.x,pos.y,"#fbbf24",150,true);
    spawn(pos.x,pos.y,"#fde68a",20,80,700,[4,10]);
    spawnShockwave(pos.x,pos.y,150,"#fbbf24"); doShake(0.5);
    setObjects(p=>p.map(o=>{const dx=(o.x+o.w/2)-pos.x,dy=(o.y+o.h/2)-pos.y,dist=Math.sqrt(dx*dx+dy*dy)||1;if(dist<170){dealDamage(o.id,pw.dmg);return{...o,vx:o.vx+(dx/dist)*12,vy:o.vy+(dy/dist)*12};}return o;}));
    break;
  case "lantern_construct":
    spawnRing(pos.x,pos.y,"#22c55e",70,true);
    spawn(pos.x,pos.y,"#86efac",12,30,1200,[3,6]);
    // Objects inside get armored
    setObjects(p=>p.map(o=>{const dist=Math.sqrt((o.x+o.w/2-pos.x)**2+(o.y+o.h/2-pos.y)**2);if(dist<80){applyStatus(o.id,"armored",6);return{...o,vx:0,vy:0};}return o;}));
    if(obj) dealDamage(obj.id,eDmg);
    break;
  case "anti_life":
    spawnPsi(pos.x,pos.y,"#4b0082",15,60);
    spawnPostFx("vignette",{life:300}); doShake(0.3);
    createZone(pos.x,pos.y,"void",3,1);
    createZone(pos.x,pos.y,"toxic",2,0.7);
    setObjects(p=>p.map(o=>{const dist=Math.sqrt((o.x+o.w/2-pos.x)**2+(o.y+o.h/2-pos.y)**2);if(dist<160){applyStatus(o.id,"corroding",5);dealDamage(o.id,pw.dmg);}return o;}));
    break;
  case "starheart":
    spawnFire(pos.x,pos.y,15,50,900);
    spawn(pos.x,pos.y,"#4ade80",10,40,700,[3,7]);
    spawnRing(pos.x,pos.y,"#22c55e",100,true);
    createZone(pos.x,pos.y,"fire",1,0.7);
    if(obj){dealDamage(obj.id,eDmg);applyStatus(obj.id,"burning",4);}
    break;
  case "celestial_beam":
    spawnBeamFx(pos.x,0,pos.x,pos.y,"#fde68a",10,"energy");
    spawnShockwave(pos.x,pos.y,180,"#fde68a");
    spawnRing(pos.x,pos.y,"#fbbf24",200,true);
    spawnPostFx("tint",{color:"#fde68a",life:300}); doShake(0.8);
    setObjects(p=>p.map(o=>{const dist=Math.sqrt((o.x+o.w/2-pos.x)**2+(o.y+o.h/2-pos.y)**2);if(dist<200){dealDamage(o.id,pw.dmg);return{...o,vx:o.vx+(Math.random()-0.5)*15,vy:o.vy-10};}return o;}));
    break;

  // ═══ TECH ═══
  case "repulsor":
    if(obj){
      tryReflect(obj,powerId,pos);
      spawnBeamFx(pos.x-60,pos.y+20,cx,cy,"#38bdf8",5,"energy");
      spawn(cx,cy,"#93c5fd",10,30,500,[3,6]); spawnHitFlash(cx,cy,pw.dmg);
      dealDamage(obj.id,eDmg);
      addEffect(obj.id,{type:"hit",dur:400});
      setObjects(p=>p.map(o=>o.id===obj.id?{...o,vx:o.vx+(cx-pos.x)*0.08,vy:o.vy+(cy-pos.y)*0.08}:o));
    }
    break;
  case "batarang": {
    const targets=(objectsRef.current||[]).sort((a,b)=>{const da=Math.sqrt((a.x+a.w/2-pos.x)**2+(a.y+a.h/2-pos.y)**2);const db=Math.sqrt((b.x+b.w/2-pos.x)**2+(b.y+b.h/2-pos.y)**2);return da-db;}).slice(0,2);
    targets.forEach((t,i)=>{
      const tx=t.x+t.w/2,ty=t.y+t.h/2;
      setTimeout(()=>{
        const fx=i>0?targets[i-1].x+targets[i-1].w/2:pos.x,fy=i>0?targets[i-1].y+targets[i-1].h/2:pos.y;
        spawnBeamFx(fx,fy,tx,ty,"#6b7280",2,"slash");
        spawnHitFlash(tx,ty,pw.dmg); dealDamage(t.id,eDmg);
      },i*150);
    });
    break;
  }
  case "sonic_cannon":
    spawnRing(pos.x,pos.y,"#60a5fa",180);
    spawn(pos.x,pos.y,"#93c5fd",15,100,600,[3,7]);
    doShake(0.3);
    setObjects(p=>p.map(o=>{const dx=(o.x+o.w/2)-pos.x,dist=Math.abs(dx)||1;if(dist<200){if(dist<100)dealDamage(o.id,pw.dmg);const push=(dx>0?1:-1)*Math.min(15,2000/dist);return{...o,vx:o.vx+push};}return o;}));
    break;
  case "nano_swarm":
    if(obj){
      spawn(cx,cy,"#ef4444",20,30,1200,[1,3]);
      addEffect(obj.id,{type:"dissolve",dur:1500});
      dealDamage(obj.id,eDmg);
      setTimeout(()=>{
        setObjects(p=>p.map(o=>o.id===obj.id?{...o,hp:Math.min(o.maxHp,o.hp+15),mat:"metal"}:o));
        spawn(cx,cy,"#22c55e",10,20,500,[2,4]);
      },1500);
    }
    break;
  case "technopathy":
    if(obj){
      setObjects(p=>p.map(o=>o.id===obj.id?{...o,mat:"metal",hp:Math.min(o.hp+20,MAT_HP.metal),maxHp:MAT_HP.metal}:o));
      spawn(cx,cy,"#22c55e",10,25,500,[2,5]);
      addEffect(obj.id,{type:"steel",dur:1500});
      dealDamage(obj.id,eDmg);
    }
    break;
  case "power_ring":
    if(obj){
      spawnRing(cx,cy,"#22c55e",50,true);
      spawn(cx,cy,"#86efac",8,20,800,[3,5]);
      setObjects(p=>p.map(o=>o.id===obj.id?{...o,vx:0,vy:0}:o));
      applyStatus(obj.id,"armored",6);
      addEffect(obj.id,{type:"armor",dur:2500});
      dealDamage(obj.id,eDmg);
    }
    break;
  case "web_bomb":
    spawn(pos.x,pos.y,"#d4d4d8",20,60,1000,[2,5]);
    spawnRing(pos.x,pos.y,"#e5e7eb",80);
    setObjects(p=>p.map(o=>{const dist=Math.sqrt((o.x+o.w/2-pos.x)**2+(o.y+o.h/2-pos.y)**2);if(dist<100){dealDamage(o.id,pw.dmg);return{...o,vx:o.vx*0.1,vy:o.vy*0.1};}return o;}));
    break;

  // ═══ DARK ═══
  case "shadow_form":
    spawnSmoke(pos.x,pos.y,"#1e1b4b",15,50);
    spawnPostFx("vignette",{life:300});
    createZone(pos.x,pos.y,"void",2,0.8);
    if(obj){dealDamage(obj.id,eDmg);addEffect(obj.id,{type:"drain",dur:1500});}
    break;
  case "soul_self":
    if(obj){
      spawnPsi(cx,cy,"#6366f1",15,45);
      spawnBeamFx(pos.x,pos.y,cx,cy,"#6366f1",5,"psi");
      spawnPostFx("tint",{color:"#1e1b4b",life:200});
      dealDamage(obj.id,eDmg);
      addEffect(obj.id,{type:"hex",dur:800});
      setObjects(p=>p.map(o=>o.id===obj.id?{...o,vx:o.vx+(Math.random()-0.5)*10,vy:o.vy-5}:o));
    }
    break;
  case "necroplasm":
    spawnFire(pos.x,pos.y,15,50,1000);
    spawn(pos.x,pos.y,"#16a34a",8,30,600,[3,7]);
    createZone(pos.x,pos.y,"fire",1,0.8);
    createZone(pos.x,pos.y,"toxic",1,0.6);
    doShake(0.3);
    if(obj){dealDamage(obj.id,eDmg);applyStatus(obj.id,"burning",5);applyStatus(obj.id,"corroding",3);}
    break;
  case "void_touch":
    if(obj){
      spawnPsi(cx,cy,"#fbbf24",8,20);
      spawn(cx,cy,"#1e1b4b",15,40,600,[3,8]);
      spawnPostFx("vignette",{life:250});
      addEffect(obj.id,{type:"dissolve",dur:1200});
      dealDamage(obj.id,eDmg);
      setObjects(p=>p.map(o=>o.id===obj.id?{...o,scale:Math.max(0.2,o.scale-0.2)}:o));
    }
    break;
  case "fear_toxin":
    spawnSmoke(pos.x,pos.y,"#84cc16",15,50);
    createZone(pos.x,pos.y,"toxic",2,0.9);
    setObjects(p=>p.map(o=>{const dist=Math.sqrt((o.x+o.w/2-pos.x)**2+(o.y+o.h/2-pos.y)**2);if(dist<130){applyStatus(o.id,"corroding",4);dealDamage(o.id,pw.dmg);return{...o,vx:o.vx+(Math.random()-0.5)*12,vy:o.vy+(Math.random()-0.5)*12};}return o;}));
    break;
  case "darkforce":
    spawnPsi(pos.x,pos.y,"#7c3aed",15,50);
    spawnBeamFx(pos.x-100,pos.y-100,pos.x,pos.y,"#7c3aed",6,"psi");
    spawnShockwave(pos.x,pos.y,100,"#7c3aed");
    doShake(0.3); spawnPostFx("tint",{color:"#1e1b4b",life:200});
    if(obj){dealDamage(obj.id,eDmg);addEffect(obj.id,{type:"hit",dur:500});}
    setObjects(p=>p.map(o=>{const dist=Math.sqrt((o.x+o.w/2-pos.x)**2+(o.y+o.h/2-pos.y)**2);if(dist<120)return{...o,vx:o.vx+(Math.random()-0.5)*8,vy:o.vy+(Math.random()-0.5)*8};return o;}));
    break;
  case "black_lantern":
    spawnRing(pos.x,pos.y,"#333",130,true);
    spawn(pos.x,pos.y,"#1e1b4b",20,60,800,[4,8]);
    spawnPostFx("vignette",{life:400}); doShake(0.4);
    createZone(pos.x,pos.y,"void",2,1);
    setObjects(p=>p.map(o=>{const dist=Math.sqrt((o.x+o.w/2-pos.x)**2+(o.y+o.h/2-pos.y)**2);if(dist<150){dealDamage(o.id,pw.dmg);applyStatus(o.id,"corroding",5);return{...o,scale:Math.max(0.3,o.scale-0.1)};}return o;}));
    break;

  default:
    if(obj){spawn(cx,cy,c,10,40,600,[3,7]);dealDamage(obj.id,pw.dmg||10);}
}
```

}, [bounds,spawn,spawnFire,spawnIce,spawnPsi,spawnSmoke,spawnSlash,spawnBeamFx,spawnRing,spawnGroundFx,spawnShockwave,spawnSlashImpact,spawnHitFlash,spawnPostFx,spawnLightning,doShake,addEffect,applyStatus,dealDamage,recordDiscovery,createZone,playPowerSound,env,tryReflect]);

// ─── APPLY COMBO (v3.5 — 28 combos) ───
const applyCombo = useCallback((combo, obj, pos) => {
const cx=obj?obj.x+obj.w/2:pos.x, cy=obj?obj.y+obj.h/2:pos.y;
setComboFlash(combo);
setTimeout(()=>setComboFlash(null),1500);
recordDiscovery(`combo:${combo.effect}`);
playComboSound(combo.effect);

```
switch(combo.effect) {
  case "charged_slash":
    if(obj){
      addEffect(obj.id,{type:"charge",dur:600});addEffect(obj.id,{type:"slash",dur:600});
      spawnPsi(cx,cy,"#d946ef",12,50); spawnSlash(cx,cy,Math.random()*Math.PI*2,"#e5e7eb",70); spawnSlashImpact(cx,cy,Math.random()*Math.PI*2,"#d946ef",2);
      doShake(0.5);dealDamage(obj.id,combo.dmg);
      setObjects(p=>p.map(o=>o.id===obj.id?{...o,rotation:o.rotation+30}:o));
    }
    break;
  case "fire_tornado":
    spawnFire(pos.x,pos.y,25,80,1000);
    spawnFire(pos.x,pos.y,15,60,800); spawnRing(pos.x,pos.y,"#fb923c",80,true);
    doShake(0.5);createZone(pos.x,pos.y,"fire",3,1);
    setObjects(p=>p.map(o=>{
      const dx=(o.x+o.w/2)-pos.x,dy=(o.y+o.h/2)-pos.y,dist=Math.sqrt(dx*dx+dy*dy)||1;
      if(dist<160){const a=Math.atan2(dy,dx)+0.4;applyStatus(o.id,"burning",6);dealDamage(o.id,combo.dmg);return{...o,vx:o.vx+Math.cos(a)*10,vy:o.vy+Math.sin(a)*10,rotation:o.rotation+20};}
      return o;
    }));
    break;
  case "blizzard":
    spawnIce(pos.x,pos.y,30,120);
    spawnRing(pos.x,pos.y,"#a5f3fc",140,true);doShake(0.4);
    createZone(pos.x,pos.y,"ice",3,1);
    setObjects(p=>p.map(o=>{
      const dist=Math.sqrt((o.x+o.w/2-pos.x)**2+(o.y+o.h/2-pos.y)**2);
      if(dist<180){applyStatus(o.id,"frozen",8);dealDamage(o.id,combo.dmg);return{...o,vx:o.vx+(Math.random()-0.5)*8};}
      return o;
    }));
    break;
  case "prism":
    if(obj){
      spawnBeamFx(0,bounds.h*0.4,cx,cy,"#dc2626",6,"energy");
      for(let i=0;i<6;i++){const a=(Math.PI*2*i)/6,ex=cx+Math.cos(a)*200,ey=cy+Math.sin(a)*200;setTimeout(()=>spawnBeamFx(cx,cy,ex,ey,["#ef4444","#f97316","#fbbf24","#22c55e","#3b82f6","#a855f7"][i],3,"rainbow"),200);}
      spawn(cx,cy,"#e0f2fe",20,50,800,[3,8]);doShake(0.5);dealDamage(obj.id,combo.dmg);
    }
    break;
  case "railgun": {
    let projSnap=null;
    setObjects(p=>{const metals=p.filter(o=>o.mat==="metal");if(metals.length>0){projSnap={x:metals[0].x+metals[0].w/2,y:metals[0].y+metals[0].h/2,id:metals[0].id};return p.map(o=>o.id===metals[0].id?{...o,vx:(pos.x>cx?25:-25),vy:-10}:o);}return p;});
    if(projSnap){setTimeout(()=>{spawn(projSnap.x,projSnap.y,"#a855f7",15,40,500,[3,7]);dealDamage(projSnap.id,combo.dmg);},0);}
    doShake(0.6);
    break;
  }
  case "bamf_blitz":
    if(obj){
      for(let i=0;i<4;i++) setTimeout(()=>{spawnSmoke(obj.x+(Math.random()-0.5)*100,obj.y+(Math.random()-0.5)*100,"#6366f1",8,20);spawnSlashImpact(cx,cy,Math.random()*Math.PI*2,"#6366f1");addEffect(obj.id,{type:"slash",dur:250});dealDamage(obj.id,Math.floor(combo.dmg/4));},i*200);
      doShake(0.4);createZone(cx,cy,"toxic",1,0.6);
    }
    break;
  case "sonic_laser":
    spawnRing(pos.x,pos.y,"#86efac",100);
    for(let i=0;i<5;i++){const a=(Math.random()-0.5)*1.5,ex=pos.x+Math.cos(a)*300,ey=pos.y+Math.sin(a)*300;setTimeout(()=>spawnBeamFx(pos.x,pos.y,ex,ey,["#ec4899","#fbbf24","#a78bfa","#67e8f9","#86efac"][i],3),i*80);}
    doShake(0.3);if(obj)dealDamage(obj.id,combo.dmg);
    break;
  case "thermal_shock":
    if(obj){
      spawnFire(cx,cy,15,40,500);spawnIce(cx,cy,15,40);
      addEffect(obj.id,{type:"shock",dur:800});doShake(0.5);dealDamage(obj.id,combo.dmg);
      recordDiscovery("status:fire+ice");
    }
    break;
  case "phoenix_storm":
    spawnFire(pos.x,pos.y,40,160,1200);spawnRing(pos.x,pos.y,"#f59e0b",220,true);spawnShockwave(pos.x,pos.y,220,"#f59e0b");spawnPostFx("vignette",{life:400});doShake(1);
    createZone(pos.x,pos.y,"fire",3,1);createZone(pos.x,pos.y,"void",2,0.7);
    setObjects(p=>p.map(o=>{const dx=(o.x+o.w/2)-pos.x,dy=(o.y+o.h/2)-pos.y,dist=Math.sqrt(dx*dx+dy*dy)||1;applyStatus(o.id,"burning",6);dealDamage(o.id,combo.dmg);return{...o,vx:o.vx+(dx/dist)*Math.min(25,8000/dist),vy:o.vy+(dy/dist)*Math.min(25,8000/dist)};}));
    break;
  case "chaos":
    spawn(pos.x,pos.y,"#dc2626",20,80,900,[4,10]);spawn(pos.x,pos.y,"#34d399",15,60,700,[3,8]);doShake(0.7);
    createZone(pos.x,pos.y,"void",3,1);
    setObjects(p=>p.map(o=>{const newMats=MATERIALS.filter(m=>m!==o.mat);dealDamage(o.id,combo.dmg);return{...o,mat:newMats[Math.floor(Math.random()*newMats.length)],scale:0.5+Math.random()*1.5,vx:(Math.random()-0.5)*20,vy:(Math.random()-0.5)*20,rotation:Math.random()*360};}));
    break;
  case "metal_barrage": {
    let metalSnaps=[];
    setObjects(p=>{metalSnaps=p.filter(o=>o.mat==="metal").map(o=>({id:o.id,x:o.x+o.w/2,y:o.y+o.h/2}));return p.map(o=>o.mat==="metal"?{...o,vx:(Math.random()-0.5)*15,vy:-8-Math.random()*10}:o);});
    setTimeout(()=>metalSnaps.forEach(m=>{addEffect(m.id,{type:"charge",dur:800});spawn(m.x,m.y,"#d946ef",8,20,500,[3,6]);dealDamage(m.id,combo.dmg);}),0);
    doShake(0.5);createZone(pos.x,pos.y,"magnetic",2,0.7);
    break;
  }
  case "phase_lightning": {
    setObjects(p=>p.map(o=>({...o,opacity:0.3})));
    setTimeout(()=>setObjects(p=>{p.forEach(o=>applyStatus(o.id,"phased",2));return p;}),0);
    setTimeout(()=>setObjects(p=>{p.forEach(o=>{applyStatus(o.id,"electrified",4);dealDamage(o.id,combo.dmg);spawnLightning(o.x+o.w/2,0,o.x+o.w/2,o.y+o.h/2,"#93c5fd");});return p;}),500);
    setTimeout(()=>setObjects(p=>p.map(o=>({...o,opacity:1}))),1300);
    createZone(pos.x,pos.y,"electric",3,0.9);
    break;
  }
  case "overcharged":
    spawn(pos.x,pos.y,"#fbbf24",10,30,400,[3,6]);
    setTimeout(()=>{spawnBeamFx(pos.x,pos.y,bounds.w,pos.y,"#dc2626",10);spawn(bounds.w/2,pos.y,"#fbbf24",30,100,800,[5,12]);doShake(0.8);
      setObjects(p=>p.map(o=>{const dy=Math.abs((o.y+o.h/2)-pos.y);if(dy<60){dealDamage(o.id,combo.dmg);return{...o,vx:o.vx+20};}return o;}));
    },600);
    break;
  case "fastball":
    if(obj){
      spawn(cx-30,cy,"#94a3b8",8,20,300,[3,6]);
      setObjects(p=>p.map(o=>o.id===obj.id?{...o,vx:30,vy:-5}:o));
      doShake(0.4);dealDamage(obj.id,combo.dmg);
      setTimeout(()=>{spawn(cx+200,cy-30,"#e5e7eb",20,60,600,[4,9]);doShake(0.6);},400);
    }
    break;

  // ═══ NEW DC+MARVEL COMBOS ═══
  case "thermal_vision":
    // Heat Vision + Freeze Breath = flash-freeze shatter
    if(obj){
      spawnBeamFx(cx-40,0,cx,cy,"#ef4444",4,"energy");
      spawnBeamFx(cx+40,0,cx,cy,"#a5f3fc",4,"frost");
      spawnFire(cx,cy,8,20,400); spawnIce(cx,cy,12,30);
      doShake(0.6); dealDamage(obj.id,combo.dmg);
      addEffect(obj.id,{type:"shock",dur:800});
      recordDiscovery("status:fire+ice");
      if(MAT_RULES.shatters[obj.mat]) setObjects(p=>p.map(o=>o.id===obj.id?{...o,fragile:true,hp:Math.max(0,o.hp-30)}:o));
    }
    break;
  case "thunder_god":
    // Mjolnir + Lightning = divine storm
    doShake(0.8);
    for(let i=0;i<5;i++){
      setTimeout(()=>{
        const lx=pos.x+(Math.random()-0.5)*200,ly=pos.y+(Math.random()-0.5)*100;
        spawnLightning(lx+(Math.random()-0.5)*30,0,lx,ly,"#fbbf24");
        spawnBeamFx(lx,0,lx,ly,"#fbbf24",5,"lightning");
        spawnShockwave(lx,ly,80,"#fbbf24");
        setObjects(p=>p.map(o=>{const dist=Math.sqrt((o.x+o.w/2-lx)**2+(o.y+o.h/2-ly)**2);if(dist<100){dealDamage(o.id,Math.floor(combo.dmg/3));applyStatus(o.id,"electrified",3);}return o;}));
      },i*150);
    }
    createZone(pos.x,pos.y,"electric",3,1);
    spawnPostFx("tint",{color:"#fbbf24",life:400});
    break;
  case "web_shadows":
    // Web + Symbiote = dark web trap
    spawn(pos.x,pos.y,"#1e1b4b",20,60,1000,[3,8]);
    spawn(pos.x,pos.y,"#d4d4d8",15,50,800,[2,5]);
    createZone(pos.x,pos.y,"toxic",2,0.8);
    doShake(0.3);
    setObjects(p=>p.map(o=>{const dist=Math.sqrt((o.x+o.w/2-pos.x)**2+(o.y+o.h/2-pos.y)**2);if(dist<140){dealDamage(o.id,combo.dmg);applyStatus(o.id,"corroding",5);return{...o,vx:0,vy:0};}return o;}));
    break;
  case "world_breaker":
    // Hulk Smash + Thunder Clap = absolute destruction
    doShake(1);
    spawnShockwave(pos.x,pos.y,250,"#22c55e");
    spawnShockwave(pos.x,pos.y,200,"#86efac");
    spawn(pos.x,pos.y,"#22c55e",30,120,800,[5,15],"shard",{sides:4,spin:3});
    spawnPostFx("vignette",{life:500}); spawnPostFx("tint",{color:"#22c55e",life:300});
    setObjects(p=>p.map(o=>{const dx=(o.x+o.w/2)-pos.x,dy=(o.y+o.h/2)-pos.y,dist=Math.sqrt(dx*dx+dy*dy)||1;dealDamage(o.id,combo.dmg);return{...o,vx:o.vx+(dx/dist)*Math.min(30,8000/dist),vy:o.vy-15};}));
    break;
  case "nexus_event":
    // Mystic Shield + Chaos Magic = reality collapse
    spawnRing(pos.x,pos.y,"#dc2626",200,true);
    spawnRing(pos.x,pos.y,"#f59e0b",160,true);
    spawnPsi(pos.x,pos.y,"#dc2626",25,80);
    doShake(0.9); spawnPostFx("vignette",{life:500}); spawnPostFx("tint",{color:"#dc2626",life:400});
    createZone(pos.x,pos.y,"void",3,1);
    setObjects(p=>p.map(o=>{
      dealDamage(o.id,combo.dmg);
      const r=Math.random();
      if(r<0.33)return{...o,mat:MATERIALS[Math.floor(Math.random()*MATERIALS.length)],scale:0.3+Math.random()*2};
      if(r<0.66)return{...o,x:Math.random()*bounds.w,y:Math.random()*bounds.h,rotation:Math.random()*360};
      return{...o,vx:(Math.random()-0.5)*25,vy:(Math.random()-0.5)*25};
    }));
    break;
  case "speed_lightning":
    // Speed Force + Lightning = chain lightning at Mach 10
    const targets=(objectsRef.current||[]).slice();
    targets.forEach((t,i)=>{
      const tx=t.x+t.w/2,ty=t.y+t.h/2;
      const prev=i>0?targets[i-1]:null;
      const fx=prev?prev.x+prev.w/2:pos.x,fy=prev?prev.y+prev.h/2:pos.y;
      setTimeout(()=>{
        spawnLightning(fx,fy,tx,ty,"#ef4444");
        spawn(tx,ty,"#fbbf24",4,15,200,[1,3],"spark",{trail:[]});
        dealDamage(t.id,Math.floor(combo.dmg/Math.max(1,targets.length)));
        applyStatus(t.id,"electrified",3);
      },i*60);
    });
    doShake(0.5);
    createZone(pos.x,pos.y,"electric",2,0.8);
    break;
  case "spirit_vengeance":
    // Hellfire + Chains = full Ghost Rider mode
    spawnFire(pos.x,pos.y,30,80,1200);
    spawn(pos.x,pos.y,"#22c55e",10,40,700,[3,7]);
    for(let i=0;i<3;i++){
      const a=(Math.PI*2*i)/3,ex=pos.x+Math.cos(a)*120,ey=pos.y+Math.sin(a)*120;
      spawnBeamFx(pos.x,pos.y,ex,ey,"#f97316",3,"energy");
    }
    doShake(0.6);createZone(pos.x,pos.y,"fire",2,1);spawnPostFx("tint",{color:"#f97316",life:300});
    setObjects(p=>p.map(o=>{const dist=Math.sqrt((o.x+o.w/2-pos.x)**2+(o.y+o.h/2-pos.y)**2);if(dist<140){dealDamage(o.id,combo.dmg);applyStatus(o.id,"burning",6);return{...o,vx:0,vy:0,statuses:o.statuses.filter(s=>s.type!=="armored")};}return o;}));
    break;
  case "worthy_shield":
    // Shield + Mjolnir = lightning redirect off shield
    if(obj){
      spawnBeamFx(pos.x-60,pos.y,cx,cy,"#3b82f6",3,"energy"); // shield trajectory
      spawnLightning(cx,0,cx,cy,"#60a5fa"); // lightning strikes shield
      spawnBeamFx(cx,cy,cx+150,cy-80,"#fbbf24",5,"lightning"); // redirected bolt
      spawnShockwave(cx,cy,100,"#93c5fd");
      doShake(0.5);dealDamage(obj.id,combo.dmg);
      applyStatus(obj.id,"electrified",4);
      setObjects(p=>p.map(o=>{const dx=(o.x+o.w/2)-cx,dy=(o.y+o.h/2)-cy,dist=Math.sqrt(dx*dx+dy*dy)||1;if(dist<120)return{...o,vx:o.vx+(dx/dist)*10,vy:o.vy+(dy/dist)*10};return o;}));
    }
    break;
  case "full_arsenal":
    // Repulsor + Nano Swarm = Iron Man at max
    spawnRing(pos.x,pos.y,"#ef4444",120,true);
    for(let i=0;i<4;i++){
      const a=(Math.PI*2*i)/4,ex=pos.x+Math.cos(a)*150,ey=pos.y+Math.sin(a)*150;
      setTimeout(()=>spawnBeamFx(pos.x,pos.y,ex,ey,"#38bdf8",4,"energy"),i*100);
    }
    spawn(pos.x,pos.y,"#ef4444",20,40,800,[1,3]); // nano swarm
    doShake(0.5);
    setObjects(p=>p.map(o=>{const dist=Math.sqrt((o.x+o.w/2-pos.x)**2+(o.y+o.h/2-pos.y)**2);if(dist<170){dealDamage(o.id,combo.dmg);return{...o,vx:o.vx+(Math.random()-0.5)*10};}return o;}));
    break;
  case "max_carnage":
    // Symbiote + Void Touch = symbiote explosion
    spawn(pos.x,pos.y,"#1e1b4b",25,80,900,[4,10]);
    spawn(pos.x,pos.y,"#dc2626",15,60,700,[3,8]);
    spawnShockwave(pos.x,pos.y,160,"#dc2626");
    doShake(0.7); spawnPostFx("vignette",{life:350}); spawnPostFx("tint",{color:"#dc2626",life:250});
    createZone(pos.x,pos.y,"void",2,0.9); createZone(pos.x,pos.y,"toxic",2,0.7);
    setObjects(p=>p.map(o=>{const dist=Math.sqrt((o.x+o.w/2-pos.x)**2+(o.y+o.h/2-pos.y)**2);if(dist<180){dealDamage(o.id,combo.dmg);applyStatus(o.id,"corroding",5);return{...o,scale:Math.max(0.3,o.scale-0.15)};}return o;}));
    break;
  case "apokolips":
    // Omega Beams + Anti-Life = Darkseid unleashed
    doShake(1);
    spawnPostFx("vignette",{life:600}); spawnPostFx("tint",{color:"#4b0082",life:500});
    spawnRing(pos.x,pos.y,"#dc2626",200,true);
    spawnPsi(pos.x,pos.y,"#4b0082",20,80);
    createZone(pos.x,pos.y,"void",4,1); createZone(pos.x,pos.y,"toxic",3,0.8);
    // Zigzag beams to all objects
    setObjects(p=>{
      p.forEach(o=>{
        const tx=o.x+o.w/2,ty=o.y+o.h/2;
        const arr=particlesRef.current,now=Date.now();
        const pts=[{x:pos.x,y:pos.y}];
        const dx=tx-pos.x,dy=ty-pos.y;
        for(let s=1;s<=4;s++){const t=s/5;pts.push({x:pos.x+dx*t+(s%2===0?30:-30),y:pos.y+dy*t+(s%2===0?-20:20)});}
        pts.push({x:tx,y:ty});
        arr.push({x:pos.x,y:pos.y,dx:0,dy:0,color:"#dc2626",life:400,size:4,born:now,type:"lightning",extra:{points:pts,width:4}});
        dealDamage(o.id,combo.dmg);applyStatus(o.id,"corroding",6);
      });
      return p.map(o=>({...o,vx:o.vx+(Math.random()-0.5)*15,vy:o.vy+(Math.random()-0.5)*15,scale:Math.max(0.3,o.scale-0.1)}));
    });
    break;
  case "cosmic_judgment":
    // Infinity Snap + Celestial Beam = universal reset
    doShake(1);
    spawnPostFx("vignette",{life:800}); spawnPostFx("tint",{color:"#fde68a",life:700});
    spawnBeamFx(pos.x,0,pos.x,pos.y,"#fde68a",12,"energy");
    spawnRing(pos.x,pos.y,"#c084fc",300,true);
    spawnRing(pos.x,pos.y,"#fbbf24",250,true);
    spawn(pos.x,pos.y,"#fde68a",40,150,1200,[5,15]);
    setObjects(p=>{
      p.forEach(o=>{spawn(o.x+o.w/2,o.y+o.h/2,"#c084fc",10,30,600,[3,7]);spawnSmoke(o.x+o.w/2,o.y+o.h/2,"#fde68a",4,15);});
      return []; // everything gone
    });
    break;
  case "dark_knight":
    // Batarang + Fear Toxin = terror + precision
    if(obj){
      spawnBeamFx(pos.x,pos.y,cx,cy,"#6b7280",2,"slash");
      spawnSmoke(cx,cy,"#84cc16",10,30);
      spawnHitFlash(cx,cy,combo.dmg);
      dealDamage(obj.id,combo.dmg);
      applyStatus(obj.id,"corroding",4);
      addEffect(obj.id,{type:"hex",dur:1000});
    }
    // Fear spread to nearby
    createZone(pos.x,pos.y,"toxic",1,0.7);
    setObjects(p=>p.map(o=>{const dist=Math.sqrt((o.x+o.w/2-pos.x)**2+(o.y+o.h/2-pos.y)**2);if(dist<100&&(!obj||o.id!==obj.id)){applyStatus(o.id,"corroding",3);return{...o,vx:o.vx+(Math.random()-0.5)*8};}return o;}));
    break;
  case "azarath":
    // Soul Self + Empathy = Raven unleashed
    spawnPsi(pos.x,pos.y,"#6366f1",25,80);
    spawnRing(pos.x,pos.y,"#4c1d95",150,true);
    spawnPostFx("vignette",{life:400}); spawnPostFx("tint",{color:"#6366f1",life:300});
    doShake(0.6);
    // Lift all objects and slam
    setObjects(p=>p.map(o=>({...o,vy:o.vy-10})));
    setTimeout(()=>{
      doShake(0.8);
      spawnShockwave(pos.x,pos.y,200,"#6366f1");
      setObjects(p=>p.map(o=>{dealDamage(o.id,combo.dmg);return{...o,vy:o.vy+15};}));
    },500);
    break;

  // ═══ v5.0 COMBOS ═══
  case "singularity": {
    createZone(pos.x,pos.y,"gravity",4,1.0); createZone(pos.x,pos.y,"magnetic",3,0.9);
    spawnRing(pos.x,pos.y,"#4338ca",150,true); doShake(0.8);
    setObjects(p=>p.map(o=>{const dx=pos.x-(o.x+o.w/2),dy=pos.y-(o.y+o.h/2),dist=Math.sqrt(dx*dx+dy*dy)||1;if(dist<250){dealDamage(o.id,combo.dmg*(1-dist/250));return{...o,vx:o.vx+(dx/dist)*15,vy:o.vy+(dy/dist)*15,scale:Math.max(0.3,o.scale-0.1)};}return o;}));
    break; }
  case "permafrost": {
    const iw=[];for(let i=0;i<7;i++){const sx=pos.x-70+i*20;iw.push({id:Date.now()+i+Math.random(),x:sx,y:pos.y,w:20,h:30,mat:"ice",label:"IceWall",shape:"rect",origX:sx,origY:pos.y,origMat:"ice",scale:1,rotation:0,opacity:1,effects:[],statuses:[{type:"frozen",ticksLeft:20}],vx:0,vy:0,hp:35,maxHp:35,fragile:true,temp:-10,mass:20*30*0.9*0.01,resting:0});}
    setObjects(p=>p.length+iw.length<=50?[...p,...iw]:p);
    createZone(pos.x,pos.y,"ice",3,0.9); spawnRing(pos.x,pos.y,"#a5f3fc",100,true); break; }
  case "wildfire": {
    const fv=[];for(let i=0;i<5;i++){const vx=pos.x+(Math.random()-0.5)*100,vy=pos.y+(Math.random()-0.5)*60;fv.push({id:Date.now()+i+Math.random(),x:vx,y:vy,w:18,h:18,mat:"organic",label:"BurningVine",shape:"circle",origX:vx,origY:vy,origMat:"organic",scale:1,rotation:0,opacity:1,effects:[],statuses:[{type:"burning",ticksLeft:8}],vx:(Math.random()-0.5)*6,vy:-2,hp:30,maxHp:30,fragile:false,temp:200,mass:18*18*0.7*0.01,resting:0});}
    setObjects(p=>p.length+fv.length<=50?[...p,...fv]:p);
    createZone(pos.x,pos.y,"fire",3,0.8); doShake(0.4); break; }
  case "prism_burst": {
    for(let a=0;a<6;a++){const angle=a*Math.PI/3,range=150,tx=pos.x+Math.cos(angle)*range,ty=pos.y+Math.sin(angle)*range;spawnBeamFx(pos.x,pos.y,tx,ty,["#f0f","#0ff","#ff0","#f00","#0f0","#00f"][a],3,"energy");}
    spawnRing(pos.x,pos.y,"#e879f9",80,true); spawn(pos.x,pos.y,"#e879f9",20,50,500,[3,7]);
    setObjects(p=>p.map(o=>{const dd=(o.x+o.w/2-pos.x)**2+(o.y+o.h/2-pos.y)**2;if(dd<150*150)dealDamage(o.id,combo.dmg/3);return o;})); break; }
  case "time_stop": {
    createZone(pos.x,pos.y,"time",4,1.0);
    setObjects(p=>p.map(o=>{const dd=(pos.x-(o.x+o.w/2))**2+(pos.y-(o.y+o.h/2))**2;if(dd<180*180)return{...o,vx:0,vy:0};return o;}));
    spawnRing(pos.x,pos.y,"#fbbf24",180,false); spawn(pos.x,pos.y,"#fbbf24",15,40,800,[3,7]); break; }
  case "clone_army": {
    const near=objectsRef.current.filter(o=>(o.x+o.w/2-pos.x)**2+(o.y+o.h/2-pos.y)**2<150*150);
    const cl=near.slice(0,5).map((o,i)=>({...o,id:Date.now()+i+Math.random(),x:o.x+(Math.random()-0.5)*60,y:o.y-30,vx:(Math.random()-0.5)*8,vy:-4-Math.random()*4,statuses:[],effects:[]}));
    setObjects(p=>p.length+cl.length<=50?[...p,...cl]:p);
    spawnRing(pos.x,pos.y,"#8b5cf6",100,true); spawn(pos.x,pos.y,"#8b5cf6",20,50,500,[3,7]); doShake(0.3); break; }
  default:
    spawn(pos.x,pos.y,combo.color,20,80,800,[4,10]);
}
```

}, [bounds,spawn,spawnFire,spawnIce,spawnPsi,spawnSmoke,spawnLightning,spawnSlash,spawnBeamFx,spawnRing,spawnGroundFx,spawnShockwave,spawnSlashImpact,spawnHitFlash,spawnPostFx,doShake,addEffect,applyStatus,dealDamage,recordDiscovery,createZone,playComboSound]);

// ─── POWER-ON-POWER INTERACTIONS (v4.5 — Item 1) ───
const registerActiveEffect = useCallback((type, props) => {
if(activeEffectsRef.current.length>=EFFECT_CAP)activeEffectsRef.current.shift();
activeEffectsRef.current.push({…props, type, born:Date.now(), id:Date.now()+Math.random()});
}, []);

const tickPowerInteractions = useCallback((now) => {
const fx = activeEffectsRef.current;
// Cleanup expired
activeEffectsRef.current = fx.filter(e => now - e.born < (e.life||1500));
const live = activeEffectsRef.current;
if(live.length < 2 || now - lastPwrCollRef.current < 200) return;

```
for(let i=0;i<live.length;i++){
  for(let j=i+1;j<live.length;j++){
    const a=live[i], b=live[j];
    const coolKey=a.element+'_'+b.element;
    if(pwrCollCooldowns.current[coolKey]&&now-pwrCollCooldowns.current[coolKey]<500) continue;

    // Zone×Zone overlap
    if(a.type==='zone'&&b.type==='zone'){
      const d=Math.sqrt((a.cx-b.cx)**2+(a.cy-b.cy)**2);
      if(d > (a.radius||60)+(b.radius||60)) continue;
      const mx=(a.cx+b.cx)/2, my=(a.cy+b.cy)/2;
      pwrCollCooldowns.current[coolKey]=now;
      lastPwrCollRef.current=now;

      // Fire × Ice → steam
      if((a.element==='fire'&&b.element==='ice')||(a.element==='ice'&&b.element==='fire')){
        spawnSmoke(mx,my,'#d1d5db',12,35);
        spawnRing(mx,my,'#e5e7eb',40,false);
        a.life=0; b.life=0;
        recordDiscovery('power_collision:fire+ice');
        continue;
      }
      // Fire × Fire → merge
      if(a.element==='fire'&&b.element==='fire'){
        a.radius=(a.radius||60)*1.5; b.life=0;
        spawnFire(mx,my,8,30,400);
        recordDiscovery('power_collision:fire+fire');
        continue;
      }
      // Electric × Water → supercharge
      if((a.element==='electric'&&b.element==='water')||(a.element==='water'&&b.element==='electric')){
        const wz=a.element==='water'?a:b;
        wz.element='electric'; wz.radius=(wz.radius||60)*2;
        spawn(mx,my,'#38bdf8',8,30,400,[2,5],'spark',{trail:[]});
        recordDiscovery('power_collision:electric+water');
        continue;
      }
      // Dark × Energy → void zone
      if((a.element==='dark'&&b.element==='energy')||(a.element==='energy'&&b.element==='dark')){
        createZone(mx,my,'void',2,0.9);
        spawnPsi(mx,my,'#4b0082',15,50);
        recordDiscovery('power_collision:dark+energy');
        continue;
      }
      // Psychic × Psychic → feedback burst
      if(a.element==='psychic'&&b.element==='psychic'){
        spawnShockwave(mx,my,120,'#a855f7');
        doShake(0.5);
        setObjects(p=>p.map(o=>{const dx=o.x+o.w/2-mx,dy=o.y+o.h/2-my,dd=Math.sqrt(dx*dx+dy*dy)||1;if(dd<120)return{...o,vx:o.vx+dx/dd*8,vy:o.vy+dy/dd*8};return o;}));
        a.life=0; b.life=0;
        recordDiscovery('power_collision:psychic+psychic');
        continue;
      }
    }

    // Beam×Beam intersection
    if(a.type==='beam'&&b.type==='beam'){
      // Parametric line intersection
      const x1=a.x1,y1=a.y1,x2=a.x2,y2=a.y2,x3=b.x1,y3=b.y1,x4=b.x2,y4=b.y2;
      const den=(x1-x2)*(y3-y4)-(y1-y2)*(x3-x4);
      if(Math.abs(den)<0.001) continue;
      const t=((x1-x3)*(y3-y4)-(y1-y3)*(x3-x4))/den;
      const u=-((x1-x2)*(y1-y3)-(y1-y2)*(x1-x3))/den;
      if(t>=0&&t<=1&&u>=0&&u<=1){
        const ix=x1+t*(x2-x1), iy=y1+t*(y2-y1);
        pwrCollCooldowns.current[coolKey]=now;
        lastPwrCollRef.current=now;
        spawnShockwave(ix,iy,80,'#fde68a');
        spawn(ix,iy,'#fff',15,50,500,[3,8]);
        doShake(0.3);
        // AOE damage at intersection
        setObjects(p=>p.map(o=>{const dd=Math.sqrt((o.x+o.w/2-ix)**2+(o.y+o.h/2-iy)**2);if(dd<80){dealDamage(o.id,20);return{...o,vx:o.vx+((o.x+o.w/2-ix)/(dd||1))*5,vy:o.vy+((o.y+o.h/2-iy)/(dd||1))*5};}return o;}));
        recordDiscovery('power_collision:beam+beam');
      }
    }

    // Beam×Zone
    if((a.type==='beam'&&b.type==='zone')||(a.type==='zone'&&b.type==='beam')){
      const beam=a.type==='beam'?a:b, zone=a.type==='zone'?a:b;
      const zcx=zone.cx,zcy=zone.cy,zr=zone.radius||60;
      // Line-circle intersection
      const dx=beam.x2-beam.x1,dy=beam.y2-beam.y1;
      const fx=beam.x1-zcx,fy=beam.y1-zcy;
      const aa=dx*dx+dy*dy,bb=2*(fx*dx+fy*dy),cc=fx*fx+fy*fy-zr*zr;
      const disc=bb*bb-4*aa*cc;
      if(disc>=0){
        pwrCollCooldowns.current[coolKey]=now;
        // Shield × beam → deflect
        if(zone.element==='kinetic'&&beam.type==='beam'){
          const nx=zcx-beam.x1,ny=zcy-beam.y1,nl=Math.sqrt(nx*nx+ny*ny)||1;
          spawnBeamFx(zcx,zcy,zcx+nx/nl*100,zcy+ny/nl*100,beam.color||'#fff',150);
          spawn(zcx,zcy,'#fff',5,20,200,[1,3],'spark',{trail:[]});
          beam.life=0;
          recordDiscovery('power_collision:deflect');
        }
      }
    }
  }
}
```

}, [spawn,spawnSmoke,spawnRing,spawnFire,spawnPsi,spawnShockwave,spawnBeamFx,doShake,dealDamage,createZone,recordDiscovery]);

const resetSceneRef = useRef(null);
const lastMagPullRef = useRef(0);

// ─── UNDO SYSTEM (v4.5) ───
const handleUndo = useCallback(() => {
const buf = snapshotsRef.current;
if(buf.length < 2) return;
buf.pop(); // remove current
const snap = buf[buf.length-1];
if(snap) {
setObjects(snap.objects.map(o=>({…o,statuses:[…o.statuses],effects:[…o.effects]})));
spawnPostFx(‘tint’,{color:’#ffffff’,life:150});
}
}, [spawnPostFx]);

// Keyboard shortcuts (v4.5)
useEffect(() => {
const handler = (e) => {
if(e.key===‘z’ && (e.ctrlKey||e.metaKey)) { e.preventDefault(); handleUndo(); return; }
if(e.key===‘Escape’) { setPowers([]); setBuildMode(false); if(replayMode) setReplayMode(null); return; }
if(e.key===‘b’||e.key===‘B’) { setBuildMode(b=>!b); setPowers([]); return; }
if(e.key===‘r’||e.key===‘R’) { if(!e.ctrlKey&&!e.metaKey) { resetSceneRef.current?.(); return; } }
// 1-9 power select
const num = parseInt(e.key);
if(num>=1 && num<=9) {
const filtered = catFilter ? POWERS.filter(p=>p.cat===catFilter) : POWERS;
if(num<=filtered.length) {
const pw = filtered[num-1];
setPowers(prev => {
if(prev.some(p=>p===pw.id)) return prev.filter(p=>p!==pw.id);
if(prev.length>=2) return [prev[1],pw.id];
return […prev,pw.id];
});
setBuildMode(false);
if(onboardStep===1) setOnboardStep(2);
}
}
};
window.addEventListener(‘keydown’, handler);
return () => window.removeEventListener(‘keydown’, handler);
}, [handleUndo, catFilter, replayMode, onboardStep]);

// ─── REPLAY SYSTEM (v4.5) ───
const startReplay = useCallback(() => {
const buf = snapshotsRef.current;
if(buf.length < 5) return;
setReplayMode({frames:[…buf],currentFrame:0,speed:0.25,playing:true});
}, []);

// ─── RESIZE + ANIMATION LOOP ───
useEffect(() => {
const el = sceneRef.current; if(!el) return;
const ob = new ResizeObserver(([e]) => setBounds({w:e.contentRect.width,h:e.contentRect.height}));
ob.observe(el); return () => ob.disconnect();
}, []);

useEffect(() => {
let running = true;
let _lastFrameTime = 0, _slowFrames = 0, _fastFrames = 0, _particleBudget = 1.0;
const tick = () => {
const now = Date.now();
// Frame budget: reduce particles if consistently slow
const _dt = now - _lastFrameTime; _lastFrameTime = now;
if(_dt > 20) { _slowFrames++; _fastFrames=0; if(_slowFrames>3) _particleBudget=Math.max(0.25,_particleBudget-0.25); }
else if(_dt < 12) { _fastFrames++; _slowFrames=0; if(_fastFrames>30) _particleBudget=Math.min(1.0,_particleBudget+0.1); }

```
  // Physics (v4.5 — gravity, boundaries, collisions, stacking)
  // Skip physics during replay
  if(!replayModeRef.current) {
  const E = envRef.current;
  let bestCollV=0, bestCollA='', bestCollB='';
  const pendingDisc=[];
  setObjects(prev => {
    const bw = bounds.w, bh = bounds.h;
    if(bw<10) return prev;
    // Step 1: apply gravity + velocity + friction
    let objs = prev.map(o => {
      if(o.hp<=0) return o; // v5.1 — skip dead objects in physics
      if(o.statuses.some(s=>s.type==="frozen")) return {...o, vx:o.vx*0.5, vy:o.vy*0.5};
      let {x,y,vx,vy} = o;
      vy += E.gravity;
      // Weather wind force (v5.0)
      const _wf=WEATHER_TYPES[weatherRef.current]?.force;
      if(_wf){vx+=_wf.x/(o.mass||1)*0.3;vy+=(_wf.y||0)/(o.mass||1)*0.3;}
      // Material transforms (v5.0)
      if(o.mat==="ice"&&o.temp>30&&o.hp>0){o={...o,mat:"water",label:"Melt",hp:MAT_HP.water,maxHp:MAT_HP.water,fragile:false,mass:o.w*o.h*(MAT_PROPS.water?.density||1)*0.01};}
      if(o.mat==="sand"&&o.temp>1700){o={...o,mat:"glass",label:"Glass",fragile:true,hp:MAT_HP.glass,maxHp:MAT_HP.glass,mass:o.w*o.h*(MAT_PROPS.glass?.density||1)*0.01};}
      x += vx; y += vy;
      vx *= E.friction; vy *= E.friction;
      if(Math.abs(vx)<0.08) vx=0;
      if(Math.abs(vy)<0.08) vy=0;
      // Terrain collision (v5.0)
      const _tc=checkTerrainCollision({...o,x,y,vx,vy}); x=_tc.x; y=_tc.y; vx=_tc.vx; vy=_tc.vy;
      // Step 2: boundary clamp
      const _bounce=MAT_PROPS[o.mat]?.bounce||0.3;
      if(E.floor && y+o.h > bh) { y=bh-o.h; vy = vy<-1 ? vy*-_bounce : 0; }
      if(x<0){x=0;vx=Math.abs(vx)*_bounce;}
      if(x+o.w>bw){x=bw-o.w;vx=-Math.abs(vx)*_bounce;}
      if(y<0){y=0;vy=Math.abs(vy)*_bounce;}
      if(!E.floor && y>bh+100) y=-o.h-50; // space wrap
      const rest = (Math.abs(vx)+Math.abs(vy)<0.3) ? (o.resting||0)+1 : 0;
      return {...o, x,y,vx,vy, resting:rest};
    });
    // Rebuild spatial grid
    spatialGrid.rebuild(objs);
    const collChecked = new Set();
    // Step 3: object-to-object collision via spatial grid (O(n·k) instead of O(n²))
    for(let i=0;i<objs.length;i++){
      const a=objs[i]; if(a.resting>30 && Math.abs(a.vx)+Math.abs(a.vy)<0.5) continue;
      const acx=a.x+a.w/2, acy=a.y+a.h/2;
      const nearby=spatialGrid.query(acx,acy,Math.max(a.w,a.h)+80);
      for(let n=0;n<nearby.length;n++){
        const b=nearby[n]; if(b.id<=a.id) continue; // avoid duplicate pairs
        const pairKey=a.id<b.id?a.id*1e6+b.id:b.id*1e6+a.id;
        if(collChecked.has(pairKey)) continue; collChecked.add(pairKey);
        const ox=Math.min(a.x+a.w,b.x+b.w)-Math.max(a.x,b.x);
        const oy=Math.min(a.y+a.h,b.y+b.h)-Math.max(a.y,b.y);
        if(ox<=0||oy<=0) continue;
        const ma=a.mass||1, mb=b.mass||1, totalM=ma+mb;
        if(ox<oy){
          const push=a.x<b.x?1:-1, sep=ox*0.5;
          a.x-=push*sep*(mb/totalM); b.x+=push*sep*(ma/totalM);
          const rv=a.vx-b.vx, impulse=rv*1.4/totalM;
          a.vx-=impulse*mb; b.vx+=impulse*ma;
        } else {
          const push=a.y<b.y?1:-1, sep=oy*0.5;
          a.y-=push*sep*(mb/totalM); b.y+=push*sep*(ma/totalM);
          const rv=a.vy-b.vy, impulse=rv*1.4/totalM;
          a.vy-=impulse*mb; b.vy+=impulse*ma;
        }
        const relV=Math.abs(a.vx-b.vx)+Math.abs(a.vy-b.vy);
        if(relV>bestCollV){bestCollV=relV;bestCollA=a.mat;bestCollB=b.mat;}
        if(relV>5){const ck=a.mat<b.mat?a.mat+'+'+b.mat:b.mat+'+'+a.mat;if(!collDiscRef.current.has(ck)){collDiscRef.current.add(ck);pendingDisc.push('collision:'+ck);}}
      }
    }
    // 5.1: Stacking via spatial grid
    for(let i=0;i<objs.length;i++){
      const a=objs[i]; if(Math.abs(a.vx)<0.3) continue;
      const nearby=spatialGrid.query(a.x+a.w/2,a.y,a.w+20);
      for(let n=0;n<nearby.length;n++){
        const b=nearby[n]; if(b.id===a.id) continue;
        if(Math.abs((b.y+b.h)-a.y)<3 && b.x+b.w>a.x && b.x<a.x+a.w){
          b.vx+=a.vx*0.8;
        }
      }
    }
    // 5.4: Debris auto-despawn (single pass)
    {const _kept=[];
    for(let _di=0;_di<objs.length;_di++){
      const o=objs[_di];
      if(o.label==='Debris'&&o.resting>200){
        const no={...o,opacity:Math.max(0,o.opacity-0.02)};
        if(no.opacity>0.01) _kept.push(no);
      } else { _kept.push(o); }
    }
    objs=_kept;}
    return objs;
  });
  // 5.3: Play material collision sound outside setter
  if(bestCollV>2 && now-lastCollisionAudioRef.current>100){
    lastCollisionAudioRef.current=now;
    playImpact(bestCollV*1.5);
  }
  // Flush collision discoveries
  pendingDisc.forEach(d=>recordDiscovery(d));
  // Collision audio (outside state setter)
  if(now-lastCollisionAudioRef.current>80){
    const objs=objectsRef.current||[];
    for(let i=0;i<objs.length;i++){
      if(Math.abs(objs[i].vx)+Math.abs(objs[i].vy)>3){lastCollisionAudioRef.current=now;playImpact(Math.abs(objs[i].vx)+Math.abs(objs[i].vy));break;}
    }
  }
  } // end replay guard

  // Status ticks (every 500ms) — also skip during replay
  if(!replayModeRef.current)
  if (now - lastStatusTickRef.current > 500) {
    lastStatusTickRef.current = now;
    const deferredStatuses = []; // collect {id, type, ticks} to apply after
    const deferredDiscoveries = [];
    // Rebuild grid for spatial queries in status tick
    spatialGrid.rebuild(objectsRef.current);
    setObjects(prev => {
      let changed = false;
      const next = prev.map(o => {
        if(o.statuses.length === 0) return o;
        changed = true;
        let newHp = o.hp;
        const newStatuses = [];
        for (const s of o.statuses) {
          const tl = s.ticksLeft - 1;
          if (tl <= 0) {
            if (s.type === "charged") {
              const ocx=o.x+o.w/2, ocy=o.y+o.h/2;
              spawnPsi(ocx,ocy,"#d946ef",20,80);
              spawnShockwave(ocx,ocy,80,"#d946ef");
              doShake(0.5);
            }
            continue;
          }
          if (s.type === "burning") {
            newHp = Math.max(0, newHp - 5);
            spawn(o.x+o.w/2+(Math.random()-0.5)*10, o.y+o.h/2-5, "#fb923c", 1, 8, 300, [2,4], "fire");
            // Fire spread via spatial grid (d² comparison)
            const _fsNearby=spatialGrid.query(o.x+o.w/2,o.y+o.h/2,120);
            for(let _fi=0;_fi<_fsNearby.length;_fi++){
              const other=_fsNearby[_fi]; if(other.id===o.id) continue;
              const fl=MAT_PROPS[other.mat]?.flammability||0; if(fl<0.1) continue;
              if(other.statuses.some(ss=>ss.type==="burning")) continue;
              const dd=(other.x-o.x)**2+(other.y-o.y)**2;
              if(dd<14400 && Math.random()<fl*0.4) { deferredStatuses.push({id:other.id,type:"burning",ticks:4}); deferredDiscoveries.push("status:fire+spread"); }
            }
          }
          if (s.type === "corroding") { newHp = Math.max(0, newHp - 3); if(MAT_PROPS[o.mat]?.density>2) newHp = Math.max(0, newHp - 2); }
          if (s.type === "electrified") {
            newHp = Math.max(0, newHp - 8);
            spawn(o.x+o.w/2, o.y+o.h/2, "#38bdf8", 1, 12, 200, [1,3], "spark", {trail:[]});
            const cond = MAT_PROPS[o.mat]?.conductivity||0;
            // Chain lightning via spatial grid (d² comparison)
            const _clR=150*cond, _clR2=_clR*_clR;
            const _clNearby=spatialGrid.query(o.x+o.w/2,o.y+o.h/2,_clR);
            for(let _ci=0;_ci<_clNearby.length;_ci++){
              const other=_clNearby[_ci]; if(other.id===o.id) continue;
              const oCond=MAT_PROPS[other.mat]?.conductivity||0; if(oCond<0.3) continue;
              if(other.statuses.some(ss=>ss.type==="electrified")) continue;
              const dd=(other.x-o.x)**2+(other.y-o.y)**2;
              if(dd<_clR2) { deferredStatuses.push({id:other.id,type:"electrified",ticks:2}); deferredDiscoveries.push("status:electric+chain"); }
            }
          }
          newStatuses.push({...s, ticksLeft:tl});
        }
        // Temperature model
        let temp = o.temp || 20;
        if(newStatuses.some(s=>s.type==="burning")) temp = Math.min(temp+30, 999);
        if(newStatuses.some(s=>s.type==="frozen")) temp = Math.max(temp-40, -200);
        temp += (20 - temp) * 0.05;
        const mp = MAT_PROPS[o.mat];
        if(mp?.meltPoint && temp > mp.meltPoint) newHp = Math.max(0, newHp - 10);
        if(mp?.freezePoint !== null && mp?.freezePoint !== undefined && temp < mp.freezePoint && !newStatuses.some(s=>s.type==="frozen")) {
          newStatuses.push({type:"frozen",ticksLeft:3});
        }
        // 6.2: Material transformations
        const burnTicks = o.statuses.filter(s=>s.type==="burning").reduce((a,s)=>a+s.ticksLeft,0);
        if(o.mat==='metal' && burnTicks>8 && !newStatuses.some(s=>s.type==='molten')){
          newStatuses.push({type:'molten',ticksLeft:6});
          deferredDiscoveries.push('transform:metal+molten');
        }
        if(newStatuses.some(s=>s.type==='molten')){
          {const _mn=spatialGrid.query(o.x+o.w/2,o.y+o.h/2,60);for(let _mi=0;_mi<_mn.length;_mi++){const nb=_mn[_mi];if(nb.id!==o.id){const dd=(nb.x-o.x)**2+(nb.y-o.y)**2;if(dd<3600)deferredStatuses.push({id:nb.id,type:'burning',ticks:2});}}}
        }
        if(newStatuses.some(s=>s.type==='electrified') && o.mat==='glass'){
          newHp=0; deferredDiscoveries.push('transform:glass+shatter');
        }
        if(newStatuses.some(s=>s.type==='corroding') && o.mat==='metal'){
          return {...o, hp:newHp, maxHp:Math.max(o.maxHp*0.1, o.maxHp-o.maxHp*0.05), statuses:newStatuses, temp};
        }
        // 6.4: Zone-material auto-interactions
        const objCol=Math.floor((o.x+o.w/2)/(bounds.w/10||1)), objRow=Math.floor((o.y+o.h/2)/(bounds.h/8||1));
        if(objCol>=0&&objCol<10&&objRow>=0&&objRow<8){
          const zn=zonesRef.current[objRow]?.[objCol];
          if(zn?.type==='fire'&&zn.intensity>0.3&&MAT_PROPS[o.mat]?.flammability>0.5&&!newStatuses.some(s=>s.type==='burning')){
            newStatuses.push({type:'burning',ticksLeft:4}); deferredDiscoveries.push('zone:auto_ignite');
          }
          if(zn?.type==='ice'&&zn.intensity>0.3&&o.mat==='water'&&!newStatuses.some(s=>s.type==='frozen')){
            newStatuses.push({type:'frozen',ticksLeft:5}); deferredDiscoveries.push('zone:auto_freeze');
          }
        }
        return {...o, hp:newHp, statuses:newStatuses, temp};
      });
      // Destroy objects at 0 hp from DoT
      const toDestroy = next.filter(o=>o.hp<=0);
      if(toDestroy.length > 0) {
        const allDebris = [];
        toDestroy.forEach(t => {
          const tcx=t.x+t.w/2, tcy=t.y+t.h/2;
          // Explosive chain detonation (v4.5 — escalating)
          if(t.mat==="explosive") {
            const chainIdx=(t._chainIdx||0)+1;
            const blastR=200+chainIdx*40, blastDmg=40+chainIdx*10;
            const shockR=t.w*1.2+chainIdx*20;
            // Explosive via spatial grid
            const _exNearby=spatialGrid.query(tcx,tcy,blastR);
            for(let _ei=0;_ei<_exNearby.length;_ei++){
              const nb=_exNearby[_ei]; if(nb.id===t.id||nb.hp<=0) continue;
              const dd=(nb.x+nb.w/2-tcx)**2+(nb.y+nb.h/2-tcy)**2;
              if(dd<blastR*blastR) { const d=Math.sqrt(dd)||1; nb.hp=Math.max(0,nb.hp-blastDmg); nb.vx+=(nb.x-tcx)/d*12; nb.vy+=(nb.y-tcy)/d*12; if(nb.mat==='explosive') nb._chainIdx=chainIdx; }
            }
            spawnShockwave(tcx,tcy,shockR,"#ef4444");
            deferredDiscoveries.push("chain:explosive");
          }
          const matColor=MAT_COLORS[t.mat]||"#fff";
          const arr=particlesRef.current, tnow=Date.now();
          for(let s=0;s<6;s++){const a=(Math.PI*2*s)/6+(Math.random()-0.5)*0.4,d=20+Math.random()*30;arr.push({x:tcx,y:tcy,dx:Math.cos(a)*d,dy:Math.sin(a)*d,color:matColor,life:400,size:2+Math.random()*3,born:tnow,type:"shard",extra:{sides:3,spin:(Math.random()-0.5)*4}});}
          spawnShockwave(tcx,tcy,t.w*1.2,matColor);
          spawnGroundFx(tcx,tcy,matColor,t.w*0.6,4000);
          deferredDiscoveries.push(`destroy:${t.mat}`);
          for(let i=0;i<2;i++){
            const a=(Math.PI*2*i)/2+(Math.random()-0.5)*0.5;
            allDebris.push({id:Date.now()+i+Math.random(),x:tcx+Math.cos(a)*20-8,y:tcy+Math.sin(a)*20-8,w:Math.max(12,t.w*0.3),h:Math.max(12,t.h*0.3),mat:t.mat,label:"Debris",shape:"rect",origX:tcx,origY:tcy,origMat:t.mat,scale:0.7,rotation:Math.random()*360,opacity:0.7,effects:[],statuses:[],vx:Math.cos(a)*5,vy:Math.sin(a)*5-2,hp:t.maxHp*0.15,maxHp:t.maxHp*0.15,fragile:true,temp:20,mass:Math.max(12,t.w*0.3)*Math.max(12,t.h*0.3)*(MAT_PROPS[t.mat]?.density||1)*0.01*0.3,resting:0});
          }
        });
        const destroyIds = new Set(toDestroy.map(t=>t.id));
        return [...next.filter(o=>!destroyIds.has(o.id)), ...allDebris];
      }
      return changed ? next : prev;
    });
    // Apply deferred spread/chain statuses
    deferredStatuses.forEach(ds => applyStatus(ds.id, ds.type, ds.ticks));
    deferredDiscoveries.forEach(dk => recordDiscovery(dk));
  }

  // Power-on-power collision tick
  if(!replayModeRef.current) tickPowerInteractions(now);

  // Zone decay (skip during replay)
  if(!replayModeRef.current) {
  const zones = zonesRef.current;
  // v5.1 — Sparse zone iteration (only active cells)
  const _activeZones = activeZonesRef.current;
  for (const key of _activeZones) {
    const r = (key / ZONE_COLS) | 0, c = key % ZONE_COLS;
    const z=zones[r][c]; if(!z) { _activeZones.delete(key); continue; }
    z.intensity -= z.intensity > 0 ? (ZONE_TYPES[z.type]?.decay || 0.002) : 0;
    if(z.intensity <= 0.01) { zones[r][c] = null; _activeZones.delete(key); }
    // Electric spreads to adjacent water zones
    if(z.type==="electric") {
      for(const [dr,dc] of [[0,1],[0,-1],[1,0],[-1,0]]) {
        const nr=r+dr, nc=c+dc;
        if(nr>=0&&nr<ZONE_ROWS&&nc>=0&&nc<ZONE_COLS) {
          const adj=zones[nr][nc];
          if(adj&&adj.type==="ice"&&!zones[nr][nc]?.elecSpread) { zones[nr][nc]={type:"electric",intensity:z.intensity*0.5,source:"chain"}; _activeZones.add(nr*ZONE_COLS+nc); }
        }
      }
    }
  }

  // Zone → object effects
  if(bounds.w > 0) {
    const cw=bounds.w/ZONE_COLS, ch=bounds.h/ZONE_ROWS;
    setObjects(prev => prev.map(o => {
      const col=Math.floor((o.x+o.w/2)/cw), row=Math.floor((o.y+o.h/2)/ch);
      if(col<0||col>=ZONE_COLS||row<0||row>=ZONE_ROWS) return o;
      const z=zones[row][col]; if(!z||z.intensity<0.15) return o;
      const zt=ZONE_TYPES[z.type];
      if(zt.statusType && !o.statuses.some(s=>s.type===zt.statusType)) {
        if(z.type==="fire"&&MAT_RULES.burns[o.mat]) applyStatus(o.id,"burning",3);
        else if(z.type==="ice") { return {...o, vx:o.vx*0.5, vy:o.vy*0.5}; }
        else if(z.type==="electric"&&MAT_RULES.conducts[o.mat]) applyStatus(o.id,"electrified",2);
        else if(z.type==="toxic") applyStatus(o.id,"corroding",2);
      }
      if(z.type==="void") return {...o, scale:Math.max(0.3, o.scale-0.002*z.intensity)};
      if(z.type==="magnetic"&&o.mat==="metal") {
        const zcx=col*cw+cw/2, zcy=row*ch+ch/2;
        const dx=zcx-(o.x+o.w/2), dy=zcy-(o.y+o.h/2), dist=Math.sqrt(dx*dx+dy*dy)||1;
        return {...o, vx:o.vx+(dx/dist)*z.intensity*4, vy:o.vy+(dy/dist)*z.intensity*4};
      }
      if(z.type==="gravity") {
        const zcx=col*cw+cw/2, zcy=row*ch+ch/2;
        const dx=zcx-(o.x+o.w/2), dy=zcy-(o.y+o.h/2), dist=Math.sqrt(dx*dx+dy*dy)||1;
        return {...o, vx:o.vx+(dx/dist)*z.intensity*3, vy:o.vy+(dy/dist)*z.intensity*3};
      }
      if(z.type==="time"&&z.intensity>0.3) {
        return {...o, vx:o.vx*0.4, vy:o.vy*0.4}; // slow field
      }
      if(z.type==="magnetic"&&o.mat!=="metal"&&z.intensity>0.4) {
        // Non-metal diamagnetic repulsion
        const zcx=col*cw+cw/2, zcy=row*ch+ch/2;
        const dx=(o.x+o.w/2)-zcx, dy=(o.y+o.h/2)-zcy, dist=Math.sqrt(dx*dx+dy*dy)||1;
        return {...o, vx:o.vx+(dx/dist)*z.intensity*0.5, vy:o.vy+(dy/dist)*z.intensity*0.5};
      }
      return o;
    }));
  }

  } // end zone decay replay guard

  // v5.1 — flush damage queue (batched)
  flushDamageQueue();

  // v5.0 system ticks
  if(!replayModeRef.current) {
    tickWeather(now, bounds.w, bounds.h);
    tickMeteors(now, bounds.w, bounds.h);
    tickJoints();
    if(now % 3 < 1) tickBehaviors(now); // ~20Hz
    if(scenarioRef.current && !scenarioRef.current._done && now % 7 < 1) checkScenarioGoal();
  }

  // Shake decay
  if(shakeRef.current>0){ shakeRef.current=Math.max(0,shakeRef.current-0.05); setShake(shakeRef.current); }

  // Phoenix level decay
  if(now-phoenixRef.current.lastUsed>30000 && phoenixRef.current.level>1) phoenixRef.current.level=1;

  // Ambient zone particle emission (Item 5)
  if(now - lastAmbientRef.current > 200){
    lastAmbientRef.current = now;
    const zones=zonesRef.current, arr=particlesRef.current;
    for(let r=0;r<ZONE_ROWS;r++) for(let c=0;c<ZONE_COLS;c++){
      const z=zones[r][c]; if(!z||z.intensity<0.2) continue;
      const zx=c*(bounds.w/ZONE_COLS)+Math.random()*(bounds.w/ZONE_COLS), zy=r*(bounds.h/ZONE_ROWS)+Math.random()*(bounds.h/ZONE_ROWS);
      if(z.type==="fire"&&Math.random()<0.15) arr.push({x:zx,y:zy,dx:(Math.random()-0.5)*2,dy:-0.5-Math.random(),color:"#f97316",life:2500,size:1.5,born:now,type:"ember",extra:null});
      if(z.type==="ice"&&Math.random()<0.1) arr.push({x:zx,y:zy,dx:(Math.random()-0.5)*0.5,dy:(Math.random()-0.5)*0.5,color:"#a5f3fc",life:1500,size:2,born:now,type:"ice",extra:{sides:5,spin:1}});
      if(z.type==="electric"&&Math.random()<0.12) arr.push({x:zx,y:zy,dx:(Math.random()-0.5)*15,dy:(Math.random()-0.5)*15,color:"#93c5fd",life:200,size:1.5,born:now,type:"spark",extra:{trail:[]}});
      if(z.type==="toxic"&&Math.random()<0.1) arr.push({x:zx,y:zy,dx:(Math.random()-0.5)*2,dy:-0.8,color:"#a855f7",life:1500,size:6,born:now,type:"smoke",extra:null});
      if(z.type==="gravity"&&Math.random()<0.12) arr.push({x:zx+Math.random()*cw,y:zy+Math.random()*ch,dx:(Math.random()-0.5)*3,dy:(Math.random()-0.5)*3,color:"#6366f1",life:800,size:1.5,born:now,type:"circle",extra:null});
      if(z.type==="time"&&Math.random()<0.08) arr.push({x:zx+Math.random()*cw,y:zy+Math.random()*ch,dx:0,dy:-0.2,color:"#fbbf24",life:1200,size:2,born:now,type:"circle",extra:null});
      if(z.type==="magnetic"&&Math.random()<0.1) arr.push({x:zx+Math.random()*cw,y:zy+Math.random()*ch,dx:(Math.random()-0.5)*4,dy:(Math.random()-0.5)*4,color:"#c084fc",life:600,size:1,born:now,type:"spark",extra:{trail:[]}});
    }
    // Debris smoke trails
    const objs=objectsRef.current;
    if(objs) objs.forEach(o=>{ if(o.trailUntil&&o.trailUntil>now) arr.push({x:o.x+o.w/2+(Math.random()-0.5)*6,y:o.y+o.h/2,dx:(Math.random()-0.5)*2,dy:-1,color:"#9ca3af",life:600,size:4,born:now,type:"smoke",extra:null}); });
    // Ambient dust motes
    if(Math.random()<0.3*_particleBudget) arr.push({x:Math.random()*bounds.w,y:Math.random()*bounds.h,dx:(Math.random()-0.5)*0.5,dy:(Math.random()-0.5)*0.3,color:"#d1d5db",life:8000,size:1,born:now,type:"circle",extra:null});
  }

  // Undo snapshot capture (every 100ms) — skip during replay
  if(!replayModeRef.current && now - lastSnapshotRef.current > 100) {
    lastSnapshotRef.current = now;
    const curObjs = objectsRef.current;
    if(curObjs && curObjs.length > 0) {
      const snap = curObjs.map(o=>({id:o.id,x:o.x,y:o.y,w:o.w,h:o.h,mat:o.mat,hp:o.hp,maxHp:o.maxHp,
        statuses:[...o.statuses],vx:o.vx,vy:o.vy,temp:o.temp||20,scale:o.scale,rotation:o.rotation,
        opacity:o.opacity,label:o.label,shape:o.shape,origX:o.origX,origY:o.origY,origMat:o.origMat,
        effects:[...o.effects],mass:o.mass,resting:o.resting||0,fragile:o.fragile}));
      const buf = snapshotsRef.current;
      buf.push({objects:snap,time:now});
      if(buf.length > snapshotMaxRef.current) buf.shift();
    }
  }

  // Canvas draw
  const canvas=canvasRef.current;
  if(canvas){
    const ctx=canvas.getContext("2d");
    if(canvas.width!==bounds.w||canvas.height!==bounds.h){ canvas.width=bounds.w; canvas.height=bounds.h; }
    drawCanvas(ctx,bounds.w,bounds.h,now,powersRef.current,objectsRef.current);
  }

  // Replay tick
  if(replayModeRef.current) {
    const rm = replayModeRef.current;
    if(rm.playing) {
      const newFrame = Math.min(rm.frames.length-1, Math.floor(rm.currentFrame + rm.speed));
      if(newFrame !== rm.currentFrame) {
        const snap = rm.frames[newFrame];
        if(snap) setObjects(snap.objects.map(o=>({...o,statuses:[...o.statuses],effects:[...o.effects]})));
        replayModeRef.current = {...rm, currentFrame: newFrame};
        if(newFrame >= rm.frames.length-1) replayModeRef.current.playing = false;
      }
    }
  }

  // Force periodic re-render for HP bars, status visuals
  setTick(t=>t+1);

  if(running) animRef.current = requestAnimationFrame(tick);
};
animRef.current = requestAnimationFrame(tick);
return () => { running=false; cancelAnimationFrame(animRef.current); if(toastFlushTimerRef.current){clearTimeout(toastFlushTimerRef.current);toastFlushTimerRef.current=null;} };
```

}, [bounds, drawCanvas, spawn, spawnPsi, spawnGroundFx, spawnShockwave, doShake, applyStatus, recordDiscovery, tickPowerInteractions, tickWeather, tickMeteors, tickJoints, tickBehaviors, checkScenarioGoal, checkTerrainCollision, flushDamageQueue]);

// ─── SCENE HANDLERS ───
const handleDown = useCallback((e) => {
e.preventDefault();
initAudio();
if(replayModeRef.current) return; // Block input during replay
if(onboardStep===0) setOnboardStep(1);
const pos=getPos(e);
pointerRef.current=pos;
holdStartRef.current=Date.now();
const obj=findObj(pos.x,pos.y);

```
// Build mode: place new objects
if(buildMode && !obj) {
  const maxObjs = 50;
  setObjects(prev => {
    if(prev.length >= maxObjs) return prev;
    const id = Date.now()+Math.random();
    const labels = {metal:"Block",wood:"Plank",glass:"Pane",stone:"Rock",water:"Drop",organic:"Blob",energy:"Orb",explosive:"Bomb"};
    const w=45,h=45;
    return [...prev, {id,x:pos.x-w/2,y:pos.y-h/2,w,h,mat:buildMat,label:labels[buildMat]||"Block",shape:buildShape,
      origX:pos.x-w/2,origY:pos.y-h/2,origMat:buildMat,scale:1,rotation:0,opacity:1,effects:[],statuses:[],
      vx:0,vy:0,hp:MAT_HP[buildMat],maxHp:MAT_HP[buildMat],fragile:!!MAT_FRAGILE[buildMat],
      temp:20,mass:w*h*(MAT_PROPS[buildMat]?.density||1)*0.01,resting:0}];
  });
  sfxUI(600);
  return;
}
if(buildMode && obj) {
  // Delete in build mode
  setObjects(prev => prev.filter(o => o.id !== obj.id));
  playDestroy(obj.mat);
  return;
}

if(powers.length===0){
  if(obj){setDragging(obj.id);setDragOff({x:pos.x-obj.x,y:pos.y-obj.y});sfxPickup();}
  return;
}
if(activeCombo){applyCombo(activeCombo,obj,pos);return;}

const pw=activePower; if(!pw) return;

if(pw.mode==="drag"){
  if(obj&&(!pw.material||obj.mat===pw.material)){
    setDragging(obj.id);setDragOff({x:pos.x-obj.x,y:pos.y-obj.y});sfxPickup();
    spawn(obj.x+obj.w/2,obj.y+obj.h/2,pw.color,6,20,400,[2,4]);
  } else if(obj && pw.material && obj.mat!==pw.material && (pw.id==="magnetism"||pw.id==="magnetism_g")){
    // Magnetism on non-metal: diamagnetic repulsion shove
    const mCol=pw.id==="magnetism"?"#a855f7":"#22c55e";
    const dx=(obj.x+obj.w/2)-pos.x, dy=(obj.y+obj.h/2)-pos.y, dist=Math.sqrt(dx*dx+dy*dy)||1;
    setObjects(p=>p.map(o=>o.id===obj.id?{...o,vx:o.vx+(dx/dist)*8,vy:o.vy+(dy/dist)*8}:o));
    spawn(obj.x+obj.w/2,obj.y+obj.h/2,mCol,6,15,300,[2,4]);
    spawnShockwave(obj.x+obj.w/2,obj.y+obj.h/2,40,mCol);
    playPowerSound(pw.id);
  } else if(!obj && (pw.id==="magnetism"||pw.id==="magnetism_g")){
    // Magnetism click-on-empty: pulse + zone + metal pull
    const mCol=pw.id==="magnetism"?"#a855f7":"#22c55e";
    spawnRing(pos.x,pos.y,mCol,100,true);
    spawn(pos.x,pos.y,mCol,10,40,500,[2,5]);
    createZone(pos.x,pos.y,"magnetic",2,0.8);
    spawnShockwave(pos.x,pos.y,80,mCol);
    doShake(0.15);
    playPowerSound(pw.id);
    setObjects(p=>p.map(o=>{
      const dx=pos.x-(o.x+o.w/2), dy=pos.y-(o.y+o.h/2), dist=Math.sqrt(dx*dx+dy*dy)||1;
      if(o.mat==="metal" && dist<300){
        const force=8*(1-dist/300);
        return {...o, vx:o.vx+(dx/dist)*force, vy:o.vy+(dy/dist)*force};
      }
      // Non-metal slight repulsion
      if(o.mat!=="metal" && dist<150){
        return {...o, vx:o.vx-(dx/dist)*1.5, vy:o.vy-(dy/dist)*1.5};
      }
      return o;
    }));
    recordDiscovery('p:'+pw.id);
  }
  return;
}

if(pw.mode==="two_click"){
  if(!twoClickSrc){
    if(obj){setTwoClickSrc({objId:obj.id,pos});spawn(pos.x,pos.y,pw.color,6,15,500,[2,5]);}
  } else {
    if(pw.id==="teleport"){
      const src=objects.find(o=>o.id===twoClickSrc.objId);
      if(src){
        spawn(src.x+src.w/2,src.y+src.h/2,"#6366f1",12,35,500,[3,7]);
        spawn(pos.x,pos.y,"#6366f1",12,35,500,[3,7]);
        // Nightcrawler bamf smoke at departure
        createZone(src.x+src.w/2,src.y+src.h/2,"toxic",1,0.7);
        spawnGroundFx(src.x+src.w/2,src.y+src.h/2,"#6366f1",30,4000);
        setObjects(p=>p.map(o=>o.id===twoClickSrc.objId?{...o,x:pos.x-o.w/2,y:pos.y-o.h/2,vx:0,vy:0}:o));
        // Brief invulnerability
        applyStatus(twoClickSrc.objId,"phased",2);
      }
    } else if(pw.id==="portal"){
      const src=objects.find(o=>o.id===twoClickSrc.objId);
      if(src){spawn(src.x+src.w/2,src.y+src.h/2,"#fde68a",10,25,600,[3,6]);spawn(pos.x,pos.y,"#818cf8",10,25,600,[3,6]);setObjects(p=>p.map(o=>o.id===twoClickSrc.objId?{...o,x:pos.x-o.w/2,y:pos.y-o.h/2}:o));}
    } else if(pw.id==="bifrost"){
      const src=objects.find(o=>o.id===twoClickSrc.objId);
      if(src){
        // Rainbow bridge beam between points
        spawnBeamFx(src.x+src.w/2,src.y+src.h/2,pos.x,pos.y,"#c084fc",6,"rainbow");
        spawn(src.x+src.w/2,src.y+src.h/2,"#c084fc",10,30,500,[3,7]);
        spawn(pos.x,pos.y,"#fde68a",10,30,500,[3,7]);
        spawnShockwave(pos.x,pos.y,60,"#c084fc");
        setObjects(p=>p.map(o=>o.id===twoClickSrc.objId?{...o,x:pos.x-o.w/2,y:pos.y-o.h/2,vx:0,vy:0}:o));
        dealDamage(twoClickSrc.objId,pw.dmg);
      }
    } else if(pw.id==="boom_tube"){
      const src=objects.find(o=>o.id===twoClickSrc.objId);
      if(src){
        spawnRing(src.x+src.w/2,src.y+src.h/2,"#f59e0b",50,true);
        spawnRing(pos.x,pos.y,"#f59e0b",50,true);
        spawn(src.x+src.w/2,src.y+src.h/2,"#fbbf24",8,25,400,[3,6]);
        spawn(pos.x,pos.y,"#fbbf24",8,25,400,[3,6]);
        doShake(0.2);
        setObjects(p=>p.map(o=>o.id===twoClickSrc.objId?{...o,x:pos.x-o.w/2,y:pos.y-o.h/2,vx:0,vy:0}:o));
        dealDamage(twoClickSrc.objId,pw.dmg);
      }
    }
    setTwoClickSrc(null);
  }
  return;
}

if(pw.mode==="hold"){
  const objId=obj?.id??null;

  // Gambit charge-timing
  if(pw.id==="kinetic_charge"){
    if(obj){
      addEffect(obj.id,{type:"charge",dur:3000});
      spawn(obj.x+obj.w/2,obj.y+obj.h/2,pw.color,4,10,300,[2,3]);
      const chargeId=obj.id;
      const chargeStart=Date.now();
      const iv=setInterval(()=>{
        const elapsed=Date.now()-chargeStart;
        const fp=pointerRef.current;
        spawn(fp.x+(Math.random()-0.5)*20,fp.y+(Math.random()-0.5)*20,"#d946ef",1,5,200,[2,3]);
        if(elapsed>2500){
          // Fizzle
          clearInterval(iv); setHoldIv(null);
          spawn(fp.x,fp.y,"#666",8,20,300,[2,4]);
        }
      },80);
      setHoldIv(iv);
    }
    return;
  }

  // Storm weather build
  if(pw.id==="weather"){
    stormStageRef.current=0;
    const stormStart=Date.now();
    const run=()=>{
      const fp=pointerRef.current;
      const elapsed=Date.now()-stormStart;
      const stage = elapsed<1000?1:elapsed<2000?2:3;
      stormStageRef.current=stage;
      spawn(fp.x+(Math.random()-0.5)*60,fp.y-20,"#93c5fd",1,15,300,[1,3]);
      if(stage>=2) spawn(fp.x+(Math.random()-0.5)*100,fp.y-10,"#a5f3fc",1,20,400,[1,2]);
    };
    run();
    const iv=setInterval(run,80);
    setHoldIv(iv);
    return;
  }

  // Captain Marvel binary mode — sustained damage aura
  if(pw.id==="binary_mode"){
    const run=()=>{
      const fp=pointerRef.current;
      spawnFire(fp.x+(Math.random()-0.5)*40,fp.y+(Math.random()-0.5)*40,2,20,400);
      spawn(fp.x+(Math.random()-0.5)*30,fp.y+(Math.random()-0.5)*30,"#fbbf24",1,10,300,[2,4]);
      setObjects(p=>p.map(o=>{const dist=Math.sqrt((o.x+o.w/2-fp.x)**2+(o.y+o.h/2-fp.y)**2);if(dist<80){dealDamage(o.id,pw.dmg*0.15);return{...o,vx:o.vx+(Math.random()-0.5)*2};}return o;}));
    };
    run();
    const iv=setInterval(run,100);
    setHoldIv(iv);
    return;
  }

  const run=()=>{
    const fp=pointerRef.current;
    if(pw.id==="mind_control"&&objId!==null){
      setObjects(p=>p.map(o=>{if(o.id!==objId)return o;const dx=fp.x-(o.x+o.w/2),dy=fp.y-(o.y+o.h/2);return{...o,x:o.x+dx*0.1,y:o.y+dy*0.1};}));
      spawn(fp.x,fp.y,pw.color,1,15,400,[2,3]);
    } else if(pw.id==="time_slow"){
      spawn(fp.x,fp.y,pw.color,2,40,500,[3,5]);
      setObjects(p=>p.map(o=>({...o,vx:o.vx*0.8,vy:o.vy*0.8})));
    } else if(pw.id==="gravity"){
      setObjects(p=>p.map(o=>{const dx=fp.x-(o.x+o.w/2),dy=fp.y-(o.y+o.h/2),dist=Math.sqrt(dx*dx+dy*dy)||1;if(dist<200)return{...o,vx:o.vx+(dx/dist)*3,vy:o.vy+(dy/dist)*3};return o;}));
      spawn(fp.x,fp.y,"#dc2626",2,15,300,[2,4]);
    }
  };
  run();
  const iv=setInterval(run,80);
  setHoldIv(iv);
  return;
}

applyPower(pw.id,obj,pos);
```

}, [powers,activePower,activeCombo,findObj,getPos,twoClickSrc,objects,spawn,applyPower,applyCombo,addEffect,applyStatus,createZone,spawnGroundFx,spawnRing,spawnShockwave,doShake,playPowerSound,recordDiscovery,buildMode,buildMat,buildShape,onboardStep,sfxUI,sfxPickup,playDestroy,initAudio]);

const handleMove = useCallback((e) => {
const pos=getPos(e);
pointerRef.current=pos;

```
// Iceman frost trail on mouse move (not just drag)
const pw=activePower;
if(pw?.id==="cryokinesis"&&!dragging){
  createZone(pos.x,pos.y,"ice",0,0.3);
  if(Math.random()<0.3) spawn(pos.x,pos.y,"#67e8f9",1,3,150,[1,2]);
}
// Magneto passive pull on metal objects toward cursor (throttled 50ms)
if((pw?.id==="magnetism"||pw?.id==="magnetism_g")&&!dragging){
  const _now=Date.now();
  if(_now - lastMagPullRef.current > 50){
    lastMagPullRef.current = _now;
    const mCol=pw?.id==="magnetism"?"#a855f7":"#22c55e";
    // Store cursor for rAF to use
    pointerRef.current = pos;
    setObjects(p=>p.map(o=>{
      if(o.mat!=="metal")return o;
      const dx=pos.x-(o.x+o.w/2),dy=pos.y-(o.y+o.h/2),dist=Math.sqrt(dx*dx+dy*dy)||1;
      if(dist<300){
        const pull=1.8*(1-dist/300);
        if(Math.random()<0.08) spawn(o.x+o.w/2,o.y+o.h/2,mCol,1,5,200,[1,2]);
        return{...o,vx:o.vx+(dx/dist)*pull,vy:o.vy+(dy/dist)*pull};
      }
      return o;
    }));
  }
}

if(dragging===null) return;
e.preventDefault();
setObjects(p=>p.map(o=>{
  if(o.id!==dragging)return o;
  let nx=pos.x-dragOff.x, ny=pos.y-dragOff.y;
  if(pw?.id==="super_strength"){nx+=(Math.random()-0.5)*6;ny+=(Math.random()-0.5)*6;}
  return{...o,x:nx,y:ny,vx:0,vy:0};
}));
if(pw?.id==="super_speed"){
  const dObj=objects.find(o=>o.id===dragging);
  if(dObj) spawn(dObj.x+dObj.w/2,dObj.y+dObj.h/2,"#e5e7eb",2,8,250,[2,3]);
}
// Magneto: other metals drift toward dragged metal (throttled)
if((pw?.id==="magnetism"||pw?.id==="magnetism_g")&&dragging){
  const _now=Date.now();
  if(_now - lastMagPullRef.current > 50){
    lastMagPullRef.current = _now;
    const mCol=pw?.id==="magnetism"?"#a855f7":"#22c55e";
    setObjects(p=>{
      const src=p.find(o=>o.id===dragging);
      if(!src)return p;
      const scx=src.x+src.w/2, scy=src.y+src.h/2;
      return p.map(o=>{
        if(o.id===dragging||o.mat!=="metal")return o;
        const dx=scx-(o.x+o.w/2),dy=scy-(o.y+o.h/2),dist=Math.sqrt(dx*dx+dy*dy)||1;
        if(dist<250){
          const pull=3.0*(1-dist/250);
          if(Math.random()<0.05) spawn(o.x+o.w/2,o.y+o.h/2,mCol,1,4,150,[1,2]);
          return{...o,vx:o.vx+(dx/dist)*pull,vy:o.vy+(dy/dist)*pull};
        }
        return o;
      });
    });
  }
}
```

}, [dragging,dragOff,activePower,objects,getPos,spawn,createZone]);

const handleUp = useCallback(() => {
const pw=activePower;

```
// Gambit release — detonate based on charge time
if(pw?.id==="kinetic_charge"&&holdIv){
  clearInterval(holdIv);setHoldIv(null);
  const elapsed=Date.now()-holdStartRef.current;
  if(elapsed<2500){
    const fp=pointerRef.current;
    const power=Math.min(1,elapsed/2000);
    const radius=60+power*120;
    const dmgMult=0.5+power*1.5;
    spawnFire(fp.x,fp.y,Math.floor(15+power*20),radius,700);
    spawnShockwave(fp.x,fp.y,radius,"#d946ef");
    doShake(0.3+power*0.5);
    createZone(fp.x,fp.y,"fire",Math.floor(power*2),power*0.7);
    setObjects(p=>p.map(o=>{
      const dx=(o.x+o.w/2)-fp.x,dy=(o.y+o.h/2)-fp.y,dist=Math.sqrt(dx*dx+dy*dy)||1;
      if(dist<radius){dealDamage(o.id,Math.floor(30*dmgMult));return{...o,vx:o.vx+(dx/dist)*Math.min(15*power,2000/dist),vy:o.vy+(dy/dist)*Math.min(15*power,2000/dist)};}
      return o;
    }));
  }
  return;
}

// Storm release — strike based on stage
if(pw?.id==="weather"&&holdIv){
  clearInterval(holdIv);setHoldIv(null);
  const fp=pointerRef.current;
  const stage=stormStageRef.current||1;

  if(stage>=1){
    // Find nearest object for bolt
    let nearObj=null, nearDist=Infinity;
    objects.forEach(o=>{const d=Math.sqrt((o.x+o.w/2-fp.x)**2+(o.y+o.h/2-fp.y)**2);if(d<200&&d<nearDist){nearDist=d;nearObj=o;}});
    if(nearObj){
      const ncx=nearObj.x+nearObj.w/2,ncy=nearObj.y+nearObj.h/2;
      spawnLightning(ncx+(Math.random()-0.5)*40,0,ncx,ncy,"#93c5fd");
      spawnBeamFx(ncx+(Math.random()-0.5)*40,0,ncx,ncy,"#93c5fd",5,"lightning");
      addEffect(nearObj.id,{type:"shock",dur:800});
      dealDamage(nearObj.id,30);
      applyStatus(nearObj.id,"electrified",3);
      doShake(0.4);
      createZone(ncx,ncy,"electric",1,0.7);
      // Conduction chain
      if(MAT_RULES.conducts[nearObj.mat]){
        objects.forEach(o=>{if(o.id!==nearObj.id&&MAT_RULES.conducts[o.mat]){const d=Math.sqrt((o.x-nearObj.x)**2+(o.y-nearObj.y)**2);if(d<200){applyStatus(o.id,"electrified",2);addEffect(o.id,{type:"shock",dur:600});recordDiscovery("status:electric+chain");}}});
      }
    }
  }
  if(stage>=2){
    // Multiple bolts
    for(let i=0;i<2;i++){
      setTimeout(()=>{
        const rx=fp.x+(Math.random()-0.5)*150, ry=fp.y+(Math.random()-0.5)*100;
        const tgt=objects.find(o=>{const d=Math.sqrt((o.x+o.w/2-rx)**2+(o.y+o.h/2-ry)**2);return d<100;});
        if(tgt){spawnLightning(tgt.x+tgt.w/2,0,tgt.x+tgt.w/2,tgt.y+tgt.h/2,"#a5f3fc");dealDamage(tgt.id,20);applyStatus(tgt.id,"electrified",2);}
        else spawnLightning(rx,0,rx,bounds.h,"#93c5fd");
      },200+i*200);
    }
  }
  if(stage>=3){
    // Full storm — wind + rain + electric zones
    doShake(0.7);
    createZone(fp.x,fp.y,"electric",3,0.9);
    setObjects(p=>p.map(o=>({...o,vx:o.vx+(Math.random()-0.5)*10})));
    spawn(fp.x,fp.y,"#93c5fd",30,150,1000,[1,3],"spark",{trail:[]}); spawnPostFx("tint",{color:"#1e3a5f",life:500});
  }
  stormStageRef.current=0;
  return;
}

if(dragging!==null){
  sfxDrop();
  if(pw?.id==="super_strength") setObjects(p=>p.map(o=>o.id===dragging?{...o,vx:(Math.random()-0.5)*30,vy:-15-Math.random()*20}:o));
  setDragging(null);
}
if(holdIv){clearInterval(holdIv);setHoldIv(null);}
```

}, [dragging,activePower,holdIv,objects,bounds,spawn,spawnFire,spawnLightning,spawnBeamFx,spawnShockwave,spawnPostFx,doShake,dealDamage,applyStatus,addEffect,createZone,recordDiscovery,sfxDrop]);

const resetScene = useCallback(() => {
if(!mutedRef.current) sfxReset();
setObjects(createObjects());
particlesRef.current=[];beamsRef.current=[];groundFxRef.current=[];impactsRef.current=[];postFxRef.current=[];floatingDmgRef.current=[];damageQueueRef.current=[];toastQueueRef.current=[];if(toastFlushTimerRef.current){clearTimeout(toastFlushTimerRef.current);toastFlushTimerRef.current=null;}
zonesRef.current=createZoneGrid(); activeZonesRef.current.clear();
activeEffectsRef.current=[];
snapshotsRef.current=[];
collDiscRef.current=new Set();
lastPwrCollRef.current=0;
pwrCollCooldowns.current={};
setTwoClickSrc(null);setShowInfo(null);setComboFlash(null);setReplayMode(null);
phoenixRef.current={level:1,lastUsed:0};
if(holdIvRef.current){clearInterval(holdIvRef.current);setHoldIv(null);}
// v5.0 reset
terrainRef.current=[];
jointsRef.current=[];
weatherParticlesRef.current=[];
setScenario(null);scenarioRef.current=null;setScenarioResult(null);
}, [sfxReset]);
useEffect(() => { resetSceneRef.current = resetScene; }, [resetScene]);  // ─── TUTORIAL COMPLETION (v4.5) ───
// 9.1: Combo hints — find partner for equipped power
const comboHint = useMemo(() => {
if(powers.length !== 1) return null;
const pid = powers[0];
for(const c of COMBOS) {
const partner = c.a===pid ? c.b : c.b===pid ? c.a : null;
if(!partner) continue;
const key = c.a+’_’+c.b;
if(discovered.has(‘combo:’+c.effect) || hintedCombosRef.current.has(key)) continue;
return { partnerId: partner, comboName: c.name, partnerPower: POWERS.find(p=>p.id===partner), _key: key };
}
return null;
}, [powers, discovered]);

// Track hinted combos (side effect moved out of useMemo)
useEffect(() => {
if(comboHint?._key) hintedCombosRef.current.add(comboHint._key);
}, [comboHint]);

// 9.2: Category nudges + 9.3: Contextual tips
useEffect(() => {
const now = Date.now();
const elapsed = (now - sessionStartRef.current) / 1000;
// Track categories used
powers.forEach(pid => {
const pw = POWERS.find(p=>p.id===pid);
if(pw) usedCatsRef.current.add(pw.cat);
});
// Category nudge at 45s
if(elapsed > 45 && usedCatsRef.current.size < 3 && !shownTipsRef.current.has(‘cat_nudge’)) {
shownTipsRef.current.add(‘cat_nudge’);
const cats = [‘Energy’,‘Elemental’,‘Kinetic’,‘Physical’,‘Psychic’,‘Spatial’,‘Mystic’,‘Cosmic’,‘Tech’,‘Dark’];
const unused = cats.filter(c => !usedCatsRef.current.has(c));
if(unused.length > 0) {
const pick = unused[Math.floor(Math.random()*unused.length)];
setToasts(t=>[…t,{id:Date.now(),text:‘💡 Try a ‘+pick+’ power!’,born:Date.now()}]);
}
}
// Combo nudge at 90s
if(elapsed > 90 && discovered.size === 0 && !shownTipsRef.current.has(‘combo_nudge’)) {
shownTipsRef.current.add(‘combo_nudge’);
setToasts(t=>[…t,{id:Date.now(),text:‘💡 Equip 2 powers at once for combos!’,born:Date.now()}]);
}
}, [powers, discovered]);

// Combo hint toast (one-time per partner)
useEffect(() => {
if(comboHint && comboHint.partnerPower) {
setToasts(t=>[…t,{id:Date.now(),text:’💡 Try combining with ‘+comboHint.partnerPower.icon+’ ’+comboHint.partnerPower.name,born:Date.now()}]);
}
}, [comboHint]);

// ─── TOAST CLEANUP ───
useEffect(() => {
if(toasts.length===0)return;
const timer=setTimeout(()=>setToasts(t=>t.filter(tt=>Date.now()-tt.born<2500)),2600);
return()=>clearTimeout(timer);
}, [toasts]);

// ─── DERIVED STATE ───
const filteredPowers = (catFilter ? POWERS.filter(p=>p.cat===catFilter) : POWERS).filter(p=>!powerSearch || p.name.toLowerCase().includes(powerSearch.toLowerCase()) || p.char.toLowerCase().includes(powerSearch.toLowerCase()) || p.cat.toLowerCase().includes(powerSearch.toLowerCase()));
const shakeX = shake>0?(Math.random()-0.5)*shake*12:0;
const shakeY = shake>0?(Math.random()-0.5)*shake*12:0;
const totalDiscoveries = 500;
const pxLevel = phoenixRef.current.level;

// ─── RENDER ───
return (
<div style={{width:“100vw”,height:“100vh”,background:”#08080f”,display:“flex”,flexDirection:“column”,fontFamily:”‘SF Pro Display’,-apple-system,system-ui,sans-serif”,color:”#d4d4d8”,overflow:“hidden”,userSelect:“none”}}>

```
  {/* TOP BAR */}
  <div style={{padding:"8px 14px",display:"flex",alignItems:"center",gap:10,borderBottom:"1px solid #18182b",background:"#0c0c18",flexShrink:0,zIndex:20}}>
    <div style={{fontSize:18,fontWeight:800,letterSpacing:"-0.03em"}}>
      <span style={{color:"#f59e0b"}}>X</span><span style={{color:"#888"}}>-</span>SIM
      <span style={{fontSize:9,color:"#444",marginLeft:4,fontWeight:400}}>v4.5</span>
    </div>

    <div style={{display:"flex",gap:6,marginLeft:12}}>
      {powers.map(pid=>{
        const pw=POWERS.find(p=>p.id===pid);
        return pw?(
          <div key={pid} style={{display:"flex",alignItems:"center",gap:5,padding:"3px 10px",borderRadius:6,background:`${pw.color}15`,border:`1px solid ${pw.color}40`,fontSize:12}}>
            <span>{pw.icon}</span>
            <span style={{color:pw.color,fontWeight:600}}>{pw.char}</span>
            {pw.id==="phoenix"&&pxLevel>1&&<span style={{fontSize:9,color:"#f59e0b",fontWeight:700}}>Lv{pxLevel}</span>}
            <button onClick={()=>togglePower(pid)} style={{background:"none",border:"none",color:"#666",cursor:"pointer",fontSize:14,lineHeight:1,padding:0,marginLeft:4}}>×</button>
          </div>
        ):null;
      })}
    </div>

    {activeCombo&&(
      <div style={{display:"flex",alignItems:"center",gap:6,padding:"3px 12px",borderRadius:6,background:`${activeCombo.color}20`,border:`1px solid ${activeCombo.color}60`,animation:"pulse 1.5s infinite"}}>
        <span style={{fontSize:13,fontWeight:700,color:activeCombo.color}}>⚡ {activeCombo.name}</span>
        <span style={{fontSize:11,color:"#888"}}>{activeCombo.desc}</span>
      </div>
    )}
    {twoClickSrc&&<div style={{fontSize:12,color:"#f59e0b",marginLeft:8,animation:"pulse 1s infinite"}}>Click destination...</div>}

    <div style={{flex:1}} />

    {activePower&&!activeCombo&&(
      <div style={{fontSize:11,color:"#555",marginRight:8}}>
        {activePower.id==="kinetic_charge"?"Hold+release to charge":activePower.id==="weather"?"Hold to build storm":activePower.mode==="drag"?"Drag objects":activePower.mode==="hold"?"Hold in scene":activePower.mode==="two_click"?"Click→Click":"Click objects"}
      </div>
    )}

    <button onClick={()=>setShowDiscovery(!showDiscovery)} style={{background:showDiscovery?"#1a1a30":"#18182b",border:`1px solid ${showDiscovery?"#f59e0b40":"#27274a"}`,color:showDiscovery?"#f59e0b":"#666",padding:"3px 10px",borderRadius:5,cursor:"pointer",fontSize:11}}>
      📖 {discovered.size}/{totalDiscoveries}
    </button>
    <button onClick={()=>setPowers([])} style={{background:"#18182b",border:"1px solid #27274a",color:"#666",padding:"3px 10px",borderRadius:5,cursor:"pointer",fontSize:11}}>Clear</button>
    <button onClick={()=>{const objs=objectsRef.current;if(objs.length>=2){const a=objs[0],b=objs[1];const dx=b.x-a.x,dy=b.y-a.y,dist=Math.sqrt(dx*dx+dy*dy);jointsRef.current.push({id:nextJointId.current++,type:"rope",objA:a.id,objB:b.id,length:dist,damping:0.1,breakForce:500});sfxUI(600);}}} style={{background:"#18182b",border:"1px solid #27274a",color:"#666",padding:"3px 6px",borderRadius:5,cursor:"pointer",fontSize:9}} title="Add rope joint between first 2 objects">🔗</button>
    <button onClick={resetScene} style={{background:"#18182b",border:"1px solid #27274a",color:"#666",padding:"3px 10px",borderRadius:5,cursor:"pointer",fontSize:11}}>Reset</button>
    {/* Weather indicator */}
    <span style={{fontSize:10,color:"#94a3b8",marginLeft:6}}>
      {weather==="clear"?"☀️":weather==="rain"?"🌧️":weather==="snow"?"❄️":weather==="wind"?"💨":weather==="storm"?"⛈️":weather==="sandstorm"?"🏜️":weather==="meteor_shower"?"☄️":""}
      <span style={{marginLeft:2,opacity:0.7}}>{weather}</span>
    </span>

    {/* ENVIRONMENT SELECTOR */}
    <div style={{display:"flex",gap:2,marginLeft:2}}>
      {ENV_KEYS.map(ek=>(
        <button key={ek} onClick={()=>{setEnv(ek);sfxUI(700);if(!shownTipsRef.current.has('env_tip')){shownTipsRef.current.add('env_tip');setToasts(t=>[...t,{id:Date.now(),text:'💡 Environments change physics + power strength!',born:Date.now()}]);}}} title={ENVIRONMENTS[ek]?.name||ek}
          style={{background:env===ek?"#1a1a3e":"#18182b",border:`1px solid ${env===ek?"#818cf840":"#27274a"}`,
          color:env===ek?"#818cf8":"#555",padding:"2px 5px",borderRadius:4,cursor:"pointer",fontSize:11}}>
          {ENVIRONMENTS[ek]?.icon||"?"}
        </button>
      ))}
    </div>

    {/* BUILD MODE */}
    <button onClick={()=>{setBuildMode(!buildMode);if(!buildMode)setPowers([]);sfxUI(buildMode?500:900);}}
      style={{background:buildMode?"#422006":"#18182b",border:`1px solid ${buildMode?"#f59e0b60":"#27274a"}`,
      color:buildMode?"#f59e0b":"#666",padding:"3px 8px",borderRadius:5,cursor:"pointer",fontSize:11,fontWeight:buildMode?700:400}}>
      {buildMode?"🔨 Build":"🔨"}
    </button>

    <div style={{display:"flex",alignItems:"center",gap:4,padding:"2px 6px",borderRadius:5,background:"#18182b",border:"1px solid #27274a"}}>
      <button onClick={()=>setMuted(!muted)} style={{background:"none",border:"none",color:muted?"#ef4444":"#666",cursor:"pointer",fontSize:13,padding:0,lineHeight:1}} title="Toggle mute (M)">{muted?"🔇":"🔊"}</button>
      <input type="range" min="0" max="100" value={Math.round(volume*100)} onChange={e=>setVolume(e.target.value/100)} style={{width:50,height:3,accentColor:"#f59e0b",cursor:"pointer",opacity:muted?0.3:0.7}} />
    </div>
    <button onClick={()=>setShowPanel(!showPanel)} style={{background:"#18182b",border:"1px solid #27274a",color:"#666",padding:"3px 10px",borderRadius:5,cursor:"pointer",fontSize:11}}>{showPanel?"◀":"▶"}</button>
  </div>

  <div style={{display:"flex",flex:1,overflow:"hidden"}}>
    {/* POWER PANEL */}
    {showPanel&&(
      <div style={{width:210,borderRight:"1px solid #18182b",background:"#0c0c18",display:"flex",flexDirection:"column",flexShrink:0,overflow:"hidden"}}>
        <div style={{display:"flex",flexWrap:"wrap",gap:3,padding:"6px 6px 4px",borderBottom:"1px solid #18182b"}}>
          <button onClick={()=>setCatFilter(null)} style={{background:!catFilter?"#1a1a30":"transparent",border:`1px solid ${!catFilter?"#33335a":"transparent"}`,color:!catFilter?"#fff":"#555",padding:"2px 7px",borderRadius:4,cursor:"pointer",fontSize:10,fontWeight:600}}>ALL</button>
          {CATEGORIES.map(c=>(
            <button key={c} onClick={()=>setCatFilter(catFilter===c?null:c)} style={{background:catFilter===c?`${CAT_COLORS[c]}18`:"transparent",border:`1px solid ${catFilter===c?CAT_COLORS[c]+"50":"transparent"}`,color:catFilter===c?CAT_COLORS[c]:"#555",padding:"2px 5px",borderRadius:4,cursor:"pointer",fontSize:10,fontWeight:500}}>{CAT_ICONS[c]} {c}</button>
          ))}
        </div>
        <div style={{flex:1,overflowY:"auto",padding:4}}>
          {filteredPowers.map(p=>{
            const active=powers.includes(p.id);
            return(
              <button key={p.id} onClick={()=>togglePower(p.id)} style={{display:"flex",alignItems:"center",gap:6,width:"100%",padding:"5px 7px",background:active?`${p.color}15`:"transparent",border:`1px solid ${active?p.color+"45":"transparent"}`,borderRadius:5,cursor:"pointer",color:active?p.color:"#888",marginBottom:1,transition:"all 0.12s",textAlign:"left",boxShadow:comboHint?.partnerId===p.id?'0 0 10px 2px #fbbf2480':'none'}}>
                <span style={{fontSize:15,width:22,textAlign:"center",flexShrink:0,position:'relative'}}>{p.icon}{(()=>{const mod=ENVIRONMENTS[env]?.powerMods?.[p.cat];if(!mod||mod===1)return null;return <span style={{position:'absolute',top:-4,right:-4,fontSize:7,fontWeight:700,color:mod>1?'#4ade80':'#f87171',lineHeight:1}}>{mod>1?'▲':'▼'}</span>;})()}</span>
                <div style={{overflow:"hidden",minWidth:0}}>
                  <div style={{fontSize:11,fontWeight:active?700:500,whiteSpace:"nowrap",overflow:"hidden",textOverflow:"ellipsis"}}>{p.name}</div>
                  <div style={{fontSize:9,color:active?`${p.color}99`:"#555",whiteSpace:"nowrap",overflow:"hidden",textOverflow:"ellipsis"}}>{p.char} — {p.desc}</div>
                </div>
              </button>
            );
          })}
        </div>
        <div style={{borderTop:"1px solid #18182b",padding:6,fontSize:9,color:"#444",lineHeight:1.5,maxHeight:90,overflowY:"auto"}}>
          {powers.length===1?(
            <>
              <div style={{fontWeight:600,color:"#666",marginBottom:2}}>Combo partners:</div>
              {COMBOS.filter(c=>c.a===powers[0]||c.b===powers[0]).map(c=>{
                const partnerId=c.a===powers[0]?c.b:c.a;
                const partner=POWERS.find(p=>p.id===partnerId);
                return partner?<div key={c.name} style={{color:"#555"}}>{partner.icon} {partner.char} → <span style={{color:c.color}}>{c.name}</span></div>:null;
              })}
              {COMBOS.filter(c=>c.a===powers[0]||c.b===powers[0]).length===0&&<div>No known combos. Try another!</div>}
            </>
          ):powers.length===0?<div>Select 1-2 powers. Certain pairs unlock combos!</div>:null}
        </div>
      </div>
    )}

    {/* SCENE */}
    <div ref={sceneRef} onPointerDown={handleDown} onPointerMove={handleMove} onPointerUp={handleUp} onPointerLeave={handleUp}
      style={{flex:1,position:"relative",overflow:"hidden",background:`radial-gradient(ellipse at 50% 120%, ${ENVIRONMENTS[env]?.ambientColor||"#111125"} 0%, #08080f 70%)`,cursor:buildMode?"copy":powers.length>0?"crosshair":"default",transform:`translate(${shakeX}px,${shakeY}px)`,touchAction:"none"}}>

      {/* Grid */}
      <svg style={{position:"absolute",inset:0,width:"100%",height:"100%",opacity:0.04,pointerEvents:"none"}}>
        {Array.from({length:40},(_,i)=><line key={`v${i}`} x1={i*40} y1="0" x2={i*40} y2="100%" stroke="#888" strokeWidth="0.5"/>)}
        {Array.from({length:30},(_,i)=><line key={`h${i}`} x1="0" y1={i*40} x2="100%" y2={i*40} stroke="#888" strokeWidth="0.5"/>)}
      </svg>

      {/* Canvas VFX layer */}
      <canvas ref={canvasRef} style={{position:"absolute",inset:0,width:"100%",height:"100%",pointerEvents:"none",zIndex:3}} />

      {/* Combo flash */}
      {comboFlash&&(
        <div style={{position:"absolute",inset:0,zIndex:15,pointerEvents:"none",background:`radial-gradient(circle at 50% 50%, ${comboFlash.color}20 0%, transparent 70%)`,animation:"combo-flash 1.5s ease-out forwards"}}>
          <div style={{position:"absolute",top:"50%",left:"50%",transform:"translate(-50%,-50%)",fontSize:20,fontWeight:800,color:comboFlash.color,textShadow:`0 0 30px ${comboFlash.color}`,animation:"combo-text 1.5s ease-out forwards",whiteSpace:"nowrap",pointerEvents:"none"}}>
            ⚡ {comboFlash.name} ⚡
          </div>
        </div>
      )}

      {/* Objects */}
      {objects.map(obj=>{
        const ef=obj.effects, st=obj.statuses;
        const has=(t)=>ef.some(e=>e.type===t);
        const hasSt=(t)=>st.some(s=>s.type===t);
        let bg=MAT_COLORS[obj.mat]||"#888";
        let border="#ffffff10";
        let shadow="";
        let anim="";

        if(hasSt("frozen")){bg="#67e8f9";border="#a5f3fc";shadow="0 0 15px #67e8f960";}
        if(hasSt("burning")){bg="#f97316";border="#fb923c";shadow="0 0 12px #f9731660";anim="flicker 0.2s infinite";}
        if(hasSt("electrified")){border="#a5f3fc";shadow="0 0 20px #38bdf8, 0 0 35px #0ea5e950";anim="shock-filter 0.1s infinite";}
        if(hasSt("corroding")){bg="#8a7a50";shadow="0 0 8px #d4a37340";anim="dissolve-filter 3s forwards";}
        if(hasSt("phased")){shadow="0 0 10px #a5b4fc40";}
        if(hasSt("armored")){border="#f87171";shadow="0 0 20px #f8717140";}
        if(has("freeze")){bg="#67e8f9";border="#a5f3fc";}
        if(has("burn")){bg="#f97316";anim="flicker 0.2s infinite";}
        if(has("shock")){anim="shock-filter 0.1s infinite";}
        if(has("diamond")){bg="#bae6fd";border="#bae6fd";shadow="0 0 12px #bae6fd60";}
        if(has("steel")){bg="#9ca3af";border="#9ca3af";}
        if(has("charge")){shadow="0 0 20px #d946ef, 0 0 35px #d946ef50";anim="pulse-glow 0.4s infinite";}
        if(has("slash")){anim="slash-flash 0.15s";border="#fff";}
        if(has("hit")){anim="hit-flash 0.3s";}
        if(has("dissolve")){anim="dissolve-filter 1.5s forwards";}
        if(has("scan")){border="#818cf8";shadow="0 0 15px #818cf860";}
        if(has("hex")){anim="hex-filter 0.3s infinite";shadow="0 0 15px #dc262660";}
        if(has("armor")){border="#f87171";shadow="0 0 20px #f8717140";}
        if(has("evolve")){anim="evolve-flash 0.8s";}
        if(has("drain")){anim="drain-filter 1s forwards";}

        // Temperature visual tint (v4.0)
        const objTemp = obj.temp||20;
        if(objTemp > 80 && !hasSt("burning")) { const a=Math.min(0.3,(objTemp-80)/300); bg=`rgba(239,68,68,${a})`; }
        if(objTemp < 0 && !hasSt("frozen")) { const a=Math.min(0.3,Math.abs(objTemp)/100); bg=`rgba(96,165,250,${a})`; }
        const densityBw = (MAT_PROPS[obj.mat]?.density||1) > 2.5 ? 3 : (MAT_PROPS[obj.mat]?.density||1) > 1.2 ? 2 : 1;

        const isCircle=obj.shape==="circle",isDiamond=obj.shape==="diamond";
        const hpPct=obj.hp/obj.maxHp;
        const showHp=hpPct<1;

        return(
          <div key={obj.id} title={`${obj.label} (${obj.mat}) HP:${obj.hp}/${obj.maxHp} T:${Math.round(obj.temp||20)}° M:${(obj.mass||1).toFixed(1)}${obj.statuses.length?' ['+obj.statuses.map(s=>s.type).join(',')+']':''}`} style={{position:"absolute",left:obj.x,top:obj.y,width:obj.w,height:obj.h,zIndex:5}}>
            <div style={{
              width:"100%",height:"100%",backgroundColor:bg,border:`${densityBw}px solid ${border}`,
              borderRadius:isCircle?"50%":isDiamond?4:5,
              transform:`scale(${obj.scale}) rotate(${obj.rotation+(isDiamond?45:0)}deg)`,
              opacity:obj.opacity,boxShadow:shadow||"0 2px 6px rgba(0,0,0,0.15)",
              transition:dragging===obj.id?"none":"transform 0.2s, opacity 0.2s, background-color 0.2s",
              animation:anim,cursor:powers.length>0?"crosshair":"grab",
              display:"flex",alignItems:"center",justifyContent:"center",flexDirection:"column",touchAction:"none",
            }}>
              <span style={{fontSize:isDiamond?12:11,transform:isDiamond?"rotate(-45deg)":"none",pointerEvents:"none"}}>{MAT_EMOJI[obj.mat]}</span>
              <span style={{fontSize:7,color:"#fff9",fontWeight:600,pointerEvents:"none",transform:isDiamond?"rotate(-45deg)":"none",marginTop:1,letterSpacing:"0.02em"}}>{obj.label}</span>
            </div>

            {/* HP bar */}
            {showHp&&(
              <div style={{position:"absolute",bottom:-6,left:0,width:"100%",height:3,background:"#1a1a2e",borderRadius:2,overflow:"hidden"}}>
                <div style={{width:`${hpPct*100}%`,height:"100%",background:hpPct>0.6?"#22c55e":hpPct>0.3?"#f59e0b":"#ef4444",borderRadius:2,transition:"width 0.2s"}} />
              </div>
            )}

            {/* Status icons */}
            {st.length>0&&(
              <div style={{position:"absolute",top:-12,left:0,display:"flex",gap:1,fontSize:8}}>
                {hasSt("burning")&&<span>🔥</span>}
                {hasSt("frozen")&&<span>❄️</span>}
                {hasSt("electrified")&&<span>⚡</span>}
                {hasSt("phased")&&<span>👻</span>}
                {hasSt("armored")&&<span>🛡️</span>}
                {hasSt("corroding")&&<span>🏜️</span>}
              </div>
            )}

            {/* Telepathy info */}
            {showInfo===obj.id&&(
              <div style={{position:"absolute",bottom:"calc(100% + 14px)",left:"50%",transform:"translateX(-50%)",background:"#12121f",border:"1px solid #818cf8",borderRadius:6,padding:"6px 10px",fontSize:10,whiteSpace:"nowrap",color:"#c4b5fd",zIndex:20,boxShadow:"0 0 12px #818cf830"}}>
                <div style={{fontWeight:700,marginBottom:2}}>{obj.label}</div>
                <div>Material: {obj.mat} {MAT_EMOJI[obj.mat]}</div>
                <div>HP: {Math.round(obj.hp)}/{obj.maxHp} {obj.fragile?"(fragile)":""}</div>
                <div>Burns: {MAT_RULES.burns[obj.mat]?"✅":"❌"} | Conducts: {MAT_RULES.conducts[obj.mat]?"✅":"❌"}</div>
                <div>Statuses: {st.length>0?st.map(s=>`${s.type}(${s.ticksLeft})`).join(", "):"none"}</div>
              </div>
            )}
          </div>
        );
      })}

      {/* Two-click source marker */}
      {twoClickSrc&&(()=>{const src=objects.find(o=>o.id===twoClickSrc.objId);return src?<div style={{position:"absolute",left:src.x-3,top:src.y-3,width:src.w+6,height:src.h+6,border:`2px dashed ${activePower?.color||"#fff"}`,borderRadius:6,animation:"pulse 0.8s infinite",pointerEvents:"none",zIndex:12}}/>:null;})()}

      {/* BUILD MODE SPAWNER TRAY */}
      {buildMode&&(
        <div style={{position:"absolute",bottom:8,left:"50%",transform:"translateX(-50%)",display:"flex",gap:3,padding:"6px 10px",background:"#0c0c18e0",border:"1px solid #f59e0b40",borderRadius:8,zIndex:22}}>
          {MATERIALS.map(m=>(
            <button key={m} onClick={()=>{setBuildMat(m);sfxUI(700);}}
              style={{display:"flex",flexDirection:"column",alignItems:"center",gap:1,padding:"3px 6px",borderRadius:5,cursor:"pointer",
              background:buildMat===m?`${MAT_COLORS[m]}25`:"transparent",border:`1px solid ${buildMat===m?MAT_COLORS[m]+"80":"#27274a"}`,minWidth:36}}>
              <span style={{fontSize:14}}>{MAT_EMOJI[m]}</span>
              <span style={{fontSize:7,color:buildMat===m?MAT_COLORS[m]:"#555"}}>{m}</span>
            </button>
          ))}
          <div style={{borderLeft:"1px solid #27274a",margin:"0 2px"}} />
          {["rect","circle","diamond"].map(sh=>(
            <button key={sh} onClick={()=>{setBuildShape(sh);sfxUI(700);}}
              style={{padding:"3px 6px",borderRadius:4,cursor:"pointer",fontSize:10,
              background:buildShape===sh?"#1a1a3e":"transparent",border:`1px solid ${buildShape===sh?"#818cf840":"#27274a"}`,
              color:buildShape===sh?"#818cf8":"#555"}}>
              {sh==="rect"?"▬":sh==="circle"?"●":"◆"}
            </button>
          ))}
          <span style={{fontSize:8,color:"#555",alignSelf:"center",marginLeft:2}}>{objects.length}/50 • Click=place • Click obj=delete</span>
            <div style={{display:'flex',gap:3,alignSelf:'center',marginLeft:4}}>
              <button onClick={()=>{const json=JSON.stringify({v:5,env,weather,objects:objects.map(o=>({x:o.x,y:o.y,w:o.w,h:o.h,mat:o.mat,label:o.label,shape:o.shape,behavior:o.behavior||undefined})),terrain:terrainRef.current,joints:jointsRef.current.map(j=>({type:j.type,objA:j.objA,objB:j.objB,length:j.length,damping:j.damping,breakForce:j.breakForce}))});navigator.clipboard?.writeText(json);setToasts(t=>[...t,{id:Date.now(),text:'Scene exported!',born:Date.now()}]);}} style={{background:'#ffffff10',border:'1px solid #ffffff20',color:'#ccc',borderRadius:4,padding:'2px 6px',cursor:'pointer',fontSize:9}}>💾</button>
              <button onClick={()=>{navigator.clipboard?.readText().then(t=>{try{const d=JSON.parse(t);if(d.objects){setObjects(d.objects.map((o,i)=>({...o,id:Date.now()+i+Math.random(),origX:o.x,origY:o.y,origMat:o.mat,scale:1,rotation:0,opacity:1,effects:[],statuses:[],vx:0,vy:0,hp:MAT_HP[o.mat]||100,maxHp:MAT_HP[o.mat]||100,fragile:!!MAT_FRAGILE[o.mat],temp:20,mass:(o.w||45)*(o.h||45)*(MAT_PROPS[o.mat]?.density||1)*0.01,resting:0})));if(d.env)setEnv(d.env);setToasts(t2=>[...t2,{id:Date.now(),text:'Scene imported!',born:Date.now()}]);}}catch(e){setToasts(t2=>[...t2,{id:Date.now(),text:'Invalid JSON',born:Date.now()}]);}});}} style={{background:'#ffffff10',border:'1px solid #ffffff20',color:'#ccc',borderRadius:4,padding:'2px 6px',cursor:'pointer',fontSize:9}}>📋</button>
              <button onClick={()=>{const mats=['metal','wood','glass','stone','water','organic','energy','explosive','rubber','ice','sand','crystal'];const shapes=['rect','circle','diamond'];const arr=[];const ct=10+Math.floor(Math.random()*11);for(let i=0;i<ct;i++){const m=mats[Math.floor(Math.random()*mats.length)],sh=shapes[Math.floor(Math.random()*3)],w=30+Math.random()*40,h=30+Math.random()*40;arr.push({id:Date.now()+i+Math.random(),x:Math.random()*(bounds.w-w),y:Math.random()*(bounds.h-h),w,h,mat:m,label:m[0].toUpperCase()+m.slice(1),shape:sh,origX:0,origY:0,origMat:m,scale:1,rotation:0,opacity:1,effects:[],statuses:[],vx:0,vy:0,hp:MAT_HP[m],maxHp:MAT_HP[m],fragile:!!MAT_FRAGILE[m],temp:20,mass:w*h*(MAT_PROPS[m]?.density||1)*0.01,resting:0});}setObjects(arr);sfxUI(800);}} style={{background:'#ffffff10',border:'1px solid #ffffff20',color:'#ccc',borderRadius:4,padding:'2px 6px',cursor:'pointer',fontSize:9}}>🎲</button>
              <button onClick={()=>{const arr=[];for(let i=0;i<8;i++){const m=['stone','wood','metal','glass','wood','stone','organic','explosive'][i],w=50,h=35;arr.push({id:Date.now()+i+Math.random(),x:bounds.w/2-w/2,y:bounds.h-(i+1)*h-5,w,h,mat:m,label:'Stack',shape:'rect',origX:0,origY:0,origMat:m,scale:1,rotation:0,opacity:1,effects:[],statuses:[],vx:0,vy:0,hp:MAT_HP[m],maxHp:MAT_HP[m],fragile:!!MAT_FRAGILE[m],temp:20,mass:w*h*(MAT_PROPS[m]?.density||1)*0.01,resting:0});}setObjects(arr);sfxUI(600);}} style={{background:'#ffffff10',border:'1px solid #ffffff20',color:'#ccc',borderRadius:4,padding:'2px 6px',cursor:'pointer',fontSize:9}}>🏗️</button>
              <button onClick={()=>{terrainRef.current=[...terrainRef.current,{id:Date.now(),type:"platform",x:Math.random()*(bounds.w-120),y:bounds.h*0.6,w:120,h:12,material:"stone",friction:0.8}];sfxUI(500);}} style={{background:'#ffffff10',border:'1px solid #ffffff20',color:'#ccc',borderRadius:4,padding:'2px 6px',cursor:'pointer',fontSize:9}} title="Add platform">🪨</button>
              <button onClick={()=>{terrainRef.current=[...terrainRef.current,{id:Date.now(),type:"wall",x:Math.random()*(bounds.w-20),y:bounds.h*0.3,w:20,h:120,material:"metal",friction:0.7}];sfxUI(500);}} style={{background:'#ffffff10',border:'1px solid #ffffff20',color:'#ccc',borderRadius:4,padding:'2px 6px',cursor:'pointer',fontSize:9}} title="Add wall">🧱</button>
              <button onClick={()=>{terrainRef.current=[...terrainRef.current,{id:Date.now(),type:"ramp",x:Math.random()*(bounds.w-100),y:bounds.h*0.5,w:100,h:60,material:"stone",angle:1}];sfxUI(500);}} style={{background:'#ffffff10',border:'1px solid #ffffff20',color:'#ccc',borderRadius:4,padding:'2px 6px',cursor:'pointer',fontSize:9}} title="Add ramp">📐</button>
              <button onClick={()=>{terrainRef.current=[];sfxUI(400);}} style={{background:'#ffffff10',border:'1px solid #ffffff20',color:'#ccc',borderRadius:4,padding:'2px 6px',cursor:'pointer',fontSize:9}} title="Clear terrain">🗑️</button>
            </div>
        </div>
      )}

      {/* Scenario selector (v5.0) */}
      {!scenario && <div style={{position:"absolute",top:50,left:10,display:"flex",flexDirection:"column",gap:2,zIndex:10}}>
        <span style={{fontSize:9,color:"#94a3b8",fontWeight:700}}>SCENARIOS</span>
        {SCENARIOS.map(sc=>(
          <button key={sc.id} onClick={()=>startScenario(sc)} style={{background:"#1a1a30e0",border:"1px solid #27274a",color:"#e5e7eb",padding:"2px 6px",borderRadius:4,cursor:"pointer",fontSize:9,textAlign:"left",whiteSpace:"nowrap"}} title={sc.desc}>
            {sc.name}
          </button>
        ))}
      </div>}

      {/* Scenario HUD */}
      {scenario && <div style={{position:"absolute",top:50,left:"50%",transform:"translateX(-50%)",background:"#0f0f23e0",border:"1px solid #f59e0b40",borderRadius:8,padding:"6px 16px",zIndex:20,textAlign:"center"}}>
        <div style={{fontSize:12,fontWeight:700,color:"#f59e0b"}}>{scenario.name}</div>
        <div style={{fontSize:9,color:"#94a3b8",marginTop:2}}>{scenario.desc}</div>
        {scenarioResult ? (
          <div style={{marginTop:4}}>
            <span style={{fontSize:14}}>{scenarioResult.win ? "⭐".repeat(scenarioResult.stars) : "❌"}</span>
            <div style={{fontSize:9,color:scenarioResult.win?"#22c55e":"#ef4444"}}>{scenarioResult.win?`${scenarioResult.time.toFixed(1)}s`:(scenarioResult.reason||"Failed")}</div>
            <button onClick={()=>{setScenario(null);scenarioRef.current=null;setScenarioResult(null);resetSceneRef.current?.();}} style={{background:"#f59e0b20",border:"1px solid #f59e0b40",color:"#f59e0b",padding:"2px 8px",borderRadius:4,cursor:"pointer",fontSize:9,marginTop:3}}>Back</button>
          </div>
        ) : (
          <div style={{fontSize:9,color:"#666",marginTop:2}}>⏱️ {((Date.now()-scenarioStartRef.current)/1000).toFixed(0)}s</div>
        )}
      </div>}

      {/* Discovery toasts */}
      <div style={{position:"absolute",bottom:buildMode?56:16,left:"50%",transform:"translateX(-50%)",display:"flex",flexDirection:"column",alignItems:"center",gap:4,zIndex:20,pointerEvents:"none"}}>
        {toasts.map(t=>(
          <div key={t.id} style={{background:"#1a1a30e0",border:"1px solid #f59e0b40",borderRadius:6,padding:"4px 12px",fontSize:11,color:"#f59e0b",fontWeight:600,animation:"toast-in 0.3s ease-out",whiteSpace:"nowrap"}}>
            🔍 {t.text}
          </div>
        ))}
      </div>

      {/* Storm build indicator */}
      {activePower?.id==="weather"&&holdIv&&(
        <div style={{position:"absolute",top:8,left:"50%",transform:"translateX(-50%)",background:"#0c0c18e0",border:"1px solid #93c5fd40",borderRadius:6,padding:"4px 12px",fontSize:12,color:"#93c5fd",fontWeight:600,zIndex:18,pointerEvents:"none"}}>
          🌩️ Stage {stormStageRef.current||1}/3 — {stormStageRef.current>=3?"FULL STORM!":"Building..."}
        </div>
      )}

      {/* Gambit charge indicator */}
      {activePower?.id==="kinetic_charge"&&holdIv&&(
        <div style={{position:"absolute",top:8,left:"50%",transform:"translateX(-50%)",background:"#0c0c18e0",border:"1px solid #d946ef40",borderRadius:6,padding:"4px 12px",fontSize:12,color:"#d946ef",fontWeight:600,zIndex:18,pointerEvents:"none"}}>
          🃏 Charging... Release to detonate!
        </div>
      )}
    </div>

    {/* DISCOVERY PANEL */}
    {showDiscovery&&(
      <div style={{width:240,borderLeft:"1px solid #18182b",background:"#0c0c18",display:"flex",flexDirection:"column",flexShrink:0,overflow:"hidden"}}>
        <div style={{padding:"8px 10px",borderBottom:"1px solid #18182b",fontSize:13,fontWeight:700,color:"#f59e0b"}}>
          📖 Discoveries ({discovered.size}/{totalDiscoveries})
          {(()=>{const cats={};discovered.forEach(d=>{const pre=d.split(':')[0];cats[pre]=(cats[pre]||0)+1;});const entries=Object.entries(cats).filter(([,v])=>v>0);return entries.length>0?<div style={{fontSize:8,color:'#9ca3af',marginTop:2}}>{entries.map(([k,v])=>k+':'+v).join(' • ')}</div>:null;})()}
        </div>
        <div style={{flex:1,overflowY:"auto",padding:8}}>
          {/* Combos section */}
          <div style={{fontSize:10,fontWeight:600,color:"#666",marginBottom:4}}>COMBOS ({[...discovered].filter(k=>k.startsWith("combo:")).length}/{COMBOS.length})</div>
          {COMBOS.map(c=>{
            const found=discovered.has(`combo:${c.effect}`);
            return(
              <div key={c.effect} style={{display:"flex",alignItems:"center",gap:6,padding:"3px 4px",borderRadius:4,marginBottom:1,opacity:found?1:0.35}}>
                <span style={{fontSize:11}}>{found?"⚡":"❓"}</span>
                <span style={{fontSize:10,color:found?c.color:"#444",fontWeight:found?600:400}}>{found?c.name:"???"}</span>
              </div>
            );
          })}

          {/* Destructions */}
          <div style={{fontSize:10,fontWeight:600,color:"#666",marginTop:8,marginBottom:4}}>DESTRUCTIONS ({[...discovered].filter(k=>k.startsWith("destroy:")).length}/{MATERIALS.length})</div>
          {MATERIALS.map(m=>{
            const found=discovered.has(`destroy:${m}`);
            return(
              <div key={m} style={{display:"flex",alignItems:"center",gap:6,padding:"2px 4px",opacity:found?1:0.35}}>
                <span style={{fontSize:11}}>{found?MAT_EMOJI[m]:"❓"}</span>
                <span style={{fontSize:10,color:found?MAT_COLORS[m]:"#444"}}>{found?m:"???"}</span>
              </div>
            );
          })}

          {/* Power × Material grid */}
          <div style={{fontSize:10,fontWeight:600,color:"#666",marginTop:8,marginBottom:4}}>INTERACTIONS ({[...discovered].filter(k=>k.startsWith("p:")).length})</div>
          <div style={{display:"flex",gap:1,flexWrap:"wrap"}}>
            {MATERIALS.map(m=>(
              <div key={m} style={{textAlign:"center",width:24}}>
                <div style={{fontSize:8}}>{MAT_EMOJI[m]}</div>
                {CATEGORIES.map(cat=>{
                  const catPowers=POWERS.filter(p=>p.cat===cat);
                  const found=catPowers.some(p=>discovered.has(`p:${p.id}+m:${m}`));
                  return <div key={cat} style={{width:8,height:8,borderRadius:"50%",background:found?CAT_COLORS[cat]:"#1a1a2e",margin:"1px auto",border:"1px solid #27274a"}} title={`${cat} × ${m}`}/>;
                })}
              </div>
            ))}
          </div>

          {/* Status interactions */}
          <div style={{fontSize:10,fontWeight:600,color:"#666",marginTop:8,marginBottom:4}}>STATUS EVENTS ({[...discovered].filter(k=>k.startsWith("status:")).length})</div>
          {["fire+ice","electric+chain"].map(k=>{
            const found=discovered.has(`status:${k}`);
            return <div key={k} style={{fontSize:10,color:found?"#a5f3fc":"#333",padding:"2px 4px"}}>{found?`🔄 ${k}`:"❓ ???"}</div>;
          })}
        </div>
      </div>
    )}
  </div>

  <style>{`
    @keyframes pulse{0%,100%{opacity:1;}50%{opacity:0.5;}}
    @keyframes pulse-glow{0%,100%{filter:brightness(1);}50%{filter:brightness(1.6);}}
    @keyframes flicker{0%{filter:brightness(1);}33%{filter:brightness(1.4);}66%{filter:brightness(0.8);}}
    @keyframes shock-filter{0%,100%{filter:brightness(1);}50%{filter:brightness(1.8) contrast(1.3);}}
    @keyframes slash-flash{0%{filter:brightness(3) contrast(2);}100%{filter:brightness(1) contrast(1);}}
    @keyframes hit-flash{0%{filter:brightness(3);}100%{filter:brightness(1);}}
    @keyframes dissolve-filter{to{opacity:0;filter:blur(4px) brightness(0.3);}}
    @keyframes hex-filter{0%,100%{filter:hue-rotate(0deg) brightness(1);}50%{filter:hue-rotate(90deg) brightness(1.5);}}
    @keyframes evolve-flash{0%{filter:brightness(2) hue-rotate(0deg);}50%{filter:brightness(3) hue-rotate(180deg);}100%{filter:brightness(1) hue-rotate(360deg);}}
    @keyframes drain-filter{to{filter:brightness(0.3) saturate(0);opacity:0.5;}}
    @keyframes combo-flash{0%{opacity:1;}100%{opacity:0;}}
    @keyframes combo-text{0%{opacity:0;transform:translate(-50%,-50%) scale(0.5);}20%{opacity:1;transform:translate(-50%,-50%) scale(1.1);}40%{transform:translate(-50%,-50%) scale(1);}100%{opacity:0;transform:translate(-50%,-50%) scale(1) translateY(-30px);}}
    @keyframes toast-in{0%{opacity:0;transform:translateY(10px);}100%{opacity:1;transform:translateY(0);}}
    div::-webkit-scrollbar{width:3px;}
    div::-webkit-scrollbar-track{background:transparent;}
    div::-webkit-scrollbar-thumb{background:#2a2a40;border-radius:3px;}
    *{box-sizing:border-box;}
    button{font-family:inherit;}
  `}</style>
  {/* ONBOARDING OVERLAY */}
  {onboardStep!==null && onboardStep<4 && (
    <div style={{position:"fixed",inset:0,zIndex:100,pointerEvents:onboardStep===0?"auto":"none",display:"flex",alignItems:"center",justifyContent:"center"}}
      onClick={onboardStep===0?()=>setOnboardStep(1):undefined}>
      {onboardStep===0&&(
        <div style={{background:"#0c0c18f0",border:"1px solid #818cf840",borderRadius:16,padding:"40px 60px",textAlign:"center",pointerEvents:"auto",maxWidth:400}}>
          <div style={{fontSize:28,fontWeight:800,marginBottom:8}}><span style={{color:"#f59e0b"}}>X</span>-SIM</div>
          <div style={{fontSize:14,color:"#888",marginBottom:16}}>Mutant Power Sandbox</div>
          <div style={{fontSize:12,color:"#555"}}>Click anywhere to begin</div>
        </div>
      )}
      {onboardStep===1&&(
        <div style={{position:"absolute",left:8,top:"50%",transform:"translateY(-50%)",background:"#0c0c18e0",border:"1px solid #818cf860",borderRadius:8,padding:"8px 14px",fontSize:12,color:"#93c5fd",pointerEvents:"none",animation:"pulse 1.5s infinite"}}>
          ← Pick a power to start
        </div>
      )}
      {onboardStep===2&&(
        <div style={{position:"absolute",top:"40%",left:"50%",transform:"translate(-50%,-50%)",background:"#0c0c18e0",border:"1px solid #818cf860",borderRadius:8,padding:"8px 14px",fontSize:12,color:"#93c5fd",pointerEvents:"none"}}>
          Now click an object to use your power!
        </div>
      )}
      {onboardStep===3&&(
        <div style={{position:"absolute",top:50,left:"50%",transform:"translateX(-50%)",background:"#0c0c18e0",border:"1px solid #f59e0b60",borderRadius:8,padding:"8px 14px",fontSize:12,color:"#f59e0b",pointerEvents:"none"}}>
          ⚡ Try equipping 2 powers for a combo!
          <button onClick={()=>setOnboardStep(4)} style={{marginLeft:8,background:"none",border:"1px solid #f59e0b40",color:"#f59e0b",borderRadius:4,padding:"2px 8px",cursor:"pointer",fontSize:10,pointerEvents:"auto"}}>Got it</button>
        </div>
      )}
    </div>
  )}
</div>
```

);
}