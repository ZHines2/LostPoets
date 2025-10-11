# Lost Poets v0.3 🌲
**Living World Simulation** — A browser-based ASCII roguelike inspired by Dwarf Fortress

Open tile exploration adventure featuring procedural generation, crafting systems, and autonomous AI.

---

## 📖 Overview

Lost Poets is a single-file HTML5 game that combines:
- **Tile-based exploration** through forests, temples, and cavern layers
- **Procedural world generation** with cellular automata and seeded randomness
- **Living world simulation** with dynamic creatures, weather, and day/night cycles
- **Crafting progression** with 15+ recipes spanning basic tools to legendary items
- **Actor-based architecture** supporting future multiplayer and tactical combat
- **Autonomous AI** that can play the game independently

---

## 🎮 Quick Start

1. Open `index.html` in a modern browser
2. Use **WASD** to move around the Origin Temple
3. Press **H** for help and controls
4. Press **C** to open the crafting menu
5. **Shift + Direction** to use your selected tool

---

## 🕹️ Controls

| Key | Action |
|-----|--------|
| `W A S D` | Move character |
| `Shift + Direction` | Use active tool |
| `1 2 3 4` | Select tool (Spade/Club/Heart/Diamond) |
| `G` | Pick up items |
| `F` | Eat food |
| `P` | Place torch |
| `R` | Rest (recover HP) |
| `O` | Observe surroundings |
| `C` | Open crafting menu |
| `I` | View inventory |
| `H` / `K` | Toggle help |
| `V` | Toggle visibility/fog of war |
| `L` | Teleport to Secret Lab |
| `Q` | Reload game |
| `<` / `>` | Use stairs (ascend/descend) |

---

## 🏗️ Architecture Overview

### System Flowchart

```
┌─────────────────────────────────────────────────────────────────┐
│                         GAME INITIALIZATION                      │
│  • Load Forest with Temple • Seed RNG • Init Actor System       │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                         INPUT HANDLING                           │
│  Keyboard Events → Action Queue → Actor Core                    │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                         WORLD TICK                               │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ 1. Process Actions (Actor Core)                           │  │
│  │    • Movement • Tool Use • Rest • Observe                 │  │
│  ├───────────────────────────────────────────────────────────┤  │
│  │ 2. Sync Player State (Legacy Compatibility)              │  │
│  ├───────────────────────────────────────────────────────────┤  │
│  │ 3. Update Camera Position                                 │  │
│  ├───────────────────────────────────────────────────────────┤  │
│  │ 4. Increment Turn Counter                                 │  │
│  ├───────────────────────────────────────────────────────────┤  │
│  │ 5. World Simulations (Conditional by Turn)               │  │
│  │    • Hunger Decay (every 5 turns)                        │  │
│  │    • Creature AI (every 10 turns)                        │  │
│  │    • Flora Growth (every 12 turns)                       │  │
│  │    • Geology Changes (every 15 turns)                    │  │
│  │    • Creature Spawns (every 30 turns)                    │  │
│  │    • Weather Updates (every 150 turns)                   │  │
│  │    • Random Events (every 50 turns)                      │  │
│  │    • Day/Night Cycle (every 100 turns)                   │  │
│  └───────────────────────────────────────────────────────────┘  │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                         RENDERING                                │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ 1. Calculate Viewport (Camera-relative coordinates)      │  │
│  │ 2. Apply Visibility System (Fog of War + Torches)        │  │
│  │ 3. Render Tiles with Shading (v0-v9 opacity classes)     │  │
│  │ 4. Render Entities & Items                               │  │
│  │ 5. Update HUD (HP, Stamina, Hunger, Stats, Tools)        │  │
│  └───────────────────────────────────────────────────────────┘  │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           └──────────────┐
                                          │
                    ┌─────────────────────┴────────────────────┐
                    │                                          │
                    ▼                                          ▼
         ┌──────────────────────┐              ┌──────────────────────┐
         │   AI DRIVER (Async)  │              │  PLAYER INPUT (Next) │
         │  • Explore Mode      │              │  Waiting for action  │
         │  • Gather Mode       │              └──────────────────────┘
         │  • Dig Mode          │
         │  • Hunt Mode         │
         │  • Idle Mode         │
         └──────────────────────┘
```

