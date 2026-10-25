--============================================================================
--  FUST SURVIVAL  |  ORE + LOOT CRATE ESP                          [Nexa UI]
--  place 79862723087324
--
--  GUI      Nexa UI - https://github.com/OgTen/Nexa-UI
--  Stop it  _G.FUST_ESP.unload()
--  Menu     RightShift
--
--  WHAT IT DRAWS
--    Ore      MetalNode / SulfurNode / StoneNode in Workspace.Nodes.
--             A metal or sulfur node keeps its "Ore" child only while it is
--             still mineable, so that child is the ready check. Stone nodes
--             never have one and are listed as Stone.
--    Loot     LootCrate / MilitaryCrate / EliteCrate / LootBarrel inside
--             Workspace.Loot, plus anything in Workspace.Airdrops.
--    Contents Crate contents are rolled on the server when the lid opens. A
--             closed crate carries only CrateKind / Id / Kind / Open and no
--             remote ships the roll, so they CANNOT be read client-side.
--             The second line therefore stays factual - roll count and pool
--             size - and naming the rarest DROPS is opt-in via "Show possible
--             drops", clearly labelled as what the type can give.
--    Odds     The Tables tab shows the module's real per-crate odds, rebuilt
--             from all of its roll stages rather than raw weights, with the
--             Saturday-only items removed (the module filters them out too).
--
--  KEEPING IT LEGIBLE
--    Only the nearest few targets get a text label, only nearby crates get an
--    item line, and every label is measured and collision tested so two labels
--    never stack. Everything else is just a box. Tuned from the Visuals tab.
--============================================================================

local Players    = game:GetService("Players")
local RunService = game:GetService("RunService")
local lp         = Players.LocalPlayer

-- Matcha's type() reports "userdata" for vectors; typeof() gives Vector3.
local typeOf = (type(typeof) == "function") and typeof or type

-- running it twice should replace the old boxes, not stack another set on top
local prev = rawget(_G, "FUST_ESP")
if type(prev) == "table" and type(prev.unload) == "function" then
    pcall(prev.unload)
end

--============================================================================
--  CONFIG
--============================================================================

local S = {
    ore = {
        on     = true,
        metal  = true,
        sulfur = true,
        stone  = false,
        unmined = true,                      -- skip nodes that are already mined out
        color  = Color3.fromRGB(90, 240, 200),
    },
    loot = {
        on       = true,
        wooden   = true,
        military = true,
        elite    = true,
        barrel   = true,
        locked   = true,
        heli     = true,
        airdrop  = true,
        hideOpen = true,                     -- opened crates are already emptied
        preview  = 2,                        -- how many possible drops to name
        previewPossible = false,             -- OFF by default: this is the loot table,
                                             -- never the crate's actual contents
        itemsOnNearest = 3,                  -- only this many crates get a second line
        color    = Color3.fromRGB(255, 190, 70),
    },
    draw = {
        names   = true,
        dist    = true,
        tracers = false,
        maxDist = 300,
        labelMax  = 8,                       -- label blocks on screen at once
        labelDist = 250,                     -- past this only the box is drawn
    },
}

local MAX_TARGETS = 56                       -- drawing pool size, also the cap

--============================================================================
--  WORLD DEFINITIONS
--============================================================================

local ORE_KINDS = {
    MetalNode  = { key = "metal",  label = "Metal Ore" },
    SulfurNode = { key = "sulfur", label = "Sulfur Ore" },
    StoneNode  = { key = "stone",  label = "Stone" },
}

local CRATE_KINDS = {
    LootCrate     = { key = "wooden",   label = "Wooden Crate",   tbl = "crate_wooden" },
    MilitaryCrate = { key = "military", label = "Military Crate", tbl = "crate_military" },
    EliteCrate    = { key = "elite",    label = "Elite Crate",    tbl = "crate_elite" },
    LootBarrel    = { key = "barrel",   label = "Loot Barrel",    tbl = "barrel" },
    LockedCrate   = { key = "locked",   label = "Locked Crate",   tbl = "locked_crate" },
    HeliCrate     = { key = "heli",     label = "Heli Crate",     tbl = "heli_crate" },
    Airdrop       = { key = "airdrop",  label = "Airdrop",        tbl = "airdrop" },
}

--============================================================================
--  LOOT TABLES
--  Taken from ReplicatedStorage.Fust.LootTables - the game's own module - and
--  converted from its internal shape into real odds:
--
--      [table] = { rollsLo, rollsHi, "item:chance item:chance ..." }
--
--  A crate is NOT one weighted pick. The module rolls its weighted `entries`
--  N times, then rolls extra weighted `pools`, then rolls a pile of flat
--  probability extras (fuel, sights, bows, shells, rockets, scar, night
--  vision). Those three sit on different scales, so each item's odds here are
--  1-(1-p1)*(1-p2)*... across every stage that can produce it: the chance that
--  a single crate of that type contains at least one.
--
--  Event-only items are left out on purpose. metal_chest_plate, heavy_plate_-
--  pants, metal_facemask, m249, coffee_can_helmet and the road sign gear live
--  in Items.SaturdayIds, and the live module filters them out unless the
--  Saturday event is running - listing them as ordinary drops is what made the
--  tables look wrong.
--============================================================================

