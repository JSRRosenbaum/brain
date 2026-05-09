# Brain OS: Global Memory
*Last updated: 2026-05-08 by Sable (autonomous session 66)*

---

## Identity

<user_persona>
- **Name**: Joseph Rosenbaum
- **Role**: Solo founder, software engineer, homelab operator
- **Company**: Janga LLC — **FILED ✅** (2026-05-08, Texas). EIN in hand. Chase business account not yet opened. Grandpa's F&F investment locked in.
- **Primary domains**: Web apps (Node.js/JS), Python automation, homelab infrastructure (Proxmox), AI/agent systems
- **AI partner**: Sable (this system) — cyber-neko technical partner, workspace at ~/auri/
</user_persona>

---

## Current Projects (Active as of 2026-05-08)

<working_memory>

### 🎵 Songsheet (app.songweb.janga.dev)
Church service planning web app for DFWICC (Dallas-Fort Worth area church).

**Status: Production-ready. Phase 2 + Phase 3 security deployed. Sprints 1–11 shipped. Sprints 1–9 ALL COMPLETE (79 features). Sprint 10 COMPLETE (10/10). Sprint 11 COMPLETE (10/10). 101 total features shipped.**

- **Infrastructure**: VMID 616 (`mission-control`, pve1-janga, 10.25.6.16:3011) for Phase 2; VMID 720 (`songweb`, pve0-janga, 10.25.7.20:3010) for Phase 1 song generation/PDF
- **Repo**: `~/git/songweb/`
- **Live URL**: `https://app.songweb.janga.dev`
- **Auth**: JWT-based, scrypt hashing, login rate limiter (10 attempts/15min/IP), JTI blacklist (Map with expiry) for logout, CORS restricted to prod domain
- **Sprint 1 complete (4/4)**: mobile drag reorder (Pointer Events), auto-resume service (localStorage), song selection as primary UX, text entries in order of service (readings/prayers/announcements)
- **Sprint 2 complete (3/3)**:
  - S2-1 (`21c7baa`): generation performance metrics — `generation_metrics` table, fire-and-forget `recordMetric()`, `GET /api/documents/metrics`, `X-Generation-Ms` header
  - S2-2 (`d372de6`): Sunday 19:00 CDT cleanup scheduler — purges expired JTI blacklist entries + `generation_metrics` rows older than 90 days; JTI blacklist upgraded from Set→Map with expiry
  - S2-3 (`ccd50db`): service cloning — `POST /api/services/:id/clone` copies orderOfService/theme/sermonTiming; ⧉ clone button hover-reveals in service list; clone modal with pre-filled title
- **Sprint 3 COMPLETE (7/7 done)**:
  - S3-1 (`86e88e8`): song recency badges in picker — amber/gray badges, "this wk"/"N wks ago", non-blocking fetch; new "Recent" tab sorted by last use with service name context
  - S3-2 (`c19bf09`): song notes per slot — inline note input on each song in order of service; save-on-blur; notes survive drag-reorder; stored as `{type:'song', songId, note}` (backward-compat bare string); renders in worklist/cuesheet/practice-guide as "Director's note". Conflict warning: adds a yellow banner if adding a song used in the last 14 days.
  - S3-2 bugfix (`57bb999`): `generateServiceListing` (Print Service Listing) was silently dropping songs with notes. Fixed + notes render in print view.
  - S3-3 (`b9539f3`): Suggest Set fixed — `buildSongStatsFromServices()` reads live `Service.orderOfService` data instead of hollow `ArchivedSongUse` table (0 records). `generateRecommendationOptions()` runs both stat sources in parallel, uses whichever is richer. Suggest Set now works with live service history.
  - S3-4 (`84a8223`): Quick-add by song number/title — inline input below order of service, dropdown of up to 8 matches, exact number + Enter adds instantly, arrow-key nav, Enter fallback pre-fills picker search. Also fixes `GET /api/songs/recency` to count `{type:'song'}` objects (S3-2 compat).
  - S3-5 (`92a2a1a`): 📋 Copy button — exports service order as plain text to clipboard (slot labels + song numbers + notes + text entries). "✅ Copied!" 1.8s feedback. Fallback textarea if clipboard API blocked.
  - S3-6 (`fadd32a`): Auto-date Sunday — new service modal pre-fills date to next upcoming Sunday; clone modal pre-fills to first Sunday after source date + focuses title field. Song count badge on each service list card. Blue dot for upcoming services.
  - S3-7 (`5b5835c`): Song Insights fixed — `getSongHistory()` and `getSongHeatmap()` now run archive+live queries in parallel, use whichever has more records. Same hollow-archive bypass pattern as S3-3. Song Insights modal now shows real data from live Service records instead of "No archived history yet."
