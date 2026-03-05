--[[
    ZURION SUPREME v106 - "STABLE HYBRID"
    -----------------------------------------------------------
    ✅ AUTO-AIM: Sistema RIVALS (funcionando)
    ✅ AUTO-SHOT: Disparo automático (funcionando)
    ✅ ESP: Sistema v102 (sem bugs)
    ✅ WHITELIST: Sistema v102 (funcionando)
    ✅ FLY: Sistema RIVALS (estável)
    ✅ NOCLIP: Sistema v102 (funcionando)
    ✅ UI: Melhorada
    -----------------------------------------------------------
]]--

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local Mouse = LocalPlayer:GetMouse()

-- [ CONFIG ]
local CONFIG = {
    COMBAT = {
        ENABLED = false,
        AUTO_SHOOT = false,
        FOV = 250,
        SMOOTH = 1.5,
        PRED = 0.15,
        PART = "Head",
        WALL_CHECK = true,
        AUTO_SHOOT_DELAY = 0.08
    },
    VISUALS = {
        ENABLED = false,
        BOXES = false,
        NAMES = false,
        TRACERS = false,
        DISTANCE_TEXT = false,
        ACCENT = Color3.fromRGB(0, 170, 255),
        FRIEND_COLOR = Color3.fromRGB(0, 255, 120)
    },
    MOVEMENT = {
        FLY_ENABLED = false,
        FLY_SPEED = 180,
        NOCLIP = false,
        KEYS = {W = false, S = false, A = false, D = false, UP = false, DOWN = false}
    },
    MENU = {
        ACCENT = Color3.fromRGB(0, 170, 255),
        ACCENT_DARK = Color3.fromRGB(0, 120, 200),
        BG = Color3.fromRGB(10, 10, 15),
        SECONDARY = Color3.fromRGB(18, 18, 25),
        TERTIARY = Color3.fromRGB(25, 25, 35)
    }
}

local WHITELIST = {}
local UI_PAGES = {}
local DRAWINGS = {ESP = {}, FOV = nil}
local LAST_SHOT_TIME = 0
local CURRENT_TARGET = nil

-- [ UI PROTECTION ]
local Screen = Instance.new("ScreenGui")
Screen.Name = "Zurion_v106_Stable"
Screen.ResetOnSpawn = false
Screen.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

pcall(function()
    Screen.Parent = CoreGui
end)

if Screen.Parent == nil then
    Screen.Parent = LocalPlayer:WaitForChild("PlayerGui")
end

-- [ FOV CIRCLE ]
local FOV_CIRCLE = Drawing.new("Circle")
FOV_CIRCLE.Thickness = 2
FOV_CIRCLE.NumSides = 60
FOV_CIRCLE.Radius = CONFIG.COMBAT.FOV
FOV_CIRCLE.Filled = false
FOV_CIRCLE.Visible = false
FOV_CIRCLE.Color = CONFIG.MENU.ACCENT
FOV_CIRCLE.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

-- [ ESP BUILDER - VERSÃO ESTÁVEL ]
local function CreateESP(p)
    if DRAWINGS.ESP[p] then return end
    DRAWINGS.ESP[p] = {
        Box = Drawing.new("Square"),
        Name = Drawing.new("Text"),
        Line = Drawing.new("Line"),
        Distance = Drawing.new("Text")
    }
    local d = DRAWINGS.ESP[p]
    d.Box.Thickness = 2
    d.Box.Filled = false
    d.Name.Size = 13
    d.Name.Center = true
    d.Name.Outline = true
    d.Distance.Size = 11
    d.Distance.Center = true
    d.Distance.Outline = true
    d.Line.Thickness = 1
end

-- [ MAIN UI ]
local Main = Instance.new("Frame", Screen)
Main.Size = UDim2.new(0, 780, 0, 600)
Main.Position = UDim2.new(0.5, -390, 0.5, -300)
Main.BackgroundColor3 = CONFIG.MENU.BG
Main.BorderSizePixel = 0
Main.Visible = true
Main.ClipsDescendants = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 15)

local MStroke = Instance.new("UIStroke", Main)
MStroke.Color = CONFIG.MENU.ACCENT
MStroke.Thickness = 2

local Gradient = Instance.new("UIGradient", Main)
Gradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, CONFIG.MENU.BG),
    ColorSequenceKeypoint.new(1, CONFIG.MENU.SECONDARY)
})
Gradient.Rotation = 45

