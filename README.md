-- ==================== OBSIDIAN UI ====================
local repo = "https://raw.githubusercontent.com/ApparentlyZen/Obsidian-Zen/main/"
local Library = loadstring(game:HttpGet(repo .. "Library.lua"))()
local ThemeManager = loadstring(game:HttpGet(repo .. "addons/ThemeManager.lua"))()
local SaveManager = loadstring(game:HttpGet(repo .. "addons/SaveManager.lua"))()

local Options = Library.Options
local Toggles = Library.Toggles

if not isfolder("NamelessConfigs") then makefolder("NamelessConfigs") end
if not isfolder("NamelessConfigs/Fonts") then makefolder("NamelessConfigs/Fonts") end

local Window = Library:CreateWindow({
    Title = "NamelessWare",
    Footer = "lumexa on top",
    ShowCustomCursor = true,
    NotifySide = "Right",
    Size = UDim2.new(0, 600, 0, 460),
})

task.delay(0.2, function()
    local CoreGui = game:GetService("CoreGui")
    local Players = game:GetService("Players")
    local lp = Players.LocalPlayer

    local function applyLogo()
        local titleLabel = nil
        local containers = { CoreGui, lp:FindFirstChild("PlayerGui") }
        
        for _, parent in ipairs(containers) do
            if parent then
                for _, descendant in ipairs(parent:GetDescendants()) do
                    if descendant:IsA("TextLabel") and (descendant.Text == "NamelessWare" or descendant.Text:find("Nameless")) then
                        if descendant.Parent and descendant.Parent:FindFirstChild("NamelessLogoIcon") then
                            return true
                        end
                        titleLabel = descendant
                        break
                    end
                end
            end
            if titleLabel then break end
        end

        if titleLabel then
            titleLabel.RichText = true
            titleLabel.Text = '<font color="#ffffff">Nameless</font><font color="#a050ff">Ware</font>'
            
            local titleParent = titleLabel.Parent
            if titleParent then
                local existingLogo = titleParent:FindFirstChild("NamelessLogoIcon")
                if existingLogo then existingLogo:Destroy() end

                local logoImg = Instance.new("ImageButton")
                logoImg.Name = "NamelessLogoIcon"
                logoImg.BackgroundTransparency = 1
                logoImg.Size = UDim2.new(0, 22, 0, 22)
                logoImg.Image = "rbxassetid://105243902490842"
                logoImg.ScaleType = Enum.ScaleType.Fit
                logoImg.AutoButtonColor = false
                
                local UICornerLogo = Instance.new("UICorner")
                UICornerLogo.CornerRadius = UDim.new(0, 4)
                UICornerLogo.Parent = logoImg

                local listLayout = titleParent:FindFirstChildOfClass("UIListLayout")
                if listLayout then
                    logoImg.LayoutOrder = 0
                    titleLabel.LayoutOrder = 1
                else
                    logoImg.Position = UDim2.new(0, 8, 0, (titleLabel.Position.Y.Offset > 0 and titleLabel.Position.Y.Offset or 6))
                    titleLabel.Position = UDim2.new(0, 36, 0, titleLabel.Position.Y.Offset)
                end
                
                logoImg.Parent = titleParent

                local windowFrame = titleParent
                while windowFrame and windowFrame.Parent and not windowFrame.Parent:IsA("ScreenGui") do
                    windowFrame = windowFrame.Parent
                end
                
                local screenGui = windowFrame and windowFrame.Parent
                local toggleBtn = nil
                
                if screenGui then
                    toggleBtn = screenGui:FindFirstChild("NamelessToggleIcon")
                    if not toggleBtn then
                        toggleBtn = Instance.new("ImageButton")
                        toggleBtn.Name = "NamelessToggleIcon"
                        toggleBtn.Size = UDim2.new(0, 42, 0, 42)
                        toggleBtn.Position = UDim2.new(0.05, 0, 0.15, 0)
                        toggleBtn.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
                        toggleBtn.BackgroundTransparency = 0.1
                        toggleBtn.Image = "rbxassetid://105243902490842"
                        toggleBtn.ScaleType = Enum.ScaleType.Fit
                        toggleBtn.Visible = false
                        toggleBtn.Active = true
                        toggleBtn.Draggable = true
                        toggleBtn.Parent = screenGui

                        local corner = Instance.new("UICorner")
                        corner.CornerRadius = UDim.new(0, 10)
                        corner.Parent = toggleBtn

                        local stroke = Instance.new("UIStroke")
                        stroke.Color = Color3.fromRGB(160, 80, 255)
                        stroke.Thickness = 2
                        stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
                        stroke.Parent = toggleBtn
                        
                        toggleBtn.MouseButton1Click:Connect(function()
                            toggleBtn.Visible = false
                            if windowFrame then windowFrame.Visible = true end
                        end)
                    end
                end

                logoImg.MouseButton1Click:Connect(function()
                    if windowFrame then
                        if toggleBtn then
                            toggleBtn.Position = UDim2.new(
                                windowFrame.Position.X.Scale,
                                windowFrame.Position.X.Offset + 10,
                                windowFrame.Position.Y.Scale,
                                windowFrame.Position.Y.Offset + 10
                            )
                            toggleBtn.Visible = true
                        end
                        windowFrame.Visible = false
                    end
                end)
                return true
            end
        end
        return false
    end

    for i = 1, 5 do
        if applyLogo() then break end
        task.wait(0.3)
    end
end)

