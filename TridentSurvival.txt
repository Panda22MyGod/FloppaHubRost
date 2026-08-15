local repo = "https://raw.githubusercontent.com/deividcomsono/Obsidian/main/"
local Library = loadstring(game:HttpGet(repo .. "Library.lua"))()
local ThemeManager = loadstring(game:HttpGet(repo .. "addons/ThemeManager.lua"))()
local SaveManager = loadstring(game:HttpGet(repo .. "addons/SaveManager.lua"))()

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService        = game:GetService("RunService")
local Players           = game:GetService("Players")
local UIS               = game:GetService("UserInputService")
local Debris            = game:GetService("Debris")
local Lighting          = game:GetService("Lighting")
local SoundService      = game:GetService("SoundService")
local Player            = Players.LocalPlayer
local Options           = Library.Options
local cam               = workspace.CurrentCamera

local Window = Library:CreateWindow({
    Title            = "OpiumWare",
    Footer           = "TridentSurvivalV5",
    NotifySide       = "Right",
    ShowCustomCursor = true
})

local Tabs = {
    COMBAT          = Window:AddTab("COMBAT", "swords"),
    VISUAL          = Window:AddTab("VISUALS", "scan-eye"),
    ["UI Settings"] = Window:AddTab("UI SETTINGS", "settings")
}


local RemoveFog = {
    enabled = false,
}

RunService.RenderStepped:Connect(function()
    if RemoveFog.enabled then
        Lighting.FogEnd = 100000
        Lighting.FogStart = 0
    end
end)



local TimeChanger = {
    enabled = false,
    time = 12,
}

RunService.Heartbeat:Connect(function()
    if TimeChanger.enabled then
        Lighting.ClockTime = TimeChanger.time
    end
end)


local RemoveShadows = {
    enabled = false,
}

local function updateShadows()
    for _, v in pairs(Lighting:GetDescendants()) do
        if v:IsA("ShadowMap") or v:IsA("Atmosphere") then
            pcall(function()
                v.Enabled = not RemoveShadows.enabled
            end)
        end
    end
end

Lighting.DescendantAdded:Connect(updateShadows)
RunService.RenderStepped:Connect(updateShadows)


local Hitmarker = {
    enabled = false,
    color = Color3.fromRGB(255, 255, 255),
    thickness = 2,
    size = 20,
    duration = 0.3,
    active = false,
    timer = 0,
}

local HitSound = {
    enabled = false,
    soundId = "rbxassetid://4764109000",
    volume = 1,
    pitch = 1,
}

local HitmarkerLines = {
    Drawing.new("Line"),
    Drawing.new("Line"),
    Drawing.new("Line"),
    Drawing.new("Line"),
}

for _, line in pairs(HitmarkerLines) do
    line.Visible = false
    line.Color = Hitmarker.color
    line.Thickness = Hitmarker.thickness
    line.ZIndex = 999
end

local function playHitSound()
    if not HitSound.enabled then return end
    local sound = Instance.new("Sound")
    sound.SoundId = HitSound.soundId
    sound.Volume = HitSound.volume
    sound.Pitch = HitSound.pitch
    sound.Parent = SoundService
    sound:Play()
    Debris:AddItem(sound, 0.5)
end

local function showHitmarker()
    Hitmarker.active = true
    Hitmarker.timer = tick()
    local center = Vector2.new(cam.ViewportSize.X / 2, cam.ViewportSize.Y / 2)
    local s = Hitmarker.size
    

    HitmarkerLines[1].From = Vector2.new(center.X - s, center.Y - s)
    HitmarkerLines[1].To = Vector2.new(center.X - s/3, center.Y - s/3)
    HitmarkerLines[1].Visible = true
    

    HitmarkerLines[2].From = Vector2.new(center.X - s, center.Y + s)
    HitmarkerLines[2].To = Vector2.new(center.X - s/3, center.Y + s/3)
    HitmarkerLines[2].Visible = true
    

    HitmarkerLines[3].From = Vector2.new(center.X + s, center.Y - s)
    HitmarkerLines[3].To = Vector2.new(center.X + s/3, center.Y - s/3)
    HitmarkerLines[3].Visible = true
    

    HitmarkerLines[4].From = Vector2.new(center.X + s, center.Y + s)
    HitmarkerLines[4].To = Vector2.new(center.X + s/3, center.Y + s/3)
    HitmarkerLines[4].Visible = true
    
    playHitSound()
end

local function checkSound(obj)
    if not Hitmarker.enabled and not HitSound.enabled then return end
    if obj:IsA("Sound") and obj.Name:lower():find("head") then
        showHitmarker()
    end
end

workspace.DescendantAdded:Connect(checkSound)

local oldPlay = Instance.new("Sound").Play
hookfunction(oldPlay, function(self)
    if Hitmarker.enabled and self.Name:lower():find("head") then
        showHitmarker()
    end
    return oldPlay(self)
end)

RunService.RenderStepped:Connect(function()

    if Hitmarker.active and tick() - Hitmarker.timer > Hitmarker.duration then
        Hitmarker.active = false
        for _, line in pairs(HitmarkerLines) do
            line.Visible = false
        end
    end
    

    for _, line in pairs(HitmarkerLines) do
        line.Color = Hitmarker.color
        line.Thickness = Hitmarker.thickness
    end
end)


local ForceHeadshots = {
    enabled = false,
}

local function SetupForceHeadshots()
    local oldHook
    oldHook = hookfunction(Instance.new("RemoteEvent").FireServer, function(self, ...)
        local args = {...}
        if ForceHeadshots.enabled and self.Name == "RemoteEvent" then
            for i, v in pairs(args) do
                if type(v) == "table" then
                    for key, value in pairs(v) do
                        if type(value) == "string" and value == "Hit" then
                            for k, val in pairs(v) do
                                if type(val) == "string" and val:lower() == "head" then
                                    v[k] = "Head"
                                elseif type(val) == "string" and val:lower() == "torso" then
                                    v[k] = "Head"
                                end
                            end
                        end
                    end
                end
            end
        end
        return oldHook(self, ...)
    end)
end

task.spawn(SetupForceHeadshots)


local PLRESP = Tabs.VISUAL:AddLeftGroupbox("PLAYERS ESP", "eye")

local Esp = {
    settings = {
        enabled = false, boxEnabled = false, nameEnabled = false,
        distanceEnabled = false, weaponEnabled = false, chamsEnabled = false,
        boxType = "Corner", boxOutline = true, boxFill = false,
        fillTransparency = 0.75, renderDistance = 3500,
        boxColor         = Color3.fromRGB(255, 0, 0),
        outlineColor     = Color3.fromRGB(20, 20, 20),
        fillColor        = Color3.fromRGB(255, 0, 0),
        fillColor2       = Color3.fromRGB(20, 20, 20),
        nameColor        = Color3.fromRGB(255, 255, 255),
        nameOutline      = true,
        nameOutlineColor = Color3.fromRGB(20, 20, 20),
        distColor        = Color3.fromRGB(255, 255, 255),
        distOutline      = true,
        distOutlineColor = Color3.fromRGB(20, 20, 20),
        weapColor        = Color3.fromRGB(255, 255, 255),
        weapOutline      = true,
        weapOutlineColor = Color3.fromRGB(20, 20, 20),
        chamsColor       = Color3.fromRGB(255, 255, 255),
        chamsTransparency = 0.5,
        sleepCheck = false, teamCheck = false, aiCheck = false,
    },
    cache = {
        boxes      = setmetatable({}, {__mode = "k"}),
        sleep      = setmetatable({}, {__mode = "k"}),
        player     = setmetatable({}, {__mode = "k"}),
        weapon     = setmetatable({}, {__mode = "k"}),
        weaponTime = setmetatable({}, {__mode = "k"}),
        chams      = setmetatable({}, {__mode = "k"}),
    },
    const = {
        V3_UP = Vector3.new(0, 2.8, 0),
        V3_DN = Vector3.new(0, 3.0, 0),
        ANCHORS = {
            LeftTop         = Vector2.new(0, 0),
            LeftSide        = Vector2.new(0, 0),
            RightTop        = Vector2.new(1, 0),
            RightSide       = Vector2.new(0, 0),
            BottomSide      = Vector2.new(0, 1),
            BottomDown      = Vector2.new(0, 1),
            BottomRightSide = Vector2.new(1, 1),
            BottomRightDown = Vector2.new(1, 1),
        },
    },
}

local gui = Instance.new("ScreenGui")
gui.Name           = "ESPHolder"
gui.ResetOnSpawn   = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.IgnoreGuiInset = true
gui.Parent         = Player:WaitForChild("PlayerGui")

Player.CharacterAdded:Connect(function()
    task.wait(0.5)
    if not gui or not gui.Parent then
        gui = Instance.new("ScreenGui")
        gui.Name           = "ESPHolder"
        gui.ResetOnSpawn   = false
        gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
        gui.IgnoreGuiInset = true
        gui.Parent         = Player:WaitForChild("PlayerGui")
    end
end)

local function ESP_IsTeam(m)
    if not m then return false end
    local h = m:FindFirstChild("Head")
    return h and h:FindFirstChild("Dot") and h.Dot.Enabled == true or false
end
local function ESP_IsSleeper(m)
    if not m then return false end
    local c = Esp.cache.sleep[m]
    if c and tick() - c.time < 1 then return c.value end
    local lt = m:FindFirstChild("LowerTorso"); local v = false
    if lt then
        local rr = lt:FindFirstChild("RootRig")
        if rr then
            local ok, a = pcall(function() return rr.CurrentAngle end)
            v = ok and type(a) == "number" and a ~= 0 or false
        end
    end
    Esp.cache.sleep[m] = {value = v, time = tick()}; return v
end
local function ESP_IsPlayer(m)
    local c = Esp.cache.player[m]
    if c and tick() - c.time < 2 then return c.value end
    local t = m:FindFirstChild("Torso")
    local v = t and t:FindFirstChild("LeftBooster") and true or false
    Esp.cache.player[m] = {value = v, time = tick()}; return v
end
local function ESP_IsAI(m)
    local t = m:FindFirstChild("Torso") or m:FindFirstChild("HumanoidRootPart")
    return t and t.CollisionGroup == "NPC" or false
end
local function shouldSkip(m)
    if not m or not m.Parent then return true end
    local s = Esp.settings
    if s.sleepCheck and ESP_IsSleeper(m) then return true end
    if s.teamCheck  and ESP_IsTeam(m)    then return true end
    if s.aiCheck    and ESP_IsAI(m)      then return true end
    return false
end

local weaponData = {
    Bow={"Arrow","Fabric","Handle","Meshes/Bow","ADS","Mover","AnimationController"},
    Ar15={"AnimSaves","Barrel","Body","Bolt","ChargingHandle","Decor","Grip","Handle","Mag","Rails","Stock","ADS","Muzzle","AnimationController"},
    Blunderbuss={"Body","Handle","Tube","thing","ADS","Muzzle","AnimationController"},
    C9={"Barrel","Body","Bolt","Decor","Grip","Handle","LowerSlide","Mag","Sight1","Sight2","UpperSlide","ADS","Muzzle","AnimationController"},
    CrossBow={"Arrow","BackMetal","Body","FrontNails","Handle","Release","SpringSteel","String","Wheel","ADS","Slide","AnimationController"},
    EnergyRifle={"DefaultSight","FrontCover","Glowing","Grip","Handle","Mag","Metal","Metal2","RearCover","RearDecor","Screws","Tubes","AnimationController"},
    GaussRifle={"DefaultSight","Barrel","Body","CoilHolders","Coils","Decals1","Decals2","Grip","Handle","Housing","Mag","StockBack","AnimationController"},
    Hmar={"DefaultSight","Body","Bolt","Bolts","Cover","Handle","Mag","Rails","Spring","Stock","Wood","Muzzle","AnimationController"},
    LeverActionRifle={"9mm","DefaultSight","Body","Brass","Hammer","Handle","Lever","Metal","Thing","Wood","Muzzle","AnimationController"},
    M4a1={"DefaultSight","Body","Bolt","ChargeHandle","Grip","Handle","Mag","Metal","mbrk","Muzzle","AnimationController"},
    PipePistol={"DefaultSight","Body","Bolt","Handle","Mag","Muzzle","AnimationController"},
    PipeSmg={"DefaultSight","Barrel","Body","Bolt","Flap","Grip","Handle","Mag","Stock","Muzzle","AnimationController"},
    PumpShotgun={"Barrel","Body","Handle","MainMetal","RearSight","Shell","Slider","ADS","Muzzle","AnimationController"},
    RPG={"RocketModel","Body","Body2","Caps","Fasteners","FireMech","Handle","Safety","Sight","Trigger","ADS","Muzzle","AnimationController"},
    Scar={"DefaultSight","Barrel","Body","ChargingHandle","Decals","Handle","Mag","Rails","ShoulderPad","Stock","Muzzle","AnimationController"},
    Svd={"DefaultSight","Body","Bolt","Handle","Magazine","Magazine2","Metal2","Wood","AnimationController"},
    Usp9={"Body","Handle","Mag","Slide","ADS","Muzzle","AnimationController"},
    Uzi={"DefaultSight","Body","Body2","Bolt","ChargingHandle","Decor","Grip","Handle","Mag","Stock","Muzzle","AnimationController"},
    Magnum={"Cylinder","Decor","EjectRod","EjectRodDecal","Frame","Grip"},
}

local function detectWeapon(m)
    local t  = tick()
    local lu = Esp.cache.weaponTime[m]
    if lu and t - lu < 2 then return Esp.cache.weapon[m] or "None" end
    local hand = m:FindFirstChild("HandModel")
    if not hand then
        Esp.cache.weapon[m] = "None"; Esp.cache.weaponTime[m] = t; return "None"
    end
    local best, bestN = "None", 0
    for wn, parts in next, weaponData do
        local cnt = 0
        for _, p in ipairs(parts) do if hand:FindFirstChild(p, true) then cnt = cnt + 1 end end
        if cnt > bestN then best = wn; bestN = cnt end
    end
    Esp.cache.weapon[m] = best; Esp.cache.weaponTime[m] = t
    return best
end

local function updateChams(m, enabled)
    if not Esp.settings.chamsEnabled or not enabled then
        local c = Esp.cache.chams[m]
        if c and c.Enabled then c.Enabled = false end
        return
    end
    local c = Esp.cache.chams[m]
    if not c or not c.Parent then
        if c then pcall(function() c:Destroy() end) end
        local hl = Instance.new("Highlight")
        hl.FillTransparency    = Esp.settings.chamsTransparency
        hl.OutlineTransparency = 1
        hl.DepthMode           = Enum.HighlightDepthMode.AlwaysOnTop
        hl.FillColor           = Esp.settings.chamsColor
        hl.Parent              = m
        Esp.cache.chams[m]     = hl
    else
        if c.FillColor ~= Esp.settings.chamsColor then
            c.FillColor = Esp.settings.chamsColor
        end
        if c.FillTransparency ~= Esp.settings.chamsTransparency then
            c.FillTransparency = Esp.settings.chamsTransparency
        end
        if not c.Enabled then c.Enabled = true end
    end
end

local function newText()
    local t = Drawing.new("Text")
    t.Visible = false; t.Size = 13; t.Center = true
    t.Font = 2; t.Outline = true; t.OutlineColor = Color3.fromRGB(0,0,0)
    return t
end

local function newCornerFrame(anchor)
    local f = Instance.new("Frame")
    f.BackgroundColor3       = Esp.settings.boxColor
    f.BorderSizePixel        = 0
    f.BackgroundTransparency = 0
    f.Visible                = false
    f.AnchorPoint            = anchor
    f.ZIndex                 = 2
    f.Parent                 = gui
    local s = Instance.new("UIStroke")
    s.Color = Color3.fromRGB(0,0,0); s.Thickness = 1; s.Transparency = 0
    s.LineJoinMode = Enum.LineJoinMode.Miter
    s.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    s.Parent = f
    return f, s
end

local function makeCorners()
    local cf = {}
    for name, anchor in next, Esp.const.ANCHORS do
        local f, s = newCornerFrame(anchor)
        cf[name] = {f = f, s = s}
    end
    return cf
end

local function makeFillFrame()
    local f = Instance.new("Frame")
    f.BorderSizePixel = 0; f.BackgroundColor3 = Color3.fromRGB(255,255,255)
    f.BackgroundTransparency = 1; f.Visible = false; f.ZIndex = 0; f.Parent = gui
    local g = Instance.new("UIGradient")
    g.Rotation = 90
    g.Color = ColorSequence.new{
        ColorSequenceKeypoint.new(0, Esp.settings.fillColor),
        ColorSequenceKeypoint.new(1, Esp.settings.fillColor2),
    }
    g.Parent = f
    return f, g
end

local function makeDefaultBox()
    local fill, fillGrad = makeFillFrame()
    fill.ZIndex = 0
    local strokeOutline = Instance.new("UIStroke")
    strokeOutline.Color = Esp.settings.outlineColor; strokeOutline.Thickness = 3
    strokeOutline.Transparency = 0; strokeOutline.LineJoinMode = Enum.LineJoinMode.Miter
    strokeOutline.ApplyStrokeMode = Enum.ApplyStrokeMode.Border; strokeOutline.Parent = fill
    local main = Instance.new("Frame")
    main.BorderSizePixel = 0; main.BackgroundColor3 = Color3.fromRGB(0,0,0)
    main.BackgroundTransparency = 1; main.Visible = false; main.ZIndex = 2; main.Parent = gui
    local stroke = Instance.new("UIStroke")
    stroke.Color = Esp.settings.boxColor; stroke.Thickness = 1; stroke.Transparency = 0
    stroke.LineJoinMode = Enum.LineJoinMode.Miter
    stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border; stroke.Parent = main
    return {fill=fill, fillGrad=fillGrad, main=main, stroke=stroke, strokeOutline=strokeOutline}
end

local function applyFill(fill, grad, l, t, w, h)
    grad.Color = ColorSequence.new{
        ColorSequenceKeypoint.new(0, Esp.settings.fillColor),
        ColorSequenceKeypoint.new(1, Esp.settings.fillColor2),
    }
    fill.BackgroundTransparency = Esp.settings.boxFill and Esp.settings.fillTransparency or 1
    fill.Position = UDim2.new(0, l, 0, t); fill.Size = UDim2.new(0, w, 0, h)
    fill.Visible = true
end

local CORNER_POS = {}
local function buildCornerPos(l, r, t, b, cx, cy)
    CORNER_POS.LeftTop         = {l,   t,  cx,  1.5}
    CORNER_POS.LeftSide        = {l,   t,  1.5, cy}
    CORNER_POS.RightTop        = {r,   t,  cx,  1.5}
    CORNER_POS.RightSide       = {r-1, t,  1.5, cy}
    CORNER_POS.BottomSide      = {l,   b,  1.5, cy}
    CORNER_POS.BottomDown      = {l,   b,  cx,  1.5}
    CORNER_POS.BottomRightSide = {r,   b,  1.5, cy}
    CORNER_POS.BottomRightDown = {r,   b,  cx,  1.5}
end

