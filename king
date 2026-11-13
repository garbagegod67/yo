--[[
    Spectator Tool — Full Overhaul
    LocalScript → StarterPlayerScripts

    Press [H] to show / hide the window (key configurable below).
]]

local Players          = game:GetService("Players")
local RunService       = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local lighting         = game:GetService("Lighting")
local TweenService     = game:GetService("TweenService")

local localPlayer = Players.LocalPlayer
local playerGui   = localPlayer:WaitForChild("PlayerGui")
local camera      = workspace.CurrentCamera

-- ─────────────────────────────────────────────
--  Config
-- ─────────────────────────────────────────────
local TOGGLE_KEY  = Enum.KeyCode.H   -- press H to show/hide the UI
local RESET_KEY   = Enum.KeyCode.R   -- press R to disable all toggles (natural view)
local KILL_KEY    = Enum.KeyCode.M   -- press M × 3 quickly to fully destroy the script

local cfg = {
    highlightsEnabled   = true,
    nameTagsEnabled     = true,
    ambientEnabled      = true,
    fillTransparency    = 0.5,
    outlineTransparency = 0.0,
    tagTextTransparency = 0.0,
    walkSpeed           = 16,
    jumpPower           = 50,
}

local entityFill    = Color3.fromRGB(255, 60,  60)
local entityOutline = Color3.fromRGB(255, 60,  60)
local playerFill    = Color3.fromRGB(80,  140, 255)
local playerOutline = Color3.fromRGB(80,  140, 255)
local corpseFill    = Color3.fromRGB(80,  255, 120)
local corpseOutline = Color3.fromRGB(80,  255, 120)

local originalAmbient    = lighting.Ambient
local originalBrightness = lighting.Brightness
local white              = Color3.fromRGB(255, 255, 255)

local trackedHighlights = {}
local trackedBillboards = {}
local watchedHumanoids  = {}
local playerNames       = {}

for _, p in ipairs(Players:GetPlayers()) do
    playerNames[p.Name] = true
end

-- ─────────────────────────────────────────────
--  Highlight / name-tag helpers
-- ─────────────────────────────────────────────
local function removeVisuals(character)
    for _, child in ipairs(character:GetChildren()) do
        if child:IsA("Highlight") then
            trackedHighlights[child] = nil
            child:Destroy()
        elseif child.Name == "SpectatorNameTag" then
            trackedBillboards[child] = nil
            child:Destroy()
        end
    end
end

local function addHighlight(character, fillColor, outlineColor)
    local existing = {}
    for _, child in ipairs(character:GetChildren()) do
        if child:IsA("Highlight") then table.insert(existing, child) end
    end

    local hl = existing[1]
    if not hl then
        hl = Instance.new("Highlight")
        hl.Parent = character
    end
    for i = 2, #existing do existing[i]:Destroy() end

    hl.Adornee             = character
    hl.FillColor           = fillColor
    hl.OutlineColor        = outlineColor
    hl.FillTransparency    = cfg.fillTransparency
    hl.OutlineTransparency = cfg.outlineTransparency
    hl.Enabled             = cfg.highlightsEnabled

    trackedHighlights[hl] = true
end

local function addNameTag(character, nameColor)
    local bb = character:FindFirstChild("SpectatorNameTag")
    if not bb then
        bb = Instance.new("BillboardGui")
        bb.Name        = "SpectatorNameTag"
        bb.Size        = UDim2.new(0, 180, 0, 36)
        bb.StudsOffset = Vector3.new(0, 3.2, 0)
        bb.AlwaysOnTop = true
        bb.Parent      = character

        local lbl = Instance.new("TextLabel")
        lbl.Name                   = "NameLabel"
        lbl.Size                   = UDim2.new(1, 0, 1, 0)
        lbl.BackgroundTransparency = 1
        lbl.Font                   = Enum.Font.GothamBold
        lbl.TextSize               = 18
        lbl.TextStrokeTransparency = 0.35
        lbl.Parent                 = bb
    end

    local lbl = bb:FindFirstChild("NameLabel")
    if lbl then
        lbl.TextColor3           = nameColor
        lbl.TextTransparency     = cfg.tagTextTransparency
        lbl.TextStrokeTransparency = math.min(1, cfg.tagTextTransparency + 0.35)
        lbl.Text                 = character.Name
    end

    bb.Adornee = character:FindFirstChild("Head")
        or character:FindFirstChildWhichIsA("BasePart", true)
    bb.Enabled = cfg.nameTagsEnabled

    trackedBillboards[bb] = true
end

local function applyAppearance(character, fillColor, outlineColor)
    addHighlight(character, fillColor, outlineColor)
    addNameTag(character, fillColor)
end

