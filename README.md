-- YEW FREE Hub - Blade Ball V1.21
-- Password: 018828772928392
-- 自动格挡优化 | 球速实时检测 | 手机/PC自适应

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local CoreGui = game:GetService("CoreGui")
local Workspace = game:GetService("Workspace")
local Camera = Workspace.CurrentCamera

repeat task.wait() until LocalPlayer and LocalPlayer.Character

-- ============ Device Detection ============
local isMobile = UserInputService.TouchEnabled and not UserInputService.MouseEnabled
print(isMobile and "📱 Mobile Mode" or "💻 PC Mode")

-- ============ Create Main GUI ============
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "YEW_Free_Hub"
screenGui.ResetOnSpawn = false
pcall(function() screenGui.Parent = CoreGui end)
if not screenGui.Parent then
    pcall(function() screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui") end)
end

local guiScale = isMobile and 0.8 or 1

-- ============ Wait for Balls Folder ============
local Balls = workspace:WaitForChild("Balls", 9e9)
print("✅ Balls folder found")

-- ============ Verify Ball ============
local function VerifyBall(Ball)
    if typeof(Ball) == "Instance" and Ball:IsA("BasePart") and Ball:IsDescendantOf(Balls) then
        if Ball:GetAttribute("realBall") == true then
            return true
        end
    end
    return false
end

-- ============ Check if Targeted ============
local function IsTarget()
    return (LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Highlight"))
end

-- ============ Variables ============
local autoParry = false
local autoSpam = false
local manualSpam = false
local ballStats = false
local parryVisualiser = false
local playerESP = false
local abilityESP = false
local rainbowSky = false
local fullbright = false
local noFog = false
local darkMode = false
local antiLag = false
local thunderDash = false
local highJump = false
local speedBoost = false
local customSky = false
local muteSounds = false
local flyMode = false
local immortal = false
local skinChanger = false
local playerSkinChanger = false

local currentSpeed = 50
local currentJump = 50
local volumeLevel = 1
local colorHue = 0
local selectedTheme = "Default"
local selectedSword = ""
local selectedPlayerSkin = "Default"
local flyBodyVelocity = nil
local flyBodyGyro = nil

-- ============ Ball Speed Detection ============
local currentBall = nil
local currentBallSpeed = 0
local maxBallSpeed = 0

local function findRealBall()
    for _, ball in pairs(Balls:GetChildren()) do
        if VerifyBall(ball) then
            return ball
        end
    end
    return nil
end

currentBall = findRealBall()

-- 实时球速检测 (每帧更新)
task.spawn(function()
    while true do
        task.wait(0.05)
        if currentBall and currentBall.Parent then
            if currentBall.AssemblyLinearVelocity then
                currentBallSpeed = currentBall.AssemblyLinearVelocity.Magnitude
                if currentBallSpeed > maxBallSpeed then
                    maxBallSpeed = currentBallSpeed
                end
            end
        else
            currentBall = findRealBall()
            currentBallSpeed = 0
        end
    end
end)

-- 监听新球出现
Balls.ChildAdded:Connect(function(Ball)
    if VerifyBall(Ball) then
        currentBall = Ball
        print("✅ Real ball detected!")
    end
end)

-- ============ Get Nearby Players ============
local function getNearbyPlayerCount()
    local count = 0
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return 0 end
    local myPos = char.HumanoidRootPart.Position
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            if (plr.Character.HumanoidRootPart.Position - myPos).Magnitude < 30 then
                count = count + 1
            end
        end
    end
    return count
end

-- ============ Parry Function ============
local function sendParry()
    if isMobile then
        pcall(function()
            local vu = game:GetService("VirtualUser")
            vu:Button1Down(Vector2.new(0,0))
            task.wait(0.02)
            vu:Button1Up(Vector2.new(0,0))
        end)
    else
        pcall(function()
            VirtualInputManager:SendMouseButtonEvent(0, 0, 0, true, game, 0)
            task.wait(0.02)
            VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 0)
        end)
    end
    
    pcall(function()
        local remotes = {"Parry", "Block", "Ability", "ParryEvent", "BlockEvent", "UseAbility", "Activate"}
        for _, name in ipairs(remotes) do
            local remote = ReplicatedStorage:FindFirstChild(name)
            if remote then remote:FireServer() end
        end
    end)
end

-- ============ Auto Parry (到达时间预测 + 球速分级) ============
local parryConnections = {}

local function setupParryForBall(Ball)
    if parryConnections[Ball] then return end
    
    local connection = RunService.Heartbeat:Connect(function()
        if not autoParry then return end
        if not IsTarget() then return end
        
        local char = LocalPlayer.Character
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        if not hrp then return end
        
        local distance = (Ball.Position - hrp.Position).Magnitude
        local speed = currentBallSpeed
        
        if speed > 0 then
            -- 到达时间 = 距离 / 速度
            local timeToImpact = distance / speed
            
            -- 动态阈值：球速越快，阈值越小（更快反应）
            local threshold = 0.4
            if speed > 250 then
                threshold = 0.2
            elseif speed > 200 then
                threshold = 0.25
            elseif speed > 150 then
                threshold = 0.3
            elseif speed > 100 then
                threshold = 0.35
            elseif speed > 60 then
                threshold = 0.38
            end
            
            -- 格挡距离限制 (避免太远就格挡)
            if distance < 45 and timeToImpact <= threshold then
                sendParry()
            end
        end
    end)
    
    parryConnections[Ball] = connection
end

-- Setup for existing balls
for _, ball in pairs(Balls:GetChildren()) do
    if VerifyBall(ball) then
        setupParryForBall(ball)
    end
end

Balls.ChildAdded:Connect(function(Ball)
    if VerifyBall(Ball) then
        setupParryForBall(Ball)
    end
end)

-- ============ Auto Spam (自动连点) ============
local autoSpamActive = false
local autoSpamThread = nil

local function startAutoSpam()
    if autoSpamThread then return end
    autoSpamActive = true
    autoSpamThread = task.spawn(function()
        while autoSpamActive do
            sendParry()
            task.wait(0.05)
        end
    end)
end

local function stopAutoSpam()
    autoSpamActive = false
    if autoSpamThread then
        task.cancel(autoSpamThread)
        autoSpamThread = nil
    end
end

-- ============ YEW SPAM Functions ============
local spamRemote = nil
local spamRawFire = nil
local spamArgs = {}
local spamHooked = false
local spamPacketActivated = false
local spamPacketThread = nil

local function hookParryPacket()
    if spamHooked then return end
    local success, mt = pcall(getrawmetatable, game)
    if not success then return end
    
    local oldIndex = mt.__index
    setreadonly(mt, false)
    mt.__index = function(self, key)
        if key == "FireServer" then
            return function(obj, ...)
                local args = {...}
                if #args == 7 and typeof(args[4]) == "CFrame" then
                    spamRemote = obj
                    spamRawFire = obj.FireServer
                    spamArgs = args
                    print("YEW SPAM: Packet hooked!")
                    spamHooked = true
                end
                return oldIndex(self, key)(obj, ...)
            end
        end
        return oldIndex(self, key)
    end
    setreadonly(mt, true)
end

local function sendSpamPacket()
    if spamRemote and spamRawFire then
        pcall(function() spamRawFire(spamRemote, unpack(spamArgs)) end)
    end
end

-- ============ Manual Spam Window ============
local spamGui, spamFrame, spamButton, spamStroke
local spamManualActivated = false
local spamManualThread = nil

local function createSpamWindow()
    spamGui = Instance.new("ScreenGui")
    spamGui.Name = "YEW_Spam"
    spamGui.ResetOnSpawn = false
    pcall(function() spamGui.Parent = CoreGui end)
    if not spamGui.Parent then
        pcall(function() spamGui.Parent = LocalPlayer:WaitForChild("PlayerGui") end)
    end
    
    spamFrame = Instance.new("Frame")
    spamFrame.Size = UDim2.new(0, 180, 0, 90)
    spamFrame.Position = UDim2.new(0, 20, 0, 120)
    spamFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    spamFrame.BackgroundTransparency = 0.15
    spamFrame.BorderSizePixel = 0
    spamFrame.Active = true
    spamFrame.Draggable = true
    spamFrame.Parent = spamGui
    spamFrame.Visible = false
    Instance.new("UICorner", spamFrame).CornerRadius = UDim.new(0, 14)
    
    spamStroke = Instance.new("UIStroke", spamFrame)
    spamStroke.Thickness = 3
    spamStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    
    spamButton = Instance.new("TextButton", spamFrame)
    spamButton.Text = "YEW: Waiting for packet"
    spamButton.Size = UDim2.new(1, -20, 1, -20)
    spamButton.Position = UDim2.new(0, 10, 0, 10)
    spamButton.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    spamButton.BackgroundTransparency = 0.3
    spamButton.BorderSizePixel = 0
    spamButton.Font = Enum.Font.SourceSansBold
    spamButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    spamButton.TextSize = 12
    Instance.new("UICorner", spamButton).CornerRadius = UDim.new(0, 8)
    
    -- Rainbow border
    task.spawn(function()
        local h = 0
        while spamFrame and spamFrame.Parent do
            h = h + 0.008
            if h > 1 then h = 0 end
            local col = Color3.fromHSV(h, 0.9, 1)
            pcall(function()
                if spamStroke then spamStroke.Color = col end
                if spamButton then spamButton.TextColor3 = col end
            end)
            task.wait(0.02)
        end
    end)
    
    spamButton.MouseButton1Click:Connect(function()
        if not spamHooked then
            hookParryPacket()
        end
        
        spamManualActivated = not spamManualActivated
        if spamManualActivated then
            spamButton.Text = "⏹ STOP SPAM"
            spamManualThread = task.spawn(function()
                while spamManualActivated do
                    sendSpamPacket()
                    task.wait(0.08)
                end
            end)
            print("Manual SPAM: Started")
        else
            spamButton.Text = "▶ START SPAM"
            if spamManualThread then
                task.cancel(spamManualThread)
                spamManualThread = nil
            end
            print("Manual SPAM: Stopped")
        end
    end)
end

-- Manual Spam controls window visibility
task.spawn(function()
    while true do
        task.wait(0.1)
        if spamFrame then
            spamFrame.Visible = manualSpam
        end
    end
end)

-- ============ Fly Mode ============
local function startFly()
    local char = LocalPlayer.Character
    if not char then return end
    local humanoid = char:FindFirstChild("Humanoid")
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not humanoid or not hrp then return end
    
    if flyBodyVelocity then flyBodyVelocity:Destroy() end
    if flyBodyGyro then flyBodyGyro:Destroy() end
    
    flyBodyVelocity = Instance.new("BodyVelocity")
    flyBodyVelocity.MaxForce = Vector3.new(1, 1, 1) * 100000
    flyBodyVelocity.Velocity = Vector3.new(0, 30, 0)
    flyBodyVelocity.Parent = hrp
    
    flyBodyGyro = Instance.new("BodyGyro")
    flyBodyGyro.MaxTorque = Vector3.new(1, 1, 1) * 100000
    flyBodyGyro.CFrame = hrp.CFrame
    flyBodyGyro.Parent = hrp
    
    RunService.RenderStepped:Connect(function()
        if not flyMode or not hrp then return end
        local moveDir = Vector3.new()
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveDir = moveDir + Vector3.new(0, 0, -1) end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveDir = moveDir + Vector3.new(0, 0, 1) end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveDir = moveDir + Vector3.new(-1, 0, 0) end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveDir = moveDir + Vector3.new(1, 0, 0) end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then moveDir = moveDir + Vector3.new(0, 1, 0) end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then moveDir = moveDir + Vector3.new(0, -1, 0) end
        
        if moveDir.Magnitude > 0 then
            flyBodyVelocity.Velocity = moveDir.Unit * 60 + Vector3.new(0, 30, 0)
        else
            flyBodyVelocity.Velocity = Vector3.new(0, 30, 0)
        end
        flyBodyGyro.CFrame = CFrame.new(hrp.Position, hrp.Position + Camera.CFrame.LookVector)
    end)
end

local function stopFly()
    if flyBodyVelocity then flyBodyVelocity:Destroy() end
    if flyBodyGyro then flyBodyGyro:Destroy() end
end

-- ============ Update Functions ============
local function updateSpeed()
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("Humanoid") then
        char.Humanoid.WalkSpeed = speedBoost and currentSpeed or 16
    end
end

local function updateHighJump()
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("Humanoid") then
        char.Humanoid.JumpHeight = highJump and currentJump or 7.2
    end
end

local function updateThunderDash()
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("Humanoid") then
        char.Humanoid.WalkSpeed = thunderDash and 80 or (speedBoost and currentSpeed or 16)
    end
end

local function updateImmortal()
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("Humanoid") then
        if immortal or flyMode then
            char.Humanoid.MaxHealth = math.huge
            char.Humanoid.Health = math.huge
        else
            char.Humanoid.MaxHealth = 100
        end
    end
end

local function updatePlayerESP()
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character then
            local highlight = plr.Character:FindFirstChild("Player_ESP")
            if playerESP then
                if not highlight then
                    highlight = Instance.new("Highlight")
                    highlight.Name = "Player_ESP"
                    highlight.FillColor = Color3.fromRGB(255, 0, 100)
                    highlight.FillTransparency = 0.4
                    highlight.Parent = plr.Character
                end
                highlight.Enabled = true
            else
                if highlight then highlight:Destroy() end
            end
        end
    end
end

local function updateAbilityESP()
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character then
            local highlight = plr.Character:FindFirstChild("Ability_ESP")
            if abilityESP then
                if not highlight then
                    highlight = Instance.new("Highlight")
                    highlight.Name = "Ability_ESP"
                    highlight.FillColor = Color3.fromRGB(0, 255, 100)
                    highlight.FillTransparency = 0.3
                    highlight.Parent = plr.Character
                end
                highlight.Enabled = true
            else
                if highlight then highlight:Destroy() end
            end
        end
    end
end

local function updateAntiLag()
    for _, v in pairs(Workspace:GetDescendants()) do
        if v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Smoke") or v:IsA("Fire") then
            pcall(function() v.Enabled = not antiLag end)
        end
    end
end

local function updateFullbright()
    Lighting.Brightness = fullbright and 5 or (darkMode and 0.3 or 1)
    Lighting.Ambient = darkMode and Color3.fromRGB(20, 20, 40) or Color3.fromRGB(127, 127, 127)
end

local function updateNoFog()
    Lighting.FogEnd = noFog and 999999 or 100000
end

local function updateMute()
    pcall(function() game:GetService("SoundService").MasterVolume = muteSounds and 0 or volumeLevel end)
end

-- Rainbow Sky
task.spawn(function()
    while true do
        task.wait(0.05)
        if rainbowSky then
            colorHue = (colorHue + 0.5) % 360
            Lighting.ColorShift_Top = Color3.fromHSV(colorHue/360, 1, 1)
        end
    end
end)

-- Custom Sky
local function updateCustomSky(theme)
    if not customSky then
        local sky = Lighting:FindFirstChild("YEW_Sky")
        if sky then sky:Destroy() end
        return
    end
    local sky = Lighting:FindFirstChild("YEW_Sky")
    if not sky then
        sky = Instance.new("Sky")
        sky.Name = "YEW_Sky"
        sky.Parent = Lighting
    end
end

-- Skin Changer
local function changeSwordSkin(skinName)
    local char = LocalPlayer.Character
    if not char then return end
    
    for _, v in pairs(char:GetDescendants()) do
        if v:IsA("Tool") and (v.Name:lower():find("sword") or v.Name:lower():find("blade")) then
            pcall(function()
                if v:FindFirstChild("Handle") and v.Handle:IsA("MeshPart") then
                    v.Handle.MeshId = "rbxassetid://" .. skinName
                end
            end)
        end
    end
end

-- Player Skin Database
local playerSkinDatabase = {
    ["Default"] = "",
    ["Dark Knight"] = "rbxassetid://12345678",
    ["Ice Mage"] = "rbxassetid://12345679",
    ["Fire Lord"] = "rbxassetid://12345680",
}

local function applyPlayerSkin(skinName)
    if not playerSkinChanger then return end
    local meshId = playerSkinDatabase[skinName]
    if not meshId or meshId == "" then return end
    local char = LocalPlayer.Character
    if not char then return end
    for _, v in pairs(char:GetDescendants()) do
        if v:IsA("MeshPart") then
            pcall(function() v.MeshId = meshId end)
        end
    end
end

-- Character Respawn
LocalPlayer.CharacterAdded:Connect(function(char)
    task.wait(0.5)
    updateSpeed()
    updateHighJump()
    updateThunderDash()
    updateImmortal()
    updatePlayerESP()
    updateAbilityESP()
    if flyMode then startFly() end
    if skinChanger and selectedSword ~= "" then
        task.wait(1)
        changeSwordSkin(selectedSword)
    end
    if playerSkinChanger then
        task.wait(1)
        applyPlayerSkin(selectedPlayerSkin)
    end
end)

-- ============ UI Components ============
local function makeToggle(parent, title, desc, onChange)
    local card = Instance.new("Frame", parent)
    card.Size = UDim2.new(1, 0, 0, 75)
    card.BackgroundColor3 = Color3.fromRGB(16, 18, 24)
    card.BorderSizePixel = 0
    Instance.new("UICorner", card).CornerRadius = UDim.new(0, 12)
    local stroke = Instance.new("UIStroke", card)
    stroke.Color = Color3.fromRGB(0, 195, 255)
    stroke.Thickness = 1
    stroke.Transparency = 0.7
    
    local titleLabel = Instance.new("TextLabel", card)
    titleLabel.Size = UDim2.new(1, -70, 0, 24)
    titleLabel.Position = UDim2.new(0, 12, 0, 10)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = title
    titleLabel.TextColor3 = Color3.fromRGB(230, 230, 240)
    titleLabel.TextSize = 14
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    
    local descLabel = Instance.new("TextLabel", card)
    descLabel.Size = UDim2.new(1, -70, 0, 18)
    descLabel.Position = UDim2.new(0, 12, 0, 36)
    descLabel.BackgroundTransparency = 1
    descLabel.Text = desc
    descLabel.TextColor3 = Color3.fromRGB(100, 100, 130)
    descLabel.TextSize = 11
    descLabel.Font = Enum.Font.Gotham
    
    local track = Instance.new("Frame", card)
    track.Size = UDim2.new(0, 44, 0, 24)
    track.Position = UDim2.new(1, -56, 0, 25)
    track.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
    track.BorderSizePixel = 0
    Instance.new("UICorner", track).CornerRadius = UDim.new(0, 24)
    
    local thumb = Instance.new("Frame", track)
    thumb.Size = UDim2.new(0, 18, 0, 18)
    thumb.Position = UDim2.new(0, 3, 0, 3)
    thumb.BackgroundColor3 = Color3.fromRGB(100, 100, 120)
    thumb.BorderSizePixel = 0
    Instance.new("UICorner", thumb).CornerRadius = UDim.new(0, 18)
    
    local enabled = false
    local function setState(state)
        enabled = state
        if enabled then
            track.BackgroundColor3 = Color3.fromRGB(0, 80, 120)
            thumb.Position = UDim2.new(0, 23, 0, 3)
            thumb.BackgroundColor3 = Color3.fromRGB(0, 195, 255)
        else
            track.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
            thumb.Position = UDim2.new(0, 3, 0, 3)
            thumb.BackgroundColor3 = Color3.fromRGB(100, 100, 120)
        end
        if onChange then pcall(onChange, state) end
    end
    
    local btn = Instance.new("TextButton", card)
    btn.Size = UDim2.new(1, 0, 1, 0)
    btn.BackgroundTransparency = 1
    btn.Text = ""
    btn.MouseButton1Click:Connect(function() setState(not enabled) end)
    
    return card
end

local function makeSlider(parent, title, minVal, maxVal, defaultVal, onChange)
    local card = Instance.new("Frame", parent)
    card.Size = UDim2.new(1, 0, 0, 85)
    card.BackgroundColor3 = Color3.fromRGB(16, 18, 24)
    card.BorderSizePixel = 0
    Instance.new("UICorner", card).CornerRadius = UDim.new(0, 12)
    Instance.new("UIStroke", card).Color = Color3.fromRGB(0, 195, 255)
    
    local titleLabel = Instance.new("TextLabel", card)
    titleLabel.Size = UDim2.new(0.6, -10, 0, 24)
    titleLabel.Position = UDim2.new(0, 12, 0, 8)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = title
    titleLabel.TextColor3 = Color3.fromRGB(200, 200, 210)
    titleLabel.TextSize = 13
    titleLabel.Font = Enum.Font.Gotham
    
    local valueLabel = Instance.new("TextLabel", card)
    valueLabel.Size = UDim2.new(0.4, -10, 0, 24)
    valueLabel.Position = UDim2.new(0.6, 0, 0, 8)
    valueLabel.BackgroundTransparency = 1
    valueLabel.Text = tostring(defaultVal)
    valueLabel.TextColor3 = Color3.fromRGB(0, 195, 255)
    valueLabel.TextSize = 13
    valueLabel.Font = Enum.Font.GothamBold
    valueLabel.TextXAlignment = Enum.TextXAlignment.Right
    
    local slider = Instance.new("Frame", card)
    slider.Size = UDim2.new(1, -24, 0, 4)
    slider.Position = UDim2.new(0, 12, 0, 50)
    slider.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
    slider.BorderSizePixel = 0
    Instance.new("UICorner", slider).CornerRadius = UDim.new(0, 2)
    
    local fill = Instance.new("Frame", slider)
    fill.Size = UDim2.new((defaultVal - minVal)/(maxVal - minVal), 0, 1, 0)
    fill.BackgroundColor3 = Color3.fromRGB(0, 195, 255)
    
    local knob = Instance.new("Frame", slider)
    knob.Size = UDim2.new(0, 14, 0, 14)
    knob.Position = UDim2.new((defaultVal - minVal)/(maxVal - minVal), -7, 0.5, -7)
    knob.BackgroundColor3 = Color3.fromRGB(0, 195, 255)
    Instance.new("UICorner", knob).CornerRadius = UDim.new(0, 14)
    
    local value = defaultVal
    local dragging = false
    
    local function updateValue(newVal)
        value = math.clamp(newVal, minVal, maxVal)
        local percent = (value - minVal) / (maxVal - minVal)
        fill.Size = UDim2.new(percent, 0, 1, 0)
        knob.Position = UDim2.new(percent, -7, 0.5, -7)
        valueLabel.Text = string.format("%.0f", value)
        if onChange then pcall(onChange, value) end
    end
    
    knob.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
        end
    end)
    knob.InputEnded:Connect(function() dragging = false end)
    
    UserInputService.InputChanged:Connect(function(input)
        if dragging then
            local mousePos = input.Position.X
            local sliderPos = slider.AbsolutePosition.X
            local sliderWidth = slider.AbsoluteSize.X
            local percent = math.clamp((mousePos - sliderPos) / sliderWidth, 0, 1)
            updateValue(minVal + percent * (maxVal - minVal))
        end
    end)
    
    return card
end

local function makeDropdown(parent, title, options, onChange)
    local card = Instance.new("Frame", parent)
    card.Size = UDim2.new(1, 0, 0, 85)
    card.BackgroundColor3 = Color3.fromRGB(16, 18, 24)
    card.BorderSizePixel = 0
    Instance.new("UICorner", card).CornerRadius = UDim.new(0, 12)
    Instance.new("UIStroke", card).Color = Color3.fromRGB(0, 195, 255)
    
    local titleLabel = Instance.new("TextLabel", card)
    titleLabel.Size = UDim2.new(1, -20, 0, 22)
    titleLabel.Position = UDim2.new(0, 12, 0, 8)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = title
    titleLabel.TextColor3 = Color3.fromRGB(200, 200, 210)
    titleLabel.TextSize = 12
    titleLabel.Font = Enum.Font.Gotham
    
    local dropdown = Instance.new("TextButton", card)
    dropdown.Size = UDim2.new(1, -24, 0, 32)
    dropdown.Position = UDim2.new(0, 12, 0, 38)
    dropdown.BackgroundColor3 = Color3.fromRGB(25, 27, 35)
    dropdown.Text = options[1]
    dropdown.TextColor3 = Color3.fromRGB(200, 200, 200)
    dropdown.TextSize = 13
    dropdown.Font = Enum.Font.Gotham
    Instance.new("UICorner", dropdown).CornerRadius = UDim.new(0, 8)
    
    local selectedIndex = 1
    dropdown.MouseButton1Click:Connect(function()
        selectedIndex = selectedIndex % #options + 1
        dropdown.Text = options[selectedIndex]
        if onChange then pcall(onChange, options[selectedIndex]) end
    end)
    
    return card
