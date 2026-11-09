# Project Bulletstorm — Complete Game Requirements Specification

**Version:** 1.0  
**Project:** The Build Hypothesis — Eight-Hour AI Coding Agent Experiment  
**Status:** Implementation-ready baseline  
**Genre:** Stationary, lane-based 3D arcade shooter  
**Targets:** Desktop browser and mobile browser  
**Hosting:** Cloudflare Workers Static Assets (or Cloudflare Pages if documented)  
**Reference inspirations:** The short shooting sequences in games such as Last Z and Last War. All game code, names, characters, visual assets, audio, and UI must be original or appropriately licensed.

## 1. Executive Summary

Build a polished, fully playable, single-player 3D arcade shooter. The player's squad occupies the near end of a bounded combat lane. The environment, enemies, hazards, and shootable upgrade targets advance toward the stationary-depth squad. The player steers the squad left/right; soldiers fire automatically. Shooting targets recruits soldiers, upgrades weapons, or grants temporary combat bonuses. Enemies and hazards can kill individual soldiers. **Defeat occurs only when every soldier is dead.** Surviving the final encounter completes the level, awards currency, and unlocks the next level. Players can purchase persistent upgrades between levels. There is **no base building, world map, city management, strategy gameplay, energy system, advertisements, or monetization**.

The primary deliverable is a working deployed game, not a mockup. A clean, playable core is more important than speculative feature breadth. Every required feature below is expected within an eight-hour autonomous coding-agent run; where scope is constrained, follow the priority order in Section 20 and report gaps transparently.

### 1.1 Locked design decisions
- Camera: fixed elevated third-person **3D perspective**, facing down the lane.
- Movement: squad stays at a fixed forward/depth coordinate and moves horizontally only; world objects move toward it.
- Defeat: **all soldiers must be eliminated**. There is no special protected leader.
- Progression: both **in-run temporary upgrades** and **between-level permanent upgrades**.
- Combat: automatic shooting; player positions the squad to select targets and avoid threats.
- Runtime AI: **not required**; deterministic game logic preferred. Do not send player input or combat decisions to an LLM.

### 1.2 Out of scope
Accounts, login, multiplayer, leaderboards requiring a server, real-money purchases, gacha, battle passes, advertising, push notifications, strategy/base building, resource production, PvP, user-generated levels, native app packages, online matchmaking, backend databases, or proprietary assets copied from reference games.

## 2. Player Experience and Game Flow

1. Launch game; show title screen with Play, Level Select, Upgrades, Settings, and How to Play.
2. First launch presents a 20–30 second interactive tutorial or clear instructional overlay.
3. Start unlocked level with a squad of at least one soldier and equipped starting weapon.
4. Move left/right while the squad automatically fires toward approaching targets and enemies.
5. Shoot reward targets to recruit units and enhance damage; dodge or destroy hazards; kill enemies.
6. Survive escalating waves and a final encounter.
7. On victory, show score, kills, soldiers remaining, rewards, and newly unlocked level.
8. Spend coins on permanent upgrades or replay/advance.
9. On defeat (zero living soldiers), show results and Retry/Upgrades/Main Menu.

**Session length:** typical level 2–4 minutes, with the first level shorter (~90–150 seconds). No unavoidable waiting, timers outside gameplay, or forced ad breaks.

## 3. Camera, World and Coordinates

### 3.1 World geometry
- Three.js scene with a stylized lane/platform, visible edges, environmental details, lighting, shadows or fake shadows, and fog/distance treatment.
- World coordinates: X = horizontal movement, Y = vertical, Z = forward/depth. Squad at a fixed near-end Z; objects spawn at far Z and move toward squad Z.
- Typical lane width: 12 world units; configurable. Squad centroid constrained to lane bounds with allowance for formation width.
- Fixed perspective camera, elevated and tilted down the lane; show the entire lane width and enough distance to react to approaching objects.
- Prefer 60–75° vertical FOV or equivalent framing adjusted to viewport; avoid excessive distortion.
- World movement must be based on `deltaTime`, not frames. Gameplay simulation must not depend on render FPS.
- No player-controlled forward/backward movement, free camera, or camera rotation.

### 3.2 Responsive layouts
- Desktop landscape: use available viewport, with camera framing preserving lane visibility and HUD legibility.
- Mobile portrait: camera framing and UI adapt; avoid clipping objects at lane edges or hiding targets behind HUD.
- Handle resizing and device-pixel-ratio changes without losing game state.
- Clamp render pixel ratio to preserve performance on high-DPI devices.

### 3.3 Visual clarity
- Friendly targets: cyan/blue/green; penalties/enemies: orange/red; rare/special: purple/gold. Do not rely on color alone: use icons and text.
- Reward labels remain legible at gameplay distance; billboard or overlay labels permitted.
- Damage feedback, target health bars, muzzle flashes, impact effects, hit flashes, and floating reward text.
- Avoid visually obscuring target values with particles or excessive projectiles.