local Sidebar = Instance.new("Frame", Main)
Sidebar.Size = UDim2.new(0, 240, 1, 0)
Sidebar.BackgroundColor3 = CONFIG.MENU.SECONDARY
Sidebar.BorderSizePixel = 0
Instance.new("UICorner", Sidebar)

local SideGradient = Instance.new("UIGradient", Sidebar)
SideGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, CONFIG.MENU.SECONDARY),
    ColorSequenceKeypoint.new(1, CONFIG.MENU.TERTIARY)
})

local Logo = Instance.new("TextLabel", Sidebar)
Logo.Size = UDim2.new(1, 0, 0, 120)
Logo.Text = "⚡ ZURION\nSUPREME v106"
Logo.Font = "GothamBold"
Logo.TextSize = 24
Logo.TextColor3 = CONFIG.MENU.ACCENT
Logo.BackgroundTransparency = 1

local TabContainer = Instance.new("Frame", Sidebar)
TabContainer.Size = UDim2.new(1, 0, 1, -140)
TabContainer.Position = UDim2.new(0, 0, 0, 130)
TabContainer.BackgroundTransparency = 1
local TabLayout = Instance.new("UIListLayout", TabContainer)
TabLayout.HorizontalAlignment = "Center"
TabLayout.Padding = UDim.new(0, 10)

local Content = Instance.new("Frame", Main)
Content.Size = UDim2.new(1, -270, 1, -50)
Content.Position = UDim2.new(0, 260, 0, 30)
Content.BackgroundTransparency = 1

local TopBar = Instance.new("Frame", Main)
TopBar.Size = UDim2.new(1, 0, 0, 30)
TopBar.BackgroundColor3 = CONFIG.MENU.ACCENT
TopBar.BorderSizePixel = 0
local TopLabel = Instance.new("TextLabel", TopBar)
TopLabel.Size = UDim2.new(1, 0, 1, 0)
TopLabel.Text = "🎯 PAINEL DE CONTROLE"
TopLabel.Font = "GothamBold"
TopLabel.TextColor3 = Color3.new(1, 1, 1)
TopLabel.TextSize = 14
TopLabel.BackgroundTransparency = 1

-- [ UI FUNCTIONS ]
local function NewPage(name)
    local p = Instance.new("ScrollingFrame", Content)
    p.Size = UDim2.new(1, 0, 1, 0)
    p.BackgroundTransparency = 1
    p.Visible = false
    p.ScrollBarThickness = 5
    p.ScrollBarImageColor3 = CONFIG.MENU.ACCENT
    p.CanvasSize = UDim2.new(0, 0, 0, 0)
    p.AutomaticCanvasSize = "Y"
    Instance.new("UIListLayout", p).Padding = UDim.new(0, 15)
    UI_PAGES[name] = p
    return p
end

local function NewTab(name, page)
    local b = Instance.new("TextButton", TabContainer)
    b.Size = UDim2.new(0, 220, 0, 50)
    b.BackgroundColor3 = CONFIG.MENU.TERTIARY
    b.Text = "📌 " .. name
    b.Font = "GothamBold"
    b.TextColor3 = Color3.new(0.6, 0.6, 0.6)
    b.TextSize = 13
    b.BorderSizePixel = 0
    Instance.new("UICorner", b).CornerRadius = UDim.new(0, 8)
    
    local bStroke = Instance.new("UIStroke", b)
    bStroke.Color = Color3.fromRGB(40, 40, 50)
    bStroke.Thickness = 1
    
    b.MouseButton1Click:Connect(function()
        for _, pg in pairs(UI_PAGES) do pg.Visible = false end
        page.Visible = true
        for _, btn in pairs(TabContainer:GetChildren()) do
            if btn:IsA("TextButton") then
                btn.TextColor3 = Color3.new(0.6, 0.6, 0.6)
                btn.BackgroundColor3 = CONFIG.MENU.TERTIARY
            end
        end
        b.TextColor3 = CONFIG.MENU.ACCENT
        b.BackgroundColor3 = CONFIG.MENU.SECONDARY
    end)
end