-- ─────────────────────────────────────────────
--  Character classification — workspace-only,
--  no folder assumptions
-- ─────────────────────────────────────────────
local function updateCharacter(humanoid)
    if not humanoid:IsA("Humanoid") then return end
    local character = humanoid.Parent
    if not character or not character:IsDescendantOf(workspace) then return end

    if playerNames[character.Name] then
        -- it's a player character
        if humanoid.Health <= 0 then
            applyAppearance(character, corpseFill, corpseOutline)
        else
            applyAppearance(character, playerFill, playerOutline)
        end
    else
        -- entity / NPC
        applyAppearance(character, entityFill, entityOutline)
    end
end

local function setupHumanoid(humanoid)
    if not humanoid:IsA("Humanoid") then return end
    updateCharacter(humanoid)
    if watchedHumanoids[humanoid] then return end
    watchedHumanoids[humanoid] = true

    humanoid.HealthChanged:Connect(function()
        updateCharacter(humanoid)
    end)
    humanoid.AncestryChanged:Connect(function()
        if humanoid.Parent and humanoid:IsDescendantOf(workspace) then
            updateCharacter(humanoid)
        else
            -- left workspace — strip visuals if parent still exists
            local character = humanoid.Parent
            if character then removeVisuals(character) end
        end
    end)
end

local function refreshAll()
    for _, d in ipairs(workspace:GetDescendants()) do
        if d:IsA("Humanoid") then updateCharacter(d) end
    end
end

-- Boot scan
for _, d in ipairs(workspace:GetDescendants()) do
    if d:IsA("Humanoid") then setupHumanoid(d) end
end

workspace.DescendantAdded:Connect(function(d)
    if d:IsA("Humanoid") then setupHumanoid(d) end
end)

Players.PlayerAdded:Connect(function(p)
    playerNames[p.Name] = true
    refreshAll()
end)
Players.PlayerRemoving:Connect(function(p)
    playerNames[p.Name] = nil
    refreshAll()
end)

-- ─────────────────────────────────────────────
--  Lighting
-- ─────────────────────────────────────────────
lighting.Ambient    = white
lighting.Brightness = 1

lighting:GetPropertyChangedSignal("Ambient"):Connect(function()
    if cfg.ambientEnabled and lighting.Ambient ~= white then
        lighting.Ambient = white
    end
end)
lighting:GetPropertyChangedSignal("Brightness"):Connect(function()
    if cfg.ambientEnabled and lighting.Brightness ~= 1 then
        lighting.Brightness = 1
    end
end)

-- ─────────────────────────────────────────────
--  Toggle / slider callbacks
-- ─────────────────────────────────────────────
local function setHighlights(state)
    cfg.highlightsEnabled = state
    for hl in pairs(trackedHighlights) do
        if hl.Parent then hl.Enabled = state
        else trackedHighlights[hl] = nil end
    end
end

local function setNameTags(state)
    cfg.nameTagsEnabled = state
    for bb in pairs(trackedBillboards) do
        if bb.Parent then bb.Enabled = state
        else trackedBillboards[bb] = nil end
    end
end

local function setAmbient(state)
    cfg.ambientEnabled = state
    if state then
        lighting.Ambient    = white
        lighting.Brightness = 1
    else
        lighting.Ambient    = originalAmbient
        lighting.Brightness = originalBrightness
    end
end

-- ─────────────────────────────────────────────
--  Movement
-- ─────────────────────────────────────────────
local function applyMovement()
    local char = localPlayer.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if hum then
        hum.WalkSpeed = cfg.walkSpeed
        hum.JumpPower = cfg.jumpPower
    end
end

localPlayer.CharacterAdded:Connect(function(char)
    char:WaitForChild("Humanoid")
    applyMovement()
end)
applyMovement()

-- ─────────────────────────────────────────────
--  Camera spectate — scans workspace for any
--  player character regardless of folder
-- ─────────────────────────────────────────────
local spectating     = false
local spectateIndex  = 1
local spectateTarget = nil
local spectateList   = {}

local function getSpectateList()
    local list = {}
    for _, d in ipairs(workspace:GetDescendants()) do
        if d:IsA("Humanoid") and playerNames[d.Parent and d.Parent.Name or ""] then
            local char = d.Parent
            if char and char.Name ~= localPlayer.Name and d.Health > 0
                and char:FindFirstChild("HumanoidRootPart") then
                table.insert(list, char)
            end
        end
    end
    return list
end

local function stopSpectate()
    spectating     = false
    spectateTarget = nil
    camera.CameraType = Enum.CameraType.Custom
    local char = localPlayer.Character
    if char then
        camera.CameraSubject = char:FindFirstChildOfClass("Humanoid")
    end
