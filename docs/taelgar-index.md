# `taelgar-index` code block specification

Applies to Taelgarverse **0.5.4**, in Obsidian and on the website.

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
| `placeType` | Places only: `built` or `natural` | Omitted: include all place types |

Field names and enumerated values are case-sensitive. Use lowercase values as shown.

### `type`

| Value | Required target | Included results |
| --- | --- | --- |
| `residents` | A place | Living people whose active home is within the target. Traveling residents remain included; visitors whose home is elsewhere are excluded. |
| `settlements` | A place | Contained place notes with `typeOf: settlement`. Includes destroyed settlements, but excludes those not yet created. |
| `members` | A group or organization | People with an active affiliation or leadership relationship to the organization. |
| `rivers` | A place, typically a watershed | Contained place notes with `typeOf: waterway` or `typeOf: lake`, including rivers, brooks, streams, and lakes. Marine features are excluded. |
| `places` | A place, typically a town | Contained place notes, including buildings, neighborhoods, settlements, and natural features. Optional `placeType` filters select built or natural places. |

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

### `placeType`

Supported only for Places. Use `placeType: built` or `placeType: natural`; omit the field for all place types. Classification uses the note's `typeOf`, not its name or `typeOfAlias`.

| Value | Included `typeOf` values |
| --- | --- |
| `built` | `settlement`, `neighborhood`, `realm`, `inn`, `building`, `holy site`, `road`, `infrastructure`, `battlefield` |
| `natural` | `region`, `watershed`, `forest`, `wetlands`, `desert`, `grassland`, `waterway`, `lake`, `marine feature`, `topographical feature`, `subterranean feature`, `island` |

The legacy spelling `topographic feature` also matches `natural`. Extraplanar, missing, and unrecognized types appear in unfiltered Places but neither filtered category. A waterway displayed as a canal still matches `natural` because its `typeOf` is `waterway`.

Filtering selects results, while containment follows all ancestor types. A building inside a forest can appear in recursive built results; natural features inside a neighborhood can appear in recursive natural results. Date, visibility, and `recursive` rules still apply. The filter does not change note metadata.

### `show`

- `both`: render in Obsidian and on the website.
- `obsidian`: render only in Obsidian.
- `website`: render only on the website.

The other renderer omits the entire block. Obsidian Source mode retains the editable definition.

## Output

Every inline index starts with an **H2 heading** matching its type: Residents, Settlements, Members, Rivers, or Places.

### Grid mode

Cards contain linked names, relevant metadata, and an available lead image from the result note’s `image` field. Images found only in note bodies are not selected automatically.

People cards include available pronouns, pronunciation, and ancestry/type information. Residents show Current Location and Home. Members show roles and the first authored prose sentence instead of locations. Sentence extraction skips headings, callouts and other blockquotes, code, lists, tables, and image-only paragraphs. Wikilink aliases and Markdown link labels become plain text. Descriptions respect date/campaign comment conditions; website descriptions also use the reader's visibility context.

Settlement cards include available pronunciation and population. River and place cards show the formatted type. Grids show a result count except for Members.

### Table mode

| Type | Columns |
| --- | --- |
| Residents | Person, Info, Current Location, Home |
| Settlements | Place, Region, Type Of, Population |
| Members | Person, Info, Current Location, Home |
| Rivers | River, Type Of |
| Places | Place, Type Of |

Tables use compact rows and have no image column or separate result-count line. Person includes available pronouns and pronunciation. Info uses ancestry and main type; Current Location uses the last-known location chain and its date, while Home uses the selected date's home. Missing locations show **Unknown**. Settlement, river, and place names also include pronunciation when available.

Sorting is fixed:

- Residents and Members: displayed name.
- Settlements: immediate dated home region, then displayed name.
- Rivers and Places: displayed type, then displayed name.

## Dates and empty results

There is no block-level `date` option. Indexes use the active rendering date: the website’s reader context or Obsidian’s header/calendar date selection.

Relationships and affiliations are evaluated at that date. Member affiliations use inclusive start/end dates. An omitted end defaults to the person's death date when known; explicit affiliation dates are honored, without a separate living-person filter. Rivers and Places exclude notes not yet created but retain destroyed places. Website results also follow the reader’s publication, campaign, and POV visibility context. Obsidian uses local author access.

An empty inline index retains its heading and shows a message such as “No places found for this date.”

## Examples

**Compact places table in a town note:**

````markdown
```taelgar-index
type: places
mode: table
```
````

**Waterways and lakes in a specified watershed:**

````markdown
```taelgar-index
type: rivers
target: Wistel-Enst Watershed
mode: table
```
````

**Built places only:**

````markdown
```taelgar-index
type: places
placeType: built
mode: table
```
````

Use `placeType: natural` for natural features.

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

Only the six documented fields are accepted. `placeType` is the only supported category filter and is valid only for Places. Custom titles, dates, columns, arbitrary filters, sorting, limits, and count controls are not supported.

## Related: header popup buttons

Header buttons are configured separately in the note’s frontmatter:

```yaml
indexes: [residents, settlements, rivers, places]
```

Organizations tagged `group` get Members automatically when `indexes` is omitted. Set `indexes: []` to suppress it, or provide an explicit array to override the default. For example:

```yaml
indexes: [members]
```

Buttons require a `taelgar-header` block and appear below the generated header, outside its information callout. Saved headers and ordinary Markdown titles do not receive them. Empty buttons are hidden, and popup contents calculate when clicked. Popups always use grids; the inline block's options do not configure them. Header `indexes: [places]` buttons always include all place types.

**Show header index popups** in Taelgarverse settings defaults to on. Turn it off to hide header buttons in Obsidian immediately and close open header popups. It does not affect inline indexes or website popups. Header blocks without buttons leave one line of space after their callout; blocks with buttons use the button row's spacing.

Grids and buttons refresh after vault metadata or calendar-date changes. Rendering leaves note text and frontmatter unchanged.
