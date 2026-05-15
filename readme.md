# Simple Excel Web Transformer

Simple Excel Web Transformer is a single-file, browser-based tool for transforming Excel and CSV files. It can map columns, filter rows, pivot, unpivot, aggregate, preview results, export output files, and save reusable transformation configs.

The app runs from `excel-transformer.html`. No server is required for normal use. File parsing, transformation, preview, export, and saved configs all run in the browser.

## Quick Start

1. Open `excel-transformer.html` in a modern browser.
2. Upload an `.xlsx`, `.xls`, or `.csv` file.
3. Choose an operation type:
   - `Regular transform`
   - `Pivot`
   - `Unpivot`
   - `Aggregate`
4. Configure the visible settings for that operation.
5. Preview the result.
6. Export the output file.
7. After export, choose whether to save or download the transformation config.

Use `Reupload File` to start over with a new source file.

## Features

- Upload `.xlsx`, `.xls`, and `.csv` files.
- Rename, reorder, keep, or drop columns.
- Add formula, constant, and split-derived columns.
- Apply data types, transforms, and output formats.
- Filter rows using field-aware operators.
- Run pivot, unpivot, and aggregate operations.
- Preview transformed rows before export.
- Export transformed data as `.xlsx` and `.csv`.
- Save configs in browser `localStorage`.
- Download and reload config JSON files.
- Use Column Script and Row Script for advanced transformations.

## Deployment

### Local or Network Drive

Double-click `excel-transformer.html` and open it in Microsoft Edge, Google Chrome, or Mozilla Firefox.

### IIS

1. Copy `excel-transformer.html` to `C:\inetpub\wwwroot\`.
2. Open `http://your-server/excel-transformer.html`.

### SharePoint

1. Upload `excel-transformer.html` to a SharePoint document library.
2. Share the direct file link with users.

All processing happens in the browser. Source files are not uploaded to a backend.

## Config JSON Workflow

Transformation settings can be saved as config JSON so the same rules can be reused later.

Supported config actions:

- Save a config to browser `localStorage`.
- Download a config as a `.json` file.
- Load a saved local config.
- Drop or browse for a config JSON file.
- Validate a config against the uploaded data file before applying it.

The app checks whether referenced source columns exist in the uploaded file. If there are mismatches, it reports the discrepancies before applying the config.

Configs can preserve Column Script logic. When a config is saved after applying a Column Script, the app stores that script and reruns it against the newly uploaded file's current headers before validating and restoring the mapping. Saved edits for old source columns are kept when those columns still exist. If an old source column is no longer present, the stale saved definition is overridden by the script-generated mapping for the new headers.

For example, if a Column Script renames headers that match a date or year/month pattern, a later file can contain a new set of matching period columns and the config can rebuild the mapping from the new headers.

The app only asks whether to save or export an updated config after an output export. Previewing does not trigger the config-save prompt.

Saved configs are browser-local. They are not shared between users, machines, or browsers.

## Operation Modes

Only the UI for the selected operation is shown. For example, Pivot hides regular transform and Unpivot controls; Unpivot hides Pivot and regular transform controls.

### Regular Transform

Use Regular Transform for column-level cleanup and row-level filtering.

Main capabilities:

- Rename output columns.
- Reorder columns by dragging or using move buttons.
- Keep selected columns or exclude columns from output.
- Add formula, constant, and split-derived columns.
- Change data type and formatting.
- Apply data-type-specific transforms and formats.
- Apply bulk rename, prefix, suffix, find/replace, and bulk settings.
- Apply data type, transform, and format to multiple selected fields at once.
- Use undo and reset while working.

### Data Types, Transforms, and Formats

Transform and format dropdowns are filtered by the selected data type. This keeps the UI focused so date fields do not show number-only options, number fields do not show text cleanup options, and so on.

#### Text

Text transforms:

- No change
- Trim spaces
- Collapse spaces
- To UPPERCASE
- To lowercase
- To Title Case
- Remove special characters
- Remove digits
- Keep digits only
- To snake_case

Text formats:

- No format
- Trim
- Collapse spaces
- UPPERCASE
- lowercase
- Title Case
- snake_case

#### Number

Number transforms:

- No change
- Parse number
- Absolute value
- Negate
- Round
- Floor
- Ceil

Number formats:

- No format
- Integer
- Integer with comma
- Decimal 2
- Decimal 2 with comma
- Decimal 4
- Decimal 4 with comma
- Currency
- Currency with comma
- Percent

#### Date

Date transforms:

- No change
- To `YYYY-MM-DD`
- To `MM/DD/YYYY`
- Extract year
- Extract month number
- Extract month name
- Extract day
- Extract quarter

Date formats:

- No format
- `YYYY-MM-DD`
- `MM/DD/YYYY`
- `YYYYMMDD`
- `MMM YYYY`
- `YYYY-MM`
- Month name

#### Boolean

Boolean transforms:

- No change
- Trim spaces
- To UPPERCASE
- To lowercase

Boolean formats:

- No format
- `true / false`
- `Yes / No`
- `Y / N`

These same type-aware controls are used in regular mapping, bulk settings, pivot and aggregate measures, and unpivot generated fields.

Comma-separated number formats are available as explicit format choices for integer, decimal, and currency outputs. `No format` and `Percent` intentionally do not include comma variants.

### Row Filters

Regular Transform supports filtering by multiple columns from the UI.

