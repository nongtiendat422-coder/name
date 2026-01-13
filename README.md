local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local StarterGui = game:GetService("StarterGui")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local threshold = 15  -- % HP thấp kích hoạt
local recoverThreshold = 40  -- % HP để quay lại
local skyHeight = 1500  -- Chiều cao dịch chuyển
local hoverHeight = 100  -- Chiều cao hover
local isSafeMode = false
local autoEnabled = false  -- Toggle auto
local bodyPosition = nil
local heartbeatConnection = nil

-- Tạo GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AntiDieGUI"
screenGui.Parent = playerGui
screenGui.ResetOnSpawn = false

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 250, 0, 200)
mainFrame.Position = UDim2.new(0, 10, 0.5, -100)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 10)
corner.Parent = mainFrame

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 40)
title.Position = UDim2.new(0, 0, 0, 0)
title.BackgroundTransparency = 1
title.Text = "🛡️ ANTI-DIE BLOX FRUITS"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextScaled = true
title.Font = Enum.Font.GothamBold
title.Parent = mainFrame

local autoToggleBtn = Instance.new("TextButton")
autoToggleBtn.Size = UDim2.new(0.9, 0, 0, 45)
autoToggleBtn.Position = UDim2.new(0.05, 0, 0, 50)
autoToggleBtn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
autoToggleBtn.Text = "AUTO SAFE: OFF"
autoToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
autoToggleBtn.TextScaled = true
autoToggleBtn.Font = Enum.Font.Gotham
autoToggleBtn.Parent = mainFrame

local toggleCorner = Instance.new("UICorner")
toggleCorner.CornerRadius = UDim.new(0, 8)
toggleCorner.Parent = autoToggleBtn

local manualBtn = Instance.new("TextButton")
manualBtn.Size = UDim2.new(0.9, 0, 0, 45)
manualBtn.Position = UDim2.new(0.05, 0, 0, 105)
manualBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
manualBtn.Text = "MANUAL SAFE NOW"
manualBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
manualBtn.TextScaled = true
manualBtn.Font = Enum.Font.Gotham
manualBtn.Parent = mainFrame

local manualCorner = Instance.new("UICorner")
manualCorner.CornerRadius = UDim.new(0, 8)
manualCorner.Parent = manualBtn

local hpLabel = Instance.new("TextLabel")
hpLabel.Size = UDim2.new(0.9, 0, 0, 30)
hpLabel.Position = UDim2.new(0.05, 0, 0, 160)
hpLabel.BackgroundTransparency = 1
hpLabel.Text = "HP: 100%"
hpLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
hpLabel.TextScaled = true
hpLabel.Font = Enum.Font.Gotham
hpLabel.Parent = mainFrame

-- Functions giống script cũ
local function makeInvisible(character)
    for _, obj in pairs(character:GetDescendants()) do
        if obj:IsA("BasePart") and obj ~= character:FindFirstChild("HumanoidRootPart") then
            obj.Transparency = 1
            obj.CanCollide = false
        elseif obj:IsA("Decal") or obj:IsA("Texture") or obj:IsA("SpecialMesh") then
            obj.Transparency = 1
        end
    end
    for _, acc in pairs(character:GetChildren()) do
        if acc:IsA("Accessory") then
            local handle = acc:FindFirstChild("Handle")
            if handle then
                handle.Transparency = 1
                handle.CanCollide = false
            end
        end
    end
end

local function makeVisible(character)
    for _, obj in pairs(character:GetDescendants()) do
        if obj:IsA("BasePart") and obj ~= character:FindFirstChild("HumanoidRootPart") then
            obj.Transparency = 0
            obj.CanCollide = true
        elseif obj:IsA("Decal") or obj:IsA("Texture") or obj:IsA("SpecialMesh") then
            obj.Transparency = 0
        end
    end
    for _, acc in pairs(character:GetChildren()) do
        if acc:IsA("Accessory") then
            local handle = acc:FindFirstChild("Handle")
            if handle then
                handle.Transparency = 0
                handle.CanCollide = false
            end
        end
    end
end

local function toggleSafeMode(character, enable)
    local humanoid = character:WaitForChild("Humanoid")
    local root = character:WaitForChild("HumanoidRootPart")
    
    if enable then
        local skyPos = root.Position + Vector3.new(0, skyHeight, 0)
        root.CFrame = CFrame.new(skyPos)
        
        bodyPosition = Instance.new("BodyPosition")
        bodyPosition.MaxForce = Vector3.new(4000, 4000, 4000)
        bodyPosition.Position = root.Position + Vector3.new(0, hoverHeight, 0)
        bodyPosition.D = 1000
        bodyPosition.P = 3000
        bodyPosition.Parent = root
        
        makeInvisible(character)
        humanoid.PlatformStand = true
        humanoid.WalkSpeed = 0
        humanoid.JumpPower = 0
        
        isSafeMode = true
        print("🛡️ SAFE MODE ON!")
        
        local tween = TweenService:Create(bodyPosition, TweenInfo.new(1, Enum.EasingStyle.Quad), {Position = root.Position})
        tween:Play()
        
    else
        if bodyPosition then
            bodyPosition:Destroy()
            bodyPosition = nil
        end
        
        makeVisible(character)
        humanoid.PlatformStand = false
        humanoid.WalkSpeed = 16
        humanoid.JumpPower = 50
        
        isSafeMode = false
        print("✅ SAFE MODE OFF!")
    end
end

local function updateHpLabel(character)
    local humanoid = character:FindFirstChild("Humanoid")
    if humanoid then
        local hpPercent = math.floor((humanoid.Health / humanoid.MaxHealth) * 100)
        hpLabel.Text = "HP: " .. hpPercent .. "%"
        if hpPercent < 30 then
            hpLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
        elseif hpPercent < 60 then
            hpLabel.TextColor3 = Color3.fromRGB(255, 165, 0)
        else
            hpLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
        end
    end
end

local function startMonitoring(character)
    local humanoid = character:WaitForChild("Humanoid")
    
    if heartbeatConnection then
        heartbeatConnection:Disconnect()
    end
    
    heartbeatConnection = RunService.Heartbeat:Connect(function()
        if humanoid.Health <= 0 then return end
        
        updateHpLabel(character)
        
        local hpPercent = (humanoid.Health / humanoid.MaxHealth) * 100
        
        if autoEnabled and hpPercent < threshold and not isSafeMode then
            toggleSafeMode(character, true)
        elseif hpPercent > recoverThreshold and isSafeMode then
            toggleSafeMode(character, false)
        end
    end)
end

local function onCharacterAdded(character)
    isSafeMode = false
    if bodyPosition then
        bodyPosition:Destroy()
        bodyPosition = nil
    end
    
    wait(1)  -- Đợi character load đầy đủ
    startMonitoring(character)
    
    character.AncestryChanged:Connect(function()
        if not character.Parent then
            if heartbeatConnection then
                heartbeatConnection:Disconnect()
            end
            if bodyPosition then
                bodyPosition:Destroy()
            end
        end
    end)
end

-- GUI Events
autoToggleBtn.MouseButton1Click:Connect(function()
    autoEnabled = not autoEnabled
    if autoEnabled then
        autoToggleBtn.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        autoToggleBtn.Text = "AUTO SAFE: ON"
        print("🔄 Auto Safe Mode: BẬT")
    else
        autoToggleBtn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
        autoToggleBtn.Text = "AUTO SAFE: OFF"
        print("🔄 Auto Safe Mode: TẮT")
    end
end)

manualBtn.MouseButton1Click:Connect(function()
    local character = player.Character
    if character then
        toggleSafeMode(character, not isSafeMode)
    end
end)

-- Hotkey Toggle Auto (Insert)
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.Insert then
        autoToggleBtn.MouseButton1Click:Fire()
    end
end)

-- Khởi động
if player.Character then
    onCharacterAdded(player.Character)
end
player.CharacterAdded:Connect(onCharacterAdded)

-- Notify
StarterGui:SetCore("SendNotification", {
    Title = "Anti-Die GUI Loaded! 🛡️";
    Text = "INSERT: Toggle | Drag GUI | HP dưới 15% auto safe";
    Duration = 5;
})

print("🚀 Anti-Die GUI Blox Fruits đã load! Có nút đàng hoàng rồi nhé! 😎")
