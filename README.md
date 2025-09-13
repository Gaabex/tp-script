-- Painel de Habilidades para Roblox
-- Coloque este script em StarterPlayer > StarterPlayerScripts
-- Pressione G para abrir/fechar o painel

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")
local mouse = player:GetMouse()

-- Variáveis de controle
local isGuiOpen = false
local teleportEnabled = false
local damageMultiplierEnabled = false
local teleportKey = "E"
local currentMultiplier = 2
local gui = nil

-- Função para criar efeito visual de teleporte
local function createTeleportEffect(position)
    local character = player.Character
    if not character then return end
    
    local effect = Instance.new("Part")
    effect.Name = "TeleportEffect"
    effect.Shape = Enum.PartType.Ball
    effect.Material = Enum.Material.Neon
    effect.BrickColor = BrickColor.new("Bright blue")
    effect.Size = Vector3.new(0.1, 0.1, 0.1)
    effect.Position = position
    effect.Anchored = true
    effect.CanCollide = false
    effect.Parent = workspace
    
    -- Animação do efeito
    local tweenInfo = TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    local tween = TweenService:Create(effect, tweenInfo, {
        Size = Vector3.new(10, 10, 10),
        Transparency = 1
    })
    
    tween:Play()
    tween.Completed:Connect(function()
        effect:Destroy()
    end)
end

-- Função de teleporte
local function teleportToMousePosition()
    if not teleportEnabled then return end
    
    local character = player.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
    
    local rootPart = character.HumanoidRootPart
    local hit = mouse.Hit
    
    if hit then
        local targetPosition = hit.Position + Vector3.new(0, 5, 0) -- Offset para não ficar dentro do chão
        rootPart.CFrame = CFrame.new(targetPosition)
        createTeleportEffect(targetPosition)
    end
end

-- Função para atualizar status visual
local function updateTeleportStatus(statusLabel, button)
    if teleportEnabled then
        statusLabel.Text = "STATUS: ATIVO"
        statusLabel.TextColor3 = Color3.new(0, 1, 0)
        button.BackgroundColor3 = Color3.new(0, 0.7, 0)
        button.Text = "DESATIVAR"
    else
        statusLabel.Text = "STATUS: DESATIVADO"
        statusLabel.TextColor3 = Color3.new(1, 0, 0)
        button.BackgroundColor3 = Color3.new(0.7, 0, 0)
        button.Text = "ATIVAR"
    end
end

local function updateDamageStatus(statusLabel, button)
    if damageMultiplierEnabled then
        statusLabel.Text = "STATUS: ATIVO (" .. currentMultiplier .. "x)"
        statusLabel.TextColor3 = Color3.new(0, 1, 0)
        button.BackgroundColor3 = Color3.new(0, 0.7, 0)
        button.Text = "DESATIVAR"
    else
        statusLabel.Text = "STATUS: DESATIVADO"
        statusLabel.TextColor3 = Color3.new(1, 0, 0)
        button.BackgroundColor3 = Color3.new(0.7, 0, 0)
        button.Text = "ATIVAR"
    end
end

-- Sistema de multiplicador de dano
local originalDamageFunction = nil
local function setupDamageMultiplier()
    local character = player.Character
    if not character then return end
    
    local humanoid = character:FindFirstChild("Humanoid")
    if not humanoid then return end
    
    -- Intercepta eventos de dano
    local connection
    connection = humanoid.HealthChanged:Connect(function(health)
        if damageMultiplierEnabled and health < humanoid.MaxHealth then
            -- Este é um exemplo básico - pode precisar de adaptação dependendo do sistema do jogo
            print("Damage multiplier active: " .. currentMultiplier .. "x")
        end
    end)
    
    -- Cleanup quando o personagem é removido
    character.AncestryChanged:Connect(function()
        if connection then
            connection:Disconnect()
        end
    end)
end

