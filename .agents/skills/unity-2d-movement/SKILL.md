---
name: unity-2d-movement-style
description: Applies DawnosaurDev's Unity 2D movement coding style. Use when writing player controllers, enemy movement, or any physics-based mechanic (springs, fans, knockback). Primary focus is player movement; patterns also apply to any feel-first Rigidbody2D system.
---

# Unity 2D Movement Coding Style

This skill captures the architecture, naming conventions, commenting style, and physics patterns
used in DawnosaurDev's PlayerMovement + PlayerData system. Follow these guidelines to produce
code that matches the original style.

## Scope

**Primary use:** player character controllers — running, jumping, wall jump, dash, slide.

**Also applicable** to any Unity mechanic that drives a Rigidbody2D with feel-first physics:
enemy patrol/chase movement, spring pads, wind fans, conveyor belts, knockback systems.
When applying to non-player objects, keep the ScriptableObject data split and timer-based
state patterns, but skip player-specific regions (INPUT CALLBACKS, wall check logic, etc.).

---

## Architecture: Two-Class Split

Always separate **data** from **logic** into two distinct classes.

### Data Class → ScriptableObject

- Inherits from `ScriptableObject`
- Decorated with `[CreateAssetMenu(menuName = "...")]`
- Contains only public fields (serialized in Inspector)
- Uses `[HideInInspector]` for derived/computed values the designer should NOT touch
- Uses `[Header("...")]` and `[Space(n)]` to group fields visually in Inspector
- Uses `[Range(min, max)]` on fields that have a meaningful physical boundary — this renders
  a slider in the Inspector **and** silently enforces limits without needing runtime checks.
  Apply `[Range]` when: the value is a multiplier (0–1 or 0–1.5), a grace-period duration
  (0.01–0.5 s), or any float where values outside the range are physically nonsensical.
  Do NOT use `[Range]` on fields whose valid maximum is dynamic (e.g. speed values that
  depend on other fields — clamp those in `OnValidate()` instead).
- Uses `OnValidate()` to **auto-compute derived values** and **clamp designer inputs**

```csharp
[CreateAssetMenu(menuName = "Player Data")]
public class PlayerData : ScriptableObject
{
    [Header("Jump")]
    public float jumpHeight;
    public float jumpTimeToApex;
    [HideInInspector] public float jumpForce; // computed in OnValidate

    [Header("Assists")]
    [Range(0.01f, 0.5f)] public float coyoteTime;       // slider: grace period has clear bounds
    [Range(0f, 1f)]      public float accelInAir;        // slider: 0 = no air control, 1 = full

    private void OnValidate()
    {
        // Step 1 — compute gravity from designer-friendly jump parameters
        // Formula: gravity = 2 * jumpHeight / timeToJumpApex^2
        gravityStrength = -(2 * jumpHeight) / (jumpTimeToApex * jumpTimeToApex);

        // Step 2 — express gravity as a scale relative to Unity's global Physics2D.gravity.y
        // This keeps the system portable — changing Project Settings gravity won't break values
        gravityScale = gravityStrength / Physics2D.gravity.y;

        // Step 3 — derive jump force from gravity and apex time
        // Formula: initialVelocity = gravity * timeToApex
        jumpForce = Mathf.Abs(gravityStrength) * jumpTimeToApex;

        // Step 4 — clamp inputs that have dynamic valid ranges (can't use [Range] here)
        runAcceleration  = Mathf.Clamp(runAcceleration,  0.01f, runMaxSpeed);
        runDecceleration = Mathf.Clamp(runDecceleration, 0.01f, runMaxSpeed);
    }
}
```

### Logic Class → MonoBehaviour

- Inherits from `MonoBehaviour`
- Holds a `public PlayerData Data;` reference (drag-assigned in Inspector)
- Never stores tunable numbers directly — always reads from `Data`
- Organizes everything into `#region` blocks

---

## Region Structure (always use these, in this order)

```csharp
#region COMPONENTS
#region STATE PARAMETERS
#region INPUT PARAMETERS
#region CHECK PARAMETERS
#region LAYERS & TAGS

// Unity callbacks (Awake, Start, Update, FixedUpdate) — no region wrapper

#region INPUT CALLBACKS
#region GENERAL METHODS
#region RUN METHODS
#region JUMP METHODS
#region DASH METHODS
#region OTHER MOVEMENT METHODS
#region CHECK METHODS
#region EDITOR METHODS
```

Keep regions ALL CAPS. Each region wraps a cohesive group of fields or methods.

---

## Naming Conventions