local Tabs = {
    Macro       = Window:AddTab("Macro", "keyboard"),
    SilentAim   = Window:AddTab("Silent Aim", "crosshair"),
    SABlacklist = Window:AddTab("SA Blacklist", "ban"),
    Glitch      = Window:AddTab("Glitch", "zap"),
    Movement    = Window:AddTab("Movement", "move"),
    Apparence   = Window:AddTab("Appearance", "user"),
    Misc        = Window:AddTab("Misc", "box"),
    ESP         = Window:AddTab("ESP", "eye"),
    Settings    = Window:AddTab("Settings", "settings"),
}

-- ==================== SERVICES ====================
local Players = game:GetService("Players")
local lp = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local camera = workspace.CurrentCamera
local RS = game:GetService("ReplicatedStorage")

local SilentAimModule

-- ==================== VSKILL & DAMAGE STATE ====================
local sharkZActive, vActive, cursedZActive = false, false, false
local rightTouchActive = false
local damageDetected = false
local isBunnyHopping = false

-- ==================== LOADERS AHK ====================
local ahk_soru_loaded = false
local function UnloadAHKsoru()
    ahk_soru_loaded = false
    _G.NamelessAHKRunning = nil
    pcall(function()
        local containers = {lp:FindFirstChild("PlayerGui"), game:GetService("CoreGui")}
        for _, parent in ipairs(containers) do
            if parent then
                for _, child in pairs(parent:GetChildren()) do
                    if child.Name:find("AHK_Soru") or child.Name:find("namelessWare_AHK_Soru") then
                        child:Destroy()
                    end
                end
            end
        end
    end)
    Library:Notify({ Title = "AHK Soru", Description = "Script disabled!", Time = 3 })
end

local function LoadAHKsoru()
    _G.NamelessAHKRunning = nil
    ahk_soru_loaded = false
    local success, err = pcall(function()
        local script_code = game:HttpGet("https://raw.githubusercontent.com/sykq0/Namelesssoru/refs/heads/main/ahk.lua")
        loadstring(script_code)()
    end)
    if success then
        ahk_soru_loaded = true
        Library:Notify({ Title = "AHK Soru", Description = "Script loaded!", Time = 3 })
    else
        Library:Notify({ Title = "AHK Soru", Description = "Loading error.", Time = 3 })
    end
end

