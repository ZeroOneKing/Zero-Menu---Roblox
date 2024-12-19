-- Zero Menu Roblox

-- Script para crear un menú visual en pantalla con botones configurables, campos de texto, funcionalidad de minimizar/maximizar y arrastre

local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
local humanoid = character:WaitForChild("Humanoid")

local reachMultiplier = 5 -- Factor de multiplicación para el alcance de ataque
local savedPosition1 = nil -- Variable para guardar la primera posición
local savedPosition2 = nil -- Variable para guardar la segunda posición

local screenGui = Instance.new("ScreenGui", player.PlayerGui)
screenGui.Name = "ZeroMenu"

local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0, 300, 0, 400)
frame.Position = UDim2.new(0, 100, 0, 100)
frame.BackgroundColor3 = Color3.new(0, 0, 0)
frame.BorderSizePixel = 0
frame.Active = true

local minimized = false

local minimizeButton = Instance.new("TextButton", frame)
minimizeButton.Size = UDim2.new(1, 0, 0, 30)
minimizeButton.Position = UDim2.new(0, 0, 0, 0)
minimizeButton.BackgroundColor3 = Color3.new(0, 0, 0)
minimizeButton.TextColor3 = Color3.new(0, 1, 0)
minimizeButton.Text = "Zero Menu"
minimizeButton.Font = Enum.Font.SourceSansBold
minimizeButton.TextSize = 24

local function toggleMinimize()
    minimized = not minimized
    if minimized then
        frame.Size = UDim2.new(0, 300, 0, 30)
        for _, child in ipairs(frame:GetChildren()) do
            if child ~= minimizeButton then
                child.Visible = false
            end
        end
    else
        frame.Size = UDim2.new(0, 300, 0, 400)
        for _, child in ipairs(frame:GetChildren()) do
            if child ~= minimizeButton then
                child.Visible = true
            end
        end
    end
end

minimizeButton.MouseButton1Click:Connect(toggleMinimize)

local function createLabel(text, pos, textSize)
    local label = Instance.new("TextLabel", frame)
    label.Size = UDim2.new(1, 0, 0, 30)
    label.Position = UDim2.new(0, 0, 0, pos)
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.new(0, 1, 0)
    label.Text = text
    label.Font = Enum.Font.SourceSansBold
    label.TextSize = textSize
    return label
end

local function createButton(name, text, pos)
    local button = Instance.new("TextButton", frame)
    button.Name = name
    button.Size = UDim2.new(0.4, 0, 0, 30)
    button.Position = UDim2.new(0.05, 0, 0, pos)
    button.BackgroundColor3 = Color3.new(0, 0, 0)
    button.TextColor3 = Color3.new(1, 0, 0)
    button.Text = text
    button.Font = Enum.Font.SourceSans
    button.TextSize = 18
    return button
end

local function createToggleButton(name, pos, onToggle)
    local toggleButton = Instance.new("TextButton", frame)
    toggleButton.Name = name
    toggleButton.Size = UDim2.new(0, 20, 0, 20)
    toggleButton.Position = UDim2.new(0.85, 0, 0, pos + 5)
    toggleButton.BackgroundColor3 = Color3.new(1, 0, 0)
    toggleButton.Text = ""
    toggleButton.MouseButton1Click:Connect(function()
        if toggleButton.BackgroundColor3 == Color3.new(1, 0, 0) then
            toggleButton.BackgroundColor3 = Color3.new(0, 1, 0)
            onToggle(true)
        else
            toggleButton.BackgroundColor3 = Color3.new(1, 0, 0)
            onToggle(false)
        end
    end)
    return toggleButton
end

-- Función para aumentar el alcance de ataque
local function increaseReach()
    humanoidRootPart.Size = humanoidRootPart.Size * reachMultiplier
end

-- Función para restaurar el alcance de ataque
local function restoreReach()
    humanoidRootPart.Size = humanoidRootPart.Size / reachMultiplier
end

-- Función para hacer invisibles las partes del cuerpo, herramientas y accesorios
local function setInvisibility(isInvisible)
    for _, part in pairs(character:GetChildren()) do
        if part:IsA("BasePart") then
            part.Transparency = isInvisible and 1 or 0
        elseif part:IsA("Tool") then
            for _, toolPart in pairs(part:GetChildren()) do
                if toolPart:IsA("BasePart") or toolPart:IsA("MeshPart") then
                    toolPart.Transparency = isInvisible and 1 or 0
                end
            end
        elseif part:IsA("Accessory") then
            part.Handle.Transparency = isInvisible and 1 or 0
        end
    end
end

