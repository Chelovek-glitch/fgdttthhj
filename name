local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local GuiService = game:GetService("GuiService")
local Workspace = game:GetService("Workspace")
local Camera = Workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

local Config = {
    SilentAim = {
        Enabled = true,
        FOV = 200,
        TeamCheck = true,
        WallCheck = false,
        HitPart = "Head",
        Prediction = 0.165,
        Keybind = Enum.KeyCode.E,
        HoldMode = true,
    },
    ESP = {
        Enabled = true,
        Box = true,
        Name = true,
        Health = true,
        Distance = true,
        Tracer = true,
        TeamCheck = true,
    },
    FOVCircle = {
        Enabled = true,
        Color = Color3.fromRGB(255, 255, 255),
        Transparency = 0.7,
        Thickness = 1,
    },
    Keybinds = {
        ToggleSilentAim = Enum.KeyCode.RightShift,
        ToggleESP = Enum.KeyCode.Insert,
    },
}

local function isAlive(plr)
    local char = plr.Character
    if not char then return false end
    local hum = char:FindFirstChildOfClass("Humanoid")
    return hum and hum.Health > 0
end

local function worldToScreen(pos)
    local sp, onScreen = Camera:WorldToViewportPoint(pos)
    return Vector2.new(sp.X, sp.Y), onScreen, sp.Z
end

local function getTargetPart(plr)
    if not plr.Character then return nil end
    return plr.Character:FindFirstChild(Config.SilentAim.HitPart)
        or plr.Character:FindFirstChild("HumanoidRootPart")
end

local function getMousePos()
    return Vector2.new(Mouse.X, Mouse.Y + GuiService:GetGuiInset().Y)
end

local function getTeamIdentity(plr)
    if plr.Team then
        return plr.Team.Name
    end
    return tostring(plr.TeamColor)
end

local function isTeammate(plr)
    if plr == LocalPlayer then return true end
    local myTeam = getTeamIdentity(LocalPlayer)
    local theirTeam = getTeamIdentity(plr)
    if myTeam and theirTeam then
        return myTeam == theirTeam
    end
    return false
end

local fovCircle = Drawing.new("Circle")
fovCircle.Color = Config.FOVCircle.Color
fovCircle.Thickness = Config.FOVCircle.Thickness
fovCircle.NumSides = 64
fovCircle.Filled = false
fovCircle.Transparency = Config.FOVCircle.Transparency
fovCircle.Visible = false

local function updateFOVCircle()
    fovCircle.Radius = Config.SilentAim.FOV
    fovCircle.Position = getMousePos()
    fovCircle.Color = Config.FOVCircle.Color
    fovCircle.Transparency = Config.FOVCircle.Transparency
    fovCircle.Visible = Config.FOVCircle.Enabled and Config.SilentAim.Enabled
end

local espCache = {}

local function createESP(plr)
    if plr == LocalPlayer then return end
    if espCache[plr] then return end

    local box = Drawing.new("Square")
    box.Thickness = 1
    box.Filled = false
    box.Transparency = 1
    box.Visible = false

    local name = Drawing.new("Text")
    name.Size = 14
    name.Center = true
    name.Outline = true
    name.Color = Color3.fromRGB(255, 255, 255)
    name.Visible = false

    local healthBar = Drawing.new("Line")
    healthBar.Thickness = 2
    healthBar.Color = Color3.fromRGB(0, 255, 0)
    healthBar.Visible = false

    local healthBg = Drawing.new("Line")
    healthBg.Thickness = 2
    healthBg.Color = Color3.fromRGB(0, 0, 0)
    healthBg.Visible = false

    local dist = Drawing.new("Text")
    dist.Size = 12
    dist.Center = true
    dist.Outline = true
    dist.Color = Color3.fromRGB(200, 200, 200)
    dist.Visible = false

    local tracer = Drawing.new("Line")
    tracer.Thickness = 1
    tracer.Color = Color3.fromRGB(255, 0, 0)
    tracer.Visible = false

    espCache[plr] = {
        Box = box, Name = name, Health = healthBar,
        HealthBg = healthBg, Distance = dist, Tracer = tracer,
    }
