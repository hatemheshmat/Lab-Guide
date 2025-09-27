# Lab-Guide
# Lab-Guide

# 🔹 Part 1/2 — Project Setup (Meta XR SDK only, Quest 2)

## ✅ To-Do Checklist (with why)

### 1) Create the project

* ⬜ **Unity Hub → New → 3D (URP)**

  * • URP is the most stable + performant path on Quest.

### 2) Switch to Android

* ⬜ **File → Build Settings → Android → Switch Platform**

  * • Quest is Android; switching now avoids later re-imports.
* ⬜ **Texture Compression = ASTC**

  * • Better memory/quality tradeoff on Quest.

### 3) Player Settings (Project Settings → Player → Other Settings)

* ⬜ **Package Name** `com.company.app`

  * • Required for Android build/installation.
* ⬜ **Min API Level ≥ 29 (Android 10)**

  * • Matches current Meta requirements.
* ⬜ **Scripting Backend = IL2CPP**, **ARM64 only**

  * • Needed for store submission + best performance.
* ⬜ **Graphics API = Vulkan** (uncheck GLES3 unless you have a reason)

  * • Usually faster on Quest 2 with modern SDKs.

### 4) Install Meta XR packages (no OpenXR)

* ⬜ **Window → Package Manager → Unity Registry**

  * ✅ Install **XR Plug-in Management**
  * ✅ Install **Meta XR** (a.k.a. Oculus) **plugin provider**
  * ✅ Install **Meta XR All-in-One SDK** (v65+)
* ⬜ **Project Settings → XR Plug-in Management → Android**

  * ✅ **Enable Meta XR**
  * ❌ **Do NOT enable OpenXR** (we’re Meta-only here)
  * • Keeps the stack simple and avoids mixed-runtime warnings.

### 5) Let Meta’s “Project Validation” auto-fix

* ⬜ **Edit → Project Settings → XR Plug-in Management → Project Validation** → **Fix All**

  * • Applies recommended flags (permissions, input backends, etc.).

### 6) URP asset tuning (Project view → select your **Mobile_RPAsset**)

* ⬜ **HDR = OFF**
* ⬜ **MSAA = 4x**
* ⬜ **Shadow Resolution = Medium (512)**, **Distance = 20–25**, **Cascades = 2**
* ⬜ **Disable Bloom/Motion Blur/Vignette**

  * • Guarantees a solid baseline for 72–90 FPS on Quest 2.

### 7) URP renderer tuning (select **Mobile_Renderer**)

* ⬜ **Post-processing = OFF**
* ⬜ **Transparent Receive Shadows = OFF**
* ⬜ **Renderer Features = (empty)**

  * • Removes heavy passes that hurt mobile VR.

### 8) Meta-specific runtime tuning (Meta XR provider)

* ⬜ **Stereo Rendering Mode = Single-Pass Instanced**

  * • Renders both eyes in one pass (large GPU win vs Multi-Pass).
* ⬜ **Foveated Rendering = Enabled (conservative)** *(optional)*

  * • Big perf gain; watch for peripheral blur.
* ⬜ **Phase Sync / ASW** *(optional, per app comfort)*

  * • Improves frame pacing; test with your content.

### 9) Silence common OVR warnings (optional)

* ⬜ If you see micro-gesture logs: **disable Microgestures** (or update Meta SDK 65+)

  * • Those “ActionSet not attached” warnings are harmless but noisy if you’re not using the feature.

---

# 🔹 Part 2/2 — Grabbing (Meta Interaction SDK “Building Blocks”)

Below you’ll set up grabbing **the easy way** (wizard) and know **why** each piece exists.
You can still build manually later—this follows how Meta structures the rig in v65+.

## ✅ To-Do Checklist (with why)

### 1) Add the ready camera rig & interactions (one click)

* ⬜ **Oculus/Meta → Tools → Building Blocks → Grab Interaction** *(or)*
  **Assets/Samples/Meta XR…/Interaction SDK → Prefabs → Camera Rig / Interactions**

  * • Creates **Camera Rig (OVRCameraRig + OVRManager)**, **Tracking Space**, **Hand Interactions** (L/R), **Controller Interactions** (L/R).
  * • Hands & controllers come with the **right interactors** already wired for grabbing.

