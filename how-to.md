# Novelle Engine - How to Create Your Game

Novelle is a lightweight visual novel engine that runs games from a simple `data/` folder. This guide will teach you how to create your own visual novel.

---

## Folder Structure

Your game assets must be organized in the `data/` folder with this structure:

```
data/
├── audio/              # Music and sound effects (.ogg)
├── images/
│   ├── backgrounds/    # Background images (.png, .jpg)
│   ├── characters/     # Character sprites (.png)
│   └── overlays/       # Overlay images (.png, .jpg)
└── scripts/            # Your story scripts (.nov)
```

---

## Script Format (.nov Files)

Scripts are plain text files with a `.nov` extension. The engine loads `scripts/main.nov` first, which can include additional scripts.

The engine uses two special labels:
- **`_setup`** - Always runs first. Use for global settings (aliases, typing sound, etc.)
- **`_start`** - The beginning of your story. New games start here.

### Including Additional Scripts

If your game has multiple script files, use the `include` command at the very beginning of `main.nov` (before any labels or commands):

```
# main.nov - Include other script files first
include chapter1 chapter2 endings

label _setup
typing key.ogg
alias MC Player

label _start
text The adventure begins!
```

The `include` command:
- Must be the first command in `main.nov` (comments and empty lines are allowed before it)
- Lists script names without the `.nov` extension
- Loads scripts from the `scripts/` folder

### Basic Syntax Rules

- **Comments** start with `#` and are ignored
- **Empty lines** are ignored (use them freely for readability)
- **Commands** are written one per line: `command arguments`
- **Labels** are defined with `label labelname`

---

## Commands Reference

### Labels and Flow Control

```
# Define a label (jump target)
label intro

# Jump to another label
goto intro

# Wait for user input (click/space/enter) before continuing
wait

# Wait for a specific duration (in seconds) before continuing
wait 2.0

# Save game progress at this point
savegame

# End the game (shows "finish" hint, click to restart)
finish
```

Labels are global across all script files, so you can jump to any label in any loaded script.

---

## The Setup Label (`_setup`)

The `_setup` label is for initializing global settings that apply to the entire game. It runs before anything else, including the save/load menu.

**What to put in `_setup`:**
- Character aliases (`alias`)
- Typing sound (`typing`)
- Any other one-time global configuration

**Special behavior in `_setup`:**
- `goto` commands are **ignored** - you cannot jump away from setup
- When any other `label` is encountered, the engine automatically jumps to `_start`
- This ensures setup always completes and leads to `_start`, regardless of script structure

```
label _setup
typing key.ogg
alias MC Player
alias AI ARIA-7
alias ??? Unknown Voice

label _start
background intro
text Welcome to the game!
```

**Note:** You cannot manually jump to `_setup` or `_start` using `goto` - these jumps are always ignored to prevent breaking the game flow.

---

## Save System and Load Menu

### Saving Progress

Use the `savegame` command to save the player's progress. It stores **only the current label name** to disk.

**Important:** When the player continues, the game jumps directly to the saved label. Any background, music, or character state set in previous labels will **not** be restored automatically. You must set up the scene at the beginning of each label that contains a `savegame`.

**Best practice:** Always set background and music at the start of a label with `savegame`:

```
label chapter2
# Set up the scene for this label
background forest
music adventure.ogg
characterA Alex
mood determined
savegame

text The journey continues...
```

**Don't worry about double-loading:** The engine automatically skips loading if the same background or music is already active. So when playing through normally (not from Continue), the commands won't reload assets that are already displayed/playing.

### The Load Menu

When the game starts and a save file exists, a **Continue / New Game** menu automatically appears after `_setup` completes:

- **Continue** - Jumps to the saved label (skipping `_start`)
- **New Game** - Deletes the save file and starts from `_start`

If no save exists, the game proceeds directly into `_start` without showing the menu.

The save file is stored at `data/save.txt` and contains only the label name.

---

### Dialogue