-- Función para teletransportarse al enemigo más cercano
local function teleportToNearestEnemy()
    local nearestEnemy = nil
    local shortestDistance = math.huge

    for _, npc in pairs(game.Workspace.Enemies:GetChildren()) do
        if npc:FindFirstChild("Humanoid") and npc.Humanoid.Health > 0 then
            local distance = (npc.HumanoidRootPart.Position - humanoidRootPart.Position).magnitude
            if distance < shortestDistance then
                shortestDistance = distance
                nearestEnemy = npc
            end
        end
    end

    if nearestEnemy then
        humanoidRootPart.CFrame = CFrame.new(nearestEnemy.HumanoidRootPart.Position + Vector3.new(0, 2, 0))
        print("Teletransportado al enemigo más cercano")
    else
        print("No hay enemigos cercanos")
    end
end

-- Crear etiquetas y botones en la GUI
createLabel("Buddha Mode", 50, 18)

local buddhaToggleButton = createToggleButton("BuddhaToggleButton", 50, function(isEnabled)
    if isEnabled then
        increaseReach()
        setInvisibility(true)
        humanoid.WalkSpeed = humanoid.WalkSpeed * 10
    else
        restoreReach()
        setInvisibility(false)
        humanoid.WalkSpeed = humanoid.WalkSpeed / 10
    end
end) -- Botón circular para Buddha Mode

createLabel("TP Positions", 90, 18)

local tpPos1Button = createButton("TPPos1Button", "TP Pos 1", 130)
local savePos1Button = createButton("SavePos1Button", "Save Pos 1", 130)
savePos1Button.Position = UDim2.new(0.6, 0, 0, 130) -- Posicionar al lado derecho del botón TP Pos 1

savePos1Button.MouseButton1Click:Connect(function()
    savePos1Button.TextColor3 = Color3.new(0, 1, 0) -- Cambiar a verde si se presiona el botón
    savedPosition1 = humanoidRootPart.Position
    print("Posición 1 guardada")
end)

tpPos1Button.MouseButton1Click:Connect(function()
    if savedPosition1 then
        humanoidRootPart.CFrame = CFrame.new(savedPosition1 + Vector3.new(0, 2, 0))
        print("Teletransportado a la posición 1")
    else
        print("No hay una posición 1 guardada")
    end
end)

local tpPos2Button = createButton("TPPos2Button", "TP Pos 2", 170)
local savePos2Button = createButton("SavePos2Button", "Save Pos 2", 170)
savePos2Button.Position = UDim2.new(0.6, 0, 0, 170) -- Posicionar al lado derecho del botón TP Pos 2

savePos2Button.MouseButton1Click:Connect(function()
    savePos2Button.TextColor3 = Color3.new(0, 1, 0) -- Cambiar a verde si se presiona el botón
    savedPosition2 = humanoidRootPart.Position
    print("Posición 2 guardada")
end)

tpPos2Button.MouseButton1Click:Connect(function()
    if savedPosition2 then
        humanoidRootPart.CFrame = CFrame.new(savedPosition2 + Vector3.new(0, 2, 0))
        print("Teletransportado a la posición 2")
    else
        print("No hay una posición 2 guardada")
    end
end)

createButton("DevilFruitsRollButton", "Devil Fruits Roll", 210)

createLabel("Enemy TP key", 250, 18) -- Ajustado para que no se superponga con otros elementos
local enemyTPButton = createButton("EnemyTPButton", "E", 290)
local selectedKey = Enum.KeyCode.E

-- Función para configurar la tecla Enemy TP
local settingKey = false

enemyTPButton.MouseButton1Click:Connect(function()
    if settingKey then
        settingKey = false
        enemyTPButton.Text = selectedKey.Name
    else
        settingKey = true
        enemyTPButton.Text = "..."
    end
end)

-- Detectar cuando se presiona una tecla para configurar Enemy TP key
game:GetService("UserInputService").InputBegan:Connect(function(input, gameProcessed)
    if settingKey then
        if input.KeyCode == Enum.KeyCode.Escape then
            enemyTPButton.Text = selectedKey.Name
        else
            selectedKey = input.KeyCode
            enemyTPButton.Text = selectedKey.Name
        end
        settingKey = false
    elseif input.KeyCode == selectedKey then
        teleportToNearestEnemy()
    end
end)

-- Funcionalidad para arrastrar el menú
local dragging = false
local dragInput, mousePos, framePos

minimizeButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        mousePos = input.Position
        framePos = frame.Position

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

minimizeButton.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

game:GetService("UserInputService").InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - mousePos
        frame.Position = UDim2.new(framePos.X.Scale, framePos.X.Offset + delta.X, framePos.Y.Scale, framePos.Y.Offset + delta.Y)
    end
end)
