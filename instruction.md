# Code Standards, Architecture & UX Guidelines

This document outlines the core guidelines for building user interfaces, structuring code, maintaining architectural integrity, and delivering premium user experiences. Follow these standards consistently across all features and projects.

---

## 1. General Development & Architecture Principles
To build robust, enterprise-grade applications, adhere to clean code practices:

* **Production-Ready Code**: Only build and commit production-grade code. Avoid placeholders, partial implementations, or temporary "todo" workarounds unless explicitly agreed.
* **Maintainability & Performance**: Prioritize code readability, scalability, high performance (avoiding memory leaks/costly redraws), and ease of maintenance.
* **SOLID Principles**: Adhere strictly to SOLID principles and clean, decoupled architecture.
* **Single Responsibility Principle (SRP)**: Keep components, services, and utility helpers focused on a single responsibility.
* **DRY (Don't Repeat Yourself)**: Avoid duplicate code. Extract repeated logic into shared services, pipes, directives, or helper utilities.
* **Composition over Duplication**: Always prefer building complex UI by composing simple, modular components rather than duplicating layout logic.

---

## 2. File Organization & Folder Structure
Keep the repository neat and easy to navigate:

* **Consistent Naming Conventions**: Files, classes, styles, and assets must follow consistent naming rules matching the project's framework standards (e.g., kebab-case for filenames, camelCase/PascalCase for class properties/types).
* **Feature-Based Grouping**: Group files by feature (e.g., `features/auth/sign-in/`) rather than by technical type (e.g., placing all HTML files in one folder and TS files in another) whenever possible. This isolates domain logic and enhances modularity.
* **Respect Existing Structure**: Consistently follow the established directory layout without introducing arbitrary folder trees.

---

## 3. Design System, Theme Compliance & Styling Uniformity
Consistency is key to a premium product. Every page, card, input, and button must look cohesive.

* **Unified Design Language**: Every page must follow the same design language.
* **Design System Usage**: Use the existing design system throughout the entire application. Do not introduce new styles unless absolutely necessary.
* **Mandatory Theme Variables**: Always use theme variables.
* **Strict Anti-Hardcoding Rules**:
  * **Never** hardcode colors (always use variables like `var(--color-primary-800)`).
  * **Never** hardcode font sizes (always use variables like `var(--text-sm)` or `var(--text-base)`).
  * **Never** hardcode spacing values (always use variables like `var(--space-2)` or `var(--space-4)` for paddings, margins, and gaps).
  * **Never** hardcode border radii (always use variables like `var(--radius-md)`).
  * **Never** hardcode shadows (always use variables like `var(--shadow-md)`).
* **Example CSS Usage**:
  ```css
  .custom-card {
    background-color: var(--color-surface);
    border: 1.5px solid var(--color-border);
    border-radius: var(--radius-md);
    box-shadow: var(--shadow-md);
    font-family: var(--font-family);
    padding: var(--space-4);
    font-size: var(--text-base);
  }
  ```

---

## 4. Reusable UI Components First
Before creating any component, always check whether one already exists. Follow this strict priority order:

| Priority | Action |
|----------|--------|
| **1st** | Use an **existing shared component** as-is |
| **2nd** | **Extend** an existing component (via inputs, slots, or configuration) |
| **3rd** | **Create a new reusable component** in the shared directory |
| **4th** | Create a **feature-specific component** only if it absolutely cannot be reused anywhere else |

* **Rule**: Always search the shared components directory (e.g., `src/app/shared/components/` or equivalent) for existing UI blocks (Buttons, Inputs, Modals, Checkboxes, etc.) before writing custom HTML markup.
* **Collaboration**: If a required component does not exist and a new one must be created, **inform the project owner before building it** to align on its API, placement, and reusability.

---

## 5. Zero Layout Shifting (CLS Prevention)
Layout shifts look unpolished and cause misclicks. Form validation states must never alter the dimensions of the parent layout or input fields.

* **Rule**: Showing or hiding validation messages must **never** shift the heights/widths of inputs, push buttons down, or resize container cards.
* **Mechanism**: Use inline warning icons next to field labels with hover-triggered tooltips instead of rendering static error text blocks underneath the fields.

---

## 6. Modern Validation & UX Patterns
When validation errors occur, implement the following tooltip and interactive patterns:

### A. Inline Warning Icons with Tooltips
* **Rule**: Place a warning/info icon (e.g., FontAwesome `fa-circle-exclamation`) directly inside the label header on error.
* **Tooltip**: Use a CSS-only hover tooltip anchored to the icon containing the warning message.

#### HTML Structure:
```html
<div class="label-container">
  <label class="field-label label-error">Password</label>
  <span class="label-error-icon-wrapper" data-tooltip="Password must be at least 8 characters long">
    <i class="fa-solid fa-circle-exclamation error-icon"></i>
  </span>
</div>
```

#### Global CSS (Tooltip Engine):
```css
.label-container {
  display: flex;
  align-items: center;
  gap: 8px;
}

.label-error-icon-wrapper {
  position: relative;
  display: inline-flex;
  cursor: pointer;
  color: var(--color-danger-500, #ef4444);
  font-size: 0.9rem;
  line-height: 1;
}

/* Tooltip Bubble */
.label-error-icon-wrapper::after {
  content: attr(data-tooltip);
  position: absolute;
  bottom: 140%;
  left: 50%;
  transform: translate(-50%, 8px) scale(0.9);
  background-color: var(--color-neutral-900, #0f172a);
  color: var(--color-neutral-0, #ffffff);
  padding: 6px 12px;
  border-radius: 6px;
  font-size: 0.75rem;
  font-weight: 500;
  white-space: nowrap;
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.15s ease, transform 0.15s ease;
  z-index: 999;
}

/* Tooltip Arrow */
.label-error-icon-wrapper::before {
  content: '';
  position: absolute;
  bottom: 120%;
  left: 50%;
  transform: translate(-50%, 4px);
  border-width: 5px;
  border-style: solid;
  border-color: var(--color-neutral-900, #0f172a) transparent transparent transparent;
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.15s ease;
  z-index: 999;
}

/* Hover States */
.label-error-icon-wrapper:hover::after {
  opacity: 1;
  transform: translate(-50%, 0) scale(1);
}
.label-error-icon-wrapper:hover::before {
  opacity: 1;
}
```

### B. Reactive Error Clearing (Type-to-Clear)
* **Rule**: Red borders and error tooltips must clear immediately when the user corrects their input. Do not wait for another form submission to clear visual errors.
* **Mechanism**: Listen to input change events and reset the validation states as soon as the criteria (e.g. valid email, length requirement) are met.

#### TypeScript Pattern:
```typescript
protected onEmailChange(val: string): void {
  this.email.set(val);
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (val.trim() && emailRegex.test(val)) {
    this.emailState.set('normal');
    this.emailMsg.set('');
  }
}
```

### C. Separation of Global and Field-Level Errors
* **Rule**: Keep global alerts (e.g. banners at the top of forms) reserved strictly for backend/API messages (e.g. network failure, invalid login credentials).
* **Rule**: Do not display duplicate versions of field-level errors (like "Terms must be accepted") in top-of-form global banners.

---

## 8. Forms
All forms in the application must follow a consistent, production-ready standard for UX, accessibility, and safety.

* **Reusable Form Components**: Always use the shared form components (e.g., `app-input`, `app-checkbox`, `app-button`) for all form fields. Never build raw inputs directly in feature templates.
* **Consistent Validation Behavior**: Validate all fields on submit. Clear individual field errors reactively as the user corrects their input (type-to-clear pattern).
* **Consistent Validation Messages**: Show validation messages using the standard **label tooltip icon** pattern (see Section 7). Never use block error text below fields that shifts layout.
* **Required Field Indication**: Clearly indicate required fields visually (e.g., asterisk `*` in label or `required` attribute) so users know what is mandatory before submitting.
* **Disable Submit While Processing**: The submit button must be disabled and show a loading spinner/indicator while an async operation (e.g., API call) is in progress.
* **Loading States**: Always show a visible loading state on the submit button using the component's `[loading]` input. Never leave the UI frozen without feedback.
* **Prevent Duplicate Submissions**: Guard against multiple submissions by setting a `loading` signal to `true` on submit and only resetting it once the API call completes (success or error).

---

## 9. Tables
All tables must be consistent in structure, behavior, and appearance throughout the application. Use the shared table component wherever one exists.

* **Do not implement custom table styles** unless there is a specific, justified requirement that the shared component cannot satisfy.

### Required Consistent Features:
| Feature | Requirement |
|---------|-------------|
| **Header Styling** | Consistent font weight, background, and border across all tables |
| **Row Spacing** | Uniform row padding and divider style on every table |
| **Pagination** | All paginated tables must use the same shared pagination control |
| **Search** | Search inputs must follow the shared input component and placeholder convention |
| **Filtering** | Filters must be visually consistent (same dropdown/chip style) across all tables |
| **Sorting** | Sortable columns must display a consistent sort indicator icon |
| **Empty State** | Always render a meaningful empty state (icon + message) when there is no data |
| **Loading State** | Always render a skeleton or spinner while data is being fetched |
| **Error State** | Always render a clear error message with a retry option when the fetch fails |
| **Responsive Behavior** | Tables must adapt gracefully on smaller screens (horizontal scroll or stacked layout) |

---

## 10. Icons
Icons must be visually uniform and sourced from a single library throughout the application.

* **Single Icon Library**: Use only one icon library (e.g., FontAwesome). Do not mix icon packs unnecessarily.
* **Consistent Sizing**: Icon sizes must align with the surrounding text or context. Never arbitrarily resize icons with inline styles — use theme-based size classes or CSS variables.
* **Semantic Usage**: Use icons to support meaning, not for decoration alone. Always pair icons with accessible labels (`aria-label` or `aria-hidden="true"` with adjacent visible text).

---

## 11. Typography
All text across the application must use the design system's typography scale exclusively. Never define custom font sizes, weights, or line heights outside of the design tokens.

* **Use design system typography only**: Do not introduce arbitrary `font-size`, `font-weight`, or `line-height` values. Always use the established type scale variables.

### Required Consistency by Text Role:

| Role | Variable | Usage |
|------|----------|-------|
| **Page Titles** | `var(--text-4xl)` / `var(--font-extrabold)` | Main `<h1>` heading on every page |
| **Section Titles** | `var(--text-2xl)` / `var(--font-bold)` | `<h2>` and `<h3>` section headings |
| **Labels** | `var(--text-sm)` / `var(--font-bold)` | Form field labels and data labels |
| **Helper Text** | `var(--text-xs)` / `var(--font-normal)` | Subtext, hints, or descriptions below labels |
| **Error Messages** | `var(--text-xs)` / `var(--font-bold)` | Validation tooltips and API error messages |
| **Body Text** | `var(--text-base)` / `var(--font-normal)` | Paragraphs, descriptions, and table cell content |

---

## 12. Page States
Every page must handle all possible states. Never leave the user without visual feedback.

| State | Requirement |
|-------|-------------|
| **Loading** | Show a skeleton or spinner while data is being fetched |
| **Empty** | Show a meaningful empty state with an icon and descriptive message |
| **Error** | Show a user-friendly error message with a retry action where possible |
| **Success** | Confirm successful actions with a clear, non-intrusive feedback (toast, alert, or redirect) |
| **Unauthorized** | Redirect to the login page or show a clear "session expired" message |
| **No Permission** | Show a "you don't have permission" page or section — never show a blank or broken UI |

> **Rule**: Never leave users without feedback. Every async operation and route must account for all six states above.

---

## 13. API Layer
Keep data-fetching logic cleanly separated from the UI.

* **Services Only**: Keep all API calls inside dedicated service files. Never call APIs directly from components or templates.
* **Central Error Handling**: Handle errors centrally using interceptors — avoid scattering `try/catch` or `.catch()` logic across every component.
* **Standardized Request/Response**: Standardize how requests are built and how responses are parsed (e.g., unwrapping a common `{ data, message }` envelope).
* **Reuse Interceptors**: Use HTTP interceptors for cross-cutting concerns such as auth token injection, global error handling, and request logging.
* **Focused Services**: Keep services focused on a single domain (e.g., `UserService`, `SchoolService`). Do not create catch-all services.

---

## 14. Error Handling
Errors must be handled gracefully at every layer — never expose raw technical details to the user.

* **User-Friendly Messages**: Always translate error codes and server messages into clear, readable language (e.g., "Something went wrong. Please try again." instead of a raw stack trace).
* **No Raw Server Errors**: Never display raw API error objects, status codes, or exception messages directly in the UI.
* **Appropriate Logging**: Log technical errors to the console (dev) or a monitoring service (prod) for debugging — separately from what the user sees.
* **Graceful Failures**: Always provide a fallback UI or recovery path when unexpected failures occur. The application must never crash silently or freeze.

---

## 15. Performance
Write code that is fast, efficient, and scales gracefully with data volume.

* **Lazy Load Feature Modules**: Load feature modules on demand — never eagerly import everything at application startup.
* **Lazy Load Heavy Components**: Defer loading of modals, charts, or large components until they are needed.
* **Avoid Unnecessary API Calls**: Cache responses where appropriate. Do not re-fetch data that is already available or hasn't changed.
* **Minimize Change Detection Work**: Prefer `OnPush` change detection strategy. Avoid triggering full component tree re-renders unnecessarily.
* **Optimize Rendering**: Avoid complex expressions directly in templates. Move computed logic to signals, `computed()`, or getters.
* **Reuse Computed Values**: Use `computed()` signals or memoized selectors instead of recalculating derived values on every render cycle.
* **Debounce Search Inputs**: Always debounce search and filter inputs (minimum 300ms) to avoid firing API calls on every keystroke.
* **Paginate Large Datasets**: Never load unbounded lists. Always paginate or virtualize lists with more than ~50 items.

---

## 16. Documentation
Write documentation where it genuinely adds value. Do not over-document — avoid commenting on code that is already self-explanatory.

**Document the following:**
* **Complex business logic** — explain the *why*, not just the *what*.
* **Public APIs** — describe inputs, outputs, side effects, and expected usage.
* **Shared utilities** — explain the purpose, parameters, and return values of any shared helper or pipe.
* **Reusable components** — document all public `@Input()` / `@Output()` properties with descriptions and accepted values.

> **Rule**: Avoid documenting obvious code. A well-named variable or function should not need a comment to explain itself.

---

## ✦ Golden Rule
Before implementing any new feature, pause and ask yourself these seven questions:

| # | Question |
|---|----------|
| 1 | **Can this be reused?** — Would another feature or page benefit from this? |
| 2 | **Does a similar component already exist?** — Have I checked the shared directory? |
| 3 | **Does it follow the application's design system?** — Am I using only theme tokens? |
| 4 | **Is it consistent with the rest of the application?** — Does it look and behave like everything else? |
| 5 | **Will another developer immediately understand it?** — Is it readable without explanation? |
| 6 | **Is this scalable for future enhancements?** — Can new requirements be added without rewriting? |
| 7 | **Would this meet production-quality standards without further refactoring?** — Is it truly done? |

> If the answer to any of these is **no**, reconsider and improve before proceeding.