end

local function makeInput(parent, title, placeholder, onChange)
    local card = Instance.new("Frame", parent)
    card.Size = UDim2.new(1, 0, 0, 90)
    card.BackgroundColor3 = Color3.fromRGB(16, 18, 24)
    card.BorderSizePixel = 0
    Instance.new("UICorner", card).CornerRadius = UDim.new(0, 12)
    Instance.new("UIStroke", card).Color = Color3.fromRGB(0, 195, 255)
    
    local titleLabel = Instance.new("TextLabel", card)
    titleLabel.Size = UDim2.new(1, -20, 0, 22)
    titleLabel.Position = UDim2.new(0, 12, 0, 8)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = title
    titleLabel.TextColor3 = Color3.fromRGB(200, 200, 210)
    titleLabel.TextSize = 12
    titleLabel.Font = Enum.Font.Gotham
    
    local input = Instance.new("TextBox", card)
    input.Size = UDim2.new(1, -24, 0, 36)
    input.Position = UDim2.new(0, 12, 0, 38)
    input.BackgroundColor3 = Color3.fromRGB(25, 27, 35)
    input.PlaceholderText = placeholder
    input.Text = ""
    input.TextColor3 = Color3.fromRGB(200, 200, 200)
    input.Font = Enum.Font.Gotham
    input.TextSize = 13
    Instance.new("UICorner", input).CornerRadius = UDim.new(0, 8)
    
    input:GetPropertyChangedSignal("Text"):Connect(function()
        if onChange then pcall(onChange, input.Text) end
    end)
    
    return card
end

-- ============ Create Main Interface ============
local mainFrame = nil
local openBtn = nil

local function createMainHub()
    mainFrame = Instance.new("Frame")
    mainFrame.Size = UDim2.new(0, 580 * guiScale, 0, 550 * guiScale)
    mainFrame.Position = UDim2.new(0.5, -290 * guiScale, 0.5, -275 * guiScale)
    mainFrame.BackgroundColor3 = Color3.fromRGB(10, 12, 16)
    mainFrame.BorderSizePixel = 0
    mainFrame.Parent = screenGui
    Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 16)
    local mainStroke = Instance.new("UIStroke", mainFrame)
    mainStroke.Color = Color3.fromRGB(0, 195, 255)
    mainStroke.Thickness = 1
    
    -- Dragging
    local dragging = false
    local dragStart, startPos
    mainFrame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = mainFrame.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local delta = input.Position - dragStart
            mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)
    
    -- Header
    local header = Instance.new("Frame", mainFrame)
    header.Size = UDim2.new(1, 0, 0, 70)
    header.BackgroundColor3 = Color3.fromRGB(10, 12, 16)
    header.BorderSizePixel = 0
    Instance.new("UICorner", header).CornerRadius = UDim.new(0, 16)
    
    local logo = Instance.new("Frame", header)
    logo.Size = UDim2.new(0, 50, 0, 50)
    logo.Position = UDim2.new(0, 12, 0, 10)
    logo.BackgroundColor3 = Color3.fromRGB(0, 195, 255)
    logo.BorderSizePixel = 0
    Instance.new("UICorner", logo).CornerRadius = UDim.new(0, 10)
    local logoText = Instance.new("TextLabel", logo)
    logoText.Size = UDim2.new(1, 0, 1, 0)
    logoText.BackgroundTransparency = 1
    logoText.Text = "Y"
    logoText.TextColor3 = Color3.fromRGB(255, 255, 255)
    logoText.TextSize = 24
    logoText.Font = Enum.Font.GothamBold
    
    local titleLabel = Instance.new("TextLabel", header)
    titleLabel.Size = UDim2.new(0, 150, 0, 35)
    titleLabel.Position = UDim2.new(0, 70, 0, 8)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = "YEW FREE"
    titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    titleLabel.TextSize = 22
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    
    local verLabel = Instance.new("TextLabel", header)
    verLabel.Size = UDim2.new(0, 55, 0, 22)
    verLabel.Position = UDim2.new(0, 70, 0, 44)
    verLabel.BackgroundTransparency = 1
    verLabel.Text = "V1.21"
    verLabel.TextColor3 = Color3.fromRGB(0, 195, 255)
    verLabel.TextSize = 12
    verLabel.Font = Enum.Font.GothamBold
    
    local closeBtn = Instance.new("TextButton", header)
    closeBtn.Size = UDim2.new(0, 32, 0, 32)
    closeBtn.Position = UDim2.new(1, -42, 0, 18)
    closeBtn.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
    closeBtn.Text = "−"
    closeBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
    closeBtn.TextSize = 18
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.BorderSizePixel = 0
    Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 8)
    
    -- Sidebar
    local sidebar = Instance.new("Frame", mainFrame)
    sidebar.Size = UDim2.new(0, 150, 1, -71)
    sidebar.Position = UDim2.new(0, 0, 0, 71)
    sidebar.BackgroundColor3 = Color3.fromRGB(10, 12, 16)
    sidebar.BorderSizePixel = 0
    
    -- Content area
    local contentFrame = Instance.new("Frame", mainFrame)
    contentFrame.Size = UDim2.new(1, -166, 1, -71)
    contentFrame.Position = UDim2.new(0, 158, 0, 75)
    contentFrame.BackgroundTransparency = 1
    
    local scrollFrame = Instance.new("ScrollingFrame", contentFrame)
    scrollFrame.Size = UDim2.new(1, 0, 1, 0)
    scrollFrame.BackgroundTransparency = 1
    scrollFrame.ScrollBarThickness = 3
    scrollFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
    
    local contentLayout = Instance.new("UIListLayout", scrollFrame)
    contentLayout.Padding = UDim.new(0, 8)
    contentLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        scrollFrame.CanvasSize = UDim2.new(0, 0, 0, contentLayout.AbsoluteContentSize.Y + 16)
    end)
    
    -- Tabs
    local tabs = {"Blatant", "Players", "Visuals", "World", "Misc", "Exclusive"}
    local icons = {"⚔", "👤", "👁", "🌐", "⊞", "★"}
    local currentTab = "Blatant"
    local tabButtons = {}
    local tabContents = {}
    
    for i, tab in ipairs(tabs) do
        local btn = Instance.new("TextButton", sidebar)
        btn.Size = UDim2.new(1, -16, 0, 38)
        btn.Position = UDim2.new(0, 8, 0, 8 + (i-1) * 46)
        btn.BackgroundColor3 = Color3.fromRGB(18, 20, 26)
        btn.Text = icons[i] .. "  " .. tab
        btn.TextColor3 = Color3.fromRGB(130, 130, 150)
        btn.TextSize = 12
        btn.Font = Enum.Font.GothamBold
        btn.TextXAlignment = Enum.TextXAlignment.Left
        btn.BorderSizePixel = 0
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
        tabButtons[tab] = btn
        
        local container = Instance.new("Frame", scrollFrame)
        container.Size = UDim2.new(1, 0, 0, 0)
        container.BackgroundTransparency = 1
        container.AutomaticSize = Enum.AutomaticSize.Y
        container.Visible = (tab == currentTab)
        tabContents[tab] = container
        
        local containerLayout = Instance.new("UIListLayout", container)
        containerLayout.Padding = UDim.new(0, 8)
    end
    
    local function switchTab(tabName)
        currentTab = tabName
        for name, btn in pairs(tabButtons) do
            if name == tabName then
                btn.BackgroundColor3 = Color3.fromRGB(0, 50, 70)
                btn.TextColor3 = Color3.fromRGB(0, 195, 255)
            else
                btn.BackgroundColor3 = Color3.fromRGB(18, 20, 26)
                btn.TextColor3 = Color3.fromRGB(130, 130, 150)
            end
        end
        for name, cont in pairs(tabContents) do
            cont.Visible = (name == tabName)
        end
    end
    
    for name, btn in pairs(tabButtons) do
        btn.MouseButton1Click:Connect(function() switchTab(name) end)
    end
    
    -- ============ Create All Features ============
    
    -- Blatant Tab
    local blatantCont = tabContents["Blatant"]
    makeToggle(blatantCont, "Auto Parry", "Auto Parry (到达时间预测 + 球速分级)", function(on) autoParry = on end)
    makeToggle(blatantCont, "Auto Spam", "Auto Spam (自动连点)", function(on) 
        autoSpam = on
        if on then startAutoSpam() else stopAutoSpam() end
    end)
    makeToggle(blatantCont, "Manual Spam", "Manual YEW SPAM (独立窗口)", function(on) manualSpam = on end)
    makeToggle(blatantCont, "Ball Stats", "显示球速 (Current | Max)", function(on) ballStats = on end)
    makeToggle(blatantCont, "Parry Visualiser", "格挡范围显示 (圆形)", function(on) parryVisualiser = on end)
    
    -- Players Tab
    local playersCont = tabContents["Players"]
    makeToggle(playersCont, "Player ESP", "玩家透视", function(on) playerESP = on; updatePlayerESP() end)
    makeToggle(playersCont, "Ability ESP", "技能ESP", function(on) abilityESP = on; updateAbilityESP() end)
    makeToggle(playersCont, "High Jump", "高跳", function(on) highJump = on; updateHighJump() end)
    makeSlider(playersCont, "Jump Power", 7.2, 120, 50, function(val) currentJump = val; if highJump then updateHighJump() end end)
    makeToggle(playersCont, "Speed Boost", "速度提升", function(on) speedBoost = on; updateSpeed() end)
    makeSlider(playersCont, "Walk Speed", 16, 120, 50, function(val) currentSpeed = val; updateSpeed() end)
    
    -- Visuals Tab
    local visualsCont = tabContents["Visuals"]
    makeToggle(visualsCont, "Rainbow Sky", "彩虹天空", function(on) rainbowSky = on end)
    makeToggle(visualsCont, "Fullbright", "全亮", function(on) fullbright = on; updateFullbright() end)
    makeToggle(visualsCont, "No Fog", "去除雾气", function(on) noFog = on; updateNoFog() end)
    makeToggle(visualsCont, "Dark Mode", "暗黑模式", function(on) darkMode = on; updateFullbright() end)
    
    -- World Tab
    local worldCont = tabContents["World"]
    makeToggle(worldCont, "Custom Sky", "自定义天空", function(on) customSky = on; updateCustomSky(selectedTheme) end)
    makeDropdown(worldCont, "Sky Theme", {"Default", "Night", "Sunset", "Rainbow"}, function(val) selectedTheme = val; updateCustomSky(val) end)
    makeToggle(worldCont, "Mute Sounds", "静音音效", function(on) muteSounds = on; updateMute() end)
    makeSlider(worldCont, "Volume Level", 0, 10, 1, function(val) volumeLevel = val; updateMute() end)
    makeToggle(worldCont, "Anti Lag", "减少卡顿", function(on) antiLag = on; updateAntiLag() end)
    
    -- Misc Tab
    local miscCont = tabContents["Misc"]
    makeToggle(miscCont, "Skin Changer", "剑皮肤更换 (输入 Mesh ID)", function(on) skinChanger = on end)
    makeInput(miscCont, "Sword ID", "输入剑的 Mesh ID...", function(val)
        selectedSword = val
        if skinChanger then changeSwordSkin(val) end
        print("✅ 皮肤已更换: " .. val)
    end)
    makeToggle(miscCont, "Player Skin Changer", "玩家皮肤更换", function(on) playerSkinChanger = on end)
    makeDropdown(miscCont, "Player Skin", {"Default", "Dark Knight", "Ice Mage", "Fire Lord"}, function(val)
        selectedPlayerSkin = val
        if playerSkinChanger then applyPlayerSkin(val) end
    end)
    makeToggle(miscCont, "Thunder Dash", "雷霆冲刺", function(on) thunderDash = on; updateThunderDash() end)
    
    -- Exclusive Tab
    local exclusiveCont = tabContents["Exclusive"]
    makeToggle(exclusiveCont, "Fly Mode", "飞行模式 + 无敌 (WASD + Space)", function(on)
        flyMode = on
        immortal = on
        if on then startFly() else stopFly() end
        updateImmortal()
    end)
    
    -- Open/Close Button
    openBtn = Instance.new("TextButton", screenGui)
    openBtn.Size = UDim2.new(0, 60, 0, 60)
    openBtn.Position = UDim2.new(1, -75, 1, -85)
    openBtn.BackgroundColor3 = Color3.fromRGB(0, 60, 80)
    openBtn.Text = "⚡"
    openBtn.TextColor3 = Color3.fromRGB(0, 195, 255)
    openBtn.TextSize = 26
    openBtn.Font = Enum.Font.GothamBold
    openBtn.BorderSizePixel = 0
    openBtn.Visible = false
    Instance.new("UICorner", openBtn).CornerRadius = UDim.new(1, 30)
    Instance.new("UIStroke", openBtn).Color = Color3.fromRGB(0, 195, 255)
    
    openBtn.MouseButton1Click:Connect(function()
        mainFrame.Visible = true
        openBtn.Visible = false
    end)
    
    closeBtn.MouseButton1Click:Connect(function()
        mainFrame.Visible = false
        openBtn.Visible = true
    end)
    
    -- Shortcut Shift+X
    UserInputService.InputBegan:Connect(function(input, gp)
        if gp then return end
        if input.KeyCode == Enum.KeyCode.X and UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
            if mainFrame then
                mainFrame.Visible = not mainFrame.Visible
                if openBtn then openBtn.Visible = not mainFrame.Visible end
            end
        end
    end)
    
    switchTab("Blatant")
    
    print("========================================")
    print("✅ YEW FREE Hub - Blade Ball V1.21 Started")
    print(isMobile and "📱 Mobile mode (Touch Optimized)" or "💻 PC mode (Mouse/Keyboard)")
    print("⚔️ Auto Parry: 到达时间预测 + 球速分级")
    print("🔑 Press Shift+X to open/close menu")
    print("========================================")