## 4. Controls and Accessibility

### 4.1 Inputs
- Desktop: A/D and left/right arrow keys; mouse movement or click-and-drag across the game area.
- Mobile: one-finger drag across the game area; movement is relative to drag to prevent finger occlusion.
- Fire automatically when a level is active; no tap-to-fire requirement.
- Pause: Esc/P and visible pause button; restart from pause/results.
- Settings: sound/music volume, mute, reduced effects, sensitivity (or movement smoothing), and optional vibration where supported.

### 4.2 Movement
- Default horizontal movement speed ~9 world units/second; tune through playtesting.
- Keyboard: smooth acceleration/deceleration or responsive direct velocity.
- Pointer/touch: map displacement to target X and interpolate to avoid teleportation.
- Squad formation follows the center anchor with smoothing and is clamped to lane bounds.
- Prevent accidental page scrolling and text selection during touch gameplay.
- Input disabled during pause, modal overlays, and results.

### 4.3 Accessibility
- Legible high-contrast UI and scalable text where practical.
- Important information conveyed by icons/text in addition to colors.
- Reduced-effects option for particles and screen shake; no flashing sequences.
- Full gameplay accessible with keyboard on desktop; touch support on mobile.
- Provide clear instructions, controls, pause/resume, and mute.

## 5. Squad System

### 5.1 Units and formation
- Start level 1 with 1 soldier, modified by permanent starting-squad upgrade.
- Every living soldier contributes weapon fire and has its own health; default soldier health = 100.
- No privileged leader; any soldier may die and the remaining soldiers continue.
- Formation: centered staggered rows or compact grid. Maintain visually understandable spacing as squad grows.
- For up to ~30 soldiers, render individual characters; for larger squads, use instanced meshes or other optimized representation.
- Suggested practical soft cap = 100 active soldiers; any rewards beyond cap convert to a small score bonus. Document cap visibly or in help.
- When a soldier dies, remove it from combat and compact formation smoothly.
- Damage distribution should be fair and deterministic (nearest/impacted unit first, not arbitrary hidden losses).

### 5.2 Recruitment
- Positive recruitment rewards add the displayed number of soldiers **when the target is successfully destroyed**.
- Typical values: +1, +3, +5, +10, +20; larger values require more damage or more risk.
- Recruitment is immediate, accompanied by spawn animation and HUD count update.
- Squad multiplier targets (e.g., ×2) may be introduced in later levels; multiplier applies to current living count, rounded down, capped at squad limit.
- Do not award rewards for targets that merely pass the squad undestroyed.

### 5.3 Damage and defeat
- Enemy contact, enemy projectiles, and hazards damage soldiers.
- Soldier death occurs at health <= 0; dead soldiers stop firing immediately.
- Game enters Defeat state when living soldier count reaches zero; prevent subsequent rewards/spawns from resurrecting a defeated run.
- Invulnerability windows after damage are optional but must be consistent and documented.
- No unavoidable instant wipe during early levels; telegraph dangerous threats.

## 6. Combat and Targeting

### 6.1 Auto-fire
- All living soldiers auto-fire at their weapon's fire rate while playing.
- Primary aiming direction is forward down the lane. Slight automatic correction toward a target directly in the soldier's firing corridor is permitted.
- **Positioning matters:** do not allow global auto-aim that hits reward targets across the entire lane.
- Bullets or hitscan are acceptable if visual projectiles and consistent collision outcomes are shown; prefer pooled projectile objects.
- Combat logic uses simulation time, with consistent DPS independent of frame rate.
- Targets are selected based on forward alignment, proximity, and line of sight; no shots through blocking obstacles unless a piercing upgrade is active.
- Each soldier's fire can contribute to a target's health; clearly display progress on shootable rewards.

### 6.2 Damage model
- Default pistol: 10 damage/shot, 2 shots/second per soldier = 20 theoretical DPS per soldier.
- Enemy armor may reduce damage by a documented factor; use minimum effective damage of 1.
- Crits, splash, and piercing apply only when corresponding upgrades are active.
- All damage, firing intervals, and projectile speed configured in data rather than scattered constants.
- Collision groups: player projectiles vs enemies/reward targets/hazards; enemy projectiles vs soldiers; enemy contact vs soldiers; hazards vs soldiers. Avoid friendly fire.

### 6.3 Combat feedback
- Muzzle flash, shot sound, impact feedback, enemy hit reaction, kill animation/dissolve, health bars for durable targets, reward popups, and optional damage numbers.
- Prioritize feedback legibility over expensive particle counts.
- Explosions may deal area damage but must not exceed performance budgets.

## 7. Weapons and Temporary Upgrades

### 7.1 Weapons (minimum four implemented)