local function updateCorners(cf, fill, grad, px, py, w, h)
    local cx, cy = w*0.22, h*0.22
    local l, r   = px - w*0.5, px + w*0.5
    local t, b   = py - h*0.5, py + h*0.5
    buildCornerPos(l, r, t, b, cx, cy)
    local bc  = Esp.settings.boxColor
    local oc  = Esp.settings.outlineColor
    local otr = Esp.settings.boxOutline and 0 or 1
    for name, seg in next, cf do
        local d = CORNER_POS[name]
        seg.f.Position = UDim2.new(0, d[1], 0, d[2]); seg.f.Size = UDim2.new(0, d[3], 0, d[4])
        seg.f.BackgroundColor3 = bc; seg.f.Visible = true
        seg.s.Color = oc; seg.s.Transparency = otr
    end
    applyFill(fill, grad, l, t, w, h)
end

local function updateDefaultBox(db, px, py, w, h)
    local l, t = px - w*0.5, py - h*0.5
    applyFill(db.fill, db.fillGrad, l, t, w, h)
    db.strokeOutline.Color = Esp.settings.outlineColor
    db.strokeOutline.Transparency = Esp.settings.boxOutline and 0 or 1
    db.main.Position = UDim2.new(0, l, 0, t); db.main.Size = UDim2.new(0, w, 0, h)
    db.main.Visible = true; db.stroke.Color = Esp.settings.boxColor; db.stroke.Transparency = 0
end

local function hideCorners(cf, fill)
    for _, seg in next, cf do
        if seg.f.Visible then seg.f.Visible = false end
    end
    if fill.Visible then fill.Visible = false end
end
local function hideDefault(db)
    if db.fill.Visible then db.fill.Visible = false end
    if db.main.Visible then db.main.Visible = false end
end

local function destroyEntry(m, d)
    pcall(function() d.cacheConn:Disconnect() end)
    for _, seg in next, d.corners do pcall(function() seg.f:Destroy() end) end
    pcall(function() d.cFill:Destroy() end)
    pcall(function() d.default.fill:Destroy() end)
    pcall(function() d.default.main:Destroy() end)
    pcall(function() d.nameText:Remove() end)
    pcall(function() d.distText:Remove() end)
    pcall(function() d.weapText:Remove() end)
    Esp.cache.boxes[m] = nil
end

local function createEntry(m)
    if Esp.cache.boxes[m] then return end
    local cFill, cGrad = makeFillFrame()
    local entry = {
        corners   = makeCorners(),
        cFill     = cFill,
        cGrad     = cGrad,
        default   = makeDefaultBox(),
        nameText  = newText(),
        distText  = newText(),
        weapText  = newText(),
        hrp       = m:FindFirstChild("HumanoidRootPart"),
        lastCache = 0,
        isPlayer  = false,
        isSleeper = false,
    }
    entry.cacheConn = m.ChildAdded:Connect(function()
        entry.hrp = m:FindFirstChild("HumanoidRootPart")
    end)
    Esp.cache.boxes[m] = entry
end

local ESP_INTERVAL = 1 / 90
local espLastTick  = 0

RunService.Heartbeat:Connect(function()
    local now = tick()
    if now - espLastTick < ESP_INTERVAL then return end
    espLastTick = now

    local camPos = cam.CFrame.Position
    local s      = Esp.settings

    for m, d in next, Esp.cache.boxes do
        local function hide()
            hideCorners(d.corners, d.cFill)
            hideDefault(d.default)
            if d.nameText.Visible then d.nameText.Visible = false end
            if d.distText.Visible then d.distText.Visible = false end
            if d.weapText.Visible then d.weapText.Visible = false end
        end

        if not m or not m.Parent then
            hide(); destroyEntry(m, d); continue
        end
        if not s.enabled then hide(); updateChams(m, false); continue end

        local hrp = d.hrp
        if not hrp or not hrp.Parent then
            d.hrp = m:FindFirstChild("HumanoidRootPart")
            hide(); continue
        end

        local hrpPos = hrp.Position
        local dx = hrpPos.X - camPos.X
        local dy = hrpPos.Y - camPos.Y
        local dz = hrpPos.Z - camPos.Z
        local distSq = dx*dx + dy*dy + dz*dz
        local rd = s.renderDistance
        if distSq > rd*rd then
            hide(); updateChams(m, false); continue
        end

        if shouldSkip(m) then hide(); updateChams(m, false); continue end

        if now - d.lastCache > 5 then
            d.isPlayer  = ESP_IsPlayer(m)
            d.isSleeper = ESP_IsSleeper(m)
            d.lastCache = now
        end

        local topPos, topVis = cam:WorldToViewportPoint(hrpPos + Esp.const.V3_UP)
        local botPos         = cam:WorldToViewportPoint(hrpPos - Esp.const.V3_DN)

        if not topVis then
            hide(); updateChams(m, true); continue
        end

        local h = botPos.Y - topPos.Y
        if h < 5 then hide(); continue end

        local w  = h * 0.65
        local px = topPos.X
        local py = topPos.Y + h * 0.5
        local dist = math.floor(math.sqrt(distSq))

        if s.boxEnabled then
            if s.boxType == "Corner" then
                hideDefault(d.default)
                updateCorners(d.corners, d.cFill, d.cGrad, px, py, w, h)
            else
                hideCorners(d.corners, d.cFill)
                updateDefaultBox(d.default, px, py, w, h)
            end
        else
            hideCorners(d.corners, d.cFill)
            hideDefault(d.default)
        end

        local dtype = d.isSleeper and "Sleeper" or (d.isPlayer and "Player" or "Bot")

        if s.nameEnabled then
            d.nameText.Text         = dtype
            d.nameText.Position     = Vector2.new(px, py - h*0.5 - 16)
            d.nameText.Color        = s.nameColor
            d.nameText.Outline      = s.nameOutline
            d.nameText.OutlineColor = s.nameOutlineColor
            d.nameText.Visible      = true
        else
            if d.nameText.Visible then d.nameText.Visible = false end
        end

        if s.distanceEnabled then
            d.distText.Text         = "[" .. dist .. "m]"
            d.distText.Position     = Vector2.new(px, py + h*0.5 + 4)
            d.distText.Color        = s.distColor
            d.distText.Outline      = s.distOutline
            d.distText.OutlineColor = s.distOutlineColor
            d.distText.Visible      = true
        else
            if d.distText.Visible then d.distText.Visible = false end
        end

        if s.weaponEnabled then
            d.weapText.Text         = detectWeapon(m)
            d.weapText.Position     = Vector2.new(px, py + h*0.5 + (s.distanceEnabled and 18 or 4))
            d.weapText.Color        = s.weapColor
            d.weapText.Outline      = s.weapOutline
            d.weapText.OutlineColor = s.weapOutlineColor
            d.weapText.Visible      = true
        else
            if d.weapText.Visible then d.weapText.Visible = false end
        end

        updateChams(m, true)
    end
end)

workspace.DescendantAdded:Connect(function(obj)
    if obj:IsA("Model") then
        task.wait(0.1)
        if obj:FindFirstChild("HumanoidRootPart") and not Esp.cache.boxes[obj] then
            createEntry(obj)
        end
    elseif obj.Name == "HandModel" and obj.Parent then
        Esp.cache.weapon[obj.Parent]     = nil
        Esp.cache.weaponTime[obj.Parent] = nil
    end
end)

workspace.DescendantRemoving:Connect(function(obj)
    if obj:IsA("Model") then
        local d = Esp.cache.boxes[obj]
        if d then destroyEntry(obj, d) end
        local ch = Esp.cache.chams[obj]
        if ch then
            pcall(function() ch:Destroy() end)
            Esp.cache.chams[obj] = nil
        end
        Esp.cache.weapon[obj]     = nil
        Esp.cache.weaponTime[obj] = nil
        Esp.cache.player[obj]     = nil
        Esp.cache.sleep[obj]      = nil
    elseif obj.Name == "HandModel" and obj.Parent then
        Esp.cache.weapon[obj.Parent]     = nil
        Esp.cache.weaponTime[obj.Parent] = nil
    end
end)

task.spawn(function()
    while true do
        task.wait(15)
        for m in next, Esp.cache.weapon do
            if not m or not m.Parent then
                Esp.cache.weapon[m] = nil
                Esp.cache.weaponTime[m] = nil
            end
        end
        for m in next, Esp.cache.sleep do
            if not m or not m.Parent then Esp.cache.sleep[m] = nil end
        end
        for m in next, Esp.cache.player do
            if not m or not m.Parent then Esp.cache.player[m] = nil end
        end
        for m, d in next, Esp.cache.boxes do
            if not m or not m.Parent then destroyEntry(m, d) end
        end
        for m, c in next, Esp.cache.chams do
            if not m or not m.Parent then
                pcall(function() c:Destroy() end)
                Esp.cache.chams[m] = nil
            end
        end
    end
end)

for _, m in next, workspace:GetChildren() do
    if m:IsA("Model") then
        task.spawn(function()
            task.wait(0.1)
            if m:FindFirstChild("HumanoidRootPart") then createEntry(m) end
        end)
    end
end