```
# Display dialogue text
text Hello, this is dialogue text.
text Each text command shows one text box.

# Hide the text box (use text with no argument)
text
```

The text is displayed with a typewriter effect. Click or press Space/Enter to advance.

### Characters

```
# Set who is speaking (appears on left side, slides in from left)
characterA Alex

# Set who is speaking (appears on right side, slides in from right)
characterB Robot

# Set character mood/expression (loads characters/[name]_[mood].png)
mood happy
mood angry
mood default

# Hide the character sprite (use mood with no argument)
mood
```

The engine looks for sprite files named: `images/characters/[charactername]_[mood].png`

Example: If you use `characterA Alex` and `mood happy`, it loads `images/characters/Alex_happy.png`

Using `mood` without any argument will hide the character image while the character's name still appears in the dialog box. This is useful for narration or when a character is speaking off-screen.

### Character Aliases

```
# Create a display name alias for a character
# Format: alias shortname Display Name
alias AI ARIA-7
alias ??? Unknown Voice
alias Protagonist You
```

When `characterA Protagonist` is used, the dialog box will show "You" instead of "Protagonist".

### Backgrounds

```
# Set background (from images/backgrounds/)
background cryo_room

# Clear the background (use background with no argument)
background
```

Background changes use a smooth crossfade transition. The file extension is not needed - it looks for `images/backgrounds/[name].png` or `.jpg`.

### Overlays

Overlays are images displayed on screen, in front of the background and characters but behind the dialog box. They fade in/out smoothly. There are two display modes:

```
# Show overlay with letterboxing (fits entirely on screen)
showoverlay title_image

# Show overlay that covers entire screen (may crop edges)
coveroverlay title_image

# Hide the currently displayed overlay
hideoverlay
```

Use `showoverlay` when you want the entire image visible. Use `coveroverlay` for full-screen images like title screens where you want the image to fill the entire screen.

The script continues after the fade-in/fade-out animation completes. You can have dialogue while an overlay is visible.

### Audio

```
# Play background music (loops automatically)
music ambient_ship.ogg

# Stop music
music stop

# Play a sound effect (plays once)
sound explosion.ogg

# Set the typing sound effect (played during text display)
typing key.ogg
```

Audio files should be in OGG format, placed in `data/audio/`.

### Visual Effects

```
# Fade to black with centered text
# Format: fade [duration] [text]
fade 2.0 Three days later...
fade 4.0 THE END
fade 1.0 ...
```

The fade effect:
1. Fades the scene to black
2. Shows centered text
3. Waits for the specified duration (or until player clicks)
4. Fades back out
5. Continues to next command

Note: The fade command clears the current background, character, and dialogue.

### Player Choices (Branching)

```
branch What do you want to do?
- option1_label First choice text
- option2_label Second choice text
```

- The text after `branch` is the prompt shown to the player
- Each option must start with `-` followed by: `label_to_jump_to` then `Display Text`
- **Maximum 4 options** - any additional options beyond 4 will be silently ignored
- When the player selects an option, the story jumps to that label

### Example Branch Flow

```
label intro
text What should we do?

branch Choose your path:
- go_left Go through the left door
- go_right Go through the right door

label go_left
text You went left and found treasure!
goto ending

label go_right
text You went right and fell into a pit!
goto ending

label ending
text The end.
finish
```

---

## Complete Example: main.nov

```
# My Visual Novel - Main Entry Point

include chapter1 endings

label _setup
# Global settings
typing key.ogg

# Character aliases
alias MC Player
alias Friend Best Friend

label _start
# Start the story
goto intro

label intro
background classroom
music school_ambience.ogg

characterA MC
mood neutral
text Another boring day at school...

characterB Friend
mood excited
text Hey! Did you hear the news?

characterA MC
mood surprised
text What news?

branch How do you respond?
- curious Ask excitedly
- dismissive Shrug it off

label curious
savegame
characterA MC
mood excited
text Tell me everything!
goto story_continues

label dismissive
savegame
characterA MC
mood default
text I'm sure it's nothing important.
goto story_continues

label story_continues
text And so the adventure began...

fade 2.0 To be continued...

finish
```