| Weapon | Damage/shot | Shots/sec | Special property | Target role |
|---|---:|---:|---|---|
| Pistol | 10 | 2.0 | Reliable single projectile | Starter |
| SMG | 7 | 5.0 | High fire rate, mild spread | Early unlock |
| Assault Rifle | 14 | 4.0 | Balanced sustained damage | Midgame |
| Shotgun | 7 × 5 pellets | 1.5 | Cone spread; falloff | Close-range crowds |
| Minigun | 9 | 9.0 | Sustained fire; spin-up optional | Late unlock |
| Rocket Launcher | 55 | 0.8 | Splash radius ~1.5 units | Heavy groups/boss |

Implement at least pistol, SMG, assault rifle, and one of shotgun/minigun/rocket launcher; additional weapons earn stretch credit. Values are initial tuning targets, not hard constraints. Weapons must differ visibly and functionally.

### 7.2 Weapon assignment
- The squad uses a shared active weapon tier by default for clarity and efficiency; changing weapon upgrades all living soldiers.
- Individual soldier weapon variation is optional stretch scope, not required.
- Temporary weapon changes last until end of the level; permanent upgrades affect starting loadout or baseline stats.

### 7.3 Temporary upgrades
Required types: weapon upgrade, +damage, +fire rate, +soldiers, and heal/shield. Optional: multishot, piercing, explosive rounds, critical chance, temporary invulnerability.
- Collect by destroying a shootable target before it reaches the squad.
- Upgrades take effect immediately and expire at level end unless explicitly permanent.
- Multiplicative stacking must be bounded to avoid runaway performance (e.g., fire rate cap, projectile cap).
- Clearly distinguish current run upgrades from persistent upgrades in UI.

## 8. Shootable Targets and Hazards

### 8.1 Target categories
- Recruitment crate: grants displayed +N soldiers on destruction.
- Weapon crate: upgrades shared weapon tier or grants displayed weapon.
- Damage/fire-rate crate: grants a clearly labeled stat bonus.
- Health/shield crate: heals living units or provides temporary protection.
- Multiplier gate: optional, increases squad count on destruction.
- Penalty hazard: damages/removes soldiers if it reaches the squad; can be dodged or destroyed when indicated.
- Solid blocker: stops or absorbs shots, forcing horizontal repositioning.

### 8.2 Health and rewards
- Each target has explicit HP, a displayed reward, and destruction effect.
- Example: +5 recruit crate HP 80; +10 HP 160; +20 HP 300; damage upgrade HP 120.
- Balance HP against expected squad DPS and target travel time.
- Positive rewards must not be granted on timeout, miss, or collision unless clearly specified.
- Penalty objects should not be mistaken for positive rewards; readable red iconography and warning effects.

### 8.3 Escalating reward target (required signature mechanic)
- At least one target type evolves its reward with sustained shooting: e.g., +2 → +5 → +10 → +20.
- Progress thresholds are visible (HP segments or progress ring), with a maximum reward.
- Reward is granted when the target reaches its final break threshold, or when its reward is locked in by a clearly communicated interaction. **Choose and document one unambiguous rule.**
- Suggested rule: successive thresholds upgrade reward, and the target breaks/awards at final destruction HP; UI shows current prospective reward throughout.
- Targets disappear safely when reaching player depth; missed positive targets give no reward.

## 9. Enemies and AI

### 9.1 Required enemy roster
| Enemy | Example HP | Speed relative to baseline | Attack | Counterplay |
|---|---:|---:|---|---|
| Walker | 35 | 1.0× | Contact | Basic shooting |
| Runner | 20 | 1.8× | Fast contact | Prioritize quickly |
| Armored | 120 | 0.7× | Heavy contact | Focus fire / upgrades |
| Shielded | 80 | 0.9× | Front damage reduction | Flanking / piercing |
| Exploder | 45 | 1.2× | Small-area explosion on arrival | Kill at distance |
| Boss | 1,500+ scaled | 0.3–0.7× | Phased attacks / summons | Dodge, sustain DPS |

Minimum five regular enemy types plus one boss in the full game. All enemy behaviors must be meaningfully distinct; recolors alone do not qualify.

### 9.2 Enemy behavior
- Move toward player at configurable speed; some may change lanes or strafe.
- Basic enemies pursue nearest available soldier when within attack range.
- Ranged enemy is optional; if present, telegraph shots and allow dodging.
- Enemies may have spawn lanes and approach patterns; avoid impossible overlapping hazard walls.
- Enemy health and speed increase moderately by level; no exponential spikes without compensating upgrades.
- Boss has at least two discernible behaviors/phases and a visible boss health bar.

### 9.3 Contact and collision
- Contact deals configured damage to affected soldier(s); apply cooldown or consume enemy to prevent accidental per-frame damage spam.
- Exploders produce a telegraphed radius; explosion occurs on reaching squad or death if specified.
- Projectiles/area effects obey fixed collision semantics and do not double-apply damage.

