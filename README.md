--[[
    Blox Fruits - Sea Events, Farm & Itens Script
    Desenvolvido para Delta Executor (Mobile)
    Sistema Anti-Ban 100% integrado
    CDK, TTK e Soul Guitar incluídos
]]

-- ============================================
-- SISTEMA ANTI-BAN 100%
-- ============================================

local antiBan = {
    enabled = true,
    bypassMethods = {"identity", "behavior", "memory", "injection"}
}

if syn then
    syn.set_readonly(syn, true)
    syn.set_thread_identity(8)
end

local function antiMemoryScan()
    local success, err = pcall(function()
        local mt = getrawmetatable(game)
        local old = mt.__namecall
        setreadonly(mt, false)
        
        mt.__namecall = newcclosure(function(self, ...)
            local method = getnamecallmethod()
            local args = {...}
            
            if method == "FindService" or method == "GetService" then
                if args[1] and type(args[1]) == "string" then
                    local blockedServices = {
                        "ScriptContext", "ScriptService", "LogService",
                        "HttpService", "MarketplaceService"
                    }
                    for _, blocked in pairs(blockedServices) do
                        if args[1]:lower():find(blocked:lower()) then
                            return nil
                        end
                    end
                end
            end
            
            if method == "FireServer" or method == "InvokeServer" then
                local caller = debug.info(2, "f")
                if caller and caller:lower():find("anticheat") then
                    return nil
                end
            end
            
            return old(self, ...)
        end)
        
        setreadonly(mt, true)
    end)
end

local function antiIdentityDetection()
    if setidentity then
        setidentity(8)
    end
    
    if getgenv then
        getgenv()._G = nil
    end
end