PLRESP:AddToggle("EnableEsp", {
    Text = "Enable ESP", Default = false,
    Callback = function(v) Esp.settings.enabled = v end
})
PLRESP:AddToggle("SleepCheck", {
    Text = "Sleep Check", Default = false,
    Callback = function(v) Esp.settings.sleepCheck = v end
})
PLRESP:AddToggle("AICheck", {
    Text = "AI Check", Default = false,
    Callback = function(v) Esp.settings.aiCheck = v end
})
PLRESP:AddToggle("TeamCheck", {
    Text = "Team Check", Default = false,
    Callback = function(v) Esp.settings.teamCheck = v end
})
PLRESP:AddSlider("RenderDistance", {
    Text = "Render Distance", Default = 3500, Min = 500, Max = 3500, Rounding = 0, Suffix = "st",
    Callback = function(v) Esp.settings.renderDistance = v end
})
local boxToggle = PLRESP:AddToggle("EnableBox", {
    Text = "Enable Box", Default = false,
    Callback = function(v) Esp.settings.boxEnabled = v end
})
boxToggle:AddColorPicker("BoxColor", {
    Title = "Box Color", Default = Color3.fromRGB(255, 0, 0),
    Callback = function(v) Esp.settings.boxColor = v end
})
PLRESP:AddDropdown("BoxType", {
    Values = {"Corner","Default"}, Default = 1, Multi = false, Text = "Box Type",
    Callback = function(v) Esp.settings.boxType = v end
})
local outlineToggle = PLRESP:AddToggle("BoxOutline", {
    Text = "Box Outline", Default = true,
    Callback = function(v) Esp.settings.boxOutline = v end
})
outlineToggle:AddColorPicker("OutlineColor", {
    Title = "Outline Color", Default = Color3.fromRGB(20, 20, 20),
    Callback = function(v) Esp.settings.outlineColor = v end
})
local fillToggle = PLRESP:AddToggle("BoxFill", {
    Text = "Box Fill", Default = false,
    Callback = function(v) Esp.settings.boxFill = v end
})
fillToggle:AddColorPicker("FillColor1", {
    Title = "Fill Color 1", Default = Color3.fromRGB(255, 0, 0),
    Callback = function(v) Esp.settings.fillColor = v end
})
fillToggle:AddColorPicker("FillColor2", {
    Title = "Fill Color 2", Default = Color3.fromRGB(20, 20, 20),
    Callback = function(v) Esp.settings.fillColor2 = v end
})
PLRESP:AddSlider("FillTransparency", {
    Text = "Fill Transparency", Default = 0.75, Min = 0, Max = 1, Rounding = 2, Suffix = "%",
    Callback = function(v) Esp.settings.fillTransparency = v end
})
local nameToggle = PLRESP:AddToggle("EnableName", {
    Text = "Name", Default = false,
    Callback = function(v) Esp.settings.nameEnabled = v end
})
nameToggle:AddColorPicker("NameColor", {
    Title = "Name Color", Default = Color3.fromRGB(255, 255, 255),
    Callback = function(v) Esp.settings.nameColor = v end
})
local nameOutlineToggle = PLRESP:AddToggle("NameOutline", {
    Text = "Name Outline", Default = true,
    Callback = function(v) Esp.settings.nameOutline = v end
})
nameOutlineToggle:AddColorPicker("NameOutlineColor", {
    Title = "Name Outline Color", Default = Color3.fromRGB(20, 20, 20),
    Callback = function(v) Esp.settings.nameOutlineColor = v end
})
local distToggle = PLRESP:AddToggle("EnableDist", {
    Text = "Distance", Default = false,
    Callback = function(v) Esp.settings.distanceEnabled = v end
})
distToggle:AddColorPicker("DistColor", {
    Title = "Distance Color", Default = Color3.fromRGB(255, 255, 255),
    Callback = function(v) Esp.settings.distColor = v end
})
local distOutlineToggle = PLRESP:AddToggle("DistOutline", {
    Text = "Distance Outline", Default = true,
    Callback = function(v) Esp.settings.distOutline = v end
})
distOutlineToggle:AddColorPicker("DistOutlineColor", {
    Title = "Distance Outline Color", Default = Color3.fromRGB(20, 20, 20),
    Callback = function(v) Esp.settings.distOutlineColor = v end
})
local weapToggle = PLRESP:AddToggle("EnableWeapon", {
    Text = "Weapon", Default = false,
    Callback = function(v) Esp.settings.weaponEnabled = v end
})
weapToggle:AddColorPicker("WeapColor", {
    Title = "Weapon Color", Default = Color3.fromRGB(255, 255, 255),
    Callback = function(v) Esp.settings.weapColor = v end
})
local weapOutlineToggle = PLRESP:AddToggle("WeapOutline", {
    Text = "Weapon Outline", Default = true,
    Callback = function(v) Esp.settings.weapOutline = v end
})
weapOutlineToggle:AddColorPicker("WeapOutlineColor", {
    Title = "Weapon Outline Color", Default = Color3.fromRGB(20, 20, 20),
    Callback = function(v) Esp.settings.weapOutlineColor = v end
})
local chamsToggle = PLRESP:AddToggle("EnableChams", {
    Text = "Chams", Default = false,
    Callback = function(v)
        Esp.settings.chamsEnabled = v
        if not v then
            for _, c in next, Esp.cache.chams do
                if c.Enabled then c.Enabled = false end
            end
        end
    end
})
chamsToggle:AddColorPicker("ChamsColor", {
    Title = "Chams Color", Default = Color3.fromRGB(255, 255, 255),
    Callback = function(v)
        Esp.settings.chamsColor = v
        for _, c in next, Esp.cache.chams do
            if c.Enabled then c.FillColor = v end
        end
    end
})
PLRESP:AddSlider("ChamsTransparency", {
    Text = "Chams Transparency", Default = 0.5, Min = 0, Max = 1, Rounding = 2, Suffix = "%",
    Callback = function(v)
        Esp.settings.chamsTransparency = v
        for _, c in next, Esp.cache.chams do
            if c.Enabled then c.FillTransparency = v end
        end
    end
})


local visGroup = Tabs.VISUAL:AddLeftGroupbox("WORLD VISUALS", "eye")
visGroup:AddToggle("RemoveFog", {
    Text = "Remove Fog", Default = false,
    Callback = function(v) RemoveFog.enabled = v end
})

local timeToggle = visGroup:AddToggle("TimeChanger", {
    Text = "Time Changer", Default = false,
    Callback = function(v) TimeChanger.enabled = v end
})
visGroup:AddSlider("TimeValue", {
    Text = "Time", Default = 12, Min = 0, Max = 24, Rounding = 1,
    Callback = function(v) TimeChanger.time = v end
})


visGroup:AddToggle("RemoveShadows", {
    Text = "Remove Shadows", Default = false,
    Callback = function(v) RemoveShadows.enabled = v; updateShadows() end
})


local hmGroup = Tabs.VISUAL:AddLeftGroupbox("HITMARKER", "crosshair")
local hmToggle = hmGroup:AddToggle("Hitmarker", {
    Text = "Enable Hitmarker", Default = false,
    Callback = function(v) Hitmarker.enabled = v end
})
hmToggle:AddColorPicker("HitmarkerColor", {
    Title = "Hitmarker Color", Default = Color3.fromRGB(255, 255, 255),
    Callback = function(v) Hitmarker.color = v end
})
hmGroup:AddSlider("HitmarkerSize", {
    Text = "Size", Default = 20, Min = 5, Max = 50, Rounding = 1,
    Callback = function(v) Hitmarker.size = v end
})
hmGroup:AddSlider("HitmarkerThickness", {
    Text = "Thickness", Default = 2, Min = 1, Max = 5, Rounding = 1,
    Callback = function(v) Hitmarker.thickness = v end
})
hmGroup:AddSlider("HitmarkerDuration", {
    Text = "Duration (s)", Default = 0.3, Min = 0.1, Max = 1, Rounding = 2,
    Callback = function(v) Hitmarker.duration = v end
})



Tabs.COMBAT:AddLeftGroupbox("FORCE HEADSHOTS", "target"):AddToggle("ForceHeadshots", {
    Text = "Force Headshots", Default = false,
    Callback = function(v) ForceHeadshots.enabled = v end
})


local Hitbox = {
    UI = Tabs.COMBAT:AddLeftGroupbox("HITBOX EXPANDER", "box"),
    Settings = {
        enabled = false, sizeX = 4, sizeY = 4, sizeZ = 4,
        transparency = 0.50, color = Color3.fromRGB(255, 255, 255),
        material = "Neon",
        sleepCheck = false, aiCheck = false, teamCheck = false,
        target = "Head",
    },
    Original = {},
    OverrideParts = {},
}

local function HB_IsTeam(model)
    if not model then return false end
    local h = model:FindFirstChild("Head")
    return h and h:FindFirstChild("Dot") and h.Dot.Enabled == true or false
end
local function HB_IsSleeper(model)
    if not model then return false end
    local lt = model:FindFirstChild("LowerTorso")
    if lt then
        local rr = lt:FindFirstChild("RootRig")
        if rr and typeof(rr.CurrentAngle) == "number" and rr.CurrentAngle ~= 0 then return true end
    end
    return false
end
local function HB_IsNPC(model)
    local torso = model:FindFirstChild("Torso")
    if torso then
        if torso.CollisionGroup == "NPC" then return true end
        if torso:FindFirstChild("LeftBooster") then return false end
        return true
    end
    local hrp = model:FindFirstChild("HumanoidRootPart")
    return hrp and hrp.CollisionGroup == "NPC" or false
end

Hitbox.ShouldSkip = function(model)
    if not model or not model.Parent then return true end
    local s = Hitbox.Settings
    if s.aiCheck    and HB_IsNPC(model)     then return true end
    if s.sleepCheck and HB_IsSleeper(model) then return true end
    if s.teamCheck  and HB_IsTeam(model)    then return true end
    return false
end

Hitbox.Save = function(part)
    if not Hitbox.Original[part] then
        Hitbox.Original[part] = {
            Size = part.Size, Transparency = part.Transparency,
            Color = part.Color, Material = part.Material, CanCollide = part.CanCollide
        }
    end