> If you prefer minimalism, you can delete the demo cube—they add one so you can test fast.

---

### 2) Create your own Grabbable (from scratch)

* ⬜ **Create Cube** → **Reset** (scale `0.1, 0.1, 0.1`) → move slightly up/forward

  * • A tiny cube is easy to test, won’t clip the floor.
* ⬜ **Add Rigidbody** → *(often)* **Use Gravity OFF**, **Is Kinematic ON**

  * • Many “physics-assisted” grab patterns drive pose directly; kinematic avoids unstable physics while held.
* ⬜ **Right-click the cube → Interaction SDK → Add Grab Interaction** (Wizard)

  * Choose **Interactors**: Hands, Controllers, or **Both**
  * Choose **Grab Types**: **Pinch**, **Palm**, or **Both**
  * Let the wizard **generate collider** / **fix missing rigidbody** if needed
  * Click **Create**
  * • The wizard adds:

    * **Grabbable** (core state machine for selection)
    * **Hand Grab Interactable** (for hands)
    * **Grab Interactable** (for controllers)

**Why this split?**

* *Hand Grab Interactable* supports hand poses, poke/touch, and nuanced affordances.
* *Grab Interactable* is tailored to controllers (grip/trigger), distance rays, etc.
* Having both lets players grab with either input modality.

---

### 3) One-hand grab (basic)

* ⬜ On the cube’s **Grabbable** → add **Grab Free Transformer**
* ⬜ In **Grabbable → Optionals**: set **One Grab Transformer = Grab Free Transformer**
* ⬜ *(Optional)* Uncheck **Transfer On Second Selection**

  * • Prevents auto-handoff when the other hand touches it.
* **Concept:** “Transformer” decides how the object moves/rotates/scales while selected.

---

### 4) Two-hand grab

* ⬜ Keep **Grab Free Transformer**
* ⬜ In **Grabbable → Optionals**: set **Two Grab Transformer = Grab Free Transformer**
* ⬜ Uncheck **Transfer On Second Selection**

  * • Requires both hands to stay on it (good for heavy props).

---

### 5) One-hand grab + two-hand **scaling**

* ⬜ **Grab Free Transformer** in **both** fields (One & Two Grab Transformer)
* ⬜ In transformer: set **Min/Max Scale**, **Constraints Are Relative = ON**

  * • Lets player scale by widening/narrowing hands; relative keeps ratios consistent.

---

### 6) **Constrain to a plane** (slide on a surface)

* ⬜ **Grab Free Transformer** → lock **Rotation (all)**, allow **Position** only on axes you want (e.g., only X & Z)
* ⬜ **Constraints Are Relative = ON**

  * • Great for sliders/table-top items; object won’t drift off plane.

---

### 7) **Constrain to an axis** (door hinge)

* ⬜ Create parent **Door** GO; the interactable is a child
* ⬜ Create child **Hinge** at pivot (use vertex snapping)
* ⬜ Add **One Grab Rotate Transformer** to the interactable
* ⬜ In **Grabbable → One Grab Transformer = One Grab Rotate Transformer**
* ⬜ Set **Axis = Up** (or desired), **Pivot = Hinge**, **Min/Max Angle** (e.g., −90 → 90)

  * • Locks rotation to the hinge axis for realistic doors/lids.

---

### 8) **Distance grab** (three styles)

* ⬜ **Interaction SDK → Add Distance Grab Interaction** (Wizard)

  * Choose **Hands**, **Controllers**, or **Both**
  * Pick mode:

    * **Hand-relative** (follows with offset)
    * **Pull to hand** (snaps to grip)
    * **Manipulate in place** (stays in world; limited offset)
  * Optional: **Timeout Snap Zone** (auto-return after release)
  * Click **Create**
  * • Wizard adds any missing **Distance Hand/Controller Interactors** and an **ISDK Distance Hand Grab Interaction** component on your object.

---

