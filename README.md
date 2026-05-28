if not game:IsLoaded() then game.Loaded:Wait() end

local VirtualInputManager = game:GetService("VirtualInputManager")
local ProximityPromptService = game:GetService("ProximityPromptService")
local Players = game:GetService("Players")
local LocalizationService = game:GetService("LocalizationService")
local LocalPlayer = Players.LocalPlayer

if _G.AutoKick == nil then _G.AutoKick = false end
if _G.AutoTrain == nil then _G.AutoTrain = false end
if _G.AutoRebirth == nil then _G.AutoRebirth = false end
if _G.AutoBuySpeed == nil then _G.AutoBuySpeed = false end
-- Mặc định tự động kích hoạt Instant Interact khi chạy script
if _G.InstantInteract == nil then _G.InstantInteract = true end 
if _G.KickSpeed == nil then _G.KickSpeed = 0.4 end
if _G.TrainSpeed == nil then _G.TrainSpeed = 0.2 end

local weightKeys = {
    "BuySmallNormal", "BuyBigNormal", "BuySmallElectro", "BuyBigElectro",
    "BuySmallIce", "BuyBigIce", "BuySmallLava", "BuyBigLava",
    "BuySmallNeon", "BuyBigNeon", "BuySmallVenom", "BuyBigVenom"
}
for _, key in ipairs(weightKeys) do
    if _G[key] == nil then _G[key] = false end
end

local Languages = {
    ["Tiếng Việt"] = {
        Title = "👑 HỆ THỐNG MIKEY 👑", Loading = "ĐANG KHỞI TẠO MIKEY...", SubLoading = "bởi MIKEY Team",
        FarmTab = "MIKEY Cày Cuốc", ShopTab = "MIKEY Mua Đồ", TeleportTab = "MIKEY Dịch Chuyển", SettingsTab = "MIKEY Cài Đặt",
        AutoKick = "Auto Kick & Collect (Đá bóng + Nhặt)", AutoTrain = "Auto Train Gym (Tự cầm tạ)", AutoRebirth = "Auto Rebirth (Đủ điểm tự hồi sinh)",
        AutoBuySpeed = "Auto Buy Speed (Tự nâng tốc độ)", InstantInteract = "Instant Interact (Ấn E ăn ngay)",
        AutoBuy = "Tự động mua: ", TeleTo = "⚡ Dịch chuyển đến ", TeleNotif = "Đã đến ", KickSpeed = "Tốc độ Đá bóng (Giây)", TrainSpeed = "Tốc độ Tập Gym (Giây)",
        LangSelect = "Chọn Ngôn Ngữ Hệ Thống", LangNotify = "Đang áp dụng giao diện Tiếng Việt..."
    },
    ["English"] = {
        Title = "👑 MIKEY SYSTEM 👑", Loading = "INITIALIZING MIKEY...", SubLoading = "by MIKEY Team",
        FarmTab = "MIKEY Farm", ShopTab = "MIKEY Shop", TeleportTab = "MIKEY Teleport", SettingsTab = "MIKEY Settings",
        AutoKick = "Auto Kick & Collect", AutoTrain = "Auto Train Gym (Auto-Equip)", AutoRebirth = "Auto Rebirth (Smart Check)",
        AutoBuySpeed = "Auto Buy Speed (Upgrade)", InstantInteract = "Instant Interact (Fast E)",
        AutoBuy = "Auto Buy: ", TeleTo = "⚡ Teleport to ", TeleNotif = "Arrived at ", KickSpeed = "Kick Speed (Sec)", TrainSpeed = "Train Speed (Sec)",
        LangSelect = "Select Menu Language", LangNotify = "Applying English Interface..."
    },
    ["中文"] = {
        Title = "👑 MIKEY 系统 👑", Loading = "MIKEY 初始化中...", SubLoading = "由 MIKEY 团队制作",
        FarmTab = "MIKEY 自动刷", ShopTab = "MIKEY 商店", TeleportTab = "MIKEY 传送", SettingsTab = "MIKEY 设置",
        AutoKick = "自动踢球与收集 (踢球 + 捡取)", AutoTrain = "自动健身 (自动拿哑铃)", AutoRebirth = "自动转生 (分数足够时)",
        AutoBuySpeed = "自动购买速度 (自动升级)", InstantInteract = "瞬间交互 (秒按E键)",
        AutoBuy = "自动购买: ", TeleTo = "⚡ 传送到 ", TeleNotif = "已到达 ", KickSpeed = "踢球速度 (秒)", TrainSpeed = "健身速度 (秒)",
        LangSelect = "选择菜单语言", LangNotify = "正在应用中文界面..."
    },
    ["Español"] = {
        Title = "👑 SISTEMA MIKEY 👑", Loading = "INICIALIZANDO MIKEY...", SubLoading = "por MIKEY Team",
        FarmTab = "MIKEY Farm", ShopTab = "MIKEY Tienda", TeleportTab = "MIKEY Teletransporte", SettingsTab = "MIKEY Ajustes",
        AutoKick = "Auto Patear y Recoger", AutoTrain = "Auto Entrenar Gimnasio", AutoRebirth = "Auto Renacimiento",
        AutoBuySpeed = "Auto Comprar Velocidad", InstantInteract = "Interacción Instantánea (E)",
        AutoBuy = "Auto Comprar: ", TeleTo = "⚡ Teletransporte a ", TeleNotif = "Llegaste a ", KickSpeed = "Velocidad de Patada (Seg)", TrainSpeed = "Velocidad de Entreno (Seg)",
        LangSelect = "Seleccionar Idioma", LangNotify = "Aplicando interfaz en Español..."
    }
}

