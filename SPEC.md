# Cabworld — Spec

Taxi simulation on the GridWorld framework, for TUM's OOP for Transport Engineers coursework (Assignment 5).

This spec is the contract. It defines *what* must be true when the implementation is done — not the code itself. When implementing, treat this as the source of intent; if something here is ambiguous, ask before guessing.

## Problem statement

Build a program that moves four independent taxi agents around a shared 10×10 grid, over discrete time steps, until each has completed its own ordered list of pickup-and-dropoff jobs — visually animated and reported to console.

## Pipeline (execution order)

1. **Generate** — create 200 requests, write to `./input/taxi_requests.csv`
2. **Read** — load that CSV back into `Request` objects
3. **Assemble** — build the grid, cities, taxis; assign requests to their taxis
4. **Simulate** — step the world; taxis move, pick up, drop off, report, self-remove
5. **Report** — console output of total steps per taxi as each finishes

Each phase only depends on the *output* of the previous phase, not its internals. `RequestReader` doesn't need to know how the CSV was generated; `SimpleTaxi` doesn't need to know the CSV exists at all.

## Data contract: `taxi_requests.csv`

| column | type | constraint |
|---|---|---|
| `request_order_id` | int | starts at 1, increments |
| `request_taxi_id` | int | randomly assigned |
| `origin_city_id` | str | one of A/B/C/D |
| `destination_city_id` | str | one of A/B/C/D, must differ from origin |

- Exactly 200 rows.
- All randomness drawn from a single `Random` instance seeded with the matriculation number.
- Written and read via **relative** paths only (`./input/taxi_requests.csv`).

## Class contracts

### `City` (is a `Rock`)
- **Knows:** `id`, `location`
- **Does:** nothing — immovable, just occupies a grid cell
- **Acceptance:** four instances exist, one per corner (NW=A, NE=B, SW=C, SE=D)

### `Request` (plain data, not an `Actor`, not visually displayed)
- **Knows:** `order_id`, `taxi_id`, `origin_city_id`, `destination_city_id`
- **Does:** nothing
- **Acceptance:** never rendered in the grid world

### `Taxi` (interface)
- **Declares:** `is_occupied() -> bool`, `add_request(request: Request) -> None`
- **Acceptance:** no shared implementation — a pure contract that `SimpleTaxi` and (bonus) `SmartTaxi` both fulfill, so calling code can treat any taxi type identically

### `SimpleTaxi` (is an `Actor`, implements `Taxi`)
- **Knows:** `id`, `steps`, `occupied`, `bookings` (sorted list of `Request`)
- **Does:**
  - `add_request()` inserts into `bookings` in sorted order — only entry point to the list, nothing outside touches it directly
  - drives one cell at a time, straight line only
  - if blocked (grid edge, another taxi, a city) → turns 45° right, retries next step
  - when adjacent to the correct city for the current request's pickup/dropoff → toggles `occupied`, changes color, consumes the whole time step (no movement that step)
  - on dropoff, removes the fulfilled request from `bookings`
  - once `bookings` is empty → prints total `steps` to console, removes self from grid
- **Acceptance:** requests are served strictly in the order they were added

### `RequestGeneratorAndWriter`
- **Does:** generates 200 requests, writes `taxi_requests.csv` per the data contract above
- **Non-goal:** does not know taxis or cities exist as objects — only IDs

### `RequestReader`
- **Does:** static `read_requests(file_name) -> list[Request]`, reads from relative path, handles IOErrors
- **Non-goal:** does not know how the file was generated

### `SimpleTaxiWorld` (is an `ActorWorldWithGUI`)
- **Does**, in `set_scenario()`:
  1. create 10×10 grid
  2. call `RequestReader.read_requests()` → `list_rq`
  3. create four `City` instances at the corners, store in a container
  4. create four `SimpleTaxi` instances, store in a key-value container keyed by `id`
  5. loop `list_rq`, assign each request to its taxi via `add_request()`
  6. add all `City` and `SimpleTaxi` objects to the world, taxis placed near grid center

### `SmartTaxi` (bonus, implements `Taxi`)
- Same contract as `SimpleTaxi`, smarter movement (e.g. using `Location.get_direction_toward()`)
- **Acceptance:** must complete the same requests in fewer total steps than `SimpleTaxi`

## Non-negotiable constraints

- Random seed = matriculation number, one shared `Random` instance for all randomness
- Relative paths only, never absolute
- `bookings` stays sorted and is only mutated through `add_request()`
- Occupancy color change costs one full time step with no movement

## GridWorld framework API (confirmed from source)

This is infrastructure the course provides — not part of what you design. Reference only.

**`Actor`** (base class for anything in the grid)
- Constructor: `Actor(grid, location, direction=Location.NORTH, color=DEFAULT_COLOR)`
- Properties: `.grid`, `.color`, `.direction`, `.location` (read-only location — changes only via movement methods)
- `put_self_in_grid(grid, loc)` — places the actor; bumps whatever was already there
- `remove_self_from_grid()` — takes the actor off the grid
- `move_to(new_location)` — moves the actor; raises if `new_location` isn't valid in the grid
- `act()` — override this; called once per actor per world step

**`Rock`** (extends `Actor`)
- `act()` does nothing — confirms `City` should extend this, since cities are immovable

**`Location`**
- Compass constants (multiples of 45°): `NORTH=0, NORTHEAST=45, EAST=90, SOUTHEAST=135, SOUTH=180, SOUTHWEST=225, WEST=270, NORTHWEST=315`
- Turn constants: `LEFT=-90, RIGHT=90, HALF_LEFT=-45, HALF_RIGHT=45`
- `get_adjacent_location(direction)` — returns the neighboring `Location` in that compass direction
- `get_direction_toward(target)` — returns the compass direction from self toward another location, rounded to the nearest 45°
- `.row`, `.col` accessors

**`Grid`** (abstract; `BoundedGrid` is the concrete 10×10 implementation)
- `get(loc)` / `put(loc, actor)` / `remove(loc)` — direct grid access
- `is_valid(loc)` — bounds check
- `get_neighbors(loc)` — returns the **actors** occupying adjacent cells (this is the "hint" method the assignment doc references for checking who's next to you)
- `get_valid_adjacent_locations(loc)`, `get_empty_adjacent_locations(loc)`, `get_occupied_adjacent_locations(loc)`, `get_occupied_locations()`

**World (`ActorWorldWithGUI` / base `ActorWorld`)**
- Subclasses override `set_scenario()` to build the initial layout
- `.add(actor, loc)` — places an actor in the world's grid
- `.step()` — each world step, calls `act()` once on every actor currently occupying a grid location, in the order returned by `get_occupied_locations()`

## Still open — genuinely your design decision, not resolved by the framework

- **Data container types for cities/taxis.** The assignment explicitly asks for "appropriate data structures... especially regarding sorted and unsorted." The framework doesn't dictate this — `City`/`SimpleTaxi` storage choice (dict keyed by id vs. list vs. something ordered) is still yours to decide.
- **The actual movement/turn/occupancy logic inside `SimpleTaxi.act()`.** The framework gives you the primitives (`get_adjacent_location`, `is_valid`, `get_neighbors`) — how you compose them into "drive straight, turn 45° when blocked, toggle occupancy near the right city" is the actual assignment, not something the framework hands you.