## 10. Waves, Level Design, and Generation

### 10.1 Level structure
- Minimum **10 complete playable levels**, with level 1 tutorialized and levels 5 and 10 containing boss finales; additional boss encounters optional.
- Normal levels: approximately 2–4 minutes; tutorial level: ~90–150 seconds.
- Each level contains scripted or seeded sequences of reward targets, hazards, and enemies, not purely uniform random spawns.
- Suggested phases: introduction → growth → pressure → final horde → boss/finale → results.
- Distinct layouts: paired target choices, staggered targets, alternating hazards, moving blockers, elite waves, mixed enemy formations.
- At least three distinct environmental visual themes across 10 levels (e.g., road, industrial yard, desert), using original simple assets.

### 10.2 Level progression
| Level | Introduced mechanic | Finale |
|---|---|---|
| 1 | Movement, auto-fire, +soldier targets, walkers | Small walker wave |
| 2 | Damage upgrades, runners | Mixed wave |
| 3 | Weapon crates, hazards | Armored enemy cluster |
| 4 | Escalating reward targets, shielded enemies | Formation challenge |
| 5 | First boss and special attack telegraph | Boss A |
| 6 | Faster approach, paired upgrade choices | Dense horde |
| 7 | Exploders, lane pressure | Survival wave |
| 8 | Mixed upgrade/penalty lanes | Elite wave |
| 9 | High-value recruitment, multiple enemy types | Large horde |
| 10 | Full mechanic combination | Multi-phase Boss B |

### 10.3 Fairness rules
- Player must have sufficient visual warning to respond to hazards.
- Never generate an unavoidable lethal barrier across all reachable lanes without a destructible escape.
- Ensure a viable route for a reasonably upgraded player; allow skill to compensate for missed rewards.
- Do not spawn rewards inside obstacles or behind impenetrable blockers.
- Level generator must support deterministic seed reproduction for debugging and evaluation.

### 10.4 Spawn system
- Data-driven timeline: `time`, `kind`, `laneX`, `speed`, `health`, `payload`, optional `pattern`.
- Seeded PRNG for procedural variations; avoid unseeded `Math.random()` in core level outcomes.
- Define maximum concurrent enemy and projectile counts; use pooling/instancing when necessary.
- All spawn scheduling pauses with simulation time.

## 11. Progression, Economy, and Saving

### 11.1 In-level progression
- Temporary recruitment, weapon changes, and combat buffs reset at the start of each new level.
- Permanent upgrades establish the starting baseline before temporary bonuses are applied.
- Clear separation between level score and spendable coins.

### 11.2 Permanent upgrades
| Upgrade | Effect per level | Max level | Example cost formula |
|---|---|---:|---|
| Starting Squad | +1 starting soldier | 10 | 100 × 1.6^n |
| Base Damage | +8% base damage | 10 | 120 × 1.6^n |
| Fire Rate | +5% base fire rate | 10 | 140 × 1.6^n |
| Soldier Health | +10% soldier max HP | 10 | 100 × 1.6^n |
| Starting Weapon | Unlock stronger initial weapon | 4 | Tiered: 300/700/1500/3000 |

Here `n` is the number of levels already purchased; round costs to whole coins. These values are defaults subject to tuning.

### 11.3 Currency
- Victory grants e.g. `100 + 25 × (level - 1)` coins plus up to 50 performance bonus.
- Defeat grants modest consolation coins (e.g., 25% of base reward), so repeated attempts can progress without trivializing difficulty.
- No premium currency, real-money purchases, or time-gated earning.
- Upgrade purchases must be atomic and reject insufficient funds.

### 11.4 Save data
- Store locally in `localStorage` with a versioned schema: unlockedLevel, highestCompletedLevel, coins, upgradeLevels, settings, bestScores, and optional run statistics.
- Autosave after completed level, upgrade purchase, or settings change.
- Corrupt or old save data must fail gracefully; migrate known schema versions or reset with clear messaging.
- Include Reset Progress with confirmation.
- No account required and no cloud persistence required.

## 12. Scoring and Difficulty

### 12.1 Score
- Score rewards enemy kills, target destruction, remaining soldiers, and completion speed.
- Show score during gameplay and breakdown on results screen.
- Score does not automatically equal coins; prevent replay exploit from awarding completion bonuses without completing.

### 12.2 Difficulty targets
- First level: most first-time players who understand movement should be able to complete after one or two tries.
- Levels 2–4: teach tradeoffs without abrupt difficulty walls.
- Levels 5–10: demand intentional upgrade choices, positioning, and threat prioritization.
- A typical competent player with reasonable permanent upgrades should finish most levels in 1–3 attempts.
- No forced grinding or unwinnable default starting state.

