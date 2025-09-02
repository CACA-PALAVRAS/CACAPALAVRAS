
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- Pulo
local jumpsEnabled = true
local jumpPower = 50
local itemOrder = {}

local function simulateJump(times)
    local char = LocalPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return end
    for i = 1, times do
        root.Velocity = Vector3.new(root.Velocity.X, jumpPower, root.Velocity.Z)
        wait(0.1)
    end
end

UserInputService.JumpRequest:Connect(function()
    if jumpsEnabled then simulateJump(3) end
end)

local function saveBackpackOrder()
    local backpack = LocalPlayer:FindFirstChild("Backpack")
    if not backpack then return end
    itemOrder = {}
    for _, tool in ipairs(backpack:GetChildren()) do
        table.insert(itemOrder, tool.Name)
    end
end

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

-- ESP
local espEnabled = false
local espBoxes = {}

local function addESP(player)
    if player == LocalPlayer then return end
    if not player.Character then return end
    local hrp = player.Character:FindFirstChild("HumanoidRootPart")
    local head = player.Character:FindFirstChild("Head")
    if hrp and head and not espBoxes[player] then
        local box = Instance.new("BoxHandleAdornment")
        box.Adornee = hrp
        box.AlwaysOnTop = true
        box.ZIndex = 10
        box.Size = Vector3.new(4,6,2)
        box.Color3 = Color3.fromRGB(128,0,128)
        box.Transparency = 0.5
        box.Parent = workspace

        local billboard = Instance.new("BillboardGui")
        billboard.Adornee = head
        billboard.Size = UDim2.new(0,200,0,50)
        billboard.StudsOffset = Vector3.new(0,2,0)
        billboard.AlwaysOnTop = true
        local label = Instance.new("TextLabel", billboard)
        label.Size = UDim2.new(1,0,1,0)
        label.BackgroundTransparency = 1
        label.TextColor3 = Color3.fromRGB(128,0,128)
        label.Font = Enum.Font.ArialBold
        label.TextSize = 30
        label.Text = player.DisplayName
        label.TextStrokeTransparency = 0.5
        billboard.Parent = PlayerGui

        espBoxes[player] = {box=box, label=billboard}
    end
end

local function removeESP(player)
    local data = espBoxes[player]
    if data then
        if data.box then data.box:Destroy() end
        if data.label then data.label:Destroy() end
        espBoxes[player] = nil
    end
end

Players.PlayerRemoving:Connect(removeESP)

RunService.RenderStepped:Connect(function()
    if espEnabled then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer then
                if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                    addESP(player)
                else
                    removeESP(player)
                end
            end
        end
    else
        for p,_ in pairs(espBoxes) do removeESP(p) end
    end
end)

-- Anti-Ragdoll
local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local hum = char:WaitForChild("Humanoid")
local hrp = char:WaitForChild("HumanoidRootPart")
local antiRagdollEnabled = false
local lastVel = hrp.Velocity
local impactThreshold = 60
local arConnection

local function removeForces()
    for _, obj in pairs(hrp:GetChildren()) do
        if obj:IsA("BodyVelocity") or obj:IsA("VectorForce") or obj:IsA("BodyThrust") or obj:IsA("BodyForce") then
            obj:Destroy()
        end
    end
end

local function fixMotor6D()
    for _, v in pairs(char:GetDescendants()) do
        if v:IsA("Motor6D") then
            v.Enabled = true
        end
    end
end

local function AntiRagdoll()
    if hum:GetState() == Enum.HumanoidStateType.Physics or hum.PlatformStand then
        hum:ChangeState(Enum.HumanoidStateType.GettingUp)
        hum.PlatformStand = false
    end
    for _, v in pairs(char:GetDescendants()) do
        if v:IsA("BallSocketConstraint") or v:IsA("HingeConstraint") then
            v:Destroy()
        end
    end
    fixMotor6D()
    local velDelta = (hrp.Velocity - lastVel).Magnitude
    if velDelta > impactThreshold then
        hrp.Velocity = Vector3.new(0,0,0)
        hrp.RotVelocity = Vector3.new(0,0,0)
    end
    lastVel = hrp.Velocity
    removeForces()
