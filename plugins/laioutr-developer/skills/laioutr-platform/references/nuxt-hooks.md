# Nuxt.js Hook Naming Convention

Name hooks as `namespace:entity:action` using colon-separated kebab-case segments — e.g. `frontend-core:link:resolve`, `cms:locale:change`. Use present tense for hooks called before/during an action and past tense for hooks called after — e.g. `app:page:render` vs `app:page:rendered`.
