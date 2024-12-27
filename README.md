local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = game.Workspace.CurrentCamera

-- Function to create ESP for a given player
local function createESP(player)
    local character = player.Character
    if character then
        local head = character:FindFirstChild("Head")
        if head then
            -- Create BillboardGui for player info
            local BillboardGui = Instance.new('BillboardGui', head)
            BillboardGui.Adornee = head
            BillboardGui.ExtentsOffset = Vector3.new(0, 3, 0)
            BillboardGui.Size = UDim2.new(0, 200, 0, 50)
            BillboardGui.AlwaysOnTop = true

            local TextLabel = Instance.new('TextLabel', BillboardGui)
            TextLabel.Size = UDim2.new(1, 0, 1, 0)
            TextLabel.BackgroundTransparency = 1
            TextLabel.TextScaled = true
            TextLabel.Font = Enum.Font.SourceSansBold

            -- Create BoxHandleAdornment for player box
            local Box = Instance.new('BoxHandleAdornment', head)
            Box.Adornee = character
            Box.Size = character:GetExtentsSize()
            Box.Transparency = 0.5
            Box.ZIndex = 0
            Box.AlwaysOnTop = true

            -- Update ESP
            RunService.RenderStepped:Connect(function()
                if character and head and player and player.Parent then
                    local distance = (Camera.CFrame.Position - head.Position).Magnitude
                    local health = character:FindFirstChild("Humanoid") and character.Humanoid.Health or 0
                    local maxHealth = character:FindFirstChild("Humanoid") and character.Humanoid.MaxHealth or 0
                    local healthPercent = maxHealth > 0 and (health / maxHealth) * 100 or 0

                    TextLabel.Text = string.format("%s\n%.1f Health\n%.1f Distance", player.Name, healthPercent, distance)
                    if player.Team == LocalPlayer.Team then
                        TextLabel.TextColor3 = Color3.fromRGB(0, 255, 0)  -- Green for teammates
                        Box.Color3 = Color3.fromRGB(0, 255, 0)  -- Green for teammates
                    else
                        TextLabel.TextColor3 = Color3.fromRGB(255, 0, 0)  -- Red for enemies
                        Box.Color3 = Color3.fromRGB(255, 0, 0)  -- Red for enemies
                    end
                    Box.Size = character:GetExtentsSize()
                else
                    BillboardGui:Destroy()
                    Box:Destroy()
                end
            end)
        end
    end
end

-- Function to handle players
local function onPlayerAdded(player)
    if player.Character then
        createESP(player)
    end
    player.CharacterAdded:Connect(function()
        createESP(player)
    end)
end

-- Connect functions
for _, player in pairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        onPlayerAdded(player)
    end
end
Players.PlayerAdded:Connect(onPlayerAdded)