local ahk_combo_script_loaded = false
local function UnloadAHKComboScript()
    ahk_combo_script_loaded = false
    _G.NamelessComboRunning = nil
    pcall(function()
        local containers = {lp:FindFirstChild("PlayerGui"), game:GetService("CoreGui")}
        for _, parent in ipairs(containers) do
            if parent then
                for _, child in pairs(parent:GetChildren()) do
                    if child.Name:find("AHK_Combo") or child.Name:find("namelessWare_AHK_Combo") then
                        child:Destroy()
                    end
                end
            end
        end
    end)
    Library:Notify({ Title = "AHK Combo", Description = "Script disabled!", Time = 3 })
end

local function LoadAHKComboScript()
    _G.NamelessComboRunning = nil
    ahk_combo_script_loaded = false
    local success, err = pcall(function()
        local script_code = game:HttpGet("https://raw.githubusercontent.com/sykq0/Namelesssoru/refs/heads/main/fetched-1.lua")
        loadstring(script_code)()
    end)
    if success then
        ahk_combo_script_loaded = true
        Library:Notify({ Title = "AHK Combo", Description = "Script loaded!", Time = 3 })
    else
        Library:Notify({ Title = "AHK Combo", Description = "Loading error.", Time = 3 })
    end
end



-- ==================== DAMAGE MONITOR ====================
local function watchDamageCounter()
    task.spawn(function()
        while true do
            local gui = lp:FindFirstChild("PlayerGui") and lp.PlayerGui:FindFirstChild("Main")
            local dmgCounter = gui and gui:FindFirstChild("DmgCounter")
            local dmgTextLabel = dmgCounter and dmgCounter:FindFirstChild("Text")
            
            if dmgTextLabel then
                dmgTextLabel:GetPropertyChangedSignal("Text"):Connect(function()
                    local dmgText = tonumber(dmgTextLabel.Text) or 0
                    damageDetected = (dmgText > 0)
                end)
                break
            end
            task.wait(1)
        end
    end)
end
watchDamageCounter()

local function isValidStopCondition()
    local tool = lp.Character and lp.Character:FindFirstChildOfClass("Tool")
    return (tool and tool.Name == "Shark Anchor" and sharkZActive)
        or (vActive)
        or (tool and tool.Name == "Cursed Dual Katana" and cursedZActive)
end

-- ==================== BOOSTS (Sanguine, Diamond, EClaw) ====================
local multiEnabled = false; local multiPower = 400; local multiDuration = 0.9
local multiCharging = false; local multiChargeStart = 0; local multiRequiredCharge = 1.0

local diamondEnabled = false; local diamondPower = 250; local diamondDuration = 0.3
local diamondCharging = false; local diamondChargeStart = 0; local diamondRequiredCharge = 1.0

local dtalonEnabled = false; local dtalonPower = 400; local dtalonDuration = 0.9
local dtalonCharging = false; local dtalonChargeStart = 0; local dtalonRequiredCharge = 1.0

local EClawBoost

local function MegaBoost()
    local char = lp.Character; local hrp = char and char:FindFirstChild("HumanoidRootPart"); local hum = char and char:FindFirstChild("Humanoid")
    if not hrp or not hum then return end
    
    local dir = camera.CFrame.LookVector

    hum.PlatformStand = true
    local att = Instance.new("Attachment", hrp); local lv = Instance.new("LinearVelocity", hrp)
    lv.MaxForce = 9999999; lv.VelocityConstraintMode = Enum.VelocityConstraintMode.Vector; lv.VectorVelocity = dir * multiPower; lv.Attachment0 = att
    local boostActive = true; local boostEndTime = tick() + multiDuration; local dmgConn
    local function stopPropulsion()
        if not boostActive then return end; boostActive = false
        if lv then lv:Destroy() end; if att then att:Destroy() end; if hum then hum.PlatformStand = false end; if dmgConn then dmgConn:Disconnect() end
    end
    local dmgLabel = lp:FindFirstChild("PlayerGui") and lp.PlayerGui:FindFirstChild("Main") and lp.PlayerGui.Main:FindFirstChild("DmgCounter") and lp.PlayerGui.Main.DmgCounter:FindFirstChild("Text")
    local startText = dmgLabel and dmgLabel.Text or ""
    if dmgLabel then
        dmgConn = dmgLabel:GetPropertyChangedSignal("Text"):Connect(function()
            local currentText = dmgLabel.Text
            if (currentText ~= startText and currentText ~= "" and currentText ~= "0") or (rightTouchActive and isValidStopCondition()) then
                stopPropulsion()
            end
        end)
    end
    while boostActive and tick() < boostEndTime do task.wait() end; stopPropulsion()