- **Known data issue**: 30 `archive_weeks` in DB have 0 `archived_song_uses` (hollow archives — original import bug). S3-3 and S3-7 both work around this using live Service data. Full fix blocked on source drive at `/run/media/joseph/AUX/OneDrive`.
- **Sprint 4 COMPLETE (5/5)**:
  - S4-1 (`6393928`): Team member management complete — replaces `alert()` stub in Team Manager modal. Inline add/edit form (name, email, instruments, voice part). `saveMember()` handles POST+PUT. Edit button wired. Unlocks full role assignment workflow.
  - S4-2 (`5d8f3cf`): Role badges inline in service editor — each song row shows compact `FirstName · RoleName` pills below the title when roles are assigned. CSS: `.song-item` → column flex, `.song-item-main` for horizontal row, `.role-badge` blue-tinted pill. Zero extra API calls (reads from `appState.roles` cache). Drag system unaffected.
  - S4-3 (`0dfef7b`): Team Coverage Summary modal — "👥 Coverage" button in role panel header. Shows per-member breakdown (all assignments with slot label + song + role), 📋 copy button per member (clipboard text ready for forwarding), orange gap section (songs with no roles), green "all covered" banner. Zero API calls.
  - S4-4 (`3b50d91`): Service templates — `isTemplate` DB field was already in the model, now wired. `GET /api/services` filters out templates; `GET /api/services/templates` returns them; `POST /api/services/:id/save-as-template` creates one. Frontend: templates section below service list; "⭐ Save as Template" in editor; "Use" → date picker → new service. Role assignments not copied (templates define structure, not people). Server restart required.
  - S4-5 (`a7cf4d3`): Inline role assignment from Coverage gap section — unassigned songs in Coverage modal now show role+member dropdowns and `+ Assign` button. `quickAssignFromCoverage()` POSTs to `POST /api/services/:id/roles`, updates `appState.roles`, re-renders modal in place, refreshes inline badges. 3 clicks vs 6+ before. Static files only.
- **Sprint 5 COMPLETE (13/13)**:
  - S5-1 (`9d8c6a6`): Toast notification system — replaced `alert()` (blocking) and silent `showSuccess()` no-op. `_showToast()` core: slide-in from bottom-right, auto-dismiss (success 3s, error 6s, warn 4.5s), ✕ button, stacking, `aria-live`. 3 variants. 18 success call sites now visible. `showResumeToast()` ported. Mobile full-width. 3 static files.
  - S5-2 (`47fbc7e`): Print Service Listing fix — `generateServiceListing()` was a hardcoded 9-slot church layout (broken since S1-4 text entries). Rewrote to iterate actual `orderOfService`: song entries get `DEFAULT_SLOT_SEQUENCE` slot labels; text entries appear as ✦ italic rows. 109-line hardcoded HTML replaced with 84 lines of dynamic rendering. 1 static file.
  - S5-3 (`861b0b1`): Edit text entries in-place — ✎ button on text entry rows opens modal pre-filled with existing label+description. `showTextEntryModal(editIndex)` sets `_editingTextEntryIndex`; `confirmAddTextEntry()` updates in-place vs appending. Modal title + confirm button text toggle dynamically. 1 static file.
  - S5-4 (`eb867dd`): Role assignments in Copy-as-Text — `copyServiceText()` now appends `↳ FirstName (Role), ...` per song after notes. Zero UI change. Reads `appState.roles[songId]` + `appState.teamMembers` inline. 1 static file.
  - S5-5 (`0025809`): Auto-save metadata on blur — blur listeners on `serviceDate`/`serviceTitle`/`serviceTheme`/`expectedAttendance` call `saveCurrentService({ silent: true })`. `sermonTiming` switch also saves. Previously these 5 fields were only persisted on explicit "💾 Save" click. 1 static file.
  - S5-6 (`cd9ce85`): Song picker duplicate-add bug fix — after S3-2, songs with notes are `{type:'song',songId,note}` objects. `renderSongPicker`/`filterSongPicker`/`renderRecentSongs` only excluded bare string IDs, allowing songs with notes to appear as addable again (duplicate add possible). Fixed all 3 Sets to map both types to bare IDs. 1 static file.
  - Fix (`8cce899`): Bulk-paste same bug fixed — paste tab's `_pasteActiveSongIds` had same bare-string-only filter. 1 static file.
  - Fix (`19c5515`): Audit trail fields populated — `createdBy`/`updatedBy`/`archivedBy` now read `getUser()?.email` instead of hardcoded `'current-user@church.local'`. 1 static file.
  - S5-7 (`cebad6e`): Ctrl+S / Cmd+S keyboard shortcut — global `keydown` listener; calls `saveCurrentService()` (with toast) when a service is open. 1 static file.
  - Fix (`af782f2`): SQLite foreign key enforcement — `PRAGMA foreign_keys = ON` added to `db/init.js`. Service deletion now cascades to `role_assignments`. Team member deletion now NULLs `teamMemberId` in role_assignments. Previously silent data orphaning. Requires restart.
  - S5-8 (`e0ec1d6`): Escape key closes open modals — global keydown handler; checks 11 modal IDs in priority order, calls their respective close functions. Previously no keyboard modal dismissal. 1 static file.
