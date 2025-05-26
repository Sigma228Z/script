local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local HttpService = game:GetService("HttpService")

-- Цветовая схема с эффектом свечения
local BACKGROUND_COLOR = Color3.fromRGB(20, 20, 20)
local ACCENT_COLOR = Color3.fromRGB(0, 255, 127)
local TEXT_COLOR = Color3.fromRGB(220, 220, 220)
local SECONDARY_COLOR = Color3.fromRGB(30, 30, 30)
local BUTTON_COLOR = Color3.fromRGB(40, 40, 40)
local GLOW_COLOR = Color3.fromRGB(0, 255, 127)

-- Настройки игры
local BASE_WALK_SPEED = 16
local MAX_WALK_SPEED = 50
local AUTO_FARM_ENABLED = false
local AUTO_EAT_ENABLED = false
local ESP_ENABLED = false
local SHOW_NAMES_ENABLED = false
local MOUNTAIN_CLIMBER_ENABLED = false
local HITBOXES_ENABLED = false
local HITBOX_SIZE = 2
local HITBOX_TRANSPARENCY = 0.7
local HITBOX_TYPE = "All"
local NIGHT_VISION_ENABLED = false
local WAYPOINTS_ENABLED = false
local WAYPOINTS_MOVING = false
local TOUCH_WAYPOINTS_ENABLED = false -- Для сенсорного ввода
local KILL_AURA_ENABLED = false
local KILL_AURA_RANGE = 15
local KILL_AURA_COOLDOWN = 1
local GLOW_ENABLED = false
local GLOW_COLOR_R = 255
local GLOW_COLOR_G = 255
local GLOW_COLOR_B = 255

-- Переменная для дебounce
local lastWaypointCreationTime = 0
local WAYPOINT_CREATION_COOLDOWN = 0.3 -- Задержка в 0.3 секунды между созданием точек

-- Переменные для сенсорного ввода
local touchStartTime = 0
local touchCount = 0
local LONG_PRESS_DURATION = 0.5 -- Долгое нажатие (для круглой точки)
local DOUBLE_TAP_THRESHOLD = 0.3 -- Порог для двойного касания

-- Состояние анимаций
local ANIMATION_1_ENABLED = false
local ANIMATION_2_ENABLED = false
local ANIMATION_3_ENABLED = false
local ANIMATION_4_ENABLED = false
local ESP_HOLDER = Instance.new("Folder", workspace)
ESP_HOLDER.Name = "ESP_HOLDER"
local WAYPOINTS_HOLDER = Instance.new("Folder", workspace)
WAYPOINTS_HOLDER.Name = "WaypointsHolder"
local HITBOX_HOLDER = Instance.new("Folder", workspace)
HITBOX_HOLDER.Name = "HitboxHolder"

-- Переменные для анимаций
local animationTracks = {}
local currentAnimationTrack = nil

-- Переменные для свечения
local GlowHighlight = nil

-- Создание интерфейса
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "BoogaXUI"
ScreenGui.Parent = game:GetService("CoreGui")
ScreenGui.ResetOnSpawn = false

-- Основной фрейм
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 600, 0, 400)
MainFrame.Position = UDim2.new(0.5, -300, 0.5, -200)
MainFrame.BackgroundColor3 = BACKGROUND_COLOR
MainFrame.BackgroundTransparency = 0
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 12)
UICorner.Parent = MainFrame

local UIStroke = Instance.new("UIStroke")
UIStroke.Color = GLOW_COLOR
UIStroke.Thickness = 2
UIStroke.Parent = MainFrame

-- Эффект свечения
local GlowEffect = Instance.new("Frame")
GlowEffect.Size = UDim2.new(1, 0, 1, 0)
GlowEffect.BackgroundTransparency = 1
GlowEffect.Parent = MainFrame

local Glow = Instance.new("UIGradient")
Glow.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, GLOW_COLOR),
    ColorSequenceKeypoint.new(0.5, GLOW_COLOR),
    ColorSequenceKeypoint.new(1, GLOW_COLOR)
})
Glow.Transparency = NumberSequence.new({
    NumberSequenceKeypoint.new(0, 0.8),
    NumberSequenceKeypoint.new(0.5, 0.3),
    NumberSequenceKeypoint.new(1, 0.8)
})
Glow.Parent = GlowEffect

-- Заголовок
local Header = Instance.new("Frame")
Header.Size = UDim2.new(1, 0, 0, 50)
Header.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Header.BorderSizePixel = 0
Header.Parent = MainFrame

local HeaderCorner = Instance.new("UICorner")
HeaderCorner.CornerRadius = UDim.new(0, 12)
HeaderCorner.Parent = Header

local Title = Instance.new("TextLabel")
Title.Text = "BoogaX - Premium UI"
Title.Font = Enum.Font.GothamBold
Title.TextSize = 22
Title.TextColor3 = TEXT_COLOR
Title.Size = UDim2.new(0, 200, 1, 0)
Title.Position = UDim2.new(0, 15, 0, 0)
Title.BackgroundTransparency = 1
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = Header

local MinimizeBtn = Instance.new("TextButton")
MinimizeBtn.Size = UDim2.new(0, 30, 0, 30)
MinimizeBtn.Position = UDim2.new(1, -90, 0.5, -15)
MinimizeBtn.BackgroundColor3 = SECONDARY_COLOR
MinimizeBtn.Text = "-"
MinimizeBtn.TextColor3 = TEXT_COLOR
MinimizeBtn.Font = Enum.Font.GothamBold
MinimizeBtn.TextSize = 18
MinimizeBtn.Parent = Header

local MinimizeBtnCorner = Instance.new("UICorner")
MinimizeBtnCorner.CornerRadius = UDim.new(0, 8)
MinimizeBtnCorner.Parent = MinimizeBtn

local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 30, 0, 30)
CloseBtn.Position = UDim2.new(1, -45, 0.5, -15)
CloseBtn.BackgroundColor3 = SECONDARY_COLOR
CloseBtn.Text = "X"
CloseBtn.TextColor3 = Color3.fromRGB(255, 80, 80)
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 18
CloseBtn.Parent = Header

local CloseBtnCorner = Instance.new("UICorner")
CloseBtnCorner.CornerRadius = UDim.new(0, 8)
CloseBtnCorner.Parent = CloseBtn

-- Мини-версия
local MinimizedFrame = Instance.new("Frame")
MinimizedFrame.Size = UDim2.new(0, 200, 0, 60)
MinimizedFrame.Position = UDim2.new(0.5, -100, 0.5, -30)
MinimizedFrame.BackgroundColor3 = BACKGROUND_COLOR
MinimizedFrame.BorderSizePixel = 0
MinimizedFrame.Visible = false
MinimizedFrame.Parent = MainFrame

local MinCorner = Instance.new("UICorner")
MinCorner.CornerRadius = UDim.new(0, 12)
MinCorner.Parent = MinimizedFrame

local MinStroke = Instance.new("UIStroke")
MinStroke.Color = GLOW_COLOR
MinStroke.Thickness = 2
MinStroke.Parent = MinimizedFrame

local MinLogo = Instance.new("TextLabel")
MinLogo.Text = "BoogaX"
MinLogo.TextColor3 = TEXT_COLOR
MinLogo.Font = Enum.Font.GothamBold
MinLogo.TextSize = 24
MinLogo.Size = UDim2.new(1, 0, 1, 0)
MinLogo.BackgroundTransparency = 1
MinLogo.TextStrokeTransparency = 0.5
MinLogo.TextStrokeColor3 = GLOW_COLOR
MinLogo.Parent = MinimizedFrame

-- Боковая панель с вкладками
local Sidebar = Instance.new("Frame")
Sidebar.Size = UDim2.new(0, 150, 1, -50)
Sidebar.Position = UDim2.new(0, 0, 0, 50)
Sidebar.BackgroundColor3 = SECONDARY_COLOR
Sidebar.BorderSizePixel = 0
Sidebar.Parent = MainFrame

local SidebarLayout = Instance.new("UIListLayout")
SidebarLayout.Padding = UDim.new(0, 5)
SidebarLayout.FillDirection = Enum.FillDirection.Vertical
SidebarLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
SidebarLayout.SortOrder = Enum.SortOrder.LayoutOrder
SidebarLayout.Parent = Sidebar

-- Основное содержимое
local ContentFrame = Instance.new("Frame")
ContentFrame.Size = UDim2.new(1, -150, 1, -50)
ContentFrame.Position = UDim2.new(0, 150, 0, 50)
ContentFrame.BackgroundTransparency = 1
ContentFrame.Parent = MainFrame

-- Функция для создания вкладок
local function CreateTab(name, icon)
    local tabButton = Instance.new("TextButton")
    tabButton.Size = UDim2.new(0.9, 0, 0, 40)
    tabButton.BackgroundColor3 = SECONDARY_COLOR
    tabButton.Text = icon .. " " .. name
    tabButton.TextColor3 = TEXT_COLOR
    tabButton.Font = Enum.Font.SourceSans
    tabButton.TextSize = 16
    tabButton.Parent = Sidebar
    
    local tabCorner = Instance.new("UICorner")
    tabCorner.CornerRadius = UDim.new(0, 5)
    tabCorner.Parent = tabButton
    
    local tabContent = Instance.new("ScrollingFrame")
    tabContent.Size = UDim2.new(1, -20, 1, -20)
    tabContent.Position = UDim2.new(0, 10, 0, 10)
    tabContent.BackgroundTransparency = 1
    tabContent.Visible = false
    tabContent.CanvasSize = UDim2.new(0, 0, 0, 0)
    tabContent.ScrollBarThickness = 5
    tabContent.ScrollBarImageColor3 = ACCENT_COLOR
    tabContent.Parent = ContentFrame
    
    local contentLayout = Instance.new("UIListLayout")
    contentLayout.Padding = UDim.new(0, 10)
    contentLayout.FillDirection = Enum.FillDirection.Vertical
    contentLayout.SortOrder = Enum.SortOrder.LayoutOrder
    contentLayout.Parent = tabContent
    
    contentLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        tabContent.CanvasSize = UDim2.new(0, 0, 0, contentLayout.AbsoluteContentSize.Y + 20)
    end)
    
    tabButton.MouseButton1Click:Connect(function()
        for _, child in ipairs(ContentFrame:GetChildren()) do
            if child:IsA("ScrollingFrame") then
                child.Visible = false
            end
        end
        tabContent.Visible = true
        for _, btn in ipairs(Sidebar:GetChildren()) do
            if btn:IsA("TextButton") then
                btn.BackgroundColor3 = SECONDARY_COLOR
                btn.TextColor3 = TEXT_COLOR
            end
        end
        tabButton.BackgroundColor3 = ACCENT_COLOR
        tabButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    end)
    
    return tabContent
end

-- Функция для создания переключателей (Toggles)
local function CreateToggle(parent, name, default, callback)
    local toggleFrame = Instance.new("Frame")
    toggleFrame.Size = UDim2.new(1, 0, 0, 40)
    toggleFrame.BackgroundColor3 = SECONDARY_COLOR
    toggleFrame.Parent = parent
    
    local toggleCorner = Instance.new("UICorner")
    toggleCorner.CornerRadius = UDim.new(0, 5)
    toggleCorner.Parent = toggleFrame
    
    local toggleLabel = Instance.new("TextLabel")
    toggleLabel.Text = name
    toggleLabel.Font = Enum.Font.SourceSans
    toggleLabel.TextSize = 16
    toggleLabel.TextColor3 = TEXT_COLOR
    toggleLabel.Size = UDim2.new(0.7, 0, 1, 0)
    toggleLabel.Position = UDim2.new(0, 10, 0, 0)
    toggleLabel.BackgroundTransparency = 1
    toggleLabel.TextXAlignment = Enum.TextXAlignment.Left
    toggleLabel.Parent = toggleFrame
    
    local toggleBtn = Instance.new("TextButton")
    toggleBtn.Size = UDim2.new(0, 50, 0, 25)
    toggleBtn.Position = UDim2.new(0.85, 0, 0.5, -12.5)
    toggleBtn.BackgroundColor3 = default and ACCENT_COLOR or Color3.fromRGB(80, 80, 80)
    toggleBtn.Text = default and "ON" or "OFF"
    toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    toggleBtn.Font = Enum.Font.SourceSans
    toggleBtn.TextSize = 14
    toggleBtn.Parent = toggleFrame
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 5)
    btnCorner.Parent = toggleBtn
    
    toggleBtn.MouseButton1Click:Connect(function()
        default = not default
        toggleBtn.BackgroundColor3 = default and ACCENT_COLOR or Color3.fromRGB(80, 80, 80)
        toggleBtn.Text = default and "ON" or "OFF"
        callback(default)
    end)
    
    return toggleBtn
end

-- Функция для создания кнопок
local function CreateButton(parent, name, callback)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(1, 0, 0, 40)
    button.BackgroundColor3 = BUTTON_COLOR
    button.Text = name
    button.TextColor3 = TEXT_COLOR
    button.Font = Enum.Font.SourceSans
    button.TextSize = 16
    button.Parent = parent
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 5)
    btnCorner.Parent = button
    
    button.MouseButton1Click:Connect(callback)
end

-- Функция для создания текстового поля (для загрузки конфига)
local function CreateTextBox(parent, name, callback)
    local textBoxFrame = Instance.new("Frame")
    textBoxFrame.Size = UDim2.new(1, 0, 0, 40)
    textBoxFrame.BackgroundColor3 = SECONDARY_COLOR
    textBoxFrame.Parent = parent
    
    local textBoxCorner = Instance.new("UICorner")
    textBoxCorner.CornerRadius = UDim.new(0, 5)
    textBoxCorner.Parent = textBoxFrame
    
    local textBoxLabel = Instance.new("TextLabel")
    textBoxLabel.Text = name
    textBoxLabel.Font = Enum.Font.SourceSans
    textBoxLabel.TextSize = 16
    textBoxLabel.TextColor3 = TEXT_COLOR
    textBoxLabel.Size = UDim2.new(0.3, 0, 1, 0)
    textBoxLabel.Position = UDim2.new(0, 10, 0, 0)
    textBoxLabel.BackgroundTransparency = 1
    textBoxLabel.TextXAlignment = Enum.TextXAlignment.Left
    textBoxLabel.Parent = textBoxFrame
    
    local textBox = Instance.new("TextBox")
    textBox.Size = UDim2.new(0.6, 0, 0, 25)
    textBox.Position = UDim2.new(0.35, 0, 0.5, -12.5)
    textBox.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    textBox.TextColor3 = TEXT_COLOR
    textBox.Font = Enum.Font.SourceSans
    textBox.TextSize = 14
    textBox.Text = ""
    textBox.Parent = textBoxFrame
    
    local textBoxCornerInner = Instance.new("UICorner")
    textBoxCornerInner.CornerRadius = UDim.new(0, 5)
    textBoxCornerInner.Parent = textBox
    
    textBox.FocusLost:Connect(function()
        callback(textBox.Text)
    end)
end

-- Функция для создания слайдеров
local function CreateSlider(parent, name, min, max, default, callback)
    local sliderFrame = Instance.new("Frame")
    sliderFrame.Size = UDim2.new(1, 0, 0, 60)
    sliderFrame.BackgroundColor3 = SECONDARY_COLOR
    sliderFrame.Parent = parent
    
    local sliderCorner = Instance.new("UICorner")
    sliderCorner.CornerRadius = UDim.new(0, 5)
    sliderCorner.Parent = sliderFrame
    
    local sliderLabel = Instance.new("TextLabel")
    sliderLabel.Text = name .. ": " .. default
    sliderLabel.Font = Enum.Font.SourceSans
    sliderLabel.TextSize = 16
    sliderLabel.TextColor3 = TEXT_COLOR
    sliderLabel.Size = UDim2.new(1, 0, 0, 30)
    sliderLabel.BackgroundTransparency = 1
    sliderLabel.TextXAlignment = Enum.TextXAlignment.Left
    sliderLabel.Position = UDim2.new(0, 10, 0, 0)
    sliderLabel.Parent = sliderFrame
    
    local sliderBar = Instance.new("Frame")
    sliderBar.Size = UDim2.new(0.9, 0, 0, 10)
    sliderBar.Position = UDim2.new(0.05, 0, 0.6, 0)
    sliderBar.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
    sliderBar.Parent = sliderFrame
    
    local barCorner = Instance.new("UICorner")
    barCorner.CornerRadius = UDim.new(0, 5)
    barCorner.Parent = sliderBar
    
    local fill = Instance.new("Frame")
    fill.Size = UDim2.new((default - min) / (max - min), 0, 1, 0)
    fill.BackgroundColor3 = ACCENT_COLOR
    fill.Parent = sliderBar
    
    local fillCorner = Instance.new("UICorner")
    fillCorner.CornerRadius = UDim.new(0, 5)
    fillCorner.Parent = fill
    
    local dragging = false
    sliderBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
        end
    end)
    
    sliderBar.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local mousePos = input.Position.X
            local barPos = sliderBar.AbsolutePosition.X
            local barSize = sliderBar.AbsoluteSize.X
            local relative = math.clamp((mousePos - barPos) / barSize, 0, 1)
            local value = min + (max - min) * relative
            value = math.floor(value)
            fill.Size = UDim2.new(relative, 0, 1, 0)
            sliderLabel.Text = name .. ": " .. value
            callback(value)
        end
    end)
end

-- Вкладки
local movementTab = CreateTab("Movement", "🚀")
local visualsTab = CreateTab("Visuals", "👁️")
local farmTab = CreateTab("Farm", "🌿")
local combatTab = CreateTab("Combat", "⚔️")
local animationsTab = CreateTab("Animations", "🎭")
local miscTab = CreateTab("Misc", "✨")

-- Активируем первую вкладку
for i, child in ipairs(Sidebar:GetChildren()) do
    if child:IsA("TextButton") then
        child.BackgroundColor3 = i == 1 and ACCENT_COLOR or SECONDARY_COLOR
        child.TextColor3 = i == 1 and Color3.fromRGB(255, 255, 255) or TEXT_COLOR
        break
    end
end
for i, child in ipairs(ContentFrame:GetChildren()) do
    if child:IsA("ScrollingFrame") then
        child.Visible = i == 1
        break
    end
end

-- Movement Tab
local CurrentWalkSpeed = BASE_WALK_SPEED
local SpeedConnection

local function SetWalkSpeed(speed)
    CurrentWalkSpeed = math.clamp(speed, BASE_WALK_SPEED, MAX_WALK_SPEED)
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        LocalPlayer.Character.Humanoid.WalkSpeed = CurrentWalkSpeed
    end
end

local function UpdateWalkSpeed()
    if SpeedConnection then
        SpeedConnection:Disconnect()
    end
    SpeedConnection = RunService.Heartbeat:Connect(function()
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
            local humanoid = LocalPlayer.Character.Humanoid
            if humanoid.WalkSpeed ~= CurrentWalkSpeed then
                humanoid.WalkSpeed = CurrentWalkSpeed
            end
        end
    end)
end

LocalPlayer.CharacterAdded:Connect(function(character)
    local humanoid = character:WaitForChild("Humanoid", 5)
    if humanoid then
        humanoid.WalkSpeed = CurrentWalkSpeed
        UpdateWalkSpeed()
    end
end)

if LocalPlayer.Character then
    UpdateWalkSpeed()
end

CreateSlider(movementTab, "Walk Speed", BASE_WALK_SPEED, MAX_WALK_SPEED, BASE_WALK_SPEED, function(value)
    SetWalkSpeed(value)
end)