end

local function startSpectate(character)
    spectateTarget    = character
    spectating        = true
    camera.CameraType = Enum.CameraType.Custom
    local hum = character:FindFirstChildOfClass("Humanoid")
    if hum then camera.CameraSubject = hum end
end

local function spectateNext()
    spectateList = getSpectateList()
    if #spectateList == 0 then stopSpectate(); return end
    spectateIndex = spectateIndex % #spectateList + 1
    startSpectate(spectateList[spectateIndex])
end

local function spectatePrev()
    spectateList = getSpectateList()
    if #spectateList == 0 then stopSpectate(); return end
    spectateIndex = ((spectateIndex - 2) % #spectateList) + 1
    startSpectate(spectateList[spectateIndex])
end

local function toggleSpectate()
    if spectating then
        stopSpectate()
    else
        spectateList = getSpectateList()
        if #spectateList == 0 then return end
        spectateIndex = 1
        startSpectate(spectateList[1])
    end
end

-- ═══════════════════════════════════════════════
--  GUI
-- ═══════════════════════════════════════════════
local ACCENT   = Color3.fromRGB(100, 180, 255)
local BG_DARK  = Color3.fromRGB(12,  14,  20)
local BG_MID   = Color3.fromRGB(20,  24,  34)
local BG_PANEL = Color3.fromRGB(26,  30,  44)
local BG_ROW   = Color3.fromRGB(32,  38,  54)
local TEXT_HI  = Color3.fromRGB(230, 235, 255)
local TEXT_LO  = Color3.fromRGB(130, 140, 170)
local GREEN    = Color3.fromRGB(60,  200, 110)
local RED      = Color3.fromRGB(220, 70,  70)
local TI       = TweenInfo.new(0.15, Enum.EasingStyle.Quad)
local TI_HIDE  = TweenInfo.new(0.2,  Enum.EasingStyle.Quad)

local gui = Instance.new("ScreenGui")
gui.Name           = "SpectatorToolsGui"
gui.ResetOnSpawn   = false
gui.DisplayOrder   = 10
gui.IgnoreGuiInset = true
gui.Parent         = playerGui

-- ── Window frame ──
local win = Instance.new("Frame")
win.Name             = "Window"
win.Size             = UDim2.new(0, 300, 0, 560)
win.Position         = UDim2.new(0, 24, 0, 24)
win.BackgroundColor3 = BG_DARK
win.BorderSizePixel  = 0
win.ClipsDescendants = true
win.Parent           = gui

local winCorner = Instance.new("UICorner")
winCorner.CornerRadius = UDim.new(0, 12)
winCorner.Parent = win

-- Accent stripe
local stripe = Instance.new("Frame")
stripe.Size             = UDim2.new(1, 0, 0, 3)
stripe.BackgroundColor3 = ACCENT
stripe.BorderSizePixel  = 0
stripe.Parent           = win

-- Drop shadow
local shadow = Instance.new("ImageLabel")
shadow.Size              = UDim2.new(1, 30, 1, 30)
shadow.Position          = UDim2.new(0, -15, 0, -15)
shadow.BackgroundTransparency = 1
shadow.Image             = "rbxassetid://1316045217"
shadow.ImageColor3       = Color3.new(0, 0, 0)
shadow.ImageTransparency = 0.55
shadow.ScaleType         = Enum.ScaleType.Slice
shadow.SliceCenter       = Rect.new(10, 10, 118, 118)
shadow.ZIndex            = 0
shadow.Parent            = win

-- ── Title bar ──
local titleBar = Instance.new("Frame")
titleBar.Size            = UDim2.new(1, 0, 0, 44)
titleBar.Position        = UDim2.new(0, 0, 0, 3)
titleBar.BackgroundColor3= BG_MID
titleBar.BorderSizePixel = 0
titleBar.Parent          = win

local titleLabel = Instance.new("TextLabel")
titleLabel.Size              = UDim2.new(1, -80, 1, 0)
titleLabel.Position          = UDim2.new(0, 16, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text              = "🎮  SPECTATOR TOOLS"
titleLabel.TextColor3        = TEXT_HI
titleLabel.Font              = Enum.Font.GothamBold
titleLabel.TextSize          = 15
titleLabel.TextXAlignment    = Enum.TextXAlignment.Left
titleLabel.Parent            = titleBar

-- Key hint label (shows "[H]")
local keyHint = Instance.new("TextLabel")
keyHint.Size             = UDim2.new(0, 54, 0, 20)
keyHint.Position         = UDim2.new(1, -98, 0.5, -10)
keyHint.BackgroundColor3 = BG_PANEL
keyHint.BorderSizePixel  = 0
keyHint.Text             = "[H] [R]"
keyHint.TextColor3       = TEXT_LO
keyHint.Font             = Enum.Font.GothamBold
keyHint.TextSize         = 11
keyHint.Parent           = titleBar
local khc = Instance.new("UICorner"); khc.CornerRadius = UDim.new(0,4); khc.Parent = keyHint

-- Minimise button
local minBtn = Instance.new("TextButton")
minBtn.Size            = UDim2.new(0, 28, 0, 28)
minBtn.Position        = UDim2.new(1, -38, 0.5, -14)
minBtn.BackgroundColor3= BG_PANEL
minBtn.BorderSizePixel = 0
minBtn.Text            = "—"
minBtn.TextColor3      = TEXT_LO
minBtn.Font            = Enum.Font.GothamBold
minBtn.TextSize        = 14
minBtn.Parent          = titleBar
local minC = Instance.new("UICorner"); minC.CornerRadius = UDim.new(0,6); minC.Parent = minBtn

-- ── Drag (whole title bar) ──
do
    local dragging, dragStart, startPos
    titleBar.InputBegan:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseButton1
        or inp.UserInputType == Enum.UserInputType.Touch then
            dragging  = true
            dragStart = inp.Position
            startPos  = win.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(inp)
        if dragging and (inp.UserInputType == Enum.UserInputType.MouseMovement
            or inp.UserInputType == Enum.UserInputType.Touch) then
            local d = inp.Position - dragStart
            win.Position = UDim2.new(
                startPos.X.Scale, startPos.X.Offset + d.X,
                startPos.Y.Scale, startPos.Y.Offset + d.Y)
        end
    end)
    UserInputService.InputEnded:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseButton1
        or inp.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)