- **Sprint 6 COMPLETE (8/8)**:
  - S6-1 (`eaf558c`): Language filter in song picker — All / 🇺🇸 English / 🇲🇽 Español pill buttons. Uses `metadata.tags` (`spanish`, `cancionero`, `bilingual`). Bilingual songs appear under both filters. Filter resets on modal close. Song library: 222 songs (80 English + 140 Spanish/Cancionero + 2 bilingual). 3 static files.
  - S6-2 (`2268444`): Document generation always visible — moved out of role assignment panel into `#serviceDocsSection` (always shown when service is loaded). Was only accessible after clicking a song, blocking the primary workflow. 3 static files.
  - S6-3 (`983f48c`): `_requireConfirm()` replaces all 3 `confirm()` dialogs — double-click pattern (2.5s window), red pulse animation, auto-reset. No more blocking browser confirm dialogs for delete service/template/team member. 2 static files.
  - S6-4 (`9cac4ad`): Recency badges in order of service — `getRecencyBadgeHtml()` added to each song title row in OoS. Pre-loads `songRecency` non-blocking in `loadInitialData()` (was only fetched on picker open). Badges now visible on service load, not just in picker. 1 static file.
  - S6-5 (`15f50ee`): Coverage gap badge on 👥 Coverage button — `updateCoverageBadge()` shows "(N gaps)" amber / "✓" green without opening modal. Zero API calls. Called from renderOrderOfService() + renderRoleAssignments(). 2 static files.
  - S6-6 (`98a1868`): "With Lyrics" browse tab fix — `renderSongPicker()` now checks active mode-btn; detailed mode shows `song.firstLine` preview (was always compact). 1 static file.
  - S6-7 (`7080bec`): Click song title in OoS to open Song Insights — `.song-title-link` + `data-action="insights"` on song title span; click calls showSongInsightsModal() + loadSongInsights(songId) directly. Dotted underline on hover. 2 static files.
  - S6-8 (`3622eaf`): Service notes textarea — `notes TEXT DEFAULT NULL` column in Service model (auto-migrated via `sequelize.sync({ alter: true })` on restart). Textarea in service editor; blur auto-save alongside other metadata fields; included in Copy-as-Text output. 4 files + restart.