CreateToggle(movementTab, "Mountain Climber", false, function(state)
    MOUNTAIN_CLIMBER_ENABLED = state
    if MOUNTAIN_CLIMBER_ENABLED then
        spawn(function()
            while MOUNTAIN_CLIMBER_ENABLED and task.wait(0.05) do
                if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") and LocalPlayer.Character:FindFirstChild("Humanoid") then
                    local rootPart = LocalPlayer.Character.HumanoidRootPart
                    local humanoid = LocalPlayer.Character.Humanoid
                    
                    local raycastParams = RaycastParams.new()
                    raycastParams.FilterDescendantsInstances = {LocalPlayer.Character}
                    raycastParams.FilterType = Enum.RaycastFilterType.Exclude
                    local rayOrigin = rootPart.Position
                    local rayDirection = Vector3.new(0, -5, 0)
                    local raycastResult = workspace:Raycast(rayOrigin, rayDirection, raycastParams)
                    
                    local forwardRayDirection = rootPart.CFrame.LookVector * 3
                    local forwardRayResult = workspace:Raycast(rayOrigin, forwardRayDirection, raycastParams)
                    
                    if raycastResult and forwardRayResult then
                        local normal = raycastResult.Normal
                        local slopeAngle = math.deg(math.acos(normal.Y))
                        
                        if slopeAngle > 30 and slopeAngle < 85 then
                            local vectorForce = rootPart:FindFirstChild("ClimbForce") or Instance.new("VectorForce")
                            vectorForce.Name = "ClimbForce"
                            vectorForce.Attachment0 = rootPart:FindFirstChild("RootAttachment") or Instance.new("Attachment", rootPart)
                            vectorForce.Force = Vector3.new(0, 8000, 0)
                            vectorForce.Parent = rootPart
                            
                            local alignOrientation = rootPart:FindFirstChild("ClimbOrientation") or Instance.new("AlignOrientation")
                            alignOrientation.Name = "ClimbOrientation"
                            alignOrientation.MaxTorque = 10000
                            alignOrientation.Responsiveness = 50
                            alignOrientation.CFrame = CFrame.new(Vector3.new(0, 0, 0), normal * Vector3.new(1, 0, 1))
                            alignOrientation.Parent = rootPart
                            
                            local moveDirection = (normal * Vector3.new(1, 0, 1)).Unit * 5
                            humanoid:Move(moveDirection, true)
                            humanoid.JumpPower = 0
                        else
                            if rootPart:FindFirstChild("ClimbForce") then
                                rootPart.ClimbForce:Destroy()
                            end
                            if rootPart:FindFirstChild("ClimbOrientation") then
                                rootPart.ClimbOrientation:Destroy()
                            end
                            humanoid.JumpPower = 50
                        end
                    else
                        if rootPart:FindFirstChild("ClimbForce") then
                            rootPart.ClimbForce:Destroy()
                        end
                        if rootPart:FindFirstChild("ClimbOrientation") then
                            rootPart.ClimbOrientation:Destroy()
                        end
                        humanoid.JumpPower = 50
                    end
                end
            end
            if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") and LocalPlayer.Character:FindFirstChild("Humanoid") then
                local rootPart = LocalPlayer.Character.HumanoidRootPart
                local humanoid = LocalPlayer.Character.Humanoid
                if rootPart:FindFirstChild("ClimbForce") then
                    rootPart.ClimbForce:Destroy()
                end
                if rootPart:FindFirstChild("ClimbOrientation") then
                    rootPart.ClimbOrientation:Destroy()
                end
                humanoid.JumpPower = 50
            end
        end)
    end
end)

-- Функция для создания точки
local function CreateWaypoint(shape)
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        local waypointPart = Instance.new("Part")
        waypointPart.Position = LocalPlayer.Character.HumanoidRootPart.Position + Vector3.new(0, 1, 0)
        waypointPart.Anchored = true
        waypointPart.CanCollide = false
        waypointPart.Transparency = 0.5
        
        if shape == "Square" then
            waypointPart.Size = Vector3.new(2, 0.2, 2)
            waypointPart.BrickColor = BrickColor.new("Bright blue")
            waypointPart:SetAttribute("Shape", "Square")
        elseif shape == "Circle" then
            waypointPart.Size = Vector3.new(2, 0.2, 2)
            waypointPart.BrickColor = BrickColor.new("Bright purple")
            waypointPart.Shape = Enum.PartType.Cylinder
            waypointPart.Orientation = Vector3.new(90, 0, 0)
            waypointPart:SetAttribute("Shape", "Circle")
        else
            waypointPart.Size = Vector3.new(1, 1, 1)
            waypointPart.BrickColor = BrickColor.new("Lime green")
            waypointPart:SetAttribute("Shape", "Default")
        end
        
        waypointPart.Parent = WAYPOINTS_HOLDER
        waypointPart:SetAttribute("Visited", false)
        
        local billboard = Instance.new("BillboardGui")
        billboard.Size = UDim2.new(0, 50, 0, 20)
        billboard.StudsOffset = Vector3.new(0, 2, 0)
        billboard.Parent = waypointPart
        
        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(1, 0, 1, 0)
        label.BackgroundTransparency = 1
        label.Text = "Point " .. #WAYPOINTS_HOLDER:GetChildren()
        label.TextColor3 = Color3.new(1, 1, 1)
        label.TextStrokeTransparency = 0
        label.TextStrokeColor3 = Color3.new(0, 0, 0)
        label.Font = Enum.Font.SourceSansBold
        label.TextSize = 12
        label.Parent = billboard
    end
end

CreateToggle(movementTab, "Waypoints (E to set)", false, function(state)
    WAYPOINTS_ENABLED = state
    WAYPOINTS_MOVING = false
    if WAYPOINTS_ENABLED then
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "BoogaX",
            Text = "Press E to set waypoints! Q for Square, R for Circle",
            Duration = 3
        })
        
        local waypointConnection
        
        waypointConnection = UserInputService.InputBegan:Connect(function(input, gameProcessed)
            if not gameProcessed and WAYPOINTS_ENABLED then
                local currentTime = tick()
                if currentTime - lastWaypointCreationTime < WAYPOINT_CREATION_COOLDOWN then
                    return
                end
                lastWaypointCreationTime = currentTime
                
                if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                    local shape = "Default"
                    if input.KeyCode == Enum.KeyCode.E then
                        shape = "Default"
                    elseif input.KeyCode == Enum.KeyCode.Q then
                        shape = "Square"
                    elseif input.KeyCode == Enum.KeyCode.R then
                        shape = "Circle"
                    end
                    
                    if shape ~= "Default" or input.KeyCode == Enum.KeyCode.E then
                        CreateWaypoint(shape)
                    end
                end
            end
        end)
        
        if not WAYPOINTS_ENABLED and waypointConnection then
            waypointConnection:Disconnect()
        end
    end
end)

-- Тоггл для сенсорного ввода
CreateToggle(movementTab, "Touch Waypoints", false, function(state)
    TOUCH_WAYPOINTS_ENABLED = state
    WAYPOINTS_MOVING = false
    if TOUCH_WAYPOINTS_ENABLED then
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "BoogaX",
            Text = "Tap to set waypoints! Double-tap for Square, long press for Circle",
            Duration = 3
        })
        
        local touchConnection
        
        touchConnection = UserInputService.InputBegan:Connect(function(input, gameProcessed)
            if not gameProcessed and TOUCH_WAYPOINTS_ENABLED and input.UserInputType == Enum.UserInputType.Touch then
                local currentTime = tick()
                if currentTime - lastWaypointCreationTime < WAYPOINT_CREATION_COOLDOWN then
                    return
                end
                
                touchCount = touchCount + 1
                touchStartTime = currentTime
                
                -- Проверка на двойное касание
                if touchCount == 2 and (currentTime - touchStartTime) <= DOUBLE_TAP_THRESHOLD then
                    lastWaypointCreationTime = currentTime
                    CreateWaypoint("Square")
                    touchCount = 0
                end
            end
        end)
        
        UserInputService.InputEnded:Connect(function(input, gameProcessed)
            if not gameProcessed and TOUCH_WAYPOINTS_ENABLED and input.UserInputType == Enum.UserInputType.Touch then
                local currentTime = tick()
                local touchDuration = currentTime - touchStartTime
                
                -- Если это не двойное касание, проверяем длительность нажатия
                if touchCount == 1 then
                    if touchDuration >= LONG_PRESS_DURATION then
                        lastWaypointCreationTime = currentTime
                        CreateWaypoint("Circle")
                    else
                        lastWaypointCreationTime = currentTime
                        CreateWaypoint("Default")
                    end
                end
                touchCount = 0
            end
        end)
        
        if not TOUCH_WAYPOINTS_ENABLED and touchConnection then
            touchConnection:Disconnect()
        end
    end
end)

-- Кнопка для начала движения по точкам
CreateButton(movementTab, "Start Waypoints Movement", function()
    if WAYPOINTS_ENABLED or TOUCH_WAYPOINTS_ENABLED then
        WAYPOINTS_MOVING = true
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "BoogaX",
            Text = "Started moving to waypoints!",
            Duration = 3
        })
        
        local currentWaypointIndex = 1
        local lastWaypointPosition = nil
        
        spawn(function()
            while (WAYPOINTS_ENABLED or TOUCH_WAYPOINTS_ENABLED) and WAYPOINTS_MOVING do
                if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                    local humanoid = LocalPlayer.Character.Humanoid
                    local rootPart = LocalPlayer.Character.HumanoidRootPart
                    local waypoints = WAYPOINTS_HOLDER:GetChildren()
                    
                    local isPlayerMoving = humanoid.MoveDirection.Magnitude > 0
                    local wasMovingLastFrame = true
                    
                    if #waypoints > 0 then
                        table.sort(waypoints, function(a, b)
                            local aIndex = tonumber(a:FindFirstChildOfClass("BillboardGui"):FindFirstChildOfClass("TextLabel").Text:match("%d+"))
                            local bIndex = tonumber(b:FindFirstChildOfClass("BillboardGui"):FindFirstChildOfClass("TextLabel").Text:match("%d+"))
                            return aIndex < bIndex
                        end)
                        
                        local currentWaypoint = nil
                        for i = currentWaypointIndex, #waypoints do
                            local waypoint = waypoints[i]
                            if not waypoint:GetAttribute("Visited") then
                                currentWaypoint = waypoint
                                currentWaypointIndex = i
                                break
                            end
                        end
                        
                        if not currentWaypoint then
                            for _, waypoint in ipairs(waypoints) do
                                waypoint:SetAttribute("Visited", false)
                                waypoint.BrickColor = waypoint.Size.X == 2 and waypoint.Shape == Enum.PartType.Cylinder and BrickColor.new("Bright purple") or waypoint.Size.X == 2 and BrickColor.new("Bright blue") or BrickColor.new("Lime green")
                            end
                            currentWaypointIndex = 1
                            currentWaypoint = waypoints[1]
                        end
                        
                        if currentWaypoint then
                            lastWaypointPosition = currentWaypoint.Position
                            humanoid:MoveTo(currentWaypoint.Position)
                            local distance = (rootPart.Position - currentWaypoint.Position).Magnitude
                            if distance < 2 then
                                currentWaypoint:SetAttribute("Visited", true)
                                currentWaypoint.BrickColor = BrickColor.new("Really red")
                                currentWaypointIndex = currentWaypointIndex + 1
                            end
                        end
                    elseif lastWaypointPosition then
                        humanoid:MoveTo(lastWaypointPosition)
                    end
                    
                    if not isPlayerMoving and wasMovingLastFrame then
                        if #waypoints > 0 then
                            local currentWaypoint = nil
                            for i = currentWaypointIndex, #waypoints do
                                local waypoint = waypoints[i]
                                if not waypoint:GetAttribute("Visited") then
                                    currentWaypoint = waypoint
                                    break
                                end
                            end
                            if currentWaypoint then
                                humanoid:MoveTo(currentWaypoint.Position)
                            elseif lastWaypointPosition then
                                humanoid:MoveTo(lastWaypointPosition)
                            end
                        elseif lastWaypointPosition then
                            humanoid:MoveTo(lastWaypointPosition)
                        end
                    end
                    
                    wasMovingLastFrame = isPlayerMoving
                end
                task.wait(0.1)
            end
        end)
    else
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "BoogaX",
            Text = "Enable Waypoints or Touch Waypoints first!",
            Duration = 3
        })
    end
end)

-- Кнопка для сохранения конфигурации Waypoints
CreateButton(movementTab, "Save Waypoints Config", function()
    local waypoints = WAYPOINTS_HOLDER:GetChildren()
    local config = {}
    
    for i, waypoint in ipairs(waypoints) do
        local waypointData = {
            Position = {X = waypoint.Position.X, Y = waypoint.Position.Y, Z = waypoint.Position.Z},
            Shape = waypoint:GetAttribute("Shape") or "Default",
            Visited = waypoint:GetAttribute("Visited") or false,
            Index = i
        }
        table.insert(config, waypointData)
    end
    
    local jsonConfig = HttpService:JSONEncode(config)
    setclipboard(jsonConfig)
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "BoogaX",
        Text = "Waypoints config copied to clipboard!",
        Duration = 3
    })
end)

-- Текстовое поле и кнопка для загрузки конфигурации Waypoints
CreateTextBox(movementTab, "Paste Config", function(configText)
    if configText == "" then
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "BoogaX",
            Text = "Config is empty!",
            Duration = 3
        })
        return
    end
    
    local success, config = pcall(function()
        return HttpService:JSONDecode(configText)
    end)
    
    if not success then
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "BoogaX",
            Text = "Failed to decode config! Please check the JSON format.",
            Duration = 3
        })
        print("JSON Decode Error:", config)
        return
    end
    
    if type(config) ~= "table" then
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "BoogaX",
            Text = "Invalid config format! Expected a table.",
            Duration = 3
        })
        return
    end
    
    WAYPOINTS_HOLDER:ClearAllChildren()
    WAYPOINTS_MOVING = false
    
    for i, waypointData in ipairs(config) do
        if not waypointData.Position or type(waypointData.Position) ~= "table" or
           not waypointData.Position.X or not waypointData.Position.Y or not waypointData.Position.Z or
           type(waypointData.Position.X) ~= "number" or type(waypointData.Position.Y) ~= "number" or type(waypointData.Position.Z) ~= "number" then
            game:GetService("StarterGui"):SetCore("SendNotification", {
                Title = "BoogaX",
                Text = "Invalid position data in config at index " .. i,
                Duration = 3
            })
            continue
        end
        
        local waypointPart = Instance.new("Part")
        waypointPart.Position = Vector3.new(waypointData.Position.X, waypointData.Position.Y, waypointData.Position.Z)
        waypointPart.Anchored = true
        waypointPart.CanCollide = false
        waypointPart.Transparency = 0.5
        
        local shape = waypointData.Shape or "Default"
        if shape == "Square" then
            waypointPart.Size = Vector3.new(2, 0.2, 2)
            waypointPart.BrickColor = BrickColor.new("Bright blue")
            waypointPart:SetAttribute("Shape", "Square")
        elseif shape == "Circle" then
            waypointPart.Size = Vector3.new(2, 0.2, 2)
            waypointPart.BrickColor = BrickColor.new("Bright purple")
            waypointPart.Shape = Enum.PartType.Cylinder
            waypointPart.Orientation = Vector3.new(90, 0, 0)
            waypointPart:SetAttribute("Shape", "Circle")
        else
            waypointPart.Size = Vector3.new(1, 1, 1)
            waypointPart.BrickColor = BrickColor.new("Lime green")
            waypointPart:SetAttribute("Shape", "Default")
        end
        
        waypointPart:SetAttribute("Visited", waypointData.Visited or false)
        if waypointData.Visited then
            waypointPart.BrickColor = BrickColor.new("Really red")
        end
        
        waypointPart.Parent = WAYPOINTS_HOLDER
        
        local billboard = Instance.new("BillboardGui")
        billboard.Size = UDim2.new(0, 50, 0, 20)
        billboard.StudsOffset = Vector3.new(0, 2, 0)
        billboard.Parent = waypointPart
        
        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(1, 0, 1, 0)
        label.BackgroundTransparency = 1
        label.Text = "Point " .. (waypointData.Index or i)
        label.TextColor3 = Color3.new(1, 1, 1)
        label.TextStrokeTransparency = 0
        label.TextStrokeColor3 = Color3.new(0, 0, 0)
        label.Font = Enum.Font.SourceSansBold
        label.TextSize = 12
        label.Parent = billboard
    end
    
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "BoogaX",
        Text = "Waypoints config loaded!",
        Duration = 3
    })
end)

CreateButton(movementTab, "Clear Waypoints", function()
    WAYPOINTS_HOLDER:ClearAllChildren()
    WAYPOINTS_MOVING = false
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "BoogaX",
        Text = "All waypoints cleared!",
        Duration = 3
    })
end)

-- Visuals Tab
CreateToggle(visualsTab, "ESP Players", false, function(state)
    ESP_ENABLED = state
    if ESP_ENABLED then
        local function CreateESP(player)
            if player == LocalPlayer or not player.Character then return end
            local character = player.Character
            local rootPart = character:FindFirstChild("HumanoidRootPart")
            local humanoid = character:FindFirstChild("Humanoid")
            if not rootPart or not humanoid or humanoid.Health <= 0 then return end
            if ESP_HOLDER:FindFirstChild("ESP_" .. player.Name) then return end
            
            local box = Instance.new("BoxHandleAdornment")
            box.Name = "ESP_" .. player.Name
            box.Adornee = rootPart
            box.AlwaysOnTop = true
            box.ZIndex = 1
            box.Size = Vector3.new(3, 6, 3)
            box.Transparency = 0.7
            box.Color3 = player.TeamColor.Color or Color3.fromRGB(255, 255, 255)
            box.Parent = ESP_HOLDER
            
            local billboard = Instance.new("BillboardGui")
            billboard.Name = "Distance_" .. player.Name
            billboard.Adornee = rootPart
            billboard.Size = UDim2.new(0, 100, 0, 60)
            billboard.StudsOffset = Vector3.new(0, 4, 0)
            billboard.AlwaysOnTop = true
            billboard.Parent = ESP_HOLDER
            
            local nameLabel = Instance.new("TextLabel")
            nameLabel.Size = UDim2.new(1, 0, 0.5, 0)
            nameLabel.Position = UDim2.new(0, 0, 0, 0)
            nameLabel.BackgroundTransparency = 1
            nameLabel.TextColor3 = Color3.new(1, 1, 1)
            nameLabel.TextStrokeTransparency = 0
            nameLabel.TextStrokeColor3 = Color3.new(0, 0, 0)
            nameLabel.Font = Enum.Font.SourceSansBold
            nameLabel.TextSize = 14
            nameLabel.Text = SHOW_NAMES_ENABLED and player.Name or ""
            nameLabel.Parent = billboard
            
            local distanceLabel = Instance.new("TextLabel")
            distanceLabel.Size = UDim2.new(1, 0, 0.5, 0)
            distanceLabel.Position = UDim2.new(0, 0, 0.5, 0)
            distanceLabel.BackgroundTransparency = 1
            distanceLabel.TextColor3 = Color3.new(1, 1, 1)
            distanceLabel.TextStrokeTransparency = 0
            distanceLabel.TextStrokeColor3 = Color3.new(0, 0, 0)
            distanceLabel.Font = Enum.Font.SourceSansBold
            distanceLabel.TextSize = 14
            distanceLabel.Parent = billboard
            
            local function UpdateDistance()
                if not ESP_ENABLED or not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("HumanoidRootPart") or not character or not character.Parent then
                    if box.Parent then box:Destroy() end
                    if billboard.Parent then billboard:Destroy() end
                    return
                end
                local distance = (LocalPlayer.Character.HumanoidRootPart.Position - rootPart.Position).Magnitude
                distanceLabel.Text = string.format("%.1f m", distance)
                nameLabel.Text = SHOW_NAMES_ENABLED and player.Name or ""
            end
            
            RunService.RenderStepped:Connect(UpdateDistance)
        end
        
        for _, player in ipairs(Players:GetPlayers()) do
            CreateESP(player)
        end
        Players.PlayerAdded:Connect(CreateESP)
    else
        ESP_HOLDER:ClearAllChildren()
    end
end)

CreateToggle(visualsTab, "Show Names", false, function(state)
    SHOW_NAMES_ENABLED = state
    for _, player in ipairs(Players:GetPlayers()) do
        local billboard = ESP_HOLDER:FindFirstChild("Distance_" .. player.Name)
        if billboard then
            local nameLabel = billboard:FindFirstChildOfClass("TextLabel")
            if nameLabel and nameLabel.Position == UDim2.new(0, 0, 0, 0) then
                nameLabel.Text = SHOW_NAMES_ENABLED and player.Name or ""
            end
        end
    end
end)

CreateToggle(visualsTab, "Night Vision", false, function(state)
    NIGHT_VISION_ENABLED = state
    if NIGHT_VISION_ENABLED then
        Lighting.Ambient = Color3.new(0.5, 0.5, 0.5)
        Lighting.Brightness = 1
        Lighting.OutdoorAmbient = Color3.new(0.5, 0.5, 0.5)
    else
        Lighting.Ambient = Color3.new(0, 0, 0)
        Lighting.Brightness = 1
        Lighting.OutdoorAmbient = Color3.new(0, 0, 0)
    end
end)

