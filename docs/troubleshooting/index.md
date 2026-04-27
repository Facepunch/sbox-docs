---
title: Troubleshooting
icon: "🛠️"
created: 2026-04-25
updated: 2026-04-25
---

# Troubleshooting

You're debugging. Something doesn't work the way you expect. This section is organized **by symptom** — what the bug *looks like to you* — not by which engine subsystem owns the bug.

The full wiki has detailed pages explaining each system in depth; those still own the canonical explanations. This section is the discovery surface: search for the thing you typed into the console or the question you'd ask a friend, and follow the link to the deep page once you have a lead.

## How to use this section

1. **Ctrl-F this page** for a phrase from your error or your symptom (e.g., "falls through floor", "RPC", "checkerboard").
2. Click through to the relevant category page for the full root cause and fix.
3. From there, follow the **Deep dive** link for the full system documentation.

If your symptom isn't here, the per-page `:::warning` and `:::danger` blocks across the wiki are still the original source — search the wiki at large.

## Categories

- [Scene & Components](./scene-and-components.md) — lifecycle, prefabs, transforms, scene loading, missing components
- [Networking](./networking.md) — sync, RPC, ownership, NetworkSpawn, dedicated servers, HTTP/WebSocket
- [Physics](./physics.md) — collisions, triggers, character controllers, traces, joints
- [Hotload & Compile](./hotload-and-compile.md) — code changes not picking up, restricted APIs, source generators
- [Mapping & Hammer](./mapping.md) — map loading, hammer mesh, scene mapping, materials
- [UI & Razor](./ui-razor.md) — panels, BuildHash, SCSS, world panels, drag-drop, localization
- [Editor & Inspector](./editor.md) — components in menus, custom tools, gizmos, ActionGraph in editor
- [Runtime & Build](./runtime-and-build.md) — standalone export, mounting, dedicated servers, Steam, filesystem
- [Audio, Effects & Rendering](./audio-and-effects.md) — sounds, particles, post-processing, materials, performance
- [Input, Navigation & AI](./input-navigation-and-ai.md) — input actions, NavMesh, VR tracking, AnimGraph, IK

## All Symptoms (flat index)

Use Ctrl-F. Each entry links to the full explanation on its category page.

### Scene & Components