end
Hitbox.Restore = function(part)
    if not part or not part.Parent then return end
    local o = Hitbox.Original[part]
    if not o then return end
    part.Size = o.Size; part.Transparency = o.Transparency
    part.Color = o.Color; part.Material = o.Material; part.CanCollide = o.CanCollide
end
Hitbox.ApplyToPart = function(part)
    if not part or not part:IsA("BasePart") then return end
    Hitbox.Save(part)
    local s = Hitbox.Settings
    part.Size        = Vector3.new(s.sizeX, s.sizeY, s.sizeZ)
    part.Transparency = s.transparency
    part.Color       = s.color
    part.Material    = Enum.Material[s.material]
    part.CanCollide  = false
end

Hitbox.RestoreAll = function()
    for part, o in pairs(Hitbox.Original) do
        if part and part.Parent then
            part.Size = o.Size; part.Transparency = o.Transparency
            part.Color = o.Color; part.Material = o.Material; part.CanCollide = o.CanCollide
        end
    end
    Hitbox.Original = {}
end

Hitbox.IsCandidate = function(m)
    return m:IsA("Model") and m.Name ~= Player.Name
        and (m:FindFirstChild("Head") or m:FindFirstChild("HumanoidRootPart"))
end

Hitbox.ApplyToModel = function(model)
    if not model or not model.Parent then return end
    local s = Hitbox.Settings
    local head  = model:FindFirstChild("Head")
    local torso = model:FindFirstChild("Torso") or model:FindFirstChild("HumanoidRootPart")

    if not head or not torso then return end

    if Hitbox.ShouldSkip(model) or not s.enabled then
        Hitbox.Restore(head)
        Hitbox.Restore(torso)
        return
    end

    if s.target == "Head" then
        Hitbox.ApplyToPart(head)
        Hitbox.Restore(torso)
    elseif s.target == "Torso" then
        Hitbox.ApplyToPart(torso)
        Hitbox.Restore(head)
    end
end

Hitbox.UpdateAll = function()
    for _, model in pairs(workspace:GetChildren()) do
        if Hitbox.IsCandidate(model) then
            Hitbox.ApplyToModel(model)
        end
    end
end

Hitbox.Refresh = function()
    if not Hitbox.Settings.enabled then return end
    for _, model in pairs(workspace:GetChildren()) do
        if Hitbox.IsCandidate(model) then
            local head  = model:FindFirstChild("Head")
            local torso = model:FindFirstChild("Torso") or model:FindFirstChild("HumanoidRootPart")
            if head and torso then
                local part = Hitbox.Settings.target == "Head" and head or torso
                Hitbox.ApplyToPart(part)
            end
        end
    end
end


local OverrideHitbox = {
    enabled = false,
    sizeX = 3,
    sizeY = 5,
    sizeZ = 3,
    transparency = 0.5,
    color = Color3.fromRGB(148, 0, 211),
    material = "Neon",
    partName = "Head",
    parts = {},
}

local function CreateOverrideHitbox(model)
    if not model or not model.Parent then return end
    if model:FindFirstChild("Fake") then return end
    if not model:FindFirstChild("HumanoidRootPart") then return end
    
    local fakePart = Instance.new("Part")
    fakePart.Name = OverrideHitbox.partName
    fakePart.Size = Vector3.new(OverrideHitbox.sizeX, OverrideHitbox.sizeY, OverrideHitbox.sizeZ)
    fakePart.CFrame = model.HumanoidRootPart.CFrame
    fakePart.Anchored = true
    fakePart.CanCollide = false
    fakePart.Transparency = OverrideHitbox.transparency
    fakePart.Color = OverrideHitbox.color
    fakePart.Material = Enum.Material[OverrideHitbox.material]
    fakePart.Parent = model
    
    local fakeContainer = Instance.new("Part")
    fakeContainer.Name = "Fake"
    fakeContainer.Anchored = true
    fakeContainer.CanCollide = false
    fakeContainer.Transparency = 1
    fakeContainer.Size = Vector3.new(0.1, 0.1, 0.1)
    fakeContainer.Parent = model
    
    table.insert(OverrideHitbox.parts, fakePart)
    table.insert(OverrideHitbox.parts, fakeContainer)
end

local function RemoveOverrideHitbox(model)
    local fake = model:FindFirstChild("Fake")
    if fake then fake:Destroy() end
    local head = model:FindFirstChild(OverrideHitbox.partName)
    if head and head ~= model:FindFirstChild("Head") then
        head:Destroy()
    end
    for i = #OverrideHitbox.parts, 1, -1 do
        if not OverrideHitbox.parts[i].Parent then
            table.remove(OverrideHitbox.parts, i)
        end
    end
end

local function UpdateOverrideHitboxes()
    for _, model in pairs(workspace:GetChildren()) do
        if model:IsA("Model") and model ~= Player.Character and model:FindFirstChild("HumanoidRootPart") then
            if OverrideHitbox.enabled then
                CreateOverrideHitbox(model)
            else
                RemoveOverrideHitbox(model)
            end
        end
    end
end

task.spawn(function()
    while true do
        task.wait(0.5)
        if OverrideHitbox.enabled then
            for _, model in pairs(workspace:GetChildren()) do
                if model:IsA("Model") and model ~= Player.Character and model:FindFirstChild("HumanoidRootPart") then
                    local fake = model:FindFirstChild("Fake")
                    if not fake then
                        CreateOverrideHitbox(model)
                    end
                end
            end
        end
    end
end)

workspace.ChildAdded:Connect(function(child)
    if OverrideHitbox.enabled and child:IsA("Model") and child ~= Player.Character and child:FindFirstChild("HumanoidRootPart") then
        task.wait(0.1)
        CreateOverrideHitbox(child)
    end
end)

workspace.ChildRemoved:Connect(function(child)
    if child:IsA("Model") then
        RemoveOverrideHitbox(child)
    end
end)


local enableToggle = Hitbox.UI:AddToggle("EnableHitbox", {
    Text = "Enable Hitbox", Default = false,
    Callback = function(state)
        Hitbox.Settings.enabled = state
        if not state then
            Hitbox.RestoreAll()
        else
            Hitbox.UpdateAll()
        end
    end
})
enableToggle:AddColorPicker("HitboxColor", {
    Default = Color3.fromRGB(255, 255, 255), Title = "Hitbox Color",
    Callback = function(color) Hitbox.Settings.color = color; Hitbox.Refresh() end
})
Hitbox.UI:AddDropdown("HitboxTarget", {
    Text = "Target", Values = {"Head", "Torso"}, Default = "Head", Multi = false,
    Callback = function(value)
        Hitbox.Settings.target = value
        if Hitbox.Settings.enabled then Hitbox.UpdateAll() end
    end
})
Hitbox.UI:AddDropdown("HitboxMaterial", {
    Text = "Material", Values = {"Neon", "ForceField", "Plastic", "SmoothPlastic"}, Default = "Neon",
    Callback = function(value)
        Hitbox.Settings.material = value
        if Hitbox.Settings.enabled then Hitbox.Refresh() end
    end
})
Hitbox.UI:AddToggle("SleepCheck", {
    Text = "Sleep Check", Default = false,
    Callback = function(state)
        Hitbox.Settings.sleepCheck = state
        if Hitbox.Settings.enabled then Hitbox.UpdateAll() end
    end
})
Hitbox.UI:AddToggle("AICheck", {
    Text = "AI Check", Default = false,
    Callback = function(state)
        Hitbox.Settings.aiCheck = state
        if Hitbox.Settings.enabled then Hitbox.UpdateAll() end
    end
})
Hitbox.UI:AddToggle("TeamCheck", {
    Text = "Team Check", Default = false,
    Callback = function(state)
        Hitbox.Settings.teamCheck = state
        if Hitbox.Settings.enabled then Hitbox.UpdateAll() end
    end
})
Hitbox.UI:AddSlider("HitboxSizeX", {
    Text = "Size X", Min = 1, Max = 25, Default = 4, Rounding = 1, Suffix = "st",
    Callback = function(value) Hitbox.Settings.sizeX = value; Hitbox.Refresh() end
})
Hitbox.UI:AddSlider("HitboxSizeY", {
    Text = "Size Y", Min = 1, Max = 25, Default = 4, Rounding = 1, Suffix = "st",
    Callback = function(value) Hitbox.Settings.sizeY = value; Hitbox.Refresh() end
})
Hitbox.UI:AddSlider("HitboxSizeZ", {
    Text = "Size Z", Min = 1, Max = 25, Default = 4, Rounding = 1, Suffix = "st",
    Callback = function(value) Hitbox.Settings.sizeZ = value; Hitbox.Refresh() end
})
Hitbox.UI:AddSlider("HitboxTransparency", {
    Text = "Transparency", Min = 0, Max = 1, Default = 0.5, Rounding = 2, Suffix = "%",
    Callback = function(value) Hitbox.Settings.transparency = value; Hitbox.Refresh() end
})

Hitbox.UI:AddDivider()

