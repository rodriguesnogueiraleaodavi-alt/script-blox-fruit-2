--[[
    BLOX FRUITS ULTIMATE FARM SCRIPT - VERSÃO OTIMIZADA
    SEM BLUR, SEM TRAVAMENTOS
    COLOCAR EM: StarterPlayer > StarterPlayerScripts
--]]

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- ================================
-- CONFIGURAÇÕES PRINCIPAIS
-- ================================

local Settings = {
    AuraEnabled = true,
    ESPEnabled = true,
    AutoFarmEnabled = false,
    AuraColor = Color3.fromRGB(255, 70, 85),
    FarmRadius = 500,
    OnlyRareFruits = false
}

-- Lista de frutas raras
local RareFruits = {
    "Leopard", "Dragon", "Dough", "Spirit", "Venom", 
    "Control", "Shadow", "Gravity", "Buddha"
}

-- Variáveis
local FruitsList = {}
local CurrentTarget = nil
local MovingToTarget = false
local FruitHighlights = {}
local ESPLabels = {}
local Notifications = {}
local LastScanTime = 0
local LastESPUpdate = 0

-- ================================
-- INTERFACE PRINCIPAL (SEM BLUR)
-- ================================

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "BloxFruitsGUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- Container Principal
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 350, 0, 500)
MainFrame.Position = UDim2.new(0.5, -175, 0.5, -250)
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
MainFrame.BackgroundTransparency = 0.05
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

-- Cantos arredondados
local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 12)
MainCorner.Parent = MainFrame

-- Borda simples
local MainStroke = Instance.new("UIStroke")
MainStroke.Color = Color3.fromRGB(255, 70, 85)
MainStroke.Thickness = 1
MainStroke.Transparency = 0.5
MainStroke.Parent = MainFrame

-- ================================
-- BARRA DE TÍTULO
-- ================================

