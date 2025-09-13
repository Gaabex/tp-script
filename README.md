-- LocalScript: StarterPlayerScripts/TeleportAndDamagePanelFixed.lua
-- Painel Teleporte + Multiplicador de Dano
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local mouse = player:GetMouse()

-- CONFIGURAÇÃO
local PANEL_TOGGLE_KEY = Enum.KeyCode.G
local TELEPORT_RISE = 3
local DAMAGE_DEFAULT = 2

-- ESTADO
local teleportEnabled = true
local teleportTrigger = {type="Mouse", button=Enum.UserInputType.MouseButton1}
local listeningForTrigger = false

local damageEnabled = true
local damageMultiplier = DAMAGE_DEFAULT

-- UTILIDADES
local function getRoot(char)
	return char and (char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("Torso"))
end

local function spawnEffect(pos)
	local p = Instance.new("Part")
	p.Size = Vector3.new(0.6,0.6,0.6)
	p.Anchored=true
	p.CanCollide=false
	p.Transparency=0.25
	p.Shape=Enum.PartType.Ball
	p.Material=Enum.Material.Neon
	p.CFrame=CFrame.new(pos)
	p.Parent=workspace

	local elapsed=0
	local dur=0.35
	local initial=p.Size
	local target=initial*3
	local conn
	conn=RunService.Heartbeat:Connect(function(dt)
		elapsed=elapsed+dt
		local t=math.min(elapsed/dur,1)
		p.Size=initial:Lerp(target,t)
		if t>=1 then conn:Disconnect() p:Destroy() end
	end)
end

-- TELEPORT
local function doTeleport()
	local char=player.Character
	if not char then return end
	local root=getRoot(char)
	if not root then return end
	local hitCFrame = mouse.Hit
	if not hitCFrame then return end
	local dest = hitCFrame.p
	if dest ~= dest then return end
	local finalPos = dest + Vector3.new(0, TELEPORT_RISE,0)
	spawnEffect(finalPos)
	pcall(function()
		if char.PrimaryPart then
			char:SetPrimaryPartCFrame(CFrame.new(finalPos))
		else
			root.CFrame=CFrame.new(finalPos)
		end
	end)
end

-- TRIGGER INPUT
local function isTriggerInput(input)
	if not input then return false end
	if teleportTrigger.type=="Key" then
		return input.UserInputType==Enum.UserInputType.Keyboard and input.KeyCode==teleportTrigger.key
	elseif teleportTrigger.type=="Mouse" then
		return input.UserInputType==teleportTrigger.button
	end
	return false
end

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

