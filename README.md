local target_key = "717727271818828382818177918181718817272728181827272772718282737272881818283737373728288182828283737466E728272772YHDUDUDHDHW8SUFHHSHSIISUSUSJSHS-YEW-FREE"
local my_discord = "https://discord.gg/S34maAp3J"

-- 只有验证成功才会执行这一段你提供的核心逻辑
local function start_yew_script()
    local target_title = "YEW FREE | Blade Ball by YEW"

    local function force_clean(v)
        -- A. 彻底粉碎感叹号图标
        if v:IsA("ImageLabel") or v:IsA("ImageButton") then
            if v.Image:find("10747383580") or v.Name:find("Icon") or (v.Parent and v.Parent.Name:find("Circle")) then
                v:Destroy()
            end
        end

        -- B. 标题处理 (防止重复，强行删掉多余的)
        if v:IsA("TextLabel") or v:IsA("TextButton") or v:IsA("TextBox") then
            local function apply_fix()
                local txt = v.Text
                
                -- 1. 标题逻辑：检查是否包含关键字
                if txt:find("YEW") or txt:find("Blade Ball") or txt:find("cutie") then
                    if txt:match("YEW.*YEW") or txt:len() > #target_title + 10 then
                        v.Text = target_title
                    elseif v.Text ~= target_title then
                        v.Text = target_title
                    end
                    
                    if v.Parent then
                        for _, sibling in pairs(v.Parent:GetChildren()) do
                            if sibling ~= v and sibling:IsA("TextLabel") and (sibling.Text:find("YEW") or sibling.Text:find("Blade Ball")) then
                                sibling:Destroy()
                            end
                        end
                    end
                    return 
                end

                -- 2. 清除输入框里的旧 Discord
                if v:IsA("TextBox") then
                    if v.PlaceholderText:lower():find("cutie") then
                        v.PlaceholderText = my_discord
                    end
                    if v.Text:lower():find("cutie") or v.Text == "" then
                        v.Text = my_discord
                    end
                end

                -- 3. 替换旧链接
                if txt:lower():find("cutie") or txt:lower():find("discord.gg/cutie") then
                    v.Text = "JOIN DISCORD: " .. my_discord
                end
            end

            apply_fix()
            v:GetPropertyChangedSignal("Text"):Connect(apply_fix)
        end

        -- C. 圆圈彩虹变色
        if v.Name == "Circle" or v.Name == "LogoCircle" or (v:IsA("Frame") and v.Size.X.Offset == 50) then
            task.spawn(function()
                while v and v.Parent do
                    local color = Color3.fromHSV(tick() % 5 / 5, 0.8, 1)
                    game:GetService("TweenService"):Create(v, TweenInfo.new(1), {
                        [v:IsA("UIStroke") and "Color" or "BackgroundColor3"] = color
                    }):Play()
                    task.wait(1)
                end
            end)
        end
    end

    local function monitor(gui)
        if not gui then return end
        gui.DescendantAdded:Connect(force_clean)
        for _, v in pairs(gui:GetDescendants()) do force_clean(v) end
    end

    task.spawn(function()
        monitor(game:GetService("Players").LocalPlayer:WaitForChild("PlayerGui"))
        pcall(function() monitor(game:GetService("CoreGui")) end)
    end)

    pcall(function()
        local old; old = hookfunction(setclipboard, function(val)
            if val and string.find(string.lower(tostring(val)), "discord") then
                return old(my_discord)
            end
            return old(val)
        end)
    end)

    pcall(function() writefile("yew_free_confirm_tos.txt", "confirmed") end)
    loadstring(game:HttpGet("https://api.cutie.lt/Main"))()
end

-- 钥匙系统 UI 面板
local ScreenGui = Instance.new("ScreenGui", game:GetService("CoreGui"))
local Main = Instance.new("Frame", ScreenGui)
local Title = Instance.new("TextLabel", Main)
local KeyInput = Instance.new("TextBox", Main)
local VerifyBtn = Instance.new("TextButton", Main)
local GetKeyBtn = Instance.new("TextButton", Main)
local UIStroke = Instance.new("UIStroke", Main)
local UICorner = Instance.new("UICorner", Main)

Main.Size = UDim2.new(0, 400, 0, 250)
Main.Position = UDim2.new(0.5, -200, 0.5, -125)
Main.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
UICorner.CornerRadius = UDim.new(0, 10)
UIStroke.Thickness = 3

Title.Size = UDim2.new(1, 0, 0, 60)
Title.Text = "YEW FREE | ULTIMATE SECURITY"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 22
Title.Font = Enum.Font.GothamBold
Title.BackgroundTransparency = 1

KeyInput.Size = UDim2.new(0.9, 0, 0, 45)
KeyInput.Position = UDim2.new(0.05, 0, 0.3, 0)
KeyInput.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
KeyInput.PlaceholderText = "Enter Ultimate Key..."
KeyInput.Text = ""
KeyInput.TextColor3 = Color3.fromRGB(255, 255, 255)
KeyInput.TextScaled = true
Instance.new("UICorner", KeyInput)

VerifyBtn.Size = UDim2.new(0.9, 0, 0, 40)
VerifyBtn.Position = UDim2.new(0.05, 0, 0.55, 0)
VerifyBtn.Text = "VERIFY & DECRYPT"
VerifyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
VerifyBtn.Font = Enum.Font.GothamBold
Instance.new("UICorner", VerifyBtn)

GetKeyBtn.Size = UDim2.new(0.9, 0, 0, 40)
GetKeyBtn.Position = UDim2.new(0.05, 0, 0.75, 0)
GetKeyBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
GetKeyBtn.Text = "GET DISCORD KEY (COPY)"
GetKeyBtn.TextColor3 = Color3.fromRGB(180, 180, 180)
Instance.new("UICorner", GetKeyBtn)

GetKeyBtn.MouseButton1Click:Connect(function()
    setclipboard(my_discord)
    GetKeyBtn.Text = "DISCORD LINK COPIED!"
    task.wait(2)
    GetKeyBtn.Text = "GET DISCORD KEY (COPY)"
end)

VerifyBtn.MouseButton1Click:Connect(function()
    if KeyInput.Text == target_key then
        VerifyBtn.Text = "SUCCESS..."
        task.wait(0.5)
        ScreenGui:Destroy()
        start_yew_script()
    else
        VerifyBtn.Text = "INVALID KEY"
        task.wait(1)
        VerifyBtn.Text = "VERIFY & DECRYPT"
    end
end)

task.spawn(function()
    while Main.Parent do
        local color = Color3.fromHSV(tick() % 3 / 3, 1, 1)
        UIStroke.Color = color
        VerifyBtn.BackgroundColor3 = color
        Title.TextColor3 = color
        task.wait()
    end
end)
