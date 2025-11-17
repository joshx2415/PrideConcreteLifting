````md
# Project Migration: Nuxt.js to Astro

## 1. Overview & Mission

The goal of this project is to perform a full migration of our current Nuxt.js brochure website to a modern, high-performance Astro-based stack. The primary objectives are to increase site speed, simplify the developer experience, and reduce client-side JavaScript.

This is **not** a "lift and shift." This is a refactor and rewrite. We will be converting all components and pages to the new stack, not just porting them over.

### The Stacks

| Feature | 👎 Old Stack (Nuxt) | 👍 New Stack (Astro) |
| :--- | :--- | :--- |
| **Core Framework** | Nuxt.js (Vue) | [Astro](https://astro.build/) |
| **Styling** | [e.g., Scoped CSS / SCSS] | [Tailwind CSS](https://tailwindcss.com/) |
| **Interactivity** | Vue.js (heavy) | [Alpine.js](https://alpinejs.dev/) (light) |
| **Icons** | [e.g., Font Awesome / Vue Icons] | [Astro Icon](https://github.com/natemoo-re/astro-icon) |
| **Hosting** | Netlify | Netlify (No change) |

---

## 2. New Project Setup

We will start with a fresh Astro project. **Do not** clone the old Nuxt project and try to refactor it.

### Step 1: Create the Astro Project

In your terminal, create the new project.

```bash
# Create a new Astro project
npm create astro@latest [project-name]
````

Use the following settings in the setup wizard:

  * **Project type:** "Include sample files" (Good for a fresh start)
  * **TypeScript:** "Strict"
  * **Install dependencies:** `Yes`
  * **Initialize Git:** `Yes`

### Step 2: Install the Full Stack

This is the most critical step. Use the `astro add` commands to ensure all configurations are set up perfectly.

```bash
# Navigate into your new project
cd [project-name]

# 1. Add Tailwind CSS for styling
npx astro add tailwind

# 2. Add Alpine.js for light interactivity
npx astro add alpinejs

# 3. Add Astro Icon for all SVG icons
npx astro add astro-icon
```

### Step 3: Add Vue Integration (As an "Island")

We need a migration path for complex components. The Astro-Vue integration allows us to use old Vue components as "islands" of interactivity *without* rebuilding them in Alpine immediately.

```bash
# 4. Add Vue for complex components
npx astro add vue
```

### Step 4: Configure Netlify

Create a `netlify.toml` file in the root of the project. This is all we need for a static build.

```toml
# netlify.toml

[build]
  # Astro's build command
  command = "npm run build"

  # The output folder for the static site
  publish = "dist"
```

-----

## 3\. Core Migration Strategy

This is the file-by-file plan. Use the old Nuxt project as a *visual and content reference only*.

### 1\. Asset Migration

  * Copy everything from the old `static/` directory (Nuxt) to the new `public/` directory (Astro). This includes fonts, `favicon.ico`, `robots.txt`, and static images.

### 2\. Layout Conversion

  * Recreate the old Nuxt layouts (e.g., `layouts/default.vue`) as new Astro layouts (e.g., `src/layouts/BaseLayout.astro`).
  * This file will contain the main `<html>`, `<head>`, and `<body>` tags, along with the `<slot />` component where page content will be injected.
  * All `<head>` content (meta tags, font links, etc.) goes here.

### 3\. Page Conversion

  * Go through the old `pages/` directory one file at a time.
  * For each `.vue` page (e.g., `pages/about.vue`), create a new `.astro` page (e.g., `src/pages/about.astro`).
  * The Vue `<template>` content becomes the HTML body of the `.astro` file.
  * The Vue `<script setup>` logic (like fetching data) moves into the Astro "frontmatter" code fence (`---`).
  * Wrap the page's content in your new layout:
    ```astro
    ---
    import BaseLayout from '../layouts/BaseLayout.astro';
    // Any imports or data fetching go here
    ---
    <BaseLayout title="About Us">
      <h1>About Us</h1>
    </BaseLayout>
    ```

### 4\. Component Conversion Strategy (The Core Loop)

This is the most important task. For *every single* component in the old `components/` directory, you must follow this decision tree:

**A. Is the component purely presentational?**

  * *(e.g., `Card.vue`, `SiteHeader.vue`, `Footer.vue`)*
  * **Action:** Rewrite it from scratch as a new `.astro` component (e.g., `src/components/Card.astro`).
  * **Styling:** Use **Tailwind** utility classes directly in the HTML.
  * **Props:** Use `Astro.props` to receive properties.

**B. Does the component have SIMPLE interactivity?**

  * *(e.g., a mobile menu toggle, a dropdown, tabs, an accordion, a modal popup)*
  * **Action:** Rewrite as an `.astro` component, but use **Alpine.js** for the interactivity.
  * **Example (Dropdown):**
    ```astro
    <div x-data="{ open: false }">
      <button @click="open = !open">Toggle</button>
      <div x-show="open">
        Content...
      </div>
    </div>
    ```

**C. Is the component COMPLEX and stateful?**

  * *(e.g., a multi-step contact form, a search filter, anything with complex state)*
  * **Action:** Do **not** rewrite it yet. Use the **Vue Island** strategy.
  * 1.  Move the original `.vue` file into `src/components/`.
  * 2.  In your `.astro` page, import it and render it with a `client:` directive.
    <!-- end list -->
    ```astro
    ---
    import MyComplexForm from '../components/MyComplexForm.vue';
    ---
    <BaseLayout title="Contact">
      <h2>Contact Us</h2>
      <MyComplexForm client:load />
    </BaseLayout>
    ```
  * **Goal:** We want to use this sparingly. The priority is **Astro + Alpine**. This is our escape hatch for complex parts.

### 5\. Styling Migration

  * **This is a 100% replacement.**
  * Remove all old `<style scoped>` tags.
  * Remove all old `.scss` or `.css` file imports.
  * All styling will be done with **Tailwind utility classes** applied directly to the HTML elements.
  * Global design tokens (colors, fonts) should be configured in `tailwind.config.mjs`.

### 6\. Icon Migration

  * **This is a 100% replacement.**
  * Search the old project for the old icon system (e.g., `<font-awesome-icon>`, `<i>`).
  * **Action:** Replace every icon with the `<Icon />` component from **`astro-icon`**.
  * **Example:**
    ```astro
    ---
    import { Icon } from 'astro-icon/components';
    ---
    <button>
      <Icon name="heroicons:arrow-right" class="h-5 w-5" />
      <span>Learn More</span>
    </button>
    ```

-----

## 4\. Cleanup & "No Holdover" Rules

To be clear: this new project should have **zero** Nuxt-specific dependencies.

  * **DO NOT** install: `nuxt`, `vue-router`, `vuex`, `node-sass`, or any Nuxt modules.
  * The **only** `vue` dependency should be `vue` and `@astrojs/vue`, which are required for the Astro Islands (Step 3C).
  * All component logic should be in `.astro` files, Alpine.js directives, or (as a last resort) `.vue` client-side islands.
  * The final `package.json` should be clean and reflect only the new stack.

## 5\. Definition of Done

The migration is complete when:

1.  All pages from the old Nuxt site have been recreated in the new Astro site.
2.  The site builds without errors (`npm run build`).
3.  The site deploys and is fully functional on Netlify.
4.  All styling is handled by Tailwind.
5.  All icons are handled by `astro-icon`.
6.  All interactivity is handled by Alpine.js or, where necessary, Vue islands.
7.  The `package.json` file is free of all Nuxt-related dependencies.

<!-- end list -->

```
```
