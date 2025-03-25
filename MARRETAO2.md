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
    button.BackgroundColor3 = Color3.fromRGB(255, 0, 0) -- Vermelho = desativado

    local active = false
    button.MouseButton1Click:Connect(function()
        active = not active
        button.BackgroundColor3 = active and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
        action(active)
    end)
end

-- 🔰 Atravessar paredes corrigido
createButton("Atravessar Paredes", 10, function(active)
    local char = game:GetService("Players").LocalPlayer.Character
    for _, part in pairs(char:GetChildren()) do
        if part:IsA("BasePart") then
            part.CanCollide = not active
        end
    end

    -- Correção para evitar bug de não atravessar depois de um tempo
    if active then
        game:GetService("RunService").Stepped:Connect(function()
            for _, part in pairs(char:GetChildren()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end)
    end
end)

-- ✈️ Voo corrigido
createButton("Voar", 60, function(active)
    local player = game:GetService("Players").LocalPlayer
    local char = player.Character
    local humanoid = char:FindFirstChildOfClass("Humanoid")

    if active then
        local fly = Instance.new("BodyVelocity", char.HumanoidRootPart)
        fly.Velocity = Vector3.new(0, 0, 0)
        fly.MaxForce = Vector3.new(4000, 4000, 4000)

        local connection
        connection = game:GetService("RunService").Stepped:Connect(function()
            if not active then
                connection:Disconnect()
                fly:Destroy()
                return
            end

            local direction = humanoid.MoveDirection
            if direction.Magnitude > 0 then
                fly.Velocity = direction * 50
            else
                fly.Velocity = Vector3.new(0, 0, 0)
            end
        end)
    else
        for _, v in pairs(char.HumanoidRootPart:GetChildren()) do
            if v:IsA("BodyVelocity") then
                v:Destroy()
            end
        end
    end
end)

-- 👀 ESP corrigido
local ESPEnabled = false
local ESPObjects = {}

createButton("ESP", 110, function(active)
    ESPEnabled = active

    -- Remover ESP quando desativado
    if not ESPEnabled then
        for _, obj in pairs(ESPObjects) do
            obj:Destroy()
        end
        ESPObjects = {}
        return
    end

    -- Criar ESP
    local function createESP(player)
        if player == game.Players.LocalPlayer then return end -- Não adicionar ESP em si mesmo

        local char = player.Character
        if char then
            local head = char:FindFirstChild("Head")
            if head then
                local esp = Instance.new("BillboardGui", head)
                esp.Size = UDim2.new(0, 10, 0, 10) -- Tamanho pequeno da bolinha
                esp.Adornee = head
                esp.AlwaysOnTop = true

                local dot = Instance.new("Frame", esp)
                dot.Size = UDim2.new(1, 0, 1, 0)
                dot.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
                dot.BackgroundTransparency = 0
                dot.BorderSizePixel = 0

                ESPObjects[player] = esp
            end
        end
    end

    -- Ativar ESP para jogadores existentes
    for _, player in pairs(game.Players:GetPlayers()) do
        createESP(player)
    end

    -- Atualizar ESP para novos jogadores
    game.Players.PlayerAdded:Connect(createESP)
    game.Players.PlayerRemoving:Connect(function(player)
        if ESPObjects[player] then
            ESPObjects[player]:Destroy()
            ESPObjects[player] = nil
        end
    end)
end)

print("[✅] UI Criada! Atravessar Paredes, Voar e ESP corrigidos.")