local overrideToggle = Hitbox.UI:AddToggle("OverrideHitbox", {
    Text = "Override Hitbox", Default = false,
    Callback = function(state)
        OverrideHitbox.enabled = state
        if state then
            UpdateOverrideHitboxes()
        else
            for _, model in pairs(workspace:GetChildren()) do
                if model:IsA("Model") then
                    RemoveOverrideHitbox(model)
                end
            end
        end
    end
})
overrideToggle:AddColorPicker("OverrideColor", {
    Title = "Override Color", Default = Color3.fromRGB(148, 0, 211),
    Callback = function(v)
        OverrideHitbox.color = v
        for _, part in pairs(OverrideHitbox.parts) do
            if part and part.Parent and part.Name == "Head" then
                part.Color = v
            end
        end
    end
})
Hitbox.UI:AddDropdown("OverrideMaterial", {
    Text = "Override Material", Values = {"Neon", "ForceField", "Plastic", "SmoothPlastic"}, Default = "Neon",
    Callback = function(v)
        OverrideHitbox.material = v
        for _, part in pairs(OverrideHitbox.parts) do
            if part and part.Parent and part.Name == "Head" then
                part.Material = Enum.Material[v]
            end
        end
    end
})
Hitbox.UI:AddSlider("OverrideSizeX", {
    Text = "Override Size X", Min = 1, Max = 25, Default = 3, Rounding = 1,
    Callback = function(v)
        OverrideHitbox.sizeX = v
        for _, part in pairs(OverrideHitbox.parts) do
            if part and part.Parent and part.Name == "Head" then
                part.Size = Vector3.new(v, OverrideHitbox.sizeY, OverrideHitbox.sizeZ)
            end
        end
    end
})
Hitbox.UI:AddSlider("OverrideSizeY", {
    Text = "Override Size Y", Min = 1, Max = 25, Default = 5, Rounding = 1,
    Callback = function(v)
        OverrideHitbox.sizeY = v
        for _, part in pairs(OverrideHitbox.parts) do
            if part and part.Parent and part.Name == "Head" then
                part.Size = Vector3.new(OverrideHitbox.sizeX, v, OverrideHitbox.sizeZ)
            end
        end
    end
})
Hitbox.UI:AddSlider("OverrideSizeZ", {
    Text = "Override Size Z", Min = 1, Max = 25, Default = 3, Rounding = 1,
    Callback = function(v)
        OverrideHitbox.sizeZ = v
        for _, part in pairs(OverrideHitbox.parts) do
            if part and part.Parent and part.Name == "Head" then
                part.Size = Vector3.new(OverrideHitbox.sizeX, OverrideHitbox.sizeY, v)
            end
        end
    end
})
Hitbox.UI:AddSlider("OverrideTransparency", {
    Text = "Override Transparency", Min = 0, Max = 1, Default = 0.5, Rounding = 2,
    Callback = function(v)
        OverrideHitbox.transparency = v
        for _, part in pairs(OverrideHitbox.parts) do
            if part and part.Parent and part.Name == "Head" then
                part.Transparency = v
            end
        end
    end
})


local SilentAim = {
    UI = Tabs.COMBAT:AddRightGroupbox("SILENT AIM", "crosshair"),
    Settings = {
        enabled      = false,
        prediction   = false,
        fovEnabled   = false,
        fovFill      = false,
        fovFillTransparency = 0.3,
        fovSize      = 150,
        fovColor     = Color3.fromRGB(255, 255, 255),
        aimRadius    = 1000,
        showCross    = false,
        showSnapLine = false,
        sleeperCheck = false,
        aiCheck      = false,
        teamCheck    = false,
    },
}

local FOV_Circle = Drawing.new("Circle")
FOV_Circle.Thickness = 1.5
FOV_Circle.NumSides = 100
FOV_Circle.Filled = false
FOV_Circle.Transparency = 1
FOV_Circle.Visible = false
FOV_Circle.ZIndex = 999

local FOV_Fill = Drawing.new("Circle")
FOV_Fill.Thickness = 0
FOV_Fill.NumSides = 100
FOV_Fill.Filled = true
FOV_Fill.Transparency = 0.7
FOV_Fill.Visible = false
FOV_Fill.ZIndex = 998

local PredictionMarker = Drawing.new("Square")
PredictionMarker.Size = Vector2.new(10, 10)
PredictionMarker.Filled = true
PredictionMarker.Thickness = 1
PredictionMarker.Visible = false
PredictionMarker.ZIndex = 1001
PredictionMarker.Color = Color3.fromRGB(255, 255, 255)

local SnapLine = Drawing.new("Line")
SnapLine.Thickness = 1.5
SnapLine.Transparency = 0.5
SnapLine.Visible = false
SnapLine.ZIndex = 1000
SnapLine.Color = Color3.fromRGB(255, 255, 255)

local SnapDot = Drawing.new("Circle")
SnapDot.Radius = 3
SnapDot.Filled = true
SnapDot.Thickness = 1
SnapDot.Visible = false
SnapDot.ZIndex = 1001
SnapDot.Color = Color3.fromRGB(255, 255, 255)

local function updateFovPos()
    local s = cam.ViewportSize
    local center = Vector2.new(s.X / 2, s.Y / 2)
    FOV_Circle.Position = center
    FOV_Fill.Position = center
end
updateFovPos()
cam:GetPropertyChangedSignal("ViewportSize"):Connect(updateFovPos)


local Classes = nil
local Init_Attempt = 0
local Init_Max_Attempts = 100

repeat
    local success, result = pcall(function()
        return getrenv()._G.classes
    end)
    if success and result then
        Classes = result
    end
    task.wait(0.25)
    Init_Attempt = Init_Attempt + 1
until Classes or (Init_Attempt >= Init_Max_Attempts)

if not Classes then
    warn("OpiumWare: OPIUM WORKING")
end

local CameraClient = Classes and Classes.Camera or nil
local FPSClient = Classes and Classes.FPS or nil
local EntityMap = Classes and Classes.Player.EntityMap or nil

local VALID_GUNS = {
    "AR15", "C9", "Crossbow", "Bow", "EnergyRifle", "GaussRifle",
    "HMAR", "KABAR", "LeverActionRifle", "M4A1", "Magnum", "PipePistol",
    "PipeSMG", "PumpShotgun", "SCAR", "SVD", "USP9", "UZI", "Blunderbuss"
}

local function CalculateBallistics(targetPos, targetVel, camPos, projSpeed, projDrop)
    local distance = (targetPos - camPos).Magnitude
    local timeToTarget = distance / projSpeed

    local adjustedHorizontal = Vector3.new(targetVel.X, 0, targetVel.Z) * 7
    local adjustedVertical = Vector3.new(0, targetVel.Y, 0) * 2
    
    local predictedWorldPos = targetPos + ((adjustedHorizontal + adjustedVertical) * timeToTarget)

    local verticalDrop = -projDrop ^ (timeToTarget * projDrop) + 1
    local finalAimPosition = predictedWorldPos - Vector3.new(0, verticalDrop, 0)

    return finalAimPosition, timeToTarget
end

local function GetOptimalTarget()
    if not EntityMap then return nil, nil end
    local bestTarget, bestVelocity, minMagnitude = nil, nil, SilentAim.Settings.fovSize
    local viewportCenter = Vector2.new(cam.ViewportSize.X / 2, cam.ViewportSize.Y / 2)

    for entityIndex, entityData in next, EntityMap do
        local model = entityData.model
        if model and model:FindFirstChild("HumanoidRootPart") and not entityData.sleeping then
            if SilentAim.Settings.sleeperCheck and entityData.sleeping then continue end
            local head = model:FindFirstChild("Head")
            if head then
                local screenPosition, isOnScreen = cam:WorldToViewportPoint(head.Position)
                if isOnScreen then
                    local mouseDistance = (Vector2.new(screenPosition.X, screenPosition.Y) - viewportCenter).Magnitude
                    if mouseDistance < minMagnitude then
                        minMagnitude = mouseDistance
                        bestTarget = model
                        bestVelocity = entityData.velocityVector
                    end
                end
            end
        end
    end
    return bestTarget, bestVelocity
end

local oldCFrameHook = nil
if CameraClient then
    oldCFrameHook = CameraClient.GetCFrame
    CameraClient.GetCFrame = function(...)
        if SilentAim.Settings.enabled then
            local aimTarget, targetVel = GetOptimalTarget()
            local currentItem = FPSClient and FPSClient.GetEquippedItem and FPSClient.GetEquippedItem() or nil
            
            if aimTarget and currentItem and table.find(VALID_GUNS, currentItem.type) then
                local ballisticData = Classes[currentItem.type]
                if ballisticData then
                    local finalAimPos, _ = CalculateBallistics(
                        aimTarget.Head.Position, 
                        targetVel, 
                        cam.CFrame.Position, 
                        ballisticData.ProjectileSpeed, 
                        ballisticData.ProjectileDrop
                    )
                    return CFrame.new(cam.CFrame.Position, finalAimPos)
                end
            end
        end
        return oldCFrameHook(...)
    end
end


