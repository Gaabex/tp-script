-- LocalScript: StarterPlayerScripts/TeleportPanel.lua
-- Painel (G) + toggle + escolher botão/tecla + teleport sem limite de distância
-- Instale: cole como LocalScript em StarterPlayerScripts

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local mouse = player:GetMouse()

-- TECLA QUE ABRE O PAINEL
local TOGGLE_KEY = Enum.KeyCode.G

-- Estado
local teleportEnabled = false
local listeningForTrigger = false

-- Trigger padrão: clique esquerdo do mouse (você pode mudar via GUI)
local trigger = { type = "Mouse", button = Enum.UserInputType.MouseButton1 }

-- UTIL: pega HumanoidRootPart ou Torso
local function getRoot(char)
	return char and (char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("Torso"))
end

-- Efeito visual simples no destino
local function spawnEffectAt(position)
	local part = Instance.new("Part")
	part.Size = Vector3.new(0.6, 0.6, 0.6)
	part.Anchored = true
	part.CanCollide = false
	part.Transparency = 0.25
	part.Shape = Enum.PartType.Ball
	part.Material = Enum.Material.Neon
	part.CFrame = CFrame.new(position)
	part.Parent = workspace

	local elapsed = 0
	local duration = 0.35
	local initial = part.Size
	local target = initial * 3

	local conn
	conn = RunService.Heartbeat:Connect(function(dt)
		elapsed = elapsed + dt
		local t = math.min(elapsed / duration, 1)
		part.Size = initial:Lerp(target, t)
		if t >= 1 then
			conn:Disconnect()
			part:Destroy()
		end
	end)
end

-- FUNÇÃO DE TELEPORT (sem limite de distância)
local function doTeleport()
	local char = player.Character
	if not char then return end
	local root = getRoot(char)
	if not root then
		warn("Teleport: character root não encontrado.")
		return
	end

	-- mouse.Hit é um CFrame para onde a mira está apontando
	local hitCFrame = mouse.Hit
	if not hitCFrame then
		warn("Teleport: mouse.Hit indefinido.")
		return
	end

	local dest = hitCFrame.p
	-- validação básica: evitar NaN/inf
	if dest ~= dest then
		warn("Teleport: posição inválida.")
		return
	end

	local finalPos = dest + Vector3.new(0, 3, 0) -- sobe pra evitar prender no chão
	spawnEffectAt(finalPos)

	-- aplica teleporte localmente
	local ok, err = pcall(function()
		if char.PrimaryPart then
			char:SetPrimaryPartCFrame(CFrame.new(finalPos))
		else
			root.CFrame = CFrame.new(finalPos)
		end
	end)
	if not ok then
		warn("Erro ao teleportar:", err)
	end
end

-- UTIL: converte trigger para texto legível
local function triggerToString(t)
	if not t then return "Nenhum" end
	if t.type == "Key" then
		return tostring(t.key.Name or tostring(t.key))
	elseif t.type == "Mouse" then
		-- mapeia os nomes das enums mais comuns
		local mt = t.button
		if mt == Enum.UserInputType.MouseButton1 then return "Mouse: Esquerdo" end
		if mt == Enum.UserInputType.MouseButton2 then return "Mouse: Direito" end
		if mt == Enum.UserInputType.MouseButton3 then return "Mouse: Meio" end
		return "Mouse: "..tostring(mt)
	end
	return tostring(t)
end

-- CHECA SE input recebido bate com o trigger selecionado
local function isTriggerInput(input)
	if not input then return false end
	if trigger.type == "Key" then
		-- teclas: UserInputType deve ser Keyboard e KeyCode igual
		return input.UserInputType == Enum.UserInputType.Keyboard and input.KeyCode == trigger.key
	elseif trigger.type == "Mouse" then
		return input.UserInputType == trigger.button
	end
	return false
end

