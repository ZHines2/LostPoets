# Lost Poets - GitHub Copilot Instructions

## Project Overview

Lost Poets is a single-file HTML5 browser-based ASCII roguelike game inspired by Dwarf Fortress. The entire game is contained in `index.html` with embedded CSS and JavaScript (no build process required).

**Key characteristics:**
- Browser-based game with no external dependencies
- Vanilla JavaScript (no frameworks or libraries)
- Procedural generation with seeded randomness
- Actor-based architecture for future multiplayer support
- Living world simulation with autonomous AI

## Code Style & Conventions

### JavaScript Style
- Use compact, concise function syntax where appropriate
- Prefer single-statement function bodies: `function rf(){ return (rand()-1)/2147483646; }`
- Use clear section headers with equals signs: `// ========== SECTION NAME ==========`
- Keep functions focused and small
- Use descriptive variable names for clarity
- Inline comments for complex game logic

### Naming Conventions
- **Constants**: Use ALL_CAPS with underscores (e.g., `TILES`, `RECIPES`)
- **Functions**: Use camelCase (e.g., `worldTick`, `simulateCreatures`)
- **Variables**: Use camelCase (e.g., `state`, `actorCore`)
- **Private/internal**: Prefix with underscore (e.g., `_seed`)

### State Management
- All game state lives in the central `state` object
- Use nested structures for organization (e.g., `state.player`, `state.region`)
- Avoid global variables except for RNG seed and actor core

### Performance Considerations
- Use viewport rendering (only render visible tiles)
- Schedule heavy simulations at appropriate intervals (e.g., creature AI every 10 turns, not every turn)
- Use efficient data structures (Sets for O(1) lookups, cached maps for Z-levels)

## Architecture Guidelines

### Actor Core System
- All entities should eventually use the unified actor system
- Actor structure: `{id, type, x, y, hp, hpMax, str, int, agi, stamina, stamMax, facing, memory}`
- Use action queue pattern: `queueAction(actorId, {type, params})`
- Keep legacy `state.player` in sync for UI compatibility

### World Simulation
- Run different simulations at different frequencies:
  - Stamina regeneration: every 2 turns
  - Hunger decay: every 5 turns
  - Creature AI: every 10 turns
  - Flora growth: every 12 turns
  - Geology: every 15 turns
  - Creature spawning: every 30 turns
  - Random events: every 50 turns
  - Day/night cycle: every 100 turns
  - Weather: every 150 turns

### Map & Tile System
- Tiles are objects with properties: `{ch, color, cls, walk, dig, hp, name}`
- Use 2D arrays for maps: `map[y][x]`
- Z-levels are cached in `state.zLevels` object
- Always check bounds: `inBounds(x, y)`

### Crafting System
- Recipes organized by tier: basic, intermediate, advanced, master
- Recipe structure: `{name, inputs: {resource: amount}, makeItem, onCraft}`
- Check requirements with `canCraftRecipe(recipe)`
- Unlock recipes through discovery and progression

## Modding & Extension Points

When adding new features:
- **New recipes**: Add to `RECIPES` object with appropriate tier
- **New tiles**: Add to `TILES` object with properties
- **New resources**: Add to `RESOURCE_INFO` and initialize in `state.resources`
- **New AI behaviors**: Add modes to `aiDriver` function
- **New crafting tiers**: Follow existing tier patterns

## Testing & Debugging

### Manual Testing
- Open `index.html` directly in browser (no build required)
- Test with specific seed: `index.html?seed=42`
- Use browser console (F12) for debug output

### AI Testing
- Press `[` to toggle AI on/off
- Press `]` to show AI statistics
- Console commands: `startAI('mode', speed)`, `stopAI()`, `aiSetMode('mode')`, `aiStats()`
- AI modes: 'explore', 'gather', 'dig', 'hunt', 'idle'

### Debug Features
- Press `V` to toggle visibility/fog of war
- Press `L` to teleport to Secret Lab
- Use console logging for complex systems

## Security & Best Practices

- No external HTTP requests or API calls
- All game logic runs client-side
- RNG is deterministic and reproducible with seeds
- No user input sanitization needed (keyboard-only input)

## Comments & Documentation

- Use inline comments to explain complex game mechanics
- Add section headers for major code blocks
- Document future extension points (Phase 1, Phase 2, etc.)
- Keep README.md in sync with major feature changes

## Future Roadmap Awareness

Be aware of planned features when making changes:
- **Actor Core Phase 3**: Initiative ordering by speed
- **Actor Core Phase 4**: Tactical battle reuse
- **Multiplayer**: Network synchronization support
- **Save/Load**: Serialization support
- **Mobile**: Touch control support

When implementing new features, consider compatibility with these future enhancements.

## Common Patterns

### Adding a new tile type
```javascript
// In TILES object
NEWTILE: {ch:'?', color:'#abc', cls:'newtile', walk:true, dig:false, name:'New Tile'}
```

### Adding a new recipe
```javascript
// In RECIPES object
newItem: {
  name: 'New Item',
  tier: 'basic',
  inputs: {wood: 2, ore: 1},
  makeItem: () => ({name:'New Item', type:'tool', bonus:5, special:'effect'})
}
```

### Scheduling a new simulation
```javascript
// In worldTick() function
if(state.turns % 20 === 0) simulateNewSystem();
```

### Adding a new keyboard command
```javascript
// In keydown event listener
if(key==='N') {
  // Handle new command
  worldTick();
}
```
