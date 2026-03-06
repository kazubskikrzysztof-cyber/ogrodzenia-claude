# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Projekt

**OgrodzeniePRO** – Progressive Web App (offline-capable SPA) do wyceny montażu ogrodzenia w terenie. Bez żadnych zależności zewnętrznych (vanilla JS). Język UI: polski.

## Uruchamianie

Brak systemu budowania. Otwórz plik HTML bezpośrednio w przeglądarce lub serwuj lokalnie:

```bash
# Python
python -m http.server 8080

# Node.js
npx serve .
```

> PWA i Service Worker wymagają serwera HTTP (nie działa z `file://`). Użyj `localhost`.

## Dwa warianty plików

| Plik | Opis | SW |
|------|------|-----|
| `index_wariantA.html` | One-file: HTML + CSS + JS + SW jako blob URL | Inline (blob) |
| `index_wariantB.html` | Two-file: HTML + CSS + JS + zewnętrzny `sw.js` | `./sw.js` |

**Wariant B jest rekomendowany dla iOS/Safari.** Zmiany logiki/UI należy wprowadzać w obu wariantach lub wybrać jeden jako kanoniczny.

## Architektura

### Stan aplikacji (globalny obiekt `stan`)

```javascript
let stan = {
  klient: { nazwa, adres, telefon, data },
  ustawienia: { dlugoscPanelu_mm: 2500, szerokoscSlupka_mm: 40, jednostka: 'm' },
  zestawy: [],   // [{id, nazwa, dlugoscM}]
  dodatki: { transport, demontaz, brama, furtka, uwagi },
  cennik: { ...CENNIK_DOMYSLNY }
}
```

Persystencja: `localStorage` pod kluczem `stan`. Zapisywany automatycznie przy każdej zmianie. Eksport/import jako JSON.

### Główna logika obliczeniowa

Funkcja `obliczZestaw(dlugoscM)` przelicza metry bieżące na liczbę paneli i słupków:

```
segment = dlugoscPanelu_mm + szerokoscSlupka_mm
liczbaPełnych = floor(długość_mm / segment)
reszta_mm = długość_mm - (liczbaPełnych × segment)
jeśli reszta > 0: +1 panel, +1 słupek
```

### Struktura UI (5 zakładek)

1. **Klient** – dane klienta, parametry panelu/słupka
2. **Zestawy** – lista odcinków ogrodzenia z obliczeniami + wizualizacja SVG
3. **Dodatki** – transport, demontaż, bramy, furtki
4. **Cennik** – edytowalna tabela cen (reset do domyślnych, eksport/import)
5. **Podsumowanie** – raport, druk (CSS `@media print`), kopiowanie jako SMS

### Design System (CSS variables)

Dark high-contrast theme zaprojektowany pod użycie na słońcu (budowa). Zmienne zdefiniowane w `:root`. Minimalna wysokość elementów dotykowych: `--touch: 48px`. Mobile-first, breakpoint `@media (max-width: 380px)`.

### Service Worker (`sw.js`)

Strategia **Cache First** z fallbackiem do `./index.html`. Cache pod nazwą `ogr-v1` – przy aktualizacji zmień wersję, by wyczyścić stary cache.
