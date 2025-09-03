local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- Jump
local jumpsEnabled = true
local jumpPower = 50
local itemOrder = {}
local function simulateJump(times)
    local c = LocalPlayer.Character
    local r = c and c:FindFirstChild("HumanoidRootPart")
    if not r then return end
    for i = 1, times do
        r.Velocity = Vector3.new(r.Velocity.X, jumpPower, r.Velocity.Z)
        wait(0.1)
    end
end
UserInputService.JumpRequest:Connect(function()
    if jumpsEnabled then simulateJump(5) end
end)

-- Backpack order
local function saveBackpackOrder()
    local b = LocalPlayer:FindFirstChild("Backpack")
    if not b then return end
    itemOrder = {}
    for _, t in ipairs(b:GetChildren()) do table.insert(itemOrder, t.Name) end
end
local function restoreBackpackOrder()
    local b = LocalPlayer:FindFirstChild("Backpack")
    if not b then return end
    for _, n in ipairs(itemOrder) do
        local t = b:FindFirstChild(n)
        if t then t.Parent = nil t.Parent = b end
    end
end

-- ESP
local espEnabled = false
local espBoxes = {}
local function addESP(p)
    if p == LocalPlayer then return end
    if not p.Character then return end
    local hr = p.Character:FindFirstChild("HumanoidRootPart")
    local head = p.Character:FindFirstChild("Head")
    if hr and head and not espBoxes[p] then
        local box = Instance.new("BoxHandleAdornment")
        box.Adornee = hr
        box.AlwaysOnTop = true
        box.ZIndex = 10
        box.Size = Vector3.new(4,6,2)
        box.Color3 = Color3.fromRGB(128,0,128)
        box.Transparency = 0.5
        box.Parent = workspace
        local bill = Instance.new("BillboardGui")
        bill.Adornee = head
        bill.Size = UDim2.new(0,200,0,50)
        bill.StudsOffset = Vector3.new(0,2,0)
        bill.AlwaysOnTop = true
        local label = Instance.new("TextLabel",bill)
        label.Size = UDim2.new(1,0,1,0)
        label.BackgroundTransparency = 1
        label.TextColor3 = Color3.fromRGB(128,0,128)
        label.Font = Enum.Font.ArialBold
        label.TextSize = 30
        label.Text = p.DisplayName
        label.TextStrokeTransparency = 0.5
        bill.Parent = PlayerGui
        espBoxes[p] = {box=box,label=bill}
    end
end
local function removeESP(p)
    local d = espBoxes[p]
    if d then
        if d.box then d.box:Destroy() end
        if d.label then d.label:Destroy() end
        espBoxes[p] = nil
    end
end
Players.PlayerRemoving:Connect(removeESP)

-- Anti-Ragdoll
local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local hum = char:WaitForChild("Humanoid")
local hrp = char:WaitForChild("HumanoidRootPart")
local antiRagdollEnabled = false
local lastVel = hrp.Velocity
local impactThreshold = 60
local arConnection
local function removeForces()
    for _, o in pairs(hrp:GetChildren()) do
        if o:IsA("BodyVelocity") or o:IsA("VectorForce") or o:IsA("BodyThrust") or o:IsA("BodyForce") then o:Destroy() end
    end
end
local function fixMotor6D()
    for _, v in pairs(char:GetDescendants()) do
        if v:IsA("Motor6D") then v.Enabled = true end
    end
end
local function AntiRagdoll()
    if hum:GetState()==Enum.HumanoidStateType.Physics or hum.PlatformStand then
        hum:ChangeState(Enum.HumanoidStateType.GettingUp)
        hum.PlatformStand=false
    end
    for _,v in pairs(char:GetDescendants()) do
        if v:IsA("BallSocketConstraint") or v:IsA("HingeConstraint") then v:Destroy() end
    end
    fixMotor6D()
    local velDelta=(hrp.Velocity-lastVel).Magnitude
    if velDelta>impactThreshold then hrp.RotVelocity=Vector3.new(0,0,0) end
    lastVel=hrp.Velocity
    removeForces()
end
local function startAntiRagdoll()
    if arConnection then arConnection:Disconnect() end
    arConnection = RunService.Heartbeat:Connect(function()
        if antiRagdollEnabled then AntiRagdoll() end
    end)
end
local function stopAntiRagdoll()
    if arConnection then arConnection:Disconnect() arConnection=nil end
end
hrp.ChildAdded:Connect(function(c)
    if (c:IsA("BodyVelocity") or c:IsA("VectorForce") or c:IsA("BodyThrust") or c:IsA("BodyForce")) and antiRagdollEnabled then
        c:Destroy()
    end
end)

