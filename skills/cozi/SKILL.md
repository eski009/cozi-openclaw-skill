---
name: cozi
description: Manage your Cozi family calendar and lists — add/edit/remove appointments, shopping lists, and to-do items.
version: 1.1.0
homepage: https://github.com/eski009/py-cozi
user-invocable: true
metadata: {"openclaw":{"emoji":"📅","os":["darwin","linux","win32"],"requires":{"bins":["python3","pip3"],"env":["COZI_USERNAME","COZI_PASSWORD"]},"primaryEnv":"COZI_USERNAME"}}
---

# Cozi Family Organizer

You can manage a family's Cozi account — calendar appointments, shopping lists, and to-do lists.

## Setup

This skill requires a patched version of the `py-cozi` Python package. Install from the fork (the upstream version has several bugs):

```bash
pip3 install git+https://github.com/eski009/py-cozi.git
```

Credentials are provided via environment variables:
- `COZI_USERNAME` — the Cozi account email
- `COZI_PASSWORD` — the Cozi account password

## How to Use

Run Python scripts using `bash` to interact with the Cozi API. All calls are async — run everything inside a single `async def main()` function.

```python
import asyncio, json, os
from cozi import Cozi

async def main():
    cozi = Cozi(os.environ["COZI_USERNAME"], os.environ["COZI_PASSWORD"])
    await cozi.login()

    # ... all operations go here using await ...

asyncio.run(main())
```

### Available Operations

#### Lists

- **Get all lists**: `await cozi.get_lists()` — returns all shopping and to-do lists with their items.
- **Add a list**: `await cozi.add_list("Groceries", "shopping")` — type is `"shopping"` or `"todo"`.
- **Remove a list**: `await cozi.remove_list(list_id)`
- **Reorder list items**: `await cozi.reorder_list(list_id, "List Title", items_list, "shopping")`

#### List Items

- **Add item**: `await cozi.add_item(list_id, "Milk", 0)` — position is the index to insert at (0 = top).
- **Edit item**: `await cozi.edit_item(list_id, item_id, "Almond Milk")`
- **Mark item**: `await cozi.mark_item(list_id, item_id, "complete")` — status is `"complete"` or `"incomplete"`.
- **Remove items**: `await cozi.remove_items(list_id, [item_id1, item_id2])` — takes a list of item ID strings.

#### Calendar

- **Get calendar**: `await cozi.get_calendar(2026, 3)` — pass year and month as integers. Returns `{'days': {...}, 'items': {...}, 'startDate': ..., 'endDate': ...}`. The `items` dict is keyed by appointment ID and contains full details (subject, location, times, attendees, etc.).
- **Add appointment**: `await cozi.add_appointment(year, month, day, start, end, date_span, attendees, location, notes, subject)`
  - `start` and `end` are 24-hour time strings like `"14:00"`, or `None` for all-day events.
  - `date_span` is the number of days the event spans (usually `1`).
  - `attendees` is a list of `accountPersonId` UUIDs from `get_persons()`, **NOT names**.
  - `location`, `notes`, `subject` are strings.
- **Edit appointment**: `await cozi.edit_appointment(appt_id, year, month, day, start, end, date_span, attendees, location, notes, subject)`
- **Remove appointment**: `await cozi.remove_appointment(year, month, appt_id)`

#### People

- **Get persons**: `await cozi.get_persons()` — returns all family members on the account. Each person has a `name` and `accountPersonId` field. Use the `accountPersonId` UUIDs when setting attendees on appointments.

## Workflow

1. Always call `login()` first.
2. When the user asks to see their lists or calendar, fetch the data and present it in a readable format.
3. When adding items to a list, first call `get_lists()` to find the correct `list_id` by matching the list name.
4. When working with calendar events, first call `get_calendar(year, month)` to find appointment IDs.
5. When adding appointments, call `get_persons()` first to map family member names to their `accountPersonId` UUIDs for the `attendees` list.
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
**Action:** Run `get_persons()` if attendees are needed, then call `add_appointment(2026, 3, 20, "14:00", "15:00", 1, [], "Dentist Office", "", "Dentist Appointment")`.

**User:** "Mark milk as done on my shopping list"
**Action:** Run `get_lists()` to find the item ID for "milk", then call `mark_item()` with status `"complete"`.
