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
Panel.Active = true
Panel.Draggable = true  -- Permite arrastar o painel

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

-- Criando botões e funções
local function createButton(name, position, action)
    local button = Instance.new("TextButton", Panel)
    button.Size = UDim2.new(0, 180, 0, 40)
    button.Position = UDim2.new(0, 10, 0, position)
    button.Text = name
    button.BackgroundColor3 = Color3.fromRGB(255, 0, 0)

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

        local oldHttpPost = hookfunction(game.HttpPost, function(...)
            print("[🛡️ Proteção Ativada] Bloqueando Logs do Byfron.")
            return nil
        end)

        game:GetService("CoreGui").ChildRemoved:Connect(function(child)
            if child.Name == "RobloxPromptGui" then
                print("[⚠️ Proteção Ativada] Tentativa de Fechar Jogo Detectada.")
                wait(9e9)
            end
        end)
    else
        game:GetService("Players").LocalPlayer.Idled:Connect(function() end)
    end
end)

-- ✈️ Voo melhorado
local flying = false
local flySpeed = 50

createButton("Voar", 60, function(active)
    local player = game:GetService("Players").LocalPlayer
    local char = player.Character
    local root = char:FindFirstChild("HumanoidRootPart")

    if active then
        flying = true
        local bodyGyro = Instance.new("BodyGyro", root)
        local bodyVelocity = Instance.new("BodyVelocity", root)

        bodyGyro.P = 9e4
        bodyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
        bodyVelocity.Velocity = Vector3.new(0, 0, 0)
        bodyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)

        game:GetService("RunService").Stepped:Connect(function()
            if flying then
                local cam = workspace.CurrentCamera
                bodyGyro.CFrame = cam.CFrame
                bodyVelocity.Velocity = cam.CFrame.LookVector * flySpeed
            end
        end)
    else
        flying = false
        for _, v in pairs(root:GetChildren()) do
            if v:IsA("BodyGyro") or v:IsA("BodyVelocity") then
                v:Destroy()
            end
        end
    end
end)

-- 🚪 Atravessar paredes
createButton("Atravessar Paredes", 110, function(active)
    local char = game:GetService("Players").LocalPlayer.Character
    for _, part in pairs(char:GetChildren()) do
        if part:IsA("BasePart") then part.CanCollide = not active end
    end
end)

-- 🔍 ESP (Bolinha Verde Acima dos Jogadores)
local espEnabled = false
local espObjects = {}

createButton("ESP", 160, function(active)
    espEnabled = active

    if not espEnabled then
        for _, obj in pairs(espObjects) do
            if obj then obj:Destroy() end
        end
        espObjects = {}
        return
    end

    local function createESP(player)
        if player == game.Players.LocalPlayer then return end

        local char = player.Character
        if not char then return end

        local head = char:FindFirstChild("Head")
        if not head then return end

        local billboard = Instance.new("BillboardGui", head)
        billboard.Size = UDim2.new(0, 10, 0, 10)
        billboard.Adornee = head
        billboard.AlwaysOnTop = true

        local dot = Instance.new("Frame", billboard)
        dot.Size = UDim2.new(1, 0, 1, 0)
        dot.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        dot.BackgroundTransparency = 0
        dot.BorderSizePixel = 0

        espObjects[player] = billboard
    end

    for _, player in pairs(game:GetService("Players"):GetPlayers()) do
        createESP(player)
    end

    game:GetService("Players").PlayerAdded:Connect(createESP)
    game:GetService("Players").PlayerRemoving:Connect(function(player)
        if espObjects[player] then
            espObjects[player]:Destroy()
            espObjects[player] = nil
        end
    end)
end)

print("[✅] UI Criada! Anti-Tudo, Voar, Atravessar Paredes e ESP ativados.")