end

-- ── Scroll body ──
local body = Instance.new("ScrollingFrame")
body.Name                 = "Body"
body.Size                 = UDim2.new(1, 0, 1, -47)
body.Position             = UDim2.new(0, 0, 0, 47)
body.BackgroundTransparency = 1
body.BorderSizePixel      = 0
body.ScrollBarThickness   = 3
body.ScrollBarImageColor3 = ACCENT
body.AutomaticCanvasSize  = Enum.AutomaticSize.Y
body.CanvasSize           = UDim2.new(0, 0, 0, 0)
body.Parent               = win

local bodyLayout = Instance.new("UIListLayout")
bodyLayout.Padding             = UDim.new(0, 6)
bodyLayout.SortOrder           = Enum.SortOrder.LayoutOrder
bodyLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
bodyLayout.Parent              = body

local bodyPad = Instance.new("UIPadding")
bodyPad.PaddingTop    = UDim.new(0, 10)
bodyPad.PaddingBottom = UDim.new(0, 14)
bodyPad.PaddingLeft   = UDim.new(0, 10)
bodyPad.PaddingRight  = UDim.new(0, 10)
bodyPad.Parent        = body

-- ── Show / Hide logic ──
local winVisible = true
local FULL_SIZE  = UDim2.new(0, 300, 0, 560)
local MIN_SIZE   = UDim2.new(0, 300, 0, 47)

local function setWindowVisible(visible)
    winVisible = visible
    if visible then
        body.Visible = true
        win.Size     = FULL_SIZE
        win.BackgroundTransparency = 0
        minBtn.Text  = "—"
    else
        body.Visible = false
        win.Size     = MIN_SIZE
        win.BackgroundTransparency = 1
        stripe.Visible = false
        shadow.Visible = false
        titleBar.BackgroundTransparency = 1
        titleLabel.TextTransparency = 1
        keyHint.Visible = false
        minBtn.Visible  = false
    end
end

local function toggleWindowVisible()
    if winVisible then
        -- hide with tween
        TweenService:Create(win, TI_HIDE, {
            Size = MIN_SIZE,
            BackgroundTransparency = 1,
        }):Play()
        TweenService:Create(titleBar, TI_HIDE, {BackgroundTransparency = 1}):Play()
        TweenService:Create(titleLabel, TI_HIDE, {TextTransparency = 1}):Play()
        TweenService:Create(stripe, TI_HIDE, {BackgroundTransparency = 1}):Play()
        task.delay(0.2, function()
            body.Visible   = false
            keyHint.Visible= false
            minBtn.Visible = false
            shadow.Visible = false
        end)
        winVisible = false
    else
        -- show with tween
        win.BackgroundTransparency    = 1
        titleBar.BackgroundTransparency = 1
        titleLabel.TextTransparency   = 1
        stripe.BackgroundTransparency = 1
        body.Visible   = true
        keyHint.Visible= true
        minBtn.Visible = true
        shadow.Visible = true
        win.Size = FULL_SIZE
        TweenService:Create(win, TI_HIDE, {BackgroundTransparency = 0}):Play()
        TweenService:Create(titleBar, TI_HIDE, {BackgroundTransparency = 0}):Play()
        TweenService:Create(titleLabel, TI_HIDE, {TextTransparency = 0}):Play()
        TweenService:Create(stripe, TI_HIDE, {BackgroundTransparency = 0}):Play()
        winVisible = true
        minBtn.Text = "—"
    end