local function antiBehaviorDetection()
    local humanPattern = {
        clickDelays = {0.08, 0.12, 0.15, 0.1, 0.13, 0.09, 0.11},
        movementVariation = 0.5
    }
    
    task.spawn(function()
        while antiBan.enabled do
            local randomDelay = humanPattern.clickDelays[math.random(1, #humanPattern.clickDelays)]
            
            if rootPart and humanoid and humanoid.Health > 0 then
                local currentCFrame = rootPart.CFrame
                local microMovement = Vector3.new(
                    math.random(-2, 2) * humanPattern.movementVariation,
                    math.random(-1, 1) * humanPattern.movementVariation,
                    math.random(-2, 2) * humanPattern.movementVariation
                )
                rootPart.CFrame = currentCFrame * CFrame.new(microMovement)
            end
            
            task.wait(randomDelay)
        end
    end)
end

local function antiNetworkDetection()
    if syn and syn.request then
        local oldRequest = syn.request
        syn.request = function(options)
            if options.Url then
                if options.Url:find("roblox.com/asset") then
                    options.Url = options.Url:gsub("roblox.com/asset", "rbxassetid://")
                end
                
                if options.Headers then
                    options.Headers["User-Agent"] = "Roblox/WinInet"
                    options.Headers["RBX-Exploit-GUID"] = nil
                    options.Headers["RBX-Exploit-Level"] = nil
                end
            end
            return oldRequest(options)
        end
    end
end

local function antiLoggerDetection()
    if hookfunction then
        local oldWarn = warn
        warn = function(...) end
        
        local oldPrint = print
        print = function(...) end
        
        if getgenv().DEBUG_MODE then
            warn = oldWarn
            print = oldPrint
        end
    end
end

local function initAntiBan()
    if not antiBan.enabled then return end
    
    task.spawn(function()
        pcall(antiMemoryScan)
        pcall(antiIdentityDetection)
        pcall(antiBehaviorDetection)
        pcall(antiNetworkDetection)
        pcall(antiLoggerDetection)
        
        print("🛡️ Sistema Anti-Ban 100% ativado!")
    end)
end

-- ============================================
-- VARIÁVEIS PRINCIPAIS
-- ============================================

local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

local players = game:GetService("Players")
local runService = game:GetService("RunService")
local tweenService = game:GetService("TweenService")
local virtualInput = game:GetService("VirtualInputManager")
local replicatedStorage = game:GetService("ReplicatedStorage")

player.CharacterAdded:Connect(function(newCharacter)
    character = newCharacter
    humanoid = character:WaitForChild("Humanoid")
    rootPart = character:WaitForChild("HumanoidRootPart")
end)

-- Configurações
local settings = {
    currentSeaEvent = nil,
    seaEvents = {
        terrorShark = {
            enabled = false,
            name = "Terror Shark",
            safeHeight = 150,
            zones = {
                Sea5 = Vector3.new(-1950, 20, -1950),
                Sea6 = Vector3.new(2000, 20, 2000)
            },
            spawnKeyword = "terrorshark"
        },
        seaBeast = {
            enabled = false,
            name = "Sea Beast",
            safeHeight = 100,
            zones = {
                Sea5 = Vector3.new(-2000, 20, -2000),
                Sea6 = Vector3.new(2100, 20, 2100)
            },
            spawnKeyword = "seabeast"
        },
        piranha = {
            enabled = false,
            name = "Piranha",
            safeHeight = 80,
            zones = {
                Sea5 = Vector3.new(-1900, 20, -1900),
                Sea6 = Vector3.new(1900, 20, 1900)
            },
            spawnKeyword = "piranha"
        },
        tubarao = {
            enabled = false,
            name = "Tubarão",
            safeHeight = 120,
            zones = {
                Sea5 = Vector3.new(-2100, 20, -2100),
                Sea6 = Vector3.new(2200, 20, 2200)
            },
            spawnKeyword = "shark"
        },
        barcoAssombrado = {
            enabled = false,
            name = "Barco Assombrado",
            safeHeight = 130,
            zones = {
                Sea5 = Vector3.new(-1800, 20, -1800),
                Sea6 = Vector3.new(1800, 20, 1800)
            },
            spawnKeyword = "haunted"
        }
    },
    autoFarm = {
        enabled = false,
        safeHeight = 50
    },
    itens = {
        cdk = {
            enabled = false,
            name = "Curse Dual Katana",
            requirements = {
                level = 2200,
                items = {"Alucard Shard", "Dark Fragment", "Bone", "Scrap Metal"}
            },
            questSteps = {
                {npc = "Master of Auras", location = Vector3.new(-11000, 20, -8000)},
                {npc = "Tushita Puzzle", location = Vector3.new(-9500, 20, -7000)},
                {npc = "Yama Puzzle", location = Vector3.new(-10000, 20, -7500)}
            }
        },
        ttk = {
            enabled = false,
            name = "Triple Katana",
            requirements = {
                level = 1500,
                items = {"Katana", "Wando", "Saddi"}
            },
            questSteps = {
                {npc = "Sword Dealer", location = Vector3.new(-7000, 20, -5000)},
                {npc = "Legendary Sword Dealer", location = Vector3.new(-7500, 20, -5500)}
            }
        },
        soulGuitar = {
            enabled = false,
            name = "Soul Guitar",
            requirements = {
                level = 2300,
                items = {"Dark Fragment", "Soul", "Bone"}
            },
            questSteps = {
                {npc = "Weapons Dealer", location = Vector3.new(-12000, 20, -9000)},
                {npc = "Soul Reaper", location = Vector3.new(-11500, 20, -8500)}
            }
        }
    }
}

-- ============================================
-- UI PRINCIPAL
-- ============================================

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "ProtectedUI_" .. math.random(100000, 999999)
screenGui.Parent = game.CoreGui
screenGui.ResetOnSpawn = false

if syn then
    syn.protect_gui(screenGui)
end

-- ============================================
-- BOTÃO FANTASMA
-- ============================================

local ghostButton = Instance.new("ImageButton")
ghostButton.Name = "GhostButton"
ghostButton.Size = UDim2.new(0, 45, 0, 45)
ghostButton.Position = UDim2.new(0, 10, 0, 10)
ghostButton.BackgroundTransparency = 1
ghostButton.Image = "rbxassetid://10860981523"
ghostButton.ImageColor3 = Color3.fromRGB(0, 255, 255)
ghostButton.ScaleType = Enum.ScaleType.Fit
ghostButton.Parent = screenGui

local ghostStroke = Instance.new("UIStroke")
ghostStroke.Color = Color3.fromRGB(0, 255, 255)
ghostStroke.Thickness = 2
ghostStroke.Transparency = 0.3
ghostStroke.Parent = ghostButton

local ghostCorner = Instance.new("UICorner")
ghostCorner.CornerRadius = UDim.new(1, 0)
ghostCorner.Parent = ghostButton

local ghostBackground = Instance.new("Frame")
ghostBackground.Size = UDim2.new(1, 4, 1, 4)
ghostBackground.Position = UDim2.new(0, -2, 0, -2)
ghostBackground.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
ghostBackground.BackgroundTransparency = 0.3
ghostBackground.Parent = ghostButton

local ghostBgCorner = Instance.new("UICorner")
ghostBgCorner.CornerRadius = UDim.new(1, 0)
ghostBgCorner.Parent = ghostBackground

ghostBackground.ZIndex = 0
ghostButton.ZIndex = 1

-- ============================================
-- MAIN FRAME
-- ============================================

local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame_Protected"
mainFrame.Size = UDim2.new(0, 300, 0, 450)
mainFrame.Position = UDim2.new(0.5, -150, 0.5, -225)
mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
mainFrame.BackgroundTransparency = 0.1
mainFrame.BorderSizePixel = 0
mainFrame.ClipsDescendants = true
mainFrame.Visible = false
mainFrame.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 8)
corner.Parent = mainFrame

local stroke = Instance.new("UIStroke")
stroke.Color = Color3.fromRGB(0, 255, 255)
stroke.Thickness = 1.5
stroke.Transparency = 0.3
stroke.Parent = mainFrame

-- Título
local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, 0, 0, 40)
titleLabel.Position = UDim2.new(0, 0, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "👻 BLOX FRUITS NEON"
titleLabel.TextColor3 = Color3.fromRGB(0, 255, 255)
titleLabel.TextSize = 20
titleLabel.Font = Enum.Font.GothamBold
titleLabel.Parent = mainFrame

-- Status Anti-Ban
local antiBanStatus = Instance.new("TextLabel")
antiBanStatus.Size = UDim2.new(0, 100, 0, 15)
antiBanStatus.Position = UDim2.new(1, -140, 0, 5)
antiBanStatus.BackgroundTransparency = 1
antiBanStatus.Text = "🛡️ ANTI-BAN"
antiBanStatus.TextColor3 = Color3.fromRGB(0, 255, 0)
antiBanStatus.TextSize = 10
antiBanStatus.Font = Enum.Font.GothamBold
antiBanStatus.Parent = mainFrame

-- Botão fechar
local closeButton = Instance.new("TextButton")
closeButton.Size = UDim2.new(0, 30, 0, 30)
closeButton.Position = UDim2.new(1, -35, 0, 5)
closeButton.BackgroundTransparency = 1
closeButton.Text = "✕"
closeButton.TextColor3 = Color3.fromRGB(255, 50, 50)
closeButton.TextSize = 20
closeButton.Font = Enum.Font.GothamBold
closeButton.Parent = mainFrame

closeButton.MouseButton1Click:Connect(function()
    mainFrame.Visible = false
end)

-- Separador
local titleSeparator = Instance.new("Frame")
titleSeparator.Size = UDim2.new(0.9, 0, 0, 1)
titleSeparator.Position = UDim2.new(0.05, 0, 0, 40)
titleSeparator.BackgroundColor3 = Color3.fromRGB(0, 255, 255)
titleSeparator.BackgroundTransparency = 0.5
titleSeparator.Parent = mainFrame

-- Container abas
local tabContainer = Instance.new("Frame")
tabContainer.Size = UDim2.new(1, 0, 0, 35)
tabContainer.Position = UDim2.new(0, 0, 0, 45)
tabContainer.BackgroundTransparency = 1
tabContainer.Parent = mainFrame

-- Aba Sea Eventos
local seaTab = Instance.new("TextButton")
seaTab.Size = UDim2.new(0.31, 0, 1, 0)
seaTab.Position = UDim2.new(0.01, 0, 0, 0)
seaTab.Text = "🌊 Sea"
seaTab.BackgroundColor3 = Color3.fromRGB(0, 255, 255)
seaTab.BackgroundTransparency = 0.85
seaTab.TextColor3 = Color3.fromRGB(255, 255, 255)
seaTab.TextSize = 11
seaTab.Font = Enum.Font.GothamBold
seaTab.Parent = tabContainer

-- Aba Farmes
local farmTab = Instance.new("TextButton")
farmTab.Size = UDim2.new(0.31, 0, 1, 0)
farmTab.Position = UDim2.new(0.33, 0, 0, 0)
farmTab.Text = "⚔️ Farm"
farmTab.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
farmTab.BackgroundTransparency = 0.85
farmTab.TextColor3 = Color3.fromRGB(255, 255, 255)
farmTab.TextSize = 11
farmTab.Font = Enum.Font.GothamBold
farmTab.Parent = tabContainer

-- Aba Itens
local itensTab = Instance.new("TextButton")
itensTab.Size = UDim2.new(0.31, 0, 1, 0)
itensTab.Position = UDim2.new(0.65, 0, 0, 0)
itensTab.Text = "💎 Itens"
itensTab.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
itensTab.BackgroundTransparency = 0.85
itensTab.TextColor3 = Color3.fromRGB(255, 255, 255)
itensTab.TextSize = 11
itensTab.Font = Enum.Font.GothamBold
itensTab.Parent = tabContainer

local function addNeonEffect(button)
    local neon = Instance.new("UIStroke")
    neon.Color = Color3.fromRGB(0, 255, 255)
    neon.Thickness = 1
    neon.Transparency = 0.7
    neon.Parent = button
    return neon
end

local seaNeon = addNeonEffect(seaTab)
local farmNeon = addNeonEffect(farmTab)
local itensNeon = addNeonEffect(itensTab)

-- Conteúdo das abas
local seaContent = Instance.new("ScrollingFrame")
seaContent.Size = UDim2.new(1, 0, 1, -85)
seaContent.Position = UDim2.new(0, 0, 0, 85)
seaContent.BackgroundTransparency = 1
seaContent.ScrollBarThickness = 2
seaContent.ScrollBarImageColor3 = Color3.fromRGB(0, 255, 255)
seaContent.Visible = true
seaContent.Parent = mainFrame

local farmContent = Instance.new("ScrollingFrame")
farmContent.Size = UDim2.new(1, 0, 1, -85)
farmContent.Position = UDim2.new(0, 0, 0, 85)
farmContent.BackgroundTransparency = 1
farmContent.ScrollBarThickness = 2
farmContent.ScrollBarImageColor3 = Color3.fromRGB(0, 255, 255)
farmContent.Visible = false
farmContent.Parent = mainFrame

local itensContent = Instance.new("ScrollingFrame")
itensContent.Size = UDim2.new(1, 0, 1, -85)
itensContent.Position = UDim2.new(0, 0, 0, 85)
itensContent.BackgroundTransparency = 1
itensContent.ScrollBarThickness = 2
itensContent.ScrollBarImageColor3 = Color3.fromRGB(0, 255, 255)
itensContent.Visible = false
itensContent.Parent = mainFrame

local function setupListLayout(parent)
    local layout = Instance.new("UIListLayout")
    layout.Padding = UDim.new(0, 5)
    layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    layout.SortOrder = Enum.SortOrder.LayoutOrder
    layout.Parent = parent
    return layout
end

setupListLayout(seaContent)
setupListLayout(farmContent)
setupListLayout(itensContent)

-- ============================================
-- BOTÕES PERSONALIZADOS
-- ============================================

local function createButton(parent, text, callback)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0.95, 0, 0, 40)
    button.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    button.BackgroundTransparency = 0.2
    button.Text = text
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextSize = 12
    button.Font = Enum.Font.GothamBold
    button.Parent = parent
    
    local neon = Instance.new("UIStroke")
    neon.Color = Color3.fromRGB(0, 255, 255)
    neon.Thickness = 1
    neon.Transparency = 0.5
    neon.Parent = button
    
    button.MouseButton1Click:Connect(callback)
    
    return button
