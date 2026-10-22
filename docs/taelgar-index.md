# `taelgar-index` code block specification

Applies to Taelgarverse **0.5.0**, in Obsidian and on the website.

A `taelgar-index` block generates an index from note metadata at the selected in-world date. The fence name is **`taelgar-index`**.

## Syntax

````markdown
```taelgar-index
type: places
target: Cleenseau
mode: table
show: both
recursive: true
```
````

Only `type` is required. The smallest valid block is:

````markdown
```taelgar-index
type: residents
```
````

## Allowed YAML fields

| Field | Allowed values | Default |
| --- | --- | --- |
| `type` | `residents`, `settlements`, `members`, `rivers`, `places` | Required |
| `target` | A nonempty string containing a note name, alias, or path | The note containing the block |
| `mode` | `grid` or `table` | `grid` |
| `show` | `both`, `obsidian`, or `website` | `both` |
| `recursive` | YAML boolean `true` or `false` | `false` for Members; `true` for all other types |

Field names and enumerated values are case-sensitive. Use lowercase values as shown.

### `type`

| Value | Required target | Included results |
| --- | --- | --- |
| `residents` | A place | Living people whose active home is within the target. Traveling residents remain included; visitors whose home is elsewhere are excluded. |
| `settlements` | A place | Contained place notes with `typeOf: settlement`. Includes destroyed settlements, but excludes those not yet created. |
| `members` | A group or organization | People with an active affiliation or leadership relationship to the organization. |
| `rivers` | A place, typically a watershed | Contained place notes with `typeOf: waterway`, including rivers, brooks, streams, and other waterway subtypes. |
| `places` | A place, typically a town | All contained place notes, including buildings, shops, landmarks, quarters, and settlements. |

Classification uses the existing note tags and metadata. Rivers and Places can target any place; they are not restricted to watersheds and towns.

The target note itself is excluded from its results.

### `target`

Accepts a note name, alias, vault path, or explicit relative path. Examples:

```yaml
target: Cleenseau
```

```yaml
target: "[[Cleenseau]]"
```

```yaml
target: "./Nearby Town.md"
```

Quote wikilinks so YAML treats them as strings.

Explicit `./` and `../` paths resolve relative to the note containing the block. If that note is embedded elsewhere, its own identity remains the default target.

An unresolved, unavailable, or incorrectly classified target produces an error.

### `recursive`

For location indexes:

- `true` includes nested locations. A town index can include a shop inside one of its quarters.
- `false` requires an immediate location match.

Residents start from the person’s **home**. Settlements, Rivers, and Places match current location, last-known location when current location is unknown, and secondary locations. Recursive containment follows current and secondary locations through ancestor places.

For Members:

- `false` includes direct affiliations only.
- `true` also includes affiliations to organizations connected to the target through `partOf`.

Watershed membership must be recorded in location metadata; it is not inferred from prose or folders.

### `show`

- `both`: render in Obsidian and on the website.
- `obsidian`: render only in Obsidian.
- `website`: render only on the website.

The other renderer omits the entire block. Obsidian Source mode retains the editable definition.

## Output

Every inline index starts with an **H2 heading** matching its type: Residents, Settlements, Members, Rivers, or Places.

### Grid mode

Cards contain linked names, relevant metadata, and an available lead image from the result note’s `image` field. Images found only in note bodies are not selected automatically.

Grids show a result count except for Members.

### Table mode

| Type | Columns |
| --- | --- |
| Residents | Person, Info, Current Location, Home |
| Settlements | Place, Region, Type Of, Population |
| Members | Person, Info, Current Location |
| Rivers | River, Type Of |
| Places | Place, Type Of |

Tables use compact rows and have no image column or separate result-count line. Names include available pronouns and pronunciation where applicable.

Sorting is fixed:

- Residents and Members: displayed name.
- Settlements: immediate dated home region, then displayed name.
- Rivers and Places: displayed type, then displayed name.

## Dates and empty results

There is no block-level `date` option. Indexes use the active rendering date: the website’s reader context or Obsidian’s header/calendar date selection.

Relationships and affiliations are evaluated at that date. Website results also follow the reader’s visibility context.

An empty inline index retains its heading and shows a message such as “No places found for this date.”

## Examples

**Compact places table in a town note:**

````markdown
```taelgar-index
type: places
mode: table
```
````

**Waterways in a specified watershed:**

````markdown
```taelgar-index
type: rivers
target: Wistel-Enst Watershed
mode: table
```
````

**Organization members, including subordinate organizations:**

````markdown
```taelgar-index
type: members
recursive: true
```
````

**Direct residents, visible only in Obsidian:**

````markdown
```taelgar-index
type: residents
recursive: false
show: obsidian
```
````

## Validation and unsupported options

The block must contain a YAML mapping. Unknown fields and invalid values produce an “Index unavailable” message.

Use actual booleans:

```yaml
recursive: false
```

Values such as `yes`, `"false"`, or `0` are invalid for `recursive`.

Only the five documented fields are accepted. Custom titles, dates, columns, filters, sorting, limits, and count controls are not currently supported.

## Related: header popup buttons

Header buttons are configured separately in the note’s frontmatter:

```yaml
indexes: [residents, settlements, rivers, places]
```

An organization would normally use:

```yaml
indexes: [members]
```

Buttons require a `taelgar-header` block. Empty buttons are hidden, and popup contents calculate when clicked. Popups always use grids; the inline block’s options do not configure them.