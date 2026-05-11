--// =========================
--// YEW FREE BROKEN NOTICE
--// =========================

local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")

-- 防重复
if CoreGui:FindFirstChild("YEW_FREE_NOTICE") then
    CoreGui.YEW_FREE_NOTICE:Destroy()
end

-- GUI
local SG = Instance.new("ScreenGui")
SG.Name = "YEW_FREE_NOTICE"
SG.ResetOnSpawn = false
SG.IgnoreGuiInset = true
SG.Parent = CoreGui

-- MAIN FRAME
local Main = Instance.new("Frame")
Main.Parent = SG

Main.Size = UDim2.new(0, 420, 0, 170)
Main.Position = UDim2.new(0.5, -210, -0.3, 0)

Main.BackgroundColor3 = Color3.fromRGB(15,15,15)
Main.BorderSizePixel = 0

Instance.new("UICorner", Main).CornerRadius = UDim.new(0,14)

local Stroke = Instance.new("UIStroke")
Stroke.Parent = Main
Stroke.Thickness = 2.5

-- TITLE
local Title = Instance.new("TextLabel")
Title.Parent = Main

Title.Size = UDim2.new(1,0,0,45)
Title.BackgroundTransparency = 1

Title.Font = Enum.Font.GothamBold
Title.Text = "YEW FREE"

Title.TextColor3 = Color3.new(1,1,1)
Title.TextSize = 24

-- STATUS TEXT
local Status = Instance.new("TextLabel")
Status.Parent = Main

Status.Size = UDim2.new(1,-30,0,60)
Status.Position = UDim2.new(0,15,0,55)

Status.BackgroundTransparency = 1

Status.Font = Enum.Font.Gotham
Status.TextWrapped = true
Status.TextYAlignment = Enum.TextYAlignment.Top

Status.TextColor3 = Color3.fromRGB(220,220,220)
Status.TextSize = 17

Status.Text =
"Script is currently broken.\nPlease wait for an update."

-- DISCORD
local Discord = Instance.new("TextLabel")
Discord.Parent = Main

Discord.Size = UDim2.new(1,0,0,25)
Discord.Position = UDim2.new(0,0,1,-35)

Discord.BackgroundTransparency = 1

Discord.Font = Enum.Font.GothamBold
Discord.Text = "discord.gg/S34maAp3J"

Discord.TextColor3 = Color3.fromRGB(255,170,0)
Discord.TextSize = 15

-- CLOSE BUTTON
local Close = Instance.new("TextButton")
Close.Parent = Main

Close.Size = UDim2.new(0,30,0,30)
Close.Position = UDim2.new(1,-38,0,8)

Close.Text = "X"

Close.Font = Enum.Font.GothamBold
Close.TextSize = 18

Close.BackgroundColor3 = Color3.fromRGB(35,35,35)
Close.TextColor3 = Color3.new(1,1,1)

Instance.new("UICorner", Close).CornerRadius = UDim.new(1,0)

-- OPEN ANIMATION
TweenService:Create(
    Main,
    TweenInfo.new(0.45, Enum.EasingStyle.Quint),
    {
        Position = UDim2.new(0.5,-210,0.08,0)
    }
):Play()

-- CLOSE BUTTON
Close.MouseButton1Click:Connect(function()

    TweenService:Create(
        Main,
        TweenInfo.new(0.35, Enum.EasingStyle.Quint),
        {
            Position = UDim2.new(0.5,-210,-0.3,0)
        }
    ):Play()

    task.wait(0.35)

    SG:Destroy()

end)

-- RAINBOW BORDER
task.spawn(function()

    local h = 0

    while SG.Parent do

        h = (h + 0.01) % 1

        Stroke.Color = Color3.fromHSV(h,1,1)

        task.wait(0.02)

    end

end)
