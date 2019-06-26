# Overview

Rendering is a tough topic, not because it's complicated, but because it's **low level**, as such some of the concepts don't go with game-design thinking or object-oriented contexts: the game engine need to organize renderable **graphics primitives** by **type**, while to the developer it's easeir to deal with **objects** as a whole thing.

The intention for rendering effects was clear and "simple": add a **Blinking Material/Shader** for objects that flashes and appear as placeholder for player-placed contents, add **UI Hover text** (Checkout UI sample; Figure out ray tracing and entity name accessing), add Screen based **post-processing** effects.

In this article we will look at rendering from three perpsectives: 1) Trouble shooting; 2) Practical Setup Receipe; 3) Underlying Engine Architecture and Functional Mechnisms.

# Trouble Shooting

1. Problem: Some shader cann't be found: "... (Error Message)"
    * Context: Shader works fine in Game Studio editor, cannot run as standalone debug
    * Cause: 
    * Solution: 

# Practical Setup

# Rendering Architecture

# Gameplay and Engine Classes

Below I briefly mention some of the game play classes as a complement to existing [API manual](https://doc.xenko.com/latest/en/api/index.html).

1. `GraphicsAdapter`: Represents actual physical display, inlcuding graphics interface card names and capabilities
2. `GraphicsDeviceManager`: Manages and allows update backbuffer size and toggle (native, i.e. non-borderless) **fullscreen mode**