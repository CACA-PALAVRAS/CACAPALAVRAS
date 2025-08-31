local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
local RunService = game:GetService("RunService")

-- Configurações de pulo
local jumpsEnabled = true
local jumpPower = 50

-- Guarda a ordem dos itens
local itemOrder = {}

-- Função de simular pulo
local function simulateJump(times)
    local char = LocalPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return end
    for i = 1, times do
        root.Velocity = Vector3.new(root.Velocity.X, jumpPower, root.Velocity.Z)
        wait(0.1)
    end
end

-- Conectar ao botão de pular
UserInputService.JumpRequest:Connect(function()
    if jumpsEnabled then simulateJump(3) end
end)

-- Função para salvar ordem do Backpack
local function saveBackpackOrder()
    local backpack = LocalPlayer:FindFirstChild("Backpack")
    if not backpack then return end
    itemOrder = {}
    for _, tool in ipairs(backpack:GetChildren()) do
        table.insert(itemOrder, tool.Name)
    end
end

-- Função para restaurar ordem do Backpack
local function restoreBackpackOrder()
    local backpack = LocalPlayer:FindFirstChild("Backpack")
    if not backpack then return end
    for _, toolName in ipairs(itemOrder) do
        local tool = backpack:FindFirstChild(toolName)
        if tool then
            tool.Parent = nil
            tool.Parent = backpack
        end
    end
end

-- Cria GUI do Hub
local function createHub()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "YXS_HUB_GUI"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = PlayerGui

    -- Bolinha roxa arrastável
    local openButton = Instance.new("TextButton", screenGui)
    openButton.Size = UDim2.new(0,50,0,50)
    openButton.Position = UDim2.new(0.1,0,0.8,0)
    openButton.BackgroundColor3 = Color3.fromRGB(128,0,128)
    openButton.BorderSizePixel = 0
    openButton.Text = "YXS HUB"
    openButton.TextColor3 = Color3.fromRGB(255,255,255)
    openButton.TextScaled = true
    openButton.TextSize = 14
    local corner = Instance.new("UICorner", openButton)
    corner.CornerRadius = UDim.new(1,0)

    -- Arrastar bolinha
    local dragging, dragInput, dragStart, startPos
    openButton.InputBegan:Connect(function(input)
        if input.UserInputType==Enum.UserInputType.Touch or input.UserInputType==Enum.UserInputType.MouseButton1 then
            dragging=true
            dragStart=input.Position
            startPos=openButton.Position
            input.Changed:Connect(function() if input.UserInputState==Enum.UserInputState.End then dragging=false end end)
        end
    end)
    openButton.InputChanged:Connect(function(input)
        if input.UserInputType==Enum.UserInputType.Touch or input.UserInputType==Enum.UserInputType.MouseMovement then
            dragInput=input
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if input==dragInput and dragging then
            local delta = input.Position - dragStart
            openButton.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset+delta.X, startPos.Y.Scale, startPos.Y.Offset+delta.Y)
        end
    end)

    -- "Folha A4" menor
    local frame = Instance.new("Frame", screenGui)
    frame.Size = UDim2.new(0,200,0,250)
    frame.Position = UDim2.new(0.5,-100,0.5,-125)
    frame.BackgroundColor3 = Color3.fromRGB(128,0,128)
    frame.BorderSizePixel = 0
    frame.Visible = false
    frame.ClipsDescendants = true

    local title = Instance.new("TextLabel", frame)
    title.Size = UDim2.new(1,-40,0,40)
    title.Position = UDim2.new(0,20,0,10)
    title.BackgroundTransparency = 1
    title.Text = "TUTYXS HUB"
    title.TextColor3 = Color3.fromRGB(255,255,255)
    title.TextScaled = true

    -- Botão LIGADO/DESLIGADO azul naval
    local toggleButton = Instance.new("TextButton", frame)
    toggleButton.Size = UDim2.new(1,-40,0,40)
    toggleButton.Position = UDim2.new(0,20,0,200)
    toggleButton.BackgroundColor3 = Color3.fromRGB(0,0,128)
    toggleButton.TextColor3 = Color3.fromRGB(255,255,255)
    toggleButton.TextScaled = true
    toggleButton.Text = "LIGADO"
    toggleButton.MouseButton1Click:Connect(function()
        jumpsEnabled = not jumpsEnabled
        toggleButton.Text = jumpsEnabled and "LIGADO" or "DESLIGADO"
    end)

    -- Animação abrir
    local function openFrame()
        frame.Size = UDim2.new(0,200,0,0)
        frame.Visible = true
        local goal = UDim2.new(0,200,0,250)
        local step = 0
        local conn
        conn = RunService.RenderStepped:Connect(function(dt)
            step = step + dt/0.2
            frame.Size = frame.Size:Lerp(goal, step)
            if step>=1 then conn:Disconnect() end
        end)
    end

    -- Animação fechar
    local function closeFrame()
        local goal = UDim2.new(0,200,0,0)
        local step = 0
        local conn
        conn = RunService.RenderStepped:Connect(function(dt)
            step = step + dt/0.2
            frame.Size = frame.Size:Lerp(goal, step)
            if step>=1 then frame.Visible=false; conn:Disconnect() end
        end)
    end

    -- Clique na bolinha abre/fecha folha
    openButton.MouseButton1Click:Connect(function()
        if frame.Visible then
            closeFrame()
        else
            openFrame()
        end
    end)
end

