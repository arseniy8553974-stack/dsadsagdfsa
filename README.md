-- GOD MODE SCRIPT for "Find A Button For Brainrot"
-- Только для asdqwer90095. Все остальные увидят, что ты Бог.
-- Управление: GUI открывается автоматически для тебя.

local player = game.Players.LocalPlayer
local ownerName = "asdqwer90095"  -- Твой ник

if player.Name ~= ownerName then
    -- Не админ – просто показываем предупреждение и выходим
    print("❌ Ты не Бог. Скрипт не активирован.")
    game.StarterGui:SetCore("SendNotification", {
        Title = "Доступ запрещён",
        Text = "Только " .. ownerName .. " может использовать этот скрипт.",
        Duration = 5
    })
    return
end

-- Глобальная таблица для хранения банов (локально на этом сервере)
local bannedPlayers = {}
-- Хранилище для глобальных банов (DataStore – только для этой игры)
local DataStore = game:GetService("DataStoreService"):GetDataStore("GodBans")

-- Загружаем глобальные баны при старте
pcall(function()
    local data = DataStore:GetAsync("globalBans")
    if data then
        for _, userId in ipairs(data) do
            bannedPlayers[userId] = true
        end
    end
end)

-- Функция добавления глобального бана (сохраняется в DataStore)
function addGlobalBan(userId)
    bannedPlayers[userId] = true
    local list = {}
    for id, _ in pairs(bannedPlayers) do
        table.insert(list, id)
    end
    pcall(function()
        DataStore:SetAsync("globalBans", list)
    end)
end

-- При входе игрока проверяем, не забанен ли он глобально
game.Players.PlayerAdded:Connect(function(plr)
    if bannedPlayers[plr.UserId] then
        plr:Kick("❌ Ты забанен Богом навсегда.")
    end
end)

-- ========== КРАСИВОЕ GUI (только для тебя) ==========
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "GodPanel"
screenGui.Parent = player.PlayerGui

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 450, 0, 650)
mainFrame.Position = UDim2.new(0.5, -225, 0.5, -325)
mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 40)
mainFrame.BorderSizePixel = 3
mainFrame.BorderColor3 = Color3.fromRGB(255, 215, 0)
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 40)
title.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
title.Text = "👑 GOD PANEL 👑"
title.TextColor3 = Color3.fromRGB(255, 215, 0)
title.TextScaled = true
title.Font = Enum.Font.GothamBold
title.Parent = mainFrame

local close = Instance.new("TextButton")
close.Size = UDim2.new(0, 30, 0, 30)
close.Position = UDim2.new(1, -35, 0, 5)
close.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
close.Text = "X"
close.TextColor3 = Color3.new(1,1,1)
close.Parent = mainFrame
close.MouseButton1Click:Connect(function() screenGui.Enabled = false end)

local scroll = Instance.new("ScrollingFrame")
scroll.Size = UDim2.new(1, -10, 1, -50)
scroll.Position = UDim2.new(0, 5, 0, 45)
scroll.BackgroundTransparency = 1
scroll.ScrollBarThickness = 8
scroll.CanvasSize = UDim2.new(0, 0, 0, 1000)
scroll.Parent = mainFrame

local function createBtn(text, y, color, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 420, 0, 35)
    btn.Position = UDim2.new(0, 10, 0, y)
    btn.BackgroundColor3 = color or Color3.fromRGB(60, 60, 80)
    btn.Text = text
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Parent = scroll
    btn.MouseButton1Click:Connect(callback)
    return btn
end

local function createInput(y, placeholder)
    local box = Instance.new("TextBox")
    box.Size = UDim2.new(0, 420, 0, 30)
    box.Position = UDim2.new(0, 10, 0, y)
    box.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
    box.PlaceholderText = placeholder
    box.TextColor3 = Color3.new(1,1,1)
    box.Parent = scroll
    return box
end