end

-- ============ Ball Speed Display ============
local speedDisplay = nil
task.spawn(function()
    while true do
        task.wait(0.05)
        if ballStats then
            if not speedDisplay or not speedDisplay.Parent then
                speedDisplay = Instance.new("TextLabel", screenGui)
                speedDisplay.Size = UDim2.new(0, 240, 0, 50)
                speedDisplay.Position = UDim2.new(0.5, -120, 0, 10)
                speedDisplay.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
                speedDisplay.BackgroundTransparency = 0.5
                speedDisplay.TextColor3 = Color3.fromRGB(0, 195, 255)
                speedDisplay.TextSize = 16
                speedDisplay.Font = Enum.Font.GothamBold
                Instance.new("UICorner", speedDisplay).CornerRadius = UDim.new(0, 8)
            end
            speedDisplay.Text = "⚡ Current: " .. math.floor(currentBallSpeed) .. "  |  Max: " .. math.floor(maxBallSpeed)
            speedDisplay.Visible = true
        else
            if speedDisplay then speedDisplay.Visible = false end
        end
    end
end)

-- ============ Parry Range Display ============
local rangeCircle = nil
RunService.RenderStepped:Connect(function()
    if parryVisualiser then
        local char = LocalPlayer.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            local hrp = char.HumanoidRootPart
            if not rangeCircle or not rangeCircle.Parent then
                rangeCircle = Instance.new("Part")
                rangeCircle.Name = "ParryRangeCircle"
                rangeCircle.Shape = Enum.PartType.Ball
                rangeCircle.Size = Vector3.new(18, 18, 18)
                rangeCircle.Anchored = true
                rangeCircle.CanCollide = false
                rangeCircle.Transparency = 0.85
                rangeCircle.Color = Color3.fromRGB(0, 195, 255)
                rangeCircle.Material = Enum.Material.Neon
                rangeCircle.Parent = char
            end
            local size = 18
            if currentBallSpeed > 250 then size = 50
            elseif currentBallSpeed > 200 then size = 45
            elseif currentBallSpeed > 150 then size = 40
            elseif currentBallSpeed > 100 then size = 35
            elseif currentBallSpeed > 60 then size = 30
            elseif currentBallSpeed > 30 then size = 25
            end
            rangeCircle.Size = Vector3.new(size, size, size)
            rangeCircle.CFrame = CFrame.new(hrp.Position)
            rangeCircle.Transparency = 0.85
        end
    else
        if rangeCircle then rangeCircle:Destroy() end
    end
end)