| Category                              | Convention                           | Example                                                    |
| ------------------------------------- | ------------------------------------ | ---------------------------------------------------------- |
| Public properties (read-only outside) | PascalCase + `{ get; private set; }` | `IsJumping`, `IsFacingRight`, `LastOnGroundTime`           |
| Private fields                        | `_camelCase` with underscore prefix  | `_isJumpCut`, `_dashesLeft`, `_moveInput`                  |
| Public Data fields (ScriptableObject) | `camelCase`                          | `runMaxSpeed`, `jumpForce`, `coyoteTime`                   |
| Methods                               | PascalCase                           | `CanJump()`, `SetGravityScale()`, `CheckDirectionToFace()` |
| Coroutines                            | PascalCase, descriptive verb         | `StartDash()`, `RefillDash()`, `PerformSleep()`            |
| Bool state                            | Prefix `Is` or `Can`                 | `IsJumping`, `IsDashing`, `CanSlide()`                     |

---

## Timer-Based State Pattern

**Never use plain `bool isOnGround`**. Instead, use **countdown timers** that double as
coyote-time buffers. A timer `> 0` means the condition is active.

```csharp
// In Update() — decrement every frame
LastOnGroundTime -= Time.deltaTime;
LastPressedJumpTime -= Time.deltaTime;

// When condition is met — reset to buffer duration
LastOnGroundTime = Data.coyoteTime;
LastPressedJumpTime = Data.jumpInputBufferTime;

// To check — same as checking a bool
if (LastOnGroundTime > 0) { /* player is grounded (or within coyote window) */ }
```

This pattern gives **coyote time** and **input buffering** for free with zero extra code.
Apply it to: ground detection, wall detection, jump input, dash input.

---

## Physics & Force Patterns

### Run (FixedUpdate via AddForce)

Compute a **target speed**, find the **speed difference**, multiply by **accel rate**, apply as force.
Never set `linearVelocity.x` directly for running.

```csharp
float targetSpeed = _moveInput.x * Data.runMaxSpeed;
targetSpeed = Mathf.Lerp(RB.linearVelocity.x, targetSpeed, lerpAmount);
float speedDif = targetSpeed - RB.linearVelocity.x;
float movement = speedDif * accelRate;
RB.AddForce(movement * Vector2.right, ForceMode2D.Force);
```

### Jump (Impulse, compensates for falling velocity)

Always compensate for current downward velocity to guarantee consistent jump height.

```csharp
float force = Data.jumpForce;
if (RB.linearVelocity.y < 0)
    force -= RB.linearVelocity.y; // subtract negative = add magnitude
RB.AddForce(Vector2.up * force, ForceMode2D.Impulse);
```

### Dash (Coroutine, two-phase)

Use a coroutine with two `while` loops: attack phase (fixed velocity) → end phase (reduced velocity).
Set `gravityScale = 0` during dash attack, restore after.

```csharp
// Phase 1 — attack
while (Time.time - startTime <= Data.dashAttackTime)
{
    RB.linearVelocity = dir.normalized * Data.dashSpeed;
    yield return null;
}
// Phase 2 — end
SetGravityScale(Data.gravityScale);
RB.linearVelocity = Data.dashEndSpeed * dir.normalized;
while (Time.time - startTime <= Data.dashEndTime)
    yield return null;
IsDashing = false;
```

### Gravity (dynamic multipliers, Update)

Never use a single gravity value. Switch gravity multiplier based on movement state:

```
IsSliding           → 0 (no gravity)
Dashing             → 0
Holding down + falling → gravityScale × fastFallGravityMult
Jump cut (released) → gravityScale × jumpCutGravityMult
Near apex of jump   → gravityScale × jumpHangGravityMult  (creates "hang time")
Normal falling      → gravityScale × fallGravityMult
Default             → gravityScale
```

Always pair a gravity multiplier increase with a **terminal velocity cap**:

```csharp
RB.linearVelocity = new Vector2(RB.linearVelocity.x,
    Mathf.Max(RB.linearVelocity.y, -Data.maxFallSpeed));
```

### Slide (FixedUpdate via AddForce, clamped)

Wall sliding uses the same speedDiff × accelRate pattern as Run, but on the Y axis.
The key difference: **clamp the resulting force** to prevent over-correction oscillation
(this artifact doesn't appear in Run because horizontal corrections are small; vertical
corrections against gravity can overshoot badly without the clamp).

```csharp
private void Slide()
{
    // Cancel any remaining upward impulse first — prevents "climbing" the wall
    if (RB.linearVelocity.y > 0)
        RB.AddForce(-RB.linearVelocity.y * Vector2.up, ForceMode2D.Impulse);

    // Same speedDiff × accel pattern as Run, but Y-axis
    float speedDif  = Data.slideSpeed - RB.linearVelocity.y;
    float movement  = speedDif * Data.slideAccel;

    // Clamp: force can't exceed what would correct the full speedDif in one FixedUpdate tick
    // Without this clamp, the force overshoots and causes jitter
    movement = Mathf.Clamp(movement,
        -Mathf.Abs(speedDif) * (1 / Time.fixedDeltaTime),
         Mathf.Abs(speedDif) * (1 / Time.fixedDeltaTime));

    RB.AddForce(movement * Vector2.up);
}
```

