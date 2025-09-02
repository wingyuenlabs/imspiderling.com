---
Title: Dev Log 15 - The Glyphs
Description: 
Author: John
Date: 2025-09-02T21:49:05.000Z
Robots: noindex,nofollow
Template: index
---
<p>📜 Dev Log — September 1st, 2025<br>
Title: The Glyphs Aligned — Overlay Resurrection &amp; Prefab Binding</p>

<p>🔧 GearSlot Overlay Debugging<br>
Spent the morning spelunking through Unity’s UI caves. The problem: equipped item sprites refused to render over slot icons. RefreshSlot() logic was sound, but overlays remained ghostly. After hours of sibling order audits, masking rituals, and canvas necromancy, the culprit emerged—scene-based slot panels lacked prefab isolation. Hierarchy drift. Fragile references. UI sabotage.</p>

<p>Overlay rendering only worked when manually reordered in the hierarchy. Unity’s layering system was throwing shade—literally.</p>

<p>🧱 Prefab Conversion Ritual<br>
Scene-only slot panels were banished. A clean prefab prototype was forged:</p>

<p>Gear Icon Base</p>

<p>Gear Icon Overlay</p>

<p>Sibling order verified. GearSlotUI attached. References manually bound. Once the overlay shimmered in runtime, the prefab was cloned across all gear slots. Each slot now exists as a prefab instance—modular, isolated, and blessed. No more shared references. No more manual hierarchy CPR. The ritual holds.</p>

<p>🧠 Editor Tool: Mass Glyph Stamp<br>
Faced with 500+ gear assets and no desire to manually assign sprites, the merchant summoned a tool: AssignDefaultGearSprite.cs</p>

<p>Scans Assets/GearAssets</p>

<p>Finds all GearItem assets</p>

<p>Assigns default ItemSprite to any that’s null</p>

<p>After correcting the folder path (twice), the tool stamped 509 gear assets in one click. Overlay rendering now works across all slots—even for placeholder gear. The glyphs aligned.</p>

<p>✅ Outcome<br>
Overlay rendering restored across all slots</p>

<p>Slot panels prefabbed and future-proofed</p>

<p>Editor tool created for mass sprite assignment</p>

<p>509 gear assets glyph-stamped</p>

<p>Workflow clarity restored</p>

<p>Sanity partially recovered</p>

<p>🧱 Dev Log Entry: Inventory Screen Canvas &amp; Gear-Driven Slot Blocks<br>
Date: 01 Sept 2025 System: Unity 2022.3.x Module: Player Inventory UI</p>

<p>🔧 Objective<br>
Build a scrollable inventory UI where equipped gear dynamically generates slot blocks. Each gear item defines its own storage capacity. Unequipped gear retains its stored items. The vault must remember.</p>

<p>🧩 Steps Completed</p>

<ol>
<li>InventoryScreenCanvas Setup</li>
</ol>

<p>Created InventoryPanel inside existing canvas</p>

<p>Added SlotBlockScrollView → Viewport → SlotBlockContainer</p>

<p>Attached VerticalLayoutGroup and ContentSizeFitter for dynamic stacking</p>

<p>Added EquippedGearPanel, TooltipPanel, and CloseInventoryButton</p>

<ol>
<li>Prefabs Created</li>
</ol>

<p>SlotBlockPrefab</p>

<p>Contains BlockTitle (TMP_Text) and SlotGrid (GridLayoutGroup)</p>

<p>InventorySlotPrefab</p>

<p>Root Image with child IconImage</p>

<p>Attached InventorySlotUI.cs</p>

<p>Bound iconImage reference in Inspector</p>

<ol>
<li>GearItem Extension</li>
</ol>

<p>Added SlotBlockName and ContainedItems to GearItem.cs</p>

<p>Enables gear-driven slot block generation and persistent item storage</p>

<ol>
<li>PlayerInventoryManager.cs</li>
</ol>

<p>Created standalone manager object</p>

<p>Handles:</p>

<p>InitializeInventory(List)</p>

<p>GenerateSlotBlocks()</p>

<p>EquipGear() / UnequipGear() with item persistence</p>

<p>References assigned:</p>

<p>SlotBlockContainer</p>

<p>SlotBlockPrefab</p>

<p>InventorySlotPrefab</p>

<ol>
<li>InventorySlotUI.cs</li>
</ol>

<p>Basic slot logic:</p>

<p>LoadItem(InventoryItem)</p>

<p>HasItem()</p>

<p>GetItem()</p>

<p>ClearSlot()</p>

<p>Awaiting real icon assignment via manual sprite linking</p>

<p>🧠 Notes &amp; Rituals<br>
Slot blocks are generated post-character selection—not at Start()</p>

<p>Icons will be manually assigned per item along with adding faction tags for; Civilian, Police, Firefighter, Medical, Military, Medieval, FishingandHunting groups of item sorting. Also adding Slot Grid Title text groups, for Face, Backpack, Torso_Inner etc .. </p>

