# Verification Plan - Eye Tracking Fix

## Objective
Verify that the enemy eye tracking bug is fixed and that the game remains stable after the SOLID refactoring.

## Steps

1.  **Open the Game**: Open `index.html` in a browser.
2.  **Verify Player Movement**:
    *   Move around with WASD.
    *   Jump (W) and Wall Jump.
    *   Dash (Shift).
    *   Ensure movement feels responsive and identical to the original.
3.  **Verify Shooting**:
    *   Click to shoot.
    *   Ensure bullets travel towards the mouse cursor.
    *   Ensure camera shake and particle effects occur.
4.  **Verify Enemy Eye Tracking (CRITICAL)**:
    *   Stand still and move the mouse around. The **Player's eyes** should follow the mouse cursor.
    *   Move the Player around the Enemy. The **Enemy's eyes** should follow the Player's position.
    *   **Bug Check**: Previously, enemy eyes might have been looking at the wrong spot or not tracking correctly when the camera moved. Ensure they lock onto the player regardless of camera position.
5.  **Verify Combat**:
    *   Shoot the enemy. Ensure it flashes white and takes damage.
    *   Ensure the health bar updates.
    *   Kill the enemy and verify it respawns.
6.  **Verify UI**:
    *   Check HP bar updates.
    *   Check Jump/Dash pip updates.

## Expected Results
*   The game runs without errors in the console.
*   Enemy eyes consistently track the player.
*   Player eyes track the mouse.
*   All gameplay mechanics function as before.