end

local function removeESP(plr)
    local cache = espCache[plr]
    if not cache then return end
    for _, obj in pairs(cache) do obj:Remove() end
    espCache[plr] = nil
end

for _, plr in ipairs(Players:GetPlayers()) do createESP(plr) end
Players.PlayerAdded:Connect(createESP)
Players.PlayerRemoving:Connect(removeESP)

local function updateESP()
    for plr, cache in pairs(espCache) do
        local char = plr.Character
        local head = char and char:FindFirstChild("Head")
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        local hum = char and char:FindFirstChildOfClass("Humanoid")

        local shouldShow = char and head and hrp and hum and hum.Health > 0
            and not (Config.ESP.TeamCheck and isTeammate(plr))
            and Config.ESP.Enabled

        if not shouldShow then
            for _, obj in pairs(cache) do obj.Visible = false end
            continue
        end

        local headPos, onScreen = worldToScreen(head.Position)
        local rootPos = worldToScreen(hrp.Position)

        if not onScreen then
            for _, obj in pairs(cache) do obj.Visible = false end
            continue
        end

        local height = math.abs(headPos.Y - rootPos.Y) * 2
        local width = height * 0.55

        if Config.ESP.Box then
            cache.Box.Size = Vector2.new(width, height)
            cache.Box.Position = Vector2.new(headPos.X - width / 2, headPos.Y - height / 4)
            cache.Box.Color = plr.Team and plr.Team.TeamColor.Color or Color3.fromRGB(255, 255, 255)
            cache.Box.Visible = true
        else cache.Box.Visible = false end

        if Config.ESP.Name then
            cache.Name.Text = plr.Name
            cache.Name.Position = Vector2.new(headPos.X, headPos.Y - 20)
            cache.Name.Visible = true
        else cache.Name.Visible = false end

        if Config.ESP.Health and hum then
            local hp = hum.Health / hum.MaxHealth
            local hX = headPos.X - width / 2 - 6
            local hTop = headPos.Y - height / 4
            local hBot = hTop + height
            cache.HealthBg.From = Vector2.new(hX, hTop)
            cache.HealthBg.To = Vector2.new(hX, hBot)
            cache.HealthBg.Visible = true
            cache.Health.From = Vector2.new(hX, hBot)
            cache.Health.To = Vector2.new(hX, hBot - height * hp)
            cache.Health.Color = Color3.fromRGB(255 * (1 - hp), 255 * hp, 0)
            cache.Health.Visible = true
        else
            cache.Health.Visible = false
            cache.HealthBg.Visible = false
        end

        if Config.ESP.Distance then
            local d = (Camera.CFrame.Position - hrp.Position).Magnitude
            cache.Distance.Text = string.format("[%d]", math.floor(d))
            cache.Distance.Position = Vector2.new(headPos.X, rootPos.Y + height / 2 + 4)
            cache.Distance.Visible = true
        else cache.Distance.Visible = false end

        if Config.ESP.Tracer then
            cache.Tracer.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
            cache.Tracer.To = Vector2.new(headPos.X, headPos.Y)
            cache.Tracer.Visible = true
        else cache.Tracer.Visible = false end
    end
end

local function getClosestTarget()
    local closest, shortest = nil, Config.SilentAim.FOV
    local mousePos = getMousePos()

    for _, plr in ipairs(Players:GetPlayers()) do
        if plr == LocalPlayer then continue end
        if not isAlive(plr) then continue end
        if Config.SilentAim.TeamCheck and isTeammate(plr) then continue end

        local part = getTargetPart(plr)
        if not part then continue end

        local screenPos, onScreen = worldToScreen(part.Position)
        if not onScreen then continue end

        local dist = (screenPos - mousePos).Magnitude
        if dist < shortest then
            shortest = dist
            closest = plr
        end
    end

    return closest
end