end

RunService.Heartbeat:Connect(function()
    if not multiEnabled then return end
    local char = lp.Character; local hum = char and char:FindFirstChild("Humanoid"); if not hum then return end
    local isGC = false
    for _, a in pairs(hum:GetPlayingAnimationTracks()) do
        if a.Animation.AnimationId:find("14418367908") or a.Name == "GhoulZCharge" then isGC = true; break end
    end
    if isGC then if not multiCharging then multiCharging = true; multiChargeStart = tick() end
    else if multiCharging then if (tick() - multiChargeStart) >= multiRequiredCharge then task.spawn(MegaBoost) end; multiCharging = false; multiChargeStart = 0 end end
end)

local function DiamondBoost()
    local char = lp.Character; local hrp = char and char:FindFirstChild("HumanoidRootPart"); local hum = char and char:FindFirstChild("Humanoid")
    if not hrp or not hum then return end

    local dir = camera.CFrame.LookVector

    hum.PlatformStand = true
    local att = Instance.new("Attachment", hrp); local lv = Instance.new("LinearVelocity", hrp)
    lv.MaxForce = 9999999; lv.VelocityConstraintMode = Enum.VelocityConstraintMode.Vector; lv.VectorVelocity = dir * diamondPower; lv.Attachment0 = att
    local boostActive = true; local boostEndTime = tick() + diamondDuration; local dmgConn
    local function stopPropulsion()
        if not boostActive then return end; boostActive = false
        if lv then lv:Destroy() end; if att then att:Destroy() end; if hum then hum.PlatformStand = false end; if dmgConn then dmgConn:Disconnect() end
    end
    local dmgLabel = lp:FindFirstChild("PlayerGui") and lp.PlayerGui:FindFirstChild("Main") and lp.PlayerGui.Main:FindFirstChild("DmgCounter") and lp.PlayerGui.Main.DmgCounter:FindFirstChild("Text")
    local startText = dmgLabel and dmgLabel.Text or ""
    if dmgLabel then
        dmgConn = dmgLabel:GetPropertyChangedSignal("Text"):Connect(function()
            local currentText = dmgLabel.Text
            if (currentText ~= startText and currentText ~= "" and currentText ~= "0") or (rightTouchActive and isValidStopCondition()) then
                stopPropulsion()
            end
        end)
    end
    while boostActive and tick() < boostEndTime do task.wait() end; stopPropulsion()
end

RunService.Heartbeat:Connect(function()
    if not diamondEnabled then return end
    local char = lp.Character; local hum = char and char:FindFirstChild("Humanoid"); if not hum then return end
    local isC = false
    for _, a in pairs(hum:GetPlayingAnimationTracks()) do
        if a.Animation.AnimationId:find("14414815375") then isC = true; break end
    end
    if isC then if not diamondCharging then diamondCharging = true; diamondChargeStart = tick() end
    else if diamondCharging then if (tick() - diamondChargeStart) >= diamondRequiredCharge then task.spawn(DiamondBoost) end; diamondCharging = false; diamondChargeStart = 0 end end
end)