- **Sprint 7 in progress**:
  - S7-1 (`a9e1528`+`f279ad1`): Song Library Manager — 📚 Songs button in header; browse/search 222 songs; add/edit/delete from web UI; POST/PUT/DELETE /api/songs; SongLibrary write methods (addSong/updateSong/deleteSong/isManualSong); new custom-songs.json (29xxx English range); Spanish additions → spanish-supplemental-songs.json (19xxx range); delete blocked if song in active service. 6 files + restart.
  - S7-2 (`e1218d5`): Song Lyrics Viewer — 🎵 button on every OoS song row and all 4 song picker render paths (search/browse/recent, compact/detailed modes); full song.sections[] rendered with typed labels (Verse 1, Chorus, Bridge...); zero extra API calls (cachedSongs already has sections); Escape key closes. 3 static files.
  - S7-3 (`d3d6cb4`): Service list filter tabs + month grouping — Upcoming/Past/All tab buttons (default: Upcoming, ascending/nearest-first); services grouped by month with labeled headers (current month blue, future green, count badge); search cross-tab (ungrouped flat list); new service auto-switches tab to match date. 3 static files.
  - S7-4 (`e90445c`): PDF export fixed — was broken since S3-2 ({type:'song'} objects); per-entry type dispatch; {type:'text'} as gray divider blocks; slot labels, per-song notes, service notes in header; button → "📄 PDF with Lyrics". 2 static files.
  - S7-5 (`013eb0c`): Song key per slot — compact key input (Bb, G, Am...) on each OoS song row; preserves alongside note field; shows as [Bb] in Copy-as-Text; blue key badge in PDF. 2 static files.
  - S7-6 (`7d8c9bf`+`7c62f94`): Stats Dashboard — 📊 Stats header button; top-15 songs w/ bar chart; next 4 upcoming services + real coverage gap badges (from svc.roles, all services); unused songs tag cloud (capped 40); team size overview card. 3 static + 1 fix.
  - S7-7 (`0f624f8`): Song key in server-side generators — `getKeyFromEntry()` added to base.js; key badge threaded into worklist, cuesheet, practice-guide, service-summary. 5 backend files + restart.
  - S7-8 (`9c0f490`): Service notes in all generators — `getDocumentHeader()` updated to show service.notes (added S6-8) below theme in all 6 generated PDFs. 1 backend file + restart.
- **Sprint 8 COMPLETE (9/9)**:
  - S8-1 (`6774ad0`): Forward schedule conflict detection — `_buildForwardConflictMap()` scans appState.services for other upcoming services; 📅 badge on OoS song rows when song appears in another upcoming service (tooltip shows service names); `addSongToService()` checks both backward recency AND forward schedule; `showConflictBanner()` shows named services in conflict message. 2 static files.
  - S8-2 (`46c19a6`): Chord chart URL per song — `song.metadata.chordChartUrl` field added to song-library.js (addSong + updateSong). Song Library Manager form now has Chord Chart URL input. OoS rows show 🎸 link button when URL set. Song Library list shows 🎸 per song. PUT+POST /api/songs thread the field. 4 files + restart.
  - S8-3 (`f7109fe`): Copy All Assignments — `_buildAllAssignmentsText()` produces plain-text roster (member name/instruments → per-slot assignments → unassigned songs). "📋 Copy All Assignments" primary button in Coverage modal footer. Clipboard with 2s "✅ Copied!" feedback. Email-ready format. 2 static files.
  - S8-4 (`b2e1272`): Planning Digest — "📅 Digest" button in header. Shows next 4 upcoming services in responsive 2-col grid. Each card: date, title, full OoS with slot labels + key badges + recency badges + forward conflict badges + inline amber gap dot per unassigned song. Service notes + theme in footer. Per-card 📋 copy + global "📋 Copy Digest" (all 4 services as plain text separated by dividers — paste into email/Slack). Zero new API calls. 3 static files.
  - S8-5 (`a28f5e2`): "✅ Available only" filter toggle in song picker. `songIsAvailable(songId)` checks forward conflict map (cached) + recency < 14 days. Applied to all 3 picker tabs (Search, Browse, Recent). Green active state distinct from language pills. Reset on picker close. 3 static files.
  - S8-6 (`b34ab5a`): Director notes per song. `song.metadata.notes` was stored but invisible. Added "Director Notes" textarea to Song Library Manager form (skips auto-generated placeholder). `_showSongEditForm` pre-fills, `_saveSongEdit` sends. Amber 📝 display in picker With Lyrics mode + song library list. `addSong()` in song-library.js now accepts `notes`. 5 files + restart.
  - S8-7 (`01166ee`): Service coverage health chips in service list. Upcoming service cards show `⚠ N gaps` (amber) or `✓ covered` (green) based on `service.roles`. Only upcoming services with songs. `_serviceItemHtml()` computes gaps from flat `service.roles` array. Active (blue) card: chips inherit white-tinted style. Zero API calls. 2 static files.
  - S8-8 (`9226df9`): Per-member upcoming schedule export. Coverage modal member cards now have 📅 button alongside 📋. `_buildMemberUpcomingSchedule(memberId, memberName)` scans all `appState.services` for that member's roles across upcoming services, formats with slot labels + keys. Output: `Mary — Upcoming Schedule\n\n2026-05-11 — Title\n  Open 1: #45 Song [G] — Vocals\n...`. 1 static file.
  - S8-9 (`d9d5869`): Key history in Song Insights. `_getSongKeyHistory(songId)` scans appState.services OoS entries. Frequency chips appear above slot heatmap in modal (blue monospace font + count badge). "No keys recorded yet" fallback. Zero API calls. 2 static files.
