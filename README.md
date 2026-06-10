-- Rivals Kill Sound 完整修復版
-- 修復狙擊鏡開鏡時沒聲音
-- 練習場支援
-- 防止重置
-- 單次播放
-- 無延遲

local Players = game:GetService("Players")
local StarterGui = game:GetService("StarterGui")
local SoundService = game:GetService("SoundService")
local Workspace = game:GetService("Workspace")
local Debris = game:GetService("Debris")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer

-- 防止重置
pcall(function()
    StarterGui:SetCore("ResetButtonCallback", false)
end)

local SOUND_ID = "rbxassetid://129859930871900"

-- 音效播放
local function playKillSound()
    local sound = Instance.new("Sound")
    sound.SoundId = SOUND_ID
    sound.Volume = 25
    sound.PlayOnRemove = true
    sound.Parent = SoundService

    -- Rivals 開鏡時有時候會吞 Sound:Play()
    -- PlayOnRemove 最穩
    sound:Destroy()
end

local handled = {}

local function isNearEnemy(enemyRoot)
    local myChar = player.Character
    if not myChar then
        return false
    end

    local myRoot = myChar:FindFirstChild("HumanoidRootPart")
    if not myRoot then
        return false
    end

    -- 狙擊距離提高
    local dist = (myRoot.Position - enemyRoot.Position).Magnitude

    return dist <= 400
end

local function hookHumanoid(humanoid)
    if humanoid:GetAttribute("KillHooked") then
        return
    end

    humanoid:SetAttribute("KillHooked", true)

    humanoid:GetPropertyChangedSignal("Health"):Connect(function()
        if humanoid.Health > 0 then
            return
        end

        if handled[humanoid] then
            return
        end

        handled[humanoid] = true

        local char = humanoid.Parent
        if not char then
            return
        end

        local enemyRoot = char:FindFirstChild("HumanoidRootPart")
        if not enemyRoot then
            return
        end

        if isNearEnemy(enemyRoot) then
            playKillSound()
        end
    end)
end

-- 掃描所有 Humanoid
for _, obj in ipairs(Workspace:GetDescendants()) do
    if obj:IsA("Humanoid") then
        if obj.Parent ~= player.Character then
            hookHumanoid(obj)
        end
    end
end

-- 新 Humanoid
Workspace.DescendantAdded:Connect(function(obj)
    if obj:IsA("Humanoid") then
        if obj.Parent ~= player.Character then
            hookHumanoid(obj)
        end
    end
end)