### 12.3 Initial balance model
- Soldier pistol DPS = 20 before bonuses.
- Example 5-soldier pistol squad DPS = 100 against aligned targets, before armor/misses.
- If target spends 5 seconds in effective range, a 5-soldier squad can theoretically deal 500 damage. Use this to size target HP and approach speed.
- Keep early targets achievable with 1–3 soldiers, and later targets scaled to expected squad growth.
- Cap simultaneous projectiles and explosive effects; preserve combat feel through batching or visual pooling.
- Balance config should be editable in one location without rewriting game systems.

## 13. UI / UX Screens

### 13.1 Main menu
- Original game title/logo, animated background, Play/Continue, Level Select, Upgrades, Settings, How to Play.
- Clearly display current highest unlocked level and coin balance.

### 13.2 Gameplay HUD
- Top left: level number, wave/phase, progress bar.
- Top center: contextual target/phase information; boss HP bar when applicable.
- Top right: score, pause, sound toggle (optional).
- Bottom or near squad: living soldier count, current weapon, current temporary buffs.
- HUD must not cover incoming high-value targets, especially on portrait mobile.

### 13.3 Pause menu
- Resume, Restart Level, Settings, Quit to Menu. Confirm if quitting loses current run.
- Gameplay simulation and sound events freeze when paused.

### 13.4 Results
- Victory/Defeat, score, kills, remaining soldiers, coins earned, time, level progress.
- Retry, Next Level (victory only), Upgrades, Main Menu.
- Prevent duplicate coin awards if results screen is reopened.

### 13.5 Upgrades screen
- Show upgrade name, current rank, next effect, next cost, purchase button, maxed status.
- Immediately update balance and save.
- Clearly label persistent upgrades; no base-building visuals or resource production.

### 13.6 Tutorial
- Teach horizontal movement, auto-fire, shootable +soldier crate, enemy damage, and objective.
- Skippable and replayable; first-play instructions concise.

## 14. Visual Assets and Audio

### 14.1 Art direction
- Original stylized low-poly 3D art, bright readable silhouettes, satisfying arcade VFX.
- At minimum: recognizable soldier model, five distinguishable enemy types, boss, six target/hazard models, lane environments, weapon representations, bullets/impacts, explosions, and UI icons.
- Procedural Three.js geometry is acceptable; agents may create original textures/models programmatically. Do not hotlink third-party assets or use unlicensed assets.
- Provide attribution for any permissively licensed assets, fonts, and audio.

### 14.2 Animation
- Soldiers idle/fire/move/die; enemies approach/hit/die; target destruction; boss telegraph; recruit pop-in.
- Procedural animation is acceptable if visually coherent.
- Do not require a specific commercial model format or external asset service.

### 14.3 Sound
- Distinct weapon firing, hits, enemy death, recruitment, upgrade acquisition, damage, victory, defeat, boss warning, UI click.
- Audio may be generated with Web Audio API or shipped as properly licensed files.
- Audio must start only after user interaction where browser autoplay policies require it.
- Independent SFX/music sliders or at least mute and volume controls; settings persist.

## 15. Technical Architecture

### 15.1 Recommended stack
- TypeScript, Vite, Three.js/WebGL.
- React optional for menus/HUD; do not add React if a simpler UI implementation is more reliable.
- Vitest for deterministic unit tests; Playwright for browser smoke tests.
- Static deployment to Cloudflare Workers Static Assets or Pages. No server API required.
- No mandatory runtime AI or external paid APIs; **do not use the user's OpenAI API key** for gameplay. No secret keys should be committed.

### 15.2 Architecture boundaries
- `AppShell`: bootstrapping, screens, settings, persistence.
- `GameLoop`: fixed-step simulation + render interpolation, pause/resume.
- `InputSystem`: keyboard, mouse, touch normalization.
- `WorldSystem`: lane, camera, background, level visual theme.
- `EntitySystem`: soldiers, enemies, projectiles, rewards, hazards.
- `SquadSystem`: recruitment, formation, health, death.
- `CombatSystem`: targeting, firing, damage, collision, status effects.
- `SpawnDirector`: seeded level timelines and encounter sequencing.
- `ProgressionSystem`: score, coins, unlocks, permanent upgrades.
- `UISystem`: HUD, menus, overlays, feedback.
- `AudioSystem`: effects and music/mute.
- `SaveSystem`: versioned local storage.
- `AssetManager`: original procedural assets, caching, disposal.

### 15.3 Suggested folder structure
```text
src/
  app/          # routing, screens, boot
  game/
    core/       # loop, clock, state, PRNG, config
    systems/    # input, movement, squad, combat, collision, spawn
    entities/   # soldiers, enemies, targets, projectiles
    rendering/  # scene, camera, meshes, effects, pools
    levels/     # level data, patterns, difficulty
    balance/    # weapons, enemies, upgrades, economy
  ui/           # menus, HUD, overlays
  audio/        # audio service
  persistence/  # versioned save schema
  tests/        # unit/integration tests
public/         # original or licensed static assets
```
Equivalent structures acceptable if responsibilities are cleanly separated.