local function NewToggle(parent, text, tbl, key, call)
    local f = Instance.new("Frame", parent)
    f.Size = UDim2.new(1, -15, 0, 60)
    f.BackgroundColor3 = CONFIG.MENU.TERTIARY
    f.BorderSizePixel = 0
    Instance.new("UICorner", f).CornerRadius = UDim.new(0, 8)
    
    local fStroke = Instance.new("UIStroke", f)
    fStroke.Color = Color3.fromRGB(50, 50, 60)
    fStroke.Thickness = 1
    
    local l = Instance.new("TextLabel", f)
    l.Size = UDim2.new(1, -80, 1, 0)
    l.Position = UDim2.new(0, 15, 0, 0)
    l.Text = text
    l.Font = "GothamBold"
    l.TextColor3 = Color3.new(1, 1, 1)
    l.TextXAlignment = "Left"
    l.BackgroundTransparency = 1
    l.TextSize = 13
    
    local btn = Instance.new("TextButton", f)
    btn.Size = UDim2.new(0, 50, 0, 28)
    btn.Position = UDim2.new(1, -65, 0.5, -14)
    btn.BackgroundColor3 = tbl[key] and CONFIG.MENU.ACCENT or Color3.fromRGB(40, 40, 50)
    btn.Text = ""
    btn.BorderSizePixel = 0
    Instance.new("UICorner", btn).CornerRadius = UDim.new(1, 0)
    
    btn.MouseButton1Click:Connect(function()
        tbl[key] = not tbl[key]
        TweenService:Create(btn, TweenInfo.new(0.2), {
            BackgroundColor3 = tbl[key] and CONFIG.MENU.ACCENT or Color3.fromRGB(40, 40, 50)
        }):Play()
        if call then call(tbl[key]) end
    end)
end

local function NewSlider(parent, text, min, max, tbl, key, callback)
    local f = Instance.new("Frame", parent)
    f.Size = UDim2.new(1, -15, 0, 85)
    f.BackgroundColor3 = CONFIG.MENU.TERTIARY
    f.BorderSizePixel = 0
    Instance.new("UICorner", f).CornerRadius = UDim.new(0, 8)
    
    local fStroke = Instance.new("UIStroke", f)
    fStroke.Color = Color3.fromRGB(50, 50, 60)
    fStroke.Thickness = 1
    
    local l = Instance.new("TextLabel", f)
    l.Size = UDim2.new(1, 0, 0, 45)
    l.Position = UDim2.new(0, 15, 0, 0)
    l.Text = text .. ": " .. tbl[key]
    l.TextColor3 = Color3.new(1, 1, 1)
    l.Font = "GothamBold"
    l.TextXAlignment = "Left"
    l.BackgroundTransparency = 1
    l.TextSize = 13
    
    local bar = Instance.new("TextButton", f)
    bar.Size = UDim2.new(0.9, 0, 0, 8)
    bar.Position = UDim2.new(0.05, 0, 0.65, 0)
    bar.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
    bar.Text = ""
    bar.BorderSizePixel = 0
    Instance.new("UICorner", bar).CornerRadius = UDim.new(1, 0)
    
    local fill = Instance.new("Frame", bar)
    fill.Size = UDim2.new((tbl[key] - min) / (max - min), 0, 1, 0)
    fill.BackgroundColor3 = CONFIG.MENU.ACCENT
    fill.BorderSizePixel = 0
    Instance.new("UICorner", fill).CornerRadius = UDim.new(1, 0)
    
    bar.MouseButton1Down:Connect(function()
        local m
        m = RunService.RenderStepped:Connect(function()
            local p = math.clamp((UIS:GetMouseLocation().X - bar.AbsolutePosition.X) / bar.AbsoluteSize.X, 0, 1)
            local v = math.floor(min + (max - min) * p)
            tbl[key] = v
            l.Text = text .. ": " .. v
            fill.Size = UDim2.new(p, 0, 1, 0)
            if callback then callback(v) end
            if not UIS:IsMouseButtonPressed(Enum.UserInputType.MouseButton1) then
                m:Disconnect()
            end
        end)
    end)
end

-- [ PAGES ]
local CombatPg = NewPage("Combat")
local VisualPg = NewPage("Visuals")
local MovePg = NewPage("Movement")
local WLPg = NewPage("Whitelist")

NewTab("COMBAT", CombatPg)
NewTab("VISUALS", VisualPg)
NewTab("MOVEMENT", MovePg)
NewTab("WHITELIST", WLPg)

-- [ TARGETING - RIVALS SYSTEM ]
local function CheckLineOfSight(from, to)
    local direction = (to - from)
    local distance = direction.Magnitude
    if distance == 0 then return false end
    
    local raycastParams = RaycastParams.new()
    raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
    raycastParams.FilterDescendantsInstances = {LocalPlayer.Character}
    
    local result = workspace:Raycast(from, direction.Unit * distance, raycastParams)
    return result == nil
