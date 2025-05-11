-- Переменные
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

-- Настройки
local visitDuration = 1 -- Стандартная длительность в секундах
local isScriptActive = false
local guiVisible = true
local followMode = false
local currentTarget = nil
local followConnection = nil
local lastTarget = nil
local manualFollowMode = false
local followDistance = 1.4 -- Фиксированная дистанция следования в метрах

-- Создаем основной интерфейс
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "PlayerTeleporterGUI"
ScreenGui.Parent = CoreGui

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 320, 0, 350)
MainFrame.Position = UDim2.new(0.5, -160, 0.5, -175)
MainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
MainFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

-- Добавляем возможность перемещения MainFrame
local dragging
local dragInput
local dragStart
local startPos

local function updateInput(input)
    local delta = input.Position - dragStart
    MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

MainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

MainFrame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        updateInput(input)
    end
end)

-- Заголовок
local Title = Instance.new("TextLabel")
Title.Name = "Title"
Title.Size = UDim2.new(1, 0, 0, 30)
Title.Position = UDim2.new(0, 0, 0, 0)
Title.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Title.Text = "Телепорт к игрокам"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 18
Title.Parent = MainFrame
Title.Active = true
Title.Draggable = true

-- Отображение длительности
local DurationLabel = Instance.new("TextLabel")
DurationLabel.Name = "DurationLabel"
DurationLabel.Size = UDim2.new(1, -20, 0, 30)
DurationLabel.Position = UDim2.new(0, 10, 0, 40)
DurationLabel.BackgroundTransparency = 1
DurationLabel.Text = "Длительность: " .. visitDuration .. "сек"
DurationLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
DurationLabel.Font = Enum.Font.SourceSans
DurationLabel.TextSize = 16
DurationLabel.Parent = MainFrame

-- Кнопка увеличения длительности
local IncreaseButton = Instance.new("TextButton")
IncreaseButton.Name = "IncreaseButton"
IncreaseButton.Size = UDim2.new(0.45, 0, 0, 30)
IncreaseButton.Position = UDim2.new(0.025, 0, 0, 80)
IncreaseButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
IncreaseButton.Text = "+0.2сек"
IncreaseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
IncreaseButton.Font = Enum.Font.SourceSans
IncreaseButton.TextSize = 16
IncreaseButton.Parent = MainFrame

-- Кнопка уменьшения длительности
local DecreaseButton = Instance.new("TextButton")
DecreaseButton.Name = "DecreaseButton"
DecreaseButton.Size = UDim2.new(0.45, 0, 0, 30)
DecreaseButton.Position = UDim2.new(0.525, 0, 0, 80)
DecreaseButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
DecreaseButton.Text = "-0.2сек"
DecreaseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
DecreaseButton.Font = Enum.Font.SourceSans
DecreaseButton.TextSize = 16
DecreaseButton.Parent = MainFrame

-- Поле для ввода ника игрока
local PlayerNameInput = Instance.new("TextBox")
PlayerNameInput.Name = "PlayerNameInput"
PlayerNameInput.Size = UDim2.new(0.6, 0, 0, 30)
PlayerNameInput.Position = UDim2.new(0.025, 0, 0, 120)
PlayerNameInput.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
PlayerNameInput.Text = ""
PlayerNameInput.PlaceholderText = "Введите ник игрока"
PlayerNameInput.TextColor3 = Color3.fromRGB(255, 255, 255)
PlayerNameInput.Font = Enum.Font.SourceSans
PlayerNameInput.TextSize = 16
PlayerNameInput.Parent = MainFrame

-- Кнопка телепорта к указанному игроку
local TeleportToPlayerButton = Instance.new("TextButton")
TeleportToPlayerButton.Name = "TeleportToPlayerButton"
TeleportToPlayerButton.Size = UDim2.new(0.35, 0, 0, 30)
TeleportToPlayerButton.Position = UDim2.new(0.625, 0, 0, 120)
TeleportToPlayerButton.BackgroundColor3 = Color3.fromRGB(80, 80, 120)
TeleportToPlayerButton.Text = "Телепорт"
TeleportToPlayerButton.TextColor3 = Color3.fromRGB(255, 255, 255)
TeleportToPlayerButton.Font = Enum.Font.SourceSans
TeleportToPlayerButton.TextSize = 16
TeleportToPlayerButton.Parent = MainFrame

