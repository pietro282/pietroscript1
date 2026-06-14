--[[
    SCRIPT: Pietro Mods - Chute uma Lucky Block
    Autor: Pietro Mods
    Versão: 2.0
    Descrição: Hub completo para o jogo "Chute uma Lucky Block" - PC Compatível
--]]

-- Variáveis Globais
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local PathfindingService = game:GetService("PathfindingService")
local VirtualInput = game:GetService("VirtualInputManager")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")
local mouse = player:GetMouse()

-- Estado das funcionalidades
local Features = {
    AutoPerfectKick = false,
    TeleportToBalls = false,
    AutoReturnBase = false,
    Auto2xWeight = false
}

-- Variáveis de controle
local isKicking = false
local currentBallIndex = 1
local allBalls = {}
local basePosition = nil
local isReturningToBase = false
local weightCheckInterval = nil

-- Função para criar a interface gráfica
local function CreateUI()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "PietroModsHub"
    screenGui.Parent = player:WaitForChild("PlayerGui")
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    
    -- Frame principal
    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 380, 0, 520)
    mainFrame.Position = UDim2.new(0.5, -190, 0.5, -260)
    mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
    mainFrame.BackgroundTransparency = 0.05
    mainFrame.BorderSizePixel = 0
    mainFrame.Parent = screenGui
    
    -- Sombra
    local shadow = Instance.new("Frame")
    shadow.Name = "Shadow"
    shadow.Size = UDim2.new(1, 10, 1, 10)
    shadow.Position = UDim2.new(0, -5, 0, -5)
    shadow.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    shadow.BackgroundTransparency = 0.5
    shadow.BorderSizePixel = 0
    shadow.Parent = mainFrame
    
    -- Arredondamento
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 12)
    corner.Parent = mainFrame
    
    -- Título "Pietro Mods"
    local titleFrame = Instance.new("Frame")
    titleFrame.Size = UDim2.new(1, 0, 0, 60)
    titleFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
    titleFrame.BorderSizePixel = 0
    titleFrame.Parent = mainFrame
    
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 12)
    titleCorner.Parent = titleFrame
    
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(1, 0, 1, 0)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = "PIETRO MODS"
    titleLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
    titleLabel.TextSize = 24
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextStrokeTransparency = 0.5
    titleLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    titleLabel.Parent = titleFrame
    
    -- Subtítulo
    local subtitle = Instance.new("TextLabel")
    subtitle.Size = UDim2.new(1, 0, 0, 20)
    subtitle.Position = UDim2.new(0, 0, 1, -25)
    subtitle.BackgroundTransparency = 1
    subtitle.Text = "Chute uma Lucky Block"
    subtitle.TextColor3 = Color3.fromRGB(150, 150, 170)
    subtitle.TextSize = 12
    subtitle.Font = Enum.Font.Gotham
    subtitle.Parent = titleFrame
    
    -- ScrollingFrame para os botões
    local scrollFrame = Instance.new("ScrollingFrame")
    scrollFrame.Size = UDim2.new(1, -20, 1, -80)
    scrollFrame.Position = UDim2.new(0, 10, 0, 70)
    scrollFrame.BackgroundTransparency = 1
    scrollFrame.BorderSizePixel = 0
    scrollFrame.CanvasSize = UDim2.new(0, 0, 0, 400)
    scrollFrame.ScrollBarThickness = 4
    scrollFrame.ScrollBarImageColor3 = Color3.fromRGB(100, 100, 120)
    scrollFrame.Parent = mainFrame
    
    -- Função para criar botões
    local function createButton(text, description, yPosition, color)
        local buttonFrame = Instance.new("Frame")
        buttonFrame.Size = UDim2.new(1, -10, 0, 70)
        buttonFrame.Position = UDim2.new(0, 5, 0, yPosition)
        buttonFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
        buttonFrame.BackgroundTransparency = 0.3
        buttonFrame.BorderSizePixel = 0
        buttonFrame.Parent = scrollFrame
        
        local buttonCorner = Instance.new("UICorner")
        buttonCorner.CornerRadius = UDim.new(0, 8)
        buttonCorner.Parent = buttonFrame
        
        local buttonText = Instance.new("TextLabel")
        buttonText.Size = UDim2.new(0.6, -10, 1, 0)
        buttonText.Position = UDim2.new(0, 10, 0, 0)
        buttonText.BackgroundTransparency = 1
        buttonText.Text = text
        buttonText.TextColor3 = Color3.fromRGB(220, 220, 240)
        buttonText.TextSize = 16
        buttonText.Font = Enum.Font.GothamSemibold
        buttonText.TextXAlignment = Enum.TextXAlignment.Left
        buttonText.Parent = buttonFrame
        
        local descLabel = Instance.new("TextLabel")
        descLabel.Size = UDim2.new(0.6, -10, 0, 20)
        descLabel.Position = UDim2.new(0, 10, 0, 30)
        descLabel.BackgroundTransparency = 1
        descLabel.Text = description
        descLabel.TextColor3 = Color3.fromRGB(130, 130, 150)
        descLabel.TextSize = 11
        descLabel.Font = Enum.Font.Gotham
        descLabel.TextXAlignment = Enum.TextXAlignment.Left
        descLabel.Parent = buttonFrame
        
        local toggleButton = Instance.new("TextButton")
        toggleButton.Size = UDim2.new(0, 80, 0, 35)
        toggleButton.Position = UDim2.new(1, -90, 0.5, -17.5)
        toggleButton.BackgroundColor3 = color or Color3.fromRGB(60, 60, 75)
        toggleButton.Text = "DESATIVADO"
        toggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
        toggleButton.TextSize = 12
        toggleButton.Font = Enum.Font.GothamBold
        toggleButton.BorderSizePixel = 0
        toggleButton.Parent = buttonFrame
        
        local toggleCorner = Instance.new("UICorner")
        toggleCorner.CornerRadius = UDim.new(0, 6)
        toggleCorner.Parent = toggleButton
        
        return toggleButton
    end
    
    -- Criar botões
    local perfectKickBtn = createButton("⚡ Auto Perfect Kick", "Chute perfeito automático contínuo", 10, Color3.fromRGB(255, 100, 100))
    local teleportBallsBtn = createButton("🎯 Teleportar para Bolas", "Localiza e teleporta para todas as bolas", 90, Color3.fromRGB(100, 150, 255))
    local returnBaseBtn = createButton("🏠 Auto Voltar à Base", "Retorna automaticamente andando até a base", 170, Color3.fromRGB(100, 255, 150))
    local weight2xBtn = createButton("⚖️ Auto 2x Peso", "Ativa automaticamente o multiplicador 2x", 250, Color3.fromRGB(255, 200, 100))
    
    -- Botão de parar tudo
    local stopAllBtn = Instance.new("TextButton")
    stopAllBtn.Size = UDim2.new(0, 120, 0, 40)
    stopAllBtn.Position = UDim2.new(0.5, -60, 1, -50)
    stopAllBtn.BackgroundColor3 = Color3.fromRGB(255, 60, 60)
    stopAllBtn.Text = "🛑 PARAR TUDO"
    stopAllBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    stopAllBtn.TextSize = 14
    stopAllBtn.Font = Enum.Font.GothamBold
    stopAllBtn.BorderSizePixel = 0
    stopAllBtn.Parent = mainFrame
    
    local stopCorner = Instance.new("UICorner")
    stopCorner.CornerRadius = UDim.new(0, 6)
    stopCorner.Parent = stopAllBtn
    
    -- Sistema de notificações
    local notificationFrame = Instance.new("Frame")
    notificationFrame.Size = UDim2.new(0, 320, 0, 50)
    notificationFrame.Position = UDim2.new(0.5, -160, 1, -100)
    notificationFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    notificationFrame.BackgroundTransparency = 0.1
    notificationFrame.BorderSizePixel = 0
    notificationFrame.Parent = screenGui
    notificationFrame.Visible = false
    
    local notifyCorner = Instance.new("UICorner")
    notifyCorner.CornerRadius = UDim.new(0, 8)
    notifyCorner.Parent = notificationFrame
    
    local notificationText = Instance.new("TextLabel")
    notificationText.Size = UDim2.new(1, -20, 1, 0)
    notificationText.Position = UDim2.new(0, 10, 0, 0)
    notificationText.BackgroundTransparency = 1
    notificationText.Text = ""
    notificationText.TextColor3 = Color3.fromRGB(255, 255, 255)
    notificationText.TextSize = 13
    notificationText.Font = Enum.Font.Gotham
    notificationText.Parent = notificationFrame
    
    -- Função para mostrar notificação
    local function showNotification(message, isError)
        notificationText.Text = message
        notificationFrame.BackgroundColor3 = isError and Color3.fromRGB(255, 60, 60) or Color3.fromRGB(60, 150, 255)
        notificationFrame.BackgroundTransparency = 0.05
        notificationFrame.Visible = true
        
        local fadeOut = TweenService:Create(notificationFrame, TweenInfo.new(3, Enum.EasingStyle.Linear), {
            BackgroundTransparency = 1
        })
        
        fadeOut:Play()
        task.wait(3)
        notificationFrame.Visible = false
        notificationFrame.BackgroundTransparency = 0.05
    end
    
    -- Função para atualizar botão
    local function updateButtonState(button, isActive, featureName)
        if isActive then
            button.BackgroundColor3 = Color3.fromRGB(80, 200, 80)
            button.Text = "ATIVADO ✓"
            showNotification(featureName .. " ativado!", false)
        else
            button.BackgroundColor3 = Color3.fromRGB(60, 60, 75)
            button.Text = "DESATIVADO"
            showNotification(featureName .. " desativado!", false)
        end
    end
    
    -- Eventos dos botões
    perfectKickBtn.MouseButton1Click:Connect(function()
        Features.AutoPerfectKick = not Features.AutoPerfectKick
        updateButtonState(perfectKickBtn, Features.AutoPerfectKick, "⚡ Auto Perfect Kick")
        if Features.AutoPerfectKick then
            startPerfectKickLoop()
        end
    end)
    
    teleportBallsBtn.MouseButton1Click:Connect(function()
        Features.TeleportToBalls = not Features.TeleportToBalls
        updateButtonState(teleportBallsBtn, Features.TeleportToBalls, "🎯 Teleportar para Bolas")
        if Features.TeleportToBalls then
            startTeleportToBalls()
        end
    end)
    
    returnBaseBtn.MouseButton1Click:Connect(function()
        Features.AutoReturnBase = not Features.AutoReturnBase
        updateButtonState(returnBaseBtn, Features.AutoReturnBase, "🏠 Auto Voltar à Base")
        if Features.AutoReturnBase then
            startAutoReturnBase()
        end
    end)
    
    weight2xBtn.MouseButton1Click:Connect(function()
        Features.Auto2xWeight = not Features.Auto2xWeight
        updateButtonState(weight2xBtn, Features.Auto2xWeight, "⚖️ Auto 2x Peso")
        if Features.Auto2xWeight then
            startAuto2xWeight()
        else
            if weightCheckInterval then
                weightCheckInterval:Disconnect()
                weightCheckInterval = nil
            end
        end
    end)
    
    stopAllBtn.MouseButton1Click:Connect(function()
        Features.AutoPerfectKick = false
        Features.TeleportToBalls = false
        Features.AutoReturnBase = false
        Features.Auto2xWeight = false
        
        updateButtonState(perfectKickBtn, false, "")
        updateButtonState(teleportBallsBtn, false, "")
        updateButtonState(returnBaseBtn, false, "")
        updateButtonState(weight2xBtn, false, "")
        
        if weightCheckInterval then
            weightCheckInterval:Disconnect()
            weightCheckInterval = nil
        end
        
        isKicking = false
        isReturningToBase = false
        
        showNotification("Todas as funcionalidades foram paradas!", false)
    end)
    
    -- Sistema de drag
    local dragging = false
    local dragInput, dragStart, startPos
    
    titleFrame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = mainFrame.Position
            
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)
    
    titleFrame.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
    
    return showNotification