> Why clamp here but not in Run? Run's `Lerp` on `targetSpeed` already smooths out large
> corrections before the force is calculated. Slide has no equivalent smoothing step.

---

## CanDo() Check Methods Pattern

Every action has a corresponding `private bool CanX()` method that encapsulates
all preconditions. Call only `CanJump()`, `CanDash()` etc. in the decision logic — never
inline the condition.

```csharp
private bool CanJump()
{
    return LastOnGroundTime > 0 && !IsJumping;
}

private bool CanDash()
{
    if (!IsDashing && _dashesLeft < Data.dashAmount && LastOnGroundTime > 0 && !_dashRefilling)
        StartCoroutine(nameof(RefillDash), 1); // side-effect: trigger refill
    return _dashesLeft > 0;
}
```

`CanDash()` is allowed to have side effects (starting the refill coroutine) — this is intentional.

---

## Coroutine Conventions

- Always call coroutines via `StartCoroutine(nameof(MethodName), param)` — use `nameof()`, never a raw string
- Wrap simple coroutine calls in a helper method to avoid scattering `StartCoroutine` calls:

```csharp
private void Sleep(float duration)
{
    StartCoroutine(nameof(PerformSleep), duration);
}
```

- For time-based loops inside coroutines, use `float startTime = Time.time` + `while (Time.time - startTime <= duration)`
- For freeze effects use `Time.timeScale = 0` + `WaitForSecondsRealtime` (not `WaitForSeconds`)

---

## Input Decision Tree (Update)

Action priority in the jump block must follow this exact order (higher priority first):

```
1. CanJump()      (grounded jump)
2. CanWallJump()  (wall jump)
3. bonus jumps left > 0  (double/triple jump)
```

Use `if / else if / else if` — never parallel `if` blocks for mutually exclusive actions.

---

## Collision Detection Style

Use `Physics2D.OverlapBox()` with serialized `Transform` checkpoints and `Vector2` sizes.
Never use raycasts for ground/wall detection in this style.

```csharp
[SerializeField] private Transform _groundCheckPoint;
[SerializeField] private Vector2 _groundCheckSize = new Vector2(0.49f, 0.03f);

Physics2D.OverlapBox(_groundCheckPoint.position, _groundCheckSize, 0, _groundLayer)
```

Use two separate wall check points (`_frontWallCheckPoint`, `_backWallCheckPoint`) that swap
roles when the player turns — do NOT use a single point with signed offset.

---

## Commenting Style

- Every non-obvious line gets an **inline or single-line comment** explaining the _why_, not the _what_
- Use `:D` and casual language freely — the style is friendly and educational
- For formulas, always write the formula in a comment above the code:

```csharp
// gravity = 2 * jumpHeight / timeToJumpApex^2
gravityStrength = -(2 * jumpHeight) / (jumpTimeToApex * jumpTimeToApex);
```

- Mark TODO-style design notes as comments for future readers:

```csharp
//You could experiment with allowing for the player to slightly increase their speed whilst in this "state"
```

---

## Editor Helpers

Always include `OnDrawGizmosSelected()` to visualize check zones:

```csharp
private void OnDrawGizmosSelected()
{
    Gizmos.color = Color.green;
    Gizmos.DrawWireCube(_groundCheckPoint.position, _groundCheckSize);
    Gizmos.color = Color.blue;
    Gizmos.DrawWireCube(_frontWallCheckPoint.position, _wallCheckSize);
}
```

---

## Quick Reference Checklist

When writing a new movement feature in this style, verify:

- [ ] Tunable values live in a `ScriptableObject`, not hardcoded
- [ ] Fixed-range fields use `[Range(min, max)]`; dynamic-range fields are clamped in `OnValidate()`
- [ ] `OnValidate()` derives computed values **and** clamps designer inputs
- [ ] `gravityScale` is derived from `Physics2D.gravity.y`, not hardcoded
- [ ] State is tracked with countdown timers, not plain bools
- [ ] Action preconditions are in a `CanX()` method
- [ ] Forces use `AddForce` + speed-difference pattern (not direct velocity assignment), except dash
- [ ] Y-axis force corrections (slide-like mechanics) are clamped to `speedDif * (1/fixedDeltaTime)`
- [ ] Gravity is a dynamic multiplier with terminal velocity cap
- [ ] Coroutines called via `nameof()`
- [ ] `#region` blocks used for all logical groupings
- [ ] `OnDrawGizmosSelected()` visualizes any new collision check points
- [ ] Comments explain _why_, not _what_, in a friendly tone