- **Sprint 9 COMPLETE (10/10)**:
  - S9-1 (`b55mc2d`): Per-member practice guide PDF. New `MemberGuideGenerator(service, members, roles, {memberId})`. 2-column lyrics layout. `POST /api/documents/:id/member-guide`. 📄 button in Coverage modal per member. 3 files + restart.
  - S9-2 (`35de39a`): All Practice Guides ZIP. `POST /api/documents/:id/member-guide-bundle` packs all member PDFs. "📦 All Practice Guides" in Coverage footer. 3 files + restart.
  - S9-3 (`d18a5cc`): Team Announcement. "📣 Announce" button. WhatsApp/Signal-ready text with OoS + team groupings. 2 static files.
  - S9-4 (`41c2610`): Key auto-fill on song add. `_getSuggestedKey(songId)` returns most-common historical key. Added as blue `.picker-key-hint` badge in picker. Pre-fills entry.key on add. 2 static files.
  - S9-5 (`491fdcc`): Service search by song content. `filterServiceList()` extended to match song titles/numbers within each service's OoS. Uses already-loaded `cachedSongs`. 1 static file.
  - S9-6: Key hints in Recent picker tab (was browse/search only). Completes key hints across all 3 tabs. 1 static file.
  - S9-7 (`99be466`): Planning Digest overlap badges. Songs planned in multiple upcoming services show purple `×N` badge. `songOverlapMap` built from all upcoming. 2 static files.
  - S9-8 (`6fff047`): Director notes inline in OoS rows. `song.metadata?.notes` shown as amber strip below each song row (was picker-only). 1 static file.
  - S9-9 (`5d27520`): Member availability per service. `membersUnavailable` JSON field on Service (auto-migrated). ✓ In / ❌ Away toggle per member in Coverage modal. Persists via saveCurrentService. 3 files + restart.
  - S9-10 (`dddd5d7`): Unavailability-aware gap detection. Songs where all assigned members are Away now show as coverage gaps in badge + modal. 1 static file.
- **Sprint 10 at 7/? done**:
  - S10-1 (`ecd9493`): Role template manager in Team Manager modal. Green chip list for built-in roles, blue+deletable for custom. "+ Add Role" inline form. DELETE /api/roles/:id (custom-only). 4 files + restart.
  - S10-2 (`63a115a`): Service list search includes `service.notes` text. 1-line fix. 1 static file.
  - S10-3 (`c4af5fd`): Song picker search includes `metadata.tags`. Type "communion" to find tagged songs. 1-line fix. 1 static file.
  - S10-4 (`c9fa4b2`): Bulk schedule generator. 📅×N button per template. Modal: start date + weeks (4/6/8/12/16) + title base. Loops POST /services/:id/clone sequentially. 2 static files.
  - S10-5 (`3abb0ae`): Per-slot notes in service-summary PDF. `getNoteFromEntry()` was available but not called in this generator. Amber italic under song title. 1 generator file + restart.
  - S10-6 (`7783983`): Team Activity section in Stats Dashboard. Per-member service count + last-used date, color-coded (green≤30d, amber 31–60d, gray 61+d, faded=never). 2 static files.
  - S10-7: `📦 Full Package (ZIP)` button in document generation section — already wired to existing `POST /api/documents/:id/bundle` endpoint. Generates worklist+cuesheet+practice-guide+service-summary as ZIP. Static-only, no new code needed.
  - S10-8 (`826f60b`): Song favorites. ⭐ toggle on all picker rows + song library; `metadata.favorite` persisted to JSON; Favorites tab in picker; `toggleSongFavorite()` + `renderFavoriteSongs()`. 5 files + restart.
  - S10-9 (`251a50c`): 🔄 Due for Rotation section in Stats Dashboard. Songs used ≥2× but absent 8+ weeks; amber bordered rows; sorted by usage count; click → Song Insights. Pure frontend. 2 static files.
  - S10-10 (`47f4ce2`): Song retirement. ⊘ Retire / ↩ Restore toggle in Library Manager. `metadata.inactive` persisted. Inactive songs hidden from all 4 picker tabs + quick-add. Red "retired" badge + muted list row. "Show retired (N)" toggle in Library Manager header. Stats "Unused Songs" excludes retired (shows "+ N retired"). 5 files + restart.
