# `wuchale`

[![npm version](https://img.shields.io/npm/v/wuchale)](https://www.npmjs.com/package/wuchale) ![License](https://img.shields.io/github/license/K1DV5/wuchale)

A non-invasive compile-time internationalization (i18n) system for Svelte.
Inspired by Lingui, built from scratch with performance, clarity, and
simplicity in mind.

> 🎯 **Smart translations, tiny runtime, full HMR.** Extract strings at build
> time, generate optimized translation catalogs, support live translations
> (even with Gemini AI), and ship minimal code to production.

## Why `wuchale`?

Traditional i18n solutions require you to wrap every translatable string with
function calls or components. `wuchale` doesn't.

```svelte
<!-- Traditional i18n -->
<p>{t('Hello')}</p>
<p><Trans>Welcome {userName}</Trans></p>

<!-- With wuchale -->
<p>Hello</p>
<p>Welcome {userName}</p>
```

Write your Svelte code naturally. No imports, no wrappers, no annotations.
`wuchale` handles everything at compile time.

## ✨ Key Features

- **🔧 Zero-effort integration** - Add i18n to existing projects without rewriting code
- **🚀 Compile-time optimization** - All transformations happen during build, minimal runtime overhead
- **🔄 Full, granular HMR support** - Live updates during development, including auto-translation
- **📦 Tiny footprint** - Only 2 additional dependencies (`wuchale` + `pofile`), no bloated `node_modules`
- **🎯 Smart extraction** - Uses AST analysis: handles nested markup, conditionals, loops, and complex interpolations
- **🌍 Standard .po files** - Compatible with existing translation tools and workflows
- **🤖 Optional AI translation** - Gemini integration for automatic translations during development
- **⚡ Svelte 5 ready** - Built for the future with runes and snippets support

## 🚀 Quick Start

### 1. Install

```bash
npm install wuchale
```

### 2. Configure Vite

```javascript
// vite.config.js
import { svelte } from '@sveltejs/vite-plugin-svelte'
import { wuchale } from 'wuchale'

export default {
    plugins: [
        wuchale(),
        svelte(),
    ]
}
```

### 3. Create Configuration

Create `wuchale.config.js` in your project root:

```javascript
// @ts-check
import { defineConfig } from "wuchale"

export default defineConfig({
    locales: {
        // English included by default
        es: { name: 'Spanish' },
        fr: { name: 'French' }
    },
})
```

### 4. Add CLI Scripts

```jsonc
// package.json
{
  "scripts": {
    "extract": "wuchale",
    "clean": "wuchale --clean"
  }
}
```

### 5. Setup in Your App

#### For SvelteKit (SSR/SSG)

```typescript
// src/routes/+layout.js
import { setTranslations } from 'wuchale/runtime.svelte.js'

export async function load({ url }) {
    const locale = url.searchParams.get('locale') ?? 'en'
    // or you can use [locale] in your dir names to get something like /en/path as params here
    setTranslations(await import(`../locales/${locale}.svelte.js`))
    return { locale }
}
```

#### For Svelte (SPA)

```svelte
<!-- src/App.svelte -->
<script>
    import { setTranslations } from 'wuchale/runtime.svelte.js'
    
    let locale = $state('en')
    
    async function loadTranslations(locale) {
        setTranslations(await import(`./locales/${locale}.svelte.js`))
    }
</script>

{#await loadTranslations(locale)}
    <!-- @wc-ignore -->
    Loading translations...
{:then}
    <!-- Your app content -->
{/await}
```

### 6. Start Coding!

Write your Svelte components naturally. `wuchale` will extract and compile translations automatically:

```svelte
<h1>Welcome to our store!</h1>
<p>Hello {userName}, you have {itemCount} items in your cart.</p>
```

## 📖 How It Works

![Diagram](https://raw.githubusercontent.com/K1DV5/wuchale/main/images/diagram.svg)

### Compilation Process

1. **Extract** - AST traversal identifies translatable text
2. **Transform** - Text nodes replaced with `wuchaleTrans(n)` calls
3. **Catalog** - Updates .po files with new/changed messages
4. **Translate** - Optional Gemini AI translation for new messages
5. **Compile** - Generates optimized JavaScript modules
6. **Bundle** - Vite handles HMR in dev, optimized builds for production

### What Gets Extracted?

#### Markup Text
All text inside elements is extracted by default:
```svelte
<p>This is extracted</p>
<!-- @wc-ignore -->
<p>This is not extracted</p>
```

#### Attributes
Text attributes starting with upper case letters:
```svelte
<img alt="Profile Picture" class="not-extracted" />
```

#### Script
Capitalized strings in specific contexts:

```javascript
// In $derived or functions
const message = $derived('This is extracted')
const lowercase = $derived('not extracted')

// Force extraction with comment
const forced = $derived(/* @wc-include */ 'force extracted')
```
```svelte
<p title={'Extracted'}>{/* @wc-ignore */ 'Ignore this'}</p>
```

## 🔧 Advanced Features

### Nested Content

Complex nested structures are preserved:

```svelte
<p>Welcome to <strong>{appName}</strong>, {userName}!</p>
```

Extracted as:
```
Welcome to <0/>, {0}!
```

### Pluralization

Define your function
```javascript
// in e.g. src/utils.js
export function plural(num, candidates, rule = n => n === 1 ? 0 : 1) {
    const index = rule(num)
    return candidates[index].replace('#', num)
}
```

Use it

```svelte
<script>
    import {plural} from '/src/utils.js'
    let itemCount = 5
</script>

<p>{plural(itemCount, ['One item', '# items'])}</p>
```

### Context

Disambiguate identical texts:

```svelte
<!-- @wc-context: navigation -->
<button>Home</button>

<!-- @wc-context: building -->
<span>Home</span>
```

### AI Translation

Enable Gemini translations by setting `GEMINI_API_KEY`:

```bash
GEMINI_API_KEY=your-key npm run dev
```

## 📁 File Structure

```
src/
├── locales/
│   ├── en.po         # Source catalog (commit this)
│   ├── en.svelte.js  # Compiled data module (gitignore)
│   ├── es.po         # Translation catalog (commit this)
│   └── es.svelte.js  # Compiled data module (gitignore)
└── App.svelte        # Your components
```

## 🛠️ Configuration Reference

```javascript
export default {
    // Source language code
    sourceLocale: 'en',
    
    // Available locales with plural rules
    locales: {
        en: {
            name: 'English',
            // the number of plurals in the language
            nPlurals: 2,
            // The expression to use to decide which candidate to choose when using your plural() function
            // The number should be used as 'n' because this will be the body of an arrow function with n as an argument.
            pluralRule: 'n == 1 ? 0 : 1'
        }
    },
    
    // Where to store translation files
    localesDir: './src/locales',
    
    // Files to scan for translations
    // You can technically specify non svelte js/ts files, but they would not be reactive
    files: ['src/**/*.svelte', 'src/**/*.svelte.{js,ts}'],
    
    // Custom extraction logic
    // signature should be: (text, details: {scope, element, attribute}) => boolean
    heuristic: defaultHeuristic,
    
    // Your plural function name
    pluralFunc: 'plural',
    
    // Enable HMR updates during development. You can disable this to avoid the small overhead
    // of live translation updates and work solely with the source language.
    // HMR is highly optimized -- it updates only the affected components,
    // preserving application state and avoiding full reloads.
    hmr: true,
    
    // Gemini API key (or 'env' to use GEMINI_API_KEY)
    // if it's 'env', and GEMINI_API_KEY is not set, it is disabled
    // set it to null to disable it entirely
    geminiAPIKey: 'env'
}
```

## 🤝 Contributing

Contributions are welcome! Please check out our [test suite](https://github.com/K1DV5/wuchale/tree/main/tests) for examples of supported scenarios.

## ❤️ Sponsors

Thank you **[@hayzamjs](https://github.com/hayzamjs)** for sponsoring the
project and using it in [Sylve](https://github.com/AlchemillaHQ/Sylve), giving
valuable feedback!