-- Função para detectar tecla personalizada
local function setupCustomKey(keyLabel, button)
    keyLabel.Text = "Pressione uma tecla..."
    keyLabel.TextColor3 = Color3.new(1, 1, 0)
    
    local connection
    connection = UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if gameProcessed then return end
        
        if input.UserInputType == Enum.UserInputType.Keyboard then
            teleportKey = input.KeyCode.Name
            keyLabel.Text = "TECLA: " .. teleportKey
            keyLabel.TextColor3 = Color3.new(1, 1, 1)
            connection:Disconnect()
        elseif input.UserInputType == Enum.UserInputType.MouseButton1 then
            teleportKey = "MouseButton1"
            keyLabel.Text = "TECLA: BOTÃO ESQUERDO"
            keyLabel.TextColor3 = Color3.new(1, 1, 1)
            connection:Disconnect()
        elseif input.UserInputType == Enum.UserInputType.MouseButton2 then
            teleportKey = "MouseButton2"
            keyLabel.Text = "TECLA: BOTÃO DIREITO"
            keyLabel.TextColor3 = Color3.new(1, 1, 1)
            connection:Disconnect()
        end
    end)
end

-- Função para criar a GUI
local function createGui()
    -- Remove GUI anterior se existir
    if gui then
        gui:Destroy()
    end
    
    -- ScreenGui principal
    gui = Instance.new("ScreenGui")
    gui.Name = "SkillsPanel"
    gui.Parent = playerGui
    gui.ResetOnSpawn = false
    
    -- Frame principal
    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 500, 0, 600)
    mainFrame.Position = UDim2.new(0.5, -250, 0.5, -300)
    mainFrame.BackgroundColor3 = Color3.new(0.1, 0.1, 0.1)
    mainFrame.BorderSizePixel = 0
    mainFrame.Active = true
    mainFrame.Draggable = true
    mainFrame.Parent = gui
    mainFrame.Visible = false
    
    -- Cantos arredondados
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 15)
    corner.Parent = mainFrame
    
    -- Título
    local title = Instance.new("TextLabel")
    title.Name = "Title"
    title.Size = UDim2.new(1, 0, 0, 50)
    title.Position = UDim2.new(0, 0, 0, 0)
    title.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
    title.BorderSizePixel = 0
    title.Text = "PAINEL DE HABILIDADES"
    title.TextColor3 = Color3.new(1, 1, 1)
    title.TextScaled = true
    title.Font = Enum.Font.GothamBold
    title.Parent = mainFrame
    
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 15)
    titleCorner.Parent = title
    
    -- Seção de Teleporte
    local teleportSection = Instance.new("Frame")
    teleportSection.Name = "TeleportSection"
    teleportSection.Size = UDim2.new(1, -20, 0, 250)
    teleportSection.Position = UDim2.new(0, 10, 0, 70)
    teleportSection.BackgroundColor3 = Color3.new(0.15, 0.15, 0.15)
    teleportSection.BorderSizePixel = 0
    teleportSection.Parent = mainFrame
    
    local teleportCorner = Instance.new("UICorner")
    teleportCorner.CornerRadius = UDim.new(0, 10)
    teleportCorner.Parent = teleportSection
    
    -- Título da seção teleporte
    local teleportTitle = Instance.new("TextLabel")
    teleportTitle.Size = UDim2.new(1, 0, 0, 30)
    teleportTitle.Position = UDim2.new(0, 0, 0, 5)
    teleportTitle.BackgroundTransparency = 1
    teleportTitle.Text = "TELEPORTE"
    teleportTitle.TextColor3 = Color3.new(0.8, 0.8, 1)
    teleportTitle.TextScaled = true
    teleportTitle.Font = Enum.Font.Gotham
    teleportTitle.Parent = teleportSection
    
    -- Status do teleporte
    local teleportStatus = Instance.new("TextLabel")
    teleportStatus.Size = UDim2.new(1, -10, 0, 25)
    teleportStatus.Position = UDim2.new(0, 5, 0, 40)
    teleportStatus.BackgroundTransparency = 1
    teleportStatus.Text = "STATUS: DESATIVADO"
    teleportStatus.TextColor3 = Color3.new(1, 0, 0)
    teleportStatus.TextScaled = true
    teleportStatus.Font = Enum.Font.Gotham
    teleportStatus.Parent = teleportSection
    
    -- Botão toggle teleporte
    local teleportToggle = Instance.new("TextButton")
    teleportToggle.Size = UDim2.new(0.45, 0, 0, 40)
    teleportToggle.Position = UDim2.new(0.05, 0, 0, 70)
    teleportToggle.BackgroundColor3 = Color3.new(0.7, 0, 0)
    teleportToggle.BorderSizePixel = 0
    teleportToggle.Text = "ATIVAR"
    teleportToggle.TextColor3 = Color3.new(1, 1, 1)
    teleportToggle.TextScaled = true
    teleportToggle.Font = Enum.Font.GothamBold
    teleportToggle.Parent = teleportSection
    
    local toggleCorner = Instance.new("UICorner")
    toggleCorner.CornerRadius = UDim.new(0, 8)
    toggleCorner.Parent = teleportToggle
    
    -- Botão teleportar agora
    local teleportNow = Instance.new("TextButton")
    teleportNow.Size = UDim2.new(0.45, 0, 0, 40)
    teleportNow.Position = UDim2.new(0.5, 0, 0, 70)
    teleportNow.BackgroundColor3 = Color3.new(0, 0.5, 0.8)
    teleportNow.BorderSizePixel = 0
    teleportNow.Text = "TELEPORTAR AGORA"
    teleportNow.TextColor3 = Color3.new(1, 1, 1)
    teleportNow.TextScaled = true
    teleportNow.Font = Enum.Font.GothamBold
    teleportNow.Parent = teleportSection
    
    local nowCorner = Instance.new("UICorner")
    nowCorner.CornerRadius = UDim.new(0, 8)
    nowCorner.Parent = teleportNow
    
    -- Label da tecla atual
    local keyLabel = Instance.new("TextLabel")
    keyLabel.Size = UDim2.new(1, -10, 0, 25)
    keyLabel.Position = UDim2.new(0, 5, 0, 120)
    keyLabel.BackgroundTransparency = 1
    keyLabel.Text = "TECLA: " .. teleportKey
    keyLabel.TextColor3 = Color3.new(1, 1, 1)
    keyLabel.TextScaled = true
    keyLabel.Font = Enum.Font.Gotham
    keyLabel.Parent = teleportSection
    
    -- Botão para escolher tecla
    local chooseKey = Instance.new("TextButton")
    chooseKey.Size = UDim2.new(1, -10, 0, 40)
    chooseKey.Position = UDim2.new(0, 5, 0, 150)
    chooseKey.BackgroundColor3 = Color3.new(0.3, 0.3, 0.3)
    chooseKey.BorderSizePixel = 0
    chooseKey.Text = "ESCOLHER NOVA TECLA"
    chooseKey.TextColor3 = Color3.new(1, 1, 1)
    chooseKey.TextScaled = true
    chooseKey.Font = Enum.Font.Gotham
    chooseKey.Parent = teleportSection
    
    local chooseCorner = Instance.new("UICorner")
    chooseCorner.CornerRadius = UDim.new(0, 8)
    chooseCorner.Parent = chooseKey
    
    -- Seção de Multiplicador de Dano
    local damageSection = Instance.new("Frame")
    damageSection.Name = "DamageSection"
    damageSection.Size = UDim2.new(1, -20, 0, 200)
    damageSection.Position = UDim2.new(0, 10, 0, 340)
    damageSection.BackgroundColor3 = Color3.new(0.15, 0.15, 0.15)
    damageSection.BorderSizePixel = 0
    damageSection.Parent = mainFrame
    
    local damageCorner = Instance.new("UICorner")
    damageCorner.CornerRadius = UDim.new(0, 10)
    damageCorner.Parent = damageSection
    
    -- Título da seção dano
    local damageTitle = Instance.new("TextLabel")
    damageTitle.Size = UDim2.new(1, 0, 0, 30)
    damageTitle.Position = UDim2.new(0, 0, 0, 5)
    damageTitle.BackgroundTransparency = 1
    damageTitle.Text = "MULTIPLICADOR DE DANO"
    damageTitle.TextColor3 = Color3.new(1, 0.8, 0.8)
    damageTitle.TextScaled = true
    damageTitle.Font = Enum.Font.Gotham
    damageTitle.Parent = damageSection
    
    -- Status do multiplicador
    local damageStatus = Instance.new("TextLabel")
    damageStatus.Size = UDim2.new(1, -10, 0, 25)
    damageStatus.Position = UDim2.new(0, 5, 0, 40)
    damageStatus.BackgroundTransparency = 1
    damageStatus.Text = "STATUS: DESATIVADO"
    damageStatus.TextColor3 = Color3.new(1, 0, 0)
    damageStatus.TextScaled = true
    damageStatus.Font = Enum.Font.Gotham
    damageStatus.Parent = damageSection
    
    -- Botão toggle multiplicador
    local damageToggle = Instance.new("TextButton")
    damageToggle.Size = UDim2.new(0.45, 0, 0, 40)
    damageToggle.Position = UDim2.new(0.05, 0, 0, 70)
    damageToggle.BackgroundColor3 = Color3.new(0.7, 0, 0)
    damageToggle.BorderSizePixel = 0
    damageToggle.Text = "ATIVAR"
    damageToggle.TextColor3 = Color3.new(1, 1, 1)
    damageToggle.TextScaled = true
    damageToggle.Font = Enum.Font.GothamBold
    damageToggle.Parent = damageSection
    
    local damageToggleCorner = Instance.new("UICorner")
    damageToggleCorner.CornerRadius = UDim.new(0, 8)
    damageToggleCorner.Parent = damageToggle
    
    -- TextBox para multiplicador
    local multiplierBox = Instance.new("TextBox")
    multiplierBox.Size = UDim2.new(0.45, 0, 0, 40)
    multiplierBox.Position = UDim2.new(0.5, 0, 0, 70)
    multiplierBox.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
    multiplierBox.BorderSizePixel = 0
    multiplierBox.Text = "2"
    multiplierBox.PlaceholderText = "Multiplicador"
    multiplierBox.TextColor3 = Color3.new(1, 1, 1)
    multiplierBox.TextScaled = true
    multiplierBox.Font = Enum.Font.Gotham
    multiplierBox.Parent = damageSection
    
    local boxCorner = Instance.new("UICorner")
    boxCorner.CornerRadius = UDim.new(0, 8)
    boxCorner.Parent = multiplierBox
    
    -- Label explicativa
    local explanationLabel = Instance.new("TextLabel")
    explanationLabel.Size = UDim2.new(1, -10, 0, 50)
    explanationLabel.Position = UDim2.new(0, 5, 0, 120)
    explanationLabel.BackgroundTransparency = 1
    explanationLabel.Text = "Digite o valor do multiplicador acima\n(ex: 2 = dano dobrado, 5 = dano 5x maior)"
    explanationLabel.TextColor3 = Color3.new(0.8, 0.8, 0.8)
    explanationLabel.TextScaled = true
    explanationLabel.Font = Enum.Font.Gotham
    explanationLabel.Parent = damageSection
    
    -- Botão fechar
    local closeButton = Instance.new("TextButton")
    closeButton.Size = UDim2.new(0, 30, 0, 30)
    closeButton.Position = UDim2.new(1, -40, 0, 10)
    closeButton.BackgroundColor3 = Color3.new(0.8, 0, 0)
    closeButton.BorderSizePixel = 0
    closeButton.Text = "X"
    closeButton.TextColor3 = Color3.new(1, 1, 1)
    closeButton.TextScaled = true
    closeButton.Font = Enum.Font.GothamBold
    closeButton.Parent = mainFrame
    
    local closeCorner = Instance.new("UICorner")
    closeCorner.CornerRadius = UDim.new(0, 15)
    closeCorner.Parent = closeButton
    
    -- Eventos dos botões
    teleportToggle.MouseButton1Click:Connect(function()
        teleportEnabled = not teleportEnabled
        updateTeleportStatus(teleportStatus, teleportToggle)
    end)
    
    teleportNow.MouseButton1Click:Connect(function()
        teleportToMousePosition()
    end)
    
    chooseKey.MouseButton1Click:Connect(function()
        setupCustomKey(keyLabel, chooseKey)
    end)
    
    damageToggle.MouseButton1Click:Connect(function()
        damageMultiplierEnabled = not damageMultiplierEnabled
        updateDamageStatus(damageStatus, damageToggle)
        if damageMultiplierEnabled then
            setupDamageMultiplier()
        end
    end)
    
    multiplierBox.FocusLost:Connect(function()
        local value = tonumber(multiplierBox.Text)
        if value and value > 0 then
            currentMultiplier = value
            updateDamageStatus(damageStatus, damageToggle)
        else
            multiplierBox.Text = tostring(currentMultiplier)
        end
    end)
    
    closeButton.MouseButton1Click:Connect(function()
        toggleGui()
    end)
    
    -- Inicializar status
    updateTeleportStatus(teleportStatus, teleportToggle)
    updateDamageStatus(damageStatus, damageToggle)
    
    return mainFrame
