# Czo Template

Inserts file header templates and keeps them up to date on every save.

```c
/*
 * Filename: main.c
 * Author: Olivier Sirol <czo@free.fr>
 * License: GPL-2.0 (http://www.gnu.org/copyleft)
 * File Created: 07 March 2026
 * Last Modified: Sunday 22 March 2026, 21:38
 * Edit Time: 5:05:06
 * Description:
 *
 *      my C program
 *
 * Copyright: (C) 2026 Olivier Sirol <czo@free.fr>
 */
```

The `Filename:`, `Author:`, `License:`, `File Created:`, `Last Modified:`, `Edit Time:` and `Copyright:` fields are updated automatically on every save.

## Features

- Insert a language-appropriate header template at the top of any file
- Auto-update header fields every time you save
- Track cumulative editing time per file
- User-defined regexp substitutions with macro tokens
- Fully customizable templates per language
- Legacy field renaming for compatibility with older headers

## Commands

| Command | Shortcut | Description |
|---|---|---|
| `Czo Template: Enable` | | Enable the extension |
| `Czo Template: Disable` | | Disable the extension |
| `Czo Template: Insert Template` | `Ctrl+Alt+H  Ctrl+Alt+H` | Insert a header for the current language |
| `Czo Template: Insert Template For languageId...` | | Insert a header for a chosen language |
| `Czo Template: Update` | `Ctrl+Alt+H  Ctrl+Alt+U` | Update header fields without saving |

## Configuration

All settings live in `settings.json` (user or workspace level).

```json
{
    "czoTemplate.enabled": true,
    "czoTemplate.maxHeaderLines": 50,
    "czoTemplate.author": "Olivier Sirol <czo@free.fr>",
    "czoTemplate.license": "GPL-2.0 (http://www.gnu.org/copyleft)",
    "czoTemplate.shortdate": "%d %B %Y",
    "czoTemplate.longdate": "%A %d %B %Y, %H:%M"
}
```

| Setting | Default | Description |
|---|---|---|
| `czoTemplate.enabled` | `true` | Enable or disable the extension |
| `czoTemplate.maxHeaderLines` | `50` | Number of lines scanned for header fields |
| `czoTemplate.author` | `"John Doe <john.doe@mail.org>"` | Author name and email |
| `czoTemplate.license` | `"GPL-2.0 (http://www.gnu.org/copyleft)"` | License string |
| `czoTemplate.shortdate` | `"%d %B %Y"` | strftime format for `File Created:` |
| `czoTemplate.longdate` | `"%A %d %B %Y, %H:%M"` | strftime format for `Last Modified:` |
| `czoTemplate.templates` | `{}` | Custom template overrides (see below) |
| `czoTemplate.regexps` | `[]` | User-defined regexp substitutions (see below) |

## Supported languages

Built-in templates are provided for:

`c`, `cpp`, `crontab`, `css`, `h`, `hpp`, `html`, `java`, `javascript`, `json`, `lua`, `makefile`, `markdown`, `perl`, `php`, `python`, `shellscript`, `typescript`

Any other VS Code `languageId` can be supported by adding a custom template (see [Custom templates](#custom-templates)).

## Auto-update on save

Every time you save, any of the following fields found in the first `czoTemplate.maxHeaderLines` lines are updated:

| Field | Action |
|---|---|
| `Filename:` | Current filename (basename) |
| `FullFilename:` | Full path of the file |
| `Author:` | Value of `czoTemplate.author` |
| `License:` | Value of `czoTemplate.license` |
| `File Created:` | Set once with `czoTemplate.shortdate` format (only if no year is present) |
| `Last Modified:` | Current date/time with `czoTemplate.longdate` format |
| `Edit Time:` | Cumulative editing time since the file was opened |
| `Copyright:` | `(C) [year range] czoTemplate.author` |

### Legacy field renaming

For compatibility with older file headers, these fields are automatically renamed:

| Old field | Renamed to |
|---|---|
| `Started:` | `File Created:` |
| `Created:` | `File Created:` |
| `Last Change:` | `Last Modified:` |

## Custom templates

Override any built-in template or create new ones via `czoTemplate.templates`. Each key is a template name (file extension like `c`, `sh`, `py`, or a VS Code `languageId` like `javascript`, `perl`). The value is an array of strings, one per line.

```json
"czoTemplate.templates": {
    "c": [
        "/*",
        " * Filename:",
        " * Author:",
        " * License:",
        " * File Created:",
        " * Last Modified:",
        " * Edit Time:",
        " * Description:",
        " *",
        " * Copyright:",
        " */"
    ],
    "sh": [
        "#! /usr/bin/env bash",
        "#",
        "# Filename:",
        "# Author:",
        "# License:",
        "# File Created:",
        "# Last Modified:",
        "# Edit Time:",
        "# Description:",
        "#",
        "# Copyright:"
    ]
}
```

### Template resolution order

1. `czoTemplate.templates.<languageId>` in `settings.json`
2. `czoTemplate.templates.<fileExt>` in `settings.json`
3. Built-in template `<extensionPath>/out/templates/template.<fileExt>`

## Custom regexp substitutions

Define your own regexp substitutions applied to the **entire document** on every save or update via `czoTemplate.regexps`.

Each entry has:

| Field | Description |
|---|---|
| `find` | JavaScript regexp string (without `/…/` delimiters) |
| `replace` | Replacement string — supports `$1`, `$2`, … capture groups and macro tokens |
| `mode` | Optional flags: `i` (case-insensitive), `g` (global) |

> **Note:** `m` and `s` flags have no effect as regexps are applied line by line.

```json
"czoTemplate.regexps": [
    {
        // substitute CVS $Id: because I'm now using Git and I need an
        // identifier ($Id) from my old CVS scripts...
        "find": "(\\$Id\\:).*( czo Git \\$)",
        "replace": "$1 <<FILENAME>>,v 1.42 <<STRFTIME=\"%Y/%m/%d %T\">>$2"
    },
    {
        // substitute ( Czo\Date: ) by ( Czo\Date: 2024-03-14 14:36 )
        "find": "(\\( Czo\\Date:)\\s*[\\d :+-]*\\s*\\)",
        "replace": "$1 <<STRFTIME=\"%F %R\">> )"
    }
]
```

> **Tip:** To avoid a pattern matching itself in `settings.json` on save, break the literal string with an escaped character, e.g. `"\\$Id\\:"` matches `$Id:` in files but the string `\\$Id\\:` in `settings.json` does not match itself.

### Macro tokens

The following tokens are expanded in the `replace` string before the regexp is applied:

| Token | Value |
|---|---|
| `<<STRFTIME="%fmt">>` | Date/time formatted with strftime (e.g. `<<STRFTIME="%Y/%m/%d %T">>`) |
| `<<FILENAME>>` | Current filename (basename only, e.g. `foo.c`) |
| `<<FULLFILENAME>>` | Full path of the current file |
| `<<HOME>>` | Home directory (`$HOME` or `%USERPROFILE%`) |
| `<<USER>>` | Current user (`$USER` or `%USERNAME%`) |
| `<<HOSTNAME>>` | Hostname (`$HOSTNAME`) |

## License

Released under the [GPL-2.0 License](https://www.gnu.org/licenses/old-licenses/gpl-2.0.html).

Port of `template.vim` by Olivier Sirol (`czo@free.fr`) — see [.vimrc](https://github.com/czodroid/dotfiles/blob/master/.vimrc).

Copyright (C) 2026 Olivier Sirol <czo@free.fr>