end

local seaEventButtons = {}

local function createSeaEventButton(parent, eventKey, eventName, emoji)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0.95, 0, 0, 40)
    button.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    button.BackgroundTransparency = 0.2
    button.Text = emoji .. " " .. eventName .. " ⬜"
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextSize = 12
    button.Font = Enum.Font.GothamBold
    button.Parent = parent
    
    local neon = Instance.new("UIStroke")
    neon.Color = Color3.fromRGB(0, 255, 255)
    neon.Thickness = 1
    neon.Transparency = 0.5
    neon.Parent = button
    
    local function updateButtonState()
        if settings.seaEvents[eventKey].enabled then
            button.Text = emoji .. " " .. eventName .. " ✅"
            button.BackgroundColor3 = Color3.fromRGB(0, 255, 100)
            neon.Color = Color3.fromRGB(0, 255, 0)
        else
            button.Text = emoji .. " " .. eventName .. " ⬜"
            button.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
            neon.Color = Color3.fromRGB(0, 255, 255)
        end
    end
    
    button.MouseButton1Click:Connect(function()
        for key, event in pairs(settings.seaEvents) do
            if key ~= eventKey then
                event.enabled = false
            end
        end
        
        settings.seaEvents[eventKey].enabled = not settings.seaEvents[eventKey].enabled
        
        for _, btn in pairs(seaEventButtons) do
            btn.updateState()
        end
        
        if settings.seaEvents[eventKey].enabled then
            settings.currentSeaEvent = eventKey
            startSeaEvent(eventKey)
        else
            settings.currentSeaEvent = nil
            stopSeaEvent()
        end
    end)
    
    seaEventButtons[eventKey] = {
        button = button,
        updateState = updateButtonState
    }
    
    return button
end

local itemButtons = {}

local function createItemButton(parent, itemKey, itemName, emoji)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0.95, 0, 0, 55)
    button.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    button.BackgroundTransparency = 0.2
    button.Text = emoji .. " " .. itemName .. "\n⬜ Aguardando..."
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextSize = 11
    button.Font = Enum.Font.GothamBold
    button.Parent = parent
    
    local neon = Instance.new("UIStroke")
    neon.Color = Color3.fromRGB(0, 255, 255)
    neon.Thickness = 1
    neon.Transparency = 0.5
    neon.Parent = button
    
    local function updateButtonState()
        if settings.itens[itemKey].enabled then
            button.Text = emoji .. " " .. itemName .. "\n🔄 Pegando..."
            button.BackgroundColor3 = Color3.fromRGB(0, 255, 100)
            neon.Color = Color3.fromRGB(0, 255, 0)
        else
            button.Text = emoji .. " " .. itemName .. "\n⬜ Aguardando..."
            button.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
            neon.Color = Color3.fromRGB(0, 255, 255)
        end
    end
    
    button.MouseButton1Click:Connect(function()
        settings.itens[itemKey].enabled = not settings.itens[itemKey].enabled
        
        for _, btn in pairs(itemButtons) do
            btn.updateState()
        end
        
        if settings.itens[itemKey].enabled then
            startItemQuest(itemKey)
        else
            stopItemQuest(itemKey)
        end
    end)
    
    itemButtons[itemKey] = {
        button = button,
        updateState = updateButtonState
    }
    
    return button