---

## 🔧 Core Systems Explained

### 1. Actor Core System

The game uses a unified actor system for future-proofing multiplayer and AI-driven gameplay.

**Key Concepts:**
```javascript
// Actor structure
{
  id: number,           // Unique identifier
  type: 'player',       // Actor type (player, npc, creature)
  x, y: position,       // World coordinates
  hp, hpMax: health,    // Hit points
  str, int, agi: stats, // Core attributes
  stamina, stamMax,     // Action points
  facing: {dx, dy},     // Direction vector
  memory: {}            // AI state storage
}

// Action queue pattern
queueAction(actorId, {
  type: 'move',         // Action type
  params: {dx: 1, dy: 0}
});
```

**Flow:**
1. Player input generates actions via `requestPlayerMove()`, `requestPlayerDig()`, etc.
2. Actions are queued in `state.actorCore.actionQueue`
3. `processActions()` executes all queued actions during `worldTick()`
4. `syncPlayerFromActor()` maintains backward compatibility with legacy `state.player`

**Code Snippet:**
```javascript
function worldTick(){
  // Process actor actions first (Phase 0: player movement)
  if(state.actorCore) {
    processActions();
    syncPlayerFromActor(); // Keep legacy state.player in sync for UI/render
  }
  
  updateCamera();
  state.turns++;
  
  // Hunger decreases over time
  if(state.turns % 5 === 0 && state.player.hunger > 0){
    // Hunger mechanics...
  }
  // ... more simulation systems
}
```

---

### 2. World Simulation Loop

The world updates in discrete turns with various subsystems running at different frequencies.

**Simulation Schedule:**
- **Turn 0** (always): Process actions, update camera
- **Every 2 turns**: Stamina regeneration (0.3-1.5 based on hunger, effectively unlimited when well-fed)
- **Every 5 turns**: Hunger decay
- **Every 10 turns**: Creature AI (movement, state changes)
- **Every 12 turns**: Flora growth/regeneration
- **Every 15 turns**: Geology changes (ore regeneration)
- **Every 30 turns**: Creature spawning
- **Every 50 turns**: Random events (discoveries, changes)
- **Every 100 turns**: Day/night cycle transitions
- **Every 150 turns**: Weather updates

**Code Snippet:**
```javascript
function worldTick(){
  // ... actor processing ...
  
  // Scheduled simulations
  if(state.turns%10===0) simulateCreatures();
  if(state.turns%12===0) simulateFlora();
  if(state.turns%15===0) simulateGeology();
  if(state.turns%30===0) trySpawnCreature();
  if(state.turns%100===0) updateDayNightCycle();
  if(state.turns%150===0) updateWeather();
  if(state.turns%50===0) triggerRandomEvent();
  
  render();
}
```

---

### 3. Crafting System

Progressive crafting with 15+ recipes across 4 tiers: Basic → Intermediate → Advanced → Master.

**Recipe Structure:**
```javascript
{
  name: 'Iron Axe',
  inputs: {wood: 2, ore: 1},
  outputs: {inventory: 1},
  desc: 'Augments Spade (+1)',
  makeItem: () => ({
    name: 'Iron Axe',
    type: 'spade',      // Tool type to augment
    bonus: 1,           // Damage/power bonus
    special: 'mining'   // Special ability
  })
}
```

**Crafting Flow:**
1. Press `C` to open crafting menu
2. View available recipes (filtered by unlocked status)
3. Recipes unlock through progression:
   - **Basic**: Available at start (key, torch, axe)
   - **Intermediate**: Unlock after first craft
   - **Advanced**: Require portable workbench
   - **Master**: Require rare materials (diamond, obsidian, crystal)
4. Craft consumes inputs, adds items to inventory or resources