local y = 10
local targetInput = createInput(y, "Ник игрока")
y = y + 40
local allMode = false
local allBtn = createBtn("🎯 ЦЕЛЬ: ВЫБРАННЫЙ ИГРОК", y, nil, function()
    allMode = not allMode
    allBtn.Text = allMode and "🎯 ЦЕЛЬ: ВСЕ ИГРОКИ" or "🎯 ЦЕЛЬ: ВЫБРАННЫЙ ИГРОК"
end)
y = y + 45
createBtn("💀 УБИТЬ", y, Color3.fromRGB(200, 0, 0), function()
    local targets = allMode and game.Players:GetPlayers() or {game.Players:FindFirstChild(targetInput.Text)}
    for _, plr in ipairs(targets) do
        if plr and plr ~= player and plr.Character and plr.Character:FindFirstChild("Humanoid") then
            plr.Character.Humanoid.Health = 0
            notifyAll("💀 " .. player.Name .. " УБИЛ " .. plr.Name)
        end
    end
end)
y = y + 45
createBtn("👢 КИКНУТЬ", y, Color3.fromRGB(200, 100, 0), function()
    local targets = allMode and game.Players:GetPlayers() or {game.Players:FindFirstChild(targetInput.Text)}
    for _, plr in ipairs(targets) do
        if plr and plr ~= player then
            plr:Kick("Кикнут Богом " .. player.Name)
            notifyAll("👢 " .. player.Name .. " КИКНУЛ " .. plr.Name)
        end
    end
end)
y = y + 45
createBtn("🔨 ЗАБАНИТЬ (ГЛОБАЛЬНО)", y, Color3.fromRGB(150, 0, 150), function()
    local target = game.Players:FindFirstChild(targetInput.Text)
    if target and target ~= player then
        addGlobalBan(target.UserId)
        target:Kick("❌ Ты забанен Богом навсегда!")
        notifyAll("🔨 " .. player.Name .. " ЗАБАНИЛ " .. target.Name .. " ГЛОБАЛЬНО!")
    end
end)
y = y + 45
createBtn("❄️ ЗАМОРОЗИТЬ", y, Color3.fromRGB(0, 100, 200), function()
    local targets = allMode and game.Players:GetPlayers() or {game.Players:FindFirstChild(targetInput.Text)}
    for _, plr in ipairs(targets) do
        if plr and plr ~= player and plr.Character and plr.Character:FindFirstChild("Humanoid") then
            plr.Character.Humanoid.WalkSpeed = 0
            notifyAll("❄️ " .. player.Name .. " ЗАМОРОЗИЛ " .. plr.Name)
        end
    end
end)
y = y + 45
createBtn("🔥 РАЗМОРОЗИТЬ", y, Color3.fromRGB(0, 150, 200), function()
    local targets = allMode and game.Players:GetPlayers() or {game.Players:FindFirstChild(targetInput.Text)}
    for _, plr in ipairs(targets) do
        if plr and plr.Character and plr.Character:FindFirstChild("Humanoid") then
            plr.Character.Humanoid.WalkSpeed = 16
        end
    end
end)
y = y + 45
createBtn("📞 ТЕЛЕПОРТ КО МНЕ", y, Color3.fromRGB(0, 150, 0), function()
    local target = game.Players:FindFirstChild(targetInput.Text)
    if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") and player.Character then
        target.Character.HumanoidRootPart.CFrame = player.Character.HumanoidRootPart.CFrame
        notifyAll("📍 " .. player.Name .. " ТЕЛЕПОРТИРОВАЛ " .. target.Name .. " К СЕБЕ")
    end
end)
y = y + 45
createBtn("🚀 ТЕЛЕПОРТ К НЕМУ", y, Color3.fromRGB(0, 150, 0), function()
    local target = game.Players:FindFirstChild(targetInput.Text)
    if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") and player.Character then
        player.Character.HumanoidRootPart.CFrame = target.Character.HumanoidRootPart.CFrame
    end
end)
y = y + 45
createBtn("⚡ УСКОРИТЬ ВСЕХ (x2)", y, Color3.fromRGB(0, 200, 200), function()
    for _, plr in ipairs(game.Players:GetPlayers()) do
        if plr.Character and plr.Character:FindFirstChild("Humanoid") then
            plr.Character.Humanoid.WalkSpeed = 32
        end
    end
    notifyAll("⚡ " .. player.Name .. " УСКОРИЛ ВСЕХ!")
end)
y = y + 45
createBtn("🐢 ЗАМЕДЛИТЬ ВСЕХ", y, Color3.fromRGB(100, 100, 100), function()
    for _, plr in ipairs(game.Players:GetPlayers()) do
        if plr.Character and plr.Character:FindFirstChild("Humanoid") then
            plr.Character.Humanoid.WalkSpeed = 8
        end
    end
    notifyAll("🐢 " .. player.Name .. " ЗАМЕДЛИЛ ВСЕХ!")
end)
y = y + 45
createBtn("🌞 ДЕНЬ", y, Color3.fromRGB(255, 200, 0), function()
    game:GetService("Lighting").ClockTime = 14
    notifyAll("🌞 " .. player.Name .. " УСТАНОВИЛ ДЕНЬ")
end)
y = y + 45
createBtn("🌙 НОЧЬ", y, Color3.fromRGB(50, 50, 150), function()
    game:GetService("Lighting").ClockTime = 0
    notifyAll("🌙 " .. player.Name .. " УСТАНОВИЛ НОЧЬ")
end)
y = y + 45
createBtn("💥 ВЗОРВАТЬ ВСЕ ОБЪЕКТЫ", y, Color3.fromRGB(200, 50, 0), function()
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("BasePart") and obj.Name ~= "Baseplate" and not obj:IsDescendantOf(player.Character) then
            obj:Destroy()
        end
    end
    notifyAll("💥 " .. player.Name .. " РАЗРУШИЛ КАРТУ!")
end)
y = y + 45
createBtn("✨ СОЗДАТЬ ЗОЛОТОЙ МАЯК (виден всем)", y, Color3.fromRGB(255, 215, 0), function()
    local beacon = Instance.new("Part")
    beacon.Size = Vector3.new(3, 3, 3)
    beacon.Color = Color3.fromRGB(255, 215, 0)
    beacon.Material = Enum.Material.Neon
    beacon.Position = player.Character.HumanoidRootPart.Position + Vector3.new(0, 5, 0)
    beacon.Anchored = true
    beacon.Parent = workspace
    local light = Instance.new("PointLight")
    light.Color = Color3.fromRGB(255, 215, 0)
    light.Range = 30
    light.Parent = beacon
    local particles = Instance.new("ParticleEmitter")
    particles.Texture = "rbxasset://textures/particles/sparkles_main.dds"
    particles.Color = ColorSequence.new(Color3.fromRGB(255, 215, 0))
    particles.Rate = 50
    particles.Parent = beacon
    notifyAll("✨ " .. player.Name .. " СОЗДАЛ ЗОЛОТОЙ МАЯК!")
end)
y = y + 45
createBtn("🛡️ ВСЕМ БЕССМЕРТИЕ", y, Color3.fromRGB(0, 200, 0), function()
    for _, plr in ipairs(game.Players:GetPlayers()) do
        if plr.Character and plr.Character:FindFirstChild("Humanoid") then
            plr.Character.Humanoid.MaxHealth = 1e9
            plr.Character.Humanoid.Health = 1e9
        end
    end
    notifyAll("🛡️ " .. player.Name .. " ДАЛ ВСЕМ БЕССМЕРТИЕ!")
end)
y = y + 45
createBtn("❌ ОТКЛЮЧИТЬ БЕССМЕРТИЕ", y, Color3.fromRGB(200, 0, 0), function()
    for _, plr in ipairs(game.Players:GetPlayers()) do
        if plr.Character and plr.Character:FindFirstChild("Humanoid") then
            plr.Character.Humanoid.MaxHealth = 100
        end
    end
    notifyAll("❌ " .. player.Name .. " ОТКЛЮЧИЛ БЕССМЕРТИЕ")
end)