CreateToggle(visualsTab, "Glow", false, function(state)
    GLOW_ENABLED = state
    if GLOW_ENABLED then
        local function ApplyGlow(character)
            if not character then return end
            if GlowHighlight then GlowHighlight:Destroy() end
            GlowHighlight = Instance.new("Highlight")
            GlowHighlight.Name = "PlayerGlow"
            GlowHighlight.FillColor = Color3.fromRGB(GLOW_COLOR_R, GLOW_COLOR_G, GLOW_COLOR_B)
            GlowHighlight.OutlineColor = Color3.fromRGB(GLOW_COLOR_R, GLOW_COLOR_G, GLOW_COLOR_B)
            GlowHighlight.FillTransparency = 0.5
            GlowHighlight.OutlineTransparency = 0
            GlowHighlight.Adornee = character
            GlowHighlight.Parent = character
        end
        
        if LocalPlayer.Character then
            ApplyGlow(LocalPlayer.Character)
        end
        
        LocalPlayer.CharacterAdded:Connect(function(character)
            if GLOW_ENABLED then
                ApplyGlow(character)
            end
        end)
    else
        if GlowHighlight then
            GlowHighlight:Destroy()
            GlowHighlight = nil
        end
    end
end)

CreateSlider(visualsTab, "Glow Red", 0, 255, 255, function(value)
    GLOW_COLOR_R = value
    if GLOW_ENABLED and GlowHighlight then
        GlowHighlight.FillColor = Color3.fromRGB(GLOW_COLOR_R, GLOW_COLOR_G, GLOW_COLOR_B)
        GlowHighlight.OutlineColor = Color3.fromRGB(GLOW_COLOR_R, GLOW_COLOR_G, GLOW_COLOR_B)
    end
end)

CreateSlider(visualsTab, "Glow Green", 0, 255, 255, function(value)
    GLOW_COLOR_G = value
    if GLOW_ENABLED and GlowHighlight then
        GlowHighlight.FillColor = Color3.fromRGB(GLOW_COLOR_R, GLOW_COLOR_G, GLOW_COLOR_B)
        GlowHighlight.OutlineColor = Color3.fromRGB(GLOW_COLOR_R, GLOW_COLOR_G, GLOW_COLOR_B)
    end
end)

CreateSlider(visualsTab, "Glow Blue", 0, 255, 255, function(value)
    GLOW_COLOR_B = value
    if GLOW_ENABLED and GlowHighlight then
        GlowHighlight.FillColor = Color3.fromRGB(GLOW_COLOR_R, GLOW_COLOR_G, GLOW_COLOR_B)
        GlowHighlight.OutlineColor = Color3.fromRGB(GLOW_COLOR_R, GLOW_COLOR_G, GLOW_COLOR_B)
    end
end)

-- Farm Tab
CreateToggle(farmTab, "Auto Farm", false, function(state)
    AUTO_FARM_ENABLED = state
    if AUTO_FARM_ENABLED then
        spawn(function()
            while AUTO_FARM_ENABLED and task.wait(0.5) do
                if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                    for _, tree in ipairs(workspace:GetDescendants()) do
                        if tree.Name:find("Tree") and tree:IsA("BasePart") then
                            local distance = (LocalPlayer.Character.HumanoidRootPart.Position - tree.Position).Magnitude
                            if distance < 15 then
                                game:GetService("ReplicatedStorage").Remotes.Tool:FireServer()
                            end
                        end
                    end
                end
            end
        end)
    end
end)

CreateToggle(farmTab, "Auto Eat", false, function(state)
    AUTO_EAT_ENABLED = state
    if AUTO_EAT_ENABLED then
        spawn(function()
            while AUTO_EAT_ENABLED and task.wait(5) do
                if LocalPlayer.Character then
                    game:GetService("ReplicatedStorage").Remotes.Eat:FireServer()
                end
            end
        end)
    end
end)

-- Combat Tab
CreateToggle(combatTab, "Hitboxes", false, function(state)
    HITBOXES_ENABLED = state
    local function UpdatePlayerHitbox(player)
        if player == LocalPlayer or not player.Character then return end
        local character = player.Character
        local humanoid = character:FindFirstChild("Humanoid")
        if not humanoid or humanoid.Health <= 0 then return end
        for _, part in ipairs(character:GetChildren()) do
            if part:IsA("BasePart") then
                local highlight = HITBOX_HOLDER:FindFirstChild("HitboxVisual_" .. part.Name)
                if highlight then highlight:Destroy() end
                if HITBOXES_ENABLED then
                    if (HITBOX_TYPE == "All") or (HITBOX_TYPE == "Head" and part.Name == "Head") or (HITBOX_TYPE == "Torso" and part.Name == "Torso") then
                        part.Size = Vector3.new(HITBOX_SIZE, HITBOX_SIZE, HITBOX_SIZE)
                        local highlight = Instance.new("BoxHandleAdornment")
                        highlight.Name = "HitboxVisual_" .. part.Name
                        highlight.Adornee = part
                        highlight.Size = part.Size
                        highlight.Transparency = HITBOX_TRANSPARENCY
                        highlight.Color3 = ACCENT_COLOR
                        highlight.AlwaysOnTop = true
                        highlight.ZIndex = 2
                        highlight.Parent = HITBOX_HOLDER
                    else
                        if part.Name == "Head" then part.Size = Vector3.new(1, 1, 1)
                        elseif part.Name == "Torso" then part.Size = Vector3.new(2, 2, 1)
                        else part.Size = Vector3.new(1, 2, 1) end
                    end
                else
                    if part.Name == "Head" then part.Size = Vector3.new(1, 1, 1)
                    elseif part.Name == "Torso" then part.Size = Vector3.new(2, 2, 1)
                    else part.Size = Vector3.new(1, 2, 1) end
                end
            end
        end
    end
    if HITBOXES_ENABLED then
        for _, player in ipairs(Players:GetPlayers()) do
            UpdatePlayerHitbox(player)
        end
        Players.PlayerAdded:Connect(UpdatePlayerHitbox)
    else
        HITBOX_HOLDER:ClearAllChildren()
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                for _, part in ipairs(player.Character:GetChildren()) do
                    if part:IsA("BasePart") then
                        if part.Name == "Head" then part.Size = Vector3.new(1, 1, 1)
                        elseif part.Name == "Torso" then part.Size = Vector3.new(2, 2, 1)
                        else part.Size = Vector3.new(1, 2, 1) end
                    end
                end
            end
        end
    end
end)

CreateSlider(combatTab, "Hitbox Size", 1, 5, 2, function(value)
    HITBOX_SIZE = value
end)

CreateSlider(combatTab, "Hitbox Transparency", 0.1, 1, 0.7, function(value)
    HITBOX_TRANSPARENCY = value
end)