---

## Tips for Writing

1. **Use multiple script files** for organization:
   - `main.nov` - Setup and entry point
   - `chapter1.nov` - First chapter
   - `chapter2.nov` - Second chapter
   - `endings.nov` - All endings

2. **Character sprites**: Name them consistently:
   - `alex_happy.png`
   - `alex_sad.png`
   - `alex_angry.png`
   - `alex_default.png`

3. **Test frequently**: Run the engine often to catch typos in labels and file references.

4. **Use `mood` with no argument**: To hide the character sprite while keeping their name in the dialog box.

---

## Asset Requirements

### Images
- **Backgrounds**: Any size (will be scaled to cover the screen)
- **Characters**: PNG with transparency, tall images work best
- **Overlays**: PNG or JPG, will be scaled to fit 80% of screen while maintaining aspect ratio

### Audio
- **Format**: OGG (recommended)
- **Music**: Will loop automatically
- **Sound effects**: Play once

---

## Controls

| Input | Action |
|-------|--------|
| Space / Enter / Click / Touch | Advance dialogue / Select option |
| 1 / 2 | Select choice 1 or 2 directly |

---

## Platform Notes

### Desktop
Run the executable with the `data/` folder in the same directory:
```bash
./novelle-linux-amd64
```

The engine always looks for a `data/` folder in the current working directory.

### Web (WASM)
The `dist/web/` folder contains everything needed:
- `novelle.wasm` - The engine (uncompressed)
- `novelle.wasm.gz` - The engine (compressed, ~5MB)
- `wasm_exec.js` - Go WASM runtime
- `index.html` - Loader that auto-decompresses the gzipped WASM
- `data/` - Your game assets

Serve with any HTTP server:
```bash
cd dist/web && python3 -m http.server 8080
```

The compressed version is loaded automatically - browsers that support `DecompressionStream` will use the 5MB gzipped file instead of the 22MB uncompressed one.

---

## Troubleshooting

**Characters not showing**
- Verify sprite files exist: `images/characters/[name]_[mood].png`
- The character name and mood are combined with underscore
- Character names are case-sensitive

**Background not showing**
- Check the file exists in `images/backgrounds/`
- Don't include the file extension in the command

**Music not playing**
- OGG format is required
- Check file exists in `data/audio/`

**Labels not found**
- Label names are case-sensitive
- Make sure the label is defined with `label labelname`

---

## Quick Command Reference

| Command | Description | Example |
|---------|-------------|---------|
| `include` | Load additional scripts | `include chapter1 chapter2` |
| `text` | Display dialogue (empty hides box) | `text Hello!` |
| `characterA` | Set left-side speaker | `characterA Alice` |
| `characterB` | Set right-side speaker | `characterB Bob` |
| `mood` | Set character sprite (empty hides) | `mood happy` |
| `background` | Set background image (empty clears) | `background forest` |
| `music` | Play background music | `music theme.ogg` |
| `sound` | Play sound effect | `sound click.ogg` |
| `typing` | Set typing sound | `typing key.ogg` |
| `label` | Define a jump target | `label chapter2` |
| `goto` | Jump to label | `goto ending` |
| `branch` | Show player choices | `branch What now?` |
| `-` | Branch option | `- labelname Choice text` |
| `fade` | Fade with text | `fade 2.0 Later...` |
| `showoverlay` | Show overlay (letterboxed) | `showoverlay title` |
| `coveroverlay` | Show overlay (full screen) | `coveroverlay title` |
| `hideoverlay` | Hide overlay image | `hideoverlay` |
| `alias` | Create name alias | `alias MC Player` |
| `wait` | Wait for input or duration | `wait` / `wait 2.0` |
| `savegame` | Save progress at current label | `savegame` |
| `finish` | End the game | `finish` |

---
