--[[
    ╔═══════════════════════════════════════════════════════════════════╗
    ║                   ⛩️ GAKURAN OMNI-HUB V1 ⛩️                     ║
    ║        Auto Parry · Hitbox Expander · Combat Aimbot · Bypass      ║
    ╚═══════════════════════════════════════════════════════════════════╝
    Jogo: Gakuran (学乱)
    Estilo: Japanese Delinquent Brawler / Parry-based PvP
--]]

-- ═══════════════════════════════════════════════════════════════
-- SECURE ENVIRONMENT & BYPASS INIT
-- ═══════════════════════════════════════════════════════════════
local _cloneref = cloneref or function(o) return o end
local Players = _cloneref(game:GetService("Players"))
local RunService = _cloneref(game:GetService("RunService"))
local UIS = _cloneref(game:GetService("UserInputService"))
local CoreGui = _cloneref(game:GetService("CoreGui"))
local Workspace = _cloneref(game:GetService("Workspace"))
local TweenService = _cloneref(game:GetService("TweenService"))
local Lighting = _cloneref(game:GetService("Lighting"))

local LP = Players.LocalPlayer
local Camera = Workspace.CurrentCamera

-- Cleanup old instance
pcall(function()
    if CoreGui:FindFirstChild("GakuranOmniHub") then
        CoreGui.GakuranOmniHub:Destroy()
    end
end)
if shared._GAKURAN_KILL then pcall(shared._GAKURAN_KILL) end

-- Config
local C = {
    -- Combat
    autoParry = false,
    parryRange = 12,
    parryKey = Enum.KeyCode.F, -- Default block/parry key in most brawlers
    
    combatAimbot = false,
    aimbotRange = 25,
    aimSmoothing = 0.5,
    
    hitboxExpander = false,
    hitboxSize = 6,
    
    -- Local
    speedHack = false,
    walkSpeed = 22, -- Safe margin
    infJump = false,
    noClip = false,
    
    -- Visuals
    esp = false,
    chams = false,
    fullbright = false,
    
    -- Internal
    uiToggle = Enum.KeyCode.RightControl
}

-- ═══════════════════════════════════════════════════════════════
-- ANTI-CHEAT BYPASS (Metamethod Hooks)
-- Evita detecção de WalkSpeed, JumpPower e alterações de Hitbox
-- ═══════════════════════════════════════════════════════════════
local OldIndex, OldNamecall, OldNewIndex

OldNewIndex = hookmetamethod(game, "__newindex", newcclosure(function(self, key, val)
    if not checkcaller() then
        -- Block game from resetting our modified WalkSpeed/JumpPower
        if self:IsA("Humanoid") and (key == "WalkSpeed" or key == "JumpPower") then
            if C.speedHack then return end
        end
        -- Block game from resetting hitbox sizes
        if self:IsA("BasePart") and key == "Size" and self.Name == "HumanoidRootPart" then
            if C.hitboxExpander and self.Parent ~= LP.Character then return end
        end
    end
    return OldNewIndex(self, key, val)
end))

OldIndex = hookmetamethod(game, "__index", newcclosure(function(self, key)
    if not checkcaller() then
        -- Spoof Humanoid properties so the anti-cheat reads normal values
        if self:IsA("Humanoid") then
            if key == "WalkSpeed" then return 16 end
            if key == "JumpPower" then return 50 end
        end
        -- Spoof Hitbox sizes
        if self:IsA("BasePart") and key == "Size" and self.Name == "HumanoidRootPart" then
            if C.hitboxExpander and self.Parent ~= LP.Character then
                return Vector3.new(2, 2, 1) -- default R6/R15 HRP size
            end
        end
    end
    return OldIndex(self, key)
end))

OldNamecall = hookmetamethod(game, "__namecall", newcclosure(function(self, ...)
    local method = getnamecallmethod()
    if not checkcaller() then
        -- Block Kick/Ban requests from client
        if method == "Kick" or method == "kick" then
            return wait(9e9)
        end
    end
    return OldNamecall(self, ...)
end))