local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 45)
TitleBar.BackgroundColor3 = Color3.fromRGB(255, 70, 85)
TitleBar.BackgroundTransparency = 0.15
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 12)
TitleCorner.Parent = TitleBar

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Size = UDim2.new(1, 0, 1, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "🍎 BLOX FRUITS ULTIMATE 🍎"
TitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.TextSize = 16
TitleLabel.Parent = TitleBar

-- Indicador de status
local StatusIndicator = Instance.new("Frame")
StatusIndicator.Size = UDim2.new(0, 10, 0, 10)
StatusIndicator.Position = UDim2.new(1, -20, 0, 17)
StatusIndicator.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
StatusIndicator.BorderSizePixel = 0
StatusIndicator.Parent = TitleBar

local StatusCorner = Instance.new("UICorner")
StatusCorner.CornerRadius = UDim.new(1, 0)
StatusCorner.Parent = StatusIndicator

-- ================================
-- FUNÇÃO PARA CRIAR BOTÕES
-- ================================

local function createModernButton(parent, text, position, color)
    local Button = Instance.new("TextButton")
    Button.Size = UDim2.new(0, 310, 0, 42)
    Button.Position = position
    Button.BackgroundColor3 = color or Color3.fromRGB(35, 35, 40)
    Button.BackgroundTransparency = 0.1
    Button.BorderSizePixel = 0
    Button.Text = text
    Button.TextColor3 = Color3.fromRGB(255, 255, 255)
    Button.Font = Enum.Font.GothamSemibold
    Button.TextSize = 14
    Button.Parent = parent
    
    local ButtonCorner = Instance.new("UICorner")
    ButtonCorner.CornerRadius = UDim.new(0, 8)
    ButtonCorner.Parent = Button
    
    -- Animação hover (mais leve)
    Button.MouseEnter:Connect(function()
        TweenService:Create(Button, TweenInfo.new(0.15), {
            BackgroundTransparency = 0.05,
            Size = UDim2.new(0, 315, 0, 44)
        }):Play()
    end)
    
    Button.MouseLeave:Connect(function()
        TweenService:Create(Button, TweenInfo.new(0.15), {
            BackgroundTransparency = 0.1,
            Size = UDim2.new(0, 310, 0, 42)
        }):Play()
    end)
    
    return Button
end

local function createToggleButton(parent, text, position)
    local Container = Instance.new("Frame")
    Container.Size = UDim2.new(0, 310, 0, 42)
    Container.Position = position
    Container.BackgroundColor3 = Color3.fromRGB(35, 35, 40)
    Container.BackgroundTransparency = 0.1
    Container.BorderSizePixel = 0
    Container.Parent = parent
    
    local ContainerCorner = Instance.new("UICorner")
    ContainerCorner.CornerRadius = UDim.new(0, 8)
    ContainerCorner.Parent = Container
    
    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(0, 200, 1, 0)
    Label.Position = UDim2.new(0, 15, 0, 0)
    Label.BackgroundTransparency = 1
    Label.Text = text
    Label.TextColor3 = Color3.fromRGB(255, 255, 255)
    Label.Font = Enum.Font.Gotham
    Label.TextSize = 14
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Parent = Container
    
    local Toggle = Instance.new("TextButton")
    Toggle.Size = UDim2.new(0, 70, 0, 32)
    Toggle.Position = UDim2.new(1, -80, 0.5, -16)
    Toggle.BackgroundColor3 = Color3.fromRGB(60, 60, 65)
    Toggle.BorderSizePixel = 0
    Toggle.Text = "OFF"
    Toggle.TextColor3 = Color3.fromRGB(255, 255, 255)
    Toggle.Font = Enum.Font.GothamBold
    Toggle.TextSize = 12
    Toggle.Parent = Container
    
    local ToggleCorner = Instance.new("UICorner")
    ToggleCorner.CornerRadius = UDim.new(0, 6)
    ToggleCorner.Parent = Toggle
    
    return Container, Toggle
end

-- Criar botões
local AuraContainer, AuraToggle = createToggleButton(MainFrame, "✨ Sistema de Aura", UDim2.new(0.5, -155, 0, 70))
local ESPContainer, ESPToggle = createToggleButton(MainFrame, "👁️ Sistema de ESP", UDim2.new(0.5, -155, 0, 125))
local FarmContainer, FarmToggle = createToggleButton(MainFrame, "🤖 Farm Automático", UDim2.new(0.5, -155, 0, 180))

local ColorButton = createModernButton(MainFrame, "🎨 Mudar Cor da Aura", UDim2.new(0.5, -155, 0, 240), Color3.fromRGB(255, 70, 85))
local RareContainer, RareToggle = createToggleButton(MainFrame, "⭐ Apenas Frutas Raras", UDim2.new(0.5, -155, 0, 295))

local FarmStatus = Instance.new("TextLabel")
FarmStatus.Size = UDim2.new(0, 310, 0, 35)
FarmStatus.Position = UDim2.new(0.5, -155, 0, 355)
FarmStatus.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
FarmStatus.BackgroundTransparency = 0.3
FarmStatus.Text = "🤖 Farm: Parado"
FarmStatus.TextColor3 = Color3.fromRGB(200, 200, 200)
FarmStatus.Font = Enum.Font.Gotham
FarmStatus.TextSize = 12
FarmStatus.Parent = MainFrame

local FarmStatusCorner = Instance.new("UICorner")
FarmStatusCorner.CornerRadius = UDim.new(0, 6)
FarmStatusCorner.Parent = FarmStatus

-- Créditos
local Credits = Instance.new("TextLabel")
Credits.Size = UDim2.new(1, 0, 0, 25)
Credits.Position = UDim2.new(0, 0, 1, -25)
Credits.BackgroundTransparency = 1
Credits.Text = "Blox Fruits Ultimate | Sistema Otimizado"
Credits.TextColor3 = Color3.fromRGB(150, 150, 150)
Credits.Font = Enum.Font.Gotham
Credits.TextSize = 10
Credits.Parent = MainFrame

-- ================================
-- SISTEMA DE ARRASTAR (OTIMIZADO)
-- ================================

local Dragging = false
local DragStart = nil
local StartPos = nil

TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        Dragging = true
        DragStart = input.Position
        StartPos = MainFrame.Position
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if Dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local Delta = input.Position - DragStart
        MainFrame.Position = UDim2.new(StartPos.X.Scale, StartPos.X.Offset + Delta.X, StartPos.Y.Scale, StartPos.Y.Offset + Delta.Y)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        Dragging = false
    end
end)

