---
name: cozi
description: Manage your Cozi family calendar and lists — add/edit/remove appointments, shopping lists, and to-do items.
version: 1.0.0
homepage: https://github.com/Wetzel402/py-cozi
user-invocable: true
metadata: {"openclaw":{"emoji":"📅","os":["darwin","linux","win32"],"requires":{"bins":["python3","pip3"],"env":["COZI_USERNAME","COZI_PASSWORD"]},"primaryEnv":"COZI_USERNAME"}}
---

# Cozi Family Organizer

You can manage a family's Cozi account — calendar appointments, shopping lists, and to-do lists.

## Setup

This skill requires the `py-cozi` Python package. Install and patch it (the published version is missing a required API key fix):

```bash
pip3 install py-cozi
python3 -c "
import cozi, pathlib
p = pathlib.Path(cozi.__file__)
src = p.read_text()
if 'apikey=' not in src:
    p.write_text(src.replace(
        '/api/ext/2207/auth/login',
        '/api/ext/2207/auth/login?apikey=coziwc|v251_production'
    ))
    print('Patched py-cozi with API key fix')
else:
    print('py-cozi already patched')
"
```

Credentials are provided via environment variables:
- `COZI_USERNAME` — the Cozi account email
- `COZI_PASSWORD` — the Cozi account password

## How to Use

Run Python scripts using `bash` to interact with the Cozi API. All calls are async, so use `asyncio.run()`.

### Important: Use a Single Async Context

The py-cozi library has bugs in its retry logic (missing `await` on retry paths). To avoid issues, always run all operations inside a single `async def main()` function — never use multiple separate `asyncio.run()` calls.

```python
import asyncio, json, os, sys, aiohttp
from cozi import Cozi

async def main():
    cozi = Cozi(os.environ["COZI_USERNAME"], os.environ["COZI_PASSWORD"])
    await cozi.login()

    # ... all operations go here using await ...

asyncio.run(main())
```

### Available Operations

#### Lists

- **Get all lists**: `asyncio.run(cozi.get_lists())` — returns all shopping and to-do lists with their items.
- **Add a list**: `asyncio.run(cozi.add_list("Groceries", "shopping"))` — type is `"shopping"` or `"todo"`.
- **Remove a list**: `asyncio.run(cozi.remove_list(list_id))`
- **Reorder list items**: `asyncio.run(cozi.reorder_list(list_id, "List Title", items_list, "shopping"))`

#### List Items

- **Add item**: `asyncio.run(cozi.add_item(list_id, "Milk", 0))` — position is the index to insert at (0 = top).
- **Edit item**: `asyncio.run(cozi.edit_item(list_id, item_id, "Almond Milk"))`
- **Mark item**: `asyncio.run(cozi.mark_item(list_id, item_id, "complete"))` — status is `"complete"` or `"incomplete"`.
- **Remove items**: `asyncio.run(cozi.remove_items(list_id, [item1, item2]))` — takes a list of item objects.

#### Calendar

- **Get calendar**: `await cozi.get_calendar(2026, 3)` — pass year and month as integers. Returns `{'days': {...}, 'items': {...}, 'startDate': ..., 'endDate': ...}`. The `items` dict is keyed by appointment ID and contains full details (subject, location, times, attendees, etc.).
- **Add appointment**: Due to bugs in py-cozi's `add_appointment`, build the payload and call the API directly:
  ```python
  data = [{
      'itemType': 'appointment',
      'create': {
          'startDay': '2026-03-20',  # YYYY-MM-DD
          'details': {
              'startTime': '14:00',   # 24h format, or None for all-day
              'endTime': '15:00',     # 24h format, or None for all-day
              'dateSpan': 1,          # number of days
              'attendeeSet': [person_id1, person_id2],  # accountPersonId values from get_persons()
              'location': 'Dentist Office',
              'notes': '',
              'subject': 'Dentist Appointment'
          }
      }
  }]
  url = f'https://rest.cozi.com/api/ext/2004/{cozi._acct_id}/calendar/{year}/{month}'
  session = aiohttp.ClientSession(headers=cozi._headers)
  async with session.post(url, json=data) as resp:
      result = await resp.json()
  await session.close()
  ```
  **IMPORTANT:** `attendeeSet` requires `accountPersonId` UUIDs from `get_persons()`, NOT names.
- **Edit appointment**: Same direct approach, using `'edit'` key with an `'id'` field instead of `'create'`.
- **Remove appointment**: Same direct approach, using `'delete'` key: `[{'itemType': 'appointment', 'delete': {'id': appt_id}}]`

#### People

- **Get persons**: `asyncio.run(cozi.get_persons())` — returns all family members on the account.

## Workflow

1. Always call `login()` first.
2. When the user asks to see their lists or calendar, fetch the data and present it in a readable format.
3. When adding items to a list, first call `get_lists()` to find the correct `list_id` by matching the list name.
4. When working with calendar events, first call `get_calendar(year, month)` to find appointment IDs.
5. When adding appointments, call `get_persons()` first to map family member names to their `accountPersonId` UUIDs for the `attendeeSet`.
6. Present results clearly — format lists as checklists and calendar events as a readable schedule.

## Guardrails

- Never store or log the user's Cozi credentials.
- Confirm with the user before deleting lists or removing appointments.
- When removing items, show what will be removed and ask for confirmation.
- If login fails, tell the user to check their COZI_USERNAME and COZI_PASSWORD settings.

## Error Handling

- `InvalidLoginException` — credentials are wrong. Ask the user to verify their settings.
- `RequestException` — network issue. Suggest retrying.
- `CoziException` — general API error. Show the error message to the user.

## Examples

**User:** "What's on my shopping list?"
**Action:** Run `get_lists()`, find lists with type `"shopping"`, and display their items.

**User:** "Add eggs and butter to my grocery list"
**Action:** Run `get_lists()` to find the grocery list ID, then call `add_item()` for each item.

**User:** "What's on the calendar this month?"
**Action:** Run `get_calendar(2026, 3)` and display appointments in chronological order.

**User:** "Schedule a dentist appointment for March 20th at 2pm"
**Action:** Run `add_appointment(2026, 3, 20, "14:00", "15:00", 1, [], "Dentist Office", "", "Dentist Appointment")`.

**User:** "Mark milk as done on my shopping list"
**Action:** Run `get_lists()` to find the item ID for "milk", then call `mark_item()` with status `"complete"`.
