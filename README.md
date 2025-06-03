local espEnabled = false
local teamCheck = true
local tracers = {}

local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera

function createESP(player)
    if player == LocalPlayer then return end
    if player.Character and not player.Character:FindFirstChild("ESP_Box") then
        local box = Instance.new("BoxHandleAdornment")
        box.Name = "ESP_Box"
        box.Size = Vector3.new(4, 6, 2)
        box.Adornee = player.Character:FindFirstChild("HumanoidRootPart")
        box.AlwaysOnTop = true
        box.ZIndex = 10
        box.Color3 = Color3.new(1, 0, 0)
        box.Transparency = 0.5
        box.Parent = player.Character
    end
end

function removeESP(player)
    if player.Character and player.Character:FindFirstChild("ESP_Box") then
        player.Character.ESP_Box:Destroy()
    end
end

function createTracer(player)
    if player == LocalPlayer then return end
    if not tracers[player] then
        local line = Drawing.new("Line")
        line.Color = Color3.new(1, 0, 0)
        line.Thickness = 2
        line.Transparency = 1
        tracers[player] = line
    end
end

function removeTracer(player)
    if tracers[player] then
        tracers[player]:Remove()
        tracers[player] = nil
    end
end

RunService.RenderStepped:Connect(function()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local alreadyHasESP = player.Character:FindFirstChild("ESP_Box")
            local shouldShow = espEnabled and (not teamCheck or player.Team ~= LocalPlayer.Team)

            if shouldShow then
                if not alreadyHasESP then
                    createESP(player)
                end
                createTracer(player)

                -- Update tracer
                local tracer = tracers[player]
                if tracer then
                    local rootPos, onScreen = Camera:WorldToViewportPoint(player.Character.HumanoidRootPart.Position)
                    local screenCenter = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
                    tracer.From = screenCenter
                    tracer.To = Vector2.new(rootPos.X, rootPos.Y)
                    tracer.Visible = onScreen
                end
            else
                if alreadyHasESP then
                    removeESP(player)
                end
                removeTracer(player)
            end
        else
            removeTracer(player)
        end
    end
end)

Players.PlayerRemoving:Connect(function(player)
    removeESP(player)
    removeTracer(player)
end)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.E then
        espEnabled = not espEnabled
        print("ESP: " .. (espEnabled and "ON" or "OFF"))
    elseif input.KeyCode == Enum.KeyCode.T then
        teamCheck = not teamCheck
        print("TeamCheck: " .. (teamCheck and "ON" or "OFF"))
    end
end)