-- Кнопка следования за выбранным игроком
local FollowSelectedButton = Instance.new("TextButton")
FollowSelectedButton.Name = "FollowSelectedButton"
FollowSelectedButton.Size = UDim2.new(0.95, 0, 0, 30)
FollowSelectedButton.Position = UDim2.new(0.025, 0, 0, 160)
FollowSelectedButton.BackgroundColor3 = Color3.fromRGB(60, 100, 60)
FollowSelectedButton.Text = "Следовать за выбранным: ВЫКЛ"
FollowSelectedButton.TextColor3 = Color3.fromRGB(255, 255, 255)
FollowSelectedButton.Font = Enum.Font.SourceSans
FollowSelectedButton.TextSize = 16
FollowSelectedButton.Parent = MainFrame

-- Кнопка режима следования (автоматического)
local FollowToggle = Instance.new("TextButton")
FollowToggle.Name = "FollowToggle"
FollowToggle.Size = UDim2.new(0.95, 0, 0, 30)
FollowToggle.Position = UDim2.new(0.025, 0, 0, 200)
FollowToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 100)
FollowToggle.Text = "Авто-следование: ВЫКЛ"
FollowToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
FollowToggle.Font = Enum.Font.SourceSans
FollowToggle.TextSize = 16
FollowToggle.Parent = MainFrame

-- Кнопка старта/остановки
local ToggleButton = Instance.new("TextButton")
ToggleButton.Name = "ToggleButton"
ToggleButton.Size = UDim2.new(0.95, 0, 0, 40)
ToggleButton.Position = UDim2.new(0.025, 0, 0, 240)
ToggleButton.BackgroundColor3 = Color3.fromRGB(70, 120, 70)
ToggleButton.Text = "СТАРТ"
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.Font = Enum.Font.SourceSansBold
ToggleButton.TextSize = 18
ToggleButton.Parent = MainFrame

-- Кнопка NoClip
local NoClipButton = Instance.new("TextButton")
NoClipButton.Name = "NoClipButton"
NoClipButton.Size = UDim2.new(0.95, 0, 0, 40)
NoClipButton.Position = UDim2.new(0.025, 0, 0, 290)
NoClipButton.BackgroundColor3 = Color3.fromRGB(115, 152, 255)
NoClipButton.BorderSizePixel = 4
NoClipButton.Text = "NoClip"
NoClipButton.TextColor3 = Color3.fromRGB(0, 0, 0)
NoClipButton.Font = Enum.Font.SourceSansBold
NoClipButton.TextSize = 18
NoClipButton.Parent = MainFrame

-- Подпись под NoClip
local NoClipLabel = Instance.new("TextLabel")
NoClipLabel.Name = "NoClipLabel"
NoClipLabel.Size = UDim2.new(1, -20, 0, 20)
NoClipLabel.Position = UDim2.new(0, 10, 0, 330)
NoClipLabel.BackgroundTransparency = 1
NoClipLabel.Text = "Only works in R15 And has some Bugs"
NoClipLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
NoClipLabel.Font = Enum.Font.SourceSans
NoClipLabel.TextSize = 12
NoClipLabel.Parent = MainFrame

-- Статус
local StatusLabel = Instance.new("TextLabel")
StatusLabel.Name = "StatusLabel"
StatusLabel.Size = UDim2.new(1, -20, 0, 30)
StatusLabel.Position = UDim2.new(0, 10, 0, 350)
StatusLabel.BackgroundTransparency = 1
StatusLabel.Text = "Статус: Готов"
StatusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
StatusLabel.Font = Enum.Font.SourceSans
StatusLabel.TextSize = 14
StatusLabel.Parent = MainFrame

-- Подпись "made by cyku_cyku"
local CreditLabel = Instance.new("TextLabel")
CreditLabel.Name = "CreditLabel"
CreditLabel.Size = UDim2.new(1, -20, 0, 20)
CreditLabel.Position = UDim2.new(0, 10, 0, 370)
CreditLabel.BackgroundTransparency = 1
CreditLabel.Text = "made by cyku_cyku"
CreditLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
CreditLabel.Font = Enum.Font.SourceSans
CreditLabel.TextSize = 12
CreditLabel.TextXAlignment = Enum.TextXAlignment.Right
CreditLabel.Parent = MainFrame