- **Sprint 10 COMPLETE (10/10). 81 total features shipped across Sprints 1–10.**
- **Sprint 11 COMPLETE (10/10)**:
  - S11-1 (`d71187c`): Service history CSV export. 📥 Export History CSV in Stats footer. One row per OoS slot: date/title/theme/slot/song#/title/key/notes/attendance. BOM prefix for Excel.
  - S11-2 (`5313898`): Monthly activity chart in Stats Dashboard. 12-month bar chart, current month green, avg attendance below bars.
  - S11-3 (`3346e59`): Keyboard navigation in song picker. ArrowDown/Up moves focus; Enter adds song. Blue focus outline.
  - S11-4 (`663f7bb`): Sort controls in song picker browse tab. #, A–Z, Most Used pill buttons. `_sortSongsForPicker()`.
  - S11-5 (`61c0e7b`): No-lyrics warning badge in OoS. Amber ⚠ "No lyrics" inline when `song.sections` is empty.
  - S11-6 (`1d29b9e`): Search result highlighting. `highlightMatch()` wraps match in `<mark>`. Yellow #fef08a.
  - S11-7 (`b1fd45a`): Speaker/Pastor field per service. DB column + input + in all PDFs + Copy-as-Text. Requires restart.
  - S11-8 (`95c1cd1`): Service quick tags. 6 preset chips (Communion/Youth/Guest Speaker/Holiday/Baptism/Special). Purple active state. Tag pills on service list cards.
  - S11-9 (`0aca1a5`): Tag filter in service search. Search by tag name; click tag chip on service card to filter.
  - S11-10 (`08678c5`): Last-used date in Song Library Manager. "Used Nw ago / Used Nmo ago" next to each song.
- **Sprint 11 COMPLETE (10/10). 101 total features shipped across Sprints 1–11.**
- **⚠️ URGENT**: Bootstrap admin `joseph@janga.dev` still has default password `welcome123`. Run `node scripts/set-password.js joseph@janga.dev` on VMID 616 to fix.
- **Deploy path**: local → `scp pve` → `scp pve1` → `pct push 616` → `systemctl restart songweb-phase2`

### 📈 Polymarket Scanner (AI Trading)
Automated crypto prediction market signal system. Scans 46K+ Polymarket markets.

**Status: Running in production (paper trade mode). 40 open signals. 1 WIN settled. Resolver runs every 2 hours.**

- **Location**: `~/polymarket-scanner/`
- **Mechanism**: `barrier_v1` GBM first-passage time model (`P(min S_t ≤ K)` using reflection principle). Finds markets where Polymarket price differs significantly from fair value.
- **Assets covered**: BTC, ETH, SOL, XRP, DOGE
- **Current signals (2026-05-08)**: 40 open signals. 1 WIN confirmed: Sig 329 (BUY_YES "BTC above $80k May 8", +0.715 P&L). Model: `barrier_v1` GBM. evaluate_performance.py: expired splits into abandoned(10) vs superseded(34); BUY_NO shows NO-side entry/FV. Assets: BTC, ETH, SOL, XRP, DOGE.
- **Checkpoint**: Consider extending paper-trade review from May 22 → Jun 1 (only 2 signals resolve before May 22)
- **Schedule**: 6 systemd user timers (`systemctl --user list-timers "polymarket-*"`)
- **Paper trade checkpoint**: 2026-05-22 — do not execute real trades before then
- **Monitor**: `cd ~/polymarket-scanner && python3 -m src.jobs.monitor_signals`
- **Performance**: `cd ~/polymarket-scanner && python3 -m src.jobs.evaluate_performance`
- **DB**: `~/polymarket-scanner/data/scanner.db`