### 9) **Touch grab** (surface touch → grab)

* ⬜ Find **`OVR Touch Hand Grab Interactor`** prefab (Project search)
* ⬜ In your camera rig’s **Hand Interactions (L/R)**, drop one per hand
* ⬜ On the object:

  * **Rigidbody** (Kinematic/No Gravity recommended)
  * **Grabbable** (link Rigidbody; keep **Transfer On Second Selection = ON** for natural pass)
  * **Touch Hand Grab Interactable**
  * Create child **Bounds** GO → copy your collider here → **Is Trigger = ON**
  * Add same collider shapes to the parent as needed for collisions
  * • Touch-based grabbing is great for UI knobs, sliders, small props.

---

### 10) Hand poses (for realism)

* ⬜ Record **Grab Poses** per object/hand if hands clip or look awkward

  * • Makes contact look natural (no fingers through mesh).


---

# 🔹 Part 3/3 — Manual Meta Interaction SDK Rig & Interactions (Quest 2, Unity 6.2.2f1, Meta SDK v65+)

> **Stack:** Unity 6.2.2f1 • URP • **XR Plug-in Management → Android: Meta XR enabled only** (OpenXR **off**) • Meta XR All-in-One SDK v65+

---

## A) Build the Camera Rig by Hand (no prefab)

### ✅ Steps (and why)

1. **Create root**

   * ⬜ Empty GameObject → **XRRoot**
   * • *Keeps rig & inputs organized.*

2. **OVR Camera Rig**

   * ⬜ Add **OVRCameraRig** component to **XRRoot**
   * ⬜ Under it, ensure the three anchors exist: **LeftEyeAnchor**, **CenterEyeAnchor (Main Camera)**, **RightEyeAnchor**
   * • *OVRCameraRig drives HMD pose & eye anchors.*

3. **OVR Manager**

   * ⬜ Add **OVRManager** (on XRRoot or OVRCameraRig)
   * ⬜ In **OVRManager**: Stereo Rendering **Single-Pass Instanced**, enable **Fixed Foveated Rendering** (Low/Med) if desired
   * • *Runtime/device features (stereo mode, foveation, ASW) are controlled here.*

4. **Tracking Space**

   * ⬜ Add child GO **TrackingSpace** (0,0,0)
   * • *Attach interactors here so they follow HMD origin correctly.*

5. **Hands & Controllers (input sources)**

   * ⬜ Under **TrackingSpace** add:

     * **OVRHands** → children **LeftHand**, **RightHand** (each with **OVRHand** + **OVRSkeleton**)
     * **OVRControllers** → children **LeftController**, **RightController** (each with **OVRControllerHelper/TrackedPose** as needed)
   * • *Hands/Controllers are the sources that Interactors will sit on.*

6. **Interaction Manager & UI**

   * ⬜ Empty GO **XR Interaction Manager** → add **XRInteractionManager** (Meta ISDK)
   * ⬜ **EventSystem** + **XR UI Input Module**
   * • *Manager routes hover/select between Interactors & Interactables; UI module enables world-space UI.*

7. **Locomotion (optional now; add later if needed)**

   * ⬜ Empty GO **LocomotionSystem** → add **TeleportationProvider / SnapTurn / ContinuousTurn / DynamicMove** as desired
   * • *Matches the Starter “Comprehensive” rig but you control which providers exist.*

---

## B) Add Interactors to Hands & Controllers (manual)

> **Concept:** *Interactor = on the hand/controller (source). Interactable = on the object (target). You need both.*

### ✋ Hand Interactors (attach to **LeftHand** and **RightHand**)

* ⬜ **HandGrabInteractor**

  * • *Near-field grabbing with hands; supports grab poses & hand affordances.*
* ⬜ **HandPokeInteractor** *(for buttons/poke UI)*

  * • *Touch/poke interactions with optional poke-limiting.*
* ⬜ **HandRayInteractor** *(for far-field ray)*

  * • *Allows far grabs & ray-driven manipulation when beyond arm’s reach.*
* ⬜ (Optional) **TouchHandGrabInteractor**

  * • *“Grab by touching shape” when you author detailed colliders.*