local function getPredictedPosition(part)
    return part.Position + (part.Velocity * Config.SilentAim.Prediction)
end

local function isSilentAimActive()
    if not Config.SilentAim.Enabled then return false end
    if Config.SilentAim.HoldMode and Config.SilentAim.Keybind then
        return UserInputService:IsKeyDown(Config.SilentAim.Keybind)
    end
    return true
end

local oldIndex
oldIndex = hookmetamethod(game, "__index", newcclosure(function(self, key)
    if not checkcaller() and isSilentAimActive() and
       (key == "Hit" or key == "Target") and self == Mouse then
        local target = getClosestTarget()
        if target then
            local part = getTargetPart(target)
            if part then
                if key == "Hit" then
                    return CFrame.new(getPredictedPosition(part))
                else
                    return part
                end
            end
        end
    end
    return oldIndex(self, key)
end))

local oldNamecall
oldNamecall = hookmetamethod(game, "__namecall", newcclosure(function(self, ...)
    local method = getnamecallmethod()
    if not checkcaller() and isSilentAimActive() then
        if method == "FindPartOnRay" or method == "FindPartOnRayWithIgnoreList"
           or method == "FindPartOnRayWithWhitelist" or method == "Raycast" then
            local target = getClosestTarget()
            if target then
                local part = getTargetPart(target)
                if part then
                    local args = {...}
                    local predicted = getPredictedPosition(part)
                    args[1] = Ray.new(Camera.CFrame.Position, (predicted - Camera.CFrame.Position).Unit * 1000)
                    return oldNamecall(self, unpack(args))
                end
            end
        end
    end
    return oldNamecall(self, ...)
end))

UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Config.Keybinds.ToggleSilentAim then
        Config.SilentAim.Enabled = not Config.SilentAim.Enabled
    elseif input.KeyCode == Config.Keybinds.ToggleESP then
        Config.ESP.Enabled = not Config.ESP.Enabled
        if not Config.ESP.Enabled then
            for _, cache in pairs(espCache) do
                for _, obj in pairs(cache) do obj.Visible = false end
            end
        end
    end
end)

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
    Name = "Criminality Suite — Infection",
    LoadingTitle = "Loading...",
    LoadingSubtitle = "by ZERO2",
    Theme = "Default",
    ConfigurationSaving = { Enabled = false },
    KeySystem = false,
})

local CombatTab = Window:CreateTab("Combat", 4483362458)
CombatTab:CreateSection("Silent Aim")

CombatTab:CreateToggle({
    Name = "Silent Aim",
    CurrentValue = Config.SilentAim.Enabled,
    Flag = "SilentAim",
    Callback = function(val) Config.SilentAim.Enabled = val end,
})

CombatTab:CreateSlider({
    Name = "FOV",
    Range = { 0, 500 },
    Increment = 10,
    Suffix = "studs",
    CurrentValue = Config.SilentAim.FOV,
    Flag = "FOV",
    Callback = function(val) Config.SilentAim.FOV = val end,
})

CombatTab:CreateToggle({
    Name = "Team Check",
    CurrentValue = Config.SilentAim.TeamCheck,
    Flag = "TeamCheck",
    Callback = function(val) Config.SilentAim.TeamCheck = val end,
})

CombatTab:CreateToggle({
    Name = "Hold to Aim",
    CurrentValue = Config.SilentAim.HoldMode,
    Flag = "HoldMode",
    Callback = function(val) Config.SilentAim.HoldMode = val end,
})

CombatTab:CreateSlider({
    Name = "Prediction",
    Range = { 0, 0.5 },
    Increment = 0.01,
    Suffix = "s",
    CurrentValue = Config.SilentAim.Prediction,
    Flag = "Prediction",
    Callback = function(val) Config.SilentAim.Prediction = val end,
})

CombatTab:CreateDropdown({
    Name = "Hit Part",
    Options = { "Head", "HumanoidRootPart", "UpperTorso", "LowerTorso" },
    CurrentOption = { Config.SilentAim.HitPart },
    Flag = "HitPart",
    Callback = function(opt) Config.SilentAim.HitPart = opt[1] end,
})

