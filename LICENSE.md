-- Configuração principal
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

-- Variável para controle do GUI
local PCX_GUI = nil
local GUIEnabled = false

-- Função para criar o GUI corrigido
local function CreateFixedGUI()
    -- Destruir GUI existente
    if game:GetService("CoreGui"):FindFirstChild("PCX_GUI_FIXED") then
        game:GetService("CoreGui").PCX_GUI_FIXED:Destroy()
    end

    -- Criar a interface
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "PCX_GUI_FIXED"
    ScreenGui.Parent = game:GetService("CoreGui")
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    ScreenGui.ResetOnSpawn = false

    -- Frame principal
    local MainFrame = Instance.new("Frame")
    MainFrame.Name = "MainFrame"
    MainFrame.Size = UDim2.new(0, 300, 0, 0) -- Começa com altura zero
    MainFrame.Position = UDim2.new(0.5, -150, 0.3, 0)
    MainFrame.BackgroundColor3 = Color3.fromRGB(30, 35, 45)
    MainFrame.BackgroundTransparency = 0.15
    MainFrame.BorderSizePixel = 0
    MainFrame.ClipsDescendants = true
    MainFrame.Visible = false
    MainFrame.Parent = ScreenGui

    -- Cantos arredondados
    local UICorner = Instance.new("UICorner")
    UICorner.CornerRadius = UDim.new(0, 8)
    UICorner.Parent = MainFrame

    -- Cabeçalho
    local Header = Instance.new("Frame")
    Header.Size = UDim2.new(1, 0, 0, 40)
    Header.Position = UDim2.new(0, 0, 0, 0)
    Header.BackgroundColor3 = Color3.fromRGB(20, 25, 35)
    Header.Parent = MainFrame

    local HeaderCorner = Instance.new("UICorner")
    HeaderCorner.CornerRadius = UDim.new(0, 8)
    HeaderCorner.Parent = Header

    -- Título
    local Title = Instance.new("TextLabel")
    Title.Text = "PCX DO TG MENU"
    Title.Size = UDim2.new(1, -40, 1, 0)
    Title.Position = UDim2.new(0, 10, 0, 0)
    Title.Font = Enum.Font.GothamBold
    Title.TextSize = 16
    Title.TextColor3 = Color3.new(1, 1, 1)
    Title.BackgroundTransparency = 1
    Title.TextXAlignment = Enum.TextXAlignment.Left
    Title.Parent = Header

    -- Botão de fechar
    local CloseButton = Instance.new("TextButton")
    CloseButton.Text = "X"
    CloseButton.Size = UDim2.new(0, 30, 0, 30)
    CloseButton.Position = UDim2.new(1, -35, 0.5, -15)
    CloseButton.Font = Enum.Font.GothamBold
    CloseButton.TextSize = 16
    CloseButton.TextColor3 = Color3.new(1, 1, 1)
    CloseButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
    CloseButton.Parent = Header

    local CloseCorner = Instance.new("UICorner")
    CloseCorner.CornerRadius = UDim.new(0, 6)
    CloseCorner.Parent = CloseButton

    -- Conteúdo do menu
    local ContentFrame = Instance.new("Frame")
    ContentFrame.Size = UDim2.new(1, 0, 1, -40)
    ContentFrame.Position = UDim2.new(0, 0, 0, 40)
    ContentFrame.BackgroundTransparency = 1
    ContentFrame.Parent = MainFrame

    -- Função para criar botões de toggle
    local yOffset = 0.05
    local function CreateToggle(text, configTable, configKey)
        local ToggleFrame = Instance.new("Frame")
        ToggleFrame.Size = UDim2.new(0.9, 0, 0, 30)
        ToggleFrame.Position = UDim2.new(0.05, 0, yOffset, 0)
        ToggleFrame.BackgroundTransparency = 1
        ToggleFrame.Parent = ContentFrame

        local Label = Instance.new("TextLabel")
        Label.Text = text
        Label.Size = UDim2.new(0.7, 0, 1, 0)
        Label.TextXAlignment = Enum.TextXAlignment.Left
        Label.Font = Enum.Font.Gotham
        Label.TextSize = 14
        Label.TextColor3 = Color3.new(1, 1, 1)
        Label.BackgroundTransparency = 1
        Label.Parent = ToggleFrame

        local ToggleButton = Instance.new("TextButton")
        ToggleButton.Size = UDim2.new(0.25, 0, 0.8, 0)
        ToggleButton.Position = UDim2.new(0.75, 0, 0.1, 0)
        ToggleButton.Text = configTable[configKey] and "ON" or "OFF"
        ToggleButton.TextColor3 = Color3.new(1, 1, 1)
        ToggleButton.BackgroundColor3 = configTable[configKey] and Color3.fromRGB(0, 180, 0) or Color3.fromRGB(180, 0, 0)
        ToggleButton.Font = Enum.Font.GothamBold
        ToggleButton.TextSize = 14
        ToggleButton.Parent = ToggleFrame

        local Corner = Instance.new("UICorner")
        Corner.CornerRadius = UDim.new(0, 4)
        Corner.Parent = ToggleButton

        ToggleButton.MouseButton1Click:Connect(function()
            configTable[configKey] = not configTable[configKey]
            ToggleButton.Text = configTable[configKey] and "ON" or "OFF"
            ToggleButton.BackgroundColor3 = configTable[configKey] and Color3.fromRGB(0, 180, 0) or Color3.fromRGB(180, 0, 0)
        end)

        yOffset = yOffset + 0.1
    end

    -- Criar toggles para cada função
    CreateToggle("Aimbot", Settings.Aimbot, "Enabled")
    CreateToggle("Mira na Cabeça", Settings.Aimbot, "TargetPart")
    CreateToggle("Verificar Time", Settings.Aimbot, "TeamCheck")
    CreateToggle("No Recoil", Settings.NoRecoil, "Enabled")
    CreateToggle("ESP Box", Settings.ESP, "Enabled")

    -- Definir altura final do menu
    MainFrame.Size = UDim2.new(0, 300, 0, yOffset * 100 + 60)

    -- Configurar eventos
    CloseButton.MouseButton1Click:Connect(function()
        ToggleGUI()
    end)

    return ScreenGui
end

-- Função para alternar o GUI
function ToggleGUI()
    GUIEnabled = not GUIEnabled
    
    if not PCX_GUI then
        PCX_GUI = CreateFixedGUI()
    end

    local MainFrame = PCX_GUI:FindFirstChild("MainFrame")
    if not MainFrame then return end

    if GUIEnabled then
        MainFrame.Visible = true
        MainFrame.Size = UDim2.new(0, 300, 0, 0)
        
        local tween = TweenService:Create(
            MainFrame,
            TweenService.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
            {Size = UDim2.new(0, 300, 0, 260)}
        )
        tween:Play()
    else
        local tween = TweenService:Create(
            MainFrame,
            TweenService.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
            {Size = UDim2.new(0, 300, 0, 0)}
        )
        tween:Play()
        
        tween.Completed:Wait()
        MainFrame.Visible = false
    end
end

-- Configurar tecla F
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if input.KeyCode == Enum.KeyCode.F and not gameProcessed then
        ToggleGUI()
    end
end)

print("GUI corrigida carregada! Pressione F para abrir/fechar o menu.")