### 🎮 Controller Interactors (attach to **LeftController** and **RightController**)

* ⬜ **GrabInteractor**

  * • *Grip/trigger-based grabbing with controllers.*
* ⬜ **RayInteractor**

  * • *Controller ray for distance interaction & UI.*
* ⬜ (If using teleport) **TeleportInteractor**

  * • *Projectile ray limited to Teleport layer; pairs with TeleportationProvider.*

> **Tip:** Add **HapticImpulsePlayer** on each interactor if you want haptic feedback.

---

## C) Make an Object Grabbable (from scratch)

### ✅ Minimal “pick up” setup

1. **Create object**

   * ⬜ 3D Cube → scale `0.1` → move slightly above floor
2. **Physics**

   * ⬜ **Rigidbody** → *Use Gravity ON*, **Is Kinematic ON while held** (we’ll let the grab logic drive pose)
3. **Core components**

   * ⬜ **Grabbable** (link the Rigidbody)
   * ⬜ **GrabInteractable** (for controllers)
   * ⬜ **HandGrabInteractable** (for hands)
   * • *This mirrors what the “Quick Actions → Add Grab Interaction” wizard creates.*

### 🧠 Motion behavior (Transformers)

* ⬜ Add **GrabFreeTransformer**

  * **Grabbable → Optionals**:

    * **One Grab Transformer = GrabFreeTransformer**
    * **Two Grab Transformer = GrabFreeTransformer** *(enables 2-hand scaling/rotate)*
* ⬜ In **GrabFreeTransformer**: set **Min/Max Scale** and tick **Constraints Are Relative**

  * • *Lets you one-hand grab, two-hand scale/rotate, with clamped ranges.*

### 🔧 Constraint presets

* **Plane-only slide:** In transformer, lock **Rotation** (all), enable **Position** only on needed axes (e.g., X/Z) → **Constraints Are Relative = ON**
* **Axis-rotation (door):** Replace with **OneGrabRotateTransformer**; set **Axis** + **Pivot (Hinge)**; set **Min/Max Angle** (e.g., −90°..90°)

---

## D) Distance Grab (three styles)

1. On the same object:

   * ⬜ Add **DistanceHandGrabInteractable** (hands)
   * ⬜ Add **DistanceGrabInteractable** (controllers)
2. Pick a mode:

   * **Hand-relative** (*follows with offset*)
   * **Pull to hand** (*snaps to grip point*)
   * **Manipulate in place** (*stays put, limited offset*)
3. (Optional) **TimeoutSnapZone** for auto-return after release.

* • *This replicates what the Distance Grab wizard wires automatically.*

---

## E) Ray Interaction on the same object (manual)

1. Surfaces

   * ⬜ On the **object root**, add **ColliderSurface** (assign the object’s collider to it)
2. Pointable

   * ⬜ Ensure **Grabbable** is present (used as the **Pointable Element**)
3. Ray Interactable

   * ⬜ Add **RayInteractable** (assign **Pointable Element = Grabbable**, **Surface = ColliderSurface**)
4. Movement behavior

   * ⬜ Add **MoveFromTargetProvider** (or another **MovementProvider**) and assign it in **RayInteractable**

* • *Enables far-field manipulation through hand/controller rays.*

---

## F) Touch Grab (shape-aware grabbing)

1. Author bounds

   * ⬜ Create child GO **Bounds** under the object
   * ⬜ Copy/compose colliders to approximate the mesh; set **Is Trigger = ON**
2. Configure

   * ⬜ Add **TouchHandGrabInteractable** (assign **Grabbable** as **Pointable Element**)
   * ⬜ In its **Colliders** list, add your detailed **Bounds** colliders; for **Bounds Collider**, you can use the default sphere or a trigger that envelopes the shape

* • *Hands can “grab the body” of complex geometry rather than a generic sphere.*

---

## G) Poke Interaction (recommended manual path)

* **Simplest**: use the provided prefab

  * ⬜ **Packages/com.meta.xr.sdk.interaction/Runtime/Prefabs/Poke/PokeInteractable.prefab**
  * Place your model under its **Visuals** object; adjust **Model → local Z** to tune press depth
