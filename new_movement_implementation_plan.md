# Developer Guide: Implementing "Celeste-Style" Physics
**Target:** Gameplay Programmer / Physics Engineer
**Goal:** Refactor the platformer movement engine from a "Static Velocity" model to a "Momentum & Force" model.

---

## 1. Core Philosophy
The current movement system sets velocity directly (`dx = speed`). This makes momentum tricks impossible. The new system must treat velocity as a persistent value acted upon by forces (Acceleration, Friction, Gravity).

**The Golden Rule:**
> If the player is moving **faster** than the target run speed (due to a dash or external force), **do not clamp their speed**. Allow Friction/Air Resistance to slow them down naturally. This is what allows "Super Dashing."

---

## 2. Architecture & Configuration
Centralize all physics constants. Do not use magic numbers in the update loop.

### `PhysicsConfig` Struct
*   **Run Speed:** `4.0` (Max walking speed)
*   **Dash Speed:** `18.0` (Burst speed)
*   **Acceleration:** `1.5` (How fast we reach Run Speed)
*   **Ground Friction:** `0.8` (Stops player quickly on land)
*   **Air Friction:** `0.98` (Preserves momentum in air—crucial for Supers)
*   **Gravity:** `0.6`
*   **Jump Power:** `-13.0`
*   **Jump Cutoff:** `0.5` (Multiplier when jump button is released early)
*   **Wall Slide Speed:** `2.0`

### Helper Classes
*   **`InputBuffer`**: Tracks `jumpPressedTimer` (for jump buffering) and `groundTimer` (for Coyote time).
*   **`Cooldowns`**: Tracks `dashRefillTimer`, `dashDurationTimer`, `wallJumpLockout`.

---

## 3. Implementation Phases

### Phase 1: The Physics Foundation (Soft Caps)
**Objective:** Implement acceleration and distinct friction without breaking high speeds.

**Logic Flow:**
1.  **Get Input Direction** (-1, 0, 1).
2.  **Apply Acceleration:**
    *   If Input is Right: `dx += Acceleration`.
    *   If Input is Left: `dx -= Acceleration`.
3.  **Apply "Soft Cap" (The Critical Step):**
    *   If `abs(dx) > RunSpeed` **AND** `sign(dx) == sign(input)`:
        *   Do **NOT** add more acceleration.
        *   Apply `AirFriction` (if in air) or `GroundFriction` (if on ground) to slowly reduce `dx` toward `RunSpeed`.
    *   Else (Normal walking):
        *   Clamp `dx` to `RunSpeed` if it exceeds it slightly.
4.  **Apply Friction (No Input):**
    *   If Input is 0, multiply `dx` by `GroundFriction` or `AirFriction`.

---

### Phase 2: Input Buffering (Game Feel)
**Objective:** Make the controls feel responsive and forgiving.

**Logic Flow:**
1.  **Coyote Time:**
    *   If `onGround` is true: `coyoteTimer = 6` (frames).
    *   Else: `coyoteTimer--`.
    *   *Check:* When checking if the player can jump, check if `coyoteTimer > 0` instead of `onGround`.
2.  **Jump Buffering:**
    *   If `Jump` key is pressed: `jumpBufferTimer = 5`.
    *   Else: `jumpBufferTimer--`.
    *   *Check:* If `jumpBufferTimer > 0` AND `coyoteTimer > 0`, execute Jump.

---

### Phase 3: The Dash State Machine
**Objective:** Implement the Dash as a state that overrides standard physics.

**Logic Flow:**
1.  **Initiate Dash:**
    *   If `Shift` pressed & `dashes > 0`: Enter Dash State.
    *   Freeze frame for 3 frames (game loop pause) for impact.
    *   Set `dashVector` (8 directions). Normalize diagonals!
    *   Set `dx` and `dy` to `dashVector * DashSpeed`.
2.  **While Dashing:**
    *   Disable Gravity.
    *   Disable Input control (player cannot steer).
    *   Timer counts down.
3.  **End Dash:**
    *   Re-enable Gravity.
    *   Enter "EndDash" state: Multiply velocity by `0.8` (dampening) so the player doesn't fly off screen instantly, but **do not** reset to 0.

---

### Phase 4: Momentum Conversion (Supers & Hypers)
**Objective:** Allow the Jump button to interrupt the Dash to conserve horizontal speed.

**Logic Flow:**
1.  **The Interrupt:**
    *   Inside the `Jump()` function, check: `if (isDashing)`.
2.  **The Super Dash (Horizontal + Jump):**
    *   If `isDashing` and `onGround`:
        *   Cancel Dash State (Gravity returns).
        *   Apply Jump Force (`dy = JumpPower`).
        *   **CRITICAL:** Keep the current `dx` (which is `DashSpeed`).
        *   Because `AirFriction` is low (0.98), the player will fly forward in a long arc.
3.  **The Hyper Dash (Down-Diagonal + Jump):**
    *   If `dashVector.y > 0` (Down/Down-Right) AND `onGround`:
        *   Set Jump Force to 60% (`dy = JumpPower * 0.6`).
        *   Boost Horizontal Speed (`dx *= 1.2`).
        *   Result: Low, extremely fast long jump.

---

### Phase 5: Wave Dashing
**Objective:** Manage dash resource refilling during a Hyper.

**Logic Flow:**
1.  **Dash Refill Logic:**
    *   Normally, dashes refill when you touch the ground.
    *   **For Wave Dash:** If the player jumps *immediately* after a Hyper (within the buffer window), the game considers them as having "touched the ground" for the refill, but "in the air" for the physics.
2.  **Implementation:**
    *   If `HyperJumping`: Set `dashes = maxDashes`.

---

### Phase 6: Wall Mechanics
**Objective:** Add depth to wall interactions.

**Logic Flow:**
1.  **Neutral Jump:**
    *   If `Input.x == 0` during Wall Jump:
        *   Apply High Vertical Force (`-13`).
        *   Apply Low Horizontal Force (push slightly away from wall).
        *   *Result:* Allows climbing a single wall without stamina.
2.  **Wall Bounce:**
    *   If `isDashing` and `dashVector.y < 0` (Up/Up-Diag):
    *   Check collision 2-3 pixels *above* the player's normal wall check.
    *   If Wall Jump pressed:
        *   **Do NOT** reverse `dx` (keep horizontal momentum).
        *   **ADD** to `dy` (boost upward speed significantly).

---

## 4. Debugging Checklist for Developer
*   [ ] **Friction Test:** Run fast, let go of keys. Do you slide to a stop (good) or stop instantly (bad)?
*   [ ] **Super Test:** Dash on ground, then jump. Do you fly further than a normal jump?
*   [ ] **Corner Test:** Dash diagonally into the floor. Do you slide along the floor (good) or get stuck (bad)?
*   [ ] **Wall Test:** Can you climb a single vertical wall by jumping without holding direction keys?