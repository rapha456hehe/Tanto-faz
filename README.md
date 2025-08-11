-- HUB Hacker Neon Completo - Admin Painel + Habilidades + AntiBan
-- Coloque em StarterPlayerScripts (LocalScript)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local VirtualUser = game:GetService("VirtualUser")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
local Camera = workspace.CurrentCamera

-- Configs padrão
local DEFAULT_WALKSPEED = 16
local DEFAULT_JUMPPOWER = 50

-- Estado
local state = {
    flight = false,
    esp = false,
    noclip = false,
    invisible = false,
    flyingUp = false,
    flyingDown = false,
    targetSpeed = 50,
    targetJump = 100,
    antiBanActive = false,
}

-- AntiBan Module
local AntiBan = {}
AntiBan.__index = AntiBan

function AntiBan.new()
    local self = setmetatable({}, AntiBan)
    self.active = false
    self.humanoid = nil
    self.targetWalkSpeed = DEFAULT_WALKSPEED
    self.targetJumpPower = DEFAULT_JUMPPOWER
    self.lastChange = 0
    self.changeCooldown = 1
    self:init()
    return self
end

function AntiBan:init()
    -- Anti-AFK avançado
    LocalPlayer.Idled:Connect(function()
        VirtualUser:Button2Down(Vector2.new(0,0), Camera.CFrame)
        wait(1)
        VirtualUser:Button2Up(Vector2.new(0,0), Camera.CFrame)
    end)

    LocalPlayer.CharacterAdded:Connect(function(char)
        wait(0.5)
        self.humanoid = char:FindFirstChildOfClass("Humanoid")
        if self.active and self.humanoid then
            self:applySafeValues()
        end
    end)

    if LocalPlayer.Character then
        self.humanoid = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
    end

    RunService.Heartbeat:Connect(function()
        if self.active and self.humanoid then
            local now = tick()
            if now - self.lastChange >= self.changeCooldown then
                self.lastChange = now
                if math.abs(self.humanoid.WalkSpeed - self.targetWalkSpeed) > 0.1 then
                    self:smoothAdjust("WalkSpeed", self.targetWalkSpeed)
                end
                if math.abs(self.humanoid.JumpPower - self.targetJumpPower) > 0.1 then
                    self:smoothAdjust("JumpPower", self.targetJumpPower)
                end
            end
        end
    end)
end

function AntiBan:smoothAdjust(prop, target)
    if not self.humanoid then return end
    local base = target
    coroutine.wrap(function()
        for i=1,3 do
            if not self.humanoid then break end
            local variation = (math.random() * 0.2 - 0.1)
            self.humanoid[prop] = base + variation
            wait(0.3 + math.random()*0.3)
        end
        if self.humanoid then
            self.humanoid[prop] = base
        end
    end)()
end

function AntiBan:activate(walkSpeed, jumpPower)
    self.active = true
    self.targetWalkSpeed = walkSpeed or DEFAULT_WALKSPEED
    self.targetJumpPower = jumpPower or DEFAULT_JUMPPOWER
    if self.humanoid then
        self:applySafeValues()
    end
end

function AntiBan:deactivate()
    self.active = false
    if self.humanoid then
        self.humanoid.WalkSpeed = DEFAULT_WALKSPEED
        self.humanoid.JumpPower = DEFAULT_JUMPPOWER
    end
end

function AntiBan:applySafeValues()
    if not self.humanoid then return end
    self.humanoid.WalkSpeed = self.targetWalkSpeed
    self.humanoid.JumpPower = self.targetJumpPower
end

local antiBan = AntiBan.new()

-- Criar GUI
local function create(className, props)
    local obj = Instance.new(className)
    if props then
        for k,v in pairs(props) do
            obj[k] = v
        end
    end
    return obj
end

local screen = create("ScreenGui", {
    Name = "HackerNeonHub",
    ResetOnSpawn = false,
    Parent = PlayerGui,
})

local openBtn = create("TextButton", {
    Parent = screen,
    Size = UDim2.new(0, 160, 0, 50),
    Position = UDim2.new(0, 20, 0, 20),
    BackgroundColor3 = Color3.fromRGB(0,0,0),
    BorderSizePixel = 0,
    Text = "HUB - ADMIN",
    TextColor3 = Color3.fromRGB(0,255,170),
    Font = Enum.Font.GothamBold,
    TextSize = 20,
    AutoButtonColor = false,
})
create("UICorner", {Parent = openBtn, CornerRadius = UDim.new(0, 8)})
create("UIStroke", {Parent = openBtn, Color = Color3.fromRGB(0,255,170), Thickness = 2})

local mainFrame = create("Frame", {
    Parent = screen,
    Size = UDim2.new(0, 400, 0, 480),
    Position = UDim2.new(0, 20, 0, 80),
    BackgroundColor3 = Color3.fromRGB(10,10,10),
    BorderSizePixel = 0,
    Visible = false,
    ClipsDescendants = true,
})
create("UICorner", {Parent = mainFrame, CornerRadius = UDim.new(0, 12)})
create("UIStroke", {Parent = mainFrame, Color = Color3.fromRGB(0,255,170), Thickness = 2, Transparency = 0.6})

local title = create("TextLabel", {
    Parent = mainFrame,
    Size = UDim2.new(1, 0, 0, 50),
    BackgroundTransparency = 1,
    Text = "Hacker Neon Hub",
    TextColor3 = Color3.fromRGB(0,255,170),
    Font = Enum.Font.GothamBold,
    TextSize = 28,
})
local titleGradient = Instance.new("UIGradient", title)
titleGradient.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.fromRGB(0,255,170)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(0,100,255)),
}

local scroll = create("ScrollingFrame", {
    Parent = mainFrame,
    Position = UDim2.new(0, 12, 0, 62),
    Size = UDim2.new(1, -24, 1, -74),
    BackgroundTransparency = 1,
    ScrollBarThickness = 8,
})
local layout = Instance.new("UIListLayout", scroll)
layout.Padding = UDim.new(0, 10)
layout.SortOrder = Enum.SortOrder.LayoutOrder
local padding = Instance.new("UIPadding", scroll)
padding.PaddingTop = UDim.new(0, 6)
padding.PaddingLeft = UDim.new(0, 6)
padding.PaddingRight = UDim.new(0, 6)

-- Função para criar botões neon
local function makeNeonButton(text)
    local btn = create("TextButton", {
        Size = UDim2.new(1, 0, 0, 48),
        BackgroundColor3 = Color3.fromRGB(15, 15, 15),
        BorderSizePixel = 0,
        Text = text,
        TextColor3 = Color3.fromRGB(0, 255, 170),
        Font = Enum.Font.GothamBold,
        TextSize = 20,
        AutoButtonColor = false,
        Parent = scroll,
    })
    create("UICorner", {Parent = btn, CornerRadius = UDim.new(0, 8)})
    local stroke = create("UIStroke", {Parent = btn, Color = Color3.fromRGB(0, 255, 170), Thickness = 1.5, Transparency = 0.5})
    btn.MouseEnter:Connect(function() btn.BackgroundColor3 = Color3.fromRGB(25, 25, 25) end)
    btn.MouseLeave:Connect(function() btn.BackgroundColor3 = Color3.fromRGB(15, 15, 15) end)
    return btn
end

-- Função para criar inputs de texto com label
local function makeLabeledInput(labelText, default)
    local container = create("Frame", {Size = UDim2.new(1, 0, 0, 70), BackgroundTransparency = 1, Parent = scroll})
    local label = create("TextLabel", {
        Parent = container,
        Size = UDim2.new(1, 0, 0, 22),
        BackgroundTransparency = 1,
        Text = labelText,
        TextColor3 = Color3.fromRGB(0, 255, 170),
        Font = Enum.Font.GothamSemibold,
        TextSize = 16,
        TextXAlignment = Enum.TextXAlignment.Left,
    })
    local input = create("TextBox", {
        Parent = container,
        Size = UDim2.new(1, 0, 0, 38),
        Position = UDim2.new(0, 0, 0, 28),
        BackgroundColor3 = Color3.fromRGB(20, 20, 20),
        TextColor3 = Color3.fromRGB(0, 255, 170),
        Font = Enum.Font.Gotham,
        TextSize = 20,
        Text = tostring(default),
        ClearTextOnFocus = false,
    })
    create("UICorner", {Parent = input, CornerRadius = UDim.new(0, 8)})
    local stroke = create("UIStroke", {Parent = input, Color = Color3.fromRGB(0, 255, 170), Thickness = 1, Transparency = 0.6})
    return input
end

-- Variáveis para humanoid e personagem
local function getHumanoid()
    local char = LocalPlayer.Character
    if not char then return nil end
    return char:FindFirstChildOfClass("Humanoid")
end

-- ESP avançado
local function toggleESP(on)
    if on then
        for _, plr in pairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("Head") then
                local hl = Instance.new("Highlight")
                hl.Adornee = plr.Character
                hl.FillColor = Color3.fromRGB(255, 0, 0)
                hl.OutlineColor = Color3.fromRGB(255, 0, 0)
                hl.FillTransparency = 0.5
                hl.OutlineTransparency = 0
                hl.Parent = plr.Character
                espHighlights[plr] = hl
            end
        end
        -- Atualiza quando novos jogadores entram
        Players.PlayerAdded:Connect(function(plr)
            if plr ~= LocalPlayer and espHighlights[plr] == nil then
                plr.CharacterAdded:Connect(function(char)
                    wait(0.5)
                    if state.esp then
                        local hl = Instance.new("Highlight")
                        hl.Adornee = char
                        hl.FillColor = Color3.fromRGB(255, 0, 0)
                        hl.OutlineColor = Color3.fromRGB(255, 0, 0)
                        hl.FillTransparency = 0.5
                        hl.OutlineTransparency = 0
                        hl.Parent = char
                        espHighlights[plr] = hl
                    end
                end)
            end
        end)
    else
        for plr, hl in pairs(espHighlights) do
            if hl and hl.Parent then
                hl:Destroy()
            end
            espHighlights[plr] = nil
        end
    end
end

-- Voo avançado corrigido
local flightBodyVelocity = nil
local flightConnection = nil

local function startFlight()
    local humanoid = getHumanoid()
    if not humanoid then return end
    local rootPart = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not rootPart then return end

    flightBodyVelocity = Instance.new("BodyVelocity")
    flightBodyVelocity.MaxForce = Vector3.new(1e5, 1e5, 1e5)
    flightBodyVelocity.Velocity = Vector3.new(0,0,0)
    flightBodyVelocity.Parent = rootPart

    state.flyingUp = false
    state.flyingDown = false

    flightConnection = RunService.Heartbeat:Connect(function()
        local moveVec = Vector3.new(0,0,0)
        local cameraCFrame = Camera.CFrame
        local forwardVector = Vector3.new(cameraCFrame.LookVector.X, 0, cameraCFrame.LookVector.Z).Unit
        local humanoid = getHumanoid()
        if not humanoid then return end

        -- Movimentação básica com teclado (WASD)
        local moveDir = Vector3.new(0,0,0)
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveDir = moveDir + forwardVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveDir = moveDir - forwardVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveDir = moveDir - Camera.CFrame.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveDir = moveDir + Camera.CFrame.RightVector end

        moveDir = Vector3.new(moveDir.X, 0, moveDir.Z)
        if moveDir.Magnitude > 0 then
            moveDir = moveDir.Unit * (state.targetSpeed or 50)
        end

        if state.flyingUp then
            moveVec = moveDir + Vector3.new(0, state.targetSpeed or 50, 0)
        elseif state.flyingDown then
            moveVec = moveDir + Vector3.new(0, -state.targetSpeed or 50, 0)
        else
            moveVec = moveDir
        end

        flightBodyVelocity.Velocity = moveVec
    end)
end

local function stopFlight()
    if flightConnection then
        flightConnection:Disconnect()
        flightConnection = nil
    end
    if flightBodyVelocity then
        flightBodyVelocity:Destroy()
        flightBodyVelocity = nil
    end
end

-- Noclip toggle
local noclipConnection = nil
local function toggleNoclip(stateNoclip)
    if stateNoclip then
        noclipConnection = RunService.Stepped:Connect(function()
            local char = LocalPlayer.Character
            if not char then return end
            for _, part in pairs(char:GetChildren()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end)
    else
        if noclipConnection then
            noclipConnection:Disconnect()
            noclipConnection = nil
        end
        local char = LocalPlayer.Character
        if not char then return end
        for _, part in pairs(char:GetChildren()) do
            if part:IsA("BasePart") then
                part.CanCollide = true
            end
        end
    end
end

-- Invisibilidade toggle
local function toggleInvisible(stateInvisible)
    local char = LocalPlayer.Character
    if not char then return end
    for _, part in pairs(char:GetChildren()) do
        if part:IsA("BasePart") or part:IsA("Decal") then
            if stateInvisible then
                part.Transparency = 1
                if part:IsA("Decal") then
                    part.Transparency = 1
                end
            else
                part.Transparency = 0
                if part:IsA("Decal") then
                    part.Transparency = 0
                end
            end
        end
    end
end

-- Teleport ao clicar no chão
local teleportConnection = nil
local function enableTeleport()
    teleportConnection = UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if gameProcessed then return end
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            local mousePos = UserInputService:GetMouseLocation()
            local ray = Camera:ScreenPointToRay(mousePos.X, mousePos.Y)
            local raycastParams = RaycastParams.new()
            raycastParams.FilterDescendantsInstances = {LocalPlayer.Character}
            raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
            local raycastResult = workspace:Raycast(ray.Origin, ray.Direction * 500, raycastParams)
            if raycastResult and raycastResult.Position then
                local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                if hrp then
                    hrp.CFrame = CFrame.new(raycastResult.Position + Vector3.new(0, 5, 0))
                end
            end
        end
    end)
end
local function disableTeleport()
    if teleportConnection then
        teleportConnection:Disconnect()
        teleportConnection = nil
    end
end

-- Criar botões e inputs do painel
local buttons = {}

local function addToggleButton(name, stateKey, onToggle)
    local btn = makeNeonButton(name .. " [OFF]")
    btn.MouseButton1Click:Connect(function()
        state[stateKey] = not state[stateKey]
        btn.Text = name .. (state[stateKey] and " [ON]" or " [OFF]")
        onToggle(state[stateKey])
    end)
    return btn
end

local function addSliderInput(labelText, defaultValue, onChanged)
    local input = makeLabeledInput(labelText, defaultValue)
    input.FocusLost:Connect(function(enterPressed)
        if not enterPressed then return end
        local val = tonumber(input.Text)
        if val then
            val = math.clamp