CreateToggle(combatTab, "Kill Aura", false, function(state)
    KILL_AURA_ENABLED = state
    if KILL_AURA_ENABLED then
        spawn(function()
            while KILL_AURA_ENABLED and task.wait(KILL_AURA_COOLDOWN) do
                if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                    local rootPart = LocalPlayer.Character.HumanoidRootPart
                    for _, player in ipairs(Players:GetPlayers()) do
                        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Humanoid") and player.Character:FindFirstChild("HumanoidRootPart") then
                            local targetRoot = player.Character.HumanoidRootPart
                            local distance = (rootPart.Position - targetRoot.Position).Magnitude
                            if distance <= KILL_AURA_RANGE then
                                local humanoid = player.Character.Humanoid
                                if humanoid and humanoid.Health > 0 then
                                    local tool = LocalPlayer.Character:FindFirstChildOfClass("Tool")
                                    if tool then
                                        tool:Activate()
                                    end
                                end
                            end
                        end
                    end
                end
            end
        end)
    end
end)

CreateSlider(combatTab, "Kill Aura Range", 5, 30, 15, function(value)
    KILL_AURA_RANGE = value
end)

CreateSlider(combatTab, "Kill Aura Cooldown", 0.1, 2, 1, function(value)
    KILL_AURA_COOLDOWN = value
end)

-- Animations Tab
local function PlayAnimation(animationId, enabled, button, name)
    if not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("Humanoid") then return end
    local humanoid = LocalPlayer.Character.Humanoid
    if currentAnimationTrack then currentAnimationTrack:Stop() end
    for _, track in pairs(animationTracks) do track:Stop() end
    table.clear(animationTracks)
    if enabled then
        local animation = Instance.new("Animation")
        animation.AnimationId = animationId
        local track = humanoid:LoadAnimation(animation)
        animationTracks[name] = track
        track:Play()
        currentAnimationTrack = track
    end
end

CreateToggle(animationsTab, "Animation 1", false, function(state)
    ANIMATION_1_ENABLED = state
    ANIMATION_2_ENABLED = false
    ANIMATION_3_ENABLED = false
    ANIMATION_4_ENABLED = false
    PlayAnimation("rbxassetid://10761453950", ANIMATION_1_ENABLED, nil, "Animation 1")
end)

CreateToggle(animationsTab, "Animation 2", false, function(state)
    ANIMATION_2_ENABLED = state
    ANIMATION_1_ENABLED = false
    ANIMATION_3_ENABLED = false
    ANIMATION_4_ENABLED = false
    PlayAnimation("rbxassetid://18754971429", ANIMATION_2_ENABLED, nil, "Animation 2")
end)

CreateToggle(animationsTab, "Animation 3", false, function(state)
    ANIMATION_3_ENABLED = state
    ANIMATION_1_ENABLED = false
    ANIMATION_2_ENABLED = false
    ANIMATION_4_ENABLED = false
    PlayAnimation("rbxassetid://18754935971", ANIMATION_3_ENABLED, nil, "Animation 3")
end)

CreateToggle(animationsTab, "Animation 4", false, function(state)
    ANIMATION_4_ENABLED = state
    ANIMATION_1_ENABLED = false
    ANIMATION_2_ENABLED = false
    ANIMATION_3_ENABLED = false
    PlayAnimation("rbxassetid://18764584102", ANIMATION_4_ENABLED, nil, "Animation 4")
end)

LocalPlayer.CharacterAdded:Connect(function(character)
    local humanoid = character:WaitForChild("Humanoid", 5)
    for _, track in pairs(animationTracks) do track:Stop() end
    animationTracks = {}
    currentAnimationTrack = nil
    if ANIMATION_1_ENABLED then PlayAnimation("rbxassetid://10761453950", true, nil, "Animation 1") end
    if ANIMATION_2_ENABLED then PlayAnimation("rbxassetid://18754971429", true, nil, "Animation 2") end
    if ANIMATION_3_ENABLED then PlayAnimation("rbxassetid://18754935971", true, nil, "Animation 3") end
    if ANIMATION_4_ENABLED then PlayAnimation("rbxassetid://18764584102", true, nil, "Animation 4") end
end)

-- Misc Tab
CreateButton(miscTab, "Copy Discord Link", function()
    setclipboard("https://discord.gg/yourlink")
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "BoogaX",
        Text = "Discord link copied!",
        Duration = 3
    })
end)

local infoLabel = Instance.new("TextLabel")
infoLabel.Text = "Version: 2.4\nDeveloper: YourName"
infoLabel.Font = Enum.Font.SourceSans
infoLabel.TextSize = 16
infoLabel.TextColor3 = TEXT_COLOR
infoLabel.Size = UDim2.new(1, 0, 0, 60)
infoLabel.BackgroundTransparency = 1
infoLabel.TextXAlignment = Enum.TextXAlignment.Left
infoLabel.TextYAlignment = Enum.TextYAlignment.Top
infoLabel.Position = UDim2.new(0, 10, 0, 10)
infoLabel.Parent = miscTab

-- Уведомление о загрузке
task.spawn(function()
    local notification = Instance.new("Frame")
    notification.Size = UDim2.new(0, 250, 0, 70)
    notification.Position = UDim2.new(0.5, -125, 1, 20)
    notification.BackgroundColor3 = BACKGROUND_COLOR
    notification.Parent = ScreenGui
    
    local notifCorner = Instance.new("UICorner")
    notifCorner.CornerRadius = UDim.new(0, 8)
    notifCorner.Parent = notification
    
    local notifStroke = Instance.new("UIStroke")
    notifStroke.Color = ACCENT_COLOR
    notifStroke.Thickness = 1
    notifStroke.Parent = notification
    
    local notifText = Instance.new("TextLabel")
    notifText.Text = "BoogaX Loaded"
    notifText.Font = Enum.Font.SourceSansBold
    notifText.TextSize = 16
    notifText.TextColor3 = ACCENT_COLOR
    notifText.Size = UDim2.new(1, 0, 1, 0)
    notifText.BackgroundTransparency = 1
    notifText.Parent = notification
    
    TweenService:Create(notification, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -125, 1, -90)}):Play()
    wait(3)
    TweenService:Create(notification, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -125, 1, 20)}):Play()
    wait(0.3)
    notification:Destroy()
end)

-- Логика сворачивания
local isMinimized = false

MinimizeBtn.MouseButton1Click:Connect(function()
    isMinimized = not isMinimized
    if isMinimized then
        TweenService:Create(MainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quad), {
            Size = UDim2.new(0, 200, 0, 60),
            Position = UDim2.new(0.5, -100, 0.5, -30)
        }):Play()
        wait(0.3)
        Sidebar.Visible = false
        ContentFrame.Visible = false
        Title.Visible = false
        MinimizeBtn.Text = "+"
        MinimizedFrame.Visible = true
    else
        MinimizedFrame.Visible = false
        Sidebar.Visible = true
        ContentFrame.Visible = true
        Title.Visible = true
        MinimizeBtn.Text = "-"
        TweenService:Create(MainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quad), {
            Size = UDim2.new(0, 600, 0, 400),
            Position = UDim2.new(0.5, -300, 0.5, -200)
        }):Play()
    end
end)

CloseBtn.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)

SetWalkSpeed(BASE_WALK_SPEED)