end

-- (keyboard bindings wired at the bottom after all GUI setters are defined)

-- Minimise button (collapses body, keeps title visible)
local bodyCollapsed = false
minBtn.MouseButton1Click:Connect(function()
    bodyCollapsed = not bodyCollapsed
    body.Visible = not bodyCollapsed
    TweenService:Create(win, TI, {
        Size = bodyCollapsed and MIN_SIZE or FULL_SIZE
    }):Play()
    minBtn.Text = bodyCollapsed and "+" or "—"
end)

-- ═══════════════════════════════════════════════
--  UI factory helpers
-- ═══════════════════════════════════════════════
local function sectionLabel(text, order)
    local lbl = Instance.new("TextLabel")
    lbl.Size             = UDim2.new(1, 0, 0, 20)
    lbl.BackgroundTransparency = 1
    lbl.Text             = text
    lbl.TextColor3       = ACCENT
    lbl.Font             = Enum.Font.GothamBold
    lbl.TextSize         = 11
    lbl.TextXAlignment   = Enum.TextXAlignment.Left
    lbl.LayoutOrder      = order
    lbl.Parent           = body
    return lbl
end

local function divider(order)
    local f = Instance.new("Frame")
    f.Size             = UDim2.new(1, 0, 0, 1)
    f.BackgroundColor3 = Color3.fromRGB(40, 48, 68)
    f.BorderSizePixel  = 0
    f.LayoutOrder      = order
    f.Parent           = body
    return f
end

