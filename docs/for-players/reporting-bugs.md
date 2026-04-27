---
title: Reporting Bugs
icon: "🐛"
created: 2026-04-27
updated: 2026-04-27
---

# Reporting Bugs

While playing games on s&box, you may encounter bugs, glitches, or crashes. Because s&box is a platform hosting community-created content, it is very important to report the bug to the right place to get it fixed quickly.

The most common source of confusion is sending a bug report for a specific game (like a broken weapon in a community FPS) to Facepunch (the engine developers). Facepunch cannot fix bugs in community games.

## Is it a Game Bug or an Engine Bug?

Before reporting a bug, try to determine if it is caused by the specific game you are playing (created by the community), or if it is an issue with the s&box platform itself (created by Facepunch).

### It is likely a Game Bug if:
* You only experience the issue in one specific game mode (e.g., your character gets stuck in a wall only when playing "My Cool FPS").
* A specific weapon or item behaves incorrectly.
* The game's user interface is broken or displays error messages related to the game's code.

### It is likely an Engine Bug if:
* The s&box client crashes to the desktop immediately upon launch.
* You experience the exact same graphical glitch (like missing textures or pink/black checkers) across *multiple different* games created by different developers.
* The Main Menu itself is broken or unresponsive.
* You cannot connect to any servers, regardless of the game mode.

## Where to Report Game Bugs

If you believe the issue is isolated to a specific community game:

1. Look for the game developer's contact information on the game's details page in the Main Menu.
2. Most developers link to a Discord server or a GitHub repository where you can report issues directly to them.
3. Provide as much detail as possible: What game were you playing? What were you doing when the bug occurred? Can you reproduce it?

## Where to Report Engine Bugs

If you believe the issue is a core engine problem that affects the entire platform:

1. **Check the Forums:** Search the [s&box forums](https://sbox.game/f/) to see if others have already reported the same issue.
2. **GitHub Issue Tracker:** The official place to report platform and engine bugs is the [sbox-public GitHub repository](https://github.com/Facepunch/sbox-public/issues).
   * **Do not report community game bugs here.** The Facepunch developers cannot fix them.
   * Please search existing issues before creating a new one to avoid duplicates.
   * Provide a clear description, steps to reproduce, and any relevant error logs or screenshots.

## Finding Error Logs

If you are experiencing a crash and want to report an engine bug, including your error logs is critical.

When the engine crashes or encounters a serious error, it generates log files that are invaluable for developers trying to fix the problem.

* You can find your recent crash dumps and logs in your s&box installation directory, typically located at:
  `C:\Program Files (x86)\Steam\steamapps\common\sbox\game\core\logs\` (or similar, depending on where your Steam library is installed).
* If reporting an engine bug on GitHub, please attach these log files to your issue report.
