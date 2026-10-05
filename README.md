# colors

Kolory w terminalu dla H# — odpowiednik rustowego `colored` / `owo-colors`, jako biblioteka dla [bit](https://github.com/bit-io/bit).

* **164 gotowe kolory** — 16 klasycznych kolorów terminala + 148 nazwanych kolorów truecolor (140 nazw CSS/HTML + 8 dodatkowych)
* każdy kolor jako **tekst** (`colors::fg::coral`) i jako **tło** (`colors::bg::coral`)
* style: bold, dim, italic, underline, strike, reverse… + builder `colors::style::new()`
* kolor po nazwie, hexie, RGB, HSL, indeksie ANSI-256
* gradienty i tęcza, gotowe motywy (`fire`, `ocean`, `sunset`…)
* style komunikatów CLI: `success`, `error`, `warning`, `info`, `hint`, odznaki
* automatyczne wykrywanie terminala (`NO_COLOR`, `TERM=dumb`, `COLORTERM`…) i **automatyczne zubażanie** kolorów
  (truecolor → 256 → 16), gdy terminal nie umie więcej
* zagnieżdżanie działa: `red("a " + bold("b") + " c")` — „ c" nadal jest czerwone
* czysty H#, bez zależności

## Instalacja

W `Bit.hk` projektu:

```
[dependencies]
-> colors => newest
```

(`bit install`), a w kodzie:

```
use "bit -> colors"
```

Lokalnie, z katalogu obok: `-> colors => path ../colors`.

## Szybki start

```
use "bit -> colors"

fn main() is
    write(colors::red("error") + ": " + colors::bold("coś poszło nie tak"))
    write(colors::fg::coral("koralowy tekst"))
    write(colors::bg::navy(colors::fg::gold(" złoto na granacie ")))
    write(colors::paint("po nazwie", "dark orange"))
    write(colors::hex("po hexie", "#00c2a8"))
    write(colors::gradient::rainbow("tęcza!"))
end
```

## API

| Co | Wywołanie |
|---|---|
| 16 kolorów terminala | `colors::red(s)`, `colors::bright_blue(s)` … (`black red green yellow blue magenta cyan white` + `bright_*`) |
| atrybuty | `colors::bold` `dim` `italic` `underline` `blink` `reverse` `strike` |
| każdy kolor z palety (tekst) | `colors::fg::<nazwa>(s)` — np. `colors::fg::tomato("x")` |
| każdy kolor z palety (tło) | `colors::bg::<nazwa>(s)` |
| po nazwie / hexie | `colors::paint(s, "coral")`, `colors::on(s, "navy")`, `colors::hex(s, "#ff8000")`, `colors::on_hex(s, "f80")` |
| RGB / HSL / ANSI-256 | `colors::rgb(s, r, g, b)`, `colors::on_rgb(...)`, `colors::hsl(s, h, s, l)`, `colors::ansi256(s, n)` |
| builder | `colors::style::new().fg("coral").bg("navy").bold().underline().paint(s)` |
| gradienty | `colors::gradient::text(s, "crimson", "gold")`, `text3`, `background`, `bar(width, a, b)`, `rainbow` |
| motywy gradientów | `colors::gradient::fire` `ocean` `sunset` `forest` `pastel` |
| komunikaty CLI | `colors::theme::success` `error` `warning` `info` `hint` `debug` `step` `title` `muted` `link` `code` `badge` |
| tekst z kolorami | `colors::strip(s)`, `colors::util::visible_len(s)`, `pad_right`, `pad_left`, `center` |
| konwersje | `colors::convert::from_hex` `to_hex` `from_hsl` `mix` `lighten` `darken` `invert` `to_ansi256` `to_ansi16` |
| wykrywanie | `colors::support::level()` (0 brak · 1 = 16 · 2 = 256 · 3 = truecolor), `enabled()`, `set_level(n)`, `disable()`, `force()`, `auto()` |
| paleta | `colors::palette::all_names()`, `lookup(name)`, `colors::color_count()` |
| podgląd | `colors::showcase::print_all()` — wypisuje całą paletę |