end

-- ============================================
-- SISTEMA DE SEA EVENTS
-- ============================================

local seaEventCoroutine = nil

local function findPlayerBoat()
    for _, boat in pairs(workspace:GetDescendants()) do
        if boat:IsA("Model") and boat:FindFirstChild("VehicleSeat") then
            local seat = boat.VehicleSeat
            if seat.Occupant and seat.Occupant.Parent == character then
                return boat
            end
        end
    end
    return nil
end

local function getAvailableBoat()
    for _, boat in pairs(workspace:GetDescendants()) do
        if boat:IsA("Model") and boat:FindFirstChild("VehicleSeat") then
            local seat = boat.VehicleSeat
            if not seat.Occupant then
                local boatPos = boat:GetPivot().Position
                local playerPos = rootPart.Position
                if (boatPos - playerPos).Magnitude < 100 then
                    return boat
                end
            end
        end
    end
    
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("SpawnLocation") and obj.Name:lower():find("boat") then
            return obj
        end
    end
    
    return nil
end

local function enterBoat(boat)
    if not boat or not boat:FindFirstChild("VehicleSeat") then return false end
    
    local seat = boat.VehicleSeat
    if seat.Occupant then return false end
    
    local boatPos = boat:GetPivot().Position
    safeTeleport(boatPos + Vector3.new(0, 5, 0), 0.3)
    
    task.wait(0.5)
    
    fireproximityprompt(seat)
    task.wait(0.5)
    
    if seat.Occupant and seat.Occupant.Parent == character then
        return true
    end
    
    return false
end

local function navigateBoat(targetPosition)
    local boat = findPlayerBoat()
    if not boat then return false end
    
    local boatRoot = boat:FindFirstChild("VehicleSeat") or boat.PrimaryPart
    if not boatRoot then return false end
    
    local currentPos = boatRoot.Position
    local direction = (targetPosition - currentPos).Unit
    
    local moveDistance = 50
    local newPos = currentPos + (direction * moveDistance)
    
    boat:SetPrimaryPartCFrame(CFrame.new(newPos))
    
    return true
end

local function startSeaEvent(eventKey)
    if seaEventCoroutine then
        task.cancel(seaEventCoroutine)
    end
    
    local eventConfig = settings.seaEvents[eventKey]
    if not eventConfig then return end
    
    eventConfig.enabled = true
    
    seaEventCoroutine = task.spawn(function()
        print("🚢 Iniciando hunt: " .. eventConfig.name)
        
        local boat = getAvailableBoat()
        if not boat then
            print("❌ Nenhum barco encontrado!")
            return
        end
        
        if not enterBoat(boat) then
            print("❌ Falha ao entrar no barco!")
            return
        end
        
        print("✅ Barco adquirido!")
        
        local zone = eventConfig.zones.Sea6 or eventConfig.zones.Sea5
        local searchRadius = 200
        local currentSearchPos = zone
        
        while eventConfig.enabled do
            local currentBoat = findPlayerBoat()
            if not currentBoat then
                local newBoat = getAvailableBoat()
                if newBoat then
                    enterBoat(newBoat)
                else
                    print("❌ Perdeu o barco!")
                    break
                end
            end
            
            local searchPattern = {
                currentSearchPos + Vector3.new(math.random(-searchRadius, searchRadius), 0, 0),
                currentSearchPos + Vector3.new(0, 0, math.random(-searchRadius, searchRadius)),
                currentSearchPos + Vector3.new(math.random(-searchRadius, searchRadius), 0, math.random(-searchRadius, searchRadius)),
            }
            
            for _, targetPos in pairs(searchPattern) do
                if not eventConfig.enabled then break end
                
                navigateBoat(targetPos)
                task.wait(1)
                
                local boss = findSeaBoss(eventConfig.spawnKeyword)
                
                if boss then
                    print("🎯 " .. eventConfig.name .. " encontrado!")
                    
                    local seat = currentBoat:FindFirstChild("VehicleSeat")
                    if seat then
                        seat:Sit()
                    end
                    
                    task.wait(0.5)
                    
                    local bossPos = boss:GetPivot().Position
                    local safePos = Vector3.new(
                        bossPos.X + math.random(-2, 2),
                        bossPos.Y + eventConfig.safeHeight,
                        bossPos.Z + math.random(-2, 2)
                    )
                    
                    safeTeleport(safePos, 0.3)
                    
                    local positionLockCoroutine = task.spawn(function()
                        while eventConfig.enabled and boss and boss.Parent do
                            local hum = boss:FindFirstChild("Humanoid")
                            if hum and hum.Health > 0 then
                                local currentBossPos = boss:GetPivot().Position
                                rootPart.CFrame = CFrame.new(Vector3.new(
                                    currentBossPos.X + math.random(-1, 1),
                                    currentBossPos.Y + eventConfig.safeHeight,
                                    currentBossPos.Z + math.random(-1, 1)
                                ))
                            else
                                break
                            end
                            task.wait(math.random(1, 3) / 10)
                        end
                    end)
                    
                    startAutoAttack()
                    
                    while eventConfig.enabled do
                        task.wait(1)
                        if not boss or not boss.Parent then
                            print("✅ " .. eventConfig.name .. " derrotado!")
                            break
                        end
                        local hum = boss:FindFirstChild("Humanoid")
                        if hum and hum.Health <= 0 then
                            print("✅ " .. eventConfig.name .. " derrotado!")
                            break
                        end
                    end
                    
                    if positionLockCoroutine then
                        task.cancel(positionLockCoroutine)
                    end
                    
                    break
                end
                
                task.wait(0.5)
            end
            
            task.wait(2)
        end
        
        stopAutoAttack()
        print("🏁 Hunt finalizado: " .. eventConfig.name)
    end)
end

local function findSeaBoss(keyword)
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and obj.Name:lower():find(keyword:lower()) and obj:FindFirstChild("Humanoid") then
            local hum = obj.Humanoid
            if hum.Health > 0 then
                return obj
            end
        end
    end
    return nil
end

local function stopSeaEvent()
    if seaEventCoroutine then
        task.cancel(seaEventCoroutine)
        seaEventCoroutine = nil
    end
    
    stopAutoAttack()
    
    for key, event in pairs(settings.seaEvents) do
        event.enabled = false
    end
    
    settings.currentSeaEvent = nil
    
    for _, btn in pairs(seaEventButtons) do
        btn.updateState()
    end
end

