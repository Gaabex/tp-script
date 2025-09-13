-- StarterPlayerScripts/TeleportAbility.lua
-- Habilidade de teleporte mirando (tecla configurável)

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer
local mouse = player:GetMouse()

-- CONFIGURAÇÕES
local TELEPORT_KEY = Enum.KeyCode.F -- atalho da habilidade
local MAX_DISTANCE = 100            -- distância máxima do teleporte
local COOLDOWN = 3                  -- segundos de recarga

-- estado
local debounce = false

local function getRoot(char)
	return char and (char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("Torso"))
end

local function teleportToMouse()
	if debounce then return end
	debounce = true

	local char = player.Character or player.CharacterAdded:Wait()
	local root = getRoot(char)
	if not root then return end

	-- Pega posição da mira (mouse.Hit é um CFrame)
	local targetPos = mouse.Hit and mouse.Hit.p
	if not targetPos then return end

	-- calcula distância
	local distance = (targetPos - root.Position).Magnitude
	if distance > MAX_DISTANCE then
		warn("Alvo muito distante (" .. math.floor(distance) .. " studs)")
		debounce = false
		return
	end

	-- eleva levemente para não prender no chão
	local finalPos = targetPos + Vector3.new(0, 3, 0)

	-- aplica teleporte
	root.CFrame = CFrame.new(finalPos)

	print(player.Name .. " teleportou para " .. tostring(finalPos))

	-- cooldown
	task.delay(COOLDOWN, function()
		debounce = false
	end)
end

UserInputService.InputBegan:Connect(function(input, isProcessed)
	if isProcessed then return end
	if input.KeyCode == TELEPORT_KEY then
		teleportToMouse()
	end
end)
