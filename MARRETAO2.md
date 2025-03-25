-- Criando a interface flutuante
local ScreenGui = Instance.new("ScreenGui", game:GetService("CoreGui"))
local FloatingButton = Instance.new("TextButton", ScreenGui)

FloatingButton.Size = UDim2.new(0, 80, 0, 30)
FloatingButton.Position = UDim2.new(0.9, 0, 0.1, 0)
FloatingButton.Text = "SCRIPT"
FloatingButton.BackgroundColor3 = Color3.fromRGB(0, 0, 255)

local Panel = Instance.new("Frame", ScreenGui)
Panel.Size = UDim2.new(0, 200, 0, 250)
Panel.Position = UDim2.new(0.75, 0, 0.1, 0)
Panel.Visible = false
Panel.BackgroundColor3 = Color3.fromRGB(50, 50, 50)

local CloseButton = Instance.new("TextButton", Panel)
CloseButton.Size = UDim2.new(0, 30, 0, 30)
CloseButton.Position = UDim2.new(1, -30, 0, 0)
CloseButton.Text = "X"
CloseButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)

FloatingButton.MouseButton1Click:Connect(function()
    Panel.Visible = not Panel.Visible
end)

CloseButton.MouseButton1Click:Connect(function()
    Panel.Visible = false
end)

-- Tornar painel arrastável
local UIS = game:GetService("UserInputService")
local dragging, dragInput, dragStart, startPos

Panel.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = Panel.Position

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

Panel.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

UIS.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        Panel.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- Criar botões
local function createButton(name, position, action)
    local button = Instance.new("TextButton", Panel)
    button.Size = UDim2.new(0, 180, 0, 40)
    button.Position = UDim2.new(0, 10, 0, position)
    button.Text = name
    button.BackgroundColor3 = Color3.fromRGB(255, 0, 0) -- Vermelho = desativado

    local active = false
    button.MouseButton1Click:Connect(function()
        active = not active
        button.BackgroundColor3 = active and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
        action(active)
    end)
end

-- ✈️ Voo melhorado
createButton("Voar", 10, function(active)
    local player = game.Players.LocalPlayer
    local char = player.Character
    local humanoid = char and char:FindFirstChildOfClass("Humanoid")
    
    if not humanoid then return end
    
    if active then
        local bodyGyro = Instance.new("BodyGyro", char.HumanoidRootPart)
        local bodyVelocity = Instance.new("BodyVelocity", char.HumanoidRootPart)
        bodyGyro.P = 9e4
        bodyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
        bodyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)
        
        game:GetService("RunService").Stepped:Connect(function()
            if active then
                bodyGyro.CFrame = workspace.CurrentCamera.CFrame
                bodyVelocity.Velocity = workspace.CurrentCamera.CFrame.LookVector * 50
            else
                bodyGyro:Destroy()
                bodyVelocity:Destroy()
            end
        end)
    end
end)

-- 👻 Invisibilidade real
createButton("Invisível", 60, function(active)
    local player = game.Players.LocalPlayer
    local char = player.Character

    if active then
        for _, part in pairs(char:GetChildren()) do
            if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
                part.Transparency = 1
            end
        end
        if char:FindFirstChild("Head") then
            for _, v in pairs(char.Head:GetChildren()) do
                if v:IsA("BillboardGui") then
                    v.Enabled = false
                end
            end
        end
    else
        for _, part in pairs(char:GetChildren()) do
            if part:IsA("BasePart") then
                part.Transparency = 0
            end
        end
        if char:FindFirstChild("Head") then
            for _, v in pairs(char.Head:GetChildren()) do
                if v:IsA("BillboardGui") then
                    v.Enabled = true
                end
            end
        end
    end
end)

-- 🔍 ESP (Ver jogadores pelas paredes)
createButton("ESP", 110, function(active)
    local function createESP(player)
        if player == game.Players.LocalPlayer then return end
        if not player.Character then return end

        local box = Instance.new("BoxHandleAdornment")
        box.Size = Vector3.new(4, 6, 2) -- Tamanho do quadrado
        box.Color3 = Color3.fromRGB(0, 255, 0) -- Cor verde brilhante
        box.Transparency = 0
        box.ZIndex = 0
        box.AlwaysOnTop = true
        box.Adornee = player.Character:FindFirstChild("HumanoidRootPart")
        box.Parent = game.CoreGui
        return box
    end

    if active then
        for _, plr in pairs(game.Players:GetPlayers()) do
            if plr ~= game.Players.LocalPlayer then
                local esp = createESP(plr)
                if esp then
                    plr.CharacterAdded:Connect(function()
                        esp = createESP(plr)
                    end)
                end
            end
        end
    else
        for _, obj in pairs(game.CoreGui:GetChildren()) do
            if obj:IsA("BoxHandleAdornment") then
                obj:Destroy()
            end
        end
    end
end)

print("[✅] UI Criada! Voar, Invisibilidade e ESP funcionando.")
