local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local player = Players.LocalPlayer
local camera = Workspace.CurrentCamera

local tp_height = 3
local cooldown = 0.5
local last_tp = 0

local disappearSoundId = "rbxassetid://87879054508411"
local arriveSoundId = "rbxassetid://76509426234897"

local tool = Instance.new("Tool")
tool.Name = "TP Tool"
tool.RequiresHandle = false
tool.ToolTip = "Toca para teletransportarte"
tool.Parent = player:WaitForChild("Backpack")

local disappearSound = Instance.new("Sound")
disappearSound.Name = "DisappearSound"
disappearSound.SoundId = disappearSoundId
disappearSound.Volume = 1
disappearSound.Parent = tool

local arriveSound = Instance.new("Sound")
arriveSound.Name = "ArriveSound"
arriveSound.SoundId = arriveSoundId
arriveSound.Volume = 1
arriveSound.Parent = tool

local originalWalkSpeed, originalJumpPower

local function waitForSoundToEnd(sound, timeout)
	timeout = timeout or 10
	sound:Play()
	
	local startTime = tick()
	while sound.IsPlaying do
		if tick() - startTime > timeout then
			break
		end
		task.wait(0.1)
	end
	sound:Stop()
end

local function setLocalTransparency(char, value)
	for _, part in pairs(char:GetDescendants()) do
		if part:IsA("BasePart") then
			part.LocalTransparencyModifier = value
		elseif part:IsA("Decal") or part:IsA("Texture") then
			-- Algunos decals/textures no tienen LocalTransparencyModifier, así que usamos Transparency normal aquí
			part.Transparency = value
		end
	end
end

local function startInvisibilityLocal(char)
	local hum = char:FindFirstChild("Humanoid")
	if hum then
		originalWalkSpeed = hum.WalkSpeed
		originalJumpPower = hum.JumpPower
		hum.WalkSpeed = 0
		hum.JumpPower = 0
	end
	setLocalTransparency(char, 1)
end

local function endInvisibilityLocal(char)
	local hum = char:FindFirstChild("Humanoid")
	if hum then
		hum.WalkSpeed = originalWalkSpeed or 16
		hum.JumpPower = originalJumpPower or 50
	end
	setLocalTransparency(char, 0)
end

local function canTeleport()
	local now = tick()
	if now - last_tp < cooldown then
		return false
	end

	local char = player.Character
	if not char then return false end

	local hum = char:FindFirstChild("Humanoid")
	local root = char:FindFirstChild("HumanoidRootPart")
	if not hum or not root or hum.Health <= 0 or hum.PlatformStand then
		return false
	end

	return true
end

local function teleportTo(pos)
	if not canTeleport() then return end

	local char = player.Character
	local root = char and char:FindFirstChild("HumanoidRootPart")
	local hum = char and char:FindFirstChild("Humanoid")
	if not root or not hum then return end

	last_tp = tick()

	startInvisibilityLocal(char)

	waitForSoundToEnd(disappearSound, 10)

	root.CFrame = CFrame.new(pos + Vector3.new(0, tp_height, 0))

	waitForSoundToEnd(arriveSound, 10)

	endInvisibilityLocal(char)
end

tool.Activated:Connect(function()
	if not UserInputService.TouchEnabled then
		local mouse = player:GetMouse()
		if mouse and mouse.Hit then
			teleportTo(mouse.Hit.Position)
		end
	end
end)

local touchConnection
tool.Equipped:Connect(function()
	if UserInputService.TouchEnabled then
		touchConnection = UserInputService.InputEnded:Connect(function(input, processed)
			if input.UserInputType == Enum.UserInputType.Touch and not processed then
				local screenPos = input.Position
				local ray = camera:ScreenPointToRay(screenPos.X, screenPos.Y)
				local result = Workspace:Raycast(ray.Origin, ray.Direction * 1000)
				if result then
					teleportTo(result.Position)
				else
					teleportTo(ray.Origin + ray.Direction * 20)
				end
			end
		end)
	end
end)

tool.Unequipped:Connect(function()
	if touchConnection then
		touchConnection:Disconnect()
		touchConnection = nil
	end
end)