end

-- FUNCIONALIDADE 1: Auto Perfect Kick
local function clickAtPosition(position)
    VirtualInput:SendMouseButtonEvent(position, 0, true, Enum.UserInputType.MouseButton1, 1)
    task.wait(0.05)
    VirtualInput:SendMouseButtonEvent(position, 0, false, Enum.UserInputType.MouseButton1, 1)
end

local function findLuckyBlock()
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("Part") or obj:IsA("MeshPart") then
            if obj.Name:lower():find("lucky") or obj.Name:lower():find("block") then
                return obj
            end
        end
    end
    return nil
end

local function isAtBase()
    for _, part in pairs(workspace:GetDescendants()) do
        if part:IsA("Part") and (part.Name:lower():find("base") or part.Name:lower():find("spawn")) then
            if rootPart and (rootPart.Position - part.Position).Magnitude < 15 then
                return true
            end
        end
    end
    return false
end

local function performPerfectKick()
    if isKicking then return end
    isKicking = true
    
    local luckyBlock = findLuckyBlock()
    
    if luckyBlock then
        -- Mover para perto do bloco
        local blockPosition = luckyBlock.Position
        local moveToPosition = blockPosition + Vector3.new(5, 0, 5)
        
        humanoid:MoveTo(moveToPosition)
        humanoid.MoveToFinished:Wait(2)
        
        -- Procurar e clicar no ProximityPrompt
        local prompt = luckyBlock:FindFirstChildWhichIsA("ProximityPrompt")
        if prompt then
            -- Simular olhar para o prompt
            local lookAt = CFrame.new(rootPart.Position, prompt.Parent.Position)
            rootPart.CFrame = lookAt
            
            task.wait(0.3)
            
            -- Simular clique (E ou clique do mouse)
            VirtualInput:SendKeyEvent(true, "E", false, game)
            task.wait(0.1)
            VirtualInput:SendKeyEvent(false, "E", false, game)
            
            showNotification("⚡ Chute perfeito executado!", false)
        end
        
        task.wait(2)
        
        -- Esperar o Brainrot ser coletado (cerca de 5-8 segundos)
        showNotification("🔄 Aguardando Brainrot...", false)
        task.wait(6)
    end
    
    isKicking = false