RunService.RenderStepped:Connect(function()
    if not SilentAim.Settings.enabled then
        FOV_Circle.Visible = false
        FOV_Fill.Visible = false
        PredictionMarker.Visible = false
        SnapLine.Visible = false
        SnapDot.Visible = false
        return
    end
    
    FOV_Circle.Visible = SilentAim.Settings.fovEnabled
    FOV_Fill.Visible = SilentAim.Settings.fovEnabled and SilentAim.Settings.fovFill
    FOV_Circle.Radius = SilentAim.Settings.fovSize
    FOV_Fill.Radius = SilentAim.Settings.fovSize
    FOV_Circle.Color = SilentAim.Settings.fovColor
    FOV_Fill.Color = SilentAim.Settings.fovColor
    FOV_Fill.Transparency = 1 - SilentAim.Settings.fovFillTransparency
    
    local center = Vector2.new(cam.ViewportSize.X / 2, cam.ViewportSize.Y / 2)
    FOV_Circle.Position = center
    FOV_Fill.Position = center

    if SilentAim.Settings.prediction then
        local aimTarget, targetVel = GetOptimalTarget()
        if aimTarget and FPSClient then
            local currentGun = FPSClient.GetEquippedItem()
            if currentGun and Classes[currentGun.type] then
                local stats = Classes[currentGun.type]
                local finalAimPos, _ = CalculateBallistics(
                    aimTarget.Head.Position, 
                    targetVel, 
                    cam.CFrame.Position, 
                    stats.ProjectileSpeed, 
                    stats.ProjectileDrop
                )
                local screenAimPos, isOnScreen = cam:WorldToViewportPoint(finalAimPos)
                if isOnScreen then
                    local pos = Vector2.new(screenAimPos.X, screenAimPos.Y)
                    
                    if SilentAim.Settings.showCross then
                        PredictionMarker.Position = Vector2.new(pos.X - 5, pos.Y - 5)
                        PredictionMarker.Visible = true
                    else
                        PredictionMarker.Visible = false
                    end
                    
                    if SilentAim.Settings.showSnapLine then
                        SnapLine.From = center
                        SnapLine.To = pos
                        SnapLine.Visible = true
                        
                        SnapDot.Position = pos
                        SnapDot.Visible = true
                    else
                        SnapLine.Visible = false
                        SnapDot.Visible = false
                    end
                else 
                    PredictionMarker.Visible = false
                    SnapLine.Visible = false
                    SnapDot.Visible = false
                end
            else
                PredictionMarker.Visible = false
                SnapLine.Visible = false
                SnapDot.Visible = false
            end
        else
            PredictionMarker.Visible = false
            SnapLine.Visible = false
            SnapDot.Visible = false
        end
    else
        PredictionMarker.Visible = false
        SnapLine.Visible = false
        SnapDot.Visible = false
    end
end)

SilentAim.UI:AddToggle("Enable", {
    Text = "Enable Silent Aim", Default = false,
    Callback = function(v) SilentAim.Settings.enabled = v end
})
SilentAim.UI:AddToggle("Prediction", {
    Text = "Enable Prediction", Default = false,
    Callback = function(v) SilentAim.Settings.prediction = v end
})
SilentAim.UI:AddToggle("SleeperCheck", {
    Text = "Sleeper Check", Default = false,
    Callback = function(v) SilentAim.Settings.sleeperCheck = v end
})
SilentAim.UI:AddToggle("AICheck", {
    Text = "AI Check", Default = false,
    Callback = function(v) SilentAim.Settings.aiCheck = v end
})
SilentAim.UI:AddToggle("TeamCheck", {
    Text = "Team Check", Default = false,
    Callback = function(v) SilentAim.Settings.teamCheck = v end
})
SilentAim.UI:AddToggle("ShowCross", {
    Text = "Show Prediction Cross ✚", Default = false,
    Callback = function(v) SilentAim.Settings.showCross = v end
})
SilentAim.UI:AddToggle("ShowSnapLine", {
    Text = "Show Snap Line + Dot", Default = false,
    Callback = function(v) SilentAim.Settings.showSnapLine = v end
})
local fovToggle = SilentAim.UI:AddToggle("FovEnable", {
    Text = "Show FOV Circle", Default = false,
    Callback = function(v)
        SilentAim.Settings.fovEnabled = v
        FOV_Circle.Visible = v and SilentAim.Settings.enabled
        FOV_Fill.Visible = v and SilentAim.Settings.fovFill and SilentAim.Settings.enabled
    end
})
fovToggle:AddColorPicker("FovColor", {
    Title = "FOV Color", Default = Color3.fromRGB(255, 255, 255),
    Callback = function(v) 
        SilentAim.Settings.fovColor = v
        FOV_Circle.Color = v
        FOV_Fill.Color = v
    end
})
local fillToggle = SilentAim.UI:AddToggle("FovFill", {
    Text = "FOV Fill", Default = false,
    Callback = function(v)
        SilentAim.Settings.fovFill = v
        FOV_Fill.Visible = v and SilentAim.Settings.fovEnabled and SilentAim.Settings.enabled
    end
})
SilentAim.UI:AddSlider("FovFillTransparency", {
    Text = "Fill Transparency", Default = 0.3, Min = 0, Max = 1, Rounding = 2, Suffix = "%",
    Callback = function(v) 
        SilentAim.Settings.fovFillTransparency = v
        FOV_Fill.Transparency = 1 - v
    end
})
SilentAim.UI:AddSlider("FovSize", {
    Text = "FOV Size", Default = 150, Min = 50, Max = 500, Rounding = 0, Suffix = "px",
    Callback = function(v) 
        SilentAim.Settings.fovSize = v
        FOV_Circle.Radius = v
        FOV_Fill.Radius = v
    end
})



local Character = { Jumpshoot = false, JsPart = nil }

Character.Js = function()
    local p = Instance.new("Part", workspace)
    p.Name = "!.!"; p.Size = Vector3.new(4, 0.2, 4)
    p.Anchored = true; p.Color = Color3.fromRGB(255, 255, 255)
    p.Transparency = 1; p.Material = Enum.Material.Neon
    local mesh = Instance.new("SpecialMesh")
    mesh.MeshType = Enum.MeshType.FileMesh
    mesh.MeshId   = "rbxassetid://20329976"
    mesh.Parent   = p
    return p
end

Character.Update = function()
    while Character.Jumpshoot do
        local ok, mid = pcall(function()
            return workspace.Const.Ignore.LocalCharacter.Middle.Position
        end)
        if ok and Character.JsPart and Character.JsPart.Parent then
            Character.JsPart.Position = mid - Vector3.new(0, 3.5, 0)
        end
        RunService.Heartbeat:Wait()
    end
end

local OTH1 = Tabs.COMBAT:AddRightGroupbox("OTHER", "user-cog")

local jsToggle = OTH1:AddToggle("js", {
    Text = "Jump Shoot", Default = false,
    Callback = function(state)
        Character.Jumpshoot = state
        if state then
            if not Character.JsPart or not Character.JsPart.Parent then
                Character.JsPart = Character.Js()
            end
            task.spawn(function() Character.Update() end)
        else
            if Character.JsPart then Character.JsPart:Destroy(); Character.JsPart = nil end
        end
    end
})
jsToggle:AddKeyPicker("JsKey", {
    Default = "None", Text = "JumpShoot", Mode = "Toggle",
    Callback = function(state) jsToggle:SetValue(state) end
})


local LN = { enabled = false, strength = 5 }

local function getNeck()
    local char = workspace.Const and workspace.Const.Ignore and workspace.Const.Ignore.LocalCharacter
    if not char then return nil end
    local top = char:FindFirstChild("Top")
    return top and top:FindFirstChild("Prism1")
end

local neck = getNeck()
local origNeckCF = neck and neck.CFrame

local function applyNeck(state)
    if not neck then neck = getNeck(); origNeckCF = neck and neck.CFrame end
    if not neck or not origNeckCF then return end
    neck.CFrame = state
        and CFrame.new(origNeckCF.Position - Vector3.new(0, LN.strength, 0)) * (origNeckCF - origNeckCF.Position)
        or origNeckCF
end

local lnToggle = OTH1:AddToggle("LongNeckToggle", {
    Text = "LongNeck", Default = false,
    Callback = function(v) LN.enabled = v; applyNeck(v) end
})
lnToggle:AddKeyPicker("LongNeckKey", {
    Text = "LongNeck", Default = "None", Mode = "Toggle",
    Callback = function(v) LN.enabled = v; applyNeck(v); lnToggle:SetValue(v) end
})


local BT = { enabled = false, color = Color3.fromRGB(255, 255, 255), thickness = 0.1, lifetime = 1.5 }
local bulletNames = {"Bullet","BlueBullet","Projectile","Rocket","Missile","Grenade"}

local function isLocalArrow(obj)
    local ok, res = pcall(function()
        local fps = workspace:FindFirstChild("Const")
            and workspace.Const:FindFirstChild("Ignore")
            and workspace.Const.Ignore:FindFirstChild("FPSArms")
        if fps then
            local hm = fps:FindFirstChild("HandModel")
            return hm and hm:FindFirstChild("Arrow") == obj
        end
        return false
    end)
    return ok and res
end

local function createTracer(proj)
    if proj.Name == "Arrow" and isLocalArrow(proj) then return end
    local lastPos = proj.Position or proj.CFrame.Position
    local conn
    conn = RunService.RenderStepped:Connect(function()
        if not BT.enabled or not proj or not proj.Parent then conn:Disconnect(); return end
        if proj.Name == "Arrow" and isLocalArrow(proj) then conn:Disconnect(); return end
        local currentPos = proj.Position or proj.CFrame.Position
        local distance   = (currentPos - lastPos).Magnitude
        if distance > 0.5 and distance < 500 then
            local tracer = Instance.new("Part")
            tracer.Anchored = true; tracer.CanCollide = false
            tracer.CanQuery = false; tracer.CanTouch = false
            tracer.Size     = Vector3.new(BT.thickness, BT.thickness, distance)
            tracer.CFrame   = CFrame.new(lastPos, currentPos) * CFrame.new(0, 0, -distance/2)
            tracer.Color    = BT.color
            tracer.Material     = Enum.Material.Neon
            tracer.Transparency = 0
            tracer.Parent       = workspace
            Debris:AddItem(tracer, BT.lifetime)
        end
        lastPos = currentPos
    end)
end

workspace.DescendantAdded:Connect(function(obj)
    if obj:IsDescendantOf(ReplicatedStorage) then return end
    task.wait()
    if not BT.enabled then return end
    for _, name in ipairs(bulletNames) do
        if obj.Name == name then createTracer(obj); return end
    end
    if obj.Name == "Arrow" and not isLocalArrow(obj) then createTracer(obj) end
end)

