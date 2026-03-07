---
name: a11y
description: Accessibilité web — rôles ARIA, navigation clavier, contraste, labels, focus management, semantic HTML. Appliqué automatiquement lors de la création de composants UI.
---

# Skill : Accessibilité (a11y)

Ce skill s'applique automatiquement lors de la création ou modification de composants UI.

## Règles fondamentales

### 1. HTML sémantique avant ARIA

Utiliser les éléments HTML natifs plutôt que des `div` avec des rôles ARIA.

```tsx
// HIGH — div cliquable sans sémantique
<div onClick={handleClick}>Valider</div>

// Correctif
<button onClick={handleClick}>Valider</button>
```

| Besoin | Élément natif | Pas de |
|--------|--------------|--------|
| Bouton | `<button>` | `<div onClick>` |
| Lien | `<a href>` | `<span onClick>` |
| Liste | `<ul>` / `<ol>` | `<div>` imbriqués |
| Navigation | `<nav>` | `<div class="nav">` |
| En-tête | `<h1>`–`<h6>` | `<div class="title">` |
| Formulaire | `<form>` | `<div>` |
| Champ | `<input>` / `<select>` / `<textarea>` | `<div contenteditable>` |

---

### 2. Labels et textes alternatifs

- Chaque `<input>` doit avoir un `<label>` associé (via `htmlFor` ou imbrication)
- Chaque `<img>` doit avoir un `alt` (vide `alt=""` si décoratif)
- Chaque bouton icon-only doit avoir un `aria-label`
- Chaque `<svg>` interactif doit avoir un `aria-label` ou `<title>`

```tsx
// HIGH — input sans label
<input type="email" placeholder="Email" />

// Correctif
<label htmlFor="email">Email</label>
<input id="email" type="email" placeholder="exemple@mail.com" />

// Ou label visually-hidden si le design ne montre pas de label
<label htmlFor="email" className="sr-only">Email</label>
<input id="email" type="email" placeholder="Email" />
```

```tsx
// HIGH — bouton icône sans label accessible
<button onClick={onClose}><XIcon /></button>

// Correctif
<button onClick={onClose} aria-label="Fermer"><XIcon aria-hidden="true" /></button>
```

---

### 3. Navigation clavier

- Tous les éléments interactifs doivent être atteignables au clavier (`Tab`)
- L'ordre de tabulation doit suivre l'ordre visuel (pas de `tabIndex` > 0)
- Les actions doivent fonctionner avec `Enter` et/ou `Space`
- Les menus/dropdowns doivent supporter les flèches directionnelles
- `Escape` doit fermer les modals, dropdowns, popovers

```tsx
// MEDIUM — élément interactif non focusable
<div onClick={handleSelect} className="option">Option 1</div>

// Correctif
<div
  role="option"
  tabIndex={0}
  onClick={handleSelect}
  onKeyDown={(e) => { if (e.key === 'Enter' || e.key === ' ') handleSelect(); }}
>
  Option 1
</div>

// Mieux — utiliser un élément natif
<button onClick={handleSelect}>Option 1</button>
```

---

### 4. Focus management

- Les modals doivent trapper le focus (focus trap)
- À l'ouverture d'un modal, le focus va sur le premier élément interactif
- À la fermeture d'un modal, le focus retourne sur l'élément déclencheur
- Les notifications/toasts doivent utiliser `role="alert"` ou `aria-live="polite"`
- Après suppression d'un élément dans une liste, le focus doit aller sur l'élément suivant

```tsx
// HIGH — modal sans focus trap
<div className="modal">
  <h2>Confirmation</h2>
  <button onClick={onConfirm}>Confirmer</button>
  <button onClick={onClose}>Annuler</button>
</div>

// Correctif — utiliser un dialog natif ou une lib de focus trap
<dialog ref={dialogRef} onClose={onClose}>
  <h2>Confirmation</h2>
  <button onClick={onConfirm}>Confirmer</button>
  <button onClick={onClose}>Annuler</button>
</dialog>
```

---

### 5. ARIA patterns

Utiliser les patterns ARIA standard pour les composants custom :

| Composant | Role | Attributs clés |
|-----------|------|----------------|
| Modal | `dialog` | `aria-modal="true"`, `aria-labelledby` |
| Tabs | `tablist`, `tab`, `tabpanel` | `aria-selected`, `aria-controls` |
| Accordion | `button` + région | `aria-expanded`, `aria-controls` |
| Dropdown | `listbox` ou `menu` | `aria-expanded`, `aria-activedescendant` |
| Toast | `alert` ou `status` | `aria-live="polite"` ou `"assertive"` |
| Breadcrumb | `navigation` | `aria-label="Fil d'Ariane"`, `aria-current="page"` |
| Progress | `progressbar` | `aria-valuenow`, `aria-valuemin`, `aria-valuemax` |

---

### 6. Contraste et lisibilité

- Texte normal : ratio de contraste >= 4.5:1 (WCAG AA)
- Grand texte (>= 18pt ou 14pt bold) : ratio >= 3:1
- Éléments interactifs et icônes : ratio >= 3:1
- Ne pas utiliser la couleur seule pour transmettre une information (ajouter icône, texte, ou pattern)
- Vérifier le mode sombre — les contrastes doivent être respectés dans les deux thèmes

---

### 7. Formulaires

- Grouper les champs liés avec `<fieldset>` et `<legend>`
- Les messages d'erreur doivent être liés au champ via `aria-describedby`
- Indiquer les champs requis avec `aria-required="true"` (ou `required`)
- Les erreurs doivent être annoncées aux lecteurs d'écran (`aria-live` ou `role="alert"`)

```tsx
// Correctif complet
<div>
  <label htmlFor="password">Mot de passe</label>
  <input
    id="password"
    type="password"
    aria-required="true"
    aria-invalid={!!error}
    aria-describedby={error ? "password-error" : undefined}
  />
  {error && (
    <p id="password-error" role="alert">{error}</p>
  )}
</div>
```

---

### 8. Structure de la page

- Une seule balise `<main>` par page
- Hiérarchie de titres cohérente (`h1` → `h2` → `h3`, pas de saut)
- Landmarks : `<header>`, `<nav>`, `<main>`, `<aside>`, `<footer>`
- Les régions de navigation multiples doivent avoir un `aria-label` distinct
- Skip link pour accéder directement au contenu principal

```tsx
// Correctif — skip link
<a href="#main-content" className="sr-only focus:not-sr-only">
  Aller au contenu principal
</a>
<main id="main-content">...</main>
```

---

### 9. Médias et animations

- Les vidéos doivent avoir des sous-titres
- Les animations doivent respecter `prefers-reduced-motion`
- Les carrousels doivent être pausables et navigables au clavier

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

## Classe utilitaire sr-only (screen-reader only)

```css
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}
```

Tailwind CSS : utiliser la classe `sr-only` (incluse par défaut).

## Application

Lors de la création ou modification d'un composant UI :

1. Vérifier la sémantique HTML — est-ce le bon élément ?
2. Vérifier les labels et textes alternatifs.
3. Tester la navigation clavier (`Tab`, `Enter`, `Escape`, flèches).
4. Vérifier le focus management pour les éléments dynamiques (modals, dropdowns).
5. Ne pas sur-utiliser ARIA — les éléments natifs n'en ont pas besoin.
