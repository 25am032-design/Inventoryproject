# Inventory Engine — Core Algorithms, OOP Structures & Robust Error Handling

An object-oriented inventory management engine in pure Python (no runtime dependencies) that demonstrates **inheritance, encapsulation, polymorphism, custom exceptions, JSON/CSV persistence and unit testing** (99% coverage, 55 tests).

## Requirements covered

| Requirement | Where |
|-------------|-------|
| Class hierarchy modelling a real-world system | `models.py` — `Item` (abstract) → `StandardItem`, `PerishableItem`, `ElectronicItem`; `inventory.py` — `Inventory` |
| Custom exception classes for invalid input | `exceptions.py` — `InventoryError` and five subclasses |
| Clean file I/O with JSON and CSV | `storage.py` — `save_json/load_json`, `save_csv/load_csv` |
| Unit tests, ≥ 80% coverage | `tests/` — pytest + pytest-cov (enforced at 80%, currently ~99%) |

## OOP concepts

- **Encapsulation** – `sku`, `name`, `unit_price` and `quantity` are stored in private attributes and exposed through validated properties. Invalid values can never enter an object. SKUs are normalised (trimmed, upper-cased).
- **Inheritance** – `Item` is an abstract base class. Subclasses add their own state (`expiry_date`, `warranty_months`) and reuse the base validation and stock logic.
- **Polymorphism** – `total_value()`, `describe()` and `to_dict()` are overridden. `Inventory.total_value()` just calls `item.total_value()` on each item without knowing the type:
  - `StandardItem` / `ElectronicItem`: price × quantity
  - `PerishableItem`: 0 if expired, 50% off within 7 days of expiry, otherwise full value
- **Factory** – `Item.from_dict()` picks the right subclass from the record's `type`.

## Exceptions

```
InventoryError (base)
├── InvalidItemError      bad sku/name/price/quantity/date (also a ValueError)
├── DuplicateItemError    SKU already exists
├── ItemNotFoundError     SKU not found (also a KeyError)
├── InsufficientStockError  carries .sku, .requested, .available
└── StorageError          unreadable/unwritable/malformed JSON or CSV
```

Catch `InventoryError` to handle everything, or a specific subclass for finer control. Storage errors include file name and record/line number, e.g. `inv.json: record 2: unit_price must be a non-negative number, got -5`.

## Installation

```bash
git clone <your-repo-url> inventory-engine
cd inventory-engine
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -e ".[dev]"
```

## Usage

```python
from datetime import date
from inventory_engine import (Inventory, StandardItem, PerishableItem,
                              ElectronicItem, InventoryError, storage)

inv = Inventory()
inv.add_item(StandardItem("PEN-01", "Blue Pen", 1.50, 120))
inv.add_item(ElectronicItem("LAP-01", "Laptop", 799.99, 3, warranty_months=24))
inv.add_item(PerishableItem("MLK-01", "Milk 1L", 1.20, 40, "2026-11-22"))

try:
    inv.sell("LAP-01", 10)
except InventoryError as exc:
    print(exc)      # insufficient stock for 'LAP-01': requested 10, available 3

print(inv.total_value(as_of=date(2026, 11, 19)))
storage.save_json(inv, "inventory.json")
storage.save_csv(inv, "inventory.csv")
inv2 = storage.load_json("inventory.json")
```

Run the full demo: `python examples/demo.py` (output in `examples/demo_output.txt`).

### Inventory API

`add_item`, `get`, `remove_item`, `restock`, `sell`, `total_value(as_of)`, `low_stock(threshold)`, `expired(as_of)`, `search(text)`, `report(as_of)`, plus `len()`, `in` and iteration (sorted by SKU).

### File formats

JSON: `{"items": [{"type": "perishable", "sku": "MLK-01", "name": "Milk 1L", "unit_price": 1.2, "quantity": 40, "expiry_date": "2026-11-22"}, ...]}`

CSV header: `type,sku,name,unit_price,quantity,expiry_date,warranty_months`

## Tests

```bash
pytest
```

Coverage is measured automatically and the run fails if it drops below 80%.

| Module | Coverage |
|--------|----------|
| exceptions.py | 100% |
| inventory.py | 100% |
| models.py | 100% |
| storage.py | 99% |
| **Total** | **99.65%** |

Tests cover validation of every field, stock errors, polymorphic valuation, factory round-trips, JSON/CSV round-trips (including commas in CSV fields), and failure paths: missing files, empty files, malformed JSON/CSV, wrong structure, bad numbers, unknown types and duplicate SKUs.

## Project layout

```
inventory-engine/
├── pyproject.toml
├── README.md
├── src/inventory_engine/   exceptions.py  models.py  inventory.py  storage.py
├── tests/                  test_models.py  test_inventory.py  test_storage.py
├── examples/               demo.py  demo_output.txt  data/
└── docs/                   README.pdf  test-report.pdf
```

## License

MIT