local BAKED = {
    barrel = { 1, 2, "semi_body:1.188 bone_knife:3.13 burlap_shirt:4.166 burlap_trousers:4.166 burlap_headwrap:4.166 burlap_shoes:4.166 bandage:5 stone_hatchet:5.199 stone_pickaxe:5.199 tech_trash:5.893 sheet_metal:5.893 can_of_tuna:6 wood:6.227 stones:6.227 mushroom:6.227 torch:6.227 sulfur:6.227 corn:6.227 water_bottle:8 can_of_beans:8 tarp:11.665 charcoal:12.32 wooden_arrow:14.32 metal_spring:15.445 sewing_kit:15.445 gears:19.17 metal_pipe:21.011 metal_blade:22.838 cloth:24.087 rope:26.449 metal_fragments:27.882 low_grade_fuel:30 scrap:100" },
    crate_wooden = { 2, 3, "satchel_charge:0.2 compound_bow:1 crossbow:1.2 smg_body:4.189 bone_club:4.921 code_lock:4.921 semi_body:6.923 bandage:8 wood_armor_pants:8.12 metal_hatchet:8.12 metal_pickaxe:8.12 wood_chestplate:8.12 burlap_headwrap:9.694 burlap_shoes:9.694 burlap_shirt:9.694 burlap_trousers:9.694 wooden_spear:9.694 water_bottle_full:10 can_of_tuna:10 stone_pickaxe:11.253 stone_hatchet:11.253 wood:12.795 sulfur:12.795 hunting_bow:12.795 can_of_beans:14 tech_trash:15.161 tarp:19.885 sheet_metal:19.885 sewing_kit:22.945 wooden_arrow:24.554 cloth:24.554 low_grade_fuel:25 rope:25.934 metal_pipe:28.852 gears:28.852 metal_blade:28.852 metal_spring:28.852 metal_fragments:35.306 scrap:100" },
    crate_military = { 4, 6, "assault_rifle:0.744 scar:0.8 c4:1.114 bolt_action_rifle:1.2 hv_rocket:1.5 incendiary_rocket:1.5 drum_smg:1.5 scope_4x:2 tactical_smg:2 semi_auto_rifle:2.156 pump_shotgun:2.5 lever_carbine:2.5 holo_sight:3 double_barrel:3 crossbow:3 compound_bow:3 satchel_charge:3.674 supply_signal:4 service_pistol:4 semi_auto_pistol:5 explosive_rifle_ammo:5 red_dot_sight:6 slug_shell:6 metal_door:6.062 explosives:7.239 custom_smg:7.239 pipe_shotgun:10.698 revolver:11.827 code_lock:11.827 metal_hatchet:11.827 metal_pickaxe:11.827 bandage:12 buckshot_shell:12 sewing_kit:12.035 rope:12.035 rifle_ammo:17.305 burlap_headwrap:17.305 sheet_metal:17.709 sulfur:22.508 high_quality_metal:22.508 metal_pipe:23.157 smg_body:25.797 semi_body:25.797 gears:28.382 low_grade_fuel:30 wood_chestplate:32.128 wood_armor_pants:32.128 tech_trash:38.179 metal_fragments:40.767 metal_spring:42.757 handmade_shell:48.504 gunpowder:48.504 pistol_bullet:71.017 scrap:100" },
    crate_elite = { 5, 7, "assault_rifle:3.346 scar:4 hv_rocket:4 incendiary_rocket:4 crossbow:4 compound_bow:4 pump_shotgun:4 tactical_smg:4 drum_smg:4 large_backpack:6 bolt_action_rifle:6 c4:6.598 scope_4x:10 explosive_rifle_ammo:10 buckshot_shell:10 slug_shell:10 holo_sight:12 satchel_charge:13.691 code_lock:13.691 semi_auto_rifle:14.545 sheet_metal:18.136 explosives:21.92 wood_chestplate:25.783 wood_armor_pants:25.783 bandage:25.783 gears:26.399 custom_smg:29.485 smg_body:34.14 semi_body:34.14 metal_fragments:45.77 gunpowder:45.77 tech_trash:48.103 high_quality_metal:51.339 metal_spring:54.349 pistol_bullet:56.422 rifle_ammo:86.284 scrap:100" },
    locked_crate = { 6, 9, "armored_double_door:4.851 hv_rocket:8 incendiary_rocket:8 armored_door:9.496 code_lock:11.744 bolt_action_rifle:15 supply_signal:15 large_backpack:20 buckshot_shell:20 slug_shell:20 scope_4x:25 scar:25 explosive_rifle_ammo:25 smg_body:31.981 gears:31.981 custom_smg:34.187 gunpowder:34.61 satchel_charge:34.61 bandage:34.61 semi_body:41.038 pistol_bullet:45.212 metal_fragments:45.212 explosives:45.212 semi_auto_rifle:48.183 metal_spring:49.33 high_quality_metal:49.929 c4:50.958 assault_rifle:56.819 tech_trash:56.88 rifle_ammo:97.152 scrap:100" },
    heli_crate = { 6, 8, "lr_sniper:4 scope_4x:8 bolt_action_rifle:8 buckshot_shell:12 slug_shell:12 satchel_charge:14.299 scar:18 c4:20.055 gears:23.927 large_medkit:26.808 bandage:26.808 semi_body:31.406 gunpowder:34.248 semi_auto_rifle:38.112 metal_spring:38.623 assault_rifle:40.11 tech_trash:45.567 metal_fragments:47.204 explosives:47.204 medical_syringe:47.204 high_quality_metal:85.008 rifle_ammo:95.932 scrap:100" },
    airdrop = { 7, 11, "night_vision_goggles:1.8 scar:3.5 armored_double_door:4.046 bolt_action_rifle:5 crossbow:5 compound_bow:5 drum_smg:5 scope_4x:6 hv_rocket:6 incendiary_rocket:6 pump_shotgun:6 tactical_smg:6 armored_door:7.946 holo_sight:8 service_pistol:10 red_dot_sight:12 semi_auto_pistol:15 slug_shell:15 code_lock:15.326 c4:18.815 explosive_rifle_ammo:20 buckshot_shell:20 sheet_metal:20.988 pipe_shotgun:25.412 smg_body:27.435 gears:27.435 semi_body:27.435 semi_auto_rifle:32.696 satchel_charge:34.415 metal_spring:39.506 tech_trash:39.506 gunpowder:39.867 revolver:39.867 explosives:39.867 handmade_shell:44.912 custom_smg:44.912 low_grade_fuel:50 metal_fragments:64.925 rifle_ammo:69.933 high_quality_metal:81.556 pistol_bullet:88.364 scrap:100" },
}

