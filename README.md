-- Configurações do Aimbot
local MAX_DISTANCE = 100          -- Distância máxima para mirar
local AIM_SENSITIVITY = 0.5       -- Sensibilidade da mira (ajuste para suavizar o movimento)

-- Estado do aimbot (ativado/desativado)
local is_aimbot_active = false

-- Função para checar se o alvo é válido
local function is_valid_target(player, target)
    -- Certifica-se de que o alvo é um jogador válido e vivo
    if not target.Character or not target.Character:FindFirstChild("Humanoid") then
        return false
    end

    -- Não mira em jogadores fora do jogo (não vivos)
    if target.Character.Humanoid.Health <= 0 then
        return false
    end

    -- Não mira em jogadores do mesmo time (para jogos com times)
    if player.Team == target.Team then
        return false
    end

    -- Verifique se existe linha de visão direta (usando Raycast)
    local char = player.Character or player.CharacterAdded:Wait()
    local head = char:FindFirstChild("Head")
    local targetHead = target.Character:FindFirstChild("Head")

    if head and targetHead then
        local ray = Ray.new(head.Position, (targetHead.Position - head.Position).unit * MAX_DISTANCE)
        local part, _ = workspace:FindPartOnRay(ray, char)
        if part and part:IsDescendantOf(target.Character) then
            return true
        end
    end

    return false
end

-- Função para pegar o inimigo mais próximo
local function get_nearest_enemy(player)
    local nearest_enemy = nil
    local nearest_distance = MAX_DISTANCE

    for _, target in pairs(game.Players:GetPlayers()) do
        if target ~= player and is_valid_target(player, target) then
            local char = player.Character or player.CharacterAdded:Wait()
            local targetChar = target.Character
            if char and targetChar then
                local distance = (char.PrimaryPart.Position - targetChar.PrimaryPart.Position).Magnitude
                if distance < nearest_distance then
                    nearest_distance = distance
                    nearest_enemy = target
                end
            end
        end
    end

    return nearest_enemy
end

-- Aimbot principal
local function aimbot(player)
    local nearest_enemy = get_nearest_enemy(player)
    if nearest_enemy and nearest_enemy.Character then
        local enemyHead = nearest_enemy.Character:FindFirstChild("Head")
        if enemyHead then
            -- Mira na cabeça do inimigo ajustando suavemente
            local camera = workspace.CurrentCamera
            local targetPosition = enemyHead.Position
            camera.CFrame = camera.CFrame:Lerp(CFrame.new(camera.CFrame.Position, targetPosition), AIM_SENSITIVITY)
        end
    end
end

-- Ativar/Desativar o Aimbot ao clicar no botão direito do mouse
local UIS = game:GetService("UserInputService")
local player = game.Players.LocalPlayer

-- Evento para quando o botão direito do mouse for pressionado
UIS.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.UserInputType == Enum.UserInputType.MouseButton2 then  -- Botão direito do mouse
        is_aimbot_active = true
    end
end)

-- Evento para quando o botão direito do mouse for solto
UIS.InputEnded:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.UserInputType == Enum.UserInputType.MouseButton2 then  -- Botão direito do mouse
        is_aimbot_active = false
    end
end)

-- Loop do Aimbot
game:GetService("RunService").RenderStepped:Connect(function()
    if is_aimbot_active then
        aimbot(player)  -- Só chama o aimbot se o botão direito estiver pressionado
    end
end)