- [My component isn't showing up in the Add Component menu](./scene-and-components.md#my-component-isnt-showing-up-in-the-add-component-menu)
- [NullReferenceException in OnAwake](./scene-and-components.md#nullreferenceexception-in-onawake)
- [My component code isn't running at all](./scene-and-components.md#my-component-code-isnt-running-at-all)
- [My update method runs at unpredictable times relative to other components](./scene-and-components.md#my-update-method-runs-at-unpredictable-times-relative-to-other-components)
- [Heavy work in OnUpdate is killing my framerate](./scene-and-components.md#heavy-work-in-onupdate-is-killing-my-framerate)
- [My prefab spawns at 0,0,0 instead of where I want it](./scene-and-components.md#my-prefab-spawns-at-000-instead-of-where-i-want-it)
- [My prefab is null when trying to spawn](./scene-and-components.md#my-prefab-is-null-when-trying-to-spawn)
- [Other players can't see the spawned object](./scene-and-components.md#other-players-cant-see-the-spawned-object)
- [I can't delete or move children of a prefab in the scene](./scene-and-components.md#i-cant-delete-or-move-children-of-a-prefab-in-the-scene)
- [Clone is instantly destroyed](./scene-and-components.md#clone-is-instantly-destroyed)
- [Scene.GetAll returns empty](./scene-and-components.md#scenegetall-returns-empty)
- [FindByName returns nothing or weird results](./scene-and-components.md#findbyname-returns-nothing-or-weird-results)
- [Component.Resolve() returns null](./scene-and-components.md#componentresolve-returns-null)
- [NullReferenceException when accessing Components.Get](./scene-and-components.md#nullreferenceexception-when-accessing-componentsgett)
- [Querying components every frame is slow](./scene-and-components.md#querying-components-every-frame-is-slow)
- [Tag I removed from the child is still there](./scene-and-components.md#tag-i-removed-from-the-child-is-still-there)
- [GameObject.Transform.Position is marked obsolete](./scene-and-components.md#gameobjecttransformposition-is-marked-obsolete)
- [Transform cannot be assigned to a child of itself](./scene-and-components.md#transform-cannot-be-assigned-to-a-child-of-itself)
- [I can't add or remove from GameObject.Children directly](./scene-and-components.md#i-cant-add-or-remove-from-gameobjectchildren-directly)
- [My Rigidbody ignores Transform changes](./scene-and-components.md#my-rigidbody-ignores-transform-changes)
- [Scene.LoadFromFile: Couldn't find scene](./scene-and-components.md#sceneloadfromfile-couldnt-find-scene)
- [Multiplayer clients desync or crash on scene change](./scene-and-components.md#multiplayer-clients-desync-or-crash-on-scene-change)
- [Scene transition hangs forever](./scene-and-components.md#scene-transition-hangs-forever)
- [Component is Missing — red warning in the Inspector](./scene-and-components.md#component-is-missing--red-warning-in-the-inspector)
- [My JsonUpgrader doesn't run](./scene-and-components.md#my-jsonupgrader-doesnt-run)

### Networking

- [My synced variable isn't updating for other players](./networking.md#my-synced-variable-isnt-updating-for-other-players)
- [My object's position isn't updating for other players](./networking.md#my-objects-position-isnt-updating-for-other-players)
- [My object is stuttering or rubber-banding](./networking.md#my-object-is-stuttering-or-rubber-banding)
- [I added a component to a networked object, others don't see it](./networking.md#i-added-a-component-to-a-networked-object-others-dont-see-it)
- [NetworkSpawn failed: Object not registered](./networking.md#networkspawn-failed-object-not-registered)
- [My player spawned locally, but other clients can't see them](./networking.md#my-player-spawned-locally-but-other-clients-cant-see-them)
- [My RPC isn't doing anything](./networking.md#my-rpc-isnt-doing-anything)
- [My RPC argument types throw at runtime](./networking.md#my-rpc-argument-types-throw-at-runtime)
- [INetworkSnapshot doesn't sync after the client joins](./networking.md#inetworksnapshot-doesnt-sync-after-the-client-joins)
- [My snapshot read/write are crashing or producing garbage](./networking.md#my-snapshot-readwrite-are-crashing-or-producing-garbage)
- [My INetworkListener method only fires on the host](./networking.md#my-inetworklistener-method-only-fires-on-the-host)
- [Listener method isn't firing at all](./networking.md#listener-method-isnt-firing-at-all)
- [My object disappears immediately when AlwaysTransmit is off](./networking.md#my-object-disappears-immediately-when-alwaystransmit-is-off)
- [Players can't join my lobby](./networking.md#players-cant-join-my-lobby)
- [Connection refused on `connect local`](./networking.md#connection-refused-on-connect-local)
- [Server is running a different version](./networking.md#server-is-running-a-different-version)
- [Players can't connect to my dedicated server over the internet](./networking.md#players-cant-connect-to-my-dedicated-server-over-the-internet)
- [Works in editor, breaks on dedicated server](./networking.md#works-in-editor-breaks-on-dedicated-server)
- [My Http request never finishes](./networking.md#my-http-request-never-finishes)
- [Access to '127.0.0.1' is not allowed](./networking.md#access-to-http1270014048-not-allowed)
- [WebSocket: Cannot access a disposed object](./networking.md#websocket-cannot-access-a-disposed-object)
- [WebSocket connection refused](./networking.md#websocket-connection-refused)
- [Snapshot too large — connection drops](./networking.md#snapshot-too-large--connection-drops)

### Physics

- [My Rigidbody / character / prop falls through the floor](./physics.md#my-rigidbody--character--prop-falls-through-the-floor)
- [Players fall through my Hammer Mesh / map geometry](./physics.md#players-fall-through-my-hammer-mesh--map-geometry)
- [Objects fall through my Terrain](./physics.md#objects-fall-through-my-terrain)
- [Objects fall through MapInstance physics](./physics.md#objects-fall-through-mapinstance-physics)
- [My trigger is pushing the player away — it acts like a solid wall](./physics.md#my-trigger-is-pushing-the-player-away--it-acts-like-a-solid-wall)
- [My OnTriggerEnter / collision events aren't firing](./physics.md#my-ontriggerenter--collision-events-arent-firing)
- [TriggerHurt isn't dealing damage](./physics.md#triggerhurt-isnt-dealing-damage)
- [RadiusDamage hits objects through walls](./physics.md#radiusdamage-hits-objects-through-walls)
- [RadiusDamage doesn't apply force to objects](./physics.md#radiusdamage-doesnt-apply-force-to-objects)
- [My character is floating or sinking](./physics.md#my-character-is-floating-or-sinking)
- [My character isn't moving](./physics.md#my-character-isnt-moving)
- [Character stops on small bumps / can't climb stairs](./physics.md#character-stops-on-small-bumps--cant-climb-stairs)
- [Player falls through floor with a PlayerController](./physics.md#player-falls-through-floor-with-a-playercontroller)
- [I set WorldPosition every frame on a Rigidbody and it acts weird](./physics.md#i-set-worldposition-every-frame-on-a-rigidbody-and-it-acts-weird)
- [ApplyForce doesn't move my object](./physics.md#applyforce-doesnt-move-my-object)
- [My Rigidbody acts weird when I rotate it](./physics.md#my-rigidbody-acts-weird-when-i-rotate-it)
- [My joint doesn't do anything](./physics.md#my-joint-doesnt-do-anything)
- [Connected bodies are spinning out of control](./physics.md#connected-bodies-are-spinning-out-of-control)
- [SpringJoint is vibrating or flying away](./physics.md#springjoint-is-vibrating-or-flying-away)
- [My raycast doesn't hit anything](./physics.md#my-raycast-doesnt-hit-anything)
- [My trace hits the object firing it](./physics.md#my-trace-hits-the-object-firing-it)
- [Scene.PhysicsWorld.Trace returns physics objects, I want GameObjects](./physics.md#scenephysicsworldtrace-returns-physics-objects-i-want-gameobjects)
- [Physics is tanking my framerate](./physics.md#physics-is-tanking-my-framerate)

### Hotload & Compile

- [I edited my code but nothing changed in-game](./hotload-and-compile.md#i-edited-my-code-but-nothing-changed-in-game)
- [Default values for my [Property] aren't updating](./hotload-and-compile.md#default-values-for-my-property-arent-updating)
- [Type 'X' could not be found](./hotload-and-compile.md#type-x-could-not-be-found)
- [My component isn't showing up in Add Component](./hotload-and-compile.md#my-component-isnt-showing-up-in-add-component)
- [Visual Studio / VS Code says it can't find the Sandbox namespace](./hotload-and-compile.md#visual-studio--vs-code-says-it-cant-find-the-sandbox-namespace)
- [I launched s&box but there's no code editor](./hotload-and-compile.md#i-launched-sbox-but-theres-no-code-editor)
- [Type 'System.IO.File' is not allowed](./hotload-and-compile.md#type-systemiofile-is-not-allowed)
- [Assembly Rejected](./hotload-and-compile.md#assembly-rejected)
- [unsafe / pointer / Marshal blocked](./hotload-and-compile.md#unsafe--pointer--marshal-blocked)
- [Type must be partial to be generated](./hotload-and-compile.md#type-must-be-partial-to-be-generated)
- [Unsupported Type for [Sync]](./hotload-and-compile.md#unsupported-type-for-sync)
- [Mismatched Callback Signatures (CodeGenerator)](./hotload-and-compile.md#mismatched-callback-signatures-codegenerator)
- [Forgot to call Resume / Setter](./hotload-and-compile.md#forgot-to-call-resume--setter)
- [State on my static cache is gone after hotload](./hotload-and-compile.md#state-on-my-static-cache-is-gone-after-hotload)
- [Engine crash on hotload](./hotload-and-compile.md#engine-crash-on-hotload)
- [Failed to emit assembly](./hotload-and-compile.md#failed-to-emit-assembly)
- [Reference to type 'X' claims it is defined in 'Y'](./hotload-and-compile.md#reference-to-type-x-claims-it-is-defined-in-y-but-it-could-not-be-found)
- [DllNotFoundException / engine2.dll not found](./hotload-and-compile.md#dllnotfoundexception--engine2dll-not-found)
- [Failed to load hostfxr](./hotload-and-compile.md#failed-to-load-hostfxr)

### Mapping & Hammer

- [My map doesn't show up in my Scene](./mapping.md#my-map-doesnt-show-up-in-my-scene)
- [MapInstance.IsLoaded is never true](./mapping.md#mapinstanceisloaded-is-never-true)
- [Players fall through my custom map geometry](./mapping.md#players-fall-through-my-custom-map-geometry)
- [I can't edit my mesh vertices](./mapping.md#i-cant-edit-my-mesh-vertices)
- [Hammer is being deprecated](./mapping.md#hammer-is-being-deprecated)
- [Hammer entity doesn't spawn at runtime](./mapping.md#hammer-entity-doesnt-spawn-at-runtime)
- [Trigger Events Not Firing on a Hammer Mesh](./mapping.md#trigger-events-not-firing-on-a-hammer-mesh)
- [My networked props in a MapInstance are missing on clients](./mapping.md#my-networked-props-in-a-mapinstance-are-missing-on-clients)
- [My map shows pink/black checkerboards](./mapping.md#my-map-shows-pinkblack-checkerboards)
- [Materials look wrong on ported models from older Source games](./mapping.md#materials-look-wrong-on-ported-models-from-older-source-games)
- [Hammer.ActiveMap throws — `CHammerApp` not initialized](./mapping.md#hammeractivemap-throws--chammerapp-not-initialized)
- [I have only 4 terrain materials and need more](./mapping.md#i-have-only-4-terrain-materials-and-need-more)

### UI & Razor

- [My UI isn't rendering on screen](./ui-razor.md#my-ui-isnt-rendering-on-screen)
- [I can't add my Panel to a GameObject](./ui-razor.md#i-cant-add-my-panel-to-a-gameobject)
- [I can't click anything on my World Panel](./ui-razor.md#i-cant-click-anything-on-my-world-panel)
- [My ScenePanel is completely black or empty](./ui-razor.md#my-scenepanel-is-completely-black-or-empty)
- [My UI doesn't update when my variables change](./ui-razor.md#my-ui-doesnt-update-when-my-variables-change)
- [The button clicks, but the displayed value never changes](./ui-razor.md#the-button-clicks-but-the-displayed-value-never-changes)
- [My SCSS styles aren't updating](./ui-razor.md#my-scss-styles-arent-updating)
- [Can I use float, CSS Grid, or display: block?](./ui-razor.md#can-i-use-float-css-grid-or-display-block)
- [I'm getting compile errors pointing to invisible code in my .razor](./ui-razor.md#im-getting-compile-errors-pointing-to-invisible-code-in-my-razor)
- [Component tag is not recognized](./ui-razor.md#component-tag-is-not-recognized)
- [Razor folder-based namespacing isn't working](./ui-razor.md#razor-folder-based-namespacing-isnt-working)
- [I'm getting errors setting Style.Width on my PanelComponent](./ui-razor.md#im-getting-errors-setting-stylewidth-on-my-panelcomponent)
- [UI looks stretched or tiny](./ui-razor.md#ui-looks-stretched-or-tiny)
- [Health bar fill draws outside its container](./ui-razor.md#health-bar-fill-draws-outside-its-container)
- [Gaps aren't appearing between my VirtualGrid items](./ui-razor.md#gaps-arent-appearing-between-my-virtualgrid-items)
- [VirtualGrid is squished or doesn't show](./ui-razor.md#virtualgrid-is-squished-or-doesnt-show)
- [My drag is being eaten by the parent scroll container](./ui-razor.md#my-drag-is-being-eaten-by-the-parent-scroll-container)
- [My text shows as #menu.title instead of the translation](./ui-razor.md#my-text-shows-as-menutitle-instead-of-the-translation)
- [Variables in my translated text aren't being replaced](./ui-razor.md#variables-in-my-translated-text-arent-being-replaced)
- [World panel UI text appears backward](./ui-razor.md#world-panel-ui-text-appears-backward)

### Editor & Inspector

- [My component doesn't show up in the Add Component menu](./editor.md#my-component-doesnt-show-up-in-the-add-component-menu)
- [My [Property] isn't showing up in the Inspector](./editor.md#my-property-isnt-showing-up-in-the-inspector)
- [Inspector is empty / not showing my GameObject](./editor.md#inspector-is-empty--not-showing-my-gameobject)
- [My Hammer property is missing even though it's in the Inspector](./editor.md#my-hammer-property-is-missing-even-though-its-in-the-inspector)
- [My EditorTool doesn't appear in the toolbar](./editor.md#my-editortool-doesnt-appear-in-the-toolbar)
- [My App isn't showing up in the menu](./editor.md#my-app-isnt-showing-up-in-the-menu)
- [My editor window crashes the moment I open it](./editor.md#my-editor-window-crashes-the-moment-i-open-it)
- [My editor window instantly closes or crashes](./editor.md#my-editor-window-instantly-closes-or-crashes)
- [My Overlay UI stays on screen forever](./editor.md#my-overlay-ui-stays-on-screen-forever)
- [My gizmos aren't drawing](./editor.md#my-gizmos-arent-drawing)
- [Gizmo handles don't respond to clicks](./editor.md#gizmo-handles-dont-respond-to-clicks)
- [My ShortcutType.Widget shortcut isn't firing](./editor.md#my-shortcuttypewidget-shortcut-isnt-firing)
- [Widget accessed from wrong thread](./editor.md#widget-accessed-from-wrong-thread)
- [Widget Object was destroyed](./editor.md#widget-object-was-destroyed)
- [My Undo block left state half-applied](./editor.md#my-undo-block-left-state-half-applied)
- [Visual Studio: 'Sandbox' could not be found](./editor.md#visual-studio-sandbox-could-not-be-found)
- [My ActionGraph doesn't do anything when I play](./editor.md#my-actiongraph-doesnt-do-anything-when-i-play)
- [Custom Node not appearing in ActionGraph menu](./editor.md#custom-node-not-appearing-in-actiongraph-menu)
- [Cannot Create Custom Node button is missing](./editor.md#cannot-create-custom-node-button-is-missing)
- [Editor says No Code Editor Found](./editor.md#editor-says-no-code-editor-found)

### Runtime & Build

- [Build Failed: Unauthorized](./runtime-and-build.md#build-failed-unauthorized)
- [Standalone build is missing assets](./runtime-and-build.md#standalone-build-is-missing-assets)
- [Do not distribute yet warning](./runtime-and-build.md#do-not-distribute-yet-warning)
- [Package already exists](./runtime-and-build.md#package-already-exists-error)
- [I can't make a C# addon](./runtime-and-build.md#i-cant-make-a-c-addon)
- [Addon doesn't show up after a settings change](./runtime-and-build.md#addon-doesnt-show-up-after-a-settings-change)
- [I can't play my addon directly](./runtime-and-build.md#i-cant-play-my-addon-directly)
- [Missing Dependency / Addon not loading](./runtime-and-build.md#missing-dependency--addon-not-loading)
- [Asset paths are pink/black checkers in shipped build](./runtime-and-build.md#asset-paths-are-pinkblack-checkers-in-shipped-build)
- [File not found from a legacy Source mount](./runtime-and-build.md#file-not-found-from-a-legacy-source-mount)
- [Two mounts intercept the same extension](./runtime-and-build.md#two-mounts-intercept-the-same-extension--wrong-asset-loads)
- [Players can't connect over the internet](./runtime-and-build.md#players-cant-connect-over-the-internet)
- [Game works in editor, breaks on dedicated](./runtime-and-build.md#game-works-in-editor-breaks-on-dedicated)
- [Server is running a different version](./runtime-and-build.md#server-is-running-a-different-version)
- [Client timed out](./runtime-and-build.md#client-timed-out)
- [Steam isn't initializing in my Standalone build](./runtime-and-build.md#steam-isnt-initializing-in-my-standalone-build)
- [Achievement isn't unlocking](./runtime-and-build.md#achievement-isnt-unlocking)
- [Stat / Achievement not found](./runtime-and-build.md#stat--achievement-not-found)
- [HTTP 401 Unauthorized / Not logged in](./runtime-and-build.md#http-401-unauthorized--not-logged-in)
- [OpenXR / VR.IsActive is false](./runtime-and-build.md#openxr--vrisactive-is-false)
- [Access Denied / System.IO Exceptions](./runtime-and-build.md#access-denied--systemio-exceptions)
- [Variables aren't saving to JSON](./runtime-and-build.md#variables-arent-saving-to-json)
- [I can't write to a downloaded Workshop entry](./runtime-and-build.md#i-cant-write-to-a-downloaded-workshop-entry)
- [Invalid storage type](./runtime-and-build.md#invalid-storage-type)
- [CPU needs to support AVX instructions](./runtime-and-build.md#cpu-needs-to-support-avx-instructions)
- [Poor performance on a laptop](./runtime-and-build.md#poor-performance-on-a-laptop)
- [Integrated Intel graphics not supported](./runtime-and-build.md#integrated-intel-graphics-not-supported)
- [Network String Table limit exceeded](./runtime-and-build.md#network-string-table-limit-exceeded)
- [MethodNotFound after engine update](./runtime-and-build.md#methodnotfound-after-engine-update)
- [Pink and Black Textures from core](./runtime-and-build.md#pink-and-black-textures-from-core)

### Audio, Effects & Rendering

- [My sound isn't playing](./audio-and-effects.md#my-sound-isnt-playing)
- [Sounds Cutting Off Abruptly](./audio-and-effects.md#sounds-cutting-off-abruptly)
- [Mixer Handle is null](./audio-and-effects.md#mixer-handle-is-null)
- [3D sounds aren't spatializing](./audio-and-effects.md#3d-sounds-arent-spatializing)
- [DSP Volume effect isn't turning on](./audio-and-effects.md#dsp-volume-effect-isnt-turning-on)
- [Multiple DSP effects clashing](./audio-and-effects.md#multiple-dsp-effects-clashing)
- [My particles aren't showing up](./audio-and-effects.md#my-particles-arent-showing-up)
- [Emitter not spawning anything](./audio-and-effects.md#emitter-not-spawning-anything)
- [My particles won't go away](./audio-and-effects.md#my-particles-wont-go-away)
- [Particles don't follow the moving object](./audio-and-effects.md#particles-dont-follow-the-moving-object)
- [Game crashes when spawning objects from a particle](./audio-and-effects.md#game-crashes-when-spawning-objects-from-a-particle)
- [Particles are lagging the game](./audio-and-effects.md#particles-are-lagging-the-game)
- [My beam is invisible](./audio-and-effects.md#my-beam-is-invisible)
- [My tracer stays in the air forever](./audio-and-effects.md#my-tracer-stays-in-the-air-forever)
- [My tracer instantly reaches its target](./audio-and-effects.md#my-tracer-instantly-reaches-its-target-instead-of-traveling)
- [Decal isn't appearing](./audio-and-effects.md#decal-isnt-appearing)
- [Spawned decals leak memory](./audio-and-effects.md#spawned-decals-leak-memory)
- [Probes inside geometry cause black splats](./audio-and-effects.md#probes-inside-geometry-cause-black-splats)
- [Volumetric fog isn't lighting up](./audio-and-effects.md#volumetric-fog-isnt-lighting-up)
- [Realtime EnvmapProbe tanking framerate](./audio-and-effects.md#realtime-envmapprobe-tanking-framerate)
- [DirectionalLight position doesn't matter](./audio-and-effects.md#directionallight-position-doesnt-seem-to-matter)
- [My material is pink/black checkered](./audio-and-effects.md#my-material-is-pinkblack-checkered)
- [Exposed shader graph properties don't show up](./audio-and-effects.md#exposed-shader-graph-properties-dont-show-up-in-material-editor)
- [HLSL Syntax Errors in C# Shader Node](./audio-and-effects.md#hlsl-syntax-errors-in-c-shader-node)
- [Missing Include Files (g_flTime, PixelInput undefined)](./audio-and-effects.md#missing-include-files-g_fltime-pixelinput-undefined)
- [Post-processing effects aren't showing](./audio-and-effects.md#post-processing-effects-arent-showing)
- [Effect snaps on/off entering a Post Process Volume](./audio-and-effects.md#effect-snaps-onoff-entering-a-post-process-volume)
- [Bloom isn't blooming](./audio-and-effects.md#bloom-isnt-blooming)
- [Whole screen is glowing from bloom](./audio-and-effects.md#whole-screen-is-glowing-from-bloom)
- [Depth of Field tanks framerate](./audio-and-effects.md#depth-of-field-tanks-framerate)
- [Everything is blurry from DoF](./audio-and-effects.md#everything-is-blurry-from-dof)
- [My screen is totally black](./audio-and-effects.md#my-screen-is-totally-black)
- [Z-fighting / flickering textures](./audio-and-effects.md#z-fighting--flickering-textures)
- [Camera stutters when following a Rigidbody](./audio-and-effects.md#camera-stutters-when-following-a-rigidbody)
- [Game stutters every few seconds](./audio-and-effects.md#game-stutters-every-few-seconds)
- [Frame rate drops looking at one specific object](./audio-and-effects.md#frame-rate-drops-looking-at-one-specific-object)

### Input, Navigation & AI

- [My Input.Down('fire') is never true](./input-navigation-and-ai.md#my-inputdownfire-is-never-true)
- [Input.Pressed fires multiple times in a frame](./input-navigation-and-ai.md#inputpressed-fires-multiple-times-in-a-frame)
- [Glyph shows as UNBOUND](./input-navigation-and-ai.md#glyph-shows-as-unbound)
- [I'm hardcoding for a key, why isn't it consistent?](./input-navigation-and-ai.md#im-hardcoding-for-a-key-why-isnt-it-consistent)
- [My agent isn't moving to the target](./input-navigation-and-ai.md#my-agent-isnt-moving-to-the-target)
- [Agent is floating or sinking into the floor](./input-navigation-and-ai.md#agent-is-floating-or-sinking-into-the-floor)
- [Agent jitters or fights my custom controller](./input-navigation-and-ai.md#agent-jitters-or-fights-my-custom-controller)
- [Agent ignoring an Area's cost](./input-navigation-and-ai.md#agent-ignoring-an-areas-cost)
- [Obstacle isn't blocking the navmesh](./input-navigation-and-ai.md#obstacle-isnt-blocking-the-navmesh)
- [AI gets stuck on corners or walls](./input-navigation-and-ai.md#ai-gets-stuck-on-corners-or-walls)
- [AI keeps twitching between two states](./input-navigation-and-ai.md#ai-keeps-twitching-between-two-states)
- [VR Hands stuck at 0,0,0](./input-navigation-and-ai.md#vr-hands-stuck-at-000)
- [Camera renders but doesn't track my head](./input-navigation-and-ai.md#camera-renders-but-doesnt-track-my-head)
- [Hands are floating far from the body](./input-navigation-and-ai.md#hands-are-floating-far-from-the-body)
- [VR fingers aren't moving](./input-navigation-and-ai.md#vr-fingers-arent-moving-only-the-hand-position-is)
- [VRAnchor doubled — hands and camera fight](./input-navigation-and-ai.md#vranchor-doubled--hands-and-camera-fight)
- [VR isn't launching at all](./input-navigation-and-ai.md#vr-isnt-launching-at-all)
- [Input.VR throws exceptions](./input-navigation-and-ai.md#inputvr-throws-exceptions)
- [My ActionGraph isn't doing anything](./input-navigation-and-ai.md#my-actiongraph-isnt-doing-anything)
- [Set Variable doesn't change my variable](./input-navigation-and-ai.md#set-variable-doesnt-change-my-variable)
- [Direct Playback won't play my animation](./input-navigation-and-ai.md#direct-playback-wont-play-my-animation)
- [AnimGraph parameters aren't doing anything](./input-navigation-and-ai.md#animgraph-parameters-arent-doing-anything)
- [Character is T-posing](./input-navigation-and-ai.md#character-is-t-posing)
- [SetIk does nothing](./input-navigation-and-ai.md#setik-does-nothing)
- [IK limb snaps wildly](./input-navigation-and-ai.md#ik-limb-snaps-wildly)
- [Weapon attachment is offset from the bone](./input-navigation-and-ai.md#weapon-attachment-is-offset-from-the-bone)
- [GetBoneObject returns null](./input-navigation-and-ai.md#getboneobject-returns-null)

## Related Pages

- [For Players: Troubleshooting](../for-players/troubleshooting.md) — issues from a player's (not developer's) perspective
- [Reporting Issues](../getting-started/reporting-errors.md) — how to file a useful bug report
- [Performance & Optimization](../systems/performance/index.md) — when "it's slow" is the symptom