### 15.4 Game state machine
`BOOT -> MENU -> LEVEL_LOADING -> PLAYING <-> PAUSED -> VICTORY|DEFEAT -> RESULTS -> MENU|LEVEL_LOADING`
- Transitions must be explicit and idempotent.
- Prevent double-awarding rewards, double-spawning bosses, duplicate listeners, and stale timers.
- Restart must fully reset transient game state without reloading the browser.

### 15.5 Simulation and determinism
- Fixed simulation step recommended: 1/60 sec, with accumulator and bounded catch-up steps.
- Use delta-time independent movement and cooldowns.
- Seeded random generation; level seeds stored/logged for replayability.
- Avoid gameplay logic inside render-only animation callbacks.
- Handle browser tab visibility: auto-pause when hidden; no huge delta jump on return.
- Dispose Three.js resources and event listeners on teardown/restart.

### 15.6 Collision and performance
- Use lightweight AABB/sphere/capsule approximations; a full rigid-body physics engine is unnecessary.
- Spatial grid or lane-based filtering when entity counts rise.
- Pool bullets, hit effects, and transient objects; use instancing for repeated soldiers/enemies where useful.
- Never allocate unbounded geometry/materials every frame.
- Test both small squads and stress scenarios.

## 16. Browser Performance and Reliability

### 16.1 Targets
- Desktop: aim for 60 FPS at 1080p on mainstream hardware.
- Mobile: aim for 30 FPS on midrange devices with reduced effects.
- Under a stress scenario of 100 soldiers, 150 enemies, and active projectiles, game should remain responsive, with no runaway memory growth or prolonged freezes.
- Initial interactive load target < 10 seconds on normal broadband, subject to network/device; no heavy unnecessary dependencies.
- All menus and core gameplay must work without backend connectivity after static assets are loaded.

### 16.2 Graceful degradation
- Lower particle counts, shadow quality, or render resolution on weaker devices.
- Show helpful message when WebGL is unavailable.
- No uncaught errors during normal play, pause, restart, or navigation.
- Avoid gameplay bugs when FPS drops.

## 17. Cloudflare Deployment

### 17.1 Requirements
- Production build generated by `npm run build` or documented equivalent.
- Deploy static game assets using Cloudflare Workers Static Assets (preferred) or Cloudflare Pages.
- Deployment should be reproducible using documented Wrangler commands.
- Use environment variable `APP_PUBLIC_URL` (optional) only if necessary for canonical links; do not hardcode a particular experiment hostname.
- Ensure deep-link fallback to index.html if using client-side routes.
- Include cache policy appropriate for fingerprinted assets and fresh HTML.
- Provide a deployment URL and verify it loads and plays in browser.
- If authentication/Cloudflare access blocks deployment, complete build and report the blocker; never invent successful deployment.

### 17.2 Repository and secrets
- Do not commit `.env`, `.dev.vars`, credentials, tokens, or personal API keys.
- Do not require OpenAI API access. If optional tooling uses a key during development, keep it local and excluded from source control.
- Include `.gitignore`, README, package lockfile, and license/asset attribution.
- Do not assume permission to change Cloudflare account settings or purchase services.

## 18. Testing Requirements

### 18.1 Unit tests (minimum)
1. Recruitment adds exactly N soldiers up to cap.
2. Multipliers (if implemented) use correct rounding and cap.
3. Damage reduces health; death removes unit and its firing contribution.
4. Defeat triggers only when **all** soldiers die; loss of an arbitrary lead unit is not defeat.
5. Weapon fire rate/DPS calculations are time-step independent within tolerance.
6. Target rewards are granted once and only on valid destruction.
7. Missed targets do not award rewards.
8. Permanent upgrades deduct correct cost, persist, and affect next-level baseline.
9. Temporary upgrades reset between levels.
10. Level unlock and victory reward occur once, even on repeated result transitions.
11. Pause freezes spawns/movement/damage; resume does not skip time.
12. Save schema handles missing/corrupt data safely.
13. Deterministic seed produces the same spawn sequence.

### 18.2 Browser integration tests
- Game boots, menu appears, level starts, movement works, HUD displays soldier count.
- Pause/resume/restart works without duplicated game entities.
- Simulate or trigger victory and defeat; results and progression correct.
- Upgrades persist across reload.
- Resize from landscape to portrait without breaking rendering or input.
- No console errors on primary flows.