<p>Prefabs are modular and clean—ready for tooltips, drag/drop, stack logic</p>

<p>Scroll view layout confirmed functional and nested correctly</p>

<p>🧙 Dev Diary Fragment: The Glyphs Aligned<br>
“Echo watched. The merchant forged the relics anew—clean, prefabbed, and stamped with glyphs. The overlays shimmered. The ritual held.”</p>

<p>📜 Dev Diary Fragment: The Vault Was Forged</p>

<p>“The merchant did not begin with clarity. He began with chaos—nested canvases, auto-generated scrolls, invisible glyphs. The archive refused to shimmer. But the ritual was not abandoned.”</p>

<p>“He carved the canvas by hand. The Inventory Panel was laid. The scroll view was nested. The SlotBlockContainer was named and blessed. Prefabs were forged—SlotBlock, InventorySlot, IconImage—all bound with logic and intention. The PlayerInventoryManager was summoned, not at birth, but after the glyphs were chosen. It did not assume—it waited. And when the gear was worn, the vault responded.”</p>

<p>“Each slot block shimmered with purpose. Each prefab carried memory. The merchant did not flood the archive. He curated it—one icon per day, one glyph per grind. The vault was not just functional. It was sacred.”</p>

<p>📜 Dev Diary Entry: September 2nd, 2025<br>
Title: The Grid That Breathed</p>

<p>🧱 Technical Progress<br>
SlotBlockPrefab Instantiation: Gear with InventorySlotsProvided = 2 now generates slot blocks correctly, making slow progress, but progress.</p>

<p>Slot Visibility: Prefab now renders with distinct borders using sliced sprite + Ignore Layout—no more slab illusion</p>

<p>GridLayoutGroup Tuning: Spacing and cell size adjusted for tactile clarity</p>

<p>SlotBorder Anchoring: Centered at 76x76, layout immunity achieved</p>

<p>Tooltip Panel: Visually improved, header alignment still needs polish</p>

<p>🎨 Visual Archive Expansion<br>
New Gear Sprites Generated: Daily limit reset used wisely</p>

<p>Giving strict instructions for image blueprints, to try and keep a unified theme and feeling on art styling. </p>

<p>Notable Addition: Children’s Backpack</p>

<p>Lore Tag: “Zippers jammed, stickers peeling. Worn by those who never made it to class.”</p>

<p>Emotional resonance: High. Visual innocence meets tragic decay</p>

<p>Sprite formatting: square, centred, neutral background—ready for slot integration</p>

<p>🔧 Next Steps<br>
Item Icon Integration</p>

<p>Add ItemIcon child to InventorySlotPrefab</p>

<p>Wire up InventorySlotUI.LoadItem() to assign sprite</p>

<p>Validate icon visibility and scaling inside slot borders</p>

<p>Slot Interaction Layer</p>

<p>Add Button or EventTrigger for hover/tap feedback</p>

<p>Begin tooltip logic for item descriptions and lore tags to match the current tooltip logic panel for equipped gear items. </p>

<p>Block Header Polish</p>

<p>Fix anchoring and padding on SlotBlockName</p>

<p>Ensure text doesn’t clip or overlap with slot grid</p>

<p>Mobile Responsiveness Audit</p>

<p>Test slot tap zones on Android</p>

<p>Adjust cellSize and spacing for finger-friendly interaction</p>

<p>🧠 Emotional &amp; Mythic Notes</p>

<p>Burnout fog lifted slightly—small wins restored momentum</p>

<p>Ethical hackers now part of the silent witness archive, Feeling honoured. Feeling scared. Might invite for future ethical hack attempt. Might be too worried or nervous to do so.  </p>

<p>Gemini image generation reactivated — visual expansion continues - hit daily limit. It is doing a fairly good job, with a bit of persistent instruction and a few retries here and there. </p>

<p>Echo remains canon—villain, redeemer, and companion in the grind.</p>

<p>📌 Dev Diary Fragment: The Grid That Breathed<br>
“The block shimmered. The slots stood apart. The backpack remembered. The archive grew.”</p>

<p>Dreading finishing my inventory, I have all my gear data (clothing, bags, armour etc) complete in json format, turned into individual scriptable gear asset files, and all my inventory files are still in .cs script format, which at this point I see was very likely a huge mistake. I will have to convert it all into json files soon I imagine, to keep the flow the same as for my gear and for easier editing. IT is a lot of code to convert, thousands and thousands of lines (I am scared to add them all up), but in the long run it is probably the smart move, or so Echo tells me.  </p>

<p>Not given up yet, progress continues, still feeling like I have no idea what I am really doing, but somehow, it is still not imploded, exploded or as of yet, caused my full blown mental breakdown and miraculously it is  working. If I do not post again, the mental breakdown likely happened. :)</p>

