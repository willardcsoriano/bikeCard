# Hands-On Challenge — Create an App Page for bikeCard

**Unit:** 3  
**Prerequisite:** `bikeCard` component deployed to the org (Unit 2).

After completing this challenge, you'll be able to:

- Use Lightning App Builder to place a custom component on an app page.
- Activate an app page for all users.

---

## App Page Spec

| Field | Value |
|---|---|
| **Type** | App Page |
| **Label** | `Bike Card` |
| **API Name** | `Bike_Card` |
| **Template** | One Region |
| **Component** | `bikeCard` |
| **Activation** | Active for all users |

---

## Step 1 — Open Lightning App Builder

**Org UI:**
1. In your Trailhead Playground, click the gear icon (⚙️) → **Setup**.
2. In the Quick Find box, type `Lightning App Builder` and select it.
3. Click **New**.

**VS Code:** `Ctrl+Shift+P` → `SFDX: Open Default Org` to launch the org, then follow the Org UI steps above.

---

## Step 2 — Create the App Page

| Field | Value |
|---|---|
| **Label** | `Bike Card` |
| **API Name** | `Bike_Card` |
| **Type** | App Page |

4. Click **Next**.
5. Choose a page template — pick **One Region**.
6. Click **Finish**.

---

## Step 3 — Add the bikeCard Component

1. In the left sidebar, scroll to the **Custom** section (or search `bikeCard`).
2. Drag the **bikeCard** component onto the page region.
3. Click **Save** in the top-right.

---

## Step 4 — Activate the Page

1. Click **Activation** in the top bar.
2. Keep **Activate for all users** selected.
3. Click **Save**.
4. When asked to add to navigation menus, click **Skip and Save** (reachable from App Launcher anyway).
5. Click **Back** to exit Lightning App Builder.

### Verify

**Org UI:** App Launcher (grid icon ↦) → **Bike Card**.

**VS Code:** `Ctrl+Shift+P` → `SFDX: Open Default Org` → App Launcher → **Bike Card**.

---

## Step 5 — Verify the Challenge

1. In Trailhead, scroll to the challenge section at the bottom of the unit.
2. Click **Check Challenge**.
3. If it fails, double-check:
   - App page label is exactly `Bike Card`
   - API name is exactly `Bike_Card`
   - `bikeCard` component is on the page
   - Page is activated for all users