### 18.3 Manual evaluation checklist
- Can player understand the game within 30 seconds?
- Does horizontal movement feel responsive and precise?
- Can the player intentionally choose between two incoming targets?
- Do weapons visibly and audibly feel different?
- Is recruiting additional soldiers immediately satisfying?
- Are hazards readable and avoidable?
- Is boss behavior recognizable and fair?
- Are menus polished and navigation intuitive?
- Is difficulty challenging but not arbitrary?
- Can a full ten-level progression be completed without broken transitions?

## 19. Acceptance Criteria / Definition of Done

### P0 — Must work to count as a game
- [ ] Browser launches a functioning Three.js 3D level with visible lane, camera, and squad.
- [ ] Keyboard, mouse, and touch horizontal movement work.
- [ ] Squad fires automatically; projectiles/hits damage targets and enemies.
- [ ] Shootable recruitment and weapon upgrade targets work.
- [ ] Multiple soldiers fire; soldiers take damage and die individually.
- [ ] Defeat only when **zero living soldiers** remain.
- [ ] At least one complete level with victory, defeat, retry, and result screen.
- [ ] No base-building or strategy systems.
- [ ] Build succeeds and game runs without fatal browser errors.

### P1 — Required full specification
- [ ] Ten playable levels with escalating difficulty and clear level unlocks.
- [ ] Five distinct regular enemies and at least one boss with two behaviors/phases.
- [ ] Four meaningfully different weapons.
- [ ] Five temporary upgrade/reward types including recruitment, damage, fire rate, weapon, and heal/shield.
- [ ] Escalating reward target mechanic.
- [ ] Persistent coins, five upgrade categories, and local save.
- [ ] Main menu, HUD, pause, results, upgrades, settings, tutorial.
- [ ] Three visually distinct level themes.
- [ ] Original or properly licensed visuals and sound effects.
- [ ] Responsive desktop/mobile layout.
- [ ] Automated unit tests and browser smoke tests.
- [ ] Cloudflare production deployment or clearly documented external blocker.
- [ ] README, controls, build instructions, and final report.

### P2 — Stretch (do not sacrifice P0/P1)
- [ ] Additional weapons, enemy types, and boss encounters.
- [ ] Procedural level variations and daily seeded challenge (offline).
- [ ] More sophisticated squad animation and formations.
- [ ] Advanced VFX, particle systems, haptics, music.
- [ ] Replay recording, accessibility options beyond minimum, adaptive graphics quality.

## 20. Eight-Hour AI Agent Experiment Protocol

### 20.1 Purpose
Compare Claude Code, Codex, and Cursor CLI on their ability to autonomously build the **same complete game** from the **same specification**, within a strict eight-hour wall-clock limit. Evaluate final player experience, not just code quantity.

### 20.2 Controlled setup
- Each agent starts from an equivalent clean repository containing this specification, identical toolchain expectations, and no implementation code.
- Same operating system/runtime versions where practical, equivalent Cloudflare permissions, equivalent available credentials (none required for game), and equivalent access to documentation.
- Give each agent the same complete task prompt and this spec; no agent-specific implementation hints or private rules.
- Record agent/model version, CLI version, environment, hardware, and start/end timestamps.
- All agents receive the same eight-hour wall-clock budget; setup delays and human intervention logged.
- If an agent finishes early, do not force it to consume the remaining time. Record elapsed time and assess delivered quality.
- No manual coding or fixes during a run. Any approvals, prompts, environment blockers, or interventions must be timestamped and logged.
- Use separate repos and separate deployment destinations; do not share generated code or feedback between agents before all runs finish.

### 20.3 Agent task prompt
> You are responsible for designing, implementing, testing, and deploying Project Bulletstorm, a complete browser-based stationary 3D arcade shooter, following the attached specification in full. You have up to eight hours of wall-clock time. Work autonomously, make reasonable implementation decisions without asking for nonessential clarification, and prioritize a polished playable game over placeholder features. Implement tests, validate the game in a browser when tools permit, and deploy to Cloudflare if access is available. Do not build any strategy or base-building systems. Do not claim features or deployment succeeded without verifying them. Before stopping, create BUILD_REPORT.md describing what was built, what was tested, known limitations, deployment status, and next steps. Do not expose or commit secrets.

### 20.4 Recommended time allocation (guidance, not hard gating)
| Window | Focus |
|---|---|
| 0:00–0:30 | Plan, scaffold, game architecture, asset strategy |
| 0:30–2:00 | 3D scene, controls, squad, auto-fire, collisions |
| 2:00–3:30 | Rewards, weapons, enemies, damage, defeat |
| 3:30–5:00 | Levels, boss, progression, persistence |
| 5:00–6:15 | UI, audio, effects, mobile layout |
| 6:15–7:15 | Tests, balancing, browser QA, bug fixes |
| 7:15–8:00 | Cloudflare deployment, final verification, report |

### 20.5 Evidence to collect
- Full screen recording of each agent session, with timestamps.
- Agent prompts, interruptions, approval requests, failures, retries, and human interventions.
- First successful build time; first visible 3D scene; first playable shooting; first complete level; first successful deployment.
- Git commit history (if available), test results, browser console errors, deployed URL, and screenshots/video of gameplay.
- Final implementation inventory and known defects.

