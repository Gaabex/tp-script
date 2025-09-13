-- LocalScript: StarterPlayerScripts/TeleportAndDamagePanel.lua
-- Painel Teleporte + Multiplicador de Dano (2x padrão)
-- Instale: LocalScript em StarterPlayerScripts

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local mouse = player:GetMouse()

-- ============================
-- CONFIGURAÇÃO
-- ============================
local PANEL_TOGGLE_KEY = Enum.KeyCode.G -- tecla para abrir/fechar painel
local TELEPORT_KEY_DEFAULT = Enum.UserInputType.MouseButton1 -- trigger default do teleporte
local TELEPORT_RISE = 3 -- sobe o destino pra não prender no chão
local DAMAGE_DEFAULT = 2 -- multiplicador de dano padrão

-- ============================
-- ESTADOS
-- ============================
local teleportEnabled = true
local teleportTrigger = {type="Mouse", button=TELEPORT_KEY_DEFAULT}
local listeningForTrigger = false

local damageEnabled = true
local damageMultiplier = DAMAGE_DEFAULT

-- ============================
-- FUNÇÕES AUXILIARES
-- ============================
local function getRoot(char)
	return char and (char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("Torso"))
end

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
		local t = math.min(elapsed/duration,1)
		part.Size = initial:Lerp(target,t)
		if t >= 1 then conn:Disconnect() part:Destroy() end
	end)
end

-- ============================
-- TELEPORT
-- ============================
local function doTeleport()
	local char = player.Character
	if not char then return end
	local root = getRoot(char)
	if not root then return end
	local hitCFrame = mouse.Hit
	if not hitCFrame then return end
	local dest = hitCFrame.p
	if dest ~= dest then return end -- evita NaN
	local finalPos = dest + Vector3.new(0, TELEPORT_RISE, 0)
	spawnEffectAt(finalPos)
	local ok, err = pcall(function()
		if char.PrimaryPart then
			char:SetPrimaryPartCFrame(CFrame.new(finalPos))
		else
			root.CFrame = CFrame.new(finalPos)
		end
	end)
	if not ok then warn("Erro ao teleportar:", err) end
end

-- ============================
-- TRIGGER / INPUT
-- ============================
local function triggerToString(t)
	if not t then return "Nenhum" end
	if t.type=="Key" then return tostring(t.key.Name or t.key) end
	if t.type=="Mouse" then
		local b=t.button
		if b==Enum.UserInputType.MouseButton1 then return "Mouse Esquerdo" end
		if b==Enum.UserInputType.MouseButton2 then return "Mouse Direito" end
		if b==Enum.UserInputType.MouseButton3 then return "Mouse Meio" end
		return "Mouse: "..tostring(b)
	end
	return tostring(t)
end

local function isTriggerInput(input)
	if not input then return false end
	if teleportTrigger.type=="Key" then
		return input.UserInputType==Enum.UserInputType.Keyboard and input.KeyCode==teleportTrigger.key
	elseif teleportTrigger.type=="Mouse" then
		return input.UserInputType==teleportTrigger.button
	end
	return false
end

