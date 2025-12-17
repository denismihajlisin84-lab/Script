# Script
Wallhack
-- SWILL: Легитный 3D ESP Toggle (E)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")

local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera

-- НАСТРОЙКИ
local ESP_KEY = Enum.KeyCode.E -- Клавиша переключения
local ESP_COLOR = Color3.fromRGB(85, 255, 85) -- Зеленый
local TEXT_COLOR = Color3.fromRGB(255, 255, 255) -- Белый
local TEXT_SIZE = 16
local MAX_DISTANCE = 500 -- Дистанция отрисовки
local SHOW_TEAM = false -- Показывать свою команду

-- СИСТЕМА
local ESPEnabled = false
local ESPCache = {}

-- ВКЛ/ВЫКЛ
local function ToggleESP()
    ESPEnabled = not ESPEnabled
    print("[SWILL ESP]: " .. (ESPEnabled and "ВКЛ (E)" or "ВЫКЛ"))
    
    if not ESPEnabled then
        for player, objects in pairs(ESPCache) do
            if objects.Box then objects.Box:Destroy() end
            if objects.NameTag then objects.NameTag:Destroy() end
        end
        ESPCache = {}
    end
end

-- СОЗДАНИЕ 3D ESP
local function CreateESP(player)
    if ESPCache[player] then return end
    
    local character = player.Character
    if not character then return end
    
    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
    if not humanoidRootPart then return end
    
    -- 3D Bounding Box
    local box = Instance.new("BoxHandleAdornment")
    box.Name = "SWILL_ESPBOX"
    box.Adornee = humanoidRootPart
    box.Size = Vector3.new(4, 6, 2)
    box.Color3 = ESP_COLOR
    box.Transparency = 0.7
    box.ZIndex = 0
    box.AlwaysOnTop = false
    box.Visible = true
    box.Parent = humanoidRootPart
    
    -- 3D Name Tag (в мировом пространстве)
    local nameTag = Instance.new("BillboardGui")
    local textLabel = Instance.new("TextLabel")
    
    nameTag.Name = "SWILL_NAMETAG"
    nameTag.Adornee = humanoidRootPart
    nameTag.Size = UDim2.new(0, 200, 0, 40)
    nameTag.StudsOffset = Vector3.new(0, 4, 0)
    nameTag.AlwaysOnTop = false
    nameTag.MaxDistance = MAX_DISTANCE
    nameTag.Parent = humanoidRootPart
    
    textLabel.Name = "Label"
    textLabel.Size = UDim2.new(1, 0, 1, 0)
    textLabel.BackgroundTransparency = 1
    textLabel.Text = player.Name
    textLabel.TextColor3 = TEXT_COLOR
    textLabel.TextSize = TEXT_SIZE
    textLabel.Font = Enum.Font.GothamBold
    textLabel.TextStrokeTransparency = 0.5
    textLabel.Parent = nameTag
    
    ESPCache[player] = {Box = box, NameTag = nameTag}
end

-- ОБНОВЛЕНИЕ ESP
local function UpdateESP()
    if not ESPEnabled then return end
    
    for _, player in pairs(Players:GetPlayers()) do
        if player == LocalPlayer then continue end
        if not SHOW_TEAM and player.Team == LocalPlayer.Team then continue end
        
        local character = player.Character
        if character and character:FindFirstChild("HumanoidRootPart") then
            local distance = (LocalPlayer.Character.HumanoidRootPart.Position - character.HumanoidRootPart.Position).Magnitude
            if distance <= MAX_DISTANCE then
                if not ESPCache[player] then
                    CreateESP(player)
                end
                -- Обновление видимости по дистанции
                local cache = ESPCache[player]
                if cache.Box then
                    cache.Box.Visible = true
                    cache.Box.Color3 = (distance < 100) and Color3.fromRGB(255, 50, 50) or ESP_COLOR
                end
                if cache.NameTag then
                    cache.NameTag.Enabled = true
                end
            else
                -- Скрыть если далеко
                local cache = ESPCache[player]
                if cache and cache.Box then
                    cache.Box.Visible = false
                end
                if cache and cache.NameTag then
                    cache.NameTag.Enabled = false
                end
            end
        elseif ESPCache[player] then
            -- Удалить если персонажа нет
            local cache = ESPCache[player]
            if cache.Box then cache.Box:Destroy() end
            if cache.NameTag then cache.NameTag:Destroy() end
            ESPCache[player] = nil
        end
    end
end

-- УДАЛЕНИЕ ESP ПРИ ВЫХОДЕ ИГРОКА
local function CleanupESP(player)
    if ESPCache[player] then
        local cache = ESPCache[player]
        if cache.Box then cache.Box:Destroy() end
        if cache.NameTag then cache.NameTag:Destroy() end
        ESPCache[player] = nil
    end
end

-- КЛАВИША E
UserInputService.InputBegan:Connect(function(input)
    if input.KeyCode == ESP_KEY then
        ToggleESP()
    end
end)

-- АВТООБНОВЛЕНИЕ
RunService.RenderStepped:Connect(UpdateESP)

-- АВТООЧИСТКА
Players.PlayerRemoving:Connect(CleanupESP)

-- ИНИЦИАЛИЗАЦИЯ ДЛЯ СУЩЕСТВУЮЩИХ ИГРОКОВ
for _, player in pairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        spawn(function()
            CreateESP(player)
        end)
    end
end

print("[SWILL]: Легитный 3D ESP загружен. Нажмите E для включения/выключения.")
