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

Install the [patched fork of py-cozi](https://github.com/eski009/py-cozi) (fixes several bugs in the upstream version):

```bash
pip3 install git+https://github.com/eski009/py-cozi.git
```

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

This skill wraps [py-cozi](https://github.com/Wetzel402/py-cozi), an unofficial Python wrapper for the Cozi API originally created by [Cody Wetzel](https://github.com/Wetzel402). This project uses a [patched fork](https://github.com/eski009/py-cozi) that fixes API key authentication, missing awaits in retry logic, and bugs in `edit_appointment`.

## License

MIT