end

local function GetClosestTarget()
    local target = nil
    local shortestDist = CONFIG.COMBAT.FOV
    local screenCenter = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            if WHITELIST[player.Name] then continue end
            
            local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
            local head = player.Character:FindFirstChild(CONFIG.COMBAT.PART)
            
            if not head or not humanoid or humanoid.Health <= 0 then continue end
            
            local screenPos, onScreen = Camera:WorldToViewportPoint(head.Position)
            if not onScreen then continue end
            
            if CONFIG.COMBAT.WALL_CHECK then
                if not CheckLineOfSight(Camera.CFrame.Position, head.Position) then continue end
            end
            
            local screenDistance = (Vector2.new(screenPos.X, screenPos.Y) - screenCenter).Magnitude
            if screenDistance < shortestDist then
                shortestDist = screenDistance
                target = head
            end
        end
    end
    
    return target
end

-- [ AIMBOT - MOUSE DELTA ]
local function AimAt(target)
    if not target then return end
    
    local vel = target.Parent.PrimaryPart and target.Parent.PrimaryPart.AssemblyLinearVelocity or Vector3.new(0,0,0)
    local predictedPos = target.Position + (vel * CONFIG.COMBAT.PRED)
    
    local screenPos, onScreen = Camera:WorldToViewportPoint(predictedPos)
    if onScreen then
        local mousePos = UIS:GetMouseLocation()
        local deltaX = (screenPos.X - mousePos.X)
        local deltaY = (screenPos.Y - mousePos.Y)
        
        local smoothScale = CONFIG.COMBAT.SMOOTH
        mousemoverel(deltaX / smoothScale, deltaY / smoothScale)
    end
end

-- [ AUTO SHOOT ]
local function TriggerShoot()
    if not CONFIG.COMBAT.AUTO_SHOOT then return end
    
    local currentTime = tick()
    if currentTime - LAST_SHOT_TIME < CONFIG.COMBAT.AUTO_SHOOT_DELAY then return end
    
    LAST_SHOT_TIME = currentTime
    
    task.spawn(function()
        pcall(function()
            mouse1press()
            task.wait(0.01)
            mouse1release()
        end)
    end)
end

-- [ RENDER LOOP ]
RunService.RenderStepped:Connect(function()
    FOV_CIRCLE.Radius = CONFIG.COMBAT.FOV
    FOV_CIRCLE.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    
    -- NOCLIP
    if CONFIG.MOVEMENT.NOCLIP and LocalPlayer.Character then
        for _, v in pairs(LocalPlayer.Character:GetDescendants()) do
            if v:IsA("BasePart") then
                v.CanCollide = false
            end
        end
    end
    
    -- AIMBOT
    if CONFIG.COMBAT.ENABLED then
        CURRENT_TARGET = GetClosestTarget()
        if CURRENT_TARGET then
            AimAt(CURRENT_TARGET)
            TriggerShoot()
        end
    end
    
    -- FLY
    if CONFIG.MOVEMENT.FLY_ENABLED and LocalPlayer.Character then
        local r = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        if r then
            LocalPlayer.Character.Humanoid.PlatformStand = true
            local v = Vector3.new(0, 0, 0)
            if CONFIG.MOVEMENT.KEYS.W then v += Camera.CFrame.LookVector end
            if CONFIG.MOVEMENT.KEYS.S then v -= Camera.CFrame.LookVector end
            if CONFIG.MOVEMENT.KEYS.A then v -= Camera.CFrame.RightVector end
            if CONFIG.MOVEMENT.KEYS.D then v += Camera.CFrame.RightVector end
            r.Velocity = v * CONFIG.MOVEMENT.FLY_SPEED
        end
    end
    
    -- ESP RENDER
    for p, d in pairs(DRAWINGS.ESP) do
        if CONFIG.VISUALS.ENABLED and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
            local root = p.Character.HumanoidRootPart
            local pos, on = Camera:WorldToViewportPoint(root.Position)
            local isWhitelisted = WHITELIST[p.Name]
            local drawColor = isWhitelisted and CONFIG.VISUALS.FRIEND_COLOR or CONFIG.VISUALS.ACCENT
            local distance = (root.Position - Camera.CFrame.Position).Magnitude
            
            if on then
                local t = Camera:WorldToViewportPoint(root.Position + Vector3.new(0, 3, 0))
                local b = Camera:WorldToViewportPoint(root.Position - Vector3.new(0, 3.5, 0))
                local h = math.abs(t.Y - b.Y)
                local w = h / 1.5
                
                if CONFIG.VISUALS.BOXES then
                    d.Box.Visible = true
                    d.Box.Size = Vector2.new(w, h)
                    d.Box.Position = Vector2.new(pos.X - w / 2, pos.Y - h / 2)
                    d.Box.Color = drawColor
                else
                    d.Box.Visible = false
                end
                
                if CONFIG.VISUALS.NAMES then
                    d.Name.Visible = true
                    d.Name.Text = p.DisplayName
                    d.Name.Position = Vector2.new(pos.X, pos.Y - h / 2 - 18)
                    d.Name.Color = drawColor
                else
                    d.Name.Visible = false
                end
                
                if CONFIG.VISUALS.TRACERS then
                    d.Line.Visible = true
                    d.Line.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
                    d.Line.To = Vector2.new(pos.X, pos.Y)
                    d.Line.Color = drawColor
                else
                    d.Line.Visible = false
                end
                
                if CONFIG.VISUALS.DISTANCE_TEXT then
                    d.Distance.Visible = true
                    d.Distance.Text = string.format("%.1fm", distance)
                    d.Distance.Position = Vector2.new(pos.X, pos.Y + h / 2 + 15)
                    d.Distance.Color = drawColor
                else
                    d.Distance.Visible = false
                end
            else
                d.Box.Visible = false
                d.Name.Visible = false
                d.Line.Visible = false
                d.Distance.Visible = false
            end
        else
            d.Box.Visible = false
            d.Name.Visible = false
            d.Line.Visible = false
            d.Distance.Visible = false
        end
    end
end)