end

-- Função para abrir/fechar GUI
function toggleGui()
    if not gui then
        local mainFrame = createGui()
        isGuiOpen = true
        mainFrame.Visible = true
        
        -- Animação de entrada
        mainFrame.Size = UDim2.new(0, 0, 0, 0)
        local tweenInfo = TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
        local tween = TweenService:Create(mainFrame, tweenInfo, {
            Size = UDim2.new(0, 500, 0, 600)
        })
        tween:Play()
    else
        isGuiOpen = not isGuiOpen
        gui:FindFirstChild("MainFrame").Visible = isGuiOpen
        
        if isGuiOpen then
            local mainFrame = gui:FindFirstChild("MainFrame")
            mainFrame.Size = UDim2.new(0, 0, 0, 0)
            local tweenInfo = TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
            local tween = TweenService:Create(mainFrame, tweenInfo, {
                Size = UDim2.new(0, 500, 0, 600)
            })
            tween:Play()
        end
    end
end

-- Input handling
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    -- Abrir/fechar painel com G
    if input.KeyCode == Enum.KeyCode.G then
        toggleGui()
    end
    
    -- Teleporte com tecla customizada
    if teleportEnabled then
        if input.KeyCode.Name == teleportKey then
            teleportToMousePosition()
        elseif teleportKey == "MouseButton1" and input.UserInputType == Enum.UserInputType.MouseButton1 then
            teleportToMousePosition()
        elseif teleportKey == "MouseButton2" and input.UserInputType == Enum.UserInputType.MouseButton2 then
            teleportToMousePosition()
        end
    end
end)

-- Recriar GUI quando o jogador respawn
player.CharacterAdded:Connect(function()
    wait(1) -- Esperar um pouco para o character carregar completamente
    if isGuiOpen then
        createGui()
        gui:FindFirstChild("MainFrame").Visible = true
    end
    
    -- Reconfigurar multiplicador de dano se estiver ativo
    if damageMultiplierEnabled then
        setupDamageMultiplier()
    end
end)

print("Painel de Habilidades carregado! Pressione G para abrir/fechar.")