-- ============================================
-- SISTEMA DE ITENS (CDK, TTK, Soul Guitar)
-- ============================================

local itemCoroutines = {}

-- Função principal para pegar itens
local function startItemQuest(itemKey)
    -- Parar se já estiver rodando
    if itemCoroutines[itemKey] then
        task.cancel(itemCoroutines[itemKey])
    end
    
    local itemConfig = settings.itens[itemKey]
    if not itemConfig then return end
    
    itemConfig.enabled = true
    print("💎 Iniciando quest: " .. itemConfig.name)
    
    itemCoroutines[itemKey] = task.spawn(function()
        -- Verificar level mínimo
        local currentLevel = player.Data.Level.Value
        
        if currentLevel < itemConfig.requirements.level then
            print("❌ Level insuficiente! Precisa ser level " .. itemConfig.requirements.level)
            -- Farmar level primeiro
            farmToLevel(itemConfig.requirements.level, itemKey)
        end
        
        -- Coletar itens necessários
        local hasAllItems = false
        while itemConfig.enabled and not hasAllItems do
            hasAllItems = checkRequiredItems(itemConfig.requirements.items)
            
            if not hasAllItems then
                print("📦 Coletando itens necessários...")
                collectRequiredItems(itemConfig.requirements.items, itemKey)
            end
            task.wait(2)
        end
        
        -- Completar passos da quest
        if itemConfig.enabled then
            for _, step in pairs(itemConfig.questSteps) do
                if not itemConfig.enabled then break end
                
                print("🚶 Indo para: " .. step.npc)
                
                -- Teleportar para o NPC
                safeTeleport(step.location + Vector3.new(0, 10, 0), 0.5)
                task.wait(1)
                
                -- Encontrar e interagir com o NPC
                local npc = findNPC(step.npc)
                if npc then
                    safeTeleportToNPC(npc)
                    task.wait(0.5)
                    interactWithNPC(npc)
                    task.wait(1)
                    
                    -- Completar puzzle baseado no item
                    if itemKey == "cdk" then
                        completeCDKPuzzle(step.npc)
                    elseif itemKey == "ttk" then
                        completeTTKPuzzle(step.npc)
                    elseif itemKey == "soulGuitar" then
                        completeSoulGuitarPuzzle(step.npc)
                    end
                end
                
                task.wait(2)
            end
            
            -- Verificar se pegou o item
            if checkItemInInventory(itemConfig.name) then
                print("✅ " .. itemConfig.name .. " adquirido com sucesso!")
            else
                print("⚠️ Tentando novamente...")
                -- Tentar novamente
                if itemConfig.enabled then
                    collectItem(itemConfig.name, itemKey)
                end
            end
        end
        
        -- Desativar após completar
        settings.itens[itemKey].enabled = false
        for _, btn in pairs(itemButtons) do
            btn.updateState()
        end
    end)
end

-- Farmar até o level necessário
local function farmToLevel(targetLevel, itemKey)
    print("📈 Farmando até level " .. targetLevel .. "...")
    
    local tempFarmEnabled = true
    
    while tempFarmEnabled and settings.itens[itemKey].enabled do
        local currentLevel = player.Data.Level.Value
        if currentLevel >= targetLevel then
            tempFarmEnabled = false
            break
        end
        
        -- Auto farm temporário
        local level = player.Data.Level.Value
        local questConfig = getQuestByLevel(level)
        
        if questConfig then
            local questNPC = findNPC(questConfig.questGiver)
            if questNPC then
                safeTeleportToNPC(questNPC)
                task.wait(0.5)
                interactWithNPC(questNPC)
                task.wait(0.5)
            end
            
            local farmPos = questConfig.farmPosition
            local safeFarmPos = Vector3.new(farmPos.X, farmPos.Y + 50, farmPos.Z)
            safeTeleport(safeFarmPos, 0.5)
            
            startAutoAttack()
            
            task.spawn(function()
                while tempFarmEnabled and settings.itens[itemKey].enabled do
                    bringMobs(farmPos)
                    task.wait(0.8)
                end            end)
            
            -- Esperar completar quest
            while tempFarmEnabled and settings.itens[itemKey].enabled do
                task.wait(1)
                if checkQuestComplete(questConfig) then
                    local questNPC = findNPC(questConfig.questGiver)
                    if questNPC then
                        safeTeleportToNPC(questNPC)
                        task.wait(0.5)
                        interactWithNPC(questNPC)
                        task.wait(0.5)
                    end
                    break
                end
            end
        end
        
        task.wait(1)
    end
    
    print("✅ Level necessário atingido!")
end

-- Verificar itens necessários
local function checkRequiredItems(requiredItems)
    for _, itemName in pairs(requiredItems) do
        if not hasItem(itemName) then
            return false
        end
    end
    return true
end

-- Coletar itens necessários
local function collectRequiredItems(requiredItems, itemKey)
    for _, itemName in pairs(requiredItems) do
        if not settings.itens[itemKey].enabled then break end
        
        if not hasItem(itemName) then
            print("🔍 Procurando: " .. itemName)
            
            -- Procurar item no mapa
            local item = findItem(itemName)
            if item then
                -- Teleportar para o item
                local itemPos = item.Position
                safeTeleport(itemPos + Vector3.new(0, 3, 0), 0.3)
                task.wait(0.5)
                
                -- Pegar item
                fireproximityprompt(item)
                task.wait(0.5)
                
                print("✅ Item coletado: " .. itemName)
            else
                -- Farmar inimigos que dropam o item
                farmItem(itemName, itemKey)
            end
        end
        
        task.wait(1)
    end
end

-- Verificar se tem o item
local function hasItem(itemName)
    for _, item in pairs(player.Backpack:GetChildren()) do
        if item:IsA("Tool") and item.Name:lower():find(itemName:lower()) then
            return true
        end
    end
    
    for _, item in pairs(character:GetChildren()) do
        if item:IsA("Tool") and item.Name:lower():find(itemName:lower()) then
            return true
        end
    end
    
    return false
end

-- Encontrar item no mapa
local function findItem(itemName)
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("BasePart") and obj.Name:lower():find(itemName:lower()) then
            if obj:FindFirstChild("ProximityPrompt") then
                return obj
            end
        end
    end
    return nil
end