-- Hub
local function createHub()
    local sg = Instance.new("ScreenGui")
    sg.Name="YXS_HUB_GUI"
    sg.ResetOnSpawn=false
    sg.Parent=PlayerGui

    local ob = Instance.new("TextButton",sg)
    ob.Size=UDim2.new(0,50,0,50)
    ob.Position=UDim2.new(0.1,0,0.8,0)
    ob.BackgroundColor3=Color3.fromRGB(128,0,128)
    ob.BorderSizePixel=0
    ob.Text="YXS HUB"
    ob.TextColor3=Color3.fromRGB(255,255,255)
    ob.TextScaled=true
    local corner = Instance.new("UICorner",ob)
    corner.CornerRadius=UDim.new(1,0)

    local dragging,dragInput,dragStart,startPos
    ob.InputBegan:Connect(function(i)
        if i.UserInputType==Enum.UserInputType.Touch or i.UserInputType==Enum.UserInputType.MouseButton1 then
            dragging=true
            dragStart=i.Position
            startPos=ob.Position
            i.Changed:Connect(function() if i.UserInputState==Enum.UserInputState.End then dragging=false end end)
        end
    end)
    ob.InputChanged:Connect(function(i) if i.UserInputType==Enum.UserInputType.Touch or i.UserInputType==Enum.UserInputType.MouseMovement then dragInput=i end end)
    UserInputService.InputChanged:Connect(function(i) if i==dragInput and dragging then local delta=i.Position-dragStart ob.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+delta.X,startPos.Y.Scale,startPos.Y.Offset+delta.Y) end end)

    local f = Instance.new("Frame",sg)
    f.Size=UDim2.new(0,200,0,250)
    f.Position=UDim2.new(0.5,-100,0.5,-125)
    f.BackgroundColor3=Color3.fromRGB(128,0,128)
    f.BorderSizePixel=0
    f.Visible=false
    f.ClipsDescendants=true

    local title = Instance.new("TextLabel",f)
    title.Size=UDim2.new(1,-40,0,40)
    title.Position=UDim2.new(0,20,0,10)
    title.BackgroundTransparency=1
    title.Text="TUTYXS HUB"
    title.TextColor3=Color3.fromRGB(255,255,255)
    title.TextScaled=true

    local arButton = Instance.new("TextButton",f)
    arButton.Size=UDim2.new(1,-40,0,40)
    arButton.Position=UDim2.new(0,20,0,100)
    arButton.BackgroundColor3=Color3.fromRGB(0,0,128)
    arButton.TextColor3=Color3.fromRGB(255,255,255)
    arButton.TextScaled=true
    arButton.Text="ANTI-RAGDOLL OFF"
    arButton.MouseButton1Click:Connect(function()
        antiRagdollEnabled = not antiRagdollEnabled
        if antiRagdollEnabled then
            arButton.Text="ANTI-RAGDOLL ON"
            arButton.BackgroundColor3=Color3.fromRGB(0,0,128)
            startAntiRagdoll()
        else
            arButton.Text="ANTI-RAGDOLL OFF"
            arButton.BackgroundColor3=Color3.fromRGB(0,0,128)
            stopAntiRagdoll()
        end
    end)

    local espButton = Instance.new("TextButton",f)
    espButton.Size=UDim2.new(1,-40,0,40)
    espButton.Position=UDim2.new(0,20,0,150)
    espButton.BackgroundColor3=Color3.fromRGB(0,0,128)
    espButton.TextColor3=Color3.fromRGB(255,255,255)
    espButton.TextScaled=true
    espButton.Text="ESP DESLIGADO"
    espButton.MouseButton1Click:Connect(function()
        espEnabled = not espEnabled
        espButton.Text = espEnabled and "ESP LIGADO" or "ESP DESLIGADO"
    end)

    local jumpButton = Instance.new("TextButton",f)
    jumpButton.Size=UDim2.new(1,-40,0,40)
    jumpButton.Position=UDim2.new(0,20,0,200)
    jumpButton.BackgroundColor3=Color3.fromRGB(0,0,128)
    jumpButton.TextColor3=Color3.fromRGB(255,255,255)
    jumpButton.TextScaled=true
    jumpButton.Text=jumpsEnabled and "PULO LIGADO" or "PULO DESLIGADO"
    jumpButton.MouseButton1Click:Connect(function()
        jumpsEnabled = not jumpsEnabled
        jumpButton.Text=jumpsEnabled and "PULO LIGADO" or "PULO DESLIGADO"
    end)

    local function openF()
        f.Visible=true
        f.Position=UDim2.new(0.5,-100,0.5,-125)
    end
    local function closeF()
        f.Visible=false
    end

    ob.MouseButton1Click:Connect(function()
        if f.Visible then closeF() else openF() end
    end)
end

if not PlayerGui:FindFirstChild("YXS_HUB_GUI") then createHub() end
LocalPlayer.CharacterAdded:Connect(function()
    wait(1)
    if not PlayerGui:FindFirstChild("YXS_HUB_GUI") then createHub() end
    restoreBackpackOrder()
end)
LocalPlayer.Backpack.ChildAdded:Connect(saveBackpackOrder)
LocalPlayer.Backpack.ChildRemoved:Connect(saveBackpackOrder)
saveBackpackOrder()

RunService.RenderStepped:Connect(function()
    if espEnabled then
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= LocalPlayer then
                if p.Character and p.Character:FindFirstChild("HumanoidRootPart") then addESP(p) else removeESP(p) end
            end
        end
    else
        for p,_ in pairs(espBoxes) do removeESP(p) end
    end
end)
