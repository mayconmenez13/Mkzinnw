-- Cria UI com botão
local player = game.Players.LocalPlayer
local screenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
screenGui.Name = "AutoFarmGui"

local button = Instance.new("TextButton", screenGui)
button.Size = UDim2.new(0, 200, 0, 50)
button.Position = UDim2.new(0.5, -100, 0, 50)
button.AnchorPoint = Vector2.new(0.5, 0)
button.Text = "Toggle Auto Farm"
button.BackgroundColor3 = Color3.fromRGB(30, 180, 60)
button.TextColor3 = Color3.fromRGB(255, 255, 255)
button.TextScaled = true

-- Lógica do Auto Farm
local runService = game:GetService("RunService")
local npcFolder = workspace:WaitForChild("NPCs")
local character = player.Character or player.CharacterAdded:Wait()

local autoFarmEnabled = false
local farmConnection = nil

local function getClosestEnemy()
	local closest = nil
	local minDist = math.huge

	for _, npc in pairs(npcFolder:GetChildren()) do
		if npc:FindFirstChild("Humanoid") and npc.Humanoid.Health > 0 and npc.PrimaryPart then
			local dist = (npc.PrimaryPart.Position - character.PrimaryPart.Position).Magnitude
			if dist < minDist then
				closest = npc
				minDist = dist
			end
		end
	end

	return closest
end

local function toggleAutoFarm()
	autoFarmEnabled = not autoFarmEnabled

	if autoFarmEnabled then
		button.Text = "Auto Farm: ON"
		farmConnection = runService.RenderStepped:Connect(function()
			local target = getClosestEnemy()
			if target then
				-- Gira o personagem pra mirar
				character:PivotTo(CFrame.new(character.PrimaryPart.Position, target.PrimaryPart.Position))

				if (character.PrimaryPart.Position - target.PrimaryPart.Position).Magnitude < 5 then
					local humanoid = target:FindFirstChild("Humanoid")
					if humanoid then
						humanoid:TakeDamage(5)
					end
				end
			end
		end)
	else
		button.Text = "Toggle Auto Farm"
		if farmConnection then
			farmConnection:Disconnect()
			farmConnection = nil
		end
	end
end

-- Conecta o botão
button.MouseButton1Click:Connect(toggleAutoFarm)
