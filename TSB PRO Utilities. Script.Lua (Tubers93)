--// TSB PRO UTILITIES

local Players            = game:GetService("Players")
local RunService         = game:GetService("RunService")
local GuiService         = game:GetService("GuiService")
local StarterGui         = game:GetService("StarterGui")
local MarketplaceService = game:GetService("MarketplaceService")

local LocalPlayer = Players.LocalPlayer

--// GAMEPASS SETTINGS

local GAMEPASS_ID   = 1834703848
local GAMEPASS_LINK = "https://www.roblox.com/game-pass/1834703848/TSB-Pro-Utilities"
local OWNER_NAME    = "helloiyourfri3nd"

--// COPY LINK SYSTEM

if LocalPlayer.Name ~= OWNER_NAME then

    if setclipboard then
        setclipboard(GAMEPASS_LINK)
    end

    pcall(function()

        StarterGui:SetCore("SendNotification", {
            Title = "TSB Pro Utilities",
            Text = "Gamepass link copied! Buy it then re-execute script.",
            Duration = 8
        })
    end)

    local success, ownsPass = pcall(function()

        return MarketplaceService:UserOwnsGamePassAsync(
            LocalPlayer.UserId,
            GAMEPASS_ID
        )
    end)

    if not success or not ownsPass then
        return
    end
end

--// SETTINGS

local MOUNTAIN_POS   = Vector3.new(-239.21, 439.51, 435.84)
local MOUNTAIN_POS_2 = Vector3.new(1.24, 652.52, -353.46)

local SPAWN_POSITIONS = {
    Vector3.new(103.52, 441.11, 269.79),
    Vector3.new(-77.03, 441.15, 99.95),
    Vector3.new(214.11, 441.11, -238.51),
    Vector3.new(372.95, 441.15, -72.89),
    Vector3.new(-46.02, 441.15, -115.98),
}

local HP_TO_ESCAPE       = 36
local VOID_HEIGHT        = -100
local MAX_VELOCITY       = 320
local MAX_POSITION_DELTA = 350
local TELEPORT_LOCK_TIME = 1.5

--// VARIABLES

local Character
local Humanoid
local RootPart

local tpLock       = false
local cooldown     = false
local lastPosition = nil

local Connections = {}

--// BLOCKED INSTANCES

local BLOCKED_INSTANCES = {
    BodyVelocity     = true,
    BodyForce        = true,
    BodyPosition     = true,
    BodyThrust       = true,
    RocketPropulsion = true,

    LinearVelocity   = true,
    VectorForce      = true,
    AlignPosition    = true,
    AlignOrientation = true,
}

--// CLEANUP

local function DisconnectAll()

    for _,v in pairs(Connections) do
        if v then
            v:Disconnect()
        end
    end

    table.clear(Connections)
end

--// CHARACTER

local function GetCharacter()

    Character = LocalPlayer.Character
    if not Character then return false end

    Humanoid = Character:FindFirstChildOfClass("Humanoid")
    RootPart = Character:FindFirstChild("HumanoidRootPart")

    return Humanoid and RootPart
end

--// GUI

local Gui = Instance.new("ScreenGui")
Gui.Name = "TSBProUtilities"
Gui.ResetOnSpawn = false
Gui.IgnoreGuiInset = true
Gui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local Button = Instance.new("TextButton")
Button.Size = UDim2.new(0,55,0,55)
Button.Position = UDim2.new(0,20,0.5,0)
Button.BackgroundColor3 = Color3.fromRGB(40,80,200)
Button.BackgroundTransparency = 0.15
Button.Text = "🛡"
Button.TextScaled = true
Button.Font = Enum.Font.GothamBold
Button.TextColor3 = Color3.new(1,1,1)
Button.BorderSizePixel = 0
Button.Parent = Gui

local ButtonCorner = Instance.new("UICorner")
ButtonCorner.CornerRadius = UDim.new(0,12)
ButtonCorner.Parent = Button

local Status = Instance.new("TextLabel")
Status.Size = UDim2.new(0,160,0,28)
Status.Position = UDim2.new(0,20,0.5,-35)
Status.BackgroundTransparency = 0.35
Status.BackgroundColor3 = Color3.new(0,0,0)
Status.Text = "🛡 Protected"
Status.TextScaled = true
Status.Font = Enum.Font.GothamBold
Status.TextColor3 = Color3.fromRGB(0,255,100)
Status.BorderSizePixel = 0
Status.Parent = Gui

local StatusCorner = Instance.new("UICorner")
StatusCorner.CornerRadius = UDim.new(0,8)
StatusCorner.Parent = Status

local function SetStatus(text,color)
    Status.Text = text
    Status.TextColor3 = color
end

--// TELEPORT

local function ForceTeleport(pos,duration)

    if not GetCharacter() then return end

    tpLock = true

    local start = tick()

    while tick() - start < duration do

        RootPart.CFrame = CFrame.new(pos)

        RootPart.AssemblyLinearVelocity = Vector3.zero
        RootPart.AssemblyAngularVelocity = Vector3.zero

        RunService.Heartbeat:Wait()
    end

    lastPosition = RootPart.Position
    tpLock = false
end

local function RandomSpawn()
    return SPAWN_POSITIONS[math.random(1,#SPAWN_POSITIONS)]
end

--// BODY PROTECTION

local function CheckMover(obj)

    if BLOCKED_INSTANCES[obj.ClassName] then

        obj:Destroy()

        SetStatus("🚫 Blocked", Color3.fromRGB(255,80,80))

        task.delay(1.5,function()

            SetStatus("🛡 Protected", Color3.fromRGB(0,255,100))
        end)
    end
end

local function SetupBodyProtection()

    for _,v in ipairs(RootPart:GetChildren()) do
        CheckMover(v)
    end

    table.insert(Connections,
        RootPart.ChildAdded:Connect(CheckMover)
    )
end

--// HP ESCAPE

local function SetupHealthProtection()

    table.insert(Connections,

        Humanoid.HealthChanged:Connect(function(hp)

            if cooldown then return end

            if hp <= HP_TO_ESCAPE then

                cooldown = true

                SetStatus("⚠ Escaping", Color3.fromRGB(255,200,0))

                ForceTeleport(MOUNTAIN_POS, TELEPORT_LOCK_TIME)

                task.delay(2,function()

                    cooldown = false

                    SetStatus("🛡 Protected", Color3.fromRGB(0,255,100))
                end)
            end
        end)
    )
end

--// MAIN LOOP

local function StartProtectionLoop()

    table.insert(Connections,

        RunService.Heartbeat:Connect(function()

            if not GetCharacter() then return end
            if tpLock then return end

            local pos = RootPart.Position
            local vel = RootPart.AssemblyLinearVelocity

            if pos.Y < VOID_HEIGHT then

                SetStatus("⚠ Anti Void", Color3.fromRGB(255,120,120))

                ForceTeleport(RandomSpawn(),1)

                return
            end

            if vel.Magnitude > MAX_VELOCITY then

                SetStatus("⚡ Anti Fling", Color3.fromRGB(255,200,0))

                RootPart.AssemblyLinearVelocity = Vector3.zero
                RootPart.AssemblyAngularVelocity = Vector3.zero

                ForceTeleport(pos,0.5)

                return
            end

            if lastPosition then

                local delta = (pos - lastPosition).Magnitude

                if delta > MAX_POSITION_DELTA then

                    SetStatus("🚨 TP Detected", Color3.fromRGB(255,50,50))

                    RootPart.CFrame = CFrame.new(lastPosition)

                    RootPart.AssemblyLinearVelocity = Vector3.zero
                end
            end

            lastPosition = pos
        end)
    )
end

--// DRAG GUI

local dragging = false
local dragStart
local startPos
local dragged = false

Button.InputBegan:Connect(function(input)

    if input.UserInputType == Enum.UserInputType.Touch
    or input.UserInputType == Enum.UserInputType.MouseButton1 then

        dragging = true
        dragged = false
        dragStart = input.Position
        startPos = Button.Position
    end
end)

Button.InputChanged:Connect(function(input)

    if dragging and (
        input.UserInputType == Enum.UserInputType.Touch
        or input.UserInputType == Enum.UserInputType.MouseMovement
    ) then

        local delta = input.Position - dragStart

        if math.abs(delta.X) > 8 or math.abs(delta.Y) > 8 then
            dragged = true
        end

        Button.Position = UDim2.new(
            startPos.X.Scale,
            startPos.X.Offset + delta.X,
            startPos.Y.Scale,
            startPos.Y.Offset + delta.Y
        )

        Status.Position = UDim2.new(
            Button.Position.X.Scale,
            Button.Position.X.Offset,
            Button.Position.Y.Scale,
            Button.Position.Y.Offset - 35
        )
    end
end)

Button.InputEnded:Connect(function(input)

    if input.UserInputType == Enum.UserInputType.Touch
    or input.UserInputType == Enum.UserInputType.MouseButton1 then

        dragging = false

        if not dragged then

            SetStatus("⛰ Mountain TP", Color3.fromRGB(0,170,255))

            ForceTeleport(MOUNTAIN_POS_2,1.5)

            task.delay(2,function()

                SetStatus("🛡 Protected", Color3.fromRGB(0,255,100))
            end)
        end
    end
end)

--// INIT

local function Init()

    DisconnectAll()

    repeat
        task.wait()
    until GetCharacter()

    lastPosition = RootPart.Position

    SetupBodyProtection()
    SetupHealthProtection()
    StartProtectionLoop()

    SetStatus("🛡 Protected", Color3.fromRGB(0,255,100))
end

LocalPlayer.CharacterAdded:Connect(function()

    task.wait(1)

    Init()
end)

Init()