-- CRIA GUI
local function createGui()
	-- evita duplicar
	if player:FindFirstChild("PlayerGui") and player.PlayerGui:FindFirstChild("TeleportPanelGui") then
		player.PlayerGui.TeleportPanelGui:Destroy()
	end

	local screenGui = Instance.new("ScreenGui")
	screenGui.Name = "TeleportPanelGui"
	screenGui.ResetOnSpawn = false
	screenGui.Parent = player:WaitForChild("PlayerGui")

	local frame = Instance.new("Frame")
	frame.Size = UDim2.new(0, 220, 0, 120)
	frame.Position = UDim2.new(0, 12, 0, 12)
	frame.AnchorPoint = Vector2.new(0, 0)
	frame.BackgroundColor3 = Color3.fromRGB(28, 28, 28)
	frame.BorderSizePixel = 0
	frame.BackgroundTransparency = 0.05
	frame.Parent = screenGui
	frame.Active = true
	frame.Draggable = true

	local title = Instance.new("TextLabel")
	title.Size = UDim2.new(0, 120, 0, 28)
	title.Position = UDim2.new(0, 8, 0, 6)
	title.BackgroundTransparency = 1
	title.Text = "Teleporte"
	title.TextColor3 = Color3.new(1,1,1)
	title.Font = Enum.Font.SourceSansBold
	title.TextSize = 18
	title.TextXAlignment = Enum.TextXAlignment.Left
	title.Parent = frame

	-- status circle
	local status = Instance.new("TextLabel")
	status.Size = UDim2.new(0, 18, 0, 18)
	status.Position = UDim2.new(0, 140, 0, 8)
	status.BackgroundTransparency = 0
	status.BorderSizePixel = 0
	status.Text = ""
	status.Parent = frame

	local function updateStatusUI()
		if teleportEnabled then
			status.BackgroundColor3 = Color3.fromRGB(0, 200, 0)
		else
			status.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
		end
	end
	updateStatusUI()

	-- Toggle button (Ativar/Desativar)
	local toggleBtn = Instance.new("TextButton")
	toggleBtn.Size = UDim2.new(0, 70, 0, 28)
	toggleBtn.Position = UDim2.new(0, 144, 0, 6)
	toggleBtn.Text = teleportEnabled and "Desativar" or "Ativar"
	toggleBtn.Font = Enum.Font.SourceSans
	toggleBtn.TextSize = 14
	toggleBtn.Parent = frame

	-- Botão "Teleportar agora"
	local tpNowBtn = Instance.new("TextButton")
	tpNowBtn.Size = UDim2.new(0, 100, 0, 28)
	tpNowBtn.Position = UDim2.new(0, 8, 0, 40)
	tpNowBtn.Text = "Teleportar agora"
	tpNowBtn.Font = Enum.Font.SourceSans
	tpNowBtn.TextSize = 14
	tpNowBtn.Parent = frame

	-- Botão escolher trigger
	local chooseBtn = Instance.new("TextButton")
	chooseBtn.Size = UDim2.new(0, 100, 0, 28)
	chooseBtn.Position = UDim2.new(0, 112, 0, 40)
	chooseBtn.Text = "Escolher botão"
	chooseBtn.Font = Enum.Font.SourceSans
	chooseBtn.TextSize = 14
	chooseBtn.Parent = frame

	-- Label que mostra o trigger atual
	local currentLabel = Instance.new("TextLabel")
	currentLabel.Size = UDim2.new(0, 200, 0, 20)
	currentLabel.Position = UDim2.new(0, 8, 0, 76)
	currentLabel.BackgroundTransparency = 1
	currentLabel.Text = "Botão atual: " .. triggerToString(trigger)
	currentLabel.TextColor3 = Color3.new(1,1,1)
	currentLabel.Font = Enum.Font.SourceSans
	currentLabel.TextSize = 14
	currentLabel.TextXAlignment = Enum.TextXAlignment.Left
	currentLabel.Parent = frame

	-- função para atualizar textos
	local function refreshUI()
		toggleBtn.Text = teleportEnabled and "Desativar" or "Ativar"
		currentLabel.Text = "Botão atual: " .. triggerToString(trigger)
		updateStatusUI()
	end

	-- toggle handler
	toggleBtn.MouseButton1Click:Connect(function()
		teleportEnabled = not teleportEnabled
		refreshUI()
	end)

	-- teleport now
	tpNowBtn.MouseButton1Click:Connect(function()
		doTeleport()
	end)

	-- escolher trigger: ao clicar, aguarda próximo input do jogador
	local captureConn
	chooseBtn.MouseButton1Click:Connect(function()
		if listeningForTrigger then
			-- se já estiver ouvindo, cancela
			listeningForTrigger = false
			if captureConn then captureConn:Disconnect() end
			chooseBtn.Text = "Escolher botão"
			refreshUI()
			return
		end

		listeningForTrigger = true
		chooseBtn.Text = "Pressione agora..."
		currentLabel.Text = "Aguardando input (pressione tecla/botão)..."

		-- temporária: captura o próximo InputBegan (aceita teclado ou mouse)
		captureConn = UserInputService.InputBegan:Connect(function(input, gameProcessed)
			-- ignorar Inputs do tipo Enum.UserInputType.Focus ou outros irrelevantes
			-- Aceitamos independentemente de gameProcessed enquanto estiver escolhendo
			-- Bloqueamos apenas se for a tecla de toggle (evitar conflito com G)
			if input.UserInputType == Enum.UserInputType.Keyboard then
				local kc = input.KeyCode
				if kc == TOGGLE_KEY then
					-- não permita usar a mesma tecla de toggle para evitar conflito
					currentLabel.Text = "Não pode usar a tecla de toggle ("..tostring(TOGGLE_KEY.Name).."). Escolha outra."
					wait(1.2)
					-- volta ao estado de escolha
					chooseBtn.Text = "Escolher botão"
					listeningForTrigger = false
					captureConn:Disconnect()
					captureConn = nil
					refreshUI()
					return
				end

				-- define trigger para tecla
				trigger = { type = "Key", key = kc }
				listeningForTrigger = false
				chooseBtn.Text = "Escolher botão"
				captureConn:Disconnect()
				captureConn = nil
				refreshUI()
			else
				-- Mouse buttons (ex: MouseButton1, MouseButton2, MouseButton3)
				local uit = input.UserInputType
				-- Aceita apenas mouse buttons
				if uit == Enum.UserInputType.MouseButton1
					or uit == Enum.UserInputType.MouseButton2
					or uit == Enum.UserInputType.MouseButton3
					or uit == Enum.UserInputType.MouseButton4
					or uit == Enum.UserInputType.MouseButton5 then

					trigger = { type = "Mouse", button = uit }
					listeningForTrigger = false
					chooseBtn.Text = "Escolher botão"
					captureConn:Disconnect()
					captureConn = nil
					refreshUI()
				else
					-- se não for um mouse button, ignore e continue esperando
					-- (por exemplo, tocar na tela ou outros eventos não desejados)
				end
			end
		end)
	end)

	-- inicializa oculto (painel fechado por padrão)
	screenGui.Enabled = false

	-- retorna referências úteis
	return screenGui, refreshUI
end

local screenGui, refreshUI = createGui()

-- MANAGER: Input global
UserInputService.InputBegan:Connect(function(input, gameProcessed)
	-- Toggle do painel (G)
	if input.UserInputType == Enum.UserInputType.Keyboard and input.KeyCode == TOGGLE_KEY and not listeningForTrigger then
		-- alterna visibilidade do painel
		screenGui.Enabled = not screenGui.Enabled
		return
	end

	-- se UI está capturando trigger, ignore este listener (captura temporária faz o trabalho)
	if listeningForTrigger then return end

	-- evita interagir quando o input foi processado pela GUI (evita teleport ao clicar em botões)
	if gameProcessed then return end

	-- só teleporta se habilidade ativada
	if teleportEnabled and isTriggerInput(input) then
		doTeleport()
	end
end)

-- print de debug
print("TeleportPanel carregado. Pressione '"..TOGGLE_KEY.Name.."' para abrir o painel.")