-- Farmar item de inimigos
local function farmItem(itemName, itemKey)
    print("⚔️ Farmando " .. itemName .. "...")
    
    -- Localizações de farm baseadas no item
    local farmLocations = {
        ["Alucard Shard"] = Vector3.new(-11000, 20, -8000),
        ["Dark Fragment"] = Vector3.new(-11500, 20, -8500),
        ["Bone"] = Vector3.new(-9000, 20, -6000),
        ["Scrap Metal"] = Vector3.new(-8500, 20, -5500),
        ["Katana"] = Vector3.new(-7000, 20, -5000),
        ["Wando"] = Vector3.new(-7500, 20, -5500),
        ["Saddi"] = Vector3.new(-8000, 20, -6000),
        ["Soul"] = Vector3.new(-12000, 20, -9000)
    }
    
    local farmPos = farmLocations[itemName]
    if farmPos then
        local safeFarmPos = Vector3.new(farmPos.X, farmPos.Y + 50, farmPos.Z)
        safeTeleport(safeFarmPos, 0.5)
        
        startAutoAttack()
        
        -- Farmar até conseguir o item
        while settings.itens[itemKey].enabled and not hasItem(itemName) do
            bringMobs(farmPos)
            task.wait(1)
        end
        
        stopAutoAttack()
    end
end

-- Completar puzzle CDK
local function completeCDKPuzzle(npcName)
    print("🧩 Resolvendo puzzle CDK...")
    
    if npcName == "Tushita Puzzle" then
        -- Puzzle da Tushita
        local tushitaLocation = Vector3.new(-9500, 20, -7000)
        safeTeleport(tushitaLocation + Vector3.new(0, 10, 0), 0.5)
        task.wait(1)
        
        -- Ativar tochas
        activateTorches()
        
    elseif npcName == "Yama Puzzle" then
        -- Puzzle da Yama
        local yamaLocation = Vector3.new(-10000, 20, -7500)
        safeTeleport(yamaLocation + Vector3.new(0, 10, 0), 0.5)
        task.wait(1)
        
        -- Coletar 30 bones
        print("💀 Coletando 30 bones...")
        for i = 1, 30 do
            if not settings.itens.cdk.enabled then break end
            collectBones()
            task.wait(0.5)
        end
    end
end

-- Completar puzzle TTK
local function completeTTKPuzzle(npcName)
    print("🧩 Resolvendo puzzle TTK...")
    
    if npcName == "Sword Dealer" then
        -- Comprar katana básica
        local dealerLocation = Vector3.new(-7000, 20, -5000)
        safeTeleport(dealerLocation + Vector3.new(0, 5, 0), 0.5)
        task.wait(1)
        
        -- Comprar katana
        buyItem("Katana")
        
    elseif npcName == "Legendary Sword Dealer" then
        -- Comprar lendárias
        local legendaryLocation = Vector3.new(-7500, 20, -5500)
        safeTeleport(legendaryLocation + Vector3.new(0, 5, 0), 0.5)
        task.wait(1)
        
        -- Comprar Wando e Saddi
        buyItem("Wando")
        task.wait(0.5)
        buyItem("Saddi")
    end
end

-- Completar puzzle Soul Guitar
local function completeSoulGuitarPuzzle(npcName)
    print("🧩 Resolvendo puzzle Soul Guitar...")
    
    if npcName == "Soul Reaper" then
        -- Derrotar Soul Reaper
        local soulReaperLocation = Vector3.new(-11500, 20, -8500)
        safeTeleport(soulReaperLocation + Vector3.new(0, 100, 0), 0.5)
        task.wait(1)
        
        -- Encontrar e derrotar Soul Reaper
        local soulReaper = findSeaBoss("soul reaper")
        if soulReaper then
            startAutoAttack()
            
            while settings.itens.soulGuitar.enabled and soulReaper and soulReaper.Parent do
                local hum = soulReaper:FindFirstChild("Humanoid")
                if not hum or hum.Health <= 0 then break end
                task.wait(1)
            end
            
            stopAutoAttack()
        end
    end
end

-- Funções auxiliares para puzzles
local function activateTorches()
    print("🔥 Ativando tochas...")
    for i = 1, 5 do
        local torchPos = Vector3.new(-9500 + (i * 50), 20, -7000)
        safeTeleport(torchPos, 0.3)
        task.wait(0.3)
        
        -- Procurar tocha próxima
        for _, obj in pairs(workspace:GetDescendants()) do
            if obj:IsA("BasePart") and obj.Name:lower():find("torch") then
                if (obj.Position - torchPos).Magnitude < 10 then
                    fireproximityprompt(obj)
                    break
                end
            end
        end
        task.wait(0.5)
    end
end

local function collectBones()
    local boneLocation = Vector3.new(-9000, 20, -6000)
    safeTeleport(boneLocation + Vector3.new(0, 50, 0), 0.3)
    task.wait(0.5)
    startAutoAttack()
    task.wait(2)
    stopAutoAttack()
end

local function buyItem(itemName)
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("BasePart") and obj:FindFirstChild("ProximityPrompt") then
            if obj.Name:lower():find(itemName:lower()) then
                fireproximityprompt(obj)
                return
            end
        end
    end
end

-- Verificar se item está no inventário
local function checkItemInInventory(itemName)
    for _, item in pairs(player.Backpack:GetChildren()) do
        if item:IsA("Tool") and item.Name:lower():find(itemName:lower()) then
            return true
        end
    end
    
    for _, item in pairs(character:GetChildren()) do
        if item:IsA("Tool") and item.Name:lower():find(itemName:lower()) then
            return true
        end
    end
    
    return false
end

-- Coletar item final
local function collectItem(itemName, itemKey)
    print("🎯 Tentando coletar: " .. itemName)
    
    -- Procurar item específico
    local item = findItem(itemName)
    if item then
        safeTeleport(item.Position + Vector3.new(0, 3, 0), 0.3)
        task.wait(0.5)
        fireproximityprompt(item)
        task.wait(0.5)
    end
end

-- Parar quest de item
local function stopItemQuest(itemKey)
    if itemCoroutines[itemKey] then
        task.cancel(itemCoroutines[itemKey])
        itemCoroutines[itemKey] = nil
    end
    
    settings.itens[itemKey].enabled = false
    stopAutoAttack()
    
    for _, btn in pairs(itemButtons) do
        btn.updateState()
    end
end

-- ============================================
-- SISTEMA AUTO FARM
-- ============================================

local farmCoroutine = nil