**Code Snippet:**
```javascript
function canCraftRecipe(recipe){
  for(const [res, amt] of Object.entries(recipe.inputs)){
    if(!state.resources[res] || state.resources[res] < amt) return false;
  }
  return true;
}

function craftRecipe(recipeId){
  const recipe = RECIPES[recipeId];
  if(!canCraftRecipe(recipe)) return;
  
  // Consume inputs
  for(const [res, amt] of Object.entries(recipe.inputs)){
    state.resources[res] -= amt;
  }
  
  // Generate outputs
  if(recipe.makeItem){
    state.inventory.push(recipe.makeItem());
  }
  // ...
}
```

---

### 4. Z-Level System (Vertical Exploration)

Multi-layer world generation with procedural caverns below the surface.

**Layer Types:**
- **Z=0**: Forest surface with Origin Temple
- **Z=-1 to -∞**: Procedural cavern layers (cellular automata generation)

**Generation Algorithm:**
```
1. Random noise fill (45% walls)
2. Cellular automata smoothing (4-5 iterations)
   - Rule: Cell becomes wall if 5+ neighbors are walls
3. Flood fill validation (ensure connectivity)
4. Embed special structures (Forgotten Shrine at Z=-1)
5. Populate with resources and creatures
```

**Code Snippet:**
```javascript
function generateCavernLayer(z){
  const w = FOREST_W, h = FOREST_H;
  
  // Step 1: Random noise fill (45% walls)
  let map = Array.from({length:h}, ()=>Array(w).fill(null));
  for(let y=0; y<h; y++){
    for(let x=0; x<w; x++){
      if(y===0 || x===0 || y===h-1 || x===w-1){
        map[y][x] = TILES.WALL; // Border
      } else {
        map[y][x] = (rf() < 0.45) ? TILES.WALL : TILES.FLOOR;
      }
    }
  }
  
  // Step 2-3: Cellular automata smoothing
  for(let iter=0; iter<4; iter++){
    const newMap = map.map(row => [...row]);
    for(let y=1; y<h-1; y++){
      for(let x=1; x<w-1; x++){
        const neighbors = countWallNeighbors(map, x, y);
        if(neighbors >= 5) newMap[y][x] = TILES.WALL;
        else if(neighbors <= 3) newMap[y][x] = TILES.FLOOR;
      }
    }
    map = newMap;
  }
  
  // Step 4: Place stairs
  map[20][20] = TILES.STAIRS_UP;
  map[80][80] = TILES.STAIRS_DOWN;
  
  // Step 5: Populate with resources/creatures
  populateCaverns(map, z);
  
  return map;
}
```

---

### 5. AI Driver System

Autonomous gameplay with multiple behavioral modes.

**AI Modes:**
- **Explore**: Move to unvisited tiles, prioritize item collection
- **Gather**: Focus on collecting specific resource types
- **Dig**: Target and excavate walls for ore
- **Hunt**: Seek and attack creatures
- **Idle**: Passive observation

**AI Architecture:**
```javascript
const aiDriver = {
  enabled: false,
  mode: 'explore',        // Current behavior mode
  speed: 500,             // Tick interval (ms)
  intervalId: null,       // Timer reference
  memory: {
    visitedTiles: Set,    // Exploration tracking
    targetResource: null, // Current gathering goal
    targetEnemy: null     // Current combat target
  },
  stats: {
    moveCount: 0,
    itemsCollected: 0,
    tilesExplored: 0,
    actionsPerformed: 0
  }
}
```

**Usage:**
- Press `9` to start AI in explore mode
- Press `0` to stop AI
- AI respects game rules (collision, stamina, tool usage)

**Code Snippet:**
```javascript
function aiTick(){
  if(!aiDriver.enabled || !state.playerId) return;
  
  const actor = state.actorCore.actors[state.playerId];
  if(!actor) return;
  
  // Mark current tile as visited
  const tileKey = `${actor.x},${actor.y}`;
  if(!aiDriver.memory.visitedTiles.has(tileKey)){
    aiDriver.memory.visitedTiles.add(tileKey);
    aiDriver.stats.tilesExplored++;
  }
  
  // Mode-specific behavior
  if(aiDriver.mode === 'explore'){
    aiExplore(actor);
  } else if(aiDriver.mode === 'gather'){
    aiGather(actor);
  } else if(aiDriver.mode === 'dig'){
    aiDig(actor);
  } else if(aiDriver.mode === 'hunt'){
    aiHunt(actor);
  }
}
```

