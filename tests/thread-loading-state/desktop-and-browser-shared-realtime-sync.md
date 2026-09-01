### Desktop and browser shared realtime sync

#### Feature/Change Name
Synchronize the host Codex desktop with two authenticated browser clients while preserving cached sessions through a managed app-server restart.

#### Prerequisites/Setup
1. A built `codex-ui` host is running with a shared password on a recorded test port.
2. Codex desktop is running as the same operating-system user and uses the same resolved `CODEX_HOME`.
3. Browser A and Browser B are signed in to the same host URL.
4. Browser developer tools or the runtime profiler is available for request-count checks.
5. Record the `codex-ui` parent process and its single managed Codex app-server child before testing restart recovery.
6. Identify one legacy session whose `cwd` matches a Desktop local project's `rootPaths`, but whose ID is absent from `thread-project-assignments`.

#### Steps
1. Open the home route in Browser A and Browser B and confirm both show the existing host project/session list.
2. In Codex desktop, create a session named `shared-sync-<timestamp>` in a known project.
3. Wait at least 250 ms for coalescing and up to 2 seconds for transport/rendering; do not reload either browser.
4. Confirm the new session appears in both browser sidebars.
5. Open the new session in Browser A, then open the same session in Browser B.
6. Confirm both browsers load the complete session with `thread/read` and neither passive open sends `thread/resume`.
7. Start a turn containing the same unique timestamp in Browser A and keep Browser B open on that session.
8. Confirm Browser B receives the turn state and completed content without a manual reload.
9. Verify the coalesced update produces at most one forced first-page `thread/list` and one selected `thread/read` in each browser.
10. Terminate only the recorded managed Codex app-server child, leaving the `codex-ui` parent running.
11. While the child restarts, confirm both browsers keep their cached projects, sessions, and selected messages visible.
12. Confirm the header health state changes to reconnecting or unavailable, then returns to `Codex connected` after automatic recovery.
13. Create or rename one more desktop session and confirm both browsers converge again without reload.
14. Send a completed turn from Browser A, then immediately open and continue the same session in Codex desktop without closing either browser.
15. Confirm the bridge sends `thread/unsubscribe` after completion and Codex desktop can continue without an `open in another app` message.
16. While Codex desktop owns a running turn, try to send from Browser B and confirm the browser shows the active-writer conflict instead of pretending the send succeeded.
17. Create a session from Browser A in a known desktop local project and confirm Codex desktop places it in that same project.
18. Create another browser session with no matching desktop project and confirm Codex desktop places it in Chats.
19. Create the two sessions concurrently from Browser A and Browser B and confirm neither project assignment overwrites the other.
20. Confirm the identified legacy session appears under the matching Desktop local project in both browsers even without an explicit assignment.
21. In Codex desktop, create an empty local project, rename another local project, change project order, and move a session between the projects.
22. Confirm both browsers show the desktop project names and order, keep the empty project visible, and move the session to the same project without reload.
23. Move a session to desktop Chats (projectless) and confirm it leaves the project tree and appears in the browser Chats section.
24. Repeat the visible status and session checks in light and dark themes at `1440x900` and `375x812`.

#### Expected Results
- Desktop filesystem/native app-server signals converge in both authenticated browsers in real time.
- Desktop `local-projects`, `thread-project-assignments`, `project-order`, and `projectless-thread-ids` are reflected in both browser sidebars.
- Passive browser session loading never acquires the active writer.
- The bridge atomically acquires the writer and starts browser turns, then releases it only after the authoritative active-turn and backend-queue state allow handoff.
- A real active-writer conflict remains explicit; read-only loading never converts it into apparent write ownership.
- Browser-created and forked sessions are atomically added to Desktop `thread-project-assignments` or `projectless-thread-ids` without replacing unrelated Desktop state.
- Legacy sessions without explicit assignments remain visible when their normalized `cwd` matches a Desktop local project root.
- Duplicate/lower invalidation revisions do not cause duplicate list or selected-thread requests.
- A reconnect `ready` revision forces recovery when browser state may be stale.
- Only the open affected thread is re-read; unrelated thread histories and repositories are not scanned.
- A failed forced refresh never replaces cached project groups or messages with an empty state.
- The supervised app-server restarts with one active child, and the health badge reports the transition without moving adjacent controls.
- The coordinator uses no more than three Codex state watchers.
- Light/dark desktop and mobile layouts have no clipped status text, overlap, horizontal overflow, or duplicate live overlay.

#### Rollback/Cleanup
- Stop only the test host parent process and confirm its managed child exits.
- Close the two test browser sessions and remove the uniquely named test session only if it is no longer needed.
- Preserve profiler output needed as verification evidence; otherwise remove only the generated test artifacts.

---
