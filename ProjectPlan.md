# Project Structure

```md
project/
├── common/
│   ├── schemas/
│   ├── content/
│   │   ├── world/
│   │   │   ├── locations.json
│   │   │   ├── npcs.json
│   │   │   ├── items.json
│   │   │   └── factions.json
│   │   ├── gameplay/
│   │   │   ├── classes.json
│   │   │   ├── skills.json
│   │   │   ├── combat-rules.json
│   │   │   ├── status-effects.json
│   │   │   └── crafting.json
│   │   ├── narrative/
│   │   │   ├── quests/
│   │   │   │   ├── main-quests.json
│   │   │   │   └── side-quests.json
│   │   │   ├── dialogues/
│   │   │   │   ├── npc-dialogues.json
│   │   │   │   └── quest-dialogues.json
│   │   │   ├── events.json
│   │   │   └── lore.json
│   │   ├── config/
│   │   │   ├── game-constants.json
│   │   │   ├── progression.json
│   │   │   └── economy.json
│   │   ├── assets/
│   │   │   ├── ascii-art.json
│   │   │   └── sound.json
│   └── tests/
├── go-implementation/
│   ├── cmd/
│   ├── internal/
│   └── pkg/
├── rust-implementation/
│   ├── src/
│   │   ├── engine/
│   │   ├── systems/
│   │   └── utils/
│   └── tests/
├── csharp-implementation/
│   ├── src/
│   │   ├── Engine/
│   │   ├── Systems/
│   │   └── Utils/
│   └── Tests/
└── tools/
```

## Data Structure Examples

### Location Definition (locations.json)

```json
{
  "locations": {
    "town_square": {
      "name": "Town Square",
      "description": "A bustling town square with merchants and travelers.",
      "connections": {
        "north": "market_district",
        "east": "blacksmith",
        "south": "tavern",
        "west": "residential_area"
      },
      "available_actions": [
        "shop",
        "talk",
        "rest"
      ],
      "npcs": ["merchant_joe", "guard_sarah"],
      "requirements": {
        "quest_stage": null,
        "min_level": 1
      },
      "events": {
        "first_visit": "event_welcome_to_town",
        "night": "event_night_market"
      }
    }
  }
}
```

### NPC Definition (npcs.json)

```json
{
  "npcs": {
    "merchant_joe": {
      "name": "Merchant Joe",
      "base_attitude": "friendly",
      "dialogue_trees": {
        "default": "dialogue_joe_default",
        "quest_active": "dialogue_joe_quest"
      },
      "shop_inventory": "inventory_general_store",
      "quest_giver": true,
      "available_quests": ["fetch_supplies", "clear_rats"],
      "state_flags": {
        "met_player": false,
        "quest_completed": false
      }
    }
  }
}
```

### Quest Definition (main-quests.json or side-quests.json)

```json
{
  "quests": {
    "fetch_supplies": {
      "title": "Supply Run",
      "stages": {
        "start": {
          "dialogue": "dialogue_fetch_start",
          "objectives": [
            {
              "type": "collect",
              "item": "healing_herb",
              "amount": 5,
              "locations": ["forest_clearing", "riverside"]
            }
          ],
          "next_stage": "return"
        },
        "return": {
          "dialogue": "dialogue_fetch_complete",
          "objectives": [
            {
              "type": "deliver",
              "npc": "merchant_joe",
              "item": "healing_herb",
              "amount": 5
            }
          ],
          "completion": "quest_complete"
        }
      },
      "rewards": {
        "xp": 100,
        "gold": 50,
        "items": [
          {
            "id": "basic_potion",
            "amount": 2
          }
        ],
        "reputation": {
          "merchant_guild": 5
        }
      }
    }
  }
}
```

### Combat Rules (combat-rules.json)

```json
{
  "combat": {
    "formulas": {
      "physical_damage": {
        "base": "strength * weapon_damage",
        "critical": "base_damage * 1.5",
        "modifiers": ["weapon_type", "armor_class"]
      }
    },
    "status_effects": {
      "poison": {
        "duration": "3_turns",
        "damage_per_turn": "5 + level",
        "cure_conditions": ["antidote", "heal_spell"]
      }
    },
    "turn_order": {
      "priority": ["dexterity", "level"],
      "initiative_formula": "dexterity + d20"
    }
  }
}
```

### Character State

```json
{
  "player": {
    "basic": {
      "name": "string",
      "level": "integer",
      "xp": "integer"
    },
    "stats": {
      "current_hp": "integer",
      "max_hp": "integer",
      "attributes": {}
    },
    "progress": {
      "visited_locations": ["location_id"],
      "known_npcs": ["npc_id"],
      "active_quests": {
        "quest_id": "current_stage"
      },
      "completed_quests": ["quest_id"]
    }
  }
}
```

### Query System

```json
{
  "queries": {
    "available_actions": {
      "check": [
        "location.available_actions",
        "player.abilities",
        "active_quests"
      ],
      "combine": "union"
    },
    "quest_availability": {
      "check": [
        "npc.available_quests",
        "player.level",
        "player.completed_quests"
      ],
      "combine": "filter"
    }
  }
}
```

### Event System

```json
{
  "events": {
    "trigger_types": [
      "location_enter",
      "item_acquire",
      "npc_interact",
      "quest_progress"
    ],
    "actions": {
      "spawn_npc": {
        "params": ["npc_id", "location"]
      },
      "modify_state": {
        "params": ["target", "property", "value"]
      }
    }
  }
}
```

## Implementation Guidelines

1. Content Loading
   - Load all static content at startup
   - Cache frequently accessed data
   - Use lazy loading for dialogue and events

2. State Management
   - Keep current state in memory
   - Save on significant events
   - Track state changes for saving

3. Content Validation
   - Validate all connections are bi-directional
   - Check quest completion paths
   - Verify item availability
   - Ensure all referenced IDs exist

4. Error Handling
   - Provide default values
   - Log invalid references
   - Graceful fallbacks for missing content

## Success Criteria

1. Both implementations can run the same game content
2. Save files are interchangeable between versions
3. Core gameplay mechanics are consistent
4. Performance is comparable between implementations
5. Code is well-documented and maintainable

## Development Guidelines

1. Use consistent coding standards across both languages
2. Implement comprehensive testing
3. Document all shared interfaces
4. Version control all shared content
5. Regular cross-implementation testing

## Risk Management

1. Data format compatibility issues
2. Performance discrepancies
3. Language-specific limitations
4. Testing coverage gaps
5. Content validation challenges
