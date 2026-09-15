# i18n - Ecko Std Lib Package

Message catalogues, `{name}` interpolation, and plural rules for the languages
that need more than two forms.

## Install

```bash
ecko get github.com/ecko-lang/i18n
```

```ecko
import i18n
```

Needs `fs:read` for `load`. The `catalog` form needs no capabilities.

## Usage

`locales/fr.json` is ordinary JSON, editable by a translator and readable in a
diff:

```json
{
  "greeting": "Bonjour, {name}",
  "items": { "one": "{n} article", "other": "{n} articles" }
}
```

```ecko
import i18n

t = i18n.load("locales/fr.json")

i18n.tr(t, "greeting", { name: "Ada" })   # "Bonjour, Ada"
i18n.plural(t, "items", 3)                # "3 articles"
```

The language is taken from the filename, so `fr.json` is French and
`pt-BR.json` is Brazilian Portuguese. Pass it explicitly if your files are named
some other way.

When the messages are embedded or fetched rather than read from disk, build the
catalogue directly and skip the capability:

```ecko
t = i18n.catalog({ greeting: "Hello, {name}" }, "en")
```

## Plurals are the reason this exists

English has two forms and most software stops there, which is why so much of it
says "1 results" in other languages.

Russian has three, and the rule is **not** `n == 1`:

```ecko
i18n.plural(ru, "items", 1)    # 1 товар
i18n.plural(ru, "items", 2)    # 2 товара
i18n.plural(ru, "items", 5)    # 5 товаров
i18n.plural(ru, "items", 21)   # 21 товар     <- same form as 1
i18n.plural(ru, "items", 11)   # 11 товаров   <- not
```

Entries are keyed by CLDR category (`zero`, `one`, `two`, `few`, `many`,
`other`), so a catalogue written against CLDR works unchanged. `{n}` is filled
with the count unless you pass another value for it, which is what you want for
a formatted number:

```ecko
i18n.plural(t, "items", 1000, { n: "1,000" })   # "1,000 items"
```

Only `other` is required. A catalogue carrying just that form is valid and is
used for every count, which is what CLDR itself expects.

**Rules included:** English and the languages that share its pair; French and
Portuguese (which count zero as singular); Russian and Ukrainian; Polish; Czech
and Slovak; Arabic (all six categories); and Japanese, Chinese, Korean,
Vietnamese, Thai, Indonesian and Malay, which make no distinction at all.
Anything else falls back to the English pair - a guess, but a documented one.

## Missing things are errors

A missing key raises rather than echoing the key back, and a missing placeholder
raises rather than leaving a gap. Both are deliberate: a half-rendered sentence
or a bare `items.count` on a page looks intentional and survives review, where
an error fails the test that should have caught it.

`has(cat, key)` answers without raising, for a deliberate fallback to another
language.

## API

| call | what it does |
|---|---|
| `load(path, lang?)` | A catalogue from a JSON file. Needs `fs:read`. |
| `catalog(messages, lang)` | A catalogue from a map you already have. |
| `tr(cat, key, vars?)` | The message for `key`, placeholders filled. |
| `plural(cat, key, n, vars?)` | The form `n` selects, with `{n}` filled. |
| `has(cat, key)` | Whether the key is present, without raising. |

## Notes

`{{` and `}}` are literal braces.

Interpolation is done here rather than with Ecko's own `{expr}`, because a
catalogue is read at runtime long after the program was parsed. It also means a
translated string can never execute anything.

## Testing

```bash
ecko test tests/
```

## License

MIT