-- [ WHITELIST UPDATER ]
local function RefreshWL()
    for _, c in pairs(WLPg:GetChildren()) do
        if c:IsA("Frame") then c:Destroy() end
    end
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LocalPlayer then
            local f = Instance.new("Frame", WLPg)
            f.Size = UDim2.new(1, -15, 0, 55)
            f.BackgroundColor3 = CONFIG.MENU.TERTIARY
            f.BorderSizePixel = 0
            Instance.new("UICorner", f).CornerRadius = UDim.new(0, 8)
            
            local fStroke = Instance.new("UIStroke", f)
            fStroke.Color = Color3.fromRGB(50, 50, 60)
            fStroke.Thickness = 1
            
            local l = Instance.new("TextLabel", f)
            l.Size = UDim2.new(1, -100, 1, 0)
            l.Position = UDim2.new(0, 15, 0, 0)
            l.Text = p.DisplayName
            l.TextColor3 = Color3.new(1, 1, 1)
            l.Font = "GothamBold"
            l.TextXAlignment = "Left"
            l.BackgroundTransparency = 1
            l.TextSize = 13
            
            local b = Instance.new("TextButton", f)
            b.Size = UDim2.new(0, 85, 0, 35)
            b.Position = UDim2.new(1, -95, 0.5, -17.5)
            b.BackgroundColor3 = WHITELIST[p.Name] and Color3.fromRGB(0, 200, 100) or CONFIG.MENU.SECONDARY
            b.Text = WHITELIST[p.Name] and "✓ AMIGO" or "+ ADD"
            b.TextColor3 = Color3.new(1, 1, 1)
            b.Font = "GothamBold"
            b.BorderSizePixel = 0
            b.TextSize = 12
            Instance.new("UICorner", b).CornerRadius = UDim.new(0, 6)
            
            b.MouseButton1Click:Connect(function()
                WHITELIST[p.Name] = not WHITELIST[p.Name]
                b.BackgroundColor3 = WHITELIST[p.Name] and Color3.fromRGB(0, 200, 100) or CONFIG.MENU.SECONDARY
                b.Text = WHITELIST[p.Name] and "✓ AMIGO" or "+ ADD"
            end)
        end
    end
end

-- [ UI ELEMENTS - COMBAT ]
NewToggle(CombatPg, "🎯 Aimbot Rivals", CONFIG.COMBAT, "ENABLED", function(v)
    FOV_CIRCLE.Visible = v
end)
NewToggle(CombatPg, "🔫 Auto-Shot", CONFIG.COMBAT, "AUTO_SHOOT")
NewSlider(CombatPg, "⏱️ Shot Delay (ms)", 10, 500, CONFIG.COMBAT, "AUTO_SHOOT_DELAY", function(v)
    CONFIG.COMBAT.AUTO_SHOOT_DELAY = v / 1000
end)
NewSlider(CombatPg, "🎪 Smoothness", 0.5, 10, CONFIG.COMBAT, "SMOOTH")
NewSlider(CombatPg, "👁️ Field of View", 50, 800, CONFIG.COMBAT, "FOV")
NewToggle(CombatPg, "🚧 Wall Check", CONFIG.COMBAT, "WALL_CHECK")