if not _G.CurrentLangKey then
    _G.CurrentLangKey = "English"
    local SystemCode = LocalizationService.RobloxLocaleId:sub(1, 2)
    if SystemCode == "vi" then _G.CurrentLangKey = "Tiếng Việt"
    elseif SystemCode == "zh" then _G.CurrentLangKey = "中文"
    elseif SystemCode == "es" then _G.CurrentLangKey = "Español"
    end
end

local Teleports = {
    ["Shop"] = CFrame.new(-58.880398, 3.156276, 13.833234),
    ["Buy Speed"] = CFrame.new(49.988918, 3.156275, 15.079915),
    ["Sell"] = CFrame.new(71.593460, 3.356277, 15.712656),
    ["Bảng Xếp Hạng"] = CFrame.new(116.796227, 3.170272, 2.495253)
}

local function TeleportTo(cf)
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("HumanoidRootPart") then char.HumanoidRootPart.CFrame = cf end
end

local function HasWeight(weightName)
    local backpack = LocalPlayer:FindFirstChild("Backpack")
    local character = LocalPlayer.Character
    return (backpack and backpack:FindFirstChild(weightName)) or (character and character:FindFirstChild(weightName))
end

local function GetCurrentStrength()
    local leaderstats = LocalPlayer:FindFirstChild("leaderstats")
    if leaderstats then
        local strength = leaderstats:FindFirstChild("Strength") or leaderstats:FindFirstChild("Power") or leaderstats:FindFirstChild("Biceps")
        if strength then return strength.Value end
    end
    return 0
end

local function GetRebirthCost()
    local rebirthSettings = game:GetService("ReplicatedStorage"):FindFirstChild("RebirthSettings") or LocalPlayer:FindFirstChild("RebirthCost")
    if rebirthSettings and rebirthSettings:IsA("ValueBase") then return rebirthSettings.Value end
    local leaderstats = LocalPlayer:FindFirstChild("leaderstats")
    if leaderstats then
        local rebirths = leaderstats:FindFirstChild("Rebirths") or leaderstats:FindFirstChild("Rebirth")
        if rebirths then return math.max(5000, rebirths.Value * 10000) end
    end
    return 5000
end

local function EnsureToolEquipped()
    local char = LocalPlayer.Character
    local backpack = LocalPlayer:FindFirstChild("Backpack")
    if not char or not backpack then return end
    local weightKeywords = {"Weight", "Normal", "Electro", "Ice", "Lava", "Neon", "Venom"}
    local function isWeight(toolName)
        for _, keyword in ipairs(weightKeywords) do if string.find(toolName, keyword) then return true end end
        return false
    end
    local currentTool = char:FindFirstChildOfClass("Tool")
    if currentTool and not isWeight(currentTool.Name) then currentTool.Parent = backpack task.wait(0.05) end
    if not char:FindFirstChildOfClass("Tool") then
        for _, tool in ipairs(backpack:GetChildren()) do
            if tool:IsA("Tool") and isWeight(tool.Name) then tool.Parent = char break end
        end
    end
