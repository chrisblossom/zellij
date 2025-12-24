# Mouse-Based Pane Resizing

## Goal

Implement mouse-based pane resizing for tiled panes (GitHub Issue #1262).

Users should be able to click and drag pane borders to resize panes instead of using keyboard shortcuts.

## What We Implemented

Added drag-to-resize functionality for tiled panes:

- Click on any tiled pane border (left, right, top, bottom)
- Drag to resize the pane in that direction
- Release to stop resizing

Floating panes are unchanged - clicking their border still moves the pane.

## Files Modified

`zellij-server/src/tab/mod.rs`:

1. Added `Resize` import
2. Added `pane_being_resized_with_mouse` field to Tab struct to track resize state
3. Added `get_border_at_position()` function to detect which border was clicked
4. Modified `handle_active_pane_left_mouse_press` to start resize on border click
5. Modified `handle_left_mouse_motion` to apply resize during drag
6. Modified `handle_left_mouse_release` to clear resize state

## Bug Fix

Fixed text corruption during resize by deferring PTY resize until mouse release.

The issue was that every mouse motion event sent SIGWINCH to shells, causing dozens of redraws per second. Shells couldn't keep up and produced garbled output.

**Solution:** During drag, only update visual pane geometry. On mouse release, send PTY resize once.

Added to `tiled_panes/mod.rs`:
- `resize_pane_with_id_skip_pty()` - geometry only, no PTY notification
- `resize_pty_all_panes()` - sends PTY resize to all panes

## How to Test

1. Build: `cargo xtask build`
2. Run zellij with multiple tiled panes
3. Click and drag on pane borders to resize
4. Verify floating pane move behavior is unchanged
5. Run tests: `cargo test --package zellij-server`

## Related

- Issue: https://github.com/zellij-org/zellij/issues/1262
- Prerequisite (already merged): PR #3538 - Mouse AnyEvent tracking