-- [ UI ELEMENTS - VISUALS ]
NewToggle(VisualPg, "👁️ ESP Master", CONFIG.VISUALS, "ENABLED")
NewToggle(VisualPg, "📦 Show Boxes", CONFIG.VISUALS, "BOXES")
NewToggle(VisualPg, "📝 Show Names", CONFIG.VISUALS, "NAMES")
NewToggle(VisualPg, "📍 Show Tracers", CONFIG.VISUALS, "TRACERS")
NewToggle(VisualPg, "📐 Show Distance", CONFIG.VISUALS, "DISTANCE_TEXT")

-- [ UI ELEMENTS - MOVEMENT ]
NewToggle(MovePg, "👻 Noclip", CONFIG.MOVEMENT, "NOCLIP")
NewToggle(MovePg, "🚀 Flight", CONFIG.MOVEMENT, "FLY_ENABLED")
NewSlider(MovePg, "⚡ Fly Speed", 50, 500, CONFIG.MOVEMENT, "FLY_SPEED")

-- [ OPEN BUTTON ]
local Z = Instance.new("TextButton", Screen)
Z.Name = "ZurionOpen"
Z.Size = UDim2.new(0, 70, 0, 70)
Z.Position = UDim2.new(0, 5, 0.5, -35)
Z.BackgroundColor3 = CONFIG.MENU.BG
Z.TextColor3 = CONFIG.MENU.ACCENT
Z.Text = "Z"
Z.Font = "GothamBold"
Z.TextSize = 35
Z.BorderSizePixel = 0
Instance.new("UICorner", Z).CornerRadius = UDim.new(1, 0)
local ZStroke = Instance.new("UIStroke", Z)
ZStroke.Color = CONFIG.MENU.ACCENT
ZStroke.Thickness = 2

Z.MouseButton1Click:Connect(function()
    Main.Visible = not Main.Visible
end)

-- [ INPUTS ]
UIS.InputBegan:Connect(function(i, g)
    if g then return end
    if i.KeyCode == Enum.KeyCode.W then CONFIG.MOVEMENT.KEYS.W = true end
    if i.KeyCode == Enum.KeyCode.S then CONFIG.MOVEMENT.KEYS.S = true end
    if i.KeyCode == Enum.KeyCode.A then CONFIG.MOVEMENT.KEYS.A = true end
    if i.KeyCode == Enum.KeyCode.D then CONFIG.MOVEMENT.KEYS.D = true end
    if i.KeyCode == Enum.KeyCode.Insert then Main.Visible = not Main.Visible end
end)

UIS.InputEnded:Connect(function(i)
    if i.KeyCode == Enum.KeyCode.W then CONFIG.MOVEMENT.KEYS.W = false end
    if i.KeyCode == Enum.KeyCode.S then CONFIG.MOVEMENT.KEYS.S = false end
    if i.KeyCode == Enum.KeyCode.A then CONFIG.MOVEMENT.KEYS.A = false end
    if i.KeyCode == Enum.KeyCode.D then CONFIG.MOVEMENT.KEYS.D = false end
end)

-- [ DRAGGABLE ]
local d, ds, sp
Main.InputBegan:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then
        d = true
        ds = i.Position
        sp = Main.Position
    end
end)

Main.InputChanged:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseMovement and d then
        local dl = i.Position - ds
        Main.Position = UDim2.new(sp.X.Scale, sp.X.Offset + dl.X, sp.Y.Scale, sp.Y.Offset + dl.Y)
    end
end)

UIS.InputEnded:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then
        d = false
    end
end)

-- [ INIT ]
for _, p in pairs(Players:GetPlayers()) do
    if p ~= LocalPlayer then
        CreateESP(p)
    end
end

Players.PlayerAdded:Connect(function(p)
    CreateESP(p)
    RefreshWL()
end)

Players.PlayerRemoving:Connect(RefreshWL)

RefreshWL()
CombatPg.Visible = true

print("✅ ZURION v106 - STABLE HYBRID LOADED")
print("🎯 Auto-Aim | 🔫 Auto-Shot | 👁️ ESP | 🚀 Fly | 👻 Noclip | 👥 Whitelist")