-- GUI
local function createGui()
	local playerGui=player:WaitForChild("PlayerGui")
	if playerGui:FindFirstChild("TeleportDamageGui") then playerGui.TeleportDamageGui:Destroy() end
	local screenGui=Instance.new("ScreenGui")
	screenGui.Name="TeleportDamageGui"
	screenGui.ResetOnSpawn=false
	screenGui.Parent=playerGui

	local frame=Instance.new("Frame")
	frame.Size=UDim2.new(0,260,0,180)
	frame.Position=UDim2.new(0,12,0,12)
	frame.BackgroundColor3=Color3.fromRGB(25,25,25)
	frame.BorderSizePixel=0
	frame.Active=true
	frame.Draggable=true
	frame.Parent=screenGui

	local title=Instance.new("TextLabel")
	title.Size=UDim2.new(0,240,0,28)
	title.Position=UDim2.new(0,10,0,6)
	title.BackgroundTransparency=1
	title.Text="Painel de Habilidades"
	title.Font=Enum.Font.SourceSansBold
	title.TextColor3=Color3.new(1,1,1)
	title.TextSize=18
	title.TextXAlignment=Enum.TextXAlignment.Left
	title.Parent=frame

	-- Teleporte Label e Botões
	local tpLabel=Instance.new("TextLabel")
	tpLabel.Size=UDim2.new(0,100,0,20)
	tpLabel.Position=UDim2.new(0,10,0,40)
	tpLabel.BackgroundTransparency=1
	tpLabel.Text="Teleporte:"
	tpLabel.TextColor3=Color3.new(1,1,1)
	tpLabel.Font=Enum.Font.SourceSans
	tpLabel.TextSize=16
	tpLabel.TextXAlignment=Enum.TextXAlignment.Left
	tpLabel.Parent=frame

	local tpStatus=Instance.new("TextLabel")
	tpStatus.Size=UDim2.new(0,20,0,20)
	tpStatus.Position=UDim2.new(0,110,0,40)
	tpStatus.BackgroundTransparency=0
	tpStatus.Text=""
	tpStatus.Parent=frame

	local tpToggle=Instance.new("TextButton")
	tpToggle.Size=UDim2.new(0,80,0,20)
	tpToggle.Position=UDim2.new(0,140,0,40)
	tpToggle.Text=teleportEnabled and "Desativar" or "Ativar"
	tpToggle.Font=Enum.Font.SourceSans
	tpToggle.TextSize=14
	tpToggle.Parent=frame

	local tpNowBtn=Instance.new("TextButton")
	tpNowBtn.Size=UDim2.new(0,80,0,20)
	tpNowBtn.Position=UDim2.new(0,140,0,65)
	tpNowBtn.Text="Teleportar agora"
	tpNowBtn.Font=Enum.Font.SourceSans
	tpNowBtn.TextSize=14
	tpNowBtn.Parent=frame

	local tpChooseBtn=Instance.new("TextButton")
	tpChooseBtn.Size=UDim2.new(0,100,0,20)
	tpChooseBtn.Position=UDim2.new(0,10,0,65)
	tpChooseBtn.Text="Escolher botão"
	tpChooseBtn.Font=Enum.Font.SourceSans
	tpChooseBtn.TextSize=14
	tpChooseBtn.Parent=frame

	local tpCurrentLabel=Instance.new("TextLabel")
	tpCurrentLabel.Size=UDim2.new(0,240,0,20)
	tpCurrentLabel.Position=UDim2.new(0,10,0,90)
	tpCurrentLabel.BackgroundTransparency=1
	tpCurrentLabel.Text="Botão atual: "..triggerToString(teleportTrigger)
	tpCurrentLabel.TextColor3=Color3.fromRGB(255,255,255)
	tpCurrentLabel.Font=Enum.Font.SourceSans
	tpCurrentLabel.TextSize=14
	tpCurrentLabel.TextXAlignment=Enum.TextXAlignment.Left
	tpCurrentLabel.Parent=frame

	-- Multiplicador de Dano
	local dmgLabel=Instance.new("TextLabel")
	dmgLabel.Size=UDim2.new(0,140,0,20)
	dmgLabel.Position=UDim2.new(0,10,0,120)
	dmgLabel.BackgroundTransparency=1
	dmgLabel.Text="Multiplicador de dano:"
	dmgLabel.TextColor3=Color3.new(1,1,1)
	dmgLabel.Font=Enum.Font.SourceSans
	dmgLabel.TextSize=16
	dmgLabel.TextXAlignment=Enum.TextXAlignment.Left
	dmgLabel.Parent=frame

	local dmgBox=Instance.new("TextBox")
	dmgBox.Size=UDim2.new(0,50,0,20)
	dmgBox.Position=UDim2.new(0,160,0,120)
	dmgBox.Text=tostring(damageMultiplier)
	dmgBox.ClearTextOnFocus=false
	dmgBox.Font=Enum.Font.SourceSans
	dmgBox.TextSize=14
	dmgBox.TextColor3=Color3.fromRGB(0,0,0)
	dmgBox.BackgroundColor3=Color3.fromRGB(200,200,200)
	dmgBox.Parent=frame

	local dmgToggle=Instance.new("TextButton")
	dmgToggle.Size=UDim2.new(0,80,0,20)
	dmgToggle.Position=UDim2.new(0,210,0,120)
	dmgToggle.Text=damageEnabled and "Desativar" or "Ativar"
	dmgToggle.Font=Enum.Font.SourceSans
	dmgToggle.TextSize=14
	dmgToggle.Parent=frame

	local function refreshUI()
		tpToggle.Text = teleportEnabled and "Desativar" or "Ativar"
		tpStatus.BackgroundColor3 = teleportEnabled and Color3.fromRGB(0,200,0) or Color3.fromRGB(200,0,0)
		tpCurrentLabel.Text="Botão atual: "..triggerToString(teleportTrigger)
		dmgToggle.Text = damageEnabled and "Desativar" or "Ativar"
	end

	-- Handlers
	tpToggle.MouseButton1Click:Connect(function() teleportEnabled = not teleportEnabled; refreshUI() end)
	tpNowBtn.MouseButton1Click:Connect(doTeleport)

	tpChooseBtn.MouseButton1Click:Connect(function()
		if listeningForTrigger then return end
		listeningForTrigger=true
		tpChooseBtn.Text="Pressione botão..."
		local conn
		conn=UserInputService.InputBegan:Connect(function(input, gp)
			if gp then return end
			if input.UserInputType==Enum.UserInputType.Keyboard then
				if input.KeyCode~=PANEL_TOGGLE_KEY then teleportTrigger={type="Key", key=input.KeyCode} end
			else
				local bt=input.UserInputType
				if bt==Enum
				if bt==Enum.UserInputType.MouseButton1
					or bt==Enum.UserInputType.MouseButton2
					or bt==Enum.UserInputType.MouseButton3
					or bt==Enum.UserInputType.MouseButton4
					or bt==Enum.UserInputType.MouseButton5 then
					teleportTrigger={type="Mouse", button=bt}
				end
			end
			listeningForTrigger=false
			tpChooseBtn.Text="Escolher botão"
			conn:Disconnect()
			refreshUI()
		end)
	end)

	dmgToggle.MouseButton1Click:Connect(function()
		damageEnabled = not damageEnabled
		refreshUI()
	end)

	dmgBox.FocusLost:Connect(function(enterPressed)
		local val = tonumber(dmgBox.Text)
		if val and val>0 then
			damageMultiplier = val
		else
			dmgBox.Text = tostring(damageMultiplier)
		end
	end)

	screenGui.Enabled=false
	return screenGui, refreshUI
end

local screenGui, refreshUI = createGui()

-- INPUT GLOBAL
UserInputService.InputBegan:Connect(function(input, gp)
	-- Abrir/fechar painel
	if input.UserInputType==Enum.UserInputType.Keyboard and input.KeyCode==PANEL_TOGGLE_KEY and not listeningForTrigger then
		screenGui.Enabled = not screenGui.Enabled
	end

	-- Teleporte
	if teleportEnabled and isTriggerInput(input) and not gp and not listeningForTrigger then
		doTeleport()
	end
end)

-- ============================
-- Multiplicador de Dano: exemplo de integração
-- ============================
-- Aqui vamos interceptar dano feito pelo jogador
-- Exemplo básico: se você tem RemoteEvent para dano, multiplica
-- Caso seu jogo use Humanoid:Health, você pode adaptar
player.CharacterAdded:Connect(function(char)
	local humanoid = char:WaitForChild("Humanoid")
	-- Função para multiplicar dano recebido
	humanoid.TakingDamage:Connect(function(damage)
		if damageEnabled then
			return damage * damageMultiplier
		end
	end)
end)

print("Painel Teleporte + Dano carregado. Pressione "..PANEL_TOGGLE_KEY.Name.." para abrir o painel.")