* **Why prefab?** Poke requires a specific child-object structure (limits, volumes, affordances). The prefab is faster and less error-prone.

---

## H) Locomotion (hooking up Teleport/Move/Turn)

1. **Providers**

   * ⬜ On a **LocomotionSystem** GO add needed providers:

     * **TeleportationProvider**, **SnapTurnProvider**, **ContinuousTurnProvider**, **DynamicMoveProvider**
2. **Surfaces**

   * ⬜ On floor: **TeleportationArea** (or **TeleportationAnchor** objects)
3. **Interactors**

   * ⬜ On controllers/hands: **TeleportInteractor** (uses projectile & Teleport mask)

* • *Matches the “Comprehensive Rig” but you choose only what you need.*

---

## I) Validation & Troubleshooting

* **Actions/Warnings**

  * “`XR_ERROR_ACTIONSET_NOT_ATTACHED` (Microgesture…)”

    * ⬜ Not using microgestures? **Disable/Don’t include** that feature in Meta settings or **update to SDK 65+**. Harmless if ignored.
* **Hands don’t grab**

  * ⬜ Verify **Grabbable** links its **Rigidbody**
  * ⬜ At least one of **HandGrabInteractable / GrabInteractable** is present
  * ⬜ Interactors exist on the input source (Hand/Controller)
* **Object flops on release**

  * ⬜ Keep **Is Kinematic = true while held**, switch to non-kinematic on release if you need throws; add your own release-velocity if using pure kinematic hold.
* **Ray won’t move object**

  * ⬜ Ensure **ColliderSurface** present + assigned
  * ⬜ **MovementProvider** is assigned in **RayInteractable**
* **Teleport not working**

  * ⬜ Floor has **TeleportationArea**
  * ⬜ Interactor is **TeleportInteractor** (and layer masks include Teleport)

---

## J) Quick “Manual vs Starter” cheat-sheet

| Capability                    | **Manual Build (this guide)** | **Starter / Building Blocks**   |
| ----------------------------- | ----------------------------- | ------------------------------- |
| Rig (OVRCameraRig + Manager)  | You add & tune each piece     | Drag prefab / one-click block   |
| Interactors (Hand/Controller) | You choose exactly which      | Preconfigured set (can delete)  |
| Grab/Ray/Poke/Distance        | Add components per object     | Quick Actions/Wizards wire them |
| Constraints/Transformers      | You add & assign              | Often pre-wired; tweak values   |
| Learning & control            | Maximum                       | Fastest                         |

---

### Performance reminders (Meta-only stack)

* **OVRManager → Single-Pass Instanced**, a modest **Foveation** level
* URP **HDR Off**, **MSAA 4x**, **Shadows Medium (512), 20–25m, 2 cascades**
* Renderer **Post-processing Off**, **no extra Renderer Features**
* Keep only the Interactors you really use (disable Poke/Distance if not needed)


---
# 🔹 Final Meta SDK raycasting with 3D objects and world-space UI

# 🎯 Goal

A scene where your **controller/hand rays** can:

* Hover/select a **3D object** (move it with the ray)
* Point & click a **world-space UI** (buttons/sliders)

> Stack: **Unity 6.2.2f1 • URP • XR Plug-in Mgmt → Android: Meta XR enabled (OpenXR OFF) • Meta XR AIO SDK v65+**

---

# 🧱 0) One-time prerequisites (meta-only stack)

* ⬜ **XR Plug-in Mgmt → Android → Meta XR = ON**, **OpenXR = OFF**

  * • *Avoids mixed runtimes and OVR warnings*
* ⬜ **OVRManager → Stereo = Single-Pass Instanced**, **Foveation = Low/Med (optional)**

  * • *Mobile VR perf win*
* ⬜ **URP (Mobile_RPAsset)**: **HDR OFF**, **MSAA 4x**, **Shadows 512 / 20–25m / 2 cascades**, **PostFX OFF**

  * • *Stable 72–90 FPS baseline on Quest 2*

