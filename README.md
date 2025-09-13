-- StarterPlayerScripts/TeleportAbility_NoCooldown_Unlimited.lua
-- Teleporte mirando (sem cooldown, alcance ilimitado) + GUI toggle de ativação
-- INSTALAÇÃO: cole como LocalScript em StarterPlayerScripts

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local mouse = player:GetMouse()

-- CONFIGURAÇÃO
local TELEPORT_KEY = Enum.KeyCode.F    -- atalho (troque se quiser)
local ENABLED_BY_DEFAULT = true        -- true: começa ativo
local RISE_ON_TELEPORT = 3             -- sobe o destino pra evitar prender no chão

-- ESTADO
local enabled = ENABLED_BY_DEFAULT

-- UTIL
local function getRoot(character)
	return character and (character:FindFirstChild("HumanoidRootPart") or character:FindFirstChild("Torso"))
end

-- EFEITO VISUAL SIMPLES (cria um part temporário no destino)
local function spawnEffectAt(position)
	local part = Instance.new("Part")
	part.Size = Vector3.new(0.5, 0.5, 0.5)
	part.Anchored = true
	part.CanCollide = false
	part.Transparency = 0.3
	part.Shape = Enum.PartType.Ball
	part.CFrame = CFrame.new(position)
	part.Parent = workspace

	-- animação simples: cresce e some
	local elapsed = 0
	local duration = 0.35
	local initialSize = part.Size
	local targetSize = initialSize * 4

	local connection
	connection = RunService.Heartbeat:Connect(function(dt)
		elapsed = elapsed + dt
		local t = math.min(elapsed / duration, 1)
		part.Size = initialSize:Lerp(targetSize, t)
		if t >= 1 then
			connection:Disconnect()
			part:Destroy()
		end
	end)
end

-- TELEPORT FUNCTION (sem limite de distância)
local function doTeleport()
	local char = player.Character
	if not char then return end
	local root = getRoot(char)
	if not root then return end

	-- mouse.Hit pode ser nil se mirar no céu muito longe; validar
	local hitCFrame = mouse.Hit
	if not hitCFrame then
		-- nada a fazer
		return
	end

	local targetPos = hitCFrame.p
	-- opcional: se targetPos for NaN/inf, abortar
	if targetPos ~= targetPos then return end

	-- eleva para evitar prender em superfícies
	local finalPos = targetPos + Vector3.new(0, RISE_ON_TELEPORT, 0)

	-- efeito visual (antes ou depois da mudança)
	spawnEffectAt(finalPos)

	-- aplica teleporte localmente
	-- usamos SetPrimaryPartCFrame quando disponível; prox fallback para ajustar CFrame do root
	local success, err = pcall(function()
		if char.PrimaryPart then
			char:SetPrimaryPartCFrame(CFrame.new(finalPos))
		else
			root.CFrame = CFrame.new(finalPos)
		end
	end)
	if not success then
		warn("Erro ao teleportar:", err)
	end
end

-- INPUT (tecla)
UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then return end
	if input.KeyCode == TELEPORT_KEY and enabled then
		doTeleport()
	end
end)

-- -------------------------
-- GUI: Toggle simples e indicador de status
-- -------------------------
local function createGui()
	-- parent no PlayerGui
	local playerGui = player:WaitForChild("PlayerGui")

	-- ScreenGui
	local screenGui = Instance.new("ScreenGui")
	screenGui.Name = "TeleportAbilityGui"
	screenGui.ResetOnSpawn = false
	screenGui.Parent = playerGui

	-- Frame base
	local frame = Instance.new("Frame")
	frame.Size = UDim2.new(0, 170, 0, 56)
	frame.Position = UDim2.new(0, 12, 0, 12)
	frame.AnchorPoint = Vector2.new(0, 0)
	frame.BackgroundTransparency = 0.25
	frame.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
	frame.BorderSizePixel = 0
	frame.Parent = screenGui

	-- Label (título)
	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(0, 140, 0, 22)
	label.Position = UDim2.new(0, 8, 0, 4)
	label.BackgroundTransparency = 1
	label.Text = "Teleport habilidade"
	label.TextColor3 = Color3.fromRGB(255, 255, 255)
	label.TextScaled = true
	label.Font = Enum.Font.SourceSansBold
	label.TextXAlignment = Enum.TextXAlignment.Left
	label.Parent = frame

	-- Status indicator (círculo)
	local status = Instance.new("TextLabel")
	status.Size = UDim2.new(0, 18, 0, 18)
	status.Position = UDim2.new(0, 8, 0, 30)
	status.BackgroundTransparency = 0
	status.Text = ""
	status.BorderSizePixel = 0
	status.AnchorPoint = Vector2.new(0, 0)
	status.Parent = frame

	local function updateStatus()
		if enabled then
			status.BackgroundColor3 = Color3.fromRGB(0, 200, 0) -- verde
		else
			status.BackgroundColor3 = Color3.fromRGB(200, 0, 0) -- vermelho
		end
	end
	updateStatus()

	-- Toggle Button
	local toggleBtn = Instance.new("TextButton")
	toggleBtn.Size = UDim2.new(0, 86, 0, 26)
	toggleBtn.Position = UDim2.new(0, 76, 0, 28)
	toggleBtn.Text = enabled and "Desativar" or "Ativar"
	toggleBtn.Font = Enum.Font.SourceSans
	toggleBtn.TextScaled = true
	toggleBtn.Parent = frame

	toggleBtn.MouseButton1Click:Connect(function()
		enabled = not enabled
		toggleBtn.Text = enabled and "Desativar" or "Ativar"
		updateStatus()
	end)

	-- Dica: clique no texto do frame abre uma pequena explicação
	frame.Active = true
	frame.Draggable = true
end

createGui()

-- pronto
print("TeleportAbility (sem cooldown / alcance ilimitado) carregado. Ativo =", tostring(enabled))