local LOOT = {}

for name, d in pairs(BAKED) do
    local items = {}
    for id, chance in string.gmatch(d[3], "([%w_]+):([%d%.]+)") do
        items[#items + 1] = { id = id, p = tonumber(chance) or 0 }
    end
    table.sort(items, function(a, b) return a.p < b.p end)
    LOOT[name] = { rolls = { d[1], d[2] }, items = items }
end

local lootSource = "LootTables module"

local tableCount = 0
for _ in pairs(LOOT) do tableCount = tableCount + 1 end

local function pretty(id) return (string.gsub(id, "_", " ")) end

-- 14% and 0.014% both need to read clearly, so the precision follows the size
local function pct(p)
    if p >= 10 then return string.format("%.0f%%", p) end
    if p >= 1 then return string.format("%.1f%%", p) end
    if p >= 0.1 then return string.format("%.2f%%", p) end
    return string.format("%.3f%%", p)
end

-- preview strings are rebuilt only when the slider moves
local previewCache, previewFor = {}, nil
local function previewLine(tableKey, count)
    if previewFor ~= count then previewCache, previewFor = {}, count end
    local hit = previewCache[tableKey]
    if hit then return hit end

    local line = ""
    local t = LOOT[tableKey]
    if t and count > 0 then
        local shown = math.min(count, #t.items)
        local parts = {}
        for i = 1, shown do parts[#parts + 1] = pretty(t.items[i].id) end
        line = table.concat(parts, ", ")
        local more = #t.items - shown
        if more > 0 then line = line .. "  +" .. more end
        if #line > 46 then line = string.sub(line, 1, 43) .. "..." end
    end
    previewCache[tableKey] = line
    return line
end

--============================================================================
--  LOCAL PLAYER
--============================================================================

local rootCache
local vpCache    = Vector2.new(1280, 720)
local running    = true
local targets    = {}
local lastDrawn  = 0
local lastLabels = 0

task.spawn(function()
    while running do
        local char = lp.Character
        rootCache = char and char:FindFirstChild("HumanoidRootPart") or nil

        local ok, cam = pcall(function()
            return workspace.CurrentCamera or workspace:FindFirstChild("Camera")
        end)
        if ok and cam then
            local ok2, vp = pcall(function() return cam.ViewportSize end)
            if ok2 and typeOf(vp) == "Vector2" then vpCache = vp end
        end

        task.wait(0.4)
    end
end)

-- How this executor exposes a model's position is fixed for the session, so it
-- is detected once instead of pcall-probing every node on every sweep.
local posMode

local function posOf(model)
    if posMode == nil then
        local ok, p = pcall(function() return model.Position end)
        posMode = (ok and typeOf(p) == "Vector3") and "position" or "pivot"
    end

    if posMode == "position" then
        local ok, p = pcall(function() return model.Position end)
        if ok and typeOf(p) == "Vector3" then return p end
    end

    local ok2, cf = pcall(function() return model:GetPivot() end)
    if ok2 and cf then
        local ok3, p3 = pcall(function() return cf.Position end)
        if ok3 and typeOf(p3) == "Vector3" then return p3 end
    end

    local ok4, pp = pcall(function() return model.PrimaryPart end)
    if ok4 and pp then
        local ok5, p5 = pcall(function() return pp.Position end)
        if ok5 and typeOf(p5) == "Vector3" then return p5 end
    end

    local ok6, kids = pcall(function() return model:GetChildren() end)
    if ok6 and type(kids) == "table" then
        for i = 1, #kids do
            local c = kids[i]
            local ok7, p7 = pcall(function() return c.Position end)
            if ok7 and typeOf(p7) == "Vector3" then return p7 end
        end
    end
    return nil
end

--============================================================================
--  COLLECTOR  -  slow walk of the two folders, no drawing
--============================================================================

local collectFails = 0

local function collect()
    local out   = {}
    local myPos = rootCache and rootCache.Position
    local maxD  = S.draw.maxDist

    if S.ore.on then
        local nodes = workspace:FindFirstChild("Nodes")
        if nodes then
            local list = nodes:GetChildren()
            for i = 1, #list do
                local m = list[i]
                local def = ORE_KINDS[m.Name]
                if def and S.ore[def.key] then
                    local ready
                    if def.key == "stone" then
                        ready = true
                    else
                        ready = m:FindFirstChild("Ore") ~= nil
                    end

                    if ready or not S.ore.unmined then
                        local p = posOf(m)
                        if p then
                            local d = myPos and (p - myPos).Magnitude or 0
                            if d <= maxD then
                                local hp    = tonumber(m:GetAttribute("Hp"))
                                local maxhp = tonumber(m:GetAttribute("MaxHp"))
                                local label = def.label
                                if hp and maxhp and maxhp > 0 and hp < maxhp then
                                    label = string.format("%s  %.0f%%", def.label, 100 * hp / maxhp)
                                end
                                out[#out + 1] = {
                                    pos = p, dist = d, h = 4, kind = "ore",
                                    label = label, sub = "",
                                    color = S.ore.color,
                                }
                            end
                        end
                    end
                end
            end
        end
    end

    if S.loot.on then
        local folders = { workspace:FindFirstChild("Loot"), workspace:FindFirstChild("Airdrops") }
        for f = 1, #folders do
            local folder = folders[f]
            if folder then
                local list = folder:GetChildren()
                for i = 1, #list do
                    local m = list[i]
                    local def = CRATE_KINDS[m.Name]
                    if def and S.loot[def.key] ~= false then
                        local open = m:GetAttribute("Open") == true
                        if not (open and S.loot.hideOpen) then
                            local p = posOf(m)
                            if p then
                                local d = myPos and (p - myPos).Magnitude or 0
                                if d <= maxD then
                                    -- Factual by default: type + roll count + pool size.
                                    -- Naming rarest drops is opt-in, because a closed
                                    -- crate's real contents only exist on the server.
                                    local sub = ""
                                    local t = LOOT[def.tbl]
                                    if t then
                                        sub = string.format("%d-%d rolls", t.rolls[1], t.rolls[2])
                                        local body = ""
                                        if S.loot.previewPossible and S.loot.preview > 0 then
                                            body = previewLine(def.tbl, S.loot.preview)
                                        end
                                        if body ~= "" then
                                            sub = sub .. " | could drop: " .. body
                                        else
                                            sub = string.format("%s, %d possible items", sub, #t.items)
                                        end
                                    end
                                    out[#out + 1] = {
                                        pos = p, dist = d, h = 3, kind = "crate",
                                        label = def.label, sub = sub,
                                        color = S.loot.color,
                                    }
                                end
                            end
                        end
                    end
                end
            end
        end
    end

    if #out > 1 then
        table.sort(out, function(a, b) return a.dist < b.dist end)
    end
    for i = #out, MAX_TARGETS + 1, -1 do out[i] = nil end
    return out
end

task.spawn(function()
    while running do
        local ok, res = pcall(collect)
        if ok then
            targets = res
        else
            collectFails = collectFails + 1
            if collectFails <= 3 then print("[FUST] collect error: " .. tostring(res)) end
        end
        task.wait(0.75)
    end
end)

--============================================================================
--  DRAWING POOL
--============================================================================

local pool = {}
for i = 1, MAX_TARGETS do
    local box = Drawing.new("Square")
    box.Thickness  = 1
    box.Filled     = false
    box.ZIndex     = 2
    box.Visible    = false

    local name = Drawing.new("Text")
    name.Size    = 13
    name.Center  = true
    name.Outline = true
    name.ZIndex  = 2
    name.Visible = false

    local sub = Drawing.new("Text")
    sub.Size    = 12
    sub.Center  = true
    sub.Outline = true
    sub.ZIndex  = 2
    sub.Visible = false

    local tracer = Drawing.new("Line")
    tracer.Thickness = 1
    tracer.ZIndex    = 2
    tracer.Visible   = false

    pool[i] = { box = box, name = name, sub = sub, tracer = tracer }
end

--============================================================================
--  RENDERER  -  reads the table only, no tree lookups
--  Labels are budgeted and collision tested. One label per target turns a
--  cluster of crates into a wall of overlapping text, so only the nearest
--  S.draw.labelMax targets inside labelDist get one, and a label that would
--  land on one already placed is dropped (its box still draws).
--============================================================================

local RECTS = 128
local rects = {}
for i = 1, RECTS do rects[i] = { x = 0, y = 0, w = 0, h = 0 } end

local function rectHits(x, y, w, h, count)
    for i = 1, count do
        local r = rects[i]
        if x < r.x + r.w and x + w > r.x and y < r.y + r.h and y + h > r.y then
            return true
        end
    end
    return false
end

local function rectTake(x, y, w, h, count)
    if count >= RECTS then return count end
    local r = rects[count + 1]
    r.x, r.y, r.w, r.h = x, y, w, h
    return count + 1
end

-- real pixel width, measured by the renderer, so boxes hug the text
local function textWidth(drawObj, text, fallbackSize)
    drawObj.Text = text
    local b = drawObj.TextBounds
    if typeOf(b) == "Vector2" then return b.X end
    return #text * fallbackSize * 0.52
end

local conn = RunService.RenderStepped:Connect(function()
    local list  = targets
    local myPos = rootCache and rootCache.Position
    local vp    = vpCache
    local n     = 0
    local used  = 0
    local labelsLeft = S.draw.names and S.draw.labelMax or 0
    local itemLines  = S.loot.itemsOnNearest or 0
    local labelsOn   = 0

    for i = 1, #list do
        if n >= MAX_TARGETS then break end
        local t = list[i]
        local top, v1 = WorldToScreen(t.pos + Vector3.new(0, t.h, 0))
        local bot, v2 = WorldToScreen(t.pos - Vector3.new(0, t.h * 0.4, 0))
        if v1 and v2 then
            n = n + 1
            local e = pool[n]

            local h = bot.Y - top.Y
            if h < 3 then h = 3 end
            local w = math.max(6, h * 0.8)

            e.box.Position = Vector2.new(top.X - w * 0.5, top.Y)
            e.box.Size     = Vector2.new(w, h)
            e.box.Color    = t.color
            e.box.Visible  = true

            local named = false
            if labelsLeft > 0 and t.dist <= S.draw.labelDist and h >= 7 then
                local text = t.label
                if S.draw.dist and myPos then
                    local d = math.floor((t.pos - myPos).Magnitude)
                    if d ~= t._d then
                        t._d = d
                        t._name = string.format("%s  [%dm]", t.label, d)
                    end
                    text = t._name or t.label
                end

                local tw = textWidth(e.name, text, 13)
                local ty = top.Y - 16
                if not rectHits(top.X - tw * 0.5, ty, tw, 15, used) then
                    used = rectTake(top.X - tw * 0.5, ty, tw, 15, used)
                    labelsLeft = labelsLeft - 1
                    labelsOn = labelsOn + 1
                    named = true

                    e.name.Color    = t.color
                    e.name.Position = Vector2.new(top.X, ty)
                    e.name.Visible  = true
                end
            end

            if not named then
                e.name.Visible = false
                e.sub.Visible  = false
            elseif t.sub ~= "" and itemLines > 0 then
                local sw = textWidth(e.sub, t.sub, 12)
                local sy = bot.Y + 2
                if not rectHits(top.X - sw * 0.5, sy, sw, 15, used) then
                    used = rectTake(top.X - sw * 0.5, sy, sw, 15, used)
                    itemLines = itemLines - 1
                    e.sub.Color    = t.color
                    e.sub.Position = Vector2.new(top.X, sy)
                    e.sub.Visible  = true
                else
                    e.sub.Visible = false
                end
            else
                e.sub.Visible = false
            end

            if S.draw.tracers then
                e.tracer.From    = Vector2.new(vp.X * 0.5, vp.Y)
                e.tracer.To      = Vector2.new(top.X, bot.Y)
                e.tracer.Color   = t.color
                e.tracer.Visible = true
            else
                e.tracer.Visible = false
            end
        end
    end

    for i = n + 1, MAX_TARGETS do
        local e = pool[i]
        e.box.Visible    = false
        e.name.Visible   = false
        e.sub.Visible    = false
        e.tracer.Visible = false
    end

    lastDrawn  = n
    lastLabels = labelsOn
end)

--============================================================================
--  NEXA UI
--============================================================================

local NEXA_URL       = "https://raw.githubusercontent.com/OgTen/Nexa-UI/main/nexa-ui.lua"
local NEXA_BOOTSTRAP = "https://raw.githubusercontent.com/OgTen/Nexa-UI/refs/heads/main/nexa-showcase.lua"

local function isNexa(v)
    return type(v) == "table"
        and type(v.CreateWindow) == "function"
        and type(v.AddTab) == "function"
        and type(v.Notify) == "function"
end

local function httpGet(url)
    local ok, body = pcall(function() return game:HttpGet(url) end)
    if ok and type(body) == "string" then return body end
    if type(httpget) == "function" then
        local ok2, body2 = pcall(httpget, url)
        if ok2 and type(body2) == "string" then return body2 end
    end
    return nil
end

-- Some VMs drop a chunk's return value, so every place the library publishes
-- itself is checked after the chunk runs: plain globals, getgenv and shared.
local function findExported()
    local env = type(getgenv) == "function" and getgenv() or nil
    local list = {}
    if env then
        list[#list + 1] = env.Nexa
        list[#list + 1] = env.UI
    end
    list[#list + 1] = rawget(_G, "Nexa")
    list[#list + 1] = rawget(_G, "UI")
    if type(shared) == "table" then
        list[#list + 1] = shared.Nexa
        list[#list + 1] = shared.UI
    end
    for i = 1, #list do
        if isNexa(list[i]) then return list[i] end
    end
    return nil
end

local function loadNexa()
    local found = findExported()
    if found then return found end
    if type(loadstring) ~= "function" then return nil end

    local src = httpGet(NEXA_URL)
    if src then
        local chunk = loadstring(src)
        if chunk then
            local ok, lib = pcall(chunk)
            if ok and isNexa(lib) then return lib end
            found = findExported()
            if found then return found end
        end
    end

    -- fallback: the showcase bootstraps the same library and exports it
    local boot = httpGet(NEXA_BOOTSTRAP)
    if boot then
        local chunk = loadstring(boot)
        if chunk then pcall(chunk) end
        found = findExported()
        if found then return found end
    end
    return nil
end

local Nexa = loadNexa()

if Nexa then
    Nexa:CreateWindow({
        Title    = "Fust ESP",
        Subtitle = "Ore + loot crates",
        Width    = 780,
        Height   = 540,
        Theme    = "Midnight",
        Background = "none",
        MenuKey  = "RightShift",
        Opacity  = 100,
    })

    -- dropdown options are title case, state keys are lowercase
    local function setFromList(target, list, keys)
        for i = 1, #keys do target[keys[i]] = false end
        for i = 1, #list do
            target[string.lower(tostring(list[i]))] = true
        end
    end

    ------------------------------------------------------------------ Ore
    local oreTab = Nexa:AddTab({ Title = "Ore", Icon = "layers", Select = true })

    local oreMain = oreTab:AddSection("Ore ESP", "Metal, sulfur and stone nodes", { Column = "left" })
    oreMain:AddToggle({
        Title = "Ore ESP",
        Description = "Box every ore node in range.",
        Default = true,
        ConfigKey = "fust.ore.on",
        Callback = function(v) S.ore.on = v end,
    })
    oreMain:AddDropdown({
        Title = "Node types",
        Description = "Which nodes get a box.",
        Options = { "Metal", "Sulfur", "Stone" },
        Default = { "Metal", "Sulfur" },
        Multi = true,
        ConfigKey = "fust.ore.types",
        Callback = function(v)
            setFromList(S.ore, v, { "metal", "sulfur", "stone" })
        end,
    })
    oreMain:AddToggle({
        Title = "Only unmined",
        Description = "Metal and sulfur nodes only while their Ore chunk is still there.",
        Default = true,
        ConfigKey = "fust.ore.unmined",
        Callback = function(v) S.ore.unmined = v end,
    })
    oreMain:AddColorPicker({
        Title = "Ore color",
        Default = S.ore.color,
        DefaultAlpha = 1,
        ConfigKey = "fust.ore.color",
        Callback = function(c) S.ore.color = c end,
    })

    local oreNotes = oreTab:AddSection("What the ore check reads", "So you can trust the boxes", { Column = "right" })
    oreNotes:AddLabel({
        Title = "Ore child = still mineable",
        Description = "A MetalNode or SulfurNode carries a child named Ore while it has ore left. It disappears once the node is mined out.",
    })
    oreNotes:AddDivider({ Title = "Node types" })
    oreNotes:AddLabel({ Title = "MetalNode", Description = "metal ore, ~80 on a fresh map" })
    oreNotes:AddLabel({ Title = "SulfurNode", Description = "sulfur ore, ~52 on a fresh map" })
    oreNotes:AddLabel({ Title = "StoneNode", Description = "stone only, never carries an Ore chunk" })
    oreNotes:AddStatus({ Title = "Loot data", Default = lootSource, Tone = "accent" })

    ------------------------------------------------------------------ Loot
    local lootTab = Nexa:AddTab({ Title = "Loot", Icon = "folder" })

    local lootMain = lootTab:AddSection("Loot ESP", "Crates, barrels and airdrops", { Column = "left" })
    lootMain:AddToggle({
        Title = "Loot ESP",
        Description = "Box loot containers in range.",
        Default = true,
        ConfigKey = "fust.loot.on",
        Callback = function(v) S.loot.on = v end,
    })
    lootMain:AddDropdown({
        Title = "Container types",
        Options = { "Wooden", "Military", "Elite", "Barrel", "Locked", "Heli", "Airdrop" },
        Default = { "Wooden", "Military", "Elite", "Barrel", "Locked", "Heli", "Airdrop" },
        Multi = true,
        VisibleRows = 5,
        ConfigKey = "fust.loot.types",
        Callback = function(v)
            setFromList(S.loot, v, { "wooden", "military", "elite", "barrel", "locked", "heli", "airdrop" })
        end,
    })
    lootMain:AddToggle({
        Title = "Hide opened crates",
        Description = "A crate with Open set is already being emptied.",
        Default = true,
        ConfigKey = "fust.loot.hideopen",
        Callback = function(v) S.loot.hideOpen = v end,
    })
    lootMain:AddToggle({
        Title = "Show possible drops",
        Description = "Names the rarest items the crate CAN drop. Off by default because these are not the crate's contents.",
        Default = false,
        ConfigKey = "fust.loot.possible",
        Callback = function(v) S.loot.previewPossible = v end,
    })
    lootMain:AddSlider({
        Title = "Possible drops listed",
        Description = "Only used when \"Show possible drops\" is on.",
        Min = 0, Max = 8, Step = 1, Default = 2,
        ConfigKey = "fust.loot.preview",
        Callback = function(v)
            S.loot.preview = v
            previewCache, previewFor = {}, nil
        end,
    })
    lootMain:AddSlider({
        Title = "Crates with item lists",
        Description = "Item lines only on the nearest few crates, so the screen stays legible.",
        Min = 0, Max = 12, Step = 1, Default = 3,
        ConfigKey = "fust.loot.itemson",
        Callback = function(v) S.loot.itemsOnNearest = v end,
    })
    lootMain:AddColorPicker({
        Title = "Crate color",
        Default = S.loot.color,
        DefaultAlpha = 1,
        ConfigKey = "fust.loot.color",
        Callback = function(c) S.loot.color = c end,
    })

    local lootNotes = lootTab:AddSection("Contents", "What a crate shows", { Column = "right" })
    lootNotes:AddLabel({
        Title = "Why there are no real contents",
        Description = "A closed crate carries only CrateKind, Id, Kind and Open - the roll happens on the server when the lid opens, and no remote sends it. So the box shows the crate type, its roll count and how many items it draws from.",
    })
    lootNotes:AddLabel({
        Title = "Show possible drops",
        Description = "The optional line names the rarest items in that crate's table. Treat it as \"what this type can give\", not \"what this crate holds\".",
    })
    lootNotes:AddDivider({ Title = "TABLE PER CONTAINER" })
    for _, row in ipairs({
        { "Wooden Crate", "crate_wooden" },
        { "Military Crate", "crate_military" },
        { "Elite Crate", "crate_elite" },
        { "Loot Barrel", "barrel" },
    }) do
        local t = LOOT[row[2]]
        lootNotes:AddLabel({
            Title = row[1],
            Description = t and string.format("%d-%d rolls, %d possible items", t.rolls[1], t.rolls[2], #t.items) or "no table",
        })
    end

    ------------------------------------------------------------------ Visuals
    local visTab = Nexa:AddTab({ Title = "Visuals", Icon = "eye" })

    local visMain = visTab:AddSection("Drawing", "Boxes, text and range", { Column = "left" })
    visMain:AddToggle({
        Title = "Names",
        Default = true,
        ConfigKey = "fust.draw.names",
        Callback = function(v) S.draw.names = v end,
    })
    visMain:AddToggle({
        Title = "Distance",
        Default = true,
        ConfigKey = "fust.draw.dist",
        Callback = function(v) S.draw.dist = v end,
    })
    visMain:AddToggle({
        Title = "Tracers",
        Description = "Line from the bottom of the screen to each box.",
        Default = false,
        ConfigKey = "fust.draw.tracers",
        Callback = function(v) S.draw.tracers = v end,
    })
    visMain:AddSlider({
        Title = "Max distance",
        Min = 50, Max = 1000, Step = 10, Default = 300, Suffix = "m",
        ConfigKey = "fust.draw.maxdist",
        Callback = function(v) S.draw.maxDist = v end,
    })
    visMain:AddSlider({
        Title = "Max labels",
        Description = "Label blocks at once. The nearest targets get them; the rest stay as plain boxes.",
        Min = 1, Max = 20, Step = 1, Default = 8,
        ConfigKey = "fust.draw.labelmax",
        Callback = function(v) S.draw.labelMax = v end,
    })
    visMain:AddSlider({
        Title = "Label range",
        Description = "Past this distance only the box is drawn.",
        Min = 50, Max = 1000, Step = 10, Default = 250, Suffix = "m",
        ConfigKey = "fust.draw.labeldist",
        Callback = function(v) S.draw.labelDist = v end,
    })

    local visPresets = visTab:AddSection("Config", "Saved by ConfigKey", { Column = "right" })
    visPresets:AddButton({
        Title = "Save settings",
        ButtonText = "Save",
        Variant = "primary",
        Callback = function()
            local ok, err = Nexa:SaveConfig("fust_esp")
            Nexa:Notify({ Title = "Fust ESP", Content = ok and "Settings saved." or tostring(err), Type = ok and "success" or "error" })
        end,
    })
    visPresets:AddButton({
        Title = "Load settings",
        ButtonText = "Load",
        Callback = function()
            local ok, err = Nexa:LoadConfig("fust_esp")
            Nexa:Notify({ Title = "Fust ESP", Content = ok and "Settings loaded." or tostring(err), Type = ok and "success" or "error" })
        end,
    })
    visPresets:AddStatus({ Title = "Data source", Default = lootSource, Tone = "success" })

    ------------------------------------------------------------------ Tables
    -- Seven raw item lists at once was unreadable. One table at a time, grouped
    -- into chance bands, rarest band first.
    local TIERS = {
        { title = "Very rare - under 1%",  from = 0, to = 1 },
        { title = "Rare - 1% to 3%",       from = 1, to = 3 },
        { title = "Uncommon - 3% to 8%",   from = 3, to = 8 },
        { title = "Common - 8% and up",    from = 8, to = 1e9 },
    }

    local TABLES = {
        { label = "Wooden Crate",   key = "crate_wooden" },
        { label = "Military Crate", key = "crate_military" },
        { label = "Elite Crate",    key = "crate_elite" },
        { label = "Loot Barrel",    key = "barrel" },
        { label = "Locked Crate",   key = "locked_crate" },
        { label = "Heli Crate",     key = "heli_crate" },
        { label = "Airdrop",        key = "airdrop" },
    }

    local tblTab  = Nexa:AddTab({ Title = "Tables", Icon = "file" })
    local tblMain = tblTab:AddSection("Loot table browser", "Chance per crate, rarest first", { Column = "left" })

    local tierRows = {}
    local summary

    local function tierLines(key)
        local t = LOOT[key]
        local out = {}
        for i = 1, #TIERS do
            local tier = TIERS[i]
            local names, extra = {}, 0
            if t then
                for j = 1, #t.items do
                    local it = t.items[j]
                    local p = it.p or 0
                    if p >= tier.from and p < tier.to then
                        if #names < 5 then
                            names[#names + 1] = pretty(it.id) .. " " .. pct(p)
                        else
                            extra = extra + 1
                        end
                    end
                end
            end
            out[i] = (#names > 0)
                and (table.concat(names, ", ") .. (extra > 0 and string.format("  +%d more", extra) or ""))
                or "-"
        end
        return out
    end

    local function showTable(key)
        local t = LOOT[key]
        if not t then return end
        if summary then
            summary:SetValue(string.format("%d-%d rolls, %d items", t.rolls[1], t.rolls[2], #t.items), "accent")
        end
        local lines = tierLines(key)
        for i = 1, #tierRows do
            tierRows[i]:SetDescription(lines[i])
        end
    end

    tblMain:AddDropdown({
        Title = "Crate",
        Options = (function()
            local o = {}
            for i = 1, #TABLES do o[i] = TABLES[i].label end
            return o
        end)(),
        Default = TABLES[1].label,
        VisibleRows = 7,
        ConfigKey = "fust.tables.pick",
        Callback = function(v)
            for i = 1, #TABLES do
                if TABLES[i].label == v then showTable(TABLES[i].key) return end
            end
        end,
    })

    summary = tblMain:AddStatus({ Title = "Selected", Default = "-", Tone = "accent" })
    for i = 1, #TIERS do
        tierRows[i] = tblMain:AddLabel({ Title = TIERS[i].title, Description = "-" })
    end
    showTable(TABLES[1].key)

    local tblNotes = tblTab:AddSection("How to read it", "What the numbers mean", { Column = "right" })
    tblNotes:AddLabel({
        Title = "Chance per crate",
        Description = "Odds that one crate of that type holds at least one, across every roll the module makes: weighted entries, extra pools and the flat extras. Not a raw weight.",
    })
    tblNotes:AddLabel({
        Title = "Rarest first",
        Description = "Every list - here and on the boxes - is sorted rarest first.",
    })
    tblNotes:AddLabel({
        Title = "Closed crates",
        Description = "Contents are rolled when the lid opens, so the table is the most a client can know beforehand.",
    })
    tblNotes:AddLabel({
        Title = "Event items left out",
        Description = "Metal chest plate, m249, metal facemask, road sign gear and the rest of Items.SaturdayIds only drop while the Saturday event runs, so they are not listed as normal drops.",
    })
    tblNotes:AddStatus({ Title = "Labels on screen", Default = "budgeted, nearest first", Tone = "success" })

    Nexa:Notify({
        Title = "Fust ESP",
        Content = string.format("Loaded: %d loot tables from %s. RightShift for the menu.", tableCount, lootSource),
        Type = "success",
        Duration = 5,
    })
else
    print("[FUST] Nexa UI could not be loaded - the ESP still runs with the defaults above.")
end

--============================================================================
--  PUBLIC API
--============================================================================

local function countKinds()
    local ore, crate = 0, 0
    for i = 1, #targets do
        if targets[i].kind == "crate" then crate = crate + 1 else ore = ore + 1 end
    end
    return ore, crate
end

-- declared first on purpose: the closures below reference it, and inside the
-- table constructor a bare `api` would resolve to a global instead.
local api
api = {
    state   = S,
    targets = function() return targets end,
    stats   = function()
        local ore, crate = countKinds()
        print(string.format("[FUST] targets=%d (ore %d, crates %d) boxes=%d labels=%d pool=%d ui=%s loot=%s",
            #targets, ore, crate, lastDrawn, lastLabels, MAX_TARGETS, tostring(Nexa ~= nil), lootSource))
    end,
    rescan = function()
        targets = collect()
        print("[FUST] rescanned: " .. #targets .. " target(s)")
        return targets
    end,
    unload = function()
        running = false
        if conn then conn:Disconnect() conn = nil end
        for i = 1, #pool do
            local e = pool[i]
            pcall(function() e.box:Remove() end)
            pcall(function() e.name:Remove() end)
            pcall(function() e.sub:Remove() end)
            pcall(function() e.tracer:Remove() end)
        end
        pool = {}
        targets = {}
        if Nexa then pcall(function() Nexa:Hide() end) end
        if rawget(_G, "FUST_ESP") == api then rawset(_G, "FUST_ESP", nil) end
        print("[FUST] unloaded")
    end,
}

_G.FUST_ESP = api

print(string.format("[FUST] ready - %d loot tables from %s - _G.FUST_ESP.unload() to stop", tableCount, lootSource))