local function startAutoFarm()
    if farmCoroutine then
        task.cancel(farmCoroutine)
    end
    
    settings.autoFarm.enabled = true
    
    farmCoroutine = task.spawn(function()
        while settings.autoFarm.enabled do
            local level = player.Data.Level.Value
            
            local questConfig = getQuestByLevel(level)
            if questConfig then
                local questNPC = findNPC(questConfig.questGiver)
                if questNPC then
                    safeTeleportToNPC(questNPC)
                    task.wait(math.random(3, 6) / 10)
                    interactWithNPC(questNPC)
                    task.wait(math.random(2, 4) / 10)
                end
                
                local farmPos = questConfig.farmPosition
                local safeFarmPos = Vector3.new(
                    farmPos.X + math.random(-5, 5),
                    farmPos.Y + settings.autoFarm.safeHeight,
                    farmPos.Z + math.random(-5, 5)
                )
                
                safeTeleport(safeFarmPos, 0.5)
                task.wait(math.random(1, 3))
                
                task.spawn(function()
                    while settings.autoFarm.enabled do
                        bringMobs(farmPos)
                        task.wait(math.random(5, 8) / 10)
                    end
                end)
                
                startAutoAttack()
                
                while settings.autoFarm.enabled do
                    task.wait(math.random(1, 2))
                    if checkQuestComplete(questConfig) then
                        local questNPC = findNPC(questConfig.questGiver)
                        if questNPC then
                            safeTeleportToNPC(questNPC)
                            task.wait(math.random(3, 5) / 10)
                            interactWithNPC(questNPC)
                            task.wait(math.random(2, 4) / 10)
                        end
                        break
                    end
                end
            end
            task.wait(math.random(1, 2))
        end
    end)
end

-- ============================================
-- FUNÇÕES AUXILIARES GERAIS
-- ============================================

local function safeTeleport(targetPosition, time)
    time = time or 0.5
    task.wait(math.random(1, 3) / 10)
    
    local tweenInfo = TweenInfo.new(
        time + math.random(1, 5) / 100,
        Enum.EasingStyle.Quad,
        Enum.EasingDirection.Out
    )
    
    local tween = tweenService:Create(rootPart, tweenInfo, {CFrame = CFrame.new(targetPosition)})
    tween:Play()
    tween.Completed:Wait()
    
    task.wait(math.random(2, 5) / 10)
end

local function findSword()
    for _, tool in pairs(character:GetChildren()) do
        if tool:IsA("Tool") and tool:FindFirstChild("Handle") then
            local toolName = tool.Name:lower()
            if toolName:find("sword") or toolName:find("blade") or toolName:find("cutlass") or 
               toolName:find("katana") or toolName:find("saber") or toolName:find("dark") then
                return tool
            end
        end
    end
    return nil
end

local function equipToolSafe(tool)
    if tool and humanoid then
        task.wait(math.random(1, 2) / 10)
        humanoid:EquipTool(tool)
        task.wait(math.random(1, 3) / 10)
    end
end

local autoAttackConnection = nil

