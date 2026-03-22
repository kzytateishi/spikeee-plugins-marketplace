# Playwright QA

## Core rules

- Match the viewport to the target Figma frame as closely as possible.
- Wait for fonts and images to load before comparing.
- Disable animations when taking screenshots.
- Prefer locator-level screenshots for section comparisons.
- Full-page screenshots can be used as a secondary check, not as the main fidelity gate.

## Recommended checks

### 1. Section screenshot comparison

```ts
const section = page.locator('[data-qa="hero-section"]');
await expect(section).toHaveScreenshot('hero-section.png', {
  animations: 'disabled',
  scale: 'css',
});
```

### 2. Critical CSS verification

```ts
await expect(section).toHaveCSS('display', 'flex');
await expect(section).toHaveCSS('background-color', 'rgb(255, 255, 255)');
```

### 3. Strict text verification

`toHaveText('...')` normalizes whitespace, so use `textContent` when line breaks or repeated spaces must match exactly.

```ts
const text = await page.locator('[data-qa="hero-title"]').evaluate(el => el.textContent);
expect(text).toBe('Exact Figma copy goes here');
```

### 4. Console error verification

```ts
const consoleErrors: string[] = [];
page.on('console', msg => {
  if (msg.type() === 'error') consoleErrors.push(msg.text());
});
```

Assert at the end of the test that `consoleErrors` is empty.

## Fix priority when differences remain

spacing → position → size → typography → color → radius → border → shadow → copy → assets