local function DTalonBoost()
    local char = lp.Character; local hrp = char and char:FindFirstChild("HumanoidRootPart"); local hum = char and char:FindFirstChild("Humanoid")
    if not hrp or not hum then return end

    local dir = camera.CFrame.LookVector

    hum.PlatformStand = true
    local att = Instance.new("Attachment", hrp); local lv = Instance.new("LinearVelocity", hrp)
    lv.MaxForce = 9999999; lv.VelocityConstraintMode = Enum.VelocityConstraintMode.Vector; lv.VectorVelocity = dir * dtalonPower; lv.Attachment0 = att
    local boostActive = true; local boostEndTime = tick() + dtalonDuration; local dmgConn
    local function stopPropulsion()
        if not boostActive then return end; boostActive = false
        if lv then lv:Destroy() end; if att then att:Destroy() end; if hum then hum.PlatformStand = false end; if dmgConn then dmgConn:Disconnect() end
    end
    local dmgLabel = lp:FindFirstChild("PlayerGui") and lp.PlayerGui:FindFirstChild("Main") and lp.PlayerGui.Main:FindFirstChild("DmgCounter") and lp.PlayerGui.Main.DmgCounter:FindFirstChild("Text")
    local startText = dmgLabel and dmgLabel.Text or ""
    if dmgLabel then
        dmgConn = dmgLabel:GetPropertyChangedSignal("Text"):Connect(function()
            local currentText = dmgLabel.Text
            if (currentText ~= startText and currentText ~= "" and currentText ~= "0") or (rightTouchActive and isValidStopCondition()) then
                stopPropulsion()
            end
        end)
    end
    while boostActive and tick() < boostEndTime do task.wait() end; stopPropulsion()
end

RunService.Heartbeat:Connect(function()
    if not dtalonEnabled then return end
    local char = lp.Character; local hum = char and char:FindFirstChild("Humanoid"); if not hum then return end
    local isC = false
    for _, a in pairs(hum:GetPlayingAnimationTracks()) do
        if a.Animation.AnimationId:find("18839089313") or a.Name == "DTalon_ZCharge" then isC = true; break end
    end
    if isC then if not dtalonCharging then dtalonCharging = true; dtalonChargeStart = tick() end
    else if dtalonCharging then if (tick() - dtalonChargeStart) >= dtalonRequiredCharge then task.spawn(DTalonBoost) end; dtalonCharging = false; dtalonChargeStart = 0 end end
end)

EClawBoost = (function()
    local M = {}; local enabled = false; local power = 400; local duration = 1.7; local requiredCharge = 0.0; local boosted = false
    local function boost()
        local char = lp.Character; local hrp = char and char:FindFirstChild("HumanoidRootPart"); local hum = char and char:FindFirstChildOfClass("Humanoid")
        if not hrp or not hum then return end; local mouse = lp:GetMouse(); local dir = (mouse.Hit.p - hrp.Position).Unit
        hum.PlatformStand = true; local att = Instance.new("Attachment", hrp); local lv = Instance.new("LinearVelocity", hrp)
        lv.MaxForce = 9999999; lv.VelocityConstraintMode = Enum.VelocityConstraintMode.Vector; lv.VectorVelocity = dir * power; lv.Attachment0 = att
        local boostActive = true; local boostEndTime = tick() + duration; local dmgConn
        local function stopPropulsion()
            if not boostActive then return end; boostActive = false
            if lv then lv:Destroy() end; if att then att:Destroy() end; if hum then hum.PlatformStand = false end; if dmgConn then dmgConn:Disconnect() end
        end
        local dmgLabel = lp:FindFirstChild("PlayerGui") and lp.PlayerGui:FindFirstChild("Main") and lp.PlayerGui.Main:FindFirstChild("DmgCounter") and lp.PlayerGui.Main.DmgCounter:FindFirstChild("Text")
        local startText = dmgLabel and dmgLabel.Text or ""
        if dmgLabel then
            dmgConn = dmgLabel:GetPropertyChangedSignal("Text"):Connect(function()
                local currentText = dmgLabel.Text
                if (currentText ~= startText and currentText ~= "" and currentText ~= "0") or (rightTouchActive and isValidStopCondition()) then
                    stopPropulsion()
                end
            end)
        end
        while boostActive an