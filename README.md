-- Plataforma Elevadora - Versão Ultra Leve (Edward)
-- Otimizado sem invisibilidade e sem anti-hit

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

local platformEnabled = false
local platformSpeed = 2
local platform = nil
local connection = nil
local platformOffset = -3
local settingsLocked = false

-- Sistema de Key (Keys atualizadas)
local correctKeys = {
    "edwardv2",
    "edward1235", 
    "netov2",
    "adm5"
}
local keyVerified = false
local attempts = 0
local maxAttempts = 5

-- Sistema de Auto Click Simples
local autoClickEnabled = false
local autoClickConnection = nil
local clickSpeed = 0.1
local clickCount = 0

-- GUI Principal
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "PlatformGUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

-----------------------------------------------------------
-- JANELA DE KEY
-----------------------------------------------------------

local keyFrame = Instance.new("Frame")
keyFrame.Size = UDim2.new(0, 300, 0, 200)
keyFrame.Position = UDim2.new(0.5, -150, 0.5, -100)
keyFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
keyFrame.BorderSizePixel = 2
keyFrame.BorderColor3 = Color3.fromRGB(255, 50, 50)
keyFrame.Parent = screenGui

local keyCorner = Instance.new("UICorner")
keyCorner.CornerRadius = UDim.new(0, 10)
keyCorner.Parent = keyFrame

local keyTitle = Instance.new("TextLabel")
keyTitle.Size = UDim2.new(1, -20, 0, 40)
keyTitle.Position = UDim2.new(0, 10, 0, 10)
keyTitle.BackgroundTransparency = 1
keyTitle.Text = "SISTEMA EDWARD OFICIAL"
keyTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
keyTitle.TextSize = 18
keyTitle.Font = Enum.Font.GothamBold
keyTitle.Parent = keyFrame

local keyInstruction = Instance.new("TextLabel")
keyInstruction.Size = UDim2.new(1, -20, 0, 30)
keyInstruction.Position = UDim2.new(0, 10, 0, 50)
keyInstruction.BackgroundTransparency = 1
keyInstruction.Text = "Digite a key para acessar:"
keyInstruction.TextColor3 = Color3.fromRGB(200, 200, 200)
keyInstruction.TextSize = 14
keyInstruction.Font = Enum.Font.Gotham
keyInstruction.Parent = keyFrame

local keyTextBox = Instance.new("TextBox")
keyTextBox.Size = UDim2.new(1, -20, 0, 35)
keyTextBox.Position = UDim2.new(0, 10, 0, 85)
keyTextBox.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
keyTextBox.TextColor3 = Color3.fromRGB(255, 255, 255)
keyTextBox.PlaceholderText = "Digite a key..."
keyTextBox.TextSize = 14
keyTextBox.Font = Enum.Font.Gotham
keyTextBox.ClearTextOnFocus = false
keyTextBox.Parent = keyFrame

local verifyButton = Instance.new("TextButton")
verifyButton.Size = UDim2.new(1, -20, 0, 35)
verifyButton.Position = UDim2.new(0, 10, 0, 130)
verifyButton.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
verifyButton.TextColor3 = Color3.fromRGB(255, 255, 255)
verifyButton.Text = "VERIFICAR"
verifyButton.Font = Enum.Font.GothamBold
verifyButton.TextSize = 14
verifyButton.BorderSizePixel = 0
verifyButton.Parent = keyFrame

local keyStatus = Instance.new("TextLabel")
keyStatus.Size = UDim2.new(1, -20, 0, 20)
keyStatus.Position = UDim2.new(0, 10, 0, 170)
keyStatus.BackgroundTransparency = 1
keyStatus.Text = "Tentativas: 0/" .. maxAttempts
keyStatus.TextColor3 = Color3.fromRGB(255, 255, 255)
keyStatus.Font = Enum.Font.Gotham
keyStatus.TextSize = 12
keyStatus.Parent = keyFrame

-----------------------------------------------------------
-- ABA PRINCIPAL
-----------------------------------------------------------

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 250, 0, 220)
mainFrame.Position = UDim2.new(1, -270, 0.5, -110)
mainFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
mainFrame.BorderSizePixel = 2
mainFrame.BorderColor3 = Color3.fromRGB(0, 170, 255)
mainFrame.Visible = false
mainFrame.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 10)
corner.Parent = mainFrame

local closeButton = Instance.new("TextButton")
closeButton.Size = UDim2.new(0, 25, 0, 25)
closeButton.Position = UDim2.new(1, -30, 0, 10)
closeButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
closeButton.Text = "X"
closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
closeButton.Font = Enum.Font.GothamBold
closeButton.TextSize = 14
closeButton.BorderSizePixel = 0
closeButton.Parent = mainFrame

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -40, 0, 30)
title.Position = UDim2.new(0, 10, 0, 10)
title.BackgroundTransparency = 1
title.Text = "Plataforma Elevadora"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextSize = 16
title.Font = Enum.Font.GothamBold
title.Parent = mainFrame

local speedLabel = Instance.new("TextLabel")
speedLabel.Size = UDim2.new(1, -20, 0, 25)
speedLabel.Position = UDim2.new(0, 10, 0, 50)
speedLabel.BackgroundTransparency = 1
speedLabel.Text = "Velocidade: " .. platformSpeed
speedLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
speedLabel.TextSize = 14
speedLabel.Font = Enum.Font.Gotham
speedLabel.TextXAlignment = Enum.TextXAlignment.Left
speedLabel.Parent = mainFrame

-- Slider
local sliderFrame = Instance.new("Frame")
sliderFrame.Size = UDim2.new(1, -20, 0, 20)
sliderFrame.Position = UDim2.new(0, 10, 0, 80)
sliderFrame.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
sliderFrame.BorderSizePixel = 0
sliderFrame.Parent = mainFrame

local sliderButton = Instance.new("TextButton")
sliderButton.Size = UDim2.new(0, 30, 1, 0)
sliderButton.Position = UDim2.new((platformSpeed - 1) / 9, -15, 0, 0)
sliderButton.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
sliderButton.Text = ""
sliderButton.BorderSizePixel = 0
sliderButton.Parent = sliderFrame

local lockButton = Instance.new("TextButton")
lockButton.Size = UDim2.new(1, -20, 0, 35)
lockButton.Position = UDim2.new(0, 10, 0, 110)
lockButton.BackgroundColor3 = Color3.fromRGB(255, 200, 0)
lockButton.Text = "🔓 Destravar Velocidade"
lockButton.TextColor3 = Color3.fromRGB(255, 255, 255)
lockButton.TextSize = 14
lockButton.Font = Enum.Font.GothamBold
lockButton.BorderSizePixel = 0
lockButton.Parent = mainFrame

-- Botão Auto Click
local autoClickButton = Instance.new("TextButton")
autoClickButton.Size = UDim2.new(1, -20, 0, 35)
autoClickButton.Position = UDim2.new(0, 10, 0, 155)
autoClickButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
autoClickButton.Text = "🔴 Auto Click: OFF"
autoClickButton.TextColor3 = Color3.fromRGB(255, 255, 255)
autoClickButton.TextSize = 14
autoClickButton.Font = Enum.Font.GothamBold
autoClickButton.BorderSizePixel = 0
autoClickButton.Parent = mainFrame

-- Status do Auto Click
local autoClickStatus = Instance.new("TextLabel")
autoClickStatus.Size = UDim2.new(1, -20, 0, 20)
autoClickStatus.Position = UDim2.new(0, 10, 0, 195)
autoClickStatus.BackgroundTransparency = 1
autoClickStatus.Text = "Cliques: 0"
autoClickStatus.TextColor3 = Color3.fromRGB(200, 200, 200)
autoClickStatus.TextSize = 12
autoClickStatus.Font = Enum.Font.Gotham
autoClickStatus.TextXAlignment = Enum.TextXAlignment.Left
autoClickStatus.Parent = mainFrame

-----------------------------------------------------------
-- BOTÕES EXTERNOS
-----------------------------------------------------------

local toggleMenuButton = Instance.new("TextButton")
toggleMenuButton.Size = UDim2.new(0, 50, 0, 50)
toggleMenuButton.Position = UDim2.new(1, -60, 0.5, -25)
toggleMenuButton.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
toggleMenuButton.Text = "☰"
toggleMenuButton.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleMenuButton.TextSize = 20
toggleMenuButton.Font = Enum.Font.GothamBold
toggleMenuButton.BorderSizePixel = 0
toggleMenuButton.Visible = false
toggleMenuButton.Parent = screenGui

local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(0, 100, 0, 40)
toggleButton.Position = UDim2.new(1, -110, 0, 70)
toggleButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
toggleButton.Text = "Ativar"
toggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleButton.TextSize = 16
toggleButton.Font = Enum.Font.GothamBold
toggleButton.BorderSizePixel = 0
toggleButton.Visible = false
toggleButton.Parent = screenGui

-----------------------------------------------------------
-- FUNÇÕES PRINCIPAIS
-----------------------------------------------------------

local function verifyKey(inputKey)
    if attempts >= maxAttempts then return end
    
    attempts += 1
    keyStatus.Text = "Tentativas: " .. attempts .. "/" .. maxAttempts

    local isValidKey = false
    for _, key in ipairs(correctKeys) do
        if inputKey == key then
            isValidKey = true
            break
        end
    end

    if isValidKey then
        keyVerified = true
        keyStatus.Text = "KEY CORRETA!"
        keyStatus.TextColor3 = Color3.fromRGB(0, 255, 100)
        task.wait(1)
        keyFrame.Visible = false
        toggleMenuButton.Visible = true
        toggleButton.Visible = true
    else
        keyStatus.TextColor3 = Color3.fromRGB(255, 50, 50)
        keyStatus.Text = "INCORRETA!"

        if attempts >= maxAttempts then
            verifyButton.Active = false
            verifyButton.Text = "BLOQUEADO"
            keyTextBox.Active = false
            keyTextBox.PlaceholderText = "BLOQUEADO"
        end
    end
end

verifyButton.MouseButton1Click:Connect(function()
    if attempts < maxAttempts then
        verifyKey(keyTextBox.Text)
    end
end)

keyTextBox.FocusLost:Connect(function(enter)
    if enter and attempts < maxAttempts then
        verifyKey(keyTextBox.Text)
    end
end)

-- Criar plataforma
local function createPlatform()
    if platform then 
        platform:Destroy()
        platform = nil
    end
    
    platform = Instance.new("Part")
    platform.Size = Vector3.new(6, 0.5, 6)
    platform.Anchored = true
    platform.CanCollide = true
    platform.Material = Enum.Material.Neon
    platform.Color = Color3.fromRGB(0, 170, 255)
    platform.Transparency = 0.3
    platform.Name = "EdwardPlatform"
    platform.Parent = workspace
end

-- Sistema de Auto Click Simples
local lastClickTime = 0

local function simulateClick()
    pcall(function()
        VirtualInputManager:SendMouseButtonEvent(0, 0, 0, true, game, 1)
        task.wait(0.01)
        VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 1)
        
        clickCount += 1
        autoClickStatus.Text = "Cliques: " .. clickCount
    end)
end

local function autoClick()
    if not autoClickEnabled then return end
    
    local currentTime = tick()
    if currentTime - lastClickTime >= clickSpeed then
        simulateClick()
        lastClickTime = currentTime
    end
end

local function toggleAutoClick()
    if not keyVerified then return end
    
    autoClickEnabled = not autoClickEnabled
    
    if autoClickEnabled then
        autoClickButton.Text = "🟢 Auto Click: ON"
        autoClickButton.BackgroundColor3 = Color3.fromRGB(50, 255, 50)
        autoClickStatus.Text = "Cliques: " .. clickCount
        autoClickStatus.TextColor3 = Color3.fromRGB(100, 255, 100)
        
        if autoClickConnection then
            autoClickConnection:Disconnect()
        end
        autoClickConnection = RunService.Heartbeat:Connect(autoClick)
    else
        autoClickButton.Text = "🔴 Auto Click: OFF"
        autoClickButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
        autoClickStatus.TextColor3 = Color3.fromRGB(200, 200, 200)
        
        if autoClickConnection then
            autoClickConnection:Disconnect()
            autoClickConnection = nil
        end
    end
end

autoClickButton.MouseButton1Click:Connect(toggleAutoClick)

-- Atualizar plataforma
local dragging = false

local function updateSlider()
    if dragging and not settingsLocked then
        local mouse = player:GetMouse()
        local rX = math.clamp(mouse.X - sliderFrame.AbsolutePosition.X, 0, sliderFrame.AbsoluteSize.X)
        local pct = rX / sliderFrame.AbsoluteSize.X
        platformSpeed = math.floor(1 + pct * 9 + 0.5)
        sliderButton.Position = UDim2.new(pct, -15, 0, 0)
        speedLabel.Text = "Velocidade: " .. platformSpeed
    end
end

RunService.RenderStepped:Connect(updateSlider)

sliderButton.MouseButton1Down:Connect(function()
    if not settingsLocked then 
        dragging = true 
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then 
        dragging = false 
    end
end)

lockButton.MouseButton1Click:Connect(function()
    settingsLocked = not settingsLocked
    if settingsLocked then
        lockButton.Text = "🔒 Travado"
        lockButton.BackgroundColor3 = Color3.fromRGB(255,50,50)
    else
        lockButton.Text = "🔓 Destravado"
        lockButton.BackgroundColor3 = Color3.fromRGB(255,200,0)
    end
end)

-- Ativar plataforma
toggleButton.MouseButton1Click:Connect(function()
    if not keyVerified then return end

    platformEnabled = not platformEnabled

    if platformEnabled then
        toggleButton.Text = "Desativar"
        toggleButton.BackgroundColor3 = Color3.fromRGB(50, 255, 50)

        platformOffset = -3
        createPlatform()

        if connection then
            connection:Disconnect()
        end
        connection = RunService.Heartbeat:Connect(function(dt)
            if platform and humanoidRootPart then
                platformOffset = platformOffset + (platformSpeed / 10) * dt
                platform.Position = humanoidRootPart.Position + Vector3.new(0, platformOffset, 0)
            end
        end)
    else
        toggleButton.Text = "Ativar"
        toggleButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)

        if connection then 
            connection:Disconnect()
            connection = nil
        end
        if platform then 
            platform:Destroy()
            platform = nil
        end
    end
end)

toggleMenuButton.MouseButton1Click:Connect(function()
    mainFrame.Visible = not mainFrame.Visible
    toggleMenuButton.Text = mainFrame.Visible and "✕" or "☰"
end)

closeButton.MouseButton1Click:Connect(function()
    mainFrame.Visible = false
    toggleMenuButton.Text = "☰"
end)

-- Reset quando o personagem morrer
player.CharacterAdded:Connect(function(char)
    character = char
    humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    platformOffset = -3
    if platformEnabled then 
        createPlatform() 
    end
end)

-- Limpeza quando o script for encerrado
game:GetService("UserInputService").WindowFocusReleased:Connect(function()
    if autoClickConnection then
        autoClickConnection:Disconnect()
        autoClickConnection = nil
    end
    if connection then
        connection:Disconnect()
        connection = nil
    end
end)
