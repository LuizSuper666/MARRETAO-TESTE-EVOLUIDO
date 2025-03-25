-- Criando a interface flutuante
local ScreenGui = Instance.new("ScreenGui", game:GetService("CoreGui"))
local FloatingButton = Instance.new("TextButton", ScreenGui)

FloatingButton.Size = UDim2.new(0, 80, 0, 30)
FloatingButton.Position = UDim2.new(0.9, 0, 0.1, 0)
FloatingButton.Text = "SCRIPT"
FloatingButton.BackgroundColor3 = Color3.fromRGB(0, 0, 255)

local Panel = Instance.new("Frame", ScreenGui)
Panel.Size = UDim2.new(0, 200, 0, 200)
Panel.Position = UDim2.new(0.75, 0, 0.1, 0)
Panel.Visible = false
Panel.BackgroundColor3 = Color3.fromRGB(50, 50, 50)

-- Permitir arrastar o painel
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
        Panel.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

FloatingButton.MouseButton1Click:Connect(function()
    Panel.Visible = not Panel.Visible
end)

-- Criando botões e funções
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

-- 🔰 Anti-Tudo
createButton("Anti-Tudo", 10, function(active)
    if active then
        local mt = getrawmetatable(game)
        setreadonly(mt, false)
        local oldNamecall = mt.__namecall
        mt.__namecall = newcclosure(function(self, ...)
            local method = getnamecallmethod()
            if method == "Kick" or method == "kick" then return nil end
            return oldNamecall(self, ...)
        end)

        local Players = game:GetService("Players")
        for _, v in pairs(getconnections(Players.LocalPlayer.Idled)) do v:Disable() end
    end
end)

-- ✈️ Voar (Novo sistema, segue a câmera)
local flying = false
local speed = 50
local flyBodyVelocity
local flyGyro

createButton("Voar", 60, function(active)
    local player = game:GetService("Players").LocalPlayer
    local char = player.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")

    if active and root then
        flying = true
        flyBodyVelocity = Instance.new("BodyVelocity", root)
        flyBodyVelocity.Velocity = Vector3.new(0, 0, 0)
        flyBodyVelocity.MaxForce = Vector3.new(4000, 4000, 4000)

        flyGyro = Instance.new("BodyGyro", root)
        flyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
        flyGyro.CFrame = root.CFrame

        game:GetService("RunService").RenderStepped:Connect(function()
            if not flying then return end
            local cam = workspace.CurrentCamera
            flyGyro.CFrame = cam.CFrame
            flyBodyVelocity.Velocity = cam.CFrame.LookVector * speed
        end)
    else
        flying = false
        if flyBodyVelocity then flyBodyVelocity:Destroy() end
        if flyGyro then flyGyro:Destroy() end
    end
end)

-- 🚪 Atravessar paredes (Corrigido: Agora sempre funciona)
local atravessarAtivo = false

createButton("Atravessar Paredes", 110, function(active)
    local char = game:GetService("Players").LocalPlayer.Character
    atravessarAtivo = active

    game:GetService("RunService").Stepped:Connect(function()
        if not atravessarAtivo then return end
        if char then
            for _, part in pairs(char:GetChildren()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end
    end)
end)

-- 🔍 ESP (Bolinha verde em cima dos jogadores)
local espAtivo = false
local players = game:GetService("Players")

createButton("ESP", 160, function(active)
    espAtivo = active

    for _, player in pairs(players:GetPlayers()) do
        if player ~= players.LocalPlayer then
            local char = player.Character
            if char and char:FindFirstChild("Head") then
                if active then
                    local billboard = Instance.new("BillboardGui", char.Head)
                    billboard.Size = UDim2.new(0, 50, 0, 50)
                    billboard.Adornee = char.Head
                    billboard.Name = "ESPTag"
                    billboard.AlwaysOnTop = true

                    local frame = Instance.new("Frame", billboard)
                    frame.Size = UDim2.new(1, 0, 1, 0)
                    frame.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
                    frame.BackgroundTransparency = 0.2
                    frame.BorderSizePixel = 0
                    frame.Shape = Enum.PartType.Ball
                else
                    if char:FindFirstChild("ESPTag") then
                        char.ESPTag:Destroy()
                    end
                end
            end
        end
    end
end)

print("[✅] UI Criada! Agora o voo e o atravessar paredes funcionam sem bugs.")