### 20.6 Scoring rubric (100 points)
| Category | Points | Measurement |
|---|---:|---|
| Core gameplay and controls | 20 | Responsiveness, shooting, movement, squad, fairness |
| Feature completeness | 20 | P0/P1 implementation verified in-game |
| Fun, balance, replayability | 15 | Reward choices, pacing, satisfying growth, challenge |
| Visual polish and audio | 15 | 3D clarity, original art, effects, sound, UI polish |
| Stability and performance | 10 | Bugs, FPS, memory, restart/pause reliability |
| Levels and progression | 10 | Ten levels, enemies, boss, upgrades, save |
| Engineering and tests | 5 | Architecture, automated coverage, maintainability |
| Deployment and documentation | 5 | Live Cloudflare URL, reproducible setup, report |
| **Total** | **100** | |

**Important:** Time-to-first-playable and autonomy/interruptions are **reported separately**, not hidden inside quality scores. A fast but poor game should not automatically win. Also report completion time and count of human approvals/interruptions.

### 20.7 Evaluation method
- Evaluate all games on the same desktop and mobile/browser combinations.
- Play each game's first level without reviewing source first; record onboarding and first impressions.
- Attempt levels 1, 3, 5, and 10, using provided progress/unlock mechanics (or a documented testing mechanism) to inspect later levels.
- Repeat each critical test after a browser reload.
- Run a stress test with large squad/enemy counts and record FPS and crashes.
- Apply the same acceptance checklist and scoring rubric to all agents.
- Distinguish **implemented**, **visibly working**, **automated-test verified**, and **claimed only**.
- Include a side-by-side comparison of strengths, bugs, and player enjoyment.

## 21. Required Final Build Report

Each agent must create `BUILD_REPORT.md` at repository root containing:

1. **Summary:** What was built, how it plays, implementation approach.
2. **Run metadata:** Start/end timestamps, elapsed wall time, agent/model versions, platform.
3. **Architecture:** Major systems, folder structure, libraries, important design decisions.
4. **Feature checklist:** P0, P1, P2 with implemented/partial/missing status and evidence.
5. **Gameplay:** Controls, weapons, enemies, levels, progression, balancing choices.
6. **Testing:** Exact commands, test counts/results, browser/device checks, known failures.
7. **Performance:** Observed FPS or honest statement if not measured; stress test details.
8. **Deployment:** URL, Cloudflare configuration, commands, and whether actually verified.
9. **Assets/licensing:** How art/audio was created, third-party attributions if any.
10. **Known issues:** Severity, reproduction steps, user impact.
11. **Security:** Confirmation no keys/secrets were committed.
12. **Next steps:** Highest-impact improvements in priority order.

Also create `README.md` with local development, production build, testing, controls, save/reset behavior, and deployment instructions.

## 22. Risks and Tradeoffs

- **Scope risk:** 10 levels + 5 enemy types + boss + mobile polish is ambitious in eight hours. Deliver working P0 first; implement data-driven systems to scale efficiently.
- **Visual asset risk:** AI coding tools may struggle to produce cohesive 3D characters. Use original procedural low-poly geometry as a reliable baseline; do not block on external art generation.
- **Performance risk:** 100 soldiers shooting rapidly can create thousands of objects. Pool projectiles, instance repeated geometry, cap VFX, and decouple simulation from render.
- **Balance risk:** Exponential squad growth can trivialize combat. Cap squad size and multiplier frequency; use scripted target windows and bounded scaling.
- **Input risk:** Mouse and touch controls may behave differently. Normalize input to world X and test both orientations.
- **Deployment risk:** Cloudflare permissions may be insufficient. Keep game fully static and deployable; report blockers accurately.
- **Evaluation bias:** One agent may have more permissive approval defaults. Log interruptions separately and avoid unreported human assistance.

## 23. Final Delivery Checklist

Before declaring completion:
1. Install dependencies from clean checkout.
2. Run typecheck, lint (if configured), tests, and production build.
3. Start the game and finish at least one full level manually or with browser automation.
4. Verify squad recruitment, individual deaths, and all-soldiers-dead defeat condition.
5. Verify at least one weapon upgrade, one permanent purchase, and persistence after reload.
6. Verify pause/restart, victory/defeat, and no duplicate coin rewards.
7. Verify desktop keyboard/mouse and mobile touch interaction.
8. Check browser console for errors and inspect frame rate during a crowded wave.
9. Deploy to Cloudflare if permitted and open the live URL.
10. Complete `README.md` and `BUILD_REPORT.md` truthfully.

---

**End of specification v1.0.** This document is the common input for all three agent runs. Deviations must be reported, not silently substituted.