end

if not _G.LoopsInitialized then
    _G.LoopsInitialized = true
    
    LocalPlayer.Idled:Connect(function()
       game:GetService("VirtualUser"):CaptureController()
       game:GetService("VirtualUser"):ClickButton2(Vector2.new())
    end)

    ProximityPromptService.PromptButtonHoldBegan:Connect(function(prompt)
        if _G.InstantInteract then fireproximityprompt(prompt) end
    end)

    task.spawn(function()
        while true do
            if _G.AutoKick then
                pcall(function()
                    local K = game.ReplicatedStorage:FindFirstChild("kick", true) or game.ReplicatedStorage:FindFirstChild("shoot", true)
                    local C = game.ReplicatedStorage:FindFirstChild("collect", true) or game.ReplicatedStorage:FindFirstChild("claim", true)
                    if K then K:FireServer(100) end
                    if C then C:FireServer() end
                end)
                task.wait(_G.KickSpeed)
            else
                task.wait(0.5)
            end
        end
    end)

    task.spawn(function()
        while true do
            if _G.AutoTrain then
                pcall(function()
                    EnsureToolEquipped()
                    local T = game.ReplicatedStorage:FindFirstChild("train", true) or game.ReplicatedStorage:FindFirstChild("lift", true)
                    if T then T:FireServer() end
                end)
                task.wait(_G.TrainSpeed)
            else
                task.wait(0.5)
            end
        end
    end)

    task.spawn(function()
        while true do
            if _G.AutoRebirth then
                pcall(function()
                    local currentStrength = GetCurrentStrength()
                    local requiredStrength = GetRebirthCost()
                    if currentStrength >= requiredStrength then
                        game:GetService("ReplicatedStorage").Events.RequestRebirth:FireServer()
                        task.wait(2)
                    end
                end)
                task.wait(1)
            else
                task.wait(1)
            end
        end
    end)

    task.spawn(function()
        while true do
            if _G.AutoBuySpeed then
                pcall(function() game:GetService("ReplicatedStorage").Events.PurchaseUpgrade:FireServer("Speed", 1) end)
                task.wait(1)
            else
                task.wait(1)
            end
        end
    end)

    local weightPairs = {
        {"Small Normal", "BuySmallNormal"}, {"Big Normal", "BuyBigNormal"},
        {"Small Electro", "BuySmallElectro"}, {"Big Electro", "BuyBigElectro"},
        {"Small Ice", "BuySmallIce"}, {"Big Ice", "BuyBigIce"},
        {"Small Lava", "BuySmallLava"}, {"Big Lava", "BuyBigLava"},
        {"Small Neon", "BuySmallNeon"}, {"Big Neon", "BuyBigNeon"},
        {"Small Venom", "BuySmallVenom"}, {"BuySmallVenom"},
        {"Big Venom", "BuyBigVenom"}
    }
    task.spawn(function()
        while true do
            local anyBuy = false
            for _, pair in ipairs(weightPairs) do
                if pair[2] and _G[pair[2]] then
                    anyBuy = true
                    pcall(function()
                        if not HasWeight(pair[1]) then
                            game:GetService("ReplicatedStorage").Events.RequestWeightPurchase:FireServer(pair[1])
                        end
                    end)
                end
            end
            if anyBuy then task.wait(0.8) else task.wait(1) end
        end
    end)
end

