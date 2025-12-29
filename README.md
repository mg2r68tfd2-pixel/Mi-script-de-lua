# Mi-script-de-lua
Script para roblox
-- 😈 BALAS MAGNÉTICAS PvP (JUGADORES REALES)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local RS = game:GetService("ReplicatedStorage")

local REMOTE = RS:WaitForChild("FireBullet")

-- CONFIG (AJUSTA QUÉ TAN ROTO)
local MAGNET_RADIUS = 35        -- rango del imán
local CURVE_STRENGTH = 0.25     -- 0.1 suave / 0.3 criminal
local TARGET_HEAD = true        -- true = prioridad cabeza

local function getClosestEnemy(pos, shooterChar)
	local closest, dist
	for _, plr in pairs(Players:GetPlayers()) do
		if plr.Character and plr.Character ~= shooterChar then
			local hum = plr.Character:FindFirstChild("Humanoid")
			local partName = TARGET_HEAD and "Head" or "HumanoidRootPart"
			local part = plr.Character:FindFirstChild(partName)

			if hum and hum.Health > 0 and part then
				local d = (part.Position - pos).Magnitude
				if d <= MAGNET_RADIUS and (not dist or d < dist) then
					dist = d
					closest = part
				end
			end
		end
	end
	return closest
end

REMOTE.OnServerEvent:Connect(function(player, bullet)
	if not bullet or not bullet:IsA("BasePart") then return end
	local char = player.Character
	if not char then return end

	local conn
	conn = RunService.Heartbeat:Connect(function()
		if not bullet or not bullet.Parent then
			conn:Disconnect()
			return
		end

		local target = getClosestEnemy(bullet.Position, char)
		if target then
			local dir = (target.Position - bullet.Position).Unit
			bullet.Velocity = bullet.Velocity:Lerp(
				dir * bullet.Velocity.Magnitude,
				CURVE_STRENGTH
			)
		end
	end)
end)