-- ═══════════════════════════════════════════════════════════════
-- UI THEME & SETUP (Japanese Delinquent / Dark Crimson)
-- ═══════════════════════════════════════════════════════════════
local THEME = {
    bg = Color3.fromRGB(15, 10, 12),
    panel = Color3.fromRGB(20, 15, 18),
    header = Color3.fromRGB(130, 20, 30),
    accent = Color3.fromRGB(220, 40, 50),
    accent_hover = Color3.fromRGB(255, 60, 70),
    text = Color3.fromRGB(240, 240, 240),
    text_dim = Color3.fromRGB(150, 140, 145),
    card = Color3.fromRGB(25, 18, 22),
    on = Color3.fromRGB(220, 40, 50),
    off = Color3.fromRGB(50, 30, 35)
}

local SG = Instance.new("ScreenGui")
SG.Name = "GakuranOmniHub"; SG.ResetOnSpawn = false; SG.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
pcall(function() SG.Parent = CoreGui end)
if not SG.Parent then pcall(function() SG.Parent = (gethui and gethui()) or LP:WaitForChild("PlayerGui") end) end

local MF = Instance.new("Frame", SG)
MF.Size = UDim2.new(0, 480, 0, 340); MF.Position = UDim2.new(0.5, -240, 0.5, -170)
MF.BackgroundColor3 = THEME.bg; MF.BorderSizePixel = 0; MF.ClipsDescendants = true
Instance.new("UICorner", MF).CornerRadius = UDim.new(0, 8)
local mfStroke = Instance.new("UIStroke", MF)
mfStroke.Color = THEME.accent; mfStroke.Thickness = 2; mfStroke.Transparency = 0.2

local TopBar = Instance.new("Frame", MF)
TopBar.Size = UDim2.new(1, 0, 0, 40); TopBar.BackgroundColor3 = THEME.header; TopBar.BorderSizePixel = 0
Instance.new("UICorner", TopBar).CornerRadius = UDim.new(0, 8)

local tbGrad = Instance.new("UIGradient", TopBar)
tbGrad.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(180, 20, 30)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(90, 10, 15))
})

local Title = Instance.new("TextLabel", TopBar)
Title.Size = UDim2.new(1, -60, 1, 0); Title.Position = UDim2.new(0, 15, 0, 0)
Title.BackgroundTransparency = 1; Title.Text = "⛩️ GAKURAN OMNI-HUB"
Title.TextColor3 = Color3.new(1,1,1); Title.Font = Enum.Font.GothamBlack; Title.TextSize = 16
Title.TextXAlignment = Enum.TextXAlignment.Left

local CloseBtn = Instance.new("TextButton", TopBar)
CloseBtn.Size = UDim2.new(0, 24, 0, 24); CloseBtn.Position = UDim2.new(1, -34, 0, 8)
CloseBtn.BackgroundColor3 = THEME.off; CloseBtn.Text = "X"; CloseBtn.TextColor3 = Color3.new(1,1,1)
CloseBtn.Font = Enum.Font.GothamBold; CloseBtn.TextSize = 12; CloseBtn.BorderSizePixel = 0
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)
CloseBtn.MouseButton1Click:Connect(function()
    if shared._GAKURAN_KILL then pcall(shared._GAKURAN_KILL) end
    MF.Visible = false; SG:Destroy()
end)

-- Dragging
local dg, ds, sp = false, nil, nil
TopBar.InputBegan:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then
        dg = true; ds = i.Position; sp = MF.Position
        local rel; rel = UIS.InputEnded:Connect(function(e)
            if e.UserInputType == Enum.UserInputType.MouseButton1 then dg = false; rel:Disconnect() end
        end)
    end
end)
UIS.InputChanged:Connect(function(i)
    if dg and i.UserInputType == Enum.UserInputType.MouseMovement then
        local d = i.Position - ds; MF.Position = UDim2.new(sp.X.Scale, sp.X.Offset + d.X, sp.Y.Scale, sp.Y.Offset + d.Y)
    end
end)

-- Layout
local LayoutArea = Instance.new("Frame", MF)
LayoutArea.Size = UDim2.new(1, -20, 1, -50); LayoutArea.Position = UDim2.new(0, 10, 0, 45)
LayoutArea.BackgroundTransparency = 1

local LLayout = Instance.new("UIListLayout", LayoutArea)
LLayout.FillDirection = Enum.FillDirection.Horizontal; LLayout.Padding = UDim.new(0, 10)

local Col1 = Instance.new("ScrollingFrame", LayoutArea)
Col1.Size = UDim2.new(0.5, -5, 1, 0); Col1.BackgroundTransparency = 1; Col1.ScrollBarThickness = 2; Col1.ScrollBarImageColor3 = THEME.accent
Col1.AutomaticCanvasSize = Enum.AutomaticSize.Y; Col1.CanvasSize = UDim2.new(0,0,0,0)
local Col2 = Instance.new("ScrollingFrame", LayoutArea)
Col2.Size = UDim2.new(0.5, -5, 1, 0); Col2.BackgroundTransparency = 1; Col2.ScrollBarThickness = 2; Col2.ScrollBarImageColor3 = THEME.accent
Col2.AutomaticCanvasSize = Enum.AutomaticSize.Y; Col2.CanvasSize = UDim2.new(0,0,0,0)

local L1 = Instance.new("UIListLayout", Col1); L1.Padding = UDim.new(0, 8); L1.SortOrder = Enum.SortOrder.LayoutOrder
local L2 = Instance.new("UIListLayout", Col2); L2.Padding = UDim.new(0, 8); L2.SortOrder = Enum.SortOrder.LayoutOrder

-- UI Components
local ord1, ord2 = 0, 0

local function Sec(col, title)
    local isCol1 = col == Col1
    local l = Instance.new("TextLabel", col)
    l.Size = UDim2.new(1, 0, 0, 20); l.BackgroundTransparency = 1
    l.Text = title; l.TextColor3 = THEME.accent; l.Font = Enum.Font.GothamBold; l.TextSize = 12
    l.TextXAlignment = Enum.TextXAlignment.Left
    if isCol1 then ord1 = ord1 + 1; l.LayoutOrder = ord1 else ord2 = ord2 + 1; l.LayoutOrder = ord2 end
end

local function Tog(col, label, key, callback)
    local isCol1 = col == Col1
    local f = Instance.new("Frame", col)
    f.Size = UDim2.new(1, -5, 0, 36); f.BackgroundColor3 = THEME.card; f.BorderSizePixel = 0
    Instance.new("UICorner", f).CornerRadius = UDim.new(0, 6)
    if isCol1 then ord1 = ord1 + 1; f.LayoutOrder = ord1 else ord2 = ord2 + 1; f.LayoutOrder = ord2 end

    local l = Instance.new("TextLabel", f)
    l.Size = UDim2.new(0.7, 0, 1, 0); l.Position = UDim2.new(0, 10, 0, 0); l.BackgroundTransparency = 1
    l.Text = label; l.TextColor3 = THEME.text; l.Font = Enum.Font.GothamSemibold; l.TextSize = 11
    l.TextXAlignment = Enum.TextXAlignment.Left

    local pill = Instance.new("Frame", f)
    pill.Size = UDim2.new(0, 36, 0, 18); pill.Position = UDim2.new(1, -46, 0.5, -9)
    pill.BackgroundColor3 = C[key] and THEME.on or THEME.off; pill.BorderSizePixel = 0
    Instance.new("UICorner", pill).CornerRadius = UDim.new(1, 0)

    local dot = Instance.new("Frame", pill)
    dot.Size = UDim2.new(0, 14, 0, 14)
    dot.Position = C[key] and UDim2.new(1, -16, 0.5, -7) or UDim2.new(0, 2, 0.5, -7)
    dot.BackgroundColor3 = Color3.new(1,1,1); dot.BorderSizePixel = 0
    Instance.new("UICorner", dot).CornerRadius = UDim.new(1, 0)

    local btn = Instance.new("TextButton", f)
    btn.Size = UDim2.new(1,0,1,0); btn.BackgroundTransparency = 1; btn.Text = ""

    btn.MouseButton1Click:Connect(function()
        C[key] = not C[key]
        local s = C[key]
        TweenService:Create(pill, TweenInfo.new(0.2), {BackgroundColor3 = s and THEME.on or THEME.off}):Play()
        TweenService:Create(dot, TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Position = s and UDim2.new(1, -16, 0.5, -7) or UDim2.new(0, 2, 0.5, -7)}):Play()
        if callback then callback(s) end
    end)
end

