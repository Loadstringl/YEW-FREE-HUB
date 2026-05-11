local target_key = "717727271818828382818177918181718817272728181827272772718282737272881818283737373728288182828283737466E728272772YHDUDUDHDHW8SUFHHSHSIISUSUSJSHS-YEW-FREE"
local my_discord = "https://discord.gg/uK9ZKyXeY"

local function start_yew_script()
    -- 1. 剪贴板劫持：锁定最新 Discord 链接
    local old_setclipboard
    old_setclipboard = hookfunction(setclipboard or toclipboard, function(val)
        if val and (val:find("discord.gg") or val:find("t.me")) then
            return old_setclipboard(my_discord)
        end
        return old_setclipboard(val)
    end)

    -- 2. 深度清理逻辑
    local function ultra_clean(v)
        if v:IsA("TextLabel") or v:IsA("TextButton") or v:IsA("TextBox") then
            local function rewrite()
                local t = v.Text
                if t:lower() == "by al" then v.Text = "by YEW" end
                if t:find("@") or t:find("Submitting") then v.Text = "Submitting as @YEW" end
                if t:find("NodeX") or t:find("Node") or t == "N" then
                    local nt = t:gsub("NodeX", "YEW FREE"):gsub("Node", "YEW")
                    if nt == "N" then nt = "Y" end
                    if v.Text ~= nt then v.Text = nt end
                end
                if t:find("discord.gg") and not t:find("uK9ZKyXeY") then v.Text = my_discord end
                if t:find("Developer:") then v.Text = "Developer: YEW FREE Team" end
            end
            rewrite()
            v:GetPropertyChangedSignal("Text"):Connect(rewrite)
        end
    end

    -- 3. 开启监控
    local function start_monitor(root)
        if not root then return end
        root.DescendantAdded:Connect(ultra_clean)
        for _, obj in pairs(root:GetDescendants()) do ultra_clean(obj) end
    end

    task.spawn(function()
        start_monitor(game:GetService("Players").LocalPlayer:WaitForChild("PlayerGui"))
        pcall(function() start_monitor(game:GetService("CoreGui")) end)
    end)

    -- 4. 运行原始脚本
    loadstring(game:HttpGet("https://api.cutie.lt/Main"))()
end

-- --- 重新补全的顶级验证 UI 面板 ---
local ScreenGui = Instance.new("ScreenGui", game:GetService("CoreGui"))
local Main = Instance.new("Frame", ScreenGui)
local UICorner = Instance.new("UICorner", Main)
local UIStroke = Instance.new("UIStroke", Main)
local Title = Instance.new("TextLabel", Main)
local KeyInput = Instance.new("TextBox", Main)
local VerifyBtn = Instance.new("TextButton", Main)
local GetKeyBtn = Instance.new("TextButton", Main)

Main.Size = UDim2.new(0, 420, 0, 260) -- 稍微调高一点容纳新按钮
Main.Position = UDim2.new(0.5, -210, 0.5, -130)
Main.BackgroundColor3 = Color3.fromRGB(10, 10, 12)
UICorner.CornerRadius = UDim.new(0, 15)
UIStroke.Thickness = 3.5
UIStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border

Title.Size = UDim2.new(1, 0, 0, 60)
Title.Text = "YEW FREE | AUTHENTICATION"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.TextSize = 22
Title.Font = Enum.Font.GothamBold
Title.BackgroundTransparency = 1

KeyInput.Size = UDim2.new(0.85, 0, 0, 45)
KeyInput.Position = UDim2.new(0.075, 0, 0.28, 0)
KeyInput.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
KeyInput.PlaceholderText = "ENTER KEY..."
KeyInput.TextColor3 = Color3.new(1, 1, 1)
Instance.new("UICorner", KeyInput).CornerRadius = UDim.new(0, 10)

VerifyBtn.Size = UDim2.new(0.85, 0, 0, 45)
VerifyBtn.Position = UDim2.new(0.075, 0, 0.52, 0)
VerifyBtn.Text = "VERIFY & START"
VerifyBtn.Font = Enum.Font.GothamBold
VerifyBtn.TextColor3 = Color3.new(1, 1, 1)
Instance.new("UICorner", VerifyBtn).CornerRadius = UDim.new(0, 10)

-- 找回的 GET KEY 按钮
GetKeyBtn.Size = UDim2.new(0.85, 0, 0, 45)
GetKeyBtn.Position = UDim2.new(0.075, 0, 0.74, 0)
GetKeyBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
GetKeyBtn.Text = "GET KEY (DISCORD)"
GetKeyBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
GetKeyBtn.Font = Enum.Font.GothamBold
Instance.new("UICorner", GetKeyBtn).CornerRadius = UDim.new(0, 10)

-- 按钮逻辑
GetKeyBtn.MouseButton1Click:Connect(function()
    setclipboard(my_discord)
    GetKeyBtn.Text = "LINK COPIED!"
    task.wait(1.5)
    GetKeyBtn.Text = "GET KEY (DISCORD)"
end)

VerifyBtn.MouseButton1Click:Connect(function()
    if KeyInput.Text == target_key then
        ScreenGui:Destroy()
        start_yew_script()
    else
        VerifyBtn.Text = "INVALID KEY"
        task.wait(1)
        VerifyBtn.Text = "VERIFY & START"
    end
end)

-- 彩虹动画
task.spawn(function()
    while Main.Parent do
        local color = Color3.fromHSV(tick() % 4 / 4, 0.8, 1)
        UIStroke.Color = color
        Title.TextColor3 = color
        VerifyBtn.BackgroundColor3 = color
        task.wait()
    end
end)
