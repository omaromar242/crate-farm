local player = game.Players.LocalPlayer
local replicatedStorage = game:GetService("ReplicatedStorage")
local teleportService = game:GetService("TeleportService")
local soundId = "rbxassetid://6346446980" -- Change this to your preferred sound ID
local placeId = game.PlaceId -- Current place ID for rejoining

-- GUI Setup
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = player.PlayerGui
screenGui.Name = "ControlGui"

-- Create Start Button
local startButton = Instance.new("TextButton")
startButton.Parent = screenGui
startButton.Size = UDim2.new(0, 200, 0, 50)
startButton.Position = UDim2.new(0.5, -210, 0.5, 100)
startButton.Text = "Start"
startButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
startButton.TextScaled = true

-- Create Rejoin Button
local rejoinButton = Instance.new("TextButton")
rejoinButton.Parent = screenGui
rejoinButton.Size = UDim2.new(0, 200, 0, 50)
rejoinButton.Position = UDim2.new(0.5, 10, 0.5, 100)
rejoinButton.Text = "Rejoin"
rejoinButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
rejoinButton.TextScaled = true

-- Create Hide Button
local hideButton = Instance.new("TextButton")
hideButton.Parent = screenGui
hideButton.Size = UDim2.new(0, 100, 0, 30)
hideButton.Position = UDim2.new(0.5, -50, 0.5, 160)
hideButton.Text = "Hide"
hideButton.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
hideButton.TextScaled = true

-- Timer Label
local timerLabel = Instance.new("TextLabel")
timerLabel.Parent = screenGui
timerLabel.Size = UDim2.new(0, 500, 0, 100)
timerLabel.Position = UDim2.new(0.5, -250, 0.5, -50)
timerLabel.TextScaled = true
timerLabel.BackgroundTransparency = 0.5
timerLabel.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
timerLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
timerLabel.Visible = false

-- BillboardGui above player
local billboardGui = Instance.new("BillboardGui")
billboardGui.Size = UDim2.new(0, 200, 0, 50)
billboardGui.StudsOffset = Vector3.new(0, 3, 0)
billboardGui.Parent = player.Character:WaitForChild("Head")
billboardGui.Adornee = player.Character:WaitForChild("Head")

local messageLabel = Instance.new("TextLabel")
messageLabel.Parent = billboardGui
messageLabel.Size = UDim2.new(1, 0, 1, 0)
messageLabel.TextScaled = true
messageLabel.BackgroundTransparency = 1
messageLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
messageLabel.Text = "Don't go anywhere, stay here until the timer is done"
messageLabel.Visible = false

-- Start the illegal job
local function startIllegalJob()
    local args = {
        [1] = "StartIllegalJob",
        [2] = replicatedStorage:FindFirstChild("GUI'sForPlayer").HUD.CriminalComputerFrame.ScrollingFrame:FindFirstChild("Deliver Crate")
    }

    if replicatedStorage:FindFirstChild("Events") and replicatedStorage.Events:FindFirstChild("WeaponEvent") then
        replicatedStorage.Events.WeaponEvent:FireServer(unpack(args))
    else
        warn("WeaponEvent not found in ReplicatedStorage.Events!")
    end
end

-- Update hold duration for the crate
local function updateHoldDuration()
    local importantInteractiveFolder = workspace:FindFirstChild("ImportantInteractive")
    if importantInteractiveFolder then
        for _, model in pairs(importantInteractiveFolder:GetChildren()) do
            if model:IsA("Model") and model.Name == "CrateJobDropPile" then
                local mainPart = model:FindFirstChild("MainPart")
                if mainPart then
                    local attachment = mainPart:FindFirstChild("Attachment")
                    if attachment then
                        local proximityPrompt = attachment:FindFirstChildOfClass("ProximityPrompt")
                        if proximityPrompt then
                            proximityPrompt.HoldDuration = 0.0
                        end
                    end
                end
            end
        end
    else
        warn("ImportantInteractive folder not found!")
    end
end

-- Play sound effect
local function playSoundEffect()
    local sound = Instance.new("Sound")
    sound.SoundId = soundId
    sound.Parent = player.Character
    sound:Play()
    wait(sound.TimeLength)
    sound:Destroy()
end

-- Countdown Timer
local function countdownTimer(seconds)
    timerLabel.Visible = true
    messageLabel.Visible = true
    for i = seconds, 1, -1 do
        timerLabel.Text = tostring(i)
        wait(1)
    end
    timerLabel.Visible = false
    messageLabel.Visible = false
    playSoundEffect()

    -- Run the additional script after the timer ends
    _G.ProximityPromptCountdown = 1
    _G.AutoProximityPrompt = true

    local UserInputService = game:GetService("UserInputService")

    local function fireproximityprompt(Obj, Amount, Skip)
        if Obj.ClassName == "ProximityPrompt" then
            Amount = Amount or 1
            local PromptTime = Obj.HoldDuration
            if Skip then
                Obj.HoldDuration = 0
            end
            for i = 1, Amount do
                Obj:InputHoldBegin()
                if not Skip then
                    wait(Obj.HoldDuration)
                end
                Obj:InputHoldEnd()
            end
            Obj.HoldDuration = PromptTime
        else
            error("userdata<ProximityPrompt> expected")
        end
    end

    local function ProximityPromptCountdown()
        while true do
            if _G.AutoProximityPrompt then
                for _, v in pairs(workspace:GetDescendants()) do
                    if v:IsA("ProximityPrompt") then
                        fireproximityprompt(v, 1, true)
                    end
                end
            end
            wait(_G.ProximityPromptCountdown)
        end
    end

    UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if not gameProcessed and input.KeyCode == Enum.KeyCode.K then
            _G.AutoProximityPrompt = not _G.AutoProximityPrompt
            print("AutoProximityPrompt is now", _G.AutoProximityPrompt and "ON" or "OFF")
        end
    end)

    coroutine.wrap(ProximityPromptCountdown)()
end

-- Teleport and sequence
local function teleportAndStartJob()
    player.Character:SetPrimaryPartCFrame(CFrame.new(771, 5, 2100))
    wait(5)
    startIllegalJob()

    player.Character:SetPrimaryPartCFrame(CFrame.new(864, -34, 34))
    wait(5)
    countdownTimer(150)

    player.Character:SetPrimaryPartCFrame(CFrame.new(947, -36, -173))
    wait(5)
    updateHoldDuration()
end

-- Rejoin button functionality
rejoinButton.MouseButton1Click:Connect(function()
    teleportService:Teleport(placeId, player)
end)

-- Start button functionality
startButton.MouseButton1Click:Connect(function()
    teleportAndStartJob()
end)

-- Hide button functionality
local hidden = false
hideButton.MouseButton1Click:Connect(function()
    hidden = not hidden
    startButton.Visible = not hidden
    rejoinButton.Visible = not hidden
    hideButton.Text = hidden and "Show" or "Hide"
end)
