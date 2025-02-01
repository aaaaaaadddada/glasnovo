-- Script único no ServerScriptService

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")

-- Criando o RemoteEvent em ReplicatedStorage para comunicação entre cliente e servidor
local revelarClicked = Instance.new("RemoteEvent")
revelarClicked.Name = "RevelarClicked"
revelarClicked.Parent = ReplicatedStorage

-- Função para colorir as partes chamadas "part" de verde
local function colorirPartsDeVerde()
    local glassBridge = Workspace:FindFirstChild("GlassBridge")
    if not glassBridge then
        warn("Modelo 'GlassBridge' não encontrado no Workspace.")
        return
    end

    for _, bridge in pairs(glassBridge:GetChildren()) do
        if bridge:IsA("Model") and bridge.Name == "Bridge" then
            local glassBridgeModel = bridge:FindFirstChild("glassBridge")
            if glassBridgeModel then
                for _, part in pairs(glassBridgeModel:GetChildren()) do
                    if part:IsA("BasePart") and part.Name == "part" then
                        if part.Anchored and part.CanCollide then
                            part.BrickColor = BrickColor.new("Bright green")  -- Mudar a cor para verde
                        end
                    end
                end
            end
        end
    end
end

-- Função para esconder as partes
local function esconderParts()
    local glassBridge = Workspace:FindFirstChild("GlassBridge")
    if not glassBridge then
        warn("Modelo 'GlassBridge' não encontrado no Workspace.")
        return
    end

    for _, bridge in pairs(glassBridge:GetChildren()) do
        if bridge:IsA("Model") and bridge.Name == "Bridge" then
            local glassBridgeModel = bridge:FindFirstChild("glassBridge")
            if glassBridgeModel then
                for _, part in pairs(glassBridgeModel:GetChildren()) do
                    if part:IsA("BasePart") and part.Name == "part" then
                        part.BrickColor = BrickColor.new("Medium stone grey")  -- Cor padrão para esconder
                    end
                end
            end
        end
    end
end

-- Conectar o evento 'RevelarClicked' ao servidor
revelarClicked.OnServerEvent:Connect(function(player, reveal)
    if reveal then
        colorirPartsDeVerde()
    else
        esconderParts()
    end
end)

-- Criar a interface do usuário para os jogadores
local function criarUI(player)
    -- Criando a ScreenGui e a interface
    local screenGui = Instance.new("ScreenGui")
    screenGui.Parent = player:WaitForChild("PlayerGui")

    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Parent = screenGui
    mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    mainFrame.Position = UDim2.new(0.5, -100, 0.1, 0)
    mainFrame.Size = UDim2.new(0, 200, 0, 100)
    mainFrame.BorderSizePixel = 0
    mainFrame.Active = true
    mainFrame.Draggable = true

    local toggleButton = Instance.new("TextButton")
    toggleButton.Name = "ToggleButton"
    toggleButton.Parent = mainFrame
    toggleButton.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
    toggleButton.Size = UDim2.new(0, 180, 0, 40)
    toggleButton.Position = UDim2.new(0, 10, 0, 10)
    toggleButton.Font = Enum.Font.SourceSans
    toggleButton.Text = "Revelar"
    toggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    toggleButton.TextSize = 24

    -- Variável para controlar o estado do botão (revelar ou esconder)
    local isRevealed = false

    -- Alternar entre revelar e esconder
    toggleButton.MouseButton1Click:Connect(function()
        isRevealed = not isRevealed
        revelarClicked:FireServer(isRevealed)  -- Envia o evento para o servidor
        toggleButton.Text = isRevealed and "Esconder" or "Revelar"  -- Muda o texto do botão
    end)
end

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        criarUI(player)
    end)
end)