scroll.CanvasSize = UDim2.new(0, 0, 0, y + 50)

-- ========== ФУНКЦИЯ ОПОВЕЩЕНИЯ ВСЕХ ИГРОКОВ ==========
function notifyAll(msg)
    for _, plr in ipairs(game.Players:GetPlayers()) do
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "🔔 БОГ",
            Text = msg,
            Duration = 4
        })
    end
end

-- ========== ПОЛЁТ ДЛЯ ТЕБЯ ==========
local flyEnabled = false
local flyBodyVelocity, flyBodyGyro
local flyConnections = {}
game:GetService("UserInputService").InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.F then
        flyEnabled = not flyEnabled
        if flyEnabled then
            local char = player.Character
            if not char then return end
            local root = char:FindFirstChild("HumanoidRootPart")
            local hum = char:FindFirstChild("Humanoid")
            if not root or not hum then return end
            flyBodyVelocity = Instance.new("BodyVelocity")
            flyBodyVelocity.MaxForce = Vector3.new(1e6, 1e6, 1e6)
            flyBodyVelocity.Parent = root
            flyBodyGyro = Instance.new("BodyGyro")
            flyBodyGyro.MaxTorque = Vector3.new(1e6, 1e6, 1e6)
            flyBodyGyro.Parent = root
            hum.PlatformStand = true
            
            local moveDir = Vector3.new()
            local function update(key, state, vec)
                if state then moveDir = moveDir + vec else moveDir = moveDir - vec end
            end
            
            local iB = game:GetService("UserInputService").InputBegan:Connect(function(i)
                local k = i.KeyCode.Name
                if k == "W" then update("W", true, Vector3.new(0,0,-1))
                elseif k == "S" then update("S", true, Vector3.new(0,0,1))
                elseif k == "A" then update("A", true, Vector3.new(-1,0,0))
                elseif k == "D" then update("D", true, Vector3.new(1,0,0))
                elseif k == "Space" then update("Space", true, Vector3.new(0,1,0))
                elseif k == "LeftShift" then update("Shift", true, Vector3.new(0,-1,0)) end
            end)
            local iE = game:GetService("UserInputService").InputEnded:Connect(function(i)
                local k = i.KeyCode.Name
                if k == "W" then update("W", false, Vector3.new(0,0,-1))
                elseif k == "S" then update("S", false, Vector3.new(0,0,1))
                elseif k == "A" then update("A", false, Vector3.new(-1,0,0))
                elseif k == "D" then update("D", false, Vector3.new(1,0,0))
                elseif k == "Space" then update("Space", false, Vector3.new(0,1,0))
                elseif k == "LeftShift" then update("Shift", false, Vector3.new(0,-1,0)) end
            end)
            local rs = RunService.RenderStepped:Connect(function()
                if not flyEnabled or not root then
                    rs:Disconnect(); iB:Disconnect(); iE:Disconnect()
                    return
                end
                flyBodyVelocity.Velocity = moveDir * 60
                if moveDir.Magnitude > 0 then
                    flyBodyGyro.CFrame = CFrame.new(root.Position, root.Position + moveDir)
                end
            end)
            flyConnections = {rs, iB, iE}
        else
            if flyBodyVelocity then flyBodyVelocity:Destroy() end
            if flyBodyGyro then flyBodyGyro:Destroy() end
            for _, conn in ipairs(flyConnections) do conn:Disconnect() end
            if player.Character and player.Character:FindFirstChild("Humanoid") then
                player.Character.Humanoid.PlatformStand = false
            end
        end
    end
end)

-- ========== НАЧАЛЬНОЕ СООБЩЕНИЕ ==========
notifyAll("👑 " .. player.Name .. " ВОШЁЛ В ИГРУ КАК БОГ! 👑")
print("✅ Ты – Бог! Нажми F для полёта. Панель управления открыта.")