-- Кнопка открытия/закрытия (всегда видна)
local OpenCloseButton = Instance.new("TextButton")
OpenCloseButton.Name = "OpenCloseButton"
OpenCloseButton.Size = UDim2.new(0, 100, 0, 40)
OpenCloseButton.Position = UDim2.new(0, 10, 0, 10)
OpenCloseButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
OpenCloseButton.Text = "ОТКРЫТЬ"
OpenCloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
OpenCloseButton.Font = Enum.Font.SourceSansBold
OpenCloseButton.TextSize = 16
OpenCloseButton.Parent = ScreenGui

-- Функции
local function updateDurationDisplay()
    DurationLabel.Text = "Длительность: " .. string.format("%.1f", visitDuration) .. "сек"
end

local function stopFollowing()
    if followConnection then
        followConnection:Disconnect()
        followConnection = nil
    end
    currentTarget = nil
end

local function startFollowing(player)
    stopFollowing()
    
    if not player or not player.Character then return end
    
    currentTarget = player
    
    followConnection = RunService.Heartbeat:Connect(function()
        if (not manualFollowMode and not isScriptActive) or not currentTarget or not currentTarget.Character or not LocalPlayer.Character then
            stopFollowing()
            return
        end
        
        local targetHRP = currentTarget.Character:FindFirstChild("HumanoidRootPart")
        local localHRP = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        
        if targetHRP and localHRP then
            -- Вычисляем направление от цели к камере
            local camera = workspace.CurrentCamera
            local cameraDirection = (camera.CFrame.Position - targetHRP.Position).Unit
            cameraDirection = Vector3.new(cameraDirection.X, 0, cameraDirection.Z).Unit
            
            -- Вычисляем позицию на заданной дистанции
            local newPosition = targetHRP.Position + (cameraDirection * followDistance)
            -- Сохраняем текущую высоту
            newPosition = Vector3.new(newPosition.X, localHRP.Position.Y, newPosition.Z)
            
            -- Плавно перемещаем персонажа
            localHRP.CFrame = CFrame.new(newPosition, targetHRP.Position)
        end
    end)
end

local function teleportToPlayer(player)
    if not player or not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then
        return false
    end
    
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        local targetHRP = player.Character.HumanoidRootPart
        local localHRP = LocalPlayer.Character.HumanoidRootPart
        
        -- Вычисляем позицию за спиной игрока
        local camera = workspace.CurrentCamera
        local cameraDirection = (camera.CFrame.Position - targetHRP.Position).Unit
        cameraDirection = Vector3.new(cameraDirection.X, 0, cameraDirection.Z).Unit
        
        local newPosition = targetHRP.Position + (cameraDirection * followDistance)
        newPosition = Vector3.new(newPosition.X, localHRP.Position.Y, newPosition.Z)
        
        -- Телепортируемся
        localHRP.CFrame = CFrame.new(newPosition, targetHRP.Position)
        return true
    end
    return false
end

local function getNearestPlayer()
    local nearestPlayer = nil
    local shortestDistance = math.huge
    
    if not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        return nil
    end
    
    local localPosition = LocalPlayer.Character.HumanoidRootPart.Position
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            if player ~= lastTarget then
                local distance = (player.Character.HumanoidRootPart.Position - localPosition).Magnitude
                if distance < shortestDistance then
                    shortestDistance = distance
                    nearestPlayer = player
                end
            end
        end
    end
    
    if not nearestPlayer and lastTarget and lastTarget.Character and lastTarget.Character:FindFirstChild("HumanoidRootPart") then
        nearestPlayer = lastTarget
    end
    
    return nearestPlayer
end

local function findPlayerByName(name)
    name = name:lower()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Name:lower():find(name) then
            return player
        end
    end
    return nil
end

-- Обработчики событий
IncreaseButton.MouseButton1Click:Connect(function()
    visitDuration = visitDuration + 0.2
    if visitDuration > 10 then visitDuration = 10 end
    updateDurationDisplay()
end)

DecreaseButton.MouseButton1Click:Connect(function()
    visitDuration = visitDuration - 0.2
    if visitDuration < 0.2 then visitDuration = 0.2 end
    updateDurationDisplay()
end)

FollowToggle.MouseButton1Click:Connect(function()
    followMode = not followMode
    FollowToggle.Text = "Авто-следование: " .. (followMode and "ВКЛ" or "ВЫКЛ")
    FollowToggle.BackgroundColor3 = followMode and Color3.fromRGB(60, 100, 60) or Color3.fromRGB(60, 60, 100)
    
    if not followMode then
        stopFollowing()
    elseif isScriptActive and currentTarget then
        startFollowing(currentTarget)
    end
end)