local tracerToggle = Tabs.VISUAL:AddRightGroupbox("OTHER ESP"):AddToggle("BulletTracerToggle", {
    Text = "Bullet Tracers", Default = false,
    Callback = function(v) BT.enabled = v end
})
tracerToggle:AddColorPicker("BulletTracerColor", {
    Title = "Tracer Color", Default = Color3.fromRGB(255, 255, 255),
    Callback = function(v) BT.color = v end
})


local Visuals = {
    Chams = {
        WeaponChams = false, WeaponChams_Color = Color3.fromRGB(255, 255, 255), WeaponChams_Material = "ForceField",
        ArmChams    = false, ArmChams_Color    = Color3.fromRGB(255, 255, 255), ArmChams_Material    = "ForceField",
        Section     = Tabs.VISUAL:AddRightGroupbox("ARMS VISUALS", "package-open"),
    },
    Cache = { WeaponMaterial = {}, WeaponColor = {}, ArmMaterial = {}, ArmColor = {} },
}

local Viewmodel = ReplicatedStorage:FindFirstChild("HandModels")

if Viewmodel then
    for _, Child in pairs(Viewmodel:GetChildren()) do
        if Child:IsA("Model") then
            for _, Part in pairs(Child:GetDescendants()) do
                if Part:IsA("BasePart") then
                    Visuals.Cache.WeaponMaterial[Part] = Part.Material
                    Visuals.Cache.WeaponColor[Part]    = Part.Color
                end
            end
        end
    end
end

local armPartNames = {
    {"Const","Ignore","FPSArms","RightHand"},
    {"Const","Ignore","FPSArms","RightLowerArm"},
    {"Const","Ignore","FPSArms","LeftLowerArm"},
    {"Const","Ignore","FPSArms","LeftHand"},
    {"Const","Ignore","FPSArms","Fake","c_RightLowerArm"},
    {"Const","Ignore","FPSArms","Fake","c_LeftLowerArm"},
}

local function findPath(root, paths)
    local cur = root
    for _, p in ipairs(paths) do
        cur = cur:FindFirstChild(p)
        if not cur then return nil end
    end
    return cur
end

local function AC_RecacheArms()
    Visuals.Cache.ArmMaterial = {}; Visuals.Cache.ArmColor = {}
    for _, paths in ipairs(armPartNames) do
        local p = findPath(workspace, paths)
        if p and p:IsA("BasePart") then
            Visuals.Cache.ArmMaterial[p] = p.Material
            Visuals.Cache.ArmColor[p]    = p.Color
        end
    end
end

local function AC_GetArmParts()
    local parts = {}
    for _, paths in ipairs(armPartNames) do
        local p = findPath(workspace, paths)
        if p and p:IsA("BasePart") then table.insert(parts, p) end
    end
    return parts
end

AC_RecacheArms()

local fpsArms = findPath(workspace, {"Const","Ignore","FPSArms"})
if fpsArms then
    fpsArms.ChildAdded:Connect(function()
        task.wait(0.2); AC_RecacheArms()
        if Visuals.Chams.ArmChams then Visuals.Chams.Update() end
    end)
    fpsArms.ChildRemoved:Connect(function() task.wait(0.1); AC_RecacheArms() end)
end

Visuals.Chams.Update = function()
    if Viewmodel then
        for _, Child in pairs(Viewmodel:GetChildren()) do
            if Child:IsA("Model") then
                for _, Part in pairs(Child:GetDescendants()) do
                    if Part:IsA("BasePart") then
                        if Visuals.Chams.WeaponChams then
                            Part.Material = Enum.Material[Visuals.Chams.WeaponChams_Material]
                            Part.Color    = Visuals.Chams.WeaponChams_Color
                        else
                            local origMat = Visuals.Cache.WeaponMaterial[Part]
                            local origCol = Visuals.Cache.WeaponColor[Part]
                            if origMat then Part.Material = origMat end
                            if origCol then Part.Color    = origCol end
                        end
                    end
                end
            end
        end
    end
    local armParts = AC_GetArmParts()
    for _, part in ipairs(armParts) do
        if part and part.Parent then
            if Visuals.Chams.ArmChams then
                part.Material = Enum.Material[Visuals.Chams.ArmChams_Material]
                part.Color    = Visuals.Chams.ArmChams_Color
            else
                local origMat = Visuals.Cache.ArmMaterial[part]
                local origCol = Visuals.Cache.ArmColor[part]
                if origMat then part.Material = origMat end
                if origCol then part.Color    = origCol end
            end
        end
    end
end

RunService.Heartbeat:Connect(function()
    if not Visuals.Chams.ArmChams then return end
    local armParts = AC_GetArmParts()
    for _, part in ipairs(armParts) do
        if part and part.Parent then
            part.Material = Enum.Material[Visuals.Chams.ArmChams_Material]
            part.Color    = Visuals.Chams.ArmChams_Color
        end
    end
end)

local WeaponChams = Visuals.Chams.Section:AddToggle("WeaponChams", {
    Text = "Weapon Chams", Default = false,
    Callback = function(value) Visuals.Chams.WeaponChams = value; Visuals.Chams.Update() end
})
WeaponChams:AddColorPicker("WeaponChamsColor", {
    Default = Visuals.Chams.WeaponChams_Color, Title = "Weapon Color",
    Callback = function(value) Visuals.Chams.WeaponChams_Color = value; if Visuals.Chams.WeaponChams then Visuals.Chams.Update() end end
})
Visuals.Chams.Section:AddDropdown("WeaponMaterial", {
    Text = "Weapon Material", Default = "ForceField", Values = {"Neon","ForceField"},
    Callback = function(value) Visuals.Chams.WeaponChams_Material = value; if Visuals.Chams.WeaponChams then Visuals.Chams.Update() end end
})
local ArmChams = Visuals.Chams.Section:AddToggle("ArmChams", {
    Text = "Arm Chams", Default = false,
    Callback = function(value) Visuals.Chams.ArmChams = value; Visuals.Chams.Update() end
})
ArmChams:AddColorPicker("ArmChamsColor", {
    Default = Visuals.Chams.ArmChams_Color, Title = "Arm Color",
    Callback = function(value) Visuals.Chams.ArmChams_Color = value; if Visuals.Chams.ArmChams then Visuals.Chams.Update() end end
})
Visuals.Chams.Section:AddDropdown("ArmMaterial", {
    Text = "Arm Material", Default = "ForceField", Values = {"Neon","ForceField"},
    Callback = function(value) Visuals.Chams.ArmChams_Material = value; if Visuals.Chams.ArmChams then Visuals.Chams.Update() end end
})


local sm11coil = true
local sm11spread = true

oldhook = hookfunction(setmetatable, newcclosure(function(tbl, mt)
    if typeof(mt) == "table" and rawget(mt, "__index") then
        local index = mt.__index
        if sm11coil and type(index) == "table" and not checkcaller() then
            if rawget(index, "HipRecoil") then
                index.HipRecoil.cameraX, index.HipRecoil.cameraY, index.HipRecoil.push = 0, 0, 0
            end
            if rawget(index, "AimRecoil") then
                index.AimRecoil.cameraX, index.AimRecoil.cameraY, index.AimRecoil.push = 0, 0, 0
            end
        end
        if sm11spread and type(index) == "table" and not checkcaller() then
            index.Accuracy = -9e9
        end
    end
    return oldhook(tbl, mt)
end))

local recoilToggle = OTH1:AddToggle("NoRecoil", {
    Text = "No Recoil", Default = true,
    Callback = function(v) sm11coil = v end
})

local spreadToggle = OTH1:AddToggle("NoSpread", {
    Text = "No Spread", Default = true,
    Callback = function(v) sm11spread = v end
})


local UIGroup = Tabs["UI Settings"]:AddLeftGroupbox("UI Settings")

UIGroup:AddToggle("KeybindMenuOpen", {
    Default = Library.KeybindFrame.Visible, Text = "Show Keybind Menu",
    Callback = function(v) Library.KeybindFrame.Visible = v end
})
--source code yea

UIGroup:AddToggle("ShowCustomCursor", {
    Text = "Custom Cursor", Default = true,
    Callback = function(v) Library.ShowCustomCursor = v end
})
UIGroup:AddDropdown("NotificationSide", {
    Values = {"Left","Right"}, Default = "Right", Text = "Notification Side",
    Callback = function(v) Library:SetNotifySide(v) end
})
UIGroup:AddDropdown("DPIDropdown", {
    Values = {"50%","75%","100%","125%","150%","175%","200%"}, Default = "100%", Text = "DPI Scale",
    Callback = function(v) local dpi = tonumber(v:gsub("%%","")); Library:SetDPIScale(dpi) end
})
UIGroup:AddDivider()

local menuKeyLabel = UIGroup:AddLabel("Menu Keybind")
menuKeyLabel:AddKeyPicker("MenuKeybind", {
    Default = "RightShift", NoUI = false, Text = "Menu Keybind",
    Callback = function() end
})
UIGroup:AddButton("Unload", function() Library:Unload() end)

ThemeManager:SetLibrary(Library)
ThemeManager:SetFolder("OpiumWare")
ThemeManager:ApplyToTab(Tabs["UI Settings"])

SaveManager:SetLibrary(Library)
SaveManager:IgnoreThemeSettings()
SaveManager:SetIgnoreIndexes({"MenuKeybind"})
SaveManager:SetFolder("OpiumWare")
SaveManager:BuildConfigSection(Tabs["UI Settings"])

ThemeManager.Library:SetFont(Enum.Font.Jura)
ThemeManager.Library:UpdateColorsUsingRegistry()

SaveManager:LoadAutoloadConfig()

if Options and Options.MenuKeybind then
    Library.ToggleKeybind = Options.MenuKeybind
end