end

local function startPerfectKickLoop()
    spawn(function()
        while Features.AutoPerfectKick do
            if not isKicking and not isReturningToBase then
                -- Verificar se está na base para iniciar novo chute
                if isAtBase() then
                    showNotification("🏠 Detectado na base! Iniciando novo chute...", false)
                    performPerfectKick()
                end
            end
            task.wait(2)
        end
    end)
end

-- FUNCIONALIDADE 2: Teleportar para Bolas
local function findAllBalls()
    allBalls = {}
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("Part") or obj:IsA("MeshPart") then
            if obj.Name:lower():find("ball") or obj.Name:lower():find("bola") then
                if obj.Position.Y > 0 then
                    table.insert(allBalls, obj)
                end
            end
        end
    end
    return #allBalls
end

local function startTeleportToBalls()
    spawn(function()
        while Features.TeleportToBalls do
            local ballCount = findAllBalls()
            
            if ballCount > 0 then
                for i, ball in pairs(allBalls) do
                    if not Features.TeleportToBalls then break end
                    
                    if ball and ball.Parent then
                        local ballPosition = ball.Position
                        -- Teleportar para a bola
                        rootPart.CFrame = CFrame.new(ballPosition + Vector3.new(0, 3, 0))
                        showNotification("📍 Teleportado para bola " .. i .. "/" .. ballCount, false)
                        task.wait(0.8)
                    end
                end
                showNotification("✅ Todas as bolas foram visitadas!", false)
            else
                showNotification("⚠️ Nenhuma bola encontrada no mapa!", true)
            end
            
            task.wait(3)
        end
    end)
