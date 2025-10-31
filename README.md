local TWEEN_SERVICE = game:GetService("TweenService")
local DEBRIS = game:GetService("Debris")


local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")


local function farmTreasure()
    -- Find nearest treasure chest
    local nearestChest = nil
    local minDistance = math.huge
    
    for _, part in ipairs(workspace.Treasures:GetChildren()) do
        if part.Name == "Chest" then
            local distance = (part.Position - humanoidRootPart.Position).Magnitude
            
            if distance < minDistance then
                nearestChest = part
                minDistance = distance
            end
        end
    end
    
    -- Move to nearest chest
    if nearestChest then
        local pathfinding = require(game:GetService("ReplicatedStorage").Pathfinding)
        local path = pathfinding.new(humanoidRootPart.CFrame.p, nearestChest.CFrame.p)
        
        for _, waypoint in ipairs(path) do
            local goal = Instance.new("ObjectValue")
            goal.Value = waypoint
            goal.Parent = humanoidRootPart
            
            local info = TweenInfo.new(0.5, Enum.EasingStyle.Linear)
            local tween = TWEEN_SERVICE:Create(humanoidRootPart, info, {CFrame = waypoint})
            tween:Play()
            tween.Completed:Wait()
            
            DEBRIS:AddItem(goal, 1)
        end
        
        -- Collect treasure
        local collectRemote = game:GetService("ReplicatedStorage").CollectTreasure
        collectRemote:FireServer(nearestChest)
    end
end

-- Main loop
while true do
    farmTreasure()
    wait(5)
end
