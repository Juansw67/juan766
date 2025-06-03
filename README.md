local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local ContextActionService = game:GetService("ContextActionService")

local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local Camera = workspace.CurrentCamera

local espEnabled = false
local teamCheck = true
local aimbotEnabled = false

local espObjects = {}

local FOV = 70 -- Campo de visão do aimbot (em graus)
local AimSmoothness = 0.2 -- Quanto mais baixo, mais rápido/mortal a mira

-- ESP (igual script anterior, simplificado aqui)
local function createESP(player)
    if espObjects[player] then return end
    if player == LocalPlayer then return end
    if not player.Character then return end
    local rootPart = player.Character:FindFirstChild("HumanoidRootPart")
    local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
    if not rootPart or not humanoid then return end

    local box = Instance.new("BoxHandleAdornment")
    box.Name = "ESP_Box"
    box.Adornee = rootPart
    box.Size = Vector3.new(4, 6, 2)
    box.AlwaysOnTop = true
    box.ZIndex = 10
    box.Transparency = 0.5
    box.Color3 = (teamCheck and player.Team == LocalPlayer.Team) and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
    box.Parent = player.Character

    espObjects[player] = {
        Box = box,
        Humanoid = humanoid
    }
end

local function removeESP(player)
    if espObjects[player] then
        local esp = espObjects[player]
        if esp.Box and esp.Box.Parent then esp.Box:Destroy() end
        espObjects[player] = nil
    end
end

-- Atualiza ESP
RunService.RenderStepped:Connect(function()
    if not espEnabled then
        for player, _ in pairs(espObjects) do
            removeESP(player)
        end
        return
    end

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChildOfClass("Humanoid") then
            local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
            if humanoid.Health > 0 then
                local isAlly = teamCheck and player.Team == LocalPlayer.Team
                if teamCheck and isAlly or not teamCheck then
                    if not espObjects[player] then
                        createESP(player)
                    end
                    local esp = espObjects[player]
                    if esp then
                        esp.Box.Color3 = isAlly and Color3.fromRGB(0,255,0) or Color3.fromRGB(255,0,0)
                    end
                else
                    removeESP(player)
                end
            else
                removeESP(player)
            end
        else
            removeESP(player)
        end
    end
end)

-- Função para calcular o alvo mais próximo dentro do FOV
local function getClosestTarget()
    local closestPlayer = nil
    local shortestAngle = math.rad(FOV)

    local cameraCFrame = Camera.CFrame
    local cameraPos = cameraCFrame.Position
    local cameraLookVector = cameraCFrame.LookVector

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChildOfClass("Humanoid") then
            local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
            if humanoid.Health > 0 then
                if not (teamCheck and player.Team == LocalPlayer.Team) then
                    local rootPos = player.Character.HumanoidRootPart.Position
                    local direction = (rootPos - cameraPos).Unit
                    local angle = math.acos(cameraLookVector:Dot(direction))

                    if angle < shortestAngle then
                        shortestAngle = angle
                        closestPlayer = player
                    end
                end
            end
        end
    end

    return closestPlayer
end

-- Aimbot ativado apenas enquanto segura o botão esquerdo do mouse
local aiming = false
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        aiming = true
    end
end)

UserInputService.InputEnded:Connect(function(input, gameProcessed)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        aiming = false
    end
end)

-- Mira suave para o alvo
RunService.RenderStepped:Connect(function()
    if aimbotEnabled and aiming then
        local target = getClosestTarget()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            local headPos = target.Character.Head.Position
            local cameraPos = Camera.CFrame.Position
            local direction = (headPos - cameraPos).Unit

            local currentLookVector = Camera.CFrame.LookVector
            local newLookVector = currentLookVector:Lerp(direction, AimSmoothness)

            Camera.CFrame = CFrame.new(cameraPos, cameraPos + newLookVector)
        end
    end
end)

-- Atalhos para ligar/desligar ESP, TeamCheck e Aimbot
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.E then
        espEnabled = not espEnabled
        print("ESP: " .. (espEnabled and "ON" or "OFF"))
    elseif input.KeyCode == Enum.KeyCode.T then
        teamCheck = not teamCheck
        print("TeamCheck: " .. (teamCheck and "ON" or "OFF"))
    elseif input.KeyCode == Enum.KeyCode.Q then
        aimbotEnabled = not aimbotEnabled
        print("Aimbot: " .. (aimbotEnabled and "ON" or "OFF"))
    end
end)