---

### 6. Tool System

Four tools with distinct purposes, augmentable through crafting.

**Tools:**
1. **♠ Spade (1)**: Dig walls, gather resources
   - Augmented by: Iron Axe (+1), Pickaxe (+2, mining), Diamond Pickaxe (+4, elite mining)
2. **♣ Club (2)**: Attack creatures
   - Augmented by: Heavy Hammer (+2), Steel Sword (+3), Flame Sword (+5, fire), Legendary Blade (+10)
3. **♥ Heart (3)**: Heal, focus, buff
   - Augmented by: Healing Bandage (+3 HP), Wooden Shield (defense), Life Amulet (+5 max HP)
4. **♦ Diamond (4)**: Special abilities, crafting
   - Augmented by: Portable Workbench (craft anywhere), Elder Wand (+6, magic)

**Usage Pattern:**
```
Select tool → Shift + Direction → Execute action
```

**Code Snippet:**
```javascript
function useTool(dx, dy){
  const tool = state.activeTool;
  const x = state.player.x + dx;
  const y = state.player.y + dy;
  
  if(tool === 1) useSpade(x, y);
  else if(tool === 2) useClub(x, y);
  else if(tool === 3) useHeart();
  else if(tool === 4) useDiamond(x, y);
  
  worldTick();
}

function useSpade(x, y){
  // Check if target is diggable
  const tile = state.map[y][x];
  if(!tile || !tile.dig){
    log('Cannot dig here.');
    return;
  }
  
  // Calculate damage (1 base + inventory bonuses)
  let dmg = 1;
  for(const item of state.inventory){
    if(item.type === 'spade') dmg += item.bonus;
  }
  
  tile.hp -= dmg;
  if(tile.hp <= 0){
    // Wall destroyed, drop resources
    state.map[y][x] = TILES.FLOOR;
    state.resources.ore += 1;
    log(`Dug through ${tile.name}. Gained ore.`);
  }
}
```

---

## 🎯 Key Features

### Enhanced Graphics & Visual Quality ✨
- **Animated Tiles**: Flickering lava, sparkling crystals, flowing water effects
- **Progress Bars**: Visual HP, Stamina, and Hunger bars with color gradients
- **Particle Effects**: Damage numbers, heal effects, and sparkle animations
- **Achievement Notifications**: Pop-up notifications for unlocked achievements
- **Modal Animations**: Smooth fade-in and slide-down effects for menus
- **Rich ASCII Art**: Enhanced tile variety with special characters and colors
- **Biome Variety**: Lava Caverns (Z≤-4) with animated magma, Crystal Caves (Z=-2 to -3)

### Procedural Generation
- **Seeded RNG**: Reproducible worlds via URL parameter `?seed=12345`
- **Cellular Automata**: Organic cave systems
- **Multi-layer Design**: Vertical exploration with stairs
- **Biome Progression**: Forest → Stone Caverns → Crystal Caves → Lava Caverns

### Dynamic World
- **Creature AI**: Scavengers with states (idle, hunting, fleeing, resting)
- **Flora Growth**: Herbs and resources regenerate over time
- **Geology**: Ore deposits slowly regenerate
- **Weather System**: Clear, rain, fog (affects visibility)
- **Day/Night Cycle**: Visual ambiance changes

### Progression Systems
- **Stat Growth**: STR/INT/AGI increase through actions
- **Achievement System**: 8 achievements to unlock (First Blood, Deep Explorer, Master Crafter, etc.)
- **Quest System**: 5 progressive quests with rewards
- **Stats Tracking**: Creatures defeated, items crafted, food gathered, exploration depth
- **Hunger Mechanic**: Must eat food to maintain stamina regeneration
- **Stamina System**: Actions consume energy; passive regeneration every 2 turns (0.3-1.5 based on hunger)
- **Crafting Tiers**: 15+ recipes from basic to legendary
- **Inventory Augments**: Tools become more powerful with crafted items