-- ================================
-- NOTIFICAÇÕES SIMPLES
-- ================================

local function createNotification(text, duration)
    local Notification = Instance.new("Frame")
    Notification.Size = UDim2.new(0, 250, 0, 40)
    Notification.Position = UDim2.new(1, -270, 0, 10 + (#Notifications * 50))
    Notification.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
    Notification.BackgroundTransparency = 0.1
    Notification.BorderSizePixel = 0
    Notification.Parent = ScreenGui
    
    local NotifCorner = Instance.new("UICorner")
    NotifCorner.CornerRadius = UDim.new(0, 8)
    NotifCorner.Parent = Notification
    
    local Text = Instance.new("TextLabel")
    Text.Size = UDim2.new(1, -20, 1, 0)
    Text.Position = UDim2.new(0, 10, 0, 0)
    Text.BackgroundTransparency = 1
    Text.Text = text
    Text.TextColor3 = Color3.fromRGB(255, 255, 255)
    Text.Font = Enum.Font.Gotham
    Text.TextSize = 12
    Text.TextXAlignment = Enum.TextXAlignment.Left
    Text.Parent = Notification
    
    table.insert(Notifications, Notification)
    
    task.wait(duration)
    Notification:Destroy()
    table.remove(Notifications, table.find(Notifications, Notification))
end

-- ================================
-- SISTEMA DE AURA (SIMPLES E RÁPIDO)
-- ================================

local function updateAura(fruit, enabled)
    if enabled then
        if not fruit:FindFirstChild("FruitHighlight") then
            local Highlight = Instance.new("Highlight")
            Highlight.Name = "FruitHighlight"
            Highlight.FillColor = Settings.AuraColor
            Highlight.FillTransparency = 0.5
            Highlight.OutlineColor = Settings.AuraColor
            Highlight.OutlineTransparency = 0.3
            Highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
            Highlight.Parent = fruit
            FruitHighlights[fruit] = Highlight
        else
            local hl = fruit.FruitHighlight
            hl.FillColor = Settings.AuraColor
            hl.OutlineColor = Settings.AuraColor
        end
    else
        if fruit:FindFirstChild("FruitHighlight") then
            fruit.FruitHighlight:Destroy()
            FruitHighlights[fruit] = nil
        end
    end
end

-- ================================
-- SISTEMA DE ESP (SIMPLES)
-- ================================

local function updateESP(fruit, enabled)
    if enabled then
        if not ESPLabels[fruit] then
            local Billboard = Instance.new("BillboardGui")
            Billboard.Name = "FruitESP"
            Billboard.Size = UDim2.new(0, 150, 0, 40)
            Billboard.StudsOffset = Vector3.new(0, 2.5, 0)
            Billboard.AlwaysOnTop = true
            Billboard.Parent = fruit
            
            local Frame = Instance.new("Frame")
            Frame.Size = UDim2.new(1, 0, 1, 0)
            Frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
            Frame.BackgroundTransparency = 0.4
            Frame.BorderSizePixel = 0
            Frame.Parent = Billboard
            
            local FrameCorner = Instance.new("UICorner")
            FrameCorner.CornerRadius = UDim.new(0, 6)
            FrameCorner.Parent = Frame
            
            local FruitName = Instance.new("TextLabel")
            FruitName.Size = UDim2.new(1, 0, 0, 20)
            FruitName.Position = UDim2.new(0, 0, 0, 2)
            FruitName.BackgroundTransparency = 1
            FruitName.Text = fruit.Name or "Fruta"
            FruitName.TextColor3 = Color3.fromRGB(255, 70, 85)
            FruitName.Font = Enum.Font.GothamBold
            FruitName.TextSize = 12
            FruitName.Parent = Frame
            
            local Distance = Instance.new("TextLabel")
            Distance.Name = "Distance"
            Distance.Size = UDim2.new(1, 0, 0, 18)
            Distance.Position = UDim2.new(0, 0, 0, 22)
            Distance.BackgroundTransparency = 1
            Distance.Text = "0m"
            Distance.TextColor3 = Color3.fromRGB(200, 200, 200)
            Distance.Font = Enum.Font.Gotham
            Distance.TextSize = 10
            Distance.Parent = Frame
            
            ESPLabels[fruit] = {Billboard = Billboard, Distance = Distance}
        end
    else
        if ESPLabels[fruit] then
            ESPLabels[fruit].Billboard:Destroy()
            ESPLabels[fruit] = nil
        end
    end
end

-- ================================
-- SISTEMA DE FARM (OTIMIZADO)
-- ================================

local function getClosestFruit()
    if not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        return nil
    end
    
    local closest = nil
    local closestDistance = Settings.FarmRadius
    local playerPos = LocalPlayer.Character.HumanoidRootPart.Position
    
    for _, fruit in pairs(FruitsList) do
        if fruit and fruit.Parent then
            -- Verificar filtro de frutas raras
            if Settings.OnlyRareFruits then
                local isRare = false
                for _, rare in pairs(RareFruits) do
                    if fruit.Name:find(rare) then
                        isRare = true
                        break
                    end
                end
                if not isRare then
                    goto continue
                end
            end
            
            local distance = (fruit.Position - playerPos).Magnitude
            if distance < closestDistance then
                closestDistance = distance
                closest = fruit
            end
        end
        ::continue::
    end
    
    return closest, closestDistance
end

-- Loop de farm (otimizado)
task.spawn(function()
    while true do
        if Settings.AutoFarmEnabled and not MovingToTarget and LocalPlayer.Character then
            local closestFruit = getClosestFruit()
            
            if closestFruit then
                local humanoid = LocalPlayer.Character:FindFirstChild("Humanoid")
                local rootPart = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                
                if humanoid and rootPart then
                    MovingToTarget = true
                    CurrentTarget = closestFruit
                    
                    while MovingToTarget and Settings.AutoFarmEnabled and closestFruit and closestFruit.Parent do
                        local distance = (closestFruit.Position - rootPart.Position).Magnitude
                        
                        if distance < 5 then
                            createNotification("🍎 Coletando: " .. closestFruit.Name, 2)
                            break
                        end
                        
                        humanoid:MoveTo(closestFruit.Position)
                        FarmStatus.Text = string.format("🤖 Farm: %.1fm de %s", distance, closestFruit.Name)
                        
                        task.wait(0.15)
                    end
                    
                    MovingToTarget = false
                    CurrentTarget = nil
                    FarmStatus.Text = "🤖 Farm: Parado"
                end
            else
                FarmStatus.Text = "🤖 Farm: Nenhuma fruta próxima"
                task.wait(1)
            end
        end
        
        task.wait(0.3)
    end
end)

-- ================================
-- DETECÇÃO DE FRUTAS (OTIMIZADA)
-- ================================

local function scanForFruits()
    local fruits = {}
    local startTime = tick()
    
    for _, obj in pairs(workspace:GetDescendants()) do
        if tick() - startTime > 0.05 then -- Limitar tempo de scan
            break
        end
        
        if obj:IsA("Model") or obj:IsA("Part") then
            local name = obj.Name:lower()
            if name:find("fruit") or name:find("apple") or name:find("devil") or obj:FindFirstChild("Handle") then
                table.insert(fruits, obj)
            end
        end
    end
    
    return fruits
end

-- Loop de atualização (otimizado)
task.spawn(function()
    while true do
        local newFruits = scanForFruits()
        
        -- Adicionar novas frutas
        for _, fruit in pairs(newFruits) do
            local found = false
            for _, existing in pairs(FruitsList) do
                if existing == fruit then
                    found = true
                    break
                end
            end
            
            if not found then
                table.insert(FruitsList, fruit)
                createNotification("🍎 Nova fruta apareceu!", 2)
            end
        end
        
        -- Remover frutas coletadas
        for i = #FruitsList, 1, -1 do
            local fruit = FruitsList[i]
            if not fruit or not fruit.Parent then
                if FruitHighlights[fruit] then
                    FruitHighlights[fruit]:Destroy()
                    FruitHighlights[fruit] = nil
                end
                if ESPLabels[fruit] then
                    ESPLabels[fruit].Billboard:Destroy()
                    ESPLabels[fruit] = nil
                end
                table.remove(FruitsList, i)
            end
        end
        
        -- Atualizar efeitos
        for _, fruit in pairs(FruitsList) do
            if fruit and fruit.Parent then
                updateAura(fruit, Settings.AuraEnabled)
                updateESP(fruit, Settings.ESPEnabled)
                
                -- Atualizar distância no ESP
                if Settings.ESPEnabled and ESPLabels[fruit] and LocalPlayer.Character then
                    local rootPart = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                    if rootPart then
                        local distance = (fruit.Position - rootPart.Position).Magnitude
                        ESPLabels[fruit].Distance.Text = string.format("%.1fm", distance)
                    end
                end
            end
        end
        
        task.wait(0.5)
    end
end)

-- ================================
-- EVENTOS DOS BOTÕES
-- ================================

AuraToggle.MouseButton1Click:Connect(function()
    Settings.AuraEnabled = not Settings.AuraEnabled
    AuraToggle.Text = Settings.AuraEnabled and "ON" or "OFF"
    AuraToggle.BackgroundColor3 = Settings.AuraEnabled and Color3.fromRGB(0, 200, 0) or Color3.fromRGB(60, 60, 65)
    createNotification("✨ Aura " .. (Settings.AuraEnabled and "Ativada" or "Desativada"), 1.5)
end)

ESPToggle.MouseButton1Click:Connect(function()
    Settings.ESPEnabled = not Settings.ESPEnabled
    ESPToggle.Text = Settings.ESPEnabled and "ON" or "OFF"
    ESPToggle.BackgroundColor3 = Settings.ESPEnabled and Color3.fromRGB(0, 200, 0) or Color3.fromRGB(60, 60, 65)
    createNotification("👁️ ESP " .. (Settings.ESPEnabled and "Ativado" or "Desativado"), 1.5)
end)

FarmToggle.MouseButton1Click:Connect(function()
    Settings.AutoFarmEnabled = not Settings.AutoFarmEnabled
    FarmToggle.Text = Settings.AutoFarmEnabled and "ON" or "OFF"
    FarmToggle.BackgroundColor3 = Settings.AutoFarmEnabled and Color3.fromRGB(0, 200, 0) or Color3.fromRGB(60, 60, 65)
    createNotification("🤖 Farm " .. (Settings.AutoFarmEnabled and "Ativado" or "Desativado"), 1.5)
end)

ColorButton.MouseButton1Click:Connect(function()
    local colors = {
        Color3.fromRGB(255, 70, 85),
        Color3.fromRGB(0, 200, 255),
        Color3.fromRGB(0, 255, 0),
        Color3.fromRGB(255, 255, 0),
        Color3.fromRGB(255, 0, 255),
        Color3.fromRGB(255, 165, 0)
    }
    
    local currentIndex = 1
    for i, color in pairs(colors) do
        if color == Settings.AuraColor then
            currentIndex = i
            break
        end
    end
    
    local nextIndex = currentIndex % #colors + 1
    Settings.AuraColor = colors[nextIndex]
    createNotification("🎨 Cor da aura alterada", 1.5)
end)

RareToggle.MouseButton1Click:Connect(function()
    Settings.OnlyRareFruits = not Settings.OnlyRareFruits
    RareToggle.Text = Settings.OnlyRareFruits and "ON" or "OFF"
    RareToggle.BackgroundColor3 = Settings.OnlyRareFruits and Color3.fromRGB(0, 200, 0) or Color3.fromRGB(60, 60, 65)
    createNotification("⭐ Filtro: " .. (Settings.OnlyRareFruits and "Apenas raras" or "Todas"), 1.5)
end)

-- Inicialização
createNotification("🚀 Sistema carregado com sucesso!", 2)
print("✅ Blox Fruits GUI Otimizada carregada!")