-- ═══════════════════════════════════════════════════════════════
-- BUILD MENU
-- ═══════════════════════════════════════════════════════════════
Sec(Col1, "🗡️ COMBAT")
Tog(Col1, "Auto Parry (Timing)", "autoParry")
Tog(Col1, "Combat Aimbot (Lock-on)", "combatAimbot")
Tog(Col1, "Hitbox Expander (6x)", "hitboxExpander", function(state)
    if not state then
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LP and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                p.Character.HumanoidRootPart.Size = Vector3.new(2, 2, 1)
                p.Character.HumanoidRootPart.Transparency = 1
            end
        end
    end
end)

Sec(Col1, "🏃 MOVEMENT")
Tog(Col1, "Speed Hack (Safe)", "speedHack")
Tog(Col1, "Infinite Jump", "infJump")
Tog(Col1, "Noclip", "noClip")

Sec(Col2, "👁️ VISUALS")
Tog(Col2, "Player ESP", "esp")
Tog(Col2, "Chams (Wallhack)", "chams")
Tog(Col2, "Fullbright", "fullbright")

Sec(Col2, "⚙️ SETTINGS")
local kbl = Instance.new("TextLabel", Col2)
kbl.Size = UDim2.new(1, 0, 0, 30); kbl.BackgroundTransparency = 1; kbl.LayoutOrder = 99
kbl.Text = "Toggle Menu: Right Ctrl\nStance Key: T"; kbl.TextColor3 = THEME.text_dim
kbl.Font = Enum.Font.Code; kbl.TextSize = 10; kbl.TextXAlignment = Enum.TextXAlignment.Left

Sec(Col2, "🔧 TOOLS")

-- Remote Spy Button
local spyBtn = Instance.new("TextButton", Col2)
spyBtn.Size = UDim2.new(1, -5, 0, 34); spyBtn.BackgroundColor3 = THEME.header; spyBtn.BorderSizePixel = 0
spyBtn.Text = "  🔍 Spy Combat Remotes (F9)"; spyBtn.TextColor3 = Color3.new(1,1,1)
spyBtn.Font = Enum.Font.GothamBold; spyBtn.TextSize = 11; spyBtn.TextXAlignment = Enum.TextXAlignment.Left
Instance.new("UICorner", spyBtn).CornerRadius = UDim.new(0, 6)
ord2 = ord2 + 1; spyBtn.LayoutOrder = ord2

spyBtn.MouseButton1Click:Connect(function()
    print("\n⛩️ ═══════ GAKURAN REMOTE SPY ═══════")
    print("📁 ReplicatedStorage:")
    for _, obj in ipairs(game:GetService("ReplicatedStorage"):GetDescendants()) do
        if obj:IsA("RemoteEvent") or obj:IsA("RemoteFunction") then
            print("  ⚡ [" .. obj.ClassName .. "] " .. obj:GetFullName())
        end
    end
    if LP.Character then
        print("\n📁 Character Remotes:")
        for _, obj in ipairs(LP.Character:GetDescendants()) do
            if obj:IsA("RemoteEvent") or obj:IsA("RemoteFunction") then
                print("  ⚡ [" .. obj.ClassName .. "] " .. obj:GetFullName())
            end
        end
    end
    local tf = Workspace:FindFirstChild("Combat") or Workspace:FindFirstChild("Remotes")
    if tf then
        print("\n📁 Workspace Combat:")
        for _, obj in ipairs(tf:GetDescendants()) do
            if obj:IsA("RemoteEvent") or obj:IsA("RemoteFunction") then
                print("  ⚡ [" .. obj.ClassName .. "] " .. obj:GetFullName())
            end
        end
    end
    print("═══════════════════════════════════\n")
    print("⚠️ AGORA APERTA O BOTÃO DE BLOCK/PARRY MANUALMENTE 1x")
    print("⚠️ E VÊ QUAL REMOTE APARECEU NO CONSOLE")
    print("⚠️ DEPOIS COLOCA O NOME NA VARIÁVEL PARRY_REMOTE_NAME")
end)

