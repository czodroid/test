# VSCode Czo Template Extension

Inserts file header templates and keeps them up to date on every save.

Example for a C file:

```c
/*
 * Filename: README.md
 * Author: Olivier Sirol <czo@free.fr>
 * License: GPL-2.0 (http://www.gnu.org/copyleft)
 * File Created: 07 March 2026
 * Last Modified: Sunday 22 March 2026, 21:38
 * Edit Time: 5:05:06
 * Description:
 *
 * Copyright: (C) 2026 Olivier Sirol <czo@free.fr>
 */
```

`Filename:`, `Author:`, `License:`, `File Created:`, `Last Modified:`, `Edit Time:` and `Copyright:` fields are updated on every save.

## Features

### Command Palette

| Command | Description |
|---|---|
| `Czo Template: Enable` | Enable the extension |
| `Czo Template: Disable` | Disable the extension |
| `Czo Template: Insert Header` | Insert a language-appropriate header at the top of the file |
| `Czo Template: Update` | Apply header updates immediately without saving |

### Keyboard Shortcuts

| Shortcut | Command |
|---|---|
| `Ctrl+Alt+H  Ctrl+Alt+H` | Insert Header |
| `Ctrl+Alt+H  Ctrl+Alt+U` | Update |

### Configuration

All settings can be configured at user level (`settings.json`) or overridden per project in a workspace (`<project>/.vscode/settings.json`).

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

### Insert Header

`Czo Template: Insert Header` inserts a language-appropriate header at the top of the current file, then immediately applies all field updates.

Supported languages:
* `c`
* `cpp`
* `crontab`
* `css`
* `h`
* `hpp`
* `html`
* `java`
* `javascript`
* `json`
* `lua`
* `makefile`
* `markdown`
* `perl`
* `php`
* `python`
* `shellscript`
* `typescript`
* and any other language you define in custom templates using the VS Code `languageId`.

### Update

`Czo Template: Update` updates the header fields. Same as auto-update on save, but without saving the file.

### Auto-update on save

Every time you save a file, any of the following fields found **in the first `czoTemplate.maxHeaderLines` lines** are updated automatically:

| Field | Action |
|---|---|
| `Filename:` | Updates to current filename |
| `FullFilename:` | Updates to current full path |
| `Author:` | Updates from `czoTemplate.author` |
| `License:` | Updates from `czoTemplate.license` |
| `File Created:` | Set once using strftime format `czoTemplate.shortdate` (only if no year is present) |
| `Last Modified:` | Updates using strftime format `czoTemplate.longdate` |
| `Edit Time:` | Accumulates editing time since the file was opened |
| `Copyright:` | Updates to `(C) [year range] czoTemplate.author` |

#### Changes from all previous versions

These fields are automatically renamed for compatibility with my older file headers:

| Old field | Renamed to |
|---|---|
| `Started:` | `File Created:` |
| `Created:` | `File Created:` |
| `Last Change:` | `Last Modified:` |

### Custom regexp substitutions on save

You can define your own regexp substitutions on every save or update via `czoTemplate.regexps`. These substitutions are applied to the entire document.

Each entry has:

| Field | Description |
|---|---|
| `find` | JavaScript regexp string |
| `replace` | Replacement — supports `$1`, `$2`, ... capture groups and macro tokens |
| `mode` | Optional regexp flags: `i` (case-insensitive), `g` (global) |

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
        },
  ]
```

> **Tip:** To avoid a pattern matching itself in `settings.json` on save, break the literal string with an escaped character, e.g. `"\\$Id\\:"` matches `$Id:` in files but the string `\\$Id\\:` in `settings.json` does not match itself.

#### Macro tokens in `replace`

The following tokens are substituted in the `replace` string before applying the regexp:

| Token | Value |
|---|---|
| `<<STRFTIME="%fmt">>` | Date/time formatted with strftime (e.g. `<<STRFTIME="%Y/%m/%d %T">>`) |
| `<<FILENAME>>` | Current filename (basename only, e.g. `foo.c`) |
| `<<FULLFILENAME>>` | Full path of the current file |
| `<<HOME>>` | Home directory (`$HOME` or `%USERPROFILE%`) |
| `<<USER>>` | Current user (`$USER` or `%USERNAME%`) |
| `<<HOSTNAME>>` | Hostname (`$HOSTNAME`) |


## Custom templates

Any built-in template can be overridden in `settings.json` using `czoTemplate.templates`. The key is the template name (`c`, `cpp`, `h`, `hpp`, `sh`, `py`, ...) and the value is an array of strings, one per line.

Example — override the C and shell templates:

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

Template resolution order:

1. User override - `czoTemplate.templates.languageId` in `settings.json`, for unlisted languageId too
2. User override - `czoTemplate.templates.fileExt` in `settings.json`
3. Built-in - `<extensionPath>/out/templates/template.fileExt`

## License

This extension is released under the [GPL-2.0 License](https://www.gnu.org/licenses/old-licenses/gpl-2.0.html).

This is a port of `template.vim` by Olivier Sirol (`czo@free.fr`), you can see it in my [.vimrc](https://github.com/czodroid/dotfiles/blob/master/.vimrc).

Copyright (C) 2026 Olivier Sirol <czo@free.fr>