### 🤖 Hermes (AI Agent)
Personal AI agent with tool access (terminal, file system).

**Status: Fully deployed and running. Sudo fix live on all instances.**

- **Local**: `~/.hermes/hermes-agent/` — running at `:8001` (tool server)
- **VMID 613** (`hermes-joseph`, pve1-janga, 10.25.0.5): fully patched — both `/opt` installed pkg and `/root` source tree. 6/6 unit tests pass. (session 36, commit `9541d9475`)
- **⚠️ URGENT**: Secret rotation overdue — Omniroute key and Honcho key exposed in session logs. Rotate at `http://10.25.6.17:20128/dashboard` and `https://app.honcho.dev`

### 🧠 Brain OS (this system)
Personal AI assistant with note management, code generation, and task routing.

**Status: Running at :8002. Memory files populated (2026-05-08).**

- **Repo**: `~/git/brain/` — webhook branch merged to local main (not yet pushed to fork)
- **Memory injection**: This file (Meta) + domain files (Personal/Professional/Studio) injected per query
- **Webhook**: `POST http://localhost:8002/task` with `{"task": "..."}` or `{"task": "...", "route": "FAST"}`
- **Pending**: Push `feat/webhook-http-api` branch to JSRRosenbaum fork (`git push fork main` from ~/git/brain)

### 🔬 Gauntlet (Wearable HID)
EMG/nerve wrist device for HID input (mouse gestures via nerve signal detection).

**Status: Design phase. CRITICAL hardware error identified.**

- **⚠️ CRITICAL**: Wrong chip selected. ADS1292 noise floor = 8μVrms, above target signal range (10–100μV). **ADS1299 required** (0.18μVrms, 44× better).
- **Correct terminology**: ENG/SNC (electro-nerve/sensory nerve conduction), NOT EMG (electromyography).
- **Action before any PCB order**: Order OpenBCI Cyton (~$500) + Ag/AgCl electrodes. Run bench experiment first.
- **Brief**: `~/auri/workspace/gauntlet_corrective_brief.md`

### 📺 LG TV Security Research
Reverse engineering LG Smart TV (webOS) for security research.

**Status: Track B RE complete. Next steps require TV powered on.**

- **Current finding**: `slimFileReadPeer` is a platform I/O callback, not a JS-callable API. Dispatch table mapped. 4 attack vectors identified.
- **TV access**: `ssh prisoner@10.0.0.71 -p 9922`
- **Next probes** (requires TV on):
  - `luna-send-pub -n 1 -f luna://com.webos.service.hybridtvapiserver/getServerUrl '{}'`
  - `curl http://127.0.0.1:9998/json/list`
- **Scripts**: `~/tv-research/`

### 🌈 OpenRGB Spatial Mapping (Nollie Editor)
LED spatial layout mapping for OpenRGB custom lighting profiles.

**Status: All 17 anchors derived. Layout rules complete.**

- **Output**: `~/projects/nollie_editor_capture_front-a-anchors.json`
- **Remaining**: Manually drag Ch13 badge LEDs; retake Ch9 capture

### 📚 Field Theory (fieldtheory-cli)
Personal knowledge indexing suite: ft (full-text), fb (bookmarks), fstar (GitHub stars), fhn (Hacker News).

**Status: v3 feature-complete.**

- **Unified CLI**: `~/.local/bin/field`
- **Commands**: `field search "X"`, `field cross "X"`, `field related <url>`, `field timeline`
- **Items indexed**: ~3,886 across all 4 tools
- **Main ft repo**: actively maintained at `~/fieldtheory-cli/`

</working_memory>

---

## Infrastructure Map

