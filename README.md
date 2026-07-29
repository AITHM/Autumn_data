# AuTuMN Website Data

This repository contains the editable public content and image assets used by the AuTuMN website.

## Repository Contents

- `pages.json` - structured page content for the public website.
- `Images/` - image assets referenced from `pages.json`.

The repository is intentionally small. It is a content/data repository rather than the Blazor website source project, and it should not contain generated `.wasm`, compressed build artifacts, or compiled website output.

## Content Model

`pages.json` is a JSON array. Each item represents one website page:

```json
{
  "page_name": "Home",
  "content": []
}
```

The current pages are:

- `Home`
- `About us`
- `Our Team`
- `Publications`
- `Projects`
- `Links`
- `Contact Us`

The `content` arrays use page-specific field ordering. Before editing a section, inspect a nearby existing entry and preserve the same shape.

Common patterns:

- `Home`: one text block.
- `About us`: section title, section body, image path.
- `Our Team`: name, qualifications, role, affiliation, biography, image path.
- `Publications`: title, abstract, URL, authors, journal, year, DOI, image path.
- `Projects`: title, description, contributors, extra fields, location, start year, end year, image path.
- `Links`: label, URL.

Some text fields contain embedded HTML such as `<br>` and `<a href="...">`. Keep this HTML valid when editing.

## Editing Guidelines

1. Edit `pages.json` with a JSON-aware editor where possible.
2. Preserve each page's existing content array structure.
3. Reference local images with paths under `Images/`.
4. Do not rename or remove images unless all references are updated.
5. Keep public biographies factual and concise.
6. Do not manually edit compiled website artifacts in this repository or in the website publish repository.

## Validation

After editing, validate the JSON:

```powershell
Get-Content -Raw pages.json | ConvertFrom-Json | Out-Null
```

Check that local image references exist:

```powershell
$json = Get-Content -Raw pages.json | ConvertFrom-Json
$refs = New-Object System.Collections.Generic.HashSet[string]
function Scan($x) {
  if ($null -eq $x) { return }
  if ($x -is [string]) {
    foreach ($m in [regex]::Matches($x, 'Images/[A-Za-z0-9_.-]+|Image[0-9]+\.[A-Za-z]+')) {
      [void]$refs.Add($m.Value)
    }
  } elseif ($x -is [System.Collections.IEnumerable] -and -not ($x -is [string])) {
    foreach ($i in $x) { Scan $i }
  } else {
    foreach ($p in $x.PSObject.Properties) { Scan $p.Value }
  }
}
Scan $json
foreach ($r in ($refs | Sort-Object)) {
  $p = if ($r -like 'Images/*') { $r } else { Join-Path 'Images' $r }
  if (-not (Test-Path -LiteralPath $p)) { "Missing image: $r" }
}
```

If the command prints no missing image paths, all local image references resolved.

## Website Updates

Content changes should be made here first. If a change requires altering website layout, routing, components, or build behavior, make that change in the website source repository or propose a separate website pull request rather than editing generated output.