Nazwy kolorów są niewrażliwe na wielkość liter i separatory: `"Dark Orange"`, `"dark_orange"`, `"darkorange"` to to samo.
Nieznana nazwa nie psuje programu — tekst wraca bez zmian.

### Lista kolorów

16 kolorów terminala: `black red green yellow blue magenta cyan white` oraz `bright_black … bright_white`.
(Te osiem nazw zawsze oznacza kolor z motywu terminala — użytkownik widzi swoje własne odcienie.)

Pozostałe 148 to truecolor: 140 standardowych nazw CSS (`aliceblue` … `yellowgreen`, w tym `coral`, `crimson`, `gold`,
`indigo`, `navy`, `orchid`, `salmon`, `tomato`, `rebeccapurple`…; osiem nazw kolidujących z kolorami terminala jest wyżej) oraz 8 dodatkowych: `amber mint peach rose sand lilac slate charcoal`.
Pełną listę zobaczysz uruchamiając `examples/showcase`.

## Wykrywanie kolorów

Kolejność (pierwszy pasujący wygrywa):

1. `COLORS_LEVEL=0..3` — jawne wymuszenie (to samo robi `colors::support::set_level`)
2. `NO_COLOR` (niepuste) — bez kolorów
3. `FORCE_COLOR` / `CLICOLOR_FORCE` — kolory włączone nawet do pipe'a
4. `CLICOLOR=0` — bez kolorów
5. brak `TERM` albo `TERM=dumb` — bez kolorów
6. `COLORTERM=truecolor|24bit` albo znany terminal (iTerm, WezTerm, kitty, alacritty, VS Code, Windows Terminal…) — poziom 3
7. `TERM` zawiera `256` — poziom 2
8. w pozostałych przypadkach — poziom 1

Uwaga: H# nie ma jeszcze natywnego `isatty`, więc przy przekierowaniu do pliku w terminalu z ustawionym `TERM`
kolory zostaną wypisane. Użyj `NO_COLOR=1` albo `colors::disable()`, gdy to ważne.

## Układ plików

```
colors/
├── Bit.hk
├── src/
│   ├── lib.h#               fasada: deklaracje `mod` + krótkie skróty (colors::red, colors::paint…)
│   ├── colors_support.h#    wykrywanie poziomu kolorów
│   ├── colors_convert.h#    hex / HSL / mix / konwersja do 256 i 16 kolorów
│   ├── colors_palette.h#    nazwy → wartości            (GENEROWANY)
│   ├── colors_ansi.h#       silnik: sekwencje SGR, zagnieżdżanie, zubażanie
│   ├── colors_fg.h#         colors::fg::<nazwa>          (GENEROWANY)
│   ├── colors_bg.h#         colors::bg::<nazwa>          (GENEROWANY)
│   ├── colors_style.h#      atrybuty + builder
│   ├── colors_theme.h#      komunikaty CLI, odznaki
│   ├── colors_gradient.h#   gradienty, tęcza, motywy
│   ├── colors_util.h#       strip, visible_len, pad_*
│   └── colors_showcase.h#   podgląd palety
├── tools/gen_palette.py     jedna tabela kolorów → trzy wygenerowane pliki
├── selftest/                testy (osobny projekt zależny od colors)
└── examples/                demo i showcase
```

Nazwy plików modułów zaczynają się od `colors_`, bo H# nazywa funkcje modułów po nazwie `mod`-a — dzięki temu
`colors::fg::coral` to w środku `colors_fg_coral` i nie koliduje z modułami w Twoim programie.

### Dodawanie koloru

Dopisz wiersz do tabeli w `tools/gen_palette.py`, a potem:

```
python3 tools/gen_palette.py
```

## Testy

```
cd selftest
bit test        # albo: bit run
```

Testy pinują poziom kolorów, więc wynik nie zależy od terminala. Sprawdzają m.in. kody dla każdego poziomu,
konwersje hex/HSL, zagnieżdżanie, `strip`/`visible_len` oraz to, że **każda** nazwa z palety daje poprawny tekst i tło.

## Licencja

MIT