end

-- FUNCIONALIDADE 3: Auto Voltar para Base (com Pathfinding)
local function findBasePosition()
    for _, part in pairs(workspace:GetDescendants()) do
        if part:IsA("Part") and (part.Name:lower():find("base") or part.Name:lower():find("spawn")) then
            return part.Position
        end
    end
    return nil
end

local function startAutoReturnBase()
    spawn(function()
        while Features.AutoReturnBase do
            if not isReturningToBase then
                local basePos = findBasePosition()
                
                if basePos and (rootPart.Position - basePos).Magnitude > 10 then
                    isReturningToBase = true
                    showNotification("🚶 Voltando para a base...", false)
                    
                    -- Criar path
                    local path = PathfindingService:CreatePath({
                        AgentRadius = 2,
                        AgentHeight = 5,
                        AgentCanJump = true,
                        AgentMaxSlope = 60
                    })
                    
                    local success = pcall(function()
                        path:ComputeAsync(rootPart.Position, basePos)
                    end)
                    
                    if success then
                        local waypoints = path:GetWaypoints()
                        
                        for _, waypoint in pairs(waypoints) do
                            if not Features.AutoReturnBase then
                                isReturningToBase = false
                                break
                            end
                            
                            humanoid:MoveTo(waypoint.Position)
                            humanoid.MoveToFinished:Wait(2)
                            
                            -- Pular se necessário
                            if waypoint.Action == Enum.PathWaypointAction.Jump then
                                humanoid.Jump = true
                                task.wait(0.5)
                            end
                        end
                    end
                    
                    showNotification("✅ Chegou à base!", false)
                    isReturningToBase = false
                end
            end
            task.wait(1)
        end
    end)