### Visibility & Lighting
- **Fog of War**: Unexplored tiles are hidden
- **Dynamic Lighting**: Torches extend vision radius
- **Distance Shading**: 10 opacity levels (v0-v9) based on distance
- **Toggle Visibility**: Press `V` to see entire map (debug mode)

---

## 🏆 Achievements & Quests

### Achievements
Lost Poets features 8 unlockable achievements that track your progress:

| Achievement | Icon | Description | Requirement |
|------------|------|-------------|-------------|
| **First Blood** | ⚔ | Defeat your first creature | Defeat any creature |
| **Deep Explorer** | ↓ | Reach Z=-3 or deeper | Descend to depth -3 |
| **Master Crafter** | ⚒ | Craft 10 different items | Craft 10 unique items |
| **Hoarder** | $ | Collect 100 of any resource | Accumulate 100+ of one resource |
| **Survivor** | ⏱ | Survive 500 turns | Reach turn 500 |
| **Legendary** | ⚡ | Craft the Legendary Blade | Craft the ultimate weapon |
| **Forager** | ♠ | Gather 50 food items | Collect 50 food items |
| **Illuminator** | ⚍ | Place 10 torches | Place 10 light sources |

### Quests
Progressive quest system with rewards:

| Quest | Description | Reward |
|-------|-------------|--------|
| **Temple Access** | Craft a key to unlock the temple door | 10 EXP |
| **Tool Up** | Craft your first weapon or tool | 5 EXP |
| **Into the Depths** | Explore the caverns below (Z=-1) | 15 EXP |
| **Rare Discovery** | Find a rare gem (diamond, ruby, or crystal) | 20 EXP |
| **Masterwork** | Craft an advanced or master tier item | 50 EXP |

Achievements display as pop-up notifications in the top-right corner when unlocked, with visual effects and sound feedback.

---

## 🧩 Technical Details

**Architecture:**
- Single-file HTML5 game (~3000 lines)
- Vanilla JavaScript (no frameworks)
- CSS Grid layout with monochrome theme
- Actor-based entity system (future-proof for multiplayer)

**Map Format:**
- 2D tile arrays with properties: `{ch, color, cls, walk, dig, hp, name}`
- Viewport rendering (43x23 visible tiles)
- Camera system tracks player with smooth scrolling

**State Management:**
- Central `state` object with nested structures
- Action queue pattern for turn-based processing
- Layer caching for Z-level persistence

**Performance:**
- Efficient viewport rendering (only visible tiles)
- Scheduled subsystems (avoid every-turn overhead)
- Set-based exploration tracking (O(1) lookups)

---

## 🚀 Future Roadmap

**Planned Features:**
- [ ] Party system (companion actors)
- [ ] Tactical battle instances
- [ ] Quest system with objectives
- [ ] More creature types and behaviors
- [ ] Enchantment system
- [ ] Multiplayer synchronization
- [ ] Save/load functionality
- [ ] Mobile touch controls

**Extension Points:**
- Actor Core Phase 3: Initiative ordering by speed
- Actor Core Phase 4: Tactical battle reuse
- AI Driver: Advanced pathfinding (A*)
- Crafting: Enchantments and modifications

---

## 🛠️ Development

**No build required!** Just open `index.html` in a browser.

**Testing:**
```bash
# Test with specific seed
open index.html?seed=42

# Debug AI
Press 9 to start AI, 0 to stop, Console (F12) for logs
```

**Modding:**
- Edit `RECIPES` object to add new crafting recipes
- Edit `TILES` object to add new tile types
- Edit `RESOURCE_INFO` to add new resources
- Edit `aiDriver` modes for new AI behaviors

---

## 📜 License

Open source — feel free to fork, modify, and share!

---

## 🎨 Credits

**Inspiration:** Dwarf Fortress, Caves of Qud, NetHack  
**Version:** 0.3 (Living World Simulation)  
**Author:** ZHines2
