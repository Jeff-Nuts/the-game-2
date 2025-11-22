# Refactoring Summary

## Overview
The game code has been successfully refactored to adhere to SOLID principles while maintaining all original features. A critical bug involving enemy eye tracking has also been resolved.

## Key Changes

### 1. SOLID Principles Implementation
*   **Single Responsibility Principle (SRP)**: The monolithic `Game` class has been broken down into specialized systems:
    *   `PhysicsSystem`: Handles movement, gravity, and collisions.
    *   `RenderSystem`: Handles all drawing operations.
    *   `InputHandler`: Manages keyboard and mouse input.
    *   `ParticleSystem`: Manages visual effects.
    *   `UIManager`: Updates the DOM-based UI.
    *   `BulletManager`: Manages bullet lifecycle and collisions.
*   **Open/Closed Principle (OCP)**: The `Game` class now operates on generic `GameObject` and `Entity` types, allowing new game objects to be added without modifying the core loop.
*   **Liskov Substitution Principle (LSP)**: A clear inheritance hierarchy (`GameObject` -> `Entity` -> `Actor`) ensures that subclasses can be used interchangeably where their parent classes are expected.
*   **Interface Segregation Principle (ISP)**: Capabilities like "hasEyes" or "showHealthBar" are handled via property checks (duck typing) in the `RenderSystem`, preventing entities from being forced to implement irrelevant methods.
*   **Dependency Inversion Principle (DIP)**: Entities now depend on an abstract `context` object (Service Locator) injected during updates, rather than coupling directly to the `Game` instance.

### 2. Bug Fix: Enemy Eye Tracking
*   **Issue**: Enemy eyes were not tracking the player correctly because the `RenderSystem` was using screen-space coordinates (which change with the camera) instead of world-space coordinates for the target.
*   **Fix**:
    *   Updated `RenderSystem.drawEntity` to accept a `targetPoint` argument.
    *   Updated `Game.draw` to pass the **Player's world position** as the target for Enemies.
    *   Updated `Game.draw` to pass the **Mouse's world position** (Mouse Screen Pos + Camera Pos) as the target for the Player.
*   **Result**: Both Player and Enemy eyes now track their respective targets accurately, regardless of camera movement.

## Verification
*   **Manual Testing**: Verified movement, jumping, dashing, shooting, and UI updates.
*   **Eye Tracking**: Confirmed that enemy eyes lock onto the player and player eyes lock onto the mouse cursor.

## File Structure
All code remains in a single `index.html` file as requested, but is internally structured with clear class definitions and separation of concerns.