local function makeToggle(labelText, order, initState, onToggle)
    local row = Instance.new("Frame")
    row.Name             = labelText .. "Row"
    row.Size             = UDim2.new(1, 0, 0, 40)
    row.BackgroundColor3 = BG_ROW
    row.BorderSizePixel  = 0
    row.LayoutOrder      = order
    row.Parent           = body
    local rc = Instance.new("UICorner"); rc.CornerRadius = UDim.new(0,8); rc.Parent = row

    local lbl = Instance.new("TextLabel")
    lbl.Size           = UDim2.new(1, -68, 1, 0)
    lbl.Position       = UDim2.new(0, 12, 0, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text           = labelText
    lbl.TextColor3     = TEXT_HI
    lbl.Font           = Enum.Font.Gotham
    lbl.TextSize       = 14
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Parent         = row

    local pill = Instance.new("TextButton")
    pill.Size            = UDim2.new(0, 52, 0, 26)
    pill.Position        = UDim2.new(1, -62, 0.5, -13)
    pill.BackgroundColor3= initState and GREEN or RED
    pill.BorderSizePixel = 0
    pill.Text            = ""
    pill.AutoButtonColor = false
    pill.Parent          = row
    local pc = Instance.new("UICorner"); pc.CornerRadius = UDim.new(1,0); pc.Parent = pill

    local dot = Instance.new("Frame")
    dot.Size             = UDim2.new(0, 20, 0, 20)
    dot.Position         = initState and UDim2.new(1,-23,0.5,-10) or UDim2.new(0,3,0.5,-10)
    dot.BackgroundColor3 = Color3.new(1,1,1)
    dot.BorderSizePixel  = 0
    dot.Parent           = pill
    local dc = Instance.new("UICorner"); dc.CornerRadius = UDim.new(1,0); dc.Parent = dot

    local state = initState
    pill.MouseButton1Click:Connect(function()
        state = not state
        TweenService:Create(pill, TI, {BackgroundColor3 = state and GREEN or RED}):Play()
        TweenService:Create(dot,  TI, {Position = state
            and UDim2.new(1,-23,0.5,-10) or UDim2.new(0,3,0.5,-10)}):Play()
        onToggle(state)
    end)

    -- external setter
    local function setter(s)
        state = s
        TweenService:Create(pill, TI, {BackgroundColor3 = s and GREEN or RED}):Play()
        TweenService:Create(dot,  TI, {Position = s
            and UDim2.new(1,-23,0.5,-10) or UDim2.new(0,3,0.5,-10)}):Play()
    end

    return row, setter
end

local function makeSlider(labelText, order, initVal, minVal, maxVal, fmt, onChange)
    local row = Instance.new("Frame")
    row.Name             = labelText .. "Slider"
    row.Size             = UDim2.new(1, 0, 0, 56)
    row.BackgroundColor3 = BG_ROW
    row.BorderSizePixel  = 0
    row.LayoutOrder      = order
    row.Parent           = body
    local rc = Instance.new("UICorner"); rc.CornerRadius = UDim.new(0,8); rc.Parent = row

    local lbl = Instance.new("TextLabel")
    lbl.Size           = UDim2.new(0.6, 0, 0, 22)
    lbl.Position       = UDim2.new(0, 12, 0, 6)
    lbl.BackgroundTransparency = 1
    lbl.Text           = labelText
    lbl.TextColor3     = TEXT_HI
    lbl.Font           = Enum.Font.Gotham
    lbl.TextSize       = 13
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Parent         = row

    local valLbl = Instance.new("TextLabel")
    valLbl.Size          = UDim2.new(0.38, 0, 0, 22)
    valLbl.Position      = UDim2.new(0.62, 0, 0, 6)
    valLbl.BackgroundTransparency = 1
    valLbl.Text          = string.format(fmt, initVal)
    valLbl.TextColor3    = ACCENT
    valLbl.Font          = Enum.Font.GothamBold
    valLbl.TextSize      = 13
    valLbl.TextXAlignment= Enum.TextXAlignment.Right
    valLbl.Parent        = row

    local track = Instance.new("Frame")
    track.Size           = UDim2.new(1, -24, 0, 6)
    track.Position       = UDim2.new(0, 12, 0, 36)
    track.BackgroundColor3 = Color3.fromRGB(40, 48, 68)
    track.BorderSizePixel = 0
    track.Parent         = row
    local tc = Instance.new("UICorner"); tc.CornerRadius = UDim.new(1,0); tc.Parent = track

    local fill = Instance.new("Frame")
    fill.Size            = UDim2.new((initVal-minVal)/(maxVal-minVal), 0, 1, 0)
    fill.BackgroundColor3= ACCENT
    fill.BorderSizePixel = 0
    fill.Parent          = track
    local fc = Instance.new("UICorner"); fc.CornerRadius = UDim.new(1,0); fc.Parent = fill

    local handle = Instance.new("TextButton")
    handle.Size          = UDim2.new(0, 16, 0, 16)
    handle.AnchorPoint   = Vector2.new(0.5, 0.5)
    handle.Position      = UDim2.new((initVal-minVal)/(maxVal-minVal), 0, 0.5, 0)
    handle.BackgroundColor3 = Color3.new(1,1,1)
    handle.BorderSizePixel = 0
    handle.Text          = ""
    handle.AutoButtonColor = false
    handle.Parent        = track
    local hc = Instance.new("UICorner"); hc.CornerRadius = UDim.new(1,0); hc.Parent = handle

    local dragging = false
    handle.MouseButton1Down:Connect(function() dragging = true end)
    UserInputService.InputEnded:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseButton1
        or inp.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)

    local function setFromAbsX(absX)
        local tx = track.AbsolutePosition.X
        local tw = track.AbsoluteSize.X
        local t  = math.clamp((absX - tx) / tw, 0, 1)
        local val = minVal + t * (maxVal - minVal)
        if maxVal - minVal >= 10 then val = math.round(val) end
        fill.Size       = UDim2.new(t, 0, 1, 0)
        handle.Position = UDim2.new(t, 0, 0.5, 0)
        valLbl.Text     = string.format(fmt, val)
        onChange(val)
    end

    track.InputBegan:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseButton1
        or inp.UserInputType == Enum.UserInputType.Touch then
            setFromAbsX(inp.Position.X)
            dragging = true
        end
    end)
    UserInputService.InputChanged:Connect(function(inp)
        if dragging and (inp.UserInputType == Enum.UserInputType.MouseMovement
            or inp.UserInputType == Enum.UserInputType.Touch) then
            setFromAbsX(inp.Position.X)
        end
    end)
end