local VisualTab = Window:CreateTab("Visuals", 4483362458)
VisualTab:CreateSection("ESP")

VisualTab:CreateToggle({
    Name = "ESP Enabled",
    CurrentValue = Config.ESP.Enabled,
    Flag = "ESP",
    Callback = function(val) Config.ESP.Enabled = val end,
})

VisualTab:CreateToggle({
    Name = "Box",
    CurrentValue = Config.ESP.Box,
    Flag = "ESPBox",
    Callback = function(val) Config.ESP.Box = val end,
})

VisualTab:CreateToggle({
    Name = "Name",
    CurrentValue = Config.ESP.Name,
    Flag = "ESPName",
    Callback = function(val) Config.ESP.Name = val end,
})

VisualTab:CreateToggle({
    Name = "Health Bar",
    CurrentValue = Config.ESP.Health,
    Flag = "ESPHealth",
    Callback = function(val) Config.ESP.Health = val end,
})

VisualTab:CreateToggle({
    Name = "Distance",
    CurrentValue = Config.ESP.Distance,
    Flag = "ESPDistance",
    Callback = function(val) Config.ESP.Distance = val end,
})

VisualTab:CreateToggle({
    Name = "Tracer",
    CurrentValue = Config.ESP.Tracer,
    Flag = "ESPTracer",
    Callback = function(val) Config.ESP.Tracer = val end,
})

VisualTab:CreateToggle({
    Name = "ESP Team Check",
    CurrentValue = Config.ESP.TeamCheck,
    Flag = "ESPTeam",
    Callback = function(val) Config.ESP.TeamCheck = val end,
})

local SettingsTab = Window:CreateTab("Settings", 4483362458)
SettingsTab:CreateSection("FOV Circle")

SettingsTab:CreateToggle({
    Name = "Show FOV Circle",
    CurrentValue = Config.FOVCircle.Enabled,
    Flag = "FOVCircle",
    Callback = function(val) Config.FOVCircle.Enabled = val end,
})

SettingsTab:CreateColorPicker({
    Name = "FOV Color",
    Color = Config.FOVCircle.Color,
    Flag = "FOVColor",
    Callback = function(color) Config.FOVCircle.Color = color end,
})

SettingsTab:CreateSlider({
    Name = "FOV Transparency",
    Range = { 0, 1 },
    Increment = 0.1,
    CurrentValue = Config.FOVCircle.Transparency,
    Flag = "FOVTrans",
    Callback = function(val) Config.FOVCircle.Transparency = val end,
})

SettingsTab:CreateSection("Keybinds")
SettingsTab:CreateKeybind({
    Name = "Silent Aim Key (hold)",
    CurrentKeybind = Config.SilentAim.Keybind.Name,
    HoldToInteract = false,
    Flag = "SAKey",
    Callback = function(key) Config.SilentAim.Keybind = Enum.KeyCode[key] end,
})

SettingsTab:CreateKeybind({
    Name = "Toggle Silent Aim",
    CurrentKeybind = Config.Keybinds.ToggleSilentAim.Name,
    HoldToInteract = false,
    Flag = "SAToggle",
    Callback = function(key) Config.Keybinds.ToggleSilentAim = Enum.KeyCode[key] end,
})

SettingsTab:CreateKeybind({
    Name = "Toggle ESP",
    CurrentKeybind = Config.Keybinds.ToggleESP.Name,
    HoldToInteract = false,
    Flag = "ESPKey",
    Callback = function(key) Config.Keybinds.ToggleESP = Enum.KeyCode[key] end,
})

RunService.RenderStepped:Connect(function()
    pcall(updateFOVCircle)
    pcall(updateESP)
end)

Rayfield:Notify({
    Title = "Criminality Suite",
    Content = "Infection mode loaded.",
    Duration = 5,
})

print("[ZERO2] Infection suite loaded. 6767.")