-- ============ Password GUI ============
local function createPasswordGui()
    local passwordScreenGui = Instance.new("ScreenGui")
    passwordScreenGui.Name = "YEW_Password"
    passwordScreenGui.ResetOnSpawn = false
    pcall(function() passwordScreenGui.Parent = CoreGui end)
    if not passwordScreenGui.Parent then
        pcall(function() passwordScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui") end)
    end
    
    local bg = Instance.new("Frame", passwordScreenGui)
    bg.Size = UDim2.new(1, 0, 1, 0)
    bg.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    bg.BackgroundTransparency = 0.7
    bg.BorderSizePixel = 0
    
    local panel = Instance.new("Frame", passwordScreenGui)
    panel.Size = UDim2.new(0, 380, 0, 340)
    panel.Position = UDim2.new(0.5, -190, 0.5, -170)
    panel.BackgroundColor3 = Color3.fromRGB(10, 12, 16)
    panel.BorderSizePixel = 0
    Instance.new("UICorner", panel).CornerRadius = UDim.new(0, 20)
    local panelStroke = Instance.new("UIStroke", panel)
    panelStroke.Color = Color3.fromRGB(0, 195, 255)
    panelStroke.Thickness = 2
    
    local logo = Instance.new("Frame", panel)
    logo.Size = UDim2.new(0, 70, 0, 70)
    logo.Position = UDim2.new(0.5, -35, 0, 20)
    logo.BackgroundColor3 = Color3.fromRGB(0, 195, 255)
    logo.BorderSizePixel = 0
    Instance.new("UICorner", logo).CornerRadius = UDim.new(0, 18)
    local logoText = Instance.new("TextLabel", logo)
    logoText.Size = UDim2.new(1, 0, 1, 0)
    logoText.BackgroundTransparency = 1
    logoText.Text = "Y"
    logoText.TextColor3 = Color3.fromRGB(255, 255, 255)
    logoText.TextSize = 36
    logoText.Font = Enum.Font.GothamBold
    
    local title = Instance.new("TextLabel", panel)
    title.Size = UDim2.new(1, 0, 0, 30)
    title.Position = UDim2.new(0, 0, 0, 100)
    title.BackgroundTransparency = 1
    title.Text = "YEW FREE"
    title.TextColor3 = Color3.fromRGB(0, 195, 255)
    title.TextSize = 24
    title.Font = Enum.Font.GothamBold
    
    local subtitle = Instance.new("TextLabel", panel)
    subtitle.Size = UDim2.new(1, 0, 0, 20)
    subtitle.Position = UDim2.new(0, 0, 0, 130)
    subtitle.BackgroundTransparency = 1
    subtitle.Text = "Enter Password to Continue"
    subtitle.TextColor3 = Color3.fromRGB(150, 150, 180)
    subtitle.TextSize = 12
    subtitle.Font = Enum.Font.Gotham
    
    local inputFrame = Instance.new("Frame", panel)
    inputFrame.Size = UDim2.new(0, 280, 0, 50)
    inputFrame.Position = UDim2.new(0.5, -140, 0, 160)
    inputFrame.BackgroundColor3 = Color3.fromRGB(20, 22, 28)
    inputFrame.BorderSizePixel = 0
    Instance.new("UICorner", inputFrame).CornerRadius = UDim.new(0, 12)
    
    local passwordInput = Instance.new("TextBox", inputFrame)
    passwordInput.Size = UDim2.new(1, -20, 1, 0)
    passwordInput.Position = UDim2.new(0, 10, 0, 0)
    passwordInput.BackgroundTransparency = 1
    passwordInput.PlaceholderText = "Enter Password"
    passwordInput.Text = ""
    passwordInput.TextColor3 = Color3.fromRGB(255, 255, 255)
    passwordInput.PlaceholderColor3 = Color3.fromRGB(100, 100, 130)
    passwordInput.TextSize = 14
    passwordInput.Font = Enum.Font.Gotham
    passwordInput.ClearTextOnFocus = false
    
    local errorLabel = Instance.new("TextLabel", panel)
    errorLabel.Size = UDim2.new(1, -40, 0, 20)
    errorLabel.Position = UDim2.new(0, 20, 0, 220)
    errorLabel.BackgroundTransparency = 1
    errorLabel.Text = ""
    errorLabel.TextColor3 = Color3.fromRGB(255, 50, 50)
    errorLabel.TextSize = 11
    errorLabel.Font = Enum.Font.Gotham
    
    local submitBtn = Instance.new("TextButton", panel)
    submitBtn.Size = UDim2.new(0, 200, 0, 45)
    submitBtn.Position = UDim2.new(0.5, -100, 0, 250)
    submitBtn.BackgroundColor3 = Color3.fromRGB(0, 80, 100)
    submitBtn.Text = "UNLOCK"
    submitBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    submitBtn.TextSize = 16
    submitBtn.Font = Enum.Font.GothamBold
    submitBtn.BorderSizePixel = 0
    Instance.new("UICorner", submitBtn).CornerRadius = UDim.new(0, 12)
    
    -- Discord Button
    local discordBtn = Instance.new("TextButton", panel)
    discordBtn.Size = UDim2.new(0, 200, 0, 35)
    discordBtn.Position = UDim2.new(0.5, -100, 0, 305)
    discordBtn.BackgroundColor3 = Color3.fromRGB(88, 101, 242)
    discordBtn.Text = "Join Discord"
    discordBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    discordBtn.TextSize = 14
    discordBtn.Font = Enum.Font.GothamBold
    discordBtn.BorderSizePixel = 0
    Instance.new("UICorner", discordBtn).CornerRadius = UDim.new(0, 8)
    
    local discordLink = "https://discord.gg/kfeMqHhaR"
    
    discordBtn.MouseButton1Click:Connect(function()
        local copied = false
        pcall(function() setclipboard(discordLink); copied = true end)
        if not copied then
            pcall(function() Clipboard:set(discordLink); copied = true end)
        end
        if not copied then
            pcall(function() toclipboard(discordLink); copied = true end)
        end
        if copied then
            errorLabel.Text = "✅ Discord link copied!"
            errorLabel.TextColor3 = Color3.fromRGB(0, 255, 100)
            task.wait(2)
            errorLabel.Text = ""
        end
    end)
    
    local function checkPassword()
        if passwordInput.Text == "018828772928392" then
            passwordScreenGui:Destroy()
            createMainHub()
            createSpamWindow()
            print("✅ Password correct! YEW FREE Hub V1.21 Started")
        else
            errorLabel.Text = "Invalid Password! Please try again."
            errorLabel.TextColor3 = Color3.fromRGB(255, 50, 50)
            passwordInput.Text = ""
            for i = 1, 3 do
                submitBtn.BackgroundColor3 = Color3.fromRGB(150, 50, 50)
                task.wait(0.1)
                submitBtn.BackgroundColor3 = Color3.fromRGB(0, 80, 100)
                task.wait(0.1)
            end
        end
    end
    
    submitBtn.MouseButton1Click:Connect(checkPassword)
    passwordInput.FocusLost:Connect(function(enterPressed)
        if enterPressed then checkPassword() end
    end)
    
    panel.BackgroundTransparency = 1
    TweenService:Create(panel, TweenInfo.new(0.3, Enum.EasingStyle.Back), {BackgroundTransparency = 0}):Play()
end

-- ============ Start ============
createPasswordGui()

print("========================================")
print("🔐 YEW FREE Hub - Blade Ball V1.21")
print("🔑 Password: 018828772928392")
print("💬 Discord: https://discord.gg/kfeMqHhaR")
print("========================================")