local function makeStepper(labelText, order, initVal, minVal, maxVal, step, onChange)
    local row = Instance.new("Frame")
    row.Name             = labelText .. "Stepper"
    row.Size             = UDim2.new(1, 0, 0, 44)
    row.BackgroundColor3 = BG_ROW
    row.BorderSizePixel  = 0
    row.LayoutOrder      = order
    row.Parent           = body
    local rc = Instance.new("UICorner"); rc.CornerRadius = UDim.new(0,8); rc.Parent = row

    local lbl = Instance.new("TextLabel")
    lbl.Size           = UDim2.new(0.5, 0, 1, 0)
    lbl.Position       = UDim2.new(0, 12, 0, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text           = labelText
    lbl.TextColor3     = TEXT_HI
    lbl.Font           = Enum.Font.Gotham
    lbl.TextSize       = 14
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Parent         = row

    local function makeBtn(txt, xPos)
        local b = Instance.new("TextButton")
        b.Size           = UDim2.new(0, 30, 0, 26)
        b.Position       = UDim2.new(1, xPos, 0.5, -13)
        b.BackgroundColor3 = BG_PANEL
        b.BorderSizePixel = 0
        b.Text           = txt
        b.TextColor3     = ACCENT
        b.Font           = Enum.Font.GothamBold
        b.TextSize       = 16
        b.AutoButtonColor = true
        b.Parent         = row
        local bc = Instance.new("UICorner"); bc.CornerRadius = UDim.new(0,6); bc.Parent = b
        return b
    end

    local minusBtn = makeBtn("−", -98)
    local plusBtn  = makeBtn("+", -34)

    local valLbl = Instance.new("TextLabel")
    valLbl.Size          = UDim2.new(0, 46, 0, 26)
    valLbl.Position      = UDim2.new(1, -64, 0.5, -13)
    valLbl.BackgroundTransparency = 1
    valLbl.Text          = tostring(initVal)
    valLbl.TextColor3    = ACCENT
    valLbl.Font          = Enum.Font.GothamBold
    valLbl.TextSize      = 14
    valLbl.TextXAlignment= Enum.TextXAlignment.Center
    valLbl.Parent        = row

    local cur = initVal
    local function set(v)
        cur = math.clamp(v, minVal, maxVal)
        valLbl.Text = tostring(cur)
        onChange(cur)
    end

    minusBtn.MouseButton1Click:Connect(function() set(cur - step) end)
    plusBtn.MouseButton1Click:Connect(function()  set(cur + step) end)
end

-- ═══════════════════════════════════════════════
--  Build UI sections
-- ═══════════════════════════════════════════════

-- Pill-setter upvalues — assigned during build, used by resetAllToggles below
local setHighlightsPill, setNameTagsPill, setAmbientPill, setCamPill

local order = 1

sectionLabel("  VISUALS", order);               order += 1
local _, _setHL = makeToggle("Player Highlights", order, cfg.highlightsEnabled, setHighlights); setHighlightsPill = _setHL; order += 1
local _, _setNT = makeToggle("Name Tags",         order, cfg.nameTagsEnabled,   setNameTags);   setNameTagsPill   = _setNT; order += 1
local _, _setAM = makeToggle("Ambient Light",     order, cfg.ambientEnabled,    setAmbient);    setAmbientPill    = _setAM; order += 1
divider(order);                                  order += 1

sectionLabel("  TRANSPARENCY", order);           order += 1
makeSlider("Fill Transparency", order, cfg.fillTransparency, 0, 1, "%.2f", function(v)
    cfg.fillTransparency = v
    for hl in pairs(trackedHighlights) do
        if hl.Parent then hl.FillTransparency = v else trackedHighlights[hl] = nil end
    end
end); order += 1

makeSlider("Outline Transparency", order, cfg.outlineTransparency, 0, 1, "%.2f", function(v)
    cfg.outlineTransparency = v
    for hl in pairs(trackedHighlights) do
        if hl.Parent then hl.OutlineTransparency = v else trackedHighlights[hl] = nil end
    end
end); order += 1

makeSlider("Tag Text Transparency", order, cfg.tagTextTransparency, 0, 1, "%.2f", function(v)
    cfg.tagTextTransparency = v
    for bb in pairs(trackedBillboards) do
        if bb.Parent then
            local lbl = bb:FindFirstChild("NameLabel")
            if lbl then
                lbl.TextTransparency       = v
                lbl.TextStrokeTransparency = math.min(1, v + 0.35)
            end
        else trackedBillboards[bb] = nil end
    end
end); order += 1
divider(order); order += 1

sectionLabel("  MOVEMENT", order); order += 1
makeStepper("Walk Speed", order, cfg.walkSpeed, 4, 200, 4, function(v)
    cfg.walkSpeed = v; applyMovement()
end); order += 1
makeStepper("Jump Power", order, cfg.jumpPower, 10, 300, 10, function(v)
    cfg.jumpPower = v; applyMovement()
end); order += 1
divider(order); order += 1

sectionLabel("  CAMERA MODE", order); order += 1
local _, _setCam = makeToggle("Camera Spectate", order, false, function(state)
    if state then toggleSpectate() else stopSpectate() end
end); setCamPill = _setCam; order += 1

-- Nav row (prev / next / target label)
do
    local navRow = Instance.new("Frame")
    navRow.Name             = "NavRow"
    navRow.Size             = UDim2.new(1, 0, 0, 40)
    navRow.BackgroundColor3 = BG_ROW
    navRow.BorderSizePixel  = 0
    navRow.LayoutOrder      = order
    navRow.Parent           = body
    local nrc = Instance.new("UICorner"); nrc.CornerRadius = UDim.new(0,8); nrc.Parent = navRow

    local targetLbl = Instance.new("TextLabel")
    targetLbl.Size          = UDim2.new(1, -100, 1, 0)
    targetLbl.Position      = UDim2.new(0, 12, 0, 0)
    targetLbl.BackgroundTransparency = 1
    targetLbl.Text          = "No target"
    targetLbl.TextColor3    = TEXT_LO
    targetLbl.Font          = Enum.Font.Gotham
    targetLbl.TextSize      = 13
    targetLbl.TextXAlignment= Enum.TextXAlignment.Left
    targetLbl.TextTruncate  = Enum.TextTruncate.AtEnd
    targetLbl.Parent        = navRow

    local function makeNavBtn(txt, xOff)
        local b = Instance.new("TextButton")
        b.Size           = UDim2.new(0, 36, 0, 28)
        b.Position       = UDim2.new(1, xOff, 0.5, -14)
        b.BackgroundColor3 = BG_PANEL
        b.BorderSizePixel = 0
        b.Text           = txt
        b.TextColor3     = ACCENT
        b.Font           = Enum.Font.GothamBold
        b.TextSize       = 18
        b.AutoButtonColor = true
        b.Parent         = navRow
        local bc = Instance.new("UICorner"); bc.CornerRadius = UDim.new(0,6); bc.Parent = b
        return b
    end

    local prevBtn = makeNavBtn("◀", -84)
    local nextBtn = makeNavBtn("▶", -44)

    local function syncLabel()
        targetLbl.Text     = spectateTarget and spectateTarget.Name or "No target"
        targetLbl.TextColor3 = spectateTarget and TEXT_HI or TEXT_LO
    end

    prevBtn.MouseButton1Click:Connect(function()
        if not spectating then setCamPill(true) end
        spectatePrev(); syncLabel()
    end)
    nextBtn.MouseButton1Click:Connect(function()
        if not spectating then setCamPill(true) end
        spectateNext(); syncLabel()
    end)

    -- Auto-advance if target dies / leaves workspace
    RunService.Heartbeat:Connect(function()
        if spectating and spectateTarget then
            local hum = spectateTarget:FindFirstChildOfClass("Humanoid")
            if not hum or hum.Health <= 0
                or not spectateTarget:IsDescendantOf(workspace) then
                spectateNext(); syncLabel()
            end
        end
    end)

    order += 1
end

-- ── Hint row at the bottom ──
do
    local hint = Instance.new("TextLabel")
    hint.Size             = UDim2.new(1, 0, 0, 24)
    hint.BackgroundTransparency = 1
    hint.Text             = "[H] hide/show   [R] reset   [M×3] kill"
    hint.TextColor3       = TEXT_LO
    hint.Font             = Enum.Font.Gotham
    hint.TextSize         = 11
    hint.LayoutOrder      = order
    hint.Parent           = body
    order += 1
end

-- ═══════════════════════════════════════════════
--  Keyboard bindings (must be AFTER all pill
--  setters are assigned above)
-- ═══════════════════════════════════════════════

local function resetAllToggles()
    setHighlights(false);  setHighlightsPill(false)
    setNameTags(false);    setNameTagsPill(false)
    setAmbient(false);     setAmbientPill(false)
    stopSpectate();        setCamPill(false)
end

-- Kill key: press KILL_KEY 3 times within 1 second
local killPresses = 0
local killTimer   = 0

local function destroyAll()
    -- restore lighting first
    setAmbient(false)
    stopSpectate()
    -- destroy all highlights and name tags
    for hl in pairs(trackedHighlights) do
        if hl and hl.Parent then hl:Destroy() end
    end
    for bb in pairs(trackedBillboards) do
        if bb and bb.Parent then bb:Destroy() end
    end
    -- destroy the GUI
    gui:Destroy()
    -- stop the script's connections by destroying the script itself
    script:Destroy()
end

RunService.Heartbeat:Connect(function(dt)
    if killPresses > 0 then
        killTimer = killTimer + dt
        if killTimer > 1 then
            killPresses = 0
            killTimer   = 0
        end
    end
end)

UserInputService.InputBegan:Connect(function(inp, processed)
    if processed then return end

    if inp.KeyCode == TOGGLE_KEY then
        toggleWindowVisible()

    elseif inp.KeyCode == RESET_KEY then
        resetAllToggles()

    elseif inp.KeyCode == KILL_KEY then
        killPresses = killPresses + 1
        killTimer   = 0
        if killPresses >= 3 then
            destroyAll()
        end
    end
end)