Filter behavior is based on field type:

- Text fields support text-oriented matching.
- Number fields support numeric comparisons.
- Date fields support date-oriented comparisons.
- Custom JavaScript filters can be used for advanced logic.

### Pivot

Use Pivot when rows need to be summarized across one or more pivot dimensions.

Typical setup:

- Select grouping fields.
- Select one or more pivot columns.
- Select one or more value fields.
- Choose aggregation behavior for each value field.
- Preview the pivoted output before export.

Pivot runs after filtering, mapping, formulas, transforms, and any active Row Script.

### Unpivot

Use Unpivot when wide columns need to be converted into rows.

Checking Unpivot opens a draggable settings popup. Use `Settings` to reopen the current configuration and `Clear` to remove the active unpivot setup.

Unpivot types:

| Type | Purpose |
| --- | --- |
| `Standard` | Converts selected measure columns into attribute/value rows while repeating fixed ID columns. |
| `Delimited Header` | Splits selected source headers by a delimiter and creates one output row per selected cell. |
| `Paired Metric` | Groups related columns with matching prefixes into one row containing multiple metric fields. |

#### Standard Unpivot

Fixed ID columns remain unchanged and are repeated for every generated row.

Selected unpivot columns are converted into:

- An attribute column containing the original column name.
- A value column containing the original cell value.

Example:

Input:

| Student_ID | Math_Score | Science_Score |
| --- | ---: | ---: |
| 1 | 90 | 85 |

Output:

| Student_ID | Attribute | Value |
| --- | --- | ---: |
| 1 | Math_Score | 90 |
| 1 | Science_Score | 85 |

The attribute and value output column names can be customized. Type, transform, and format options are available for generated fields.

#### Delimited Header Unpivot

Delimited Header uses a vertical split approach. Every selected source cell creates its own output row.

For each selected column:

1. Split the header by the selected delimiter.
2. Put the first split part into the configured first attribute column.
3. Put the remaining split text into the configured second attribute column.
4. Put the cell value into the configured value column.

Example with delimiter `_`:

Input:

| id | 2024_Sales | 2024_Profit |
| --- | ---: | ---: |
| 1 | 100 | 40 |

Output:

| id | Year | Metric | Amount |
| --- | --- | --- | ---: |
| 1 | 2024 | Sales | 100 |
| 1 | 2024 | Profit | 40 |

The first attribute name, second attribute name, and value column name are user-configurable. Attribute and value fields can also have type, transform, and format settings.

#### Paired Metric Unpivot

Paired Metric groups related columns by a shared prefix and outputs multiple value columns on the same generated row.

Example with delimiter `_`:

Input:

| id | Q1_Sales | Q1_Profit | Q2_Sales | Q2_Profit |
| --- | ---: | ---: | ---: | ---: |
| 1 | 100 | 40 | 150 | 55 |

Output:

| id | Period | Sales | Profit |
| --- | --- | ---: | ---: |
| 1 | Q1 | 100 | 40 |
| 1 | Q2 | 150 | 55 |

This is useful when columns share a common identifier such as month, quarter, year, or scenario and each group contains related metrics.

### Aggregate

Use Aggregate when rows should be grouped and summarized without building a pivot table.

Typical setup:

- Select group-by fields.
- Select measure fields.
- Choose aggregation functions.
- Preview and export the summarized output.

## Scripts

The app includes JavaScript scripting for advanced cases.

### Column Script

Column scripts modify the column mapping definition. Use them to rename, drop, reorder, or change transforms across many columns.

When a transformation config is saved after a Column Script is applied, the script is included in the config. Reusing that config on a new file reruns the script against the new file headers, preserves saved source-column edits that are still valid, and overrides stale saved source-column definitions when their old headers are gone. Formula and constant columns are restored only when their source references are still valid or are recreated by the script. This is useful for source files where period columns change over time but still follow the same naming pattern.

Example:

```js
colDefs.forEach((d) => {
  if (/tmp|test/i.test(d.srcCol)) d.dropped = true;
});
```

### Row Script

Row scripts run during Preview and Export and can modify or drop rows.

Example:

```js
function transformRow(row) {
  if (!row.Amount || Number(row.Amount) === 0) return null;
  const out = Object.assign({}, row);
  out.Amount = Number(row.Amount);
  return out;
}
```

Scripts are saved in browser `localStorage` and remain local to that browser.

## Preview and Export

Preview shows transformed rows before writing a file. Use it to verify column names, values, filters, pivot output, unpivot output, and aggregate output.

Export downloads the transformed result as `.xlsx` and `.csv`. After export, the app can ask whether to save or download the latest transformation config.

## Processing Notes

- Header rows are treated as field names.
- Excel date serial headers are detected and converted where possible.
- Config validation checks saved mappings against the uploaded file headers.
- Local saved configs and scripts use browser `localStorage`.
- Browser local storage is not a database and is not shared across devices.
- For future database-backed configs, the current config JSON can be used as the portable format.

## Browser Compatibility

| Browser | Minimum Version |
| --- | --- |
| Microsoft Edge | 80+ |
| Google Chrome | 80+ |
| Mozilla Firefox | 113+ |

Internet Explorer is not supported.

## Updating the Tool

Replace the deployed `excel-transformer.html` file with the new version. Users receive the update the next time they reload the page.

Saved configs and scripts remain in each user's browser `localStorage` unless that browser data is cleared.
