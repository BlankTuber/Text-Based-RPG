# Cross-Language Text RPG Game Content Structure Design

## 1. Core Data Files Structure

```md
game-content/
├── world/
│   ├── locations.json       # All game locations and connections
│   ├── npcs.json            # NPC definitions
│   └── items.json           # Item definitions
├── gameplay/
│   ├── classes.json         # Character class definitions
│   ├── skills.json          # Skill definitions
│   └── combat-rules.json    # Combat formulas and rules
├── narrative/
│   ├── quests/
│   │   ├── main-quests.json
│   │   └── side-quests.json
│   ├── dialogues/
│   │   ├── npc-dialogues.json
│   │   └── quest-dialogues.json
│   └── events.json          # World events and triggers
└── config/
    ├── game-constants.json  # Global game settings
    └── progression.json     # Level/XP requirements
```

## 2. Data Structure Examples

### Location Definition

``` json
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

### NPC Definition

``` json
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

### Quest Definition

``` json
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

### Combat Rules

``` json
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

## 3. State Tracking System

### Character State

``` json
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

## 4. Implementation Guidelines

1. **Content Loading**
   - Load all static content at startup
   - Cache frequently accessed data
   - Use lazy loading for dialogue and events

2. **State Management**
   - Keep current state in memory
   - Save on significant events
   - Track state changes for saving

3. **Content Validation**
   - Validate all connections are bi-directional
   - Check quest completion paths
   - Verify item availability
   - Ensure all referenced IDs exist

4. **Error Handling**
   - Provide default values
   - Log invalid references
   - Graceful fallbacks for missing content

## 5. Query System

``` json
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

## 6. Event System

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

## 7. Technical Architecture

```md
project/
├── common/
│   ├── schemas/     # JSON schemas for game data
│   ├── content/     # Shared game content
│   └── tests/       # Common test scenarios
├── go-implementation/
│   ├── cmd/         # Entry points
│   ├── internal/    # Game engine implementation
│   └── pkg/         # Reusable packages
├── rust-implementation/
│   ├── src/
│   │   ├── engine/  # Core game logic
│   │   ├── systems/ # Game subsystems
│   │   └── utils/   # Utility functions
│   └── tests/
└── tools/           # Shared development tools
```
