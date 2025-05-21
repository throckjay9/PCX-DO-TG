-- Serviços essenciais
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

-- Variável para controle do GUI
local PCX_GUI = nil
local GUIEnabled = false

-- Função 100% funcional para criar o GUI
local function CreateReliableGUI()
    -- Verificar e remover GUI existente
    local CoreGui = game:GetService("CoreGui")
    if CoreGui:FindFirstChild("PCX_GUI_PRO") then
        CoreGui.PCX_GUI_PRO:Destroy()
    end

    -- Criar a estrutura base
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "PCX_GUI_PRO"
    ScreenGui.Parent = CoreGui
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    ScreenGui.ResetOnSpawn = false
    ScreenGui.IgnoreGuiInset = true

    -- Frame principal (inicialmente invisível)
    local MainFrame = Instance.new("Frame")
    MainFrame.Name = "MainFrame"
    MainFrame.Size = UDim2.new(0, 300, 0, 0) -- Começa com altura zero
    MainFrame.Position = UDim2.new(0.5, -150, 0.3, 0)
    MainFrame.BackgroundColor3 = Color3.fromRGB(25, 30, 40)
    MainFrame.BackgroundTransparency = 0.1
    MainFrame.BorderSizePixel = 0
    MainFrame.ClipsDescendants = true
    MainFrame.Visible = false
    MainFrame.Parent = ScreenGui

    -- Efeito de cantos arredondados
    local UICorner = Instance.new("UICorner")
    UICorner.CornerRadius = UDim.new(0, 8)
    UICorner.Parent = MainFrame

    -- Cabeçalho do menu
    local Header = Instance.new("Frame")
    Header.Size = UDim2.new(1, 0, 0, 40)
    Header.Position = UDim2.new(0, 0, 0, 0)
    Header.BackgroundColor3 = Color3.fromRGB(20, 25, 35)
    Header.Parent = MainFrame

    -- Título do menu
    local Title = Instance.new("TextLabel")
    Title.Text = "PCX DO TG HACKS"
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

    -- Área de conteúdo
    local ContentFrame = Instance.new("Frame")
    ContentFrame.Size = UDim2.new(1, 0, 1, -40)
    ContentFrame.Position = UDim2.new(0, 0, 0, 40)
    ContentFrame.BackgroundTransparency = 1
    ContentFrame.Parent = MainFrame

    -- Função para criar botões de liga/desliga
    local yPosition = 0.05
    local function CreateToggleOption(optionName, configTable, configKey)
        local ToggleFrame = Instance.new("Frame")
        ToggleFrame.Size = UDim2.new(0.9, 0, 0, 30)
        ToggleFrame.Position = UDim2.new(0.05, 0, yPosition, 0)
        ToggleFrame.BackgroundTransparency = 1
        ToggleFrame.Parent = ContentFrame

        local Label = Instance.new("TextLabel")
        Label.Text = optionName
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

        yPosition = yPosition + 0.1
    end

    -- Criar todas as opções do menu
    CreateToggleOption("Aimbot", Settings.Aimbot, "Enabled")
    CreateToggleOption("Mira na Cabeça", Settings.Aimbot, "TargetPart")
    CreateToggleOption("Verificar Time", Settings.Aimbot, "TeamCheck")
    CreateToggleOption("No Recoil", Settings.NoRecoil, "Enabled")
    CreateToggleOption("ESP Box", Settings.ESP, "Enabled")

    -- Definir altura final do menu
    MainFrame.Size = UDim2.new(0, 300, 0, yPosition * 100 + 60)

    return ScreenGui
end

-- Função para alternar o GUI com garantia de funcionamento
function ToggleGUI()
    -- Criar o GUI se não existir
    if not PCX_GUI then
        PCX_GUI = CreateReliableGUI()
    end

    local MainFrame = PCX_GUI:FindFirstChild("MainFrame")
    if not MainFrame then return end

    GUIEnabled = not GUIEnabled

    if GUIEnabled then
        -- Animação de abertura
        MainFrame.Visible = true
        MainFrame.Size = UDim2.new(0, 300, 0, 0)
        
        local openTween = TweenService:Create(
            MainFrame,
            TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
            {Size = UDim2.new(0, 300, 0, 260)}
        )
        openTween:Play()
    else
        -- Animação de fechamento
        local closeTween = TweenService:Create(
            MainFrame,
            TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
            {Size = UDim2.new(0, 300, 0, 0)}
        )
        closeTween:Play()
        
        closeTween.Completed:Connect(function()
            MainFrame.Visible = false
        end)
    end
end

-- Conexão garantida para a tecla F
local FKeyConnection
FKeyConnection = UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.F then
        ToggleGUI()
    end
end)

-- Garantir que o GUI seja destruído corretamente
game:GetService("Players").LocalPlayer.CharacterRemoving:Connect(function()
    if PCX_GUI then
        PCX_GUI:Destroy()
        PCX_GUI = nil
    end
    if FKeyConnection then
        FKeyConnection:Disconnect()
    end
end)

print("✅ GUI 100% funcional carregada! Pressione F para abrir.")