---

# 🗺️ 1) New scene scaffold

* ⬜ **Create Scene** → `Raycast_Objects_UI`

  * • *Keeps the demo self-contained*
* ⬜ **Delete Main Camera**

  * • *OVR supplies the tracked camera*

---

# 🎥 2) Camera rig & managers (manual, Meta SDK)

* ⬜ Create **Empty GO** `XRRoot` (0,0,0)

  * • *Keeps rig tidy*
* ⬜ Add **OVRCameraRig** to `XRRoot`

  * • *Tracked HMD + eye anchors*
* ⬜ Add **OVRManager** (on `XRRoot` or `OVRCameraRig`)

  * • *Runtime features: stereo mode, foveation*
* ⬜ Child GO `TrackingSpace` under `XRRoot`

  * • *Attach inputs here*
* ⬜ Under `TrackingSpace`, add **OVRHands** (with **OVRHand + OVRSkeleton** on Left/Right)

  * • *Hand input modality*
* ⬜ Under `TrackingSpace`, add **OVRControllers** (Left/Right)

  * • *Controller input modality*
* ⬜ **XR Interaction Manager** GO → add **XRInteractionManager** (Meta ISDK)

  * • *Routes hover/select between Interactors & Interactables*
* ⬜ **EventSystem** → add **Event System** + **XR UI Input Module**

  * • *Enables XR pointer events for Canvas UI*

> If you prefer a jump-start: dropping **OVRInteractionComprehensive** gives you most of this pre-wired. We’re staying manual so you learn each piece.

---

# 🔦 3) Add **ray Interactors** to inputs

> **Concept**: Interactor = on the **hand/controller** (source). Interactable = on the **target** (object/UI).

### Controllers

* ⬜ On **LeftController**: add **RayInteractor** (+ **XR Interactor Line Visual** if you want a beam)
* ⬜ On **RightController**: add **RayInteractor** (+ Line Visual)

  * • *Far-field pointing, hover, and select*

### Hands (optional if you want hand rays)