end

local function startAntiRagdoll()
    if arConnection then arConnection:Disconnect() end
    arConnection = RunService.Heartbeat:Connect(function()
        if antiRagdollEnabled then AntiRagdoll() end
    end)
end

local function stopAntiRagdoll()
    if arConnection then
        arConnection:Disconnect()
        arConnection = nil
    end
end

hrp.ChildAdded:Connect(function(child)
    if (child:IsA("BodyVelocity") or child:IsA("VectorForce") or child:IsA("BodyThrust") or child:IsA("BodyForce")) and antiRagdollEnabled then
        child:Destroy()
    end
end)

-- Hub GUI
local function createHub()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "YXS_HUB_GUI"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = PlayerGui

    local openButton = Instance.new("TextButton", screenGui)
    openButton.Size = UDim2.new(0,50,0,50)
    openButton.Position = UDim2.new(0.1,0,0.8,0)
    openButton.BackgroundColor3 = Color3.fromRGB(128,0,128)
    openButton.BorderSizePixel = 0
    openButton.Text = "YXS HUB"
    openButton.TextColor3 = Color3.fromRGB(255,255,255)
    openButton.TextScaled = true
    local corner = Instance.new("UICorner", openButton)
    corner.CornerRadius = UDim.new(1,0)

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

    local espButton = Instance.new("TextButton", frame)
    espButton.Size = UDim2.new(1,-40,0,40)
    espButton.Position = UDim2.new(0,20,0,150)
    espButton.BackgroundColor3 = Color3.fromRGB(0,0,128)
    espButton.TextColor3 = Color3.fromRGB(255,255,255)
    espButton.TextScaled = true
    espButton.Text = "ESP DESLIGADO"
    espButton.MouseButton1Click:Connect(function()
        espEnabled = not espEnabled
        espButton.Text = espEnabled and "ESP LIGADO" or "ESP DESLIGADO"
    end)

    local jumpButton = Instance.new("TextButton", frame)
    jumpButton.Size = UDim2.new(1,-40,0,40)
    jumpButton.Position = UDim2.new(0,20,0,200)
    jumpButton.BackgroundColor3 = Color3.fromRGB(0,0,128)
    jumpButton.TextColor3 = Color3.fromRGB(255,255,255)
    jumpButton.TextScaled = true
    jumpButton.Text = jumpsEnabled and "PULO LIGADO" or "PULO DESLIGADO"
    jumpButton.MouseButton1Click:Connect(function()
        jumpsEnabled = not jumpsEnabled
        jumpButton.Text = jumpsEnabled and "PULO LIGADO" or "PULO DESLIGADO"
    end)

    local arButton = Instance.new("TextButton", frame)
    arButton.Size = UDim2.new(1,-40,0,40)
    arButton.Position = UDim2.new(0,20,0,100)
    arButton.BackgroundColor3 = Color3.fromRGB(60,60,60)
    arButton.TextColor3 = Color3.fromRGB(255,255,255)
    arButton.TextScaled = true
    arButton.Text = "ANTI-RAGDOLL OFF"
    arButton.MouseButton1Click:Connect(function()
        antiRagdollEnabled = not antiRagdollEnabled
        if antiRagdollEnabled then
            arButton.Text = "ANTI-RAGDOLL ON"
            arButton.BackgroundColor3 = Color3.fromRGB(0,170,0)
            startAntiRagdoll()
        else
            arButton.Text = "ANTI-RAGDOLL OFF"
            arButton.BackgroundColor3 = Color3.fromRGB(170,0,0)
            stopAntiRagdoll()
        end
    end)

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