-- ============================
-- GUI
-- ============================
local function createGui()
	local playerGui = player:WaitForChild("PlayerGui")
	if playerGui:FindFirstChild("TeleportDamageGui") then
		playerGui.TeleportDamageGui:Destroy()
	end

	local screenGui = Instance.new("ScreenGui")
	screenGui.Name = "TeleportDamageGui"
	screenGui.ResetOnSpawn = false
	screenGui.Parent = playerGui

	local frame = Instance.new("Frame")
	frame.Size = UDim2.new(0, 250, 0, 180)
	frame.Position = UDim2.new(0, 12, 0, 12)
	frame.BackgroundColor3 = Color3.fromRGB(20,20,20)
	frame.BorderSizePixel = 0
	frame.BackgroundTransparency = 0.05
	frame.Active = true
	frame.Draggable = true
	frame.Parent = screenGui

	-- Título
	local title = Instance.new("TextLabel")
	title.Size = UDim2.new(0,200,0,28)
	title.Position = UDim2.new(0,8,0,6)
	title.BackgroundTransparency = 1
	title.Text="Painel de Habilidades"
	title.Font=Enum.Font.SourceSansBold
	title.TextColor3=Color3.fromRGB(255,255,255)
	title.TextSize=18
	title.TextXAlignment = Enum.TextXAlignment.Left
	title.Parent = frame

	-- ===== Teleporte =====
	local tpLabel = Instance.new("TextLabel")
	tpLabel.Size = UDim2.new(0,100,0,20)
	tpLabel.Position=UDim2.new(0,8,0,40)
	tpLabel.BackgroundTransparency=1
	tpLabel.Text="Teleporte:"
	tpLabel.TextColor3=Color3.fromRGB(255,255,255)
	tpLabel.Font=Enum.Font.SourceSans
	tpLabel.TextSize=16
	tpLabel.TextXAlignment=Enum.TextXAlignment.Left
	tpLabel.Parent=frame

	local tpStatus = Instance.new("TextLabel")
	tpStatus.Size=UDim2.new(0,20,0,20)
	tpStatus.Position=UDim2.new(0,100,0,40)
	tpStatus.BackgroundTransparency=0
	tpStatus.Text=""
	tpStatus.Parent=frame

	local tpToggle = Instance.new("TextButton")
	tpToggle.Size=UDim2.new(0,80,0,20)
	tpToggle.Position=UDim2.new(0,130,0,40)
	tpToggle.Text = teleportEnabled and "Desativar" or "Ativar"
	tpToggle.Font=Enum.Font.SourceSans
	tpToggle.TextSize=14
	tpToggle.Parent=frame

	local tpNowBtn = Instance.new("TextButton")
	tpNowBtn.Size=UDim2.new(0,80,0,20)
	tpNowBtn.Position=UDim2.new(0,130,0,65)
	tpNowBtn.Text="Teleportar agora"
	tpNowBtn.Font=Enum.Font.SourceSans
	tpNowBtn.TextSize=14
	tpNowBtn.Parent=frame

	local tpChooseBtn = Instance.new("TextButton")
	tpChooseBtn.Size=UDim2.new(0,100,0,20)
	tpChooseBtn.Position=UDim2.new(0,8,0,65)
	tpChooseBtn.Text="Escolher botão"
	tpChooseBtn.Font=Enum.Font.SourceSans
	tpChooseBtn.TextSize=14
	tpChooseBtn.Parent=frame

	local tpCurrentLabel = Instance.new("TextLabel")
	tpCurrentLabel.Size=UDim2.new(0,230,0,20)
	tpCurrentLabel.Position=UDim2.new(0,8,0,90)
	tpCurrentLabel.BackgroundTransparency=1
	tpCurrentLabel.Text="Botão atual: "..triggerToString(teleportTrigger)
	tpCurrentLabel.TextColor3=Color3.fromRGB(255,255,255)
	tpCurrentLabel.Font=Enum.Font.SourceSans
	tpCurrentLabel.TextSize=14
	tpCurrentLabel.TextXAlignment=Enum.TextXAlignment.Left
	tpCurrentLabel.Parent=frame

	-- ===== Multiplicador de Dano =====
	local dmgLabel = Instance.new("TextLabel")
	dmgLabel.Size=UDim2.new(0,120,0,20)
	dmgLabel.Position=UDim2.new(0,8,0,120)
	dmgLabel.BackgroundTransparency=1
	dmgLabel.Text="Multiplicador de dano:"
	dmgLabel.TextColor3=Color3.fromRGB(255,255,255)
	dmgLabel.Font=Enum.Font.SourceSans
	dmgLabel.TextSize=16
	dmgLabel.TextXAlignment=Enum.TextXAlignment.Left
	dmgLabel.Parent=frame

	local dmgBox = Instance.new("TextBox")
	dmgBox.Size=UDim2.new(0,50,0,20)
	dmgBox.Position=UDim2.new(0,140,0,120)
	dmgBox.Text=tostring(damageMultiplier)
	dmgBox.ClearTextOnFocus=false
	dmgBox.Font=Enum.Font.SourceSans
	dmgBox.TextSize=14
	dmgBox.TextColor3=Color3.fromRGB(0,0,0)
	dmgBox.BackgroundColor3=Color3.fromRGB(200,200,200)
	dmgBox.Parent=frame

	local dmgToggle = Instance.new("TextButton")
	dmgToggle.Size=UDim2.new(0,80,0,20)
	dmgToggle.Position=UDim2.new(0,190,0,120)
	dmgToggle.Text=damageEnabled and "Desativar" or "Ativar"
	dmgToggle.Font=Enum.Font.SourceSans
	dmgToggle.TextSize=14
	dmgToggle.Parent=frame

	-- ===== Funções internas =====
	local function refreshUI()
		tpToggle.Text = teleportEnabled and "Desativar" or "Ativar"
		tpStatus
