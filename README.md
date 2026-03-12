# Cozi OpenClaw Skill

An [OpenClaw](https://github.com/openclaw/openclaw) skill for managing your [Cozi](https://www.cozi.com) family calendar and lists.

## Features

- View and manage shopping lists and to-do lists
- Add, edit, and remove list items
- View calendar appointments
- Create, edit, and delete calendar events with attendees
- Look up family members on the account

## Installation

Copy the `skills/cozi` directory into your OpenClaw skills folder:

```bash
cp -r skills/cozi ~/.openclaw/skills/cozi
```

### Dependencies

Install the [py-cozi](https://github.com/Wetzel402/py-cozi) Python package:

```bash
pip3 install py-cozi
```

> **Note:** The published version of py-cozi requires a patch for Cozi's updated API. The skill's setup instructions include an auto-patching script — see `skills/cozi/SKILL.md` for details.

### Configuration

Add your Cozi credentials to your OpenClaw config (`~/.openclaw/openclaw.json`):

```json
{
  "skills": {
    "entries": {
      "cozi": {
        "enabled": true,
        "env": {
          "COZI_USERNAME": "your-cozi-email@example.com",
          "COZI_PASSWORD": "your-cozi-password"
        }
      }
    }
  }
}
```

## Credits

This skill wraps [py-cozi](https://github.com/Wetzel402/py-cozi), an unofficial Python wrapper for the Cozi API by [Cody Wetzel](https://github.com/Wetzel402).

## License

MIT
