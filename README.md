--[[
    FFlag 优化 (提取自 message (11).txt 的 SPOOKY_FFLAGS)
    ------------------------------------------------------------------
    · 6 条 FastFlag
    · 额外: 240 FPS 上限 / 关阴影 / 去雾 / 画质 Level01
    · 开关可应用与还原
    面板可拖动。
]]

local Lighting    = game:GetService("Lighting")
local Players     = game:GetService("Players")
local UIS         = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

local FLAGS = {
    { "DFIntTaskSchedulerTargetFps", "240" },
    { "FFlagDisablePostFx",          "True" },
    { "FIntFRMMaxGrassDistance",     "0" },
    { "FIntFRMMinGrassDistance",     "0" },
    { "DebugFRMQualityLevelOverride","1" },
    { "TextureQualityOverride",      "0" },
}

-- ============================ 核心 ============================
local enabled = false
local originals = {}
local savedLighting = {}
local savedQuality = nil
local savedFps = nil

local function fastFlagSetter()
    return setfflag or set_fflag
end

local function fastFlagGetter()
    return getfflag or get_fflag
end

local function ApplyFlags()
    local setter = fastFlagSetter()
    if type(setter) ~= "function" then
        return 0
    end

    -- 保存光照 / 画质 / FPS
    savedLighting.GlobalShadows = Lighting.GlobalShadows
    savedLighting.FogEnd = Lighting.FogEnd
    pcall(function() savedQuality = settings().Rendering.QualityLevel end)
    if type(getfpscap) == "function" then
        local ok, v = pcall(getfpscap)
        if ok then savedFps = v end
    end

    -- 保存原 flag
    originals = {}
    local getter = fastFlagGetter()
    if type(getter) == "function" then
        for _, pair in ipairs(FLAGS) do
            local ok, v = pcall(getter, pair[1])
            if ok and v ~= nil then
                originals[pair[1]] = v
            end
        end
    end

    Lighting.GlobalShadows = false
    Lighting.FogEnd = 9e9
    pcall(function() settings().Rendering.QualityLevel = Enum.QualityLevel.Level01 end)
    if type(setfpscap) == "function" then pcall(setfpscap, 240) end

    local applied = 0
    for _, pair in ipairs(FLAGS) do
        local ok, result = pcall(setter, pair[1], tostring(pair[2]))
        if ok and result ~= false then
            applied = applied + 1
        end
    end
    return applied
end

local function RestoreFlags()
    local setter = fastFlagSetter()
    if type(setter) == "function" then
        for name, value in pairs(originals) do
            pcall(setter, name, tostring(value))
        end
    end
    originals = {}

    if savedLighting.GlobalShadows ~= nil then
        Lighting.GlobalShadows = savedLighting.GlobalShadows
    end
    if savedLighting.FogEnd ~= nil then
        Lighting.FogEnd = savedLighting.FogEnd
    end
    if savedQuality ~= nil then
        pcall(function() settings().Rendering.QualityLevel = savedQuality end)
    end
    if savedFps ~= nil and type(setfpscap) == "function" then
        pcall(setfpscap, savedFps)
    end
end

local function SetEnabled(on)
    enabled = on
    if on then
        return ApplyFlags()
    else
        RestoreFlags()
        return 0
    end
end

-- ============================ UI ============================
local ACCENT   = Color3.fromRGB(255, 190, 90)
local TXT      = Color3.fromRGB(236, 240, 250)
local TXT_SUB  = Color3.fromRGB(150, 158, 180)
local CARD     = Color3.fromRGB(44, 49, 64)
local PANEL_BG = Color3.fromRGB(26, 30, 42)
local GREEN    = Color3.fromRGB(70, 200, 130)

local parent = LocalPlayer:WaitForChild("PlayerGui")
local old = parent:FindFirstChild("FFlagGui")
if old then old:Destroy() end

local gui = Instance.new("ScreenGui")
gui.Name = "FFlagGui"
gui.ResetOnSpawn = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = parent

local panel = Instance.new("Frame")
panel.Size = UDim2.fromOffset(220, 118)
panel.Position = UDim2.fromOffset(60, 140)
panel.BackgroundColor3 = PANEL_BG
panel.BorderSizePixel = 0
panel.Active = true
panel.Parent = gui
Instance.new("UICorner", panel).CornerRadius = UDim.new(0, 14)
local stroke = Instance.new("UIStroke")
stroke.Color = Color3.fromRGB(60, 68, 90); stroke.Transparency = 0.3; stroke.Parent = panel