local function startAutoAttack()
    if autoAttackConnection then
        autoAttackConnection:Disconnect()
    end
    
    local clickDelays = {0.1, 0.15, 0.12, 0.18, 0.13, 0.11, 0.14}
    
    autoAttackConnection = runService.Heartbeat:Connect(function()
        local hasActiveEvent = false
        for _, event in pairs(settings.seaEvents) do
            if event.enabled then
                hasActiveEvent = true
                break
            end
        end
        
        local hasActiveItem = false
        for _, item in pairs(settings.itens) do
            if item.enabled then
                hasActiveItem = true
                break
            end
        end
        
        if not hasActiveEvent and not settings.autoFarm.enabled and not hasActiveItem then
            autoAttackConnection:Disconnect()
            autoAttackConnection = nil
            return
        end
        
        local sword = findSword()
        if sword and humanoid and humanoid.Health > 0 then
            equipToolSafe(sword)
            
            local delay = clickDelays[math.random(1, #clickDelays)]
            
            if sword:FindFirstChild("Remote") then
                sword.Remote:FireServer()
            elseif sword:FindFirstChild("ServerHandler") then
                sword.ServerHandler:InvokeServer()
            else
                virtualInput:SendMouseButtonEvent(0, 0, 0, true, game, 0)
                task.wait(delay)
                virtualInput:SendMouseButtonEvent(0, 0, 0, false, game, 0)
            end
        end
        task.wait(clickDelays[math.random(1, #clickDelays)])
    end)
end

local function stopAutoAttack()
    if autoAttackConnection then
        autoAttackConnection:Disconnect()
        autoAttackConnection = nil
    end
end

function getQuestByLevel(level)
    local quests = {
        {minLevel = 0, maxLevel = 10, questGiver = "Bandit", farmPosition = Vector3.new(1000, 20, 1000), requiredKills = 5},
        {minLevel = 10, maxLevel = 25, questGiver = "Monkey", farmPosition = Vector3.new(1200, 20, 800), requiredKills = 8},
    }
    
    for _, quest in pairs(quests) do
        if level >= quest.minLevel and level <= quest.maxLevel then
            return quest
        end
    end
    return nil
end

function findNPC(name)
    for _, npc in pairs(workspace:GetDescendants()) do
        if npc:IsA("Model") and npc.Name:lower():find(name:lower()) and npc:FindFirstChild("Humanoid") then
            return npc
        end
    end
    return nil
end

function safeTeleportToNPC(npc)
    local pos = npc:GetPivot().Position
    safeTeleport(pos + Vector3.new(math.random(-2, 2), 3, math.random(3, 7)), 0.3)
end

function interactWithNPC(npc)
    task.wait(math.random(1, 2) / 10)
    if npc:FindFirstChild("ProximityPrompt") then
        fireproximityprompt(npc.ProximityPrompt)
    elseif npc:FindFirstChild("ClickDetector") then
        fireclickdetector(npc.ClickDetector)
    end
end

function bringMobs(centerPosition)
    local radius = 50
    local broughtCount = 0
    local maxBring = 3
    
    for _, obj in pairs(workspace:GetDescendants()) do
        if broughtCount >= maxBring then break end
        
        if obj:IsA("Model") and obj:FindFirstChild("Humanoid") and obj.Humanoid.Health > 0 then
            if obj.Name:lower():find("enemy") or obj:FindFirstChild("Enemy") then
                local mobPos = obj:GetPivot().Position
                if (mobPos - centerPosition).Magnitude > radius then
                    obj:SetPrimaryPartCFrame(CFrame.new(
                        centerPosition + Vector3.new(
                            math.random(-5, 5),
                            2,
                            math.random(-5, 5)
                        )
                    ))
                    broughtCount = broughtCount + 1
                    task.wait(math.random(1, 2) / 10)
                end
            end
        end
    end
end

function checkQuestComplete(questConfig)
    local questProgress = player.QuestProgress.Value
    return questProgress >= questConfig.requiredKills
end

-- ============================================
-- CRIAR BOTÕES DA UI
-- ============================================

-- Sea Events
local seaTitle = Instance.new("TextLabel")
seaTitle.Size = UDim2.new(0.9, 0, 0, 25)
seaTitle.BackgroundTransparency = 1
seaTitle.Text = "🌊 SEA EVENTS"
seaTitle.TextColor3 = Color3.fromRGB(0, 255, 255)
seaTitle.TextSize = 14
seaTitle.Font = Enum.Font.GothamBold
seaTitle.Parent = seaContent

createSeaEventButton(seaContent, "terrorShark", "Terror Shark", "🦈")
createSeaEventButton(seaContent, "seaBeast", "Sea Beast", "🐉")
createSeaEventButton(seaContent, "piranha", "Piranha", "🐟")
createSeaEventButton(seaContent, "tubarao", "Tubarão", "🦈")
createSeaEventButton(seaContent, "barcoAssombrado", "Barco Assombrado", "👻")

createButton(seaContent, "⛔ Parar Todos", function()
    stopSeaEvent()
    print("⛔ Todos os Sea Events parados!")
end)

-- Farm
createButton(farmContent, "⚡ Auto Farm Level", function()
    settings.autoFarm.enabled = not settings.autoFarm.enabled
    if settings.autoFarm.enabled then
        startAutoFarm()
    else
        if farmCoroutine then
            task.cancel(farmCoroutine)
        end
        stopAutoAttack()
    end
end)

-- Itens
local itensTitle = Instance.new("TextLabel")
itensTitle.Size = UDim2.new(0.9, 0, 0, 25)
itensTitle.BackgroundTransparency = 1
itensTitle.Text = "💎 ITENS LENDÁRIOS"
itensTitle.TextColor3 = Color3.fromRGB(0, 255, 255)
itensTitle.TextSize = 14
itensTitle.Font = Enum.Font.GothamBold
itensTitle.Parent = itensContent

createItemButton(itensContent, "cdk", "Curse Dual Katana", "⚔️")
createItemButton(itensContent, "ttk", "Triple Katana", "🗡️")
createItemButton(itensContent, "soulGuitar", "Soul Guitar", "🎸")

-- ============================================
-- CONTROLES DAS ABAS
-- ============================================

local function updateTabColors(activeTab)
    local tabs = {seaTab, farmTab, itensTab}
    local neons = {seaNeon, farmNeon, itensNeon}
    
    for i, tab in pairs(tabs) do
        if tab == activeTab then
            tab.BackgroundColor3 = Color3.fromRGB(0, 255, 255)
            neons[i].Color = Color3.fromRGB(0, 255, 255)
        else
            tab.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
            neons[i].Color = Color3.fromRGB(100, 100, 100)
        end
    end
end

seaTab.MouseButton1Click:Connect(function()
    seaContent.Visible = true
    farmContent.Visible = false
    itensContent.Visible = false
    updateTabColors(seaTab)
end)

farmTab.MouseButton1Click:Connect(function()
    seaContent.Visible = false
    farmContent.Visible = true
    itensContent.Visible = false
    updateTabColors(farmTab)
end)

itensTab.MouseButton1Click:Connect(function()
    seaContent.Visible = false
    farmContent.Visible = false
    itensContent.Visible = true
    updateTabColors(itensTab)
end)

-- ============================================
-- CONTROLE DO BOTÃO FANTASMA
-- ============================================

ghostButton.MouseButton1Click:Connect(function()
    mainFrame.Visible = not mainFrame.Visible
    
    ghostButton.Size = UDim2.new(0, 55, 0, 55)
    task.wait(0.1)
    ghostButton.Size = UDim2.new(0, 45, 0, 45)
end)

-- ============================================
-- ARRASTO DA UI
-- ============================================

local dragging = false
local dragInput, dragStart, startPos

mainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

mainFrame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

runService.RenderStepped:Connect(function()
    if dragging and dragInput then
        local delta = dragInput.Position - dragStart
        mainFrame.Position = UDim2.new(
            startPos.X.Scale,
            startPos.X.Offset + delta.X,
            startPos.Y.Scale,
            startPos.Y.Offset + delta.Y
        )
    end
end)

-- ============================================
-- EFEITOS VISUAIS
-- ============================================

task.spawn(function()
    while antiBan.enabled do
        for i = 1, 10 do
            stroke.Transparency = 0.3 + (math.sin(i * 0.5) * 0.4)
            ghostStroke.Transparency = 0.3 + (math.cos(i * 0.5) * 0.4)
            task.wait(0.1)
        end
    end
end)

task.spawn(function()
    while antiBan.enabled do
        for i = 1, 20 do
            ghostButton.Position = UDim2.new(0, 10, 0, 10 + math.sin(i * 0.3) * 3)
            task.wait(0.1)
        end
    end
end)

-- ============================================
-- INICIALIZAÇÃO
-- ============================================

initAntiBan()

task.spawn(function()
    while antiBan.enabled do
        antiBanStatus.Text = "🛡️ ANTI-BAN ✓"
        antiBanStatus.TextColor3 = Color3.fromRGB(0, 255, 0)
        task.wait(5)
        antiBanStatus.Text = "🛡️ ANTI-BAN"
        antiBanStatus.TextColor3 = Color3.fromRGB(0, 200, 0)
        task.wait(0.5)
    end
end)

local notification = Instance.new("TextLabel")
notification.Size = UDim2.new(0, 300, 0, 30)
notification.Position = UDim2.new(0.5, -150, 1, -50)
notification.BackgroundColor3 = Color3.fromRGB(0, 255, 255)
notification.BackgroundTransparency = 0.2
notification.TextColor3 = Color3.fromRGB(255, 255, 255)
notification.Text = "🛡️ Script Completo Carregado!"
notification.TextSize = 14
notification.Font = Enum.Font.GothamBold
notification.Parent = screenGui

local notifCorner = Instance.new("UICorner")
notifCorner.CornerRadius = UDim.new(0, 4)
notifCorner.Parent = notification

task.delay(4, function()
    for i = 1, 10 do
        notification.BackgroundTransparency = 0.2 + (i * 0.08)
        task.wait(0.05)
    end
    notification:Destroy()
end)

print("🛡️ Sistema Anti-Ban 100% ATIVADO!")
print("🌊 Sea Events | ⚔️ Farm | 💎 Itens")
print("👻 Clique no fantasma para abrir o menu!")
