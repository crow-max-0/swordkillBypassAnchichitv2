local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer

local deadCount = 0
local stopped = false
local tracked = {}

-- 칼 장착 스크립트
local function runSwordScript()
	local client = Players.LocalPlayer

	local function dbug(txt)
		print(`[LocalX]: {txt}`)
	end

	dbug('칼 찾는중...')
	local gotsword = false
	local sword = nil

	local findsword = workspace.ChildAdded:Connect(function(obj)
		if obj:IsA('Tool') and obj:FindFirstChild('Handle') then
			dbug('칼 얻음!')
			client.Character.Humanoid:EquipTool(obj)
			gotsword = true
			sword = obj
		end
	end)

	repeat task.wait() until gotsword
	findsword:Disconnect()
	print(sword)
end

-- 처음 실행 시 칼 장착
runSwordScript()

-- 사망 감지
local function onCharacterAdded(otherPlayer, character)
	local humanoid = character:WaitForChild("Humanoid", 3)
	if humanoid then
		humanoid.Died:Connect(function()
			deadCount += 1
			print(otherPlayer.Name .. " 사망. 현재: " .. deadCount)
			if deadCount >= 15 and not stopped then
				stopped = true
				print("15명 처치 완료. 자동 공격 중지됨.")
			end
		end)
	end
end

-- 플레이어 감시 설정
local function setupTracker(otherPlayer)
	if tracked[otherPlayer] then return end
	tracked[otherPlayer] = true

	if otherPlayer.Character then
		onCharacterAdded(otherPlayer, otherPlayer.Character)
	end

	otherPlayer.CharacterAdded:Connect(function(char)
		onCharacterAdded(otherPlayer, char)
	end)
end

-- 친구 여부 확인
local function isFriendWith(player1, player2)
	return player1:IsFriendsWith(player2.UserId)
end

-- 공격 함수
local function attack(otherPlayer)
	if stopped then return end
	if otherPlayer == player then return end
	if isFriendWith(player, otherPlayer) then return end

	setupTracker(otherPlayer)

	local character = otherPlayer.Character
	if character then
		local tool = player.Character and player.Character:FindFirstChildOfClass("Tool")
		if tool and tool:FindFirstChild("Handle") then
			tool:Activate()
			for _, part in ipairs(character:GetChildren()) do
				if part:IsA("BasePart") then
					firetouchinterest(tool.Handle, part, 0)
					firetouchinterest(tool.Handle, part, 1)
				end
			end
		end
	end
end

-- 자동 공격 루프
RunService.RenderStepped:Connect(function()
	if stopped then return end
	for _, otherPlayer in ipairs(Players:GetPlayers()) do
		attack(otherPlayer)
	end
end)
