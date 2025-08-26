-- Evitar múltiplas execuções
if _G.MauroHubLoaded then return end
_G.MauroHubLoaded = true

local UIS = game:GetService("UserInputService")
local Players = game:GetService("Players")
local player = Players.LocalPlayer

-- Criar ScreenGui
local gui = Instance.new("ScreenGui")
gui.Name = "MauroHubGui"
gui.ResetOnSpawn = false
gui.Parent = game:GetService("CoreGui")

-- Bolinha inicial (abrir/fechar GUI)
local toggleButton = Instance.new("ImageButton")
toggleButton.Name = "ToggleButton"
toggleButton.Size = UDim2.new(0, 60, 0, 60)
toggleButton.Position = UDim2.new(0, 100, 0, 200)
toggleButton.BackgroundColor3 = Color3.fromRGB(139, 69, 19) -- marrom
toggleButton.Image = "rbxassetid://13252850408"
toggleButton.Active = true
toggleButton.Parent = gui

local uiCorner = Instance.new("UICorner", toggleButton)
uiCorner.CornerRadius = UDim.new(1, 0)

local uiStroke = Instance.new("UIStroke", toggleButton)
uiStroke.Thickness = 4
uiStroke.Color = Color3.fromRGB(90, 40, 10)

-- GUI principal
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 400, 0, 200)
mainFrame.Position = UDim2.new(0.5, -200, 0.5, -100)
mainFrame.BackgroundColor3 = Color3.fromRGB(210, 180, 140)
mainFrame.BorderSizePixel = 0
mainFrame.Visible = false
mainFrame.Active = true
mainFrame.Parent = gui

local frameCorner = Instance.new("UICorner", mainFrame)
frameCorner.CornerRadius = UDim.new(0, 12)

local frameStroke = Instance.new("UIStroke", mainFrame)
frameStroke.Thickness = 4
frameStroke.Color = Color3.fromRGB(90, 40, 10)

-- Título
local title = Instance.new("TextLabel", mainFrame)
title.Size = UDim2.new(1, 0, 0, 40)
title.BackgroundColor3 = Color3.fromRGB(200, 170, 120)
title.Text = "Mauro Hub"
title.Font = Enum.Font.GothamBold
title.TextColor3 = Color3.fromRGB(90, 40, 10)
title.TextScaled = true
title.Active = true

-- Botão Hitbox pequena ON/OFF
local wsButton = Instance.new("TextButton", mainFrame)
wsButton.Size = UDim2.new(0, 200, 0, 60)
wsButton.Position = UDim2.new(0.5, -100, 0.5, -30)
wsButton.BackgroundColor3 = Color3.fromRGB(90, 40, 10)
wsButton.Font = Enum.Font.GothamBold
wsButton.TextScaled = true
wsButton.TextColor3 = Color3.fromRGB(210, 180, 140)
wsButton.Text = "MAURO/OFF"

local wsCorner = Instance.new("UICorner", wsButton)
wsCorner.CornerRadius = UDim.new(0, 8)

-----------------------------------------------------
-- FUNÇÕES
-----------------------------------------------------

-- Função para arrastar objetos
local function makeDraggable(dragHandle, frame)
    local dragging, dragInput, dragStart, startPos
    dragHandle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)
    dragHandle.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            dragInput = input
        end
    end)
    UIS.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            local delta = input.Position - dragStart
            frame.Position = UDim2.new(
                startPos.X.Scale, startPos.X.Offset + delta.X,
                startPos.Y.Scale, startPos.Y.Offset + delta.Y
            )
        end
    end)
end

-- Deixar arrastáveis
makeDraggable(toggleButton, toggleButton) -- bolinha arrastável
makeDraggable(title, mainFrame) -- arrastar a janela pelo título

-- Mostrar/ocultar GUI
toggleButton.MouseButton1Click:Connect(function()
    mainFrame.Visible = not mainFrame.Visible
end)

-- Função para pegar humanoid mesmo após respawn
local humanoid
local function setupHumanoid(char)
    humanoid = char:WaitForChild("Humanoid")
    wsButton.Text = "MAURO/OFF"
end

setupHumanoid(player.Character or player.CharacterAdded:Wait())
player.CharacterAdded:Connect(setupHumanoid)

-- Estado da hitbox pequena
local miniActive = false

-- Função para mudar o tamanho do avatar
local function setMiniMode(enabled)
    local char = player.Character
    if not char then return end

    for _, part in ipairs(char:GetDescendants()) do
        if part:IsA("BasePart") then
            if enabled then
                part.Size = part.Size * 0.2 -- diminui 80%
            else
                part.Size = Vector3.new(2,2,1) -- reset básico humanoide (Roblox padrão)
                if part.Name == "Head" then
                    part.Size = Vector3.new(2,1,1)
                end
            end
        end
    end
    if humanoid then
        if enabled then
            humanoid.HipHeight = 0.2
        else
            humanoid.HipHeight = 2
        end
    end
end

-- Botão ON/OFF
wsButton.MouseButton1Click:Connect(function()
    miniActive = not miniActive
    if miniActive then
        wsButton.Text = "MAURO/ON"
        setMiniMode(true)
    else
        wsButton.Text = "MAURO/OFF"
        setMiniMode(false)
    end
end)
