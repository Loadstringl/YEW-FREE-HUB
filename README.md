-- [[ SCRIPT ERROR NOTIFICATION SYSTEM ]] --

local StarterGui = game:GetService("StarterGui")

-- 1. Function to show the "Subtitle" error notification
local function ShowScriptError()
    local success, err = pcall(function()
        StarterGui:SetCore("SendNotification", {
            Title = "SCRIPT ERROR";            -- 标题：脚本错误
            Text = "Please wait for update.";  -- 内容：请等待更新
            Duration = 15;                     -- 持续显示15秒
            Button1 = "OK";                    -- 确认按钮
        })
    end)
    
    -- If the executor's GUI fails, print to console as backup
    if not success then
        warn("!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!")
        warn("SCRIPT ERROR")
        warn("PLEASE WAIT FOR UPDATE.")
        warn("!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!")
    end
end

-- 2. Your Main Script Logic with Error Catching
local function StartExecution()
    -- This simulates a script failure (e.g., version mismatch)
    -- If you want to trigger the error, leave the next line as is.
    -- If you want the script to run, comment out the 'error' line.
    error("VERSION_OUTDATED") 
end

-- [[ THE PROTECTOR (PCALL) ]] --
-- This detects if your script has an issue and triggers the 15s notification
local success, result = pcall(StartExecution)

if not success then
    -- When the script fails, it shows the "SCRIPT ERROR, PLEASE WAIT FOR UPDATE" message
    ShowScriptError()
end