-- Criar Hub se não existir
if not PlayerGui:FindFirstChild("YXS_HUB_GUI") then
    createHub()
end

-- Reaparecer se resetar
LocalPlayer.CharacterAdded:Connect(function()
    wait(1)
    if not PlayerGui:FindFirstChild("YXS_HUB_GUI") then
        createHub()
    end
    -- Restaurar ordem do Backpack após reset
    restoreBackpackOrder()
end)

-- Salvar ordem do Backpack sempre que mudar
LocalPlayer.Backpack.ChildAdded:Connect(saveBackpackOrder)
LocalPlayer.Backpack.ChildRemoved:Connect(saveBackpackOrder)

-- Inicialmente salva a ordem
saveBackpackOrder()        end
    end
end

-- Cria GUI do Hub
local function createHub()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "YXS_HUB_GUI"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = PlayerGui

    -- Bolinha roxa arrastável
    local openButton = Instance.new("TextButton", screenGui)
    openButton.Size = UDim2.new(0,50,0,50)
    openButton.Position = UDim2.new(0.1,0,0.8,0)
    openButton.BackgroundColor3 = Color3.fromRGB(128,0,128)
    openButton.BorderSizePixel = 0
    openButton.Text = "YXS HUB"
    openButton.TextColor3 = Color3.fromRGB(255,255,255)
    openButton.TextScaled = true
    openButton.TextSize = 14
    local corner = Instance.new("UICorner", openButton)
    corner.CornerRadius = UDim.new(1,0)

    -- Arrastar bolinha
    local dragging, dragInput, dragStart, startPos
    openButton.InputBegan:Connect(function(input)
        if input.UserInputType==Enum.UserInputType.Touch or input.UserInputType==Enum.UserInputType.MouseButton1 then
            dragging=true
            dragStart=input.Position
            startPos=openButton.Position
            input.Changed:Connect(function() if input.UserInputState==Enum.UserInputState.End then dragging=false end end)
        end
    end)
    openButton.InputChanged:Connect(function(input)
        if input.UserInputType==Enum.UserInputType.Touch or input.UserInputType==Enum.UserInputType.MouseMovement then
            dragInput=input
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if input==dragInput and dragging then
            local delta = input.Position - dragStart
            openButton.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset+delta.X, startPos.Y.Scale, startPos.Y.Offset+delta.Y)
        end
    end)

    -- "Folha A4" menor
    local frame = Instance.new("Frame", screenGui)
    frame.Size = UDim2.new(0,200,0,250)
    frame.Position = UDim2.new(0.5,-100,0.5,-125)
    frame.BackgroundColor3 = Color3.fromRGB(128,0,128)
    frame.BorderSizePixel = 0
    frame.Visible = false
    frame.ClipsDescendants = true

    local title = Instance.new("TextLabel", frame)
    title.Size = UDim2.new(1,-40,0,40)
    title.Position = UDim2.new(0,20,0,10)
    title.BackgroundTransparency = 1
    title.Text = "TUTYXS HUB"
    title.TextColor3 = Color3.fromRGB(255,255,255)
    title.TextScaled = true

    -- Botão LIGADO/DESLIGADO azul naval
    local toggleButton = Instance.new("TextButton", frame)
    toggleButton.Size = UDim2.new(1,-40,0,40)
    toggleButton.Position = UDim2.new(0,20,0,200)
    toggleButton.BackgroundColor3 = Color3.fromRGB(0,0,128)
    toggleButton.TextColor3 = Color3.fromRGB(255,255,255)
    toggleButton.TextScaled = true
    toggleButton.Text = "LIGADO"
    toggleButton.MouseButton1Click:Connect(function()
        jumpsEnabled = not jumpsEnabled
        toggleButton.Text = jumpsEnabled and "LIGADO" or "DESLIGADO"
    end)

    -- Animação abrir
    local function openFrame()
        frame.Size = UDim2.new(0,200,0,0)
        frame.Visible = true
        local goal = UDim2.new(0,200,0,250)
        local step = 0
        local conn
        conn = RunService.RenderStepped:Connect(function(dt)
            step = step + dt/0.2
            frame.Size = frame.Size:Lerp(goal, step)
            if step>=1 then conn:Disconnect() end
        end)
    end

    -- Animação fechar
    local function closeFrame()
        local goal = UDim2.new(0,200,0,0)
        local step = 0
        local conn
        conn = RunService.RenderStepped:Connect(function(dt)
            step = step + dt/0.2
            frame.Size = frame.Size:Lerp(goal, step)
            if step>=1 then frame.Visible=false; conn:Disconnect() end
        end)
    end

    -- Clique na bolinha abre/fecha folha
    openButton.MouseButton1Click:Connect(function()
        if frame.Visible then
            closeFrame()
        else
            openFrame()
        end
    end)
end

-- Criar Hub se não existir
if not PlayerGui:FindFirstChild("YXS_HUB_GUI") then
    createHub()
end

-- Reaparecer se resetar
LocalPlayer.CharacterAdded:Connect(function()
    wait(1)
    if not PlayerGui:FindFirstChild("YXS_HUB_GUI") then
        createHub()
    end
    -- Restaurar ordem do Backpack após reset
    restoreBackpackOrder()
end)

-- Salvar ordem do Backpack sempre que mudar
LocalPlayer.Backpack.ChildAdded:Connect(saveBackpackOrder)
LocalPlayer.Backpack.ChildRemoved:Connect(saveBackpackOrder)

-- Inicialmente salva a ordem
saveBackpackOrder()