end

-- FUNCIONALIDADE 4: Auto 2x Peso
local function find2xButton()
    -- Procurar em todas as interfaces do jogador
    local guis = {player.PlayerGui, player:FindFirstChild("PlayerScripts")}
    
    for _, gui in pairs(guis) do
        if gui then
            for _, button in pairs(gui:GetDescendants()) do
                if button:IsA("TextButton") or button:IsA("ImageButton") then
                    local text = button.Text or ""
                    if text:lower():find("2x") or text:lower():find("double") or text:lower():find("multiplic") then
                        return button
                    end
                end
            end
        end
    end
    
    -- Procurar na tela
    for _, button in pairs(game:GetService("CoreGui"):GetDescendants()) do
        if button:IsA("TextButton") and button.Text:lower():find("2x") then
            return button
        end
    end
    
    return nil
end

local function startAuto2xWeight()
    if weightCheckInterval then
        weightCheckInterval:Disconnect()
    end
    
    weightCheckInterval = RunService.Heartbeat:Connect(function()
        if Features.Auto2xWeight then
            local btn = find2xButton()
            if btn and btn.Visible and btn.Active then
                btn:Click()
                showNotification("⚖️ Multiplicador 2x ativado!", false)
                task.wait(5) -- Esperar 5 segundos antes de tentar novamente
            end
        end
    end)
end

-- Inicialização
local showNotification = CreateUI()

-- Aguardar personagem
player.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = character:WaitForChild("Humanoid")
    rootPart = character:WaitForChild("HumanoidRootPart")
    showNotification("✅ Personagem atualizado!", false)
end)

-- Notificação de início
task.wait(1)
showNotification("🚀 Pietro Mods Carregado com sucesso!", false)
showNotification("💡 Ative as funcionalidades pelos botões", false)

-- Tratamento de erros global
local function safeCall(func, ...)
    local success, err = pcall(func, ...)
    if not success then
        warn("Erro: " .. tostring(err))
        if showNotification then
            showNotification("⚠️ Erro: " .. tostring(err):sub(1, 50), true)
        end
    end
    return success
end

-- Loop principal seguro
spawn(function()
    while true do
        safeCall(function()
            -- Manutenção do personagem
            if not character or not character.Parent then
                character = player.Character
                if character then
                    humanoid = character:WaitForChild("Humanoid")
                    rootPart = character:WaitForChild("HumanoidRootPart")
                end
            end
        end)
        task.wait(5)
    end
end)
