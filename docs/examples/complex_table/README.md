# Complex Table — A Rich Inspection Target for Pict Panel

<!-- docuserve:example-launch:start -->
> **[&#9654; Launch the live app](examples/complex%5Ftable/index.html)** — runs in your browser, opens in a new tab.
<!-- docuserve:example-launch:end -->

The Complex Table example is a fully-configured `pict-section-form`
application that doubles as a **target host for Pict Panel
inspection**. It is intentionally dense — six form sections, four
pick lists, a tabular grid with a record-set solver, fruit-nutrition
data with thirty-plus records, surface-area solvers, entity-bundle
autofill, tab navigation between statistics views — so that every one
of the panel's browsers has something interesting to display when you
inject it.

Open the page, paste the panel one-liner into the browser console,
and watch the AppData Browser populate with `FruitData.FruityVice[]`
plus the form's marshaled descriptors; the Template Browser fill with
the section-form template set; the View and Provider Browsers list
every view and provider the section-form metacontroller registered.
Then edit a template in the Template Browser and watch the change
survive a page reload via the Template Overrides system. This page
walks both halves — what the host app does, and what the panel
exposes about it — so the two stop feeling like separate concerns.

## What it demonstrates

| Capability | Where you see it |
|------------|------------------|
| `pict-section-form` host with deep configuration | `Complex-Tabular-Application.js` extends `libPictSectionForm.PictFormApplication`; the `default_configuration.pict_configuration` block is the whole form manifest |
| Multiple sections with grouped fields and tab navigation | Six `Sections: [...]` entries (Recipe, Book, FruitGrid, ExtraFormSections, Survey, DeliveryDestination, Documentation) plus two `TabSectionSelector`/`TabGroupSelector` descriptors |
| Tabular layout with a record-set solver | The `FruitGrid` section's only group has `Layout: "Tabular"`, `RecordSetAddress: "FruitData.FruityVice"`, `RecordManifest: "FruitEditor"`, and a `RecordSetSolvers` entry computing `PercentTotalFat` per row |
| Pick lists derived from arrays | Four top-level `PickLists` (`Families`, `Orders`, `Genuses`, `Books`) plus a group-level `PickLists` repeat on the FruitGrid group |
| Manifest solver expressions | Section-level `Solvers` array runs `SUM`, `MEAN`, and arithmetic expressions (`RecipeCounterSurfaceArea = RecipeCounterWidth * RecipeCounterDepth`) |
| Custom input-extension provider | `Complex-Tabular-CustomDataProvider.js` extends `PictInputExtensionProvider` and is wired to a descriptor via `"Providers": ["CustomDataProvider", "Pict-Input-Select"]` |
| Entity-bundle autofill (cascading fetches) | The `IDAuthor` descriptor lists `"Pict-Input-EntityBundleRequest"` + an `EntitiesBundle` array that fetches `Author` → `BookAuthorJoin` → `Book` in order |
| Reference manifests | `ReferenceManifests.FruitEditor` defines a per-row sub-manifest the FruitGrid uses as `RecordManifest` |
| Data shipped inline as `DefaultAppData` | `DefaultAppData: require("./FruitData.json")` — no server, the table populates on first paint |
| **All of the above visible in the Pict Panel** | Inject the panel and the AppData Browser, Template Browser, View Browser, Provider Browser, and Template Overrides views all light up against this host |

## Key files

- `Complex-Tabular-Application.js` — the host. A one-method class
  that registers the custom data provider; the rest is configuration.
  The bottom of the file (lines 18–628) is the form manifest:
  pick lists, sections, descriptors, solvers, reference manifests.
- `Complex-Tabular-CustomDataProvider.js` — a 25-line
  `PictInputExtensionProvider` subclass that logs each
  `onInputInitializeTabular` callback. Trivial in itself, but in the
  Provider Browser you can confirm the panel sees it like any other
  provider.
- `FruitData.json` — the seeded recordset. 30+ fruits with name,
  family, order, genus, and nutrition figures. Seeded as
  `DefaultAppData` so the FruitGrid populates immediately on render.
- `html/index.html` — the shell. One `<div
  id="Pict-Form-Container">`, a `<style id="PICT-CSS">` for the
  framework's cascade, and a `<script>` that calls
  `Pict.safeLoadPictApplication(ComplexTabularApplication, 0)` on
  DOMContentReady.
- `package.json` — the Quackage configuration plus `copyFiles`
  entries that drop `pict.min.js` into `./dist/`. (No Pict
  dependency block, no pict-section-form dependency block; the
  example shares the parent module's `node_modules` so it builds
  against whichever version is installed at the module root.)

## The data model

The application's `pict_configuration` wires three layers together:

- **`AppData.FruitData.FruityVice`** — the seeded recordset. Each
  entry has `name`, `id`, `family`, `order`, `genus`, and a
  `nutritions` sub-object with `calories`, `fat`, `sugar`,
  `carbohydrates`, `protein`. The FruitGrid section maps this array
  via `RecordSetAddress`.
- **Section / Group / Descriptor manifest** — the form layout. Six
  `Sections`; each section has `Groups` with `Layout: "Vertical"` or
  `"Tabular"`; descriptors specify `PictForm.Section`, `Group`,
  `Row`, `Width` placement. Section-form's metacontroller turns this
  into rendered inputs.
- **Pick lists** — four top-level `PickLists` derive their options
  from `FruitData.FruityVice[]` (`Families`, `Orders`, `Genuses`)
  and from `AppData.Books` (`Books`), each writing the materialized
  list to its own `ListAddress`. `UpdateFrequency: "Once"` or
  `"Always"` controls re-computation.

When the panel is injected, the AppData Browser shows all three —
the raw fruit data, the materialized pick-list arrays at
`AppData.FruitMetaLists.*`, and the form-state addresses each
descriptor marshals to.

---

## Feature 1 — Extending `PictFormApplication`

The whole application class is twelve lines of JS — the rest is the
manifest. `PictFormApplication` already knows how to consume a
section-form manifest; the host just attaches a custom provider:

```javascript
const libPictSectionForm = require('pict-section-form');

const libCustomDataProvider = require('./Complex-Tabular-CustomDataProvider.js');

class ComplexTabularApplication extends libPictSectionForm.PictFormApplication
{
	constructor(pFable, pOptions, pServiceHash)
	{
		super(pFable, pOptions, pServiceHash);

		this.pict.addProvider('CustomDataProvider', libCustomDataProvider.default_configuration, libCustomDataProvider);
	}
}

module.exports = ComplexTabularApplication;

module.exports.default_configuration = libPictSectionForm.PictFormApplication.default_configuration;
module.exports.default_configuration.pict_configuration = {
	Product: "ComplexTable",

	DefaultAppData: require("./FruitData.json"),

	DefaultFormManifest: { /* the whole manifest */ }
};
```

Three things to notice:

1. The class inherits `PictFormApplication`'s `default_configuration`
   wholesale, then **mutates** `pict_configuration` to set the
   `Product`, `DefaultAppData`, and `DefaultFormManifest`. That's the
   intended pattern when you want to keep the framework's defaults
   for everything else.
2. `DefaultAppData` is `require()`d from the JSON file. There is no
   `onLoadDataAsync` hook — the data is committed at module load,
   exactly the kind of thing the panel's AppData Browser is built to
   inspect.
3. The lone `addProvider` call wires the custom input extension. The
   custom provider isn't required for the form to work; it's there
   so the Provider Browser has something interesting to show.

## Feature 2 — Section / group / descriptor manifest

The `Sections` array drives the form's spatial layout. Each section
becomes a top-level wrapper; each group inside becomes a fieldset.
Descriptors target sections/groups by hash via their `PictForm`
block. The Recipe section's first group:

```javascript
Sections: [
    {
        Hash: "Recipe",
        Name: "Fruit-based Recipe",

        Solvers:
        [
            "TotalFruitCalories = SUM(FruitNutritionCalories)",
            "AverageFruitCalories = MEAN(FruitNutritionCalories)",
            {
                Ordinal: 99,
                Expression: "AverageFatPercent = MEAN(FruitPercentTotalFat)",
            },
            "RecipeCounterSurfaceArea = RecipeCounterWidth * RecipeCounterDepth",
            "RecipeCounterVolume = RecipeCounterSurfaceArea * RecipeVerticalClearance",
        ],

        MetaTemplates:
        [
            {
                "HashPostfix": "-Template-Wrap-Prefix",
                "Template": "<h1>Rectangular Area Solver Micro-app</h1><div><a href=\"#\" onclick=\"{~Pict~}.PictApplication.solve()\">[ solve ]</a></div><hr />"
            }
        ],

        Groups: [
            { Hash: "Recipe", Name: "Recipe" },
            { Hash: "StatisticsTabs", Name: "Select a Statistics Section" },
            { Hash: "Statistics", Name: "Statistics", Layout: "Vertical" },
            { Hash: "FruitStatistics", Name: "Statistics About the Fruit" },
        ],
    },
    // … five more sections …
]
```

What the panel reveals about this section once it's running:

- **AppData Browser** lists each descriptor's marshaled address.
  `RecipeCounterWidth`, `RecipeCounterDepth`,
  `RecipeCounterSurfaceArea`, `RecipeCounterVolume` all live under
  `AppData` and update as you type or click `[ solve ]`.
- **Template Browser** lists the section-form template fragments
  (the `-Template-Wrap-Prefix`, `-Template-Section-Prefix`,
  `-Template-Group-Prefix`, etc. — one per fragment per section
  prefix) so you can browse the section's actual rendered HTML
  generators.
- **Template Overrides** lets you replace the `<h1>Rectangular Area
  Solver Micro-app</h1>` opening header with your own markup. The
  override persists across page reloads.

## Feature 3 — Tabular layout with a record-set solver

The FruitGrid section turns `FruitData.FruityVice` into a 30-row
editable table. The trick is `Layout: "Tabular"` on the group plus a
`RecordSetAddress` and `RecordManifest`. A `RecordSetSolvers` entry
runs per row, computing a derived column:

```javascript
{
    Hash: "FruitGrid",
    Name: "Fruits of the World",
    Groups: [
        {
            Hash: "FruitGrid",
            Name: "FruitGrid",

            Layout: "Tabular",

            RecordSetSolvers: [
                {
                    Ordinal: 0,
                    Expression: "PercentTotalFat = (Fat * 9) / Calories",
                },
            ],

            PickLists:
            [
                {
                    Hash: "Families",
                    ListAddress: "AppData.FruitMetaLists.Families",
                    ListSourceAddress: "FruitData.FruityVice[]",
                    TextTemplate: "{~D:Record.family~}",
                    IDTemplate: "{~D:Record.family~}",
                    Unique: true,
                    Sorted: true,
                    UpdateFrequency: "Once",
                },
            ],
            RecordSetAddress: "FruitData.FruityVice",
            RecordManifest: "FruitEditor",
        },
    ],
}
```

The `RecordManifest` reference (`"FruitEditor"`) points at a
`ReferenceManifests.FruitEditor` block lower in the file:

```javascript
ReferenceManifests: {
    FruitEditor: {
        Scope: "FruitEditor",

        Descriptors: {
            name: {
                Name: "Fruit Name",
                Hash: "Name",
                DataType: "String",
                Default: "(unnamed fruit)",
                PictForm: { Section: "FruitGrid", Group: "FruitGrid" },
            },
            family: {
                Name: "Family",
                Hash: "Family",
                DataType: "String",
                PictForm: { Section: "FruitGrid", Group: "FruitGrid", "InputType":"Option",
                    "Providers": ["Pict-Input-Select"],
                    "SelectOptionsPickList": "Families"}
            },
            // … order / genus / lastWatered / nutritions.* …
        },
    },
},
```

So each row of `FruitData.FruityVice` ends up rendered with the
descriptors in `FruitEditor.Descriptors` — name as a string input,
family as a `<select>` populated from the `Families` pick list, order
/ genus / calories / fat / carbs / protein as appropriate.
`PercentTotalFat = (Fat * 9) / Calories` runs per row and the
computed value appears in the grid alongside the raw values.

Inject the panel and the AppData Browser shows
`FruitData.FruityVice` as an expandable array; each row carries the
raw fields plus the solver-derived `PercentTotalFat`. The View
Browser lists the FruitGrid view and its renderables; the Template
Browser exposes the tabular-row fragment templates the metacontroller
generated.

## Feature 4 — Pick lists derived from arrays

The four top-level `PickLists` materialize `<select>` options from
data already in `AppData`. Each `PickList` defines a source array, a
template for each list entry's `Text` and `ID`, and an
`UpdateFrequency` that controls re-derivation:

```javascript
PickLists:
[
    {
        Hash: "Families",
        ListAddress: "AppData.FruitMetaLists.Families",
        ListSourceAddress: "FruitData.FruityVice[]",
        TextTemplate: "{~D:Record.family~}",
        IDTemplate: "{~D:Record.family~}",
        Unique: true,
        Sorted: true,
        UpdateFrequency: "Once",
    },
    {
        Hash: "Orders",
        ListAddress: "AppData.FruitMetaLists.Orders",
        ListSourceAddress: "FruitData.FruityVice[]",
        TextTemplate: "{~D:Record.order~}",
        IDTemplate: "{~D:Record.order~}",
        Unique: true,
        UpdateFrequency: "Once",
    },
    {
        Hash: "Genuses",
        ListAddress: "AppData.FruitMetaLists.Genuses",
        ListSourceAddress: "FruitData.FruityVice[]",
        TextTemplate: "{~D:Record.genus~}",
        IDTemplate: "{~D:Record.genus~}",
        Sorted: true,
        UpdateFrequency: "Always",
    },
    {
        Hash: "Books",
        ListAddress: "AppData.AuthorsBooks",
        ListSourceAddress: "Books[]",
        TextTemplate: "{~D:Record.Title~}",
        IDTemplate: "{~D:Record.IDBook~}",
        Sorted: true,
        UpdateFrequency: "Always",
    }
],
```

`UpdateFrequency: "Once"` computes the pick list at first
materialization and never re-runs it; `"Always"` re-runs on every
solve cycle. For `Families` and `Orders` (which come from the raw
fruit data and don't change), "Once" is correct; for `Genuses` (a
sorted list — if a row's genus changes, the order changes) and
`Books` (a derived array filled by the entity-bundle fetch chain),
"Always" is correct.

In the AppData Browser these materialized lists show up under
`AppData.FruitMetaLists.*` and `AppData.AuthorsBooks`. Editing a
fruit's family in the FruitGrid and pressing solve updates
`AppData.FruitMetaLists.Genuses` immediately — visible in the panel
as the list re-sorts.

## Feature 5 — Section solver expressions

The Recipe section's `Solvers` array runs expression strings against
the form's marshaled data. Each expression is "AppData address = some
expression of other addresses." The framework parses the strings,
extracts dependencies, and re-runs them on every solve cycle:

```javascript
Solvers:
[
    "TotalFruitCalories = SUM(FruitNutritionCalories)",
    "AverageFruitCalories = MEAN(FruitNutritionCalories)",
    {
        Ordinal: 99,
        Expression: "AverageFatPercent = MEAN(FruitPercentTotalFat)",
    },
    "RecipeCounterSurfaceArea = RecipeCounterWidth * RecipeCounterDepth",
    "RecipeCounterVolume = RecipeCounterSurfaceArea * RecipeVerticalClearance",
],
```

- The first two compute `SUM` and `MEAN` over the per-row
  `FruitNutritionCalories` values (mapped via the descriptor
  `"FruitData.FruityVice[].nutritions.calories"` → `Hash:
  "FruitNutritionCalories"`).
- The `AverageFatPercent` solver has an explicit `Ordinal: 99` so it
  runs **after** the FruitGrid's `RecordSetSolvers` populate the
  per-row `PercentTotalFat`. Without the ordinal, this would race
  the record-set computation and average over zero.
- The last two are pure arithmetic — width × depth → surface area;
  surface area × clearance → volume. Click `[ solve ]` in the
  rendered "Rectangular Area Solver Micro-app" header (defined in
  the section's `MetaTemplates` block) and the volume re-computes
  from whatever values you typed.

In the panel: the AppData Browser shows every solver's output cell
live. The Provider Browser lists the section-form's solver provider
along with the host's custom one. Edit the surface-area solver's
template in the Template Browser — though for this kind of work the
direct AppData edit is more responsive.

## Feature 6 — Entity-bundle autofill (cascading fetches)

The Book section demonstrates `Pict-Input-EntityBundleRequest`, the
section-form provider that fires a configured chain of Meadow entity
fetches whenever the input's value changes:

```javascript
"Author.IDAuthor": {
    Name: "Author ID",
    Hash: "IDAuthor",
    DataType: "Number",
    PictForm: {
        Section: "Book", Group: "Author", Row: 1, Width: 1,
        // This performs an entity bundle request whenever a value is selected.
        Providers: ["Pict-Input-EntityBundleRequest", "Pict-Input-AutofillTriggerGroup"],
        EntitiesBundle: [
                {
                    "Entity": "Author",
                    "Filter": "FBV~IDAuthor~EQ~{~D:Record.Value~}",
                    "Destination": "AppData.CurrentAuthor",
                    // This marshals a single record
                    "SingleRecord": true
                },
                {
                    "Entity": "BookAuthorJoin",
                    "Filter": "FBV~IDAuthor~EQ~{~D:AppData.CurrentAuthor.IDAuthor~}",
                    "Destination": "AppData.BookAuthorJoins"
                },
                {
                    "Entity": "Book",
                    "Filter": "FBL~IDBook~INN~{~PJU:,^IDBook^AppData.BookAuthorJoins~}",
                    "Destination": "AppData.Books"
                }
            ],
        EntityBundleTriggerGroup: "BookTriggerGroup"
    }
},
```

What's happening:

1. The user types a value into `IDAuthor`.
2. The first bundle entry fetches `Author` where
   `IDAuthor == Record.Value`, marshals the single record into
   `AppData.CurrentAuthor`.
3. The second bundle entry fetches `BookAuthorJoin` filtered by the
   newly-populated `AppData.CurrentAuthor.IDAuthor`, into
   `AppData.BookAuthorJoins` (a record set).
4. The third entry uses the `PJU:` template join macro to build a
   comma-separated list of `IDBook` values from
   `AppData.BookAuthorJoins`, then fetches `Book` rows where
   `IDBook IN (…)`, into `AppData.Books`.
5. The `BookTriggerGroup` named trigger fires; the `AuthorName`
   descriptor (Pict-Input-AutofillTriggerGroup, trigger address
   `AppData.CurrentAuthor.Name`) re-marshals; the `SpecificIDBook`
   `<select>` refreshes from the `Books` pick list.

Without a Meadow back-end this example will run but the entity
fetches log a warning; the rest of the form continues to work. The
real point is that the AppData Browser exposes
`AppData.CurrentAuthor`, `AppData.BookAuthorJoins`, and
`AppData.Books` as separate destinations — exactly the slots the
bundle plants its data into.

## Feature 7 — Tab navigation between groups and sections

The form uses two specialized descriptors to drive tab navigation —
one between groups within a single section
(`InputType: "TabGroupSelector"`) and one between sections
(`InputType: "TabSectionSelector"`):

```javascript
"UI.StatisticsTabState": {
    Name: "Statistics Tab State",
    Hash: "StatisticsTabState",
    DataType: "String",
    PictForm: {
        Section: "Recipe",
        Group: "StatisticsTabs",
        InputType: "TabGroupSelector",
        TabGroupSet: ["Statistics", "FruitStatistics"],
        TabGroupNames: ["Statistics", "Fruit Statistics"]
    }
},

"UI.ExtraSectionSelection": {
    Name: "Extra Form Sections",
    Hash: "ExtraFormSectionSelection",
    DataType: "String",
    PictForm: {
        Section: "ExtraFormSections",
        InputType: "TabSectionSelector",
        TabSectionSet: ["Survey", "DeliveryDestination", "Documentation"],
        TabSectionNames: ["Survey", "Delivery Destination", "Documentation"]
    }
},
```

The `TabSectionSet` is the hashes of the sections that should swap
in/out; `TabSectionNames` is the labels shown on the tabs. The
value's address (`AppData.UI.ExtraSectionSelection`) is where the
selected tab is stored — so the AppData Browser shows you exactly
which tab is active, and editing that field in the panel switches
tabs without clicking.

The framework reuses these same descriptors for any tab-switcher in
a section-form app; this example shows that **tabs are just
descriptors with a specific input type**, not a separate concept.

## Feature 8 — Custom input-extension provider

The host adds one custom provider — a tiny subclass of
`PictInputExtensionProvider` that logs each tabular-row initialization:

```javascript
const libPictSectionInputExtension = require('pict-section-form').PictInputExtensionProvider;

class CustomInputHandler extends libPictSectionInputExtension
{
	constructor(pFable, pOptions, pServiceHash)
	{
		super(pFable, pOptions, pServiceHash);
	}

	onInputInitialize(pView, pGroup, pRow, pInput, pValue, pHTMLSelector)
	{
		this.log.trace(`CustomInputHandler.onInputInitializeTabular() for view [${pView.Hash}] called`);
		return super.onInputInitialize(pView, pGroup, pInput, pValue, pHTMLSelector);
	}

	onInputInitializeTabular(pView, pGroup, pInput, pValue, pHTMLSelector)
	{
		this.log.trace(`CustomInputHandler.onInputInitializeTabular() for view [${pView.Hash}] called`);
		return super.onInputInitializeTabular(pView, pGroup, pInput, pValue, pHTMLSelector);
	}
}

module.exports = CustomInputHandler;
```

It's wired to a single descriptor:

```javascript
"Recipe.Feeds": {
    Name: "Feeds",
    Hash: "RecipeFeeds",
    DataType: "PreciseNumber",
    Default: "1",
    PictForm: { Section: "Recipe", Group: "Statistics", Row: 1, Width: 1,
        "ExtraDescription": "How many people does this recipe feed?",
        "InputType":"Option",
        "Providers": ["CustomDataProvider", "Pict-Input-Select"],
        "SelectOptions": [{"id":"few", "text":"Few"}, {"id":"some", "text":"Some"}, {"id":"many", "text":"Many"}]
    },
},
```

`Providers: ["CustomDataProvider", "Pict-Input-Select"]` chains them
— the custom provider's `onInputInitialize` runs first, then the
standard `Pict-Input-Select` provider sets up the select. The
**Provider Browser** in Pict Panel lists `CustomDataProvider`
alongside every framework provider, so you can confirm wiring without
reading source. Hover the entry to see its `solve` action button;
click into the provider to view its configuration options.

## Feature 9 — Inspecting this app with Pict Panel

Once the page is open, paste the panel one-liner into the browser
console:

```javascript
fetch('https://cdn.jsdelivr.net/npm/pict-panel/dist/pict-panel.js').then(r=>r.text()).then(eval).then(()=>PictPanel.inject())
```

The panel appears in the top-right. Each of its five navigation views
has something to do here:

| Panel view | What you see on this host |
|------------|---------------------------|
| **AppData** | `FruitData.FruityVice` (the seed array), `FruitMetaLists.*` (pick-list materializations), the form's marshaled descriptor values, the solver outputs (`RecipeCounterSurfaceArea`, `TotalFruitCalories`, `AverageFatPercent`), and the tab-state addresses. Click into any node to expand. Click a leaf value to edit in place. |
| **Templates** | The full section-form template set — `-Template-Wrap-Prefix`, `-Template-Section-Prefix`, `-Template-Group-Prefix`, `-Template-Row-Prefix`, `-Template-Input`, and one per data-type variant. Filter by `Recipe` or `FruitGrid` to narrow. |
| **Views** | The form metacontroller's views, the FruitGrid tabular view, and any section subviews. Hover for `render` / `renderAsync` buttons; click into a view to see its renderables and template list. |
| **Providers** | `CustomDataProvider`, all the section-form input providers (`Pict-Input-Select`, `Pict-Input-EntityBundleRequest`, `Pict-Input-AutofillTriggerGroup`, `Pict-Input-DateTime`, etc.), the template provider, the metacontroller. Each has a `solve` button. |
| **Overrides** | Empty on first load. Edit any template in the Template Browser and save — the override appears here, toggleable on/off, exportable as JSON or JS source. Persists across reloads via localStorage. |

The Template Overrides system is the headline feature for iterating
on form templates without rebuilding. Find the
`-Template-Input-DataType-Number` template, edit its HTML to add a
prefix span, save — the form re-renders with your change. Reload the
page; the change is still there because
`PP-TemplateOverrideStorage.applyOverrides()` runs on inject.

## Running the example

```bash
cd example_applications/complex_table
npm run build      # quack build + quack copy → ./dist
# then open dist/index.html in a browser
```

Or, from the `example_applications/` parent, run the included server
to serve every example over HTTP on port 9998:

```bash
cd example_applications
node ServeExamples.js
# then visit http://localhost:9998/complex_table/
```

The server also serves the panel bundle at
`http://localhost:9998/pict-panel.js`, so you can inject from the
same origin:

```javascript
fetch('http://localhost:9998/pict-panel.js').then(r=>r.text()).then(eval).then(()=>PictPanel.inject())
```

## Things to try in the running app

- **Open the page and inject the panel.** Browse `AppData` and confirm
  `FruitData.FruityVice` is the seeded array of 30+ fruits.
- **Type a width and depth** in the Recipe section's Statistics
  group, then click `[ solve ]` in the section header. Watch
  `RecipeCounterSurfaceArea` and `RecipeCounterVolume` update in the
  AppData Browser. Edit `RecipeCounterWidth` directly in the panel
  and re-solve — the form view re-renders.
- **Click the "Fruit Statistics" tab** inside Recipe. `AppData.UI.StatisticsTabState`
  flips to `"FruitStatistics"`. Edit that AppData value back to
  `"Statistics"` in the panel — the tab flips back without clicking.
- **Change a fruit's family** in the FruitGrid (the row is editable
  in place). Solve. The `Genuses` pick list re-sorts because its
  `UpdateFrequency` is `"Always"`.
- **Open the Template Browser**, filter for `FruitGrid`. Edit the
  tabular row template — wrap a column header in `<em>` tags or
  similar — and save. The grid re-renders.
- **Reload the page.** The override survives. Toggle it off in the
  Overrides view, re-render — the original template is back. Toggle
  it on — your edit returns. Click `Export as JS` to copy a
  ready-to-paste template literal block.
- **Drag the panel header** to move it. Click the tab-mode icon to
  collapse to a logo tab. Click night mode for a dark theme. Reload —
  every panel state survives.

## Takeaways

1. **Pict Panel reads what the framework already knows.** Every
   browser in the panel queries the host's own services — the
   `TemplateProvider`, the views map, the providers map, the
   `AppData` tree. There is no separate inspector data model. If a
   piece of state lives in `AppData`, the panel shows it; if a
   template is registered, the panel can edit it.
2. **The richer the host, the more useful the panel.** This example
   is dense on purpose. Six sections, four pick lists, a tabular
   grid with solvers, entity-bundle autofill — every dimension
   exercises a different panel browser. When applying the panel to
   your own app, the same observation applies in reverse: anything
   missing from the panel is missing from the framework's
   registered state.
3. **Template overrides are the in-browser iteration story.** Edit a
   template, save, see it apply. Reload, the override is still
   there. Export, paste into your real source when satisfied. No
   build round-trip.
4. **The host is configuration-first.** Twelve lines of class plus
   ~600 lines of manifest. Anything you can do in this example, you
   can do without writing a custom view — the panel is your
   debugging surface for the configuration, not a place to write
   imperative code.
5. **`PictFormApplication` is the right base class for form-heavy
   hosts.** Extending it inherits the full section-form lifecycle
   (manifest parsing, descriptor materialization, solver execution,
   pick-list derivation, entity-bundle autofill). The class file
   stays small because the framework owns the heavy lifting.

## Related documentation

- [Hotloading](../../hotloading.md) — the inject-from-CDN flow, the
  alternative "bundle it" approach, and what
  `PictPanel.inject()` actually does
- [Architecture](../../architecture.md) — how the panel registers its
  providers and views into the host's Pict instance
- [Views](../../views.md) — per-view detail for AppData, Template,
  View, Provider, and Overrides browsers
- [Providers](../../providers.md) — the four providers (`PP-Router`,
  `PP-CSS-Hotloader`, `PP-ConfigStorage`,
  `PP-TemplateOverrideStorage`) the panel installs
- [Template Overrides](../../overrides.md) — the full override
  workflow: edit → save → toggle → export
