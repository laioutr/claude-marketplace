# A Query Needs an Entity; Everything Else the Frontend Fetches Is an Action

Decide the token kind before anything else. A **query** (`defineQueryToken`) returns ids of an **entity type whose components resolvers fill**: a canonical entity, or one your app defines with `defineEntityComponentToken(..., { entityType })`. A read whose result has no such entity behind it (a scalar, a URL, a value type) is an **action**, even when a page renders it. A relationship between two loaded entities is a **link**. Picking the wrong kind is the most common failure in action design.

## The four orchestr token kinds

| You want to… | Token | Why |
|---|---|---|
| Cause a mutation or side effect (add to cart, subscribe, register, submit, save, delete) | `defineActionToken` | An imperative call. Returns its `output` directly. |
| Read data **on demand** that is not an entity (a checkout URL, the current customer, a balance, a download URL) | `defineActionToken` | The app calls it and uses the result itself. |
| Load **entities** the page renders (the current cart, products in a category, a store by slug) | `defineQueryToken` | Takes a required `entity` + `type`, and Orchestr resolves the ids through component resolvers into the render tree. |
| Traverse from a loaded entity to related ones (`Cart → CartItem`) | `defineLinkToken` | A data-loading path between entities, not an imperative call. |
| Shape a connector-fed field on an entity | `defineEntityComponentToken` | Entity data, not an operation. See the `designing-entity-components` skill. |

## The line between action and query is not "does the page render it"

Canonical `ecommerce/cart/get-current` is a **query**, because `Cart` is an entity. Canonical `ecommerce/customer/get-current` is an **action**. Customer data appears on account pages, but `Customer` is a value type in `@laioutr-core/canonical-types`, not an entity, so there is nothing for Orchestr to resolve. The test is whether the data is an entity type resolved through components, not where or when the read is triggered.

Do not invent a new entity and components just to turn a one-off scalar read into a query. Return it from an action with an object-wrapped output (`{ balance: Money }`).

## An entity behind it is necessary, not sufficient

A query carries only the entity's **intrinsic** components. A read that pairs an entity with **request-scoped facts** (best-selling products *ranked by units sold this month*, an order line with a *period* revenue) has nowhere to put the metric. The query returns ids, and a component resolves the same way for every request, with no period parameter. That leaves two valid shapes:

1. **An action with the facts inlined** in its output (`{ products: [{ productId, unitsSold, revenue }] }`). This is the simplest shape and the best fit for a one-off panel.
2. **A query over a join entity** that carries the facts as its own components plus a link to the real entity, like canonical `ProductListItem → Product`. This is more idiomatic when the result is composed or cached across the app. Designing it is a `designing-entity-components` task.

Do not let "the entity exists, so it must be a query" push you into a query that cannot carry the metric.

## An action rarely ships alone

The frontend usually also has to *render* the action's subject: the appointment just booked, the application's status, the list to pick from. That read side is a query over an entity, designed with `designing-entity-components`. **Scope it alongside the action**, even if you design it separately, so the feature is not half-built. Canonical `b2b/quote/accept` returns nothing because the updated quote is read back through the `Quote` query.

## Re-check at write time

The set of entities moves. Canonical releases add entities, and so may your own app. A `get-all` action designed while no entity existed becomes wrong the day an entity and a link for that data land. Before you write the file, check the installed `node_modules/@laioutr-core/canonical-types/dist/server/entity/` and your own token files again.

## How the frontend calls an action

All of these are auto-imported (`#imports`). See [Actions → Frontend Usage](https://docs.laioutr.com/frontend/orchestr/actions#frontend-usage).

| Composable | Use for |
|---|---|
| `useMutationAction(Token)` | Event-driven calls (a button, a form submit). pinia-colada `useMutation`. |
| `useQueryAction(Token, input?)` | On-demand reads that run on mount and re-run when the input changes. pinia-colada `useQuery`. If the action throws while rendering on the server, the whole page fails, so use `useFetchAction` for a read that may throw. |
| `fetchAction(Token, input?)` | A bare POST with no caching. Returns the decoded output or throws. |
| `useFetchAction(Token, input?)` | Like `fetchAction`, but through `useAsyncData`, so it runs once on the server during SSR. |

Each action is served at `POST /api/orchestr/action/<token-name>`.