-- Re-cache Remotes
local cacheBtn = Instance.new("TextButton", Col2)
cacheBtn.Size = UDim2.new(1, -5, 0, 34); cacheBtn.BackgroundColor3 = Color3.fromRGB(40, 30, 35); cacheBtn.BorderSizePixel = 0
cacheBtn.Text = "  🔄 Re-Cache Combat Remotes"; cacheBtn.TextColor3 = THEME.text
cacheBtn.Font = Enum.Font.GothamBold; cacheBtn.TextSize = 11; cacheBtn.TextXAlignment = Enum.TextXAlignment.Left
Instance.new("UICorner", cacheBtn).CornerRadius = UDim.new(0, 6)
ord2 = ord2 + 1; cacheBtn.LayoutOrder = ord2
cacheBtn.MouseButton1Click:Connect(function()
    CacheCombatRemotes()
    print("✓ Cached " .. #CombatRemotes .. " combat remotes")
end)

-- ═══════════════════════════════════════════════════════════════
-- ENGINE 1: COMBAT (Auto Parry V3 — 5 Methods + Remote Spy)
--
-- COMO USAR: Liga "Auto Parry" e luta normalmente.
-- Se não funcionar automaticamente, usa o botão "Spy Combat 
-- Remotes" na coluna direita — ele printa TODOS os remotes.
-- Depois aperta o block/parry manual 1x pra ver qual remote
-- aparece no console (F9). Aí coloca o nome/path exato na
-- variável PARRY_REMOTE_NAME abaixo.
-- ═══════════════════════════════════════════════════════════════

-- ⚠️ SE O AUTO PARRY NÃO FUNCIONAR DE PRIMEIRA:
-- Rode o Remote Spy, veja o nome do remote de block, e coloque aqui:
local PARRY_REMOTE_NAME = nil -- ex: "Block" ou "Combat" ou "Parry"
-- Se nil, o script tenta adivinhar automaticamente.

local function GetClosestTarget(range)
    local bestTarget, bestDist = nil, range
    if not LP.Character or not LP.Character:FindFirstChild("HumanoidRootPart") then return nil, 999 end
    local myRoot = LP.Character.HumanoidRootPart
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LP and p.Character then
            local hum = p.Character:FindFirstChildOfClass("Humanoid")
            local root = p.Character:FindFirstChild("HumanoidRootPart")
            if hum and hum.Health > 0 and root then
                local dist = (myRoot.Position - root.Position).Magnitude
                if dist < bestDist then bestDist = dist; bestTarget = p.Character end
            end
        end
    end
    return bestTarget, bestDist
end

-- Attack animation keywords
local ATK_KEYS = {"punch","kick","attack","swing","combo","m1","m2","heavy","hit","strike",
    "slam","hook","jab","upper","smash","fist","elbow","knee","light","finisher","rush","barrage"}

local function IsAttackAnim(track)
    local name = track.Name:lower()
    local id = tostring(track.Animation and track.Animation.AnimationId or ""):lower()
    for _, k in ipairs(ATK_KEYS) do
        if name:find(k) or id:find(k) then return true end
    end
    if (track.Priority == Enum.AnimationPriority.Action or track.Priority == Enum.AnimationPriority.Action4
        or track.Priority == Enum.AnimationPriority.Action3 or track.Priority == Enum.AnimationPriority.Action2) then
        if track.Length > 0 and track.Length < 2 then
            if not name:find("walk") and not name:find("run") and not name:find("idle")
               and not name:find("jump") and not name:find("fall") and not name:find("land") then
                return true
            end
        end
    end
    return false
end

-- Cache combat remotes once
local CombatRemotes = {}
local function CacheCombatRemotes()
    CombatRemotes = {}
    local searchTerms = {"block","parry","defend","guard","combat","fight","action","input","attack","stance"}
    
    local function scanSource(source)
        for _, obj in ipairs(source:GetDescendants()) do
            if obj:IsA("RemoteEvent") or obj:IsA("RemoteFunction") then
                local n = obj.Name:lower()
                for _, term in ipairs(searchTerms) do
                    if n:find(term) then
                        table.insert(CombatRemotes, obj)
                        break
                    end
                end
                -- If user specified exact name
                if PARRY_REMOTE_NAME and obj.Name:lower():find(PARRY_REMOTE_NAME:lower()) then
                    table.insert(CombatRemotes, 1, obj) -- priority
                end
            end
        end
    end
    
    scanSource(game:GetService("ReplicatedStorage"))
    -- Also check player's character for local remotes
    if LP.Character then
        for _, obj in ipairs(LP.Character:GetDescendants()) do
            if obj:IsA("RemoteEvent") or obj:IsA("RemoteFunction") then
                table.insert(CombatRemotes, obj)
            end
        end
    end
end
task.spawn(function() task.wait(1); CacheCombatRemotes() end)

-- Parry cooldown
local parryCooldown = false
local PARRY_CD = 0.3

local function FireParry()
    if parryCooldown then return end
    parryCooldown = true
    
    -- METHOD 1: Set Humanoid/Character attributes (many parry games use this)
    pcall(function()
        local hum = LP.Character and LP.Character:FindFirstChildOfClass("Humanoid")
        if hum then
            hum:SetAttribute("Parrying", true)
            hum:SetAttribute("Blocking", true)
            hum:SetAttribute("isBlocking", true)
            hum:SetAttribute("IsParrying", true)
            hum:SetAttribute("Block", true)
            task.delay(0.25, function()
                pcall(function()
                    hum:SetAttribute("Parrying", false)
                    hum:SetAttribute("Blocking", false)
                    hum:SetAttribute("isBlocking", false)
                    hum:SetAttribute("IsParrying", false)
                    hum:SetAttribute("Block", false)
                end)
            end)
        end
    end)
    
    -- METHOD 2: VirtualInputManager (multiple keys: F, E, Q, mouse right)
    pcall(function()
        local VIM = game:GetService("VirtualInputManager")
        -- Try parry key
        VIM:SendKeyEvent(true, C.parryKey, false, game)
        task.delay(0.15, function() pcall(function() VIM:SendKeyEvent(false, C.parryKey, false, game) end) end)
        -- Also try mouse right click (some games use RMB for block)
        VIM:SendMouseButtonEvent(0, 0, 1, true, game, 0) -- right click down
        task.delay(0.15, function() pcall(function() VIM:SendMouseButtonEvent(0, 0, 1, false, game, 0) end) end)
    end)
    
    -- METHOD 3: Fire cached combat remotes
    for _, remote in ipairs(CombatRemotes) do
        pcall(function()
            if remote:IsA("RemoteEvent") then
                remote:FireServer()
                remote:FireServer("Block")
                remote:FireServer("Parry")
                remote:FireServer(true)
                remote:FireServer("block")
                remote:FireServer({Action = "Block"})
                remote:FireServer({action = "block"})
                remote:FireServer({Type = "Parry"})
            elseif remote:IsA("RemoteFunction") then
                pcall(function() remote:InvokeServer() end)
                pcall(function() remote:InvokeServer("Block") end)
                pcall(function() remote:InvokeServer("Parry") end)
                pcall(function() remote:InvokeServer(true) end)
            end
        end)
    end
    
    -- METHOD 4: Fire ALL remotes in ReplicatedStorage with "block"/"parry" type args
    pcall(function()
        -- Universal approach: find the main combat handler remote
        for _, obj in ipairs(game:GetService("ReplicatedStorage"):GetDescendants()) do
            if obj:IsA("RemoteEvent") then
                local n = obj.Name:lower()
                -- Try common combat framework remote names
                if n == "action" or n == "input" or n == "combat" or n == "event" 
                   or n == "main" or n == "handler" or n == "request" then
                    pcall(function() obj:FireServer("Block") end)
                    pcall(function() obj:FireServer("Parry") end)
                    pcall(function() obj:FireServer({Type = "Block"}) end)
                    pcall(function() obj:FireServer("block", true) end)
                end
            end
        end
    end)
    
    -- METHOD 5: Simulate keypress via UserInputService workaround
    pcall(function()
        -- Some games read GetKeysPressed; we inject via internal
        local uis = game:GetService("UserInputService")
        -- This is a fallback that works on some executors
        if keypress then
            keypress(C.parryKey.Value)
            task.delay(0.15, function() if keyrelease then keyrelease(C.parryKey.Value) end end)
        end
    end)
    
    task.delay(PARRY_CD, function() parryCooldown = false end)
end

-- ANIMATION LISTENER (connect to all players)
local parryConns = {}

local function ConnectParryListener(player)
    if parryConns[player] then return end
    parryConns[player] = {}
    
    local function onChar(character)
        local humanoid = character:WaitForChild("Humanoid", 5)
        if not humanoid then return end
        
        local conn = humanoid.AnimationPlayed:Connect(function(track)
            if not C.autoParry then return end
            if not LP.Character or not LP.Character:FindFirstChild("HumanoidRootPart") then return end
            
            if IsAttackAnim(track) then
                local enemyRoot = character:FindFirstChild("HumanoidRootPart")
                local myRoot = LP.Character:FindFirstChild("HumanoidRootPart")
                if enemyRoot and myRoot then
                    local dist = (myRoot.Position - enemyRoot.Position).Magnitude
                    if dist <= C.parryRange then
                        local toMe = (myRoot.Position - enemyRoot.Position).Unit
                        local dot = toMe:Dot(enemyRoot.CFrame.LookVector)
                        if dot > 0.2 then FireParry() end
                    end
                end
            end
        end)
        table.insert(parryConns[player], conn)
    end
    
    if player.Character then task.spawn(function() onChar(player.Character) end) end
    local c = player.CharacterAdded:Connect(function(ch) task.wait(0.5); onChar(ch) end)
    table.insert(parryConns[player], c)
end

for _, p in ipairs(Players:GetPlayers()) do if p ~= LP then ConnectParryListener(p) end end
Players.PlayerAdded:Connect(function(p) if p ~= LP then task.wait(1); ConnectParryListener(p) end end)
Players.PlayerRemoving:Connect(function(p)
    if parryConns[p] then
        for _, c in ipairs(parryConns[p]) do pcall(function() c:Disconnect() end) end
        parryConns[p] = nil
    end
end)

-- HITBOX DETECTION LOOP
task.spawn(function()
    while SG and SG.Parent do
        if C.autoParry then
            pcall(function()
                if not LP.Character or not LP.Character:FindFirstChild("HumanoidRootPart") then return end
                local myRoot = LP.Character.HumanoidRootPart
                for _, obj in ipairs(Workspace:GetDescendants()) do
                    if not C.autoParry then break end
                    local n = obj.Name:lower()
                    if (n:find("hitbox") or n:find("hurtbox") or n:find("attackbox") or n:find("hit_")) and obj:IsA("BasePart") then
                        local owner = obj.Parent
                        while owner and owner ~= Workspace do
                            if owner:FindFirstChildOfClass("Humanoid") then break end
                            owner = owner.Parent
                        end
                        if owner and owner ~= LP.Character then
                            local dist = (myRoot.Position - obj.Position).Magnitude
                            if dist < C.parryRange + 5 then FireParry() end
                        end
                    end
                end
            end)
        end
        task.wait(0.03) -- faster scan
    end
end)

-- VELOCITY FALLBACK + AIMBOT
local VirtualInput = game:GetService("VirtualInputManager")

RunService.Heartbeat:Connect(function()
    if not LP.Character or not LP.Character:FindFirstChild("HumanoidRootPart") then return end
    local myRoot = LP.Character.HumanoidRootPart

    if C.autoParry then
        local target, dist = GetClosestTarget(C.parryRange)
        if target and target:FindFirstChild("HumanoidRootPart") and dist < C.parryRange then
            local enemyRoot = target.HumanoidRootPart
            local relVel = enemyRoot.AssemblyLinearVelocity - myRoot.AssemblyLinearVelocity
            local toMe = (myRoot.Position - enemyRoot.Position).Unit
            if relVel:Dot(toMe) > 20 then FireParry() end
        end
    end

    if C.combatAimbot then
        if UIS:IsMouseButtonPressed(Enum.UserInputType.MouseButton1) or UIS:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
            local target = GetClosestTarget(C.aimbotRange)
            if target and target:FindFirstChild("HumanoidRootPart") then
                local tPos = target.HumanoidRootPart.Position
                local mPos = myRoot.Position
                myRoot.CFrame = myRoot.CFrame:Lerp(CFrame.new(mPos, Vector3.new(tPos.X, mPos.Y, tPos.Z)), C.aimSmoothing)
            end
        end
    end
end)

-- ═══════════════════════════════════════════════════════════════
-- ENGINE 2: HITBOX EXPANDER
-- ═══════════════════════════════════════════════════════════════
RunService.Stepped:Connect(function()
    if C.hitboxExpander then
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LP and p.Character and p.Character:FindFirstChild("HumanoidRootPart") and p.Character:FindFirstChild("Humanoid") then
                if p.Character.Humanoid.Health > 0 then
                    local hrp = p.Character.HumanoidRootPart
                    hrp.Size = Vector3.new(C.hitboxSize, C.hitboxSize, C.hitboxSize)
                    hrp.Transparency = 0.8
                    hrp.BrickColor = BrickColor.new("Bright red")
                    hrp.Material = Enum.Material.Neon
                    hrp.CanCollide = false
                end
            end
        end
    end
end)

-- ═══════════════════════════════════════════════════════════════
-- ENGINE 3: MOVEMENT & MISC
-- ═══════════════════════════════════════════════════════════════
RunService.Stepped:Connect(function()
    if C.noClip and LP.Character then
        for _, p in ipairs(LP.Character:GetDescendants()) do
            if p:IsA("BasePart") then p.CanCollide = false end
        end
    end
end)

UIS.JumpRequest:Connect(function()
    if C.infJump and LP.Character then
        local h = LP.Character:FindFirstChildOfClass("Humanoid")
        if h then h:ChangeState(Enum.HumanoidStateType.Jumping) end
    end
end)

task.spawn(function()
    while SG and SG.Parent do
        pcall(function()
            local h = LP.Character and LP.Character:FindFirstChildOfClass("Humanoid")
            if h and C.speedHack then
                h.WalkSpeed = C.walkSpeed
            end
            if C.fullbright then
                Lighting.Brightness = 2
                Lighting.ClockTime = 14
                Lighting.GlobalShadows = false
            end
        end)
        task.wait(0.1)
    end
end)

-- ═══════════════════════════════════════════════════════════════
-- ENGINE 4: ESP & CHAMS
-- ═══════════════════════════════════════════════════════════════
local espObjects = {}

local function CreateESP(p)
    local hl = Instance.new("Highlight")
    hl.FillColor = THEME.accent
    hl.FillTransparency = 0.5
    hl.OutlineColor = Color3.new(1,1,1)
    hl.OutlineTransparency = 0.2
    
    local txt = Drawing.new("Text")
    txt.Size = 14
    txt.Center = true
    txt.Outline = true
    txt.Color = THEME.text
    
    espObjects[p] = {hl = hl, txt = txt}
end

RunService.RenderStepped:Connect(function()
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LP then
            local hasChar = p.Character and p.Character:FindFirstChild("HumanoidRootPart") and p.Character:FindFirstChild("Humanoid") and p.Character.Humanoid.Health > 0
            
            if hasChar and (C.esp or C.chams) then
                if not espObjects[p] then CreateESP(p) end
                local eo = espObjects[p]
                
                -- Chams
                if C.chams then
                    eo.hl.Parent = p.Character
                    eo.hl.Enabled = true
                else
                    eo.hl.Enabled = false
                end
                
                -- ESP Text
                if C.esp then
                    local hrp = p.Character.HumanoidRootPart
                    local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
                    if onScreen then
                        local dist = (Camera.CFrame.Position - hrp.Position).Magnitude
                        eo.txt.Position = Vector2.new(pos.X, pos.Y - 25)
                        eo.txt.Text = string.format("[%s] %dm", p.Name, math.floor(dist))
                        eo.txt.Visible = true
                    else
                        eo.txt.Visible = false
                    end
                else
                    eo.txt.Visible = false
                end
            elseif espObjects[p] then
                espObjects[p].hl.Enabled = false
                espObjects[p].txt.Visible = false
            end
        end
    end
end)

Players.PlayerRemoving:Connect(function(p)
    if espObjects[p] then
        espObjects[p].hl:Destroy()
        espObjects[p].txt:Remove()
        espObjects[p] = nil
    end
end)

-- ═══════════════════════════════════════════════════════════════
-- GLOBAL KILL & KEYBIND
-- ═══════════════════════════════════════════════════════════════
shared._GAKURAN_KILL = function()
    for k, v in pairs(C) do if typeof(v) == "boolean" then C[k] = false end end
    for p, eo in pairs(espObjects) do eo.hl:Destroy(); eo.txt:Remove() end
    espObjects = {}
end

UIS.InputBegan:Connect(function(i, g)
    if not g and i.KeyCode == C.uiToggle then
        MF.Visible = not MF.Visible
    end
end)

-- Intro Animation
MF.Size = UDim2.new(0, 0, 0, 0)
TweenService:Create(MF, TweenInfo.new(0.5, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Size = UDim2.new(0, 480, 0, 340)}):Play()

print("⛩️ GAKURAN OMNI-HUB LOADED ⛩️")
