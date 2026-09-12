--[[
    N-M HUB
    Architecture:
        Configuration
        State Manager
        Cleanup Manager
        Utility
        Character Manager
        Function Manager
        GUI Manager
        Input Manager
        Initialization

    Designed for:
        PC + Mobile
        Touch + Mouse
        Character Respawn
        Cleanup
        State Management
        Instance Reuse
        Connection Management
]]

--==================================================
-- SERVICES
--==================================================

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local TeleportService = game:GetService("TeleportService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

--==================================================
-- CONFIGURATION
--==================================================

local Config = {

    GUIName = "N-M HUB",

    Theme = {
        Background = Color3.fromRGB(12, 12, 12),
        Secondary = Color3.fromRGB(20, 20, 20),
        Darker = Color3.fromRGB(28, 28, 28),
        Accent = Color3.fromRGB(160, 80, 255),
        Text = Color3.fromRGB(255, 255, 255),
        SubText = Color3.fromRGB(170, 170, 170),
        Enabled = Color3.fromRGB(55, 35, 75),
        Disabled = Color3.fromRGB(30, 30, 30)
    },

    Keybind = Enum.KeyCode.RightShift,

    KeybindEnabled = true,

    MobileEnabled = true,

    PCEnabled = true,

    Notifications = true,

    MainSize = Vector2.new(310, 390),

    MinimizedSize = Vector2.new(64, 64),

    DefaultSpeed = 16,

    DefaultJumpPower = 50,

    DefaultFOV = 70,

    SpeedStep = 5,

    MaxSpeed = 500,

    FlySpeed = 70,

    FlySpeedStep = 10,

    MaxFlySpeed = 500,

    MinFlySpeed = 10
}

--==================================================
-- STATE MANAGER
--==================================================

local State = {

    GUI = {
        Minimized = false,
        Visible = true
    },

    Character = {
        Character = nil,
        Humanoid = nil,
        RootPart = nil
    },

    Functions = {},

    OriginalValues = {},

    Runtime = {}
}

local function GetFunctionState(functionName)
    return State.Functions[functionName] == true
end

local function SetFunctionState(functionName, value)
    State.Functions[functionName] = value == true
end

--==================================================
-- CLEANUP MANAGER
--==================================================

local Cleanup = {}

Cleanup.Connections = {}
Cleanup.Instances = {}
Cleanup.Callbacks = {}

function Cleanup:AddConnection(name, connection)

    if not connection then
        return
    end

    local oldConnection = self.Connections[name]

    if oldConnection then
        oldConnection:Disconnect()
    end

    self.Connections[name] = connection
end

function Cleanup:RemoveConnection(name)

    local connection = self.Connections[name]

    if connection then
        connection:Disconnect()
        self.Connections[name] = nil
    end
end

function Cleanup:AddInstance(name, instance)

    if not instance then
        return
    end

    local oldInstance = self.Instances[name]

    if oldInstance and oldInstance.Parent then
        oldInstance:Destroy()
    end

    self.Instances[name] = instance
end

function Cleanup:RemoveInstance(name)

    local instance = self.Instances[name]

    if instance then

        if instance.Parent then
            instance:Destroy()
        end

        self.Instances[name] = nil
    end
end

function Cleanup:AddCallback(name, callback)

    if typeof(callback) ~= "function" then
        return
    end

    self.Callbacks[name] = callback
end

function Cleanup:RemoveCallback(name)

    self.Callbacks[name] = nil
end

function Cleanup:RunCallback(name)

    local callback = self.Callbacks[name]

    if callback then
        local success, err = pcall(callback)

        if not success then
            warn("[N-M HUB] Cleanup Error:", err)
        end

        self.Callbacks[name] = nil
    end
end

function Cleanup:ClearAll()

    for name, connection in pairs(self.Connections) do

        if connection then
            connection:Disconnect()
        end

        self.Connections[name] = nil
    end

    for name, instance in pairs(self.Instances) do

        if instance and instance.Parent then
            instance:Destroy()
        end

        self.Instances[name] = nil
    end

    for name, callback in pairs(self.Callbacks) do

        if typeof(callback) == "function" then

            local success, err = pcall(callback)

            if not success then
                warn("[N-M HUB] Cleanup Error:", err)
            end

        end

        self.Callbacks[name] = nil
    end
end

--==================================================
-- EXISTING GUI PROTECTION
--==================================================

local ExistingGUI = PlayerGui:FindFirstChild(Config.GUIName)

if ExistingGUI then
    ExistingGUI:Destroy()
end

--==================================================
-- UTILITY
--==================================================

local Utility = {}

function Utility:Create(className, properties)

    local instance = Instance.new(className)

    for property, value in pairs(properties or {}) do

        local success, err = pcall(function()
            instance[property] = value
        end)

        if not success then
            warn("[N-M HUB] Property Error:", property, err)
        end
    end

    return instance
end

function Utility:AddCorner(parent, radius)

    local corner = Utility:Create("UICorner", {
        CornerRadius = UDim.new(0, radius or 8),
        Parent = parent
    })

    return corner
end

function Utility:AddStroke(parent, color, thickness)

    local stroke = Utility:Create("UIStroke", {
        Color = color or Config.Theme.Accent,
        Thickness = thickness or 1,
        ApplyStrokeMode = Enum.ApplyStrokeMode.Border,
        Parent = parent
    })

    return stroke
end

function Utility:SafeFind(parent, childName)

    if not parent then
        return nil
    end

    return parent:FindFirstChild(childName)
end

function Utility:GetCharacter()

    local character = LocalPlayer.Character

    if not character then
        return nil
    end

    return character
end

function Utility:GetHumanoid()

    local character = Utility:GetCharacter()

    if not character then
        return nil
    end

    return character:FindFirstChildOfClass("Humanoid")
end

function Utility:GetRootPart()

    local character = Utility:GetCharacter()

    if not character then
        return nil
    end

    return character:FindFirstChild("HumanoidRootPart")
end

function Utility:Notify(title, message)

    if not Config.Notifications then
        return
    end

    local StarterGui = game:GetService("StarterGui")

    pcall(function()

        StarterGui:SetCore("SendNotification", {
            Title = title,
            Text = message,
            Duration = 3
        })

    end)
end

--==================================================
-- CHARACTER MANAGER
--==================================================

local CharacterManager = {}

function CharacterManager:Update(character)

    State.Character.Character = character

    if not character then

        State.Character.Humanoid = nil
        State.Character.RootPart = nil

        return
    end

    State.Character.Humanoid =
        character:FindFirstChildOfClass("Humanoid")

    State.Character.RootPart =
        character:FindFirstChild("HumanoidRootPart")
end

function CharacterManager:GetCharacter()

    local character = State.Character.Character

    if character and character.Parent then
        return character
    end

    return nil
end

function CharacterManager:GetHumanoid()

    local humanoid = State.Character.Humanoid

    if humanoid and humanoid.Parent then
        return humanoid
    end

    local character = self:GetCharacter()

    if not character then
        return nil
    end

    humanoid = character:FindFirstChildOfClass("Humanoid")

    State.Character.Humanoid = humanoid

    return humanoid
end

function CharacterManager:GetRootPart()

    local rootPart = State.Character.RootPart

    if rootPart and rootPart.Parent then
        return rootPart
    end

    local character = self:GetCharacter()

    if not character then
        return nil
    end

    rootPart = character:FindFirstChild("HumanoidRootPart")

    State.Character.RootPart = rootPart

    return rootPart
end

function CharacterManager:Initialize()

    local currentCharacter = LocalPlayer.Character

    if currentCharacter then
        self:Update(currentCharacter)
    end

    Cleanup:AddConnection(
        "CharacterAdded",
        LocalPlayer.CharacterAdded:Connect(function(character)

            self:Update(character)

            task.defer(function()

                local humanoid =
                    character:FindFirstChildOfClass("Humanoid")

                if humanoid then
                    State.Character.Humanoid = humanoid
                end

                local rootPart =
                    character:FindFirstChild("HumanoidRootPart")

                if rootPart then
                    State.Character.RootPart = rootPart
                end

            end)

        end)
    )
end

--==================================================
-- DRAG SYSTEM
--==================================================

local Drag = {}

function Drag:Enable(frame, handle)

    handle = handle or frame

    local dragging = false

    local dragStart
    local startPosition

    local inputChangedConnection

    local function update(input)

        if not dragging then
            return
        end

        local delta = input.Position - dragStart

        frame.Position = UDim2.new(
            startPosition.X.Scale,
            startPosition.X.Offset + delta.X,
            startPosition.Y.Scale,
            startPosition.Y.Offset + delta.Y
        )
    end

    Cleanup:AddConnection(

        "DragBegan_" .. frame.Name,

        handle.InputBegan:Connect(function(input)

            if input.UserInputType == Enum.UserInputType.MouseButton1
                or input.UserInputType == Enum.UserInputType.Touch then

                dragging = true

                dragStart = input.Position
                startPosition = frame.Position

                Cleanup:AddConnection(

                    "DragInputChanged_" .. frame.Name,

                    input.Changed:Connect(function()

                        if input.UserInputState ==
                            Enum.UserInputState.End then

                            dragging = false

                        end

                    end)
                )

            end

        end)
    )

    Cleanup:AddConnection(

        "DragChanged_" .. frame.Name,

        UserInputService.InputChanged:Connect(function(input)

            if input.UserInputType ==
                Enum.UserInputType.MouseMovement

                or input.UserInputType ==
                Enum.UserInputType.Touch then

                update(input)

            end

        end)
    )
end

--==================================================
-- GUI MANAGER
--==================================================

local GUIManager = {

    ScreenGui = nil,

    MainFrame = nil,

    Header = nil,

    CategoryFrame = nil,

    FunctionScroll = nil,

    MinimizeButton = nil,

    CloseButton = nil,

    MinimizedCircle = nil,

    FunctionWindows = {},

    FunctionButtons = {},

    CategoryButtons = {},

    CurrentCategory = nil
}

--==================================================
-- MAIN GUI CREATION
--==================================================

function GUIManager:CreateScreenGui()

    self.ScreenGui = Utility:Create("ScreenGui", {

        Name = Config.GUIName,

        ResetOnSpawn = false,

        ZIndexBehavior = Enum.ZIndexBehavior.Sibling,

        IgnoreGuiInset = true,

        Parent = PlayerGui
    })
end

function GUIManager:CreateMainFrame()

    self.MainFrame = Utility:Create("Frame", {

        Name = "MainFrame",

        Size = UDim2.fromOffset(
            Config.MainSize.X,
            Config.MainSize.Y
        ),

        Position = UDim2.new(
            0.5,
            -Config.MainSize.X / 2,
            0.5,
            -Config.MainSize.Y / 2
        ),

        BackgroundColor3 =
            Config.Theme.Background,

        BorderSizePixel = 0,

        ClipsDescendants = true,

        Parent = self.ScreenGui
    })

    Utility:AddCorner(self.MainFrame, 12)

    Utility:AddStroke(
        self.MainFrame,
        Config.Theme.Accent,
        1.5
    )
end

function GUIManager:CreateHeader()

    self.Header = Utility:Create("Frame", {

        Name = "Header",

        Size = UDim2.new(1, 0, 0, 45),

        BackgroundColor3 =
            Config.Theme.Secondary,

        BorderSizePixel = 0,

        Parent = self.MainFrame
    })

    Utility:AddCorner(self.Header, 12)

    local title = Utility:Create("TextLabel", {

        Name = "Title",

        Size = UDim2.new(1, -100, 1, 0),

        Position = UDim2.fromOffset(12, 0),

        BackgroundTransparency = 1,

        Text = "N-M HUB",

        TextColor3 = Config.Theme.Text,

        TextSize = 17,

        Font = Enum.Font.GothamBold,

        TextXAlignment = Enum.TextXAlignment.Left,

        Parent = self.Header
    })

    local minimizeButton = Utility:Create("TextButton", {

        Name = "Minimize",

        Size = UDim2.fromOffset(36, 32),

        Position = UDim2.new(1, -76, 0, 6),

        BackgroundColor3 =
            Config.Theme.Darker,

        BorderSizePixel = 0,

        Text = "—",

        TextColor3 = Config.Theme.Text,

        TextSize = 18,

        Font = Enum.Font.GothamBold,

        AutoButtonColor = true,

        Parent = self.Header
    })

    Utility:AddCorner(minimizeButton, 8)

    Utility:AddStroke(
        minimizeButton,
        Config.Theme.Accent,
        1
    )

    self.MinimizeButton = minimizeButton

    local closeButton = Utility:Create("TextButton", {

        Name = "Close",

        Size = UDim2.fromOffset(36, 32),

        Position = UDim2.new(1, -38, 0, 6),

        BackgroundColor3 =
            Config.Theme.Darker,

        BorderSizePixel = 0,

        Text = "×",

        TextColor3 = Config.Theme.Text,

        TextSize = 18,

        Font = Enum.Font.GothamBold,

        AutoButtonColor = true,

        Parent = self.Header
    })

    Utility:AddCorner(closeButton, 8)

    Utility:AddStroke(
        closeButton,
        Config.Theme.Accent,
        1
    )

    self.CloseButton = closeButton

    Drag:Enable(
        self.MainFrame,
        self.Header
    )

    Cleanup:AddConnection(

        "MainMinimize",

        minimizeButton.MouseButton1Click:Connect(function()
            self:Minimize()
        end)
    )

    Cleanup:AddConnection(

        "MainClose",

        closeButton.MouseButton1Click:Connect(function()

            State.GUI.Visible = false

            self.MainFrame.Visible = false

            for _, window in pairs(self.FunctionWindows) do
                window.Visible = false
            end

        end)
    )
end

--==================================================
-- CATEGORY SYSTEM
--==================================================

local Categories = {

    {
        Name = "🏃 Movement",
        Key = "Movement"
    },

    {
        Name = "❤️ Character",
        Key = "Character"
    },

    {
        Name = "🎥 Camera",
        Key = "Camera"
    },

    {
        Name = "🌍 World",
        Key = "World"
    },

    {
        Name = "✨ Visual",
        Key = "Visual"
    },

    {
        Name = "⚙️ Utility",
        Key = "Utility"
    }
}

function GUIManager:CreateCategories()

    self.CategoryFrame = Utility:Create("ScrollingFrame", {

        Name = "Categories",

        Size = UDim2.new(1, -16, 0, 42),

        Position = UDim2.fromOffset(8, 52),

        BackgroundTransparency = 1,

        BorderSizePixel = 0,

        ScrollBarThickness = 0,

        ScrollingDirection =
            Enum.ScrollingDirection.X,

        AutomaticCanvasSize =
            Enum.AutomaticSize.X,

        CanvasSize = UDim2.new(),

        Parent = self.MainFrame
    })

    local layout = Utility:Create("UIListLayout", {

        FillDirection = Enum.FillDirection.Horizontal,

        HorizontalAlignment = Enum.HorizontalAlignment.Left,

        VerticalAlignment = Enum.VerticalAlignment.Center,

        Padding = UDim.new(0, 6),

        Parent = self.CategoryFrame
    })

    for _, category in ipairs(Categories) do

        local button = Utility:Create("TextButton", {

            Name = category.Key,

            Size = UDim2.fromOffset(110, 34),

            BackgroundColor3 =
                Config.Theme.Darker,

            BorderSizePixel = 0,

            Text = category.Name,

            TextColor3 =
                Config.Theme.SubText,

            TextSize = 12,

            Font = Enum.Font.GothamSemibold,

            AutoButtonColor = true,

            Parent = self.CategoryFrame
        })

        Utility:AddCorner(button, 8)

        Utility:AddStroke(
            button,
            Config.Theme.Accent,
            1
        )

        self.CategoryButtons[category.Key] = button

        Cleanup:AddConnection(

            "Category_" .. category.Key,

            button.MouseButton1Click:Connect(function()

                self:SelectCategory(category.Key)

            end)
        )
    }
end

--==================================================
-- FUNCTION SCROLL
--==================================================

function GUIManager:CreateFunctionScroll()

    self.FunctionScroll = Utility:Create("ScrollingFrame", {

        Name = "Functions",

        Size = UDim2.new(1, -16, 1, -108),

        Position = UDim2.fromOffset(8, 100),

        BackgroundTransparency = 1,

        BorderSizePixel = 0,

        ScrollBarThickness = 4,

        ScrollBarImageColor3 =
            Config.Theme.Accent,

        AutomaticCanvasSize =
            Enum.AutomaticSize.Y,

        CanvasSize = UDim2.new(),

        Parent = self.MainFrame
    })

    local layout = Utility:Create("UIListLayout", {

        SortOrder = Enum.SortOrder.LayoutOrder,

        Padding = UDim.new(0, 6),

        Parent = self.FunctionScroll
    })

    Utility:Create("UIPadding", {

        PaddingTop = UDim.new(0, 4),

        PaddingBottom = UDim.new(0, 8),

        Parent = self.FunctionScroll
    })
end

--==================================================
-- FUNCTION MANAGER
--==================================================

local FunctionManager = {

    Registry = {},

    Order = {}
}

function FunctionManager:Register(data)

    assert(
        type(data) == "table",
        "Function data must be a table"
    )

    assert(
        type(data.Name) == "string",
        "Function Name is required"
    )

    assert(
        type(data.Category) == "string",
        "Function Category is required"
    )

    if self.Registry[data.Name] then
        warn("[N-M HUB] Duplicate function:", data.Name)
        return
    end

    self.Registry[data.Name] = {

        Name = data.Name,

        Category = data.Category,

        Description =
            data.Description or "",

        Enable =
            data.Enable or function() end,

        Disable =
            data.Disable or function() end,

        Cleanup =
            data.Cleanup or function() end,

        State = false
    }

    State.Functions[data.Name] = false

    table.insert(
        self.Order,
        data.Name
    )
end

function FunctionManager:Get(name)

    return self.Registry[name]
end

function FunctionManager:Enable(name)

    local data = 
