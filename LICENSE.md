-- Configurações atualizadas para PCX DO TG
local Settings = {
    Aimbot = {
        Enabled = true,
        Key = Enum.UserInputType.MouseButton2,  -- Botão direito do mouse
        Smoothness = 0.25,
        FOV = 100,
        TargetPart = "Head",
        VisibleCheck = true,
        TeamCheck = true
    },
    ESP = {
        Enabled = true,
        Box = true,
        TeamColor = Color3.fromRGB(0, 255, 0),
        EnemyColor = Color3.fromRGB(255, 50, 50),
        MaxDistance = 500  -- Distância máxima para ESP
    },
    NoRecoil = {
        Enabled = true,
        Power = 0.85  -- Força da redução (0-1)
    }
}

-- Variáveis globais otimizadas
local ESPObjects = {}
local RecoilConnections = {}
local AimbotActive = false

-- Função para verificar visibilidade
local function IsVisible(part)
    if not Settings.Aimbot.VisibleCheck then return true end
    local character = LocalPlayer.Character
    if not character then return false end
    
    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
    if not humanoidRootPart then return false end
    
    local raycastParams = RaycastParams.new()
    raycastParams.FilterDescendantsInstances = {character, part:FindFirstAncestorOfClass("Model")}
    raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
    raycastParams.IgnoreWater = true
    
    local raycastResult = workspace:Raycast(
        humanoidRootPart.Position,
        (part.Position - humanoidRootPart.Position).Unit * 1000,
        raycastParams
    )
    
    return raycastResult and raycastResult.Instance:IsDescendantOf(part.Parent)
end

-- Aimbot otimizado para PCX DO TG
local function UpdateAimbot()
    if not Settings.Aimbot.Enabled or not AimbotActive then return end
    
    local closestPlayer, closestDistance = nil, Settings.Aimbot.FOV
    local localTeam = LocalPlayer.Team
    local cameraPos = Camera.CFrame.Position
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player == LocalPlayer then continue end
        
        local character = player.Character
        if not character then continue end
        
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if not humanoid or humanoid.Health <= 0 then continue end
        
        local targetPart = character:FindFirstChild(Settings.Aimbot.TargetPart)
        if not targetPart then continue end
        
        -- Verificação de time
        if Settings.Aimbot.TeamCheck and player.Team == localTeam then
            continue
        end
        
        -- Verificação de visibilidade
        if not IsVisible(targetPart) then
            continue
        end
        
        -- Cálculo de distância na tela
        local screenPos, onScreen = Camera:WorldToViewportPoint(targetPart.Position)
        if onScreen then
            local mousePos = UserInputService:GetMouseLocation()
            local distance = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
            
            if distance < closestDistance then
                closestPlayer = player
                closestDistance = distance
            end
        end
    end
    
    -- Aplicar aimbot
    if closestPlayer then
        local targetPart = closestPlayer.Character:FindFirstChild(Settings.Aimbot.TargetPart)
        if targetPart then
            local currentCF = Camera.CFrame
            local targetDirection = (targetPart.Position - currentCF.Position).Unit
            local smoothFactor = Settings.Aimbot.Smoothness
            
            local newLookVector = currentCF.LookVector:Lerp(targetDirection, smoothFactor)
            Camera.CFrame = CFrame.new(currentCF.Position, currentCF.Position + newLookVector)
        end
    end
end

-- NoRecoil aprimorado
local function ApplyNoRecoil()
    if not Settings.NoRecoil.Enabled then return end
    
    local character = LocalPlayer.Character
    if not character then return end
    
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    
    -- Suavizar o recuo
    humanoid.CameraOffset = humanoid.CameraOffset * Settings.NoRecoil.Power
end

-- ESP Box otimizado
local function UpdateESP()
    for player, espData in pairs(ESPObjects) do
        if not player or not player.Character then
            RemoveESP(player)
            continue
        end
        
        local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
        if not humanoid or humanoid.Health <= 0 then
            RemoveESP(player)
            continue
        end
        
        local rootPart = player.Character:FindFirstChild("HumanoidRootPart")
        if not rootPart then continue end
        
        -- Atualizar cor com base no time
        if Settings.ESP.TeamCheck then
            espData.Box.Color3 = player.Team == LocalPlayer.Team and Settings.ESP.TeamColor or Settings.ESP.EnemyColor
        end
        
        -- Verificar distância
        local distance = (rootPart.Position - Camera.CFrame.Position).Magnitude
        if distance > Settings.ESP.MaxDistance then
            espData.Box.Visible = false
        else
            espData.Box.Visible = Settings.ESP.Enabled and Settings.ESP.Box
        end
    end
end

-- Criar ESP para um jogador
local function CreateESP(player)
    if ESPObjects[player] or player == LocalPlayer then return end
    
    local character = player.Character
    if not character then return end
    
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    if not rootPart then return end
    
    -- Criar caixa ESP
    local box = Instance.new("BoxHandleAdornment")
    box.Name = "ESP_" .. player.Name
    box.Adornee = rootPart
    box.AlwaysOnTop = true
    box.ZIndex = 10
    box.Size = Vector3.new(3, 5, 1)
    box.Transparency = 0.5
    box.Color3 = Settings.ESP.TeamCheck and player.Team == LocalPlayer.Team and Settings.ESP.TeamColor or Settings.ESP.EnemyColor
    box.Parent = character
    
    ESPObjects[player] = {
        Box = box
    }
end

-- Remover ESP
local function RemoveESP(player)
    if ESPObjects[player] then
        if ESPObjects[player].Box then
            ESPObjects[player].Box:Destroy()
        end
        ESPObjects[player] = nil
    end
end

-- Conexões de controle
local function SetupConnections()
    -- Aimbot
    UserInputService.InputBegan:Connect(function(input)
        if input.UserInputType == Settings.Aimbot.Key then
            AimbotActive = true
        end
    end)
    
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Settings.Aimbot.Key then
            AimbotActive = false
        end
    end)
    
    -- Atualizar jogadores
    Players.PlayerAdded:Connect(function(player)
        player.CharacterAdded:Connect(function(character)
            if Settings.ESP.Enabled then
                CreateESP(player)
            end
        end)
    end)
    
    Players.PlayerRemoving:Connect(function(player)
        RemoveESP(player)
    end)
    
    -- Inicializar ESP para jogadores existentes
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            CreateESP(player)
        end
    end
end

-- Loop principal
RunService.RenderStepped:Connect(function()
    UpdateAimbot()
    ApplyNoRecoil()
    if Settings.ESP.Enabled then
        UpdateESP()
    end
end)

-- Inicializar
SetupConnections()