local bar = Instance.new("Frame")
bar.Size = UDim2.new(1, 0, 0, 4); bar.BackgroundColor3 = ACCENT
bar.BorderSizePixel = 0; bar.Parent = panel
Instance.new("UICorner", bar).CornerRadius = UDim.new(0, 14)

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -40, 0, 28); title.Position = UDim2.fromOffset(14, 8)
title.BackgroundTransparency = 1; title.Font = Enum.Font.GothamBold; title.TextSize = 14
title.TextColor3 = TXT; title.TextXAlignment = Enum.TextXAlignment.Left
title.Text = "FFlag 优化"; title.Parent = panel

local close = Instance.new("TextButton")
close.Size = UDim2.fromOffset(20, 20); close.Position = UDim2.new(1, -28, 0, 12)
close.BackgroundColor3 = CARD; close.BorderSizePixel = 0; close.Text = "×"
close.Font = Enum.Font.GothamBold; close.TextSize = 14; close.TextColor3 = TXT_SUB
close.Parent = panel
Instance.new("UICorner", close).CornerRadius = UDim.new(1, 0)
close.MouseButton1Click:Connect(function() gui.Enabled = false end)

-- 开关
local track = Instance.new("TextButton")
track.Size = UDim2.fromOffset(44, 22); track.Position = UDim2.fromOffset(14, 46)
track.BackgroundColor3 = Color3.fromRGB(70, 76, 94)
track.BorderSizePixel = 0; track.Text = ""; track.AutoButtonColor = false
track.Parent = panel
Instance.new("UICorner", track).CornerRadius = UDim.new(1, 0)
local knob = Instance.new("Frame")
knob.Size = UDim2.fromOffset(18, 18); knob.Position = UDim2.fromOffset(2, 2)
knob.BackgroundColor3 = Color3.fromRGB(255, 255, 255); knob.BorderSizePixel = 0
knob.Parent = track
Instance.new("UICorner", knob).CornerRadius = UDim.new(1, 0)

local stateLabel = Instance.new("TextLabel")
stateLabel.Size = UDim2.fromOffset(140, 20); stateLabel.Position = UDim2.fromOffset(66, 47)
stateLabel.BackgroundTransparency = 1; stateLabel.Font = Enum.Font.GothamMedium
stateLabel.TextSize = 12; stateLabel.TextColor3 = TXT; stateLabel.TextXAlignment = Enum.TextXAlignment.Left
stateLabel.Text = "已关闭"; stateLabel.Parent = panel

local status = Instance.new("TextLabel")
status.Size = UDim2.new(1, -28, 0, 30); status.Position = UDim2.fromOffset(14, 76)
status.BackgroundTransparency = 1; status.Font = Enum.Font.Code; status.TextSize = 11
status.TextColor3 = TXT_SUB; status.TextXAlignment = Enum.TextXAlignment.Left
status.TextWrapped = true; status.TextYAlignment = Enum.TextYAlignment.Top
status.Text = "6 条 FFlag | 240 FPS | Level01"; status.Parent = panel

local function refresh(on)
    track.BackgroundColor3 = on and GREEN or Color3.fromRGB(70, 76, 94)
    knob.Position = on and UDim2.new(1, -20, 0, 2) or UDim2.fromOffset(2, 2)
    stateLabel.Text = on and "已开启" or "已关闭"
end

track.MouseButton1Click:Connect(function()
    local on = not enabled
    local applied = SetEnabled(on)
    if on and applied == 0 and type(fastFlagSetter()) ~= "function" then
        status.Text = "执行器不支持 setfflag"
        status.TextColor3 = Color3.fromRGB(240, 150, 150)
    else
        status.TextColor3 = TXT_SUB
        if on then
            status.Text = string.format("已应用 %d/%d 条 FFlag", applied, #FLAGS)
        else
            status.Text = "已还原（部分 flag 需重进生效）"
        end
    end
    refresh(on)
end)

-- 拖动
local drag, dragStart, startPos
title.InputBegan:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then
        drag = true; dragStart = i.Position; startPos = panel.Position
    end
end)
UIS.InputChanged:Connect(function(i)
    if drag and i.UserInputType == Enum.UserInputType.MouseMovement then
        local d = i.Position - dragStart
        panel.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X,
                                   startPos.Y.Scale, startPos.Y.Offset + d.Y)
    end
end)
UIS.InputEnded:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then drag = false end
end)

refresh(false)腦子不行