FollowSelectedButton.MouseButton1Click:Connect(function()
    manualFollowMode = not manualFollowMode
    FollowSelectedButton.Text = "Следовать за выбранным: " .. (manualFollowMode and "ВКЛ" or "ВЫКЛ")
    FollowSelectedButton.BackgroundColor3 = manualFollowMode and Color3.fromRGB(100, 60, 60) or Color3.fromRGB(60, 100, 60)
    
    if manualFollowMode then
        local playerName = PlayerNameInput.Text
        if playerName ~= "" then
            local targetPlayer = findPlayerByName(playerName)
            if targetPlayer then
                lastTarget = targetPlayer
                startFollowing(targetPlayer)
                StatusLabel.Text = "Статус: Следую за " .. targetPlayer.Name
            else
                StatusLabel.Text = "Статус: Игрок не найден"
                manualFollowMode = false
                FollowSelectedButton.Text = "Следовать за выбранным: ВЫКЛ"
                FollowSelectedButton.BackgroundColor3 = Color3.fromRGB(60, 100, 60)
            end
        else
            StatusLabel.Text = "Статус: Введите ник игрока"
            manualFollowMode = false
            FollowSelectedButton.Text = "Следовать за выбранным: ВЫКЛ"
            FollowSelectedButton.BackgroundColor3 = Color3.fromRGB(60, 100, 60)
        end
    else
        stopFollowing()
        StatusLabel.Text = "Статус: Следование остановлено"
    end
end)

NoClipButton.MouseButton1Down:connect(function()
    game:GetService("Players").LocalPlayer.Character.RightHand.Touched:connect(function(obj)
        if obj ~= workspace.Terrain then
            obj.CanCollide = false
            wait(1)
            obj.CanCollide = true
        end
    end)
end)

OpenCloseButton.MouseButton1Click:Connect(function()
    guiVisible = not guiVisible
    MainFrame.Visible = guiVisible
    OpenCloseButton.Text = guiVisible and "ЗАКРЫТЬ" or "ОТКРЫТЬ"
end)

TeleportToPlayerButton.MouseButton1Click:Connect(function()
    local playerName = PlayerNameInput.Text
    if playerName == "" then return end
    
    local targetPlayer = findPlayerByName(playerName)
    if targetPlayer then
        if teleportToPlayer(targetPlayer) then
            StatusLabel.Text = "Статус: Телепортирован к " .. targetPlayer.Name
            lastTarget = targetPlayer
            if manualFollowMode then
                startFollowing(targetPlayer)
            end
        else
            StatusLabel.Text = "Статус: Ошибка телепорта"
        end
    else
        StatusLabel.Text = "Статус: Игрок не найден"
    end
end)

ToggleButton.MouseButton1Click:Connect(function()
    isScriptActive = not isScriptActive
    
    if isScriptActive then
        ToggleButton.Text = "СТОП"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(120, 70, 70)
        StatusLabel.Text = "Статус: Работает"
        
        spawn(function()
            while isScriptActive do
                local nearestPlayer = getNearestPlayer()
                
                if nearestPlayer then
                    lastTarget = nearestPlayer
                    StatusLabel.Text = "Статус: Посещаю " .. nearestPlayer.Name
                    local success = teleportToPlayer(nearestPlayer)
                    
                    if success then
                        if followMode then
                            startFollowing(nearestPlayer)
                        end
                        
                        local startTime = tick()
                        while isScriptActive and (tick() - startTime) < visitDuration do
                            wait(0.1)
                        end
                    else
                        StatusLabel.Text = "Статус: Ошибка с " .. nearestPlayer.Name
                        wait(1)
                    end
                else
                    StatusLabel.Text = "Статус: Нет игроков рядом"
                    wait(1)
                end
            end
            
            if not isScriptActive then
                StatusLabel.Text = "Статус: Остановлено пользователем"
                if not manualFollowMode then
                    stopFollowing()
                end
            end
        end)
    else
        ToggleButton.Text = "СТАРТ"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(70, 120, 70)
        StatusLabel.Text = "Статус: Готов"
        if not manualFollowMode then
            stopFollowing()
        end
    end
end)

MainFrame.Visible = guiVisible
updateDurationDisplay()