* ⬜ On **LeftHand**/**RightHand**: add **HandRayInteractor**

  * • *Ray without controllers*

**Recommended visual polish on each Ray Interactor**

* ⬜ **Max Ray Length** ~ 5–8 m
* ⬜ **Hide When No Interactable = ON**

  * • *Clean UI: ray only when it matters*

---

# 🧊 4) Make a **3D object** react to rays (object raycast)

We’ll create a cube you can **aim at, select, and move** with the ray.

* ⬜ Create **Plane** at (0,0,0) for floor

  * • *Teleport/ground reference*
* ⬜ Create **Cube** → rename `RayObject` → scale `0.2` → lift to y=0.5

  * • *A visible target*
* ⬜ Add **Rigidbody** (Use Gravity ON, Is Kinematic ON while selected is fine)

  * • *Physics when needed*
* ⬜ Add **Grabbable** (drag Rigidbody)

  * • *Core selection state*
* ⬜ Add **ColliderSurface** on the cube → **Collider = (its collider)**

  * • *Surface data for ray interaction*
* ⬜ Add **RayInteractable**

  * • *Makes the object respond to rays*
* ⬜ Add **MoveFromTargetProvider** (or your preferred **MovementProvider**)

  * • *Defines how the object moves once selected by ray*
* ⬜ In **RayInteractable**:

  * **Pointable Element = Grabbable (self)**
  * **Surface = ColliderSurface (self)**
  * **Movement Provider = MoveFromTargetProvider (self)**

**Why this composition?**

* **RayInteractor** (on the controller/hand) fires raycasts → **RayInteractable** (on the object) negotiates selection → **MovementProvider** applies motion → **ColliderSurface** tells the system where/what was hit.

---

# 🖱️ 5) Make a **World-Space UI** work with rays (UI raycast)

We’ll add a floating panel (Canvas + Button) that responds to your controller ray.

* ⬜ **GameObject → UI → Canvas** → rename `UI_Canvas`

  * **Render Mode = World Space**
  * **Scale** to something readable in VR (e.g., `0.001, 0.001, 0.001`) and place at ~1.5m in front of camera
  * • *World-space UI is required for VR*
* ⬜ On `UI_Canvas`, add **TrackedDeviceGraphicRaycaster**

  * • *Lets XR rays drive Unity UI events*
* ⬜ (Optional) Add **Canvas Optimizer** (from Meta samples)

  * • *Cheaper UI updates*
* ⬜ Inside the Canvas, add **Button** and **Text (TMP)**

  * • *A target to click*
* ⬜ Ensure scene has **EventSystem + XR UI Input Module** (we added in §2)

  * • *Bridges RayInteractor → UI system*

**Layering / filtering (recommended):**

* ⬜ Put UI Canvas on **UI layer**; in each **RayInteractor**, include **UI** in its interaction mask.

  * • *Ensures the ray hits UI*
* ⬜ To prevent a single ray from hitting both UI and 3D at once, add **Tag Set Filter** or separate layer masks per ray.

  * • *Left ray for 3D, right ray for UI (optional pattern)*

---

# 🧪 6) Quick playtest

* ⬜ Enter Play with Quest Link/AirLink
* ⬜ Aim ray at **RayObject** → hover highlights (if you added visuals), select → object moves using **MoveFromTargetProvider**
* ⬜ Aim at **Button** on `UI_Canvas` → click (trigger/grip depending on your binding) → see OnClick fire

---

# 🩺 7) Troubleshooting cheatsheet

**No ray visible**

* ⬜ **RayInteractor** present on the controller/hand
* ⬜ Line Visual enabled or gizmo visible
* ⬜ “Hide When No Interactable” might be hiding it—point at a valid target

**Ray hits 3D but not UI**

* ⬜ Canvas **Render Mode = World Space**
* ⬜ **TrackedDeviceGraphicRaycaster** on the Canvas
* ⬜ **EventSystem + XR UI Input Module** exists
* ⬜ RayInteractor’s **interaction mask includes UI layer**
* ⬜ Canvas scale not microscopic (try 0.001) and within ray length

**Ray hits UI but not objects**

* ⬜ Add **ColliderSurface** + **RayInteractable** on the object
* ⬜ Assign **MovementProvider**
* ⬜ Make sure the object’s **collider** isn’t on an ignored layer

**Console spam: Microgesture ActionSet not attached**

* ⬜ You’re not using microgestures → disable that feature (or update Meta SDK 65+)
* ⬜ Harmless; doesn’t affect ray/ UI

---

# 🧰 (Optional) Niceties & polish

* ⬜ **Reticle** at ray hit point (Line Visual end or your own prefab)
* ⬜ **Hide Ray When No Interactable** = ON
* ⬜ **HapticImpulsePlayer** on Interactors for subtle clicks
* ⬜ **Snap/Distance Grab** on RayObject if you want “pull to hand” behavior
* ⬜ **Per-ray layer masks** (Left = 3D only, Right = UI only)

---

# 📋 Copy-paste micro-checklist

**Scene**

* [ ] New scene `Raycast_Objects_UI`
* [ ] Plane ground

**Rig & Managers**

* [ ] OVRCameraRig + OVRManager
* [ ] XRInteractionManager
* [ ] EventSystem + XR UI Input Module

**Rays**

* [ ] RayInteractor on Left/Right Controller (and/or HandRayInteractor)
* [ ] Hide When No Interactable = ON, Max Length ~6m

**3D Object**

* [ ] Rigidbody (Gravity ON, Kinematic when selected OK)
* [ ] Grabbable
* [ ] ColliderSurface (assign collider)
* [ ] RayInteractable
* [ ] MoveFromTargetProvider (assigned in RayInteractable)

**UI**

* [ ] Canvas (World Space), reasonable scale
* [ ] TrackedDeviceGraphicRaycaster on Canvas
* [ ] Button to test
* [ ] Ray mask includes UI layer

**Perf**

* [ ] Single-Pass Instanced, foveation Low/Med
* [ ] URP: HDR OFF, MSAA 4x, Shadows 512/20–25m/2 casc., PostFX OFF

---