local function BuildUI()
    local Lang = Languages[_G.CurrentLangKey]
    local MIKEY = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
    
    local Window = MIKEY:CreateWindow({
       Name = Lang.Title,
       LoadingTitle = Lang.Loading,
       LoadingSubtitle = Lang.SubLoading,
       ConfigurationSaving = { Enabled = false, FolderName = "MikeyHub", FileName = "Main" },
       Discord = { Enabled = false, Invite = "", RememberJoins = false },
       KeySystem = false
    })

    local FarmTab = Window:CreateTab(Lang.FarmTab, 4483362458)
    local ShopTab = Window:CreateTab(Lang.ShopTab, 4483362458)
    local TeleportTab = Window:CreateTab(Lang.TeleportTab, 4483362458)
    local SettingsTab = Window:CreateTab(Lang.SettingsTab, 4483362458)

    FarmTab:CreateToggle({Name = Lang.AutoKick, CurrentValue = _G.AutoKick, Callback = function(v) _G.AutoKick = v end})
    FarmTab:CreateToggle({Name = Lang.AutoTrain, CurrentValue = _G.AutoTrain, Callback = function(v) _G.AutoTrain = v end})
    FarmTab:CreateToggle({Name = Lang.AutoRebirth, CurrentValue = _G.AutoRebirth, Callback = function(v) _G.AutoRebirth = v end})
    FarmTab:CreateToggle({Name = Lang.AutoBuySpeed, CurrentValue = _G.AutoBuySpeed, Callback = function(v) _G.AutoBuySpeed = v end})
    FarmTab:CreateToggle({Name = Lang.InstantInteract, CurrentValue = _G.InstantInteract, Callback = function(v) _G.InstantInteract = v end})

    local function AddWeightToggle(name, displayName, globalVar)
        ShopTab:CreateToggle({Name = Lang.AutoBuy .. displayName, CurrentValue = _G[globalVar], Callback = function(v) _G[globalVar] = v end})
    end
    AddWeightToggle("Small Normal", "Small Normal", "BuySmallNormal")
    AddWeightToggle("Big Normal", "Big Normal", "BuyBigNormal")
    AddWeightToggle("Small Electro", "Small Electro", "BuySmallElectro")
    AddWeightToggle("Big Electro", "Big Electro", "BuyBigElectro")
    AddWeightToggle("Small Ice", "Small Ice", "BuySmallIce")
    AddWeightToggle("Big Ice", "Big Ice", "BuyBigIce")
    AddWeightToggle("Small Lava", "Small Lava", "BuySmallLava")
    AddWeightToggle("Big Lava", "Big Lava", "BuyBigLava")
    AddWeightToggle("Small Neon", "Small Neon", "BuySmallNeon")
    AddWeightToggle("Big Neon", "Big Neon", "BuyBigNeon")
    AddWeightToggle("Small Venom", "Small Venom", "BuySmallVenom")
    AddWeightToggle("Big Venom", "Big Venom", "BuyBigVenom")

    for name, pos in pairs(Teleports) do
        TeleportTab:CreateButton({
            Name = Lang.TeleTo .. name,
            Callback = function()
                TeleportTo(pos)
                MIKEY:Notify({Title = "MIKEY TELEPORT", Content = Lang.TeleNotif .. name, Duration = 2, Image = 4483362458})
            end,
        })
    end

    SettingsTab:CreateSlider({Name = Lang.KickSpeed, Range = {0.1, 2.0}, Increment = 0.1, CurrentValue = _G.KickSpeed, Callback = function(v) _G.KickSpeed = v end})
    SettingsTab:CreateSlider({Name = Lang.TrainSpeed, Range = {0.05, 1.0}, Increment = 0.05, CurrentValue = _G.TrainSpeed, Callback = function(v) _G.TrainSpeed = v end})

    SettingsTab:CreateDropdown({
       Name = Lang.LangSelect,
       Options = {"English", "Tiếng Việt", "中文", "Español"},
       CurrentOption = {_G.CurrentLangKey},
       MultipleOptions = false,
       Callback = function(SelectedOption)
           local targetLang = SelectedOption[1]
           if targetLang and targetLang ~= _G.CurrentLangKey then
               _G.CurrentLangKey = targetLang
               
               MIKEY:Notify({Title = "MIKEY SYSTEM", Content = Languages[targetLang].LangNotify, Duration = 2, Image = 4483362458})
               task.wait(0.3)
               
               MIKEY:Destroy()
               local oldGui = game:GetService("CoreGui"):FindFirstChild("Rayfield") or LocalPlayer.PlayerGui:FindFirstChild("Rayfield")
               if oldGui then oldGui:Destroy() end
               
               task.defer(BuildUI)
           end
       end,
    })

    MIKEY:Notify({Title = "👑 MIKEY SYSTEM 👑", Content = "UI Loaded (".._G.CurrentLangKey..")", Duration = 3, Image = 4483362458})
end

BuildUI()
