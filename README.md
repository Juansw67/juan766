local espEnabled = false
local teamCheck = true
local aimbotEnabled = false
local tracers = {}

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local UserInputService = game:GetService("UserInputService")
local Drawing = Drawing

local FOV = 110 -- FOV atualizado
local AimSmoothness = 0.15 -- suavização reativada

-- FOV Circle
local FOVCircle = Drawing.new("Circle")
FOVCircle.Color = Color3.new(1, 1, 1)
FOVCircle.Thickness = 2
FOVCircle.NumSides = 64
FOVCircle.Radius = FOV
FOVCircle.Filled = false
FOVCircle.Visible = true

-- Notificação Caixa
local notifBox = Drawing.new("Square")
notifBox.Filled = true
notifBox.Color = Color3.new(0, 0, 0)
notifBox.Transparency = 0.6
notifBox.Size = Vector2.new(350, 50)
notifBox.Position = Vector2.new(10, Camera.ViewportSize.Y - 70)
notifBox.Visible = true

-- Notificação Texto
local notification = Drawing.new("Text")
notification.Position = Vector2.new(15, Camera.ViewportSize.Y - 65)
notification.Size = 25
notification.Color = Color3.new(1, 0, 0)
notification.Outline = true
notification.Text = "ESP: OFF | Aimbot: OFF | TeamCheck: ON"
notification.Visible = true

function isEnemy(player)
    return not teamCheck or (player.Team ~= LocalPlayer.Team)
end

function updateNotification()
    notification.Text = "ESP: " .. (espEnabled and "ON" or "OFF") ..
        " | Aimbot: " .. (aimbotEnabled and "ON" or "OFF") ..
        " | TeamCheck: " .. (teamCheck and "ON" or "OFF")
end

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
    if not tracers[player] then
        local line = Drawing.new("Line")
        line.Color = Color3.new(0, 1, 0)
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

function getClosestTarget()
    local closest = nil
    local shortestDist = math.huge

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and isEnemy(player) then
            local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
            if humanoid and humanoid.Health > 0 then
                local rootPos = player.Character.HumanoidRootPart.Position
                local screenPos, onScreen = Camera:WorldToViewportPoint(rootPos)
                if onScreen then
                    local distFromCenter = (Vector2.new(screenPos.X, screenPos.Y) - Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)).Magnitude
                    if distFromCenter < FOV and distFromCenter < shortestDist then
                        shortestDist = distFromCenter
                        closest = player
                    end
                end
            end
        end
    end
    return closest
end

RunService.RenderStepped:Connect(function()
    -- Atualizar FOV Circle
    FOVCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    FOVCircle.Radius = FOV
    FOVCircle.Visible = true

    -- Atualizar Notificação
    notifBox.Position = Vector2.new(10, Camera.ViewportSize.Y - 70)
    notification.Position = Vector2.new(15, Camera.ViewportSize.Y - 65)
    notifBox.Visible = true
    notification.Visible = true

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and isEnemy(player) then
            if espEnabled then
                if not player.Character:FindFirstChild("ESP_Box") then
                    createESP(player)
                end
                createTracer(player)

                local tracer = tracers[player]
                if tracer then
                    local rootPos, onScreen = Camera:WorldToViewportPoint(player.Character.HumanoidRootPart.Position)
                    local screenCenter = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
                    tracer.From = screenCenter
                    tracer.To = Vector2.new(rootPos.X, rootPos.Y)
                    tracer.Visible = onScreen
                end
            else
                removeESP(player)
                removeTracer(player)
            end
        else
            removeESP(player)
            removeTracer(player)
        end
    end

    -- Aimbot com suavização
    if aimbotEnabled then
        local target = getClosestTarget()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            local headPos = target.Character.Head.Position
            local camPos = Camera.CFrame.Position
            local direction = (headPos - camPos).Unit
            local currentLook = Camera.CFrame.LookVector
            local newLook = currentLook:Lerp(direction, AimSmoothness)
            Camera.CFrame = CFrame.new(camPos, camPos + newLook)
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
        updateNotification()
    elseif input.KeyCode == Enum.KeyCode.T then
        teamCheck = not teamCheck
        updateNotification()
    elseif input.KeyCode == Enum.KeyCode.Q then
        aimbotEnabled = not aimbotEnabled
        updateNotification()
    end
end)