| VMID | Node | Name | IP | Role |
|------|------|------|----|------|
| 613 | pve1-janga | hermes-joseph | 10.25.0.5 | Hermes agent container |
| 616 | pve1-janga | mission-control | 10.25.6.16 | Songsheet Phase 2 (:3011) |
| 617 | pve1-janga | omniroute | 10.25.6.17 | OmniRoute LLM gateway (:20128) |
| 601 | pve1-janga | bifrost | 10.25.6.1 | BiFrost unified LLM gateway |
| 614 | pve1-janga | gitea | — | Gitea git server |
| 615 | pve1-janga | forgejo | — | Forgejo git server |
| 611 | pve1-janga | claw-sarah | — | Claw instance (Sarah) |
| 612 | pve1-janga | claw-joseph | — | Claw instance (Joseph) |
| 801 | pve1-janga | dev-trader-staging-01 | — | Trading dev VM |
| 720 | pve0-janga | songweb | 10.25.7.20 | Songsheet Phase 1 (:3010) |

**PVE access**: `ssh pve` (pve0, 10.0.0.254) → `ssh pve1` (pve1, 10.0.0.253) → `pct exec <VMID> -- bash`

**Local services** (running on Joseph's desktop):
- Tool Server: `:8001` (Hermes tools)
- Brain Webhook: `:8002` (this system)
- Meridian: `:3456` (LLM proxy)
- Reflecting Pool: daemon (auri autonomous loop, PID varies)

**Cloudzy** (VPS, `ssh cloudzy`): Caddy reverse proxy, routes `app.songweb.janga.dev` traffic to homelab

---

## Key Decisions & Context

<decisions>

### Company / Legal
- **Janga LLC**: Filed ✅ (2026-05-08, Texas). EIN obtained. Solo-founder, no co-founders (solo decision 2026-03-11).
- **F&F round**: Grandpa locked in for $20k. $3,500 already advanced pre-formation. $16,500 pending wire after Chase account opens. Equity: 6.25% at $300k pre-money.
- **Next step**: Open Chase Business Complete Checking (branch visit, bring: Certificate of Formation + EIN letter + Operating Agreement + ID + $25 minimum deposit). Documents ready at `~/auri/workspace/`.
- **Form D**: File within 15 days of first wire at efts.sec.gov (free, Rule 506(b))
- **Products**: Songsheet (church SaaS), AI Trading (Polymarket scanner), Gauntlet (hardware HID)

### Architecture philosophy
- Proxmox (pve0-janga + pve1-janga) for all server workloads
- LXC containers preferred over VMs for lightweight services
- No Docker where native LXC works fine
- Cloudzy as DMZ/reverse proxy, homelab behind WireGuard relay

### Songsheet auth
- JWT-based, no sessions. Token stored in localStorage (conscious decision — no cookie-parser dep).
- Phase 2 = production backend (VMID 616). Phase 1 = song generation/PDF only (VMID 720).
- Never redeploy Phase 1 path for Phase 2 fixes.

### AI Trading
- Paper trade only until 2026-05-22 checkpoint minimum.
- No real capital deployed until calibration verified over ≥14 days.
- `barrier_v1` model (first-passage GBM) is the current model — correct formula is `P(min S_t ≤ K)`.

</decisions>

---

## Urgent Action Items (requires Joseph)

1. 🔴 **Change Songsheet bootstrap password** — `joseph@janga.dev / welcome123` is live on production. Run `node scripts/set-password.js joseph@janga.dev` on VMID 616.
2. 🔴 **Rotate Omniroute + Honcho API keys** — exposed in session logs. Admin panels: `http://10.25.6.17:20128/dashboard` and `https://app.honcho.dev`
3. 🔴 **Rotate vault credentials** — Kingdom SA password and Box password in plaintext Obsidian notes; MoonDev API key in plaintext.
4. 🟡 **Open Chase business account** — Certificate of Formation + EIN letter + Operating Agreement + ID + $25. Then grandpa wires $16,500. Documents at `~/auri/workspace/chase_business_account_checklist.md`.
5. 🟡 **Gauntlet**: Order OpenBCI Cyton (~$500) + Ag/AgCl electrodes before any PCB. Full brief: `~/auri/workspace/gauntlet_corrective_brief.md`

---

## Communication Style

- Auri/Sable is Joseph's AI partner — responses should treat Joseph as a skilled engineer with broad expertise
- Direct, concise, no hedging
- Technical depth is welcome; don't oversimplify
- Joseph works late-night autonomous sessions; morning briefings at `~/auri/workspace/morning_briefing_20260508.md`
