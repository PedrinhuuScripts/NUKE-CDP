-- ============================================================
--  Dhz Scripts VIP — CDP Killer v5.1 (Anti-Kick Edition)
--  Timing randomizado para não ser detectado pelo anti-cheat
-- ============================================================

local Players           = game:GetService("Players")
local TweenService      = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local LocalPlayer = Players.LocalPlayer
local PlayerGui   = LocalPlayer:WaitForChild("PlayerGui")

-- ============================================================
--  CONFIG
-- ============================================================
local CONFIG = {
    KEYWORDS = {
        "cdp", "capacita", "patente", "capacidade",
        "cooldown", "cool_down", "timer", "tempo",
        "countdown", "recharge", "recarga", "espera",
        "habilidade", "skill", "ability", "poder",
        "delay", "interval", "remaining",
        "eb", "delta", "limite", "limit",
        "locked", "lock", "ready",
    },

    WHITELIST = {
        "coregui", "robloxgui", "chat", "screengui",
        "loadingscreen", "healthbar", "backpack", "camera",
    },

    -- Payloads do SetCDP — disparados UM POR VEZ com delay entre eles
    SETCDP_PAYLOADS = {
        { LocalPlayer.Name, 0     },
        { LocalPlayer.Name, false },
        { 0                       },
        { false                   },
    },

    -- Timing anti-detecção (segundos)
    START_DELAY   = 7.0,   -- espera antes do primeiro disparo
    PAYLOAD_DELAY = 1.8,   -- entre cada payload do SetCDP
    REMOTE_DELAY  = 2.5,   -- entre remotes diferentes
    LOOP_MIN      = 12.0,  -- intervalo mínimo do loop
    LOOP_MAX      = 20.0,  -- intervalo máximo do loop (randomizado)

    MAX_LOGS = 10,
}

-- Espera aleatória entre min e max segundos
local function jitter(min, max)
    task.wait(min + math.random() * (max - min))
end

-- ============================================================
--  UTILS
-- ============================================================
local Utils = {}

function Utils.matchesAny(str, list)
    local lower = string.lower(tostring(str))
    for _, kw in ipairs(list) do
        if string.find(lower, kw, 1, true) then return true end
    end
    return false
end

function Utils.isWhitelisted(obj)
    return Utils.matchesAny(obj:GetFullName(), CONFIG.WHITELIST)
end

function Utils.deepFind(root, name)
    return root:FindFirstChild(name, true)
end

-- ============================================================
--  LOG
-- ============================================================
local Log = { _e = {}, _f = nil }
function Log.init(f) Log._f = f end
function Log.add(t)
    table.insert(Log._e, t)
    if #Log._e > CONFIG.MAX_LOGS then table.remove(Log._e, 1) end
    if not Log._f then return end
    Log._f:ClearAllChildren()
    for i, v in ipairs(Log._e) do
        local l = Instance.new("TextLabel")
        l.Parent = Log._f
        l.BackgroundTransparency = 1
        l.Size = UDim2.new(1, -8, 0, 17)
        l.Position = UDim2.new(0, 4, 0, (i-1)*17)
        l.Font = Enum.Font.Code
        l.TextColor3 = Color3.fromRGB(200, 220, 255)
        l.Text = "› " .. v
        l.TextXAlignment = Enum.TextXAlignment.Left
        l.TextSize = 11
    end
end

-- ============================================================
--  UI
-- ============================================================
local function buildUI()
    local sg = Instance.new("ScreenGui")
    sg.Name = "DhzScripts_VIP"
    sg.Parent = PlayerGui
    sg.ResetOnSpawn = false

    local mf = Instance.new("Frame")
    mf.Parent = sg
    mf.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
    mf.Position = UDim2.new(0, 20, 0.5, -115)
    mf.Size = UDim2.new(0, 0, 0, 0)
    mf.Active = true
    mf.Draggable = true
    Instance.new("UICorner", mf).CornerRadius = UDim.new(0, 16)

    local stroke = Instance.new("UIStroke", mf)
    stroke.Color = Color3.fromRGB(100, 200, 255)
    stroke.Thickness = 2
    stroke.Transparency = 0.3

    local grad = Instance.new("UIGradient", mf)
    grad.Color = ColorSequence.new{
        ColorSequenceKeypoint.new(0, Color3.fromRGB(35, 40, 55)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(15, 20, 35)),
    }
    grad.Rotation = 135

    local function mkLabel(pos, size, text, color, font)
        local l = Instance.new("TextLabel")
        l.Parent = mf
        l.BackgroundTransparency = 1
        l.Position = pos
        l.Size = size
        l.Font = font or Enum.Font.Gotham
        l.Text = text
        l.TextColor3 = color
        l.TextScaled = true
        return l
    end

    local title = mkLabel(
        UDim2.new(0,15,0,12), UDim2.new(1,-30,0,24),
        "Dhz Scripts VIP", Color3.fromRGB(255,255,255), Enum.Font.GothamBold)
    title.TextStrokeTransparency = 0.7
    title.TextStrokeColor3 = Color3.fromRGB(100,200,255)
    local tg = Instance.new("UIGradient", title)
    tg.Color = ColorSequence.new{
        ColorSequenceKeypoint.new(0,   Color3.fromRGB(100,255,200)),
        ColorSequenceKeypoint.new(0.4, Color3.fromRGB(150,100,255)),
        ColorSequenceKeypoint.new(1,   Color3.fromRGB(255,150,100)),
    }
    tg.Rotation = 45

    mkLabel(UDim2.new(0,15,0,38), UDim2.new(1,-30,0,17),
        "CDP Killer — Anti-Kick v5.1",
        Color3.fromRGB(150,220,255), Enum.Font.GothamSemibold)

    local status = mkLabel(UDim2.new(0,15,0,58), UDim2.new(1,-30,0,20),
        "Status: Pronto", Color3.fromRGB(150,255,150))

    local lf = Instance.new("ScrollingFrame")
    lf.Parent = mf
    lf.BackgroundColor3 = Color3.fromRGB(22,22,30)
    lf.BorderSizePixel = 0
    lf.Position = UDim2.new(0,15,0,82)
    lf.Size = UDim2.new(1,-30,0,62)
    lf.ScrollBarThickness = 4
    lf.ScrollBarImageColor3 = Color3.fromRGB(100,200,255)
    lf.CanvasSize = UDim2.new(0,0,0,0)
    lf.AutomaticCanvasSize = Enum.AutomaticSize.Y
    Instance.new("UICorner", lf).CornerRadius = UDim.new(0,8)

    local function mkBtn(pos, size, text, color)
        local b = Instance.new("TextButton")
        b.Parent = mf
        b.BackgroundColor3 = color
        b.Position = pos
        b.Size = size
        b.Text = text
        b.TextColor3 = Color3.new(1,1,1)
        b.Font = Enum.Font.GothamBold
        b.TextScaled = true
        Instance.new("UICorner", b).CornerRadius = UDim.new(0,10)
        return b
    end

    local scanBtn = mkBtn(
        UDim2.new(0,15,1,-55), UDim2.new(0.46,0,0,38),
        "🔍 Procurar CDP", Color3.fromRGB(50,150,255))

    local killBtn = mkBtn(
        UDim2.new(0.52,0,1,-55), UDim2.new(0.47,-10,0,38),
        "☢️ ATIVAR NUKE", Color3.fromRGB(220,50,50))

    TweenService:Create(mf, TweenInfo.new(0.6, Enum.EasingStyle.Back),
        { Size = UDim2.new(0, 290, 0, 230) }):Play()

    return { mf=mf, status=status, lf=lf, scanBtn=scanBtn, killBtn=killBtn }
end

-- ============================================================
--  CORE — REMOTES (cada um com delay para não dar kick)
-- ============================================================
local function fireAllRemotes()
    -- 1. SetCDP via Knit — um payload por vez com delay
    pcall(function()
        local mod = ReplicatedStorage:FindFirstChild("Modules", true)
        if not mod then return end
        local res = mod:FindFirstChild("ReservedService", true)
        if not res then return end
        local rf = res:FindFirstChild("RF")
        if not rf then return end
        local setcdp = rf:FindFirstChild("SetCDP")
        if not setcdp then return end

        for i, payload in ipairs(CONFIG.SETCDP_PAYLOADS) do
            pcall(function() setcdp:InvokeServer(table.unpack(payload)) end)
            Log.add("📡 SetCDP " .. i .. "/" .. #CONFIG.SETCDP_PAYLOADS)
            if i < #CONFIG.SETCDP_PAYLOADS then
                task.wait(CONFIG.PAYLOAD_DELAY + math.random() * 0.5)
            end
        end
    end)

    task.wait(CONFIG.REMOTE_DELAY + math.random() * 1.0)

    -- 2. CDPEvent
    pcall(function()
        local ev = Utils.deepFind(ReplicatedStorage, "CDPEvent")
        if ev and ev:IsA("RemoteEvent") then
            ev:FireServer(false, 0)
            Log.add("📡 CDPEvent ok")
        end
    end)

    task.wait(CONFIG.REMOTE_DELAY + math.random() * 1.0)

    -- 3. CDPFunction
    pcall(function()
        local fn = Utils.deepFind(ReplicatedStorage, "CDPFunction")
        if fn and fn:IsA("RemoteFunction") then
            fn:InvokeServer(0)
            Log.add("📡 CDPFunction ok")
        end
    end)

    task.wait(CONFIG.REMOTE_DELAY + math.random() * 1.0)

    -- 4. Remotes.Events.CDP.*
    pcall(function()
        local folder = ReplicatedStorage
        for _, part in ipairs({"Remotes","Events","CDP"}) do
            folder = folder:FindFirstChild(part)
            if not folder then return end
        end
        for _, obj in ipairs(folder:GetDescendants()) do
            if obj:IsA("RemoteEvent") then
                pcall(function() obj:FireServer(0, LocalPlayer) end)
            elseif obj:IsA("RemoteFunction") then
                pcall(function() obj:InvokeServer(0) end)
            end
            task.wait(0.6)
        end
        Log.add("📡 Events.CDP ok")
    end)
end

-- ============================================================
--  PURGE (limpa objetos locais)
-- ============================================================
local function purgeObjects()
    local z, h, n = 0, 0, 0
    local roots = { PlayerGui, LocalPlayer }
    if LocalPlayer.Character then table.insert(roots, LocalPlayer.Character) end

    for _, root in ipairs(roots) do
        for _, obj in ipairs(root:GetDescendants()) do
            if not obj or not obj.Parent then continue end
            if Utils.isWhitelisted(obj) then continue end

            local name  = string.lower(obj.Name)
            local class = obj.ClassName

            if (class == "NumberValue" or class == "IntValue")
            and Utils.matchesAny(name, CONFIG.KEYWORDS) then
                pcall(function() obj.Value = 0; z += 1 end)

            elseif class == "BoolValue"
            and Utils.matchesAny(name, CONFIG.KEYWORDS) then
                pcall(function() obj.Value = false; z += 1 end)

            elseif obj:IsA("GuiObject")
            and Utils.matchesAny(name, CONFIG.KEYWORDS) then
                if #obj:GetChildren() == 0 then
                    pcall(function() obj:Destroy(); n += 1 end)
                else
                    pcall(function() obj.Visible = false; h += 1 end)
                end

            elseif (obj:IsA("TextLabel") or obj:IsA("TextButton")) then
                local txt = string.lower(obj.Text or "")
                if Utils.matchesAny(txt, CONFIG.KEYWORDS) then
                    pcall(function() obj.Visible = false; h += 1 end)
                end
            end
        end
    end
    return z, h, n
end

-- ============================================================
--  SCAN
-- ============================================================
local function scan()
    Log.add("🔍 Varrendo...")
    local count = 0
    local roots = { PlayerGui, LocalPlayer }
    if LocalPlayer.Character then table.insert(roots, LocalPlayer.Character) end
    for _, root in ipairs(roots) do
        for _, obj in ipairs(root:GetDescendants()) do
            if Utils.matchesAny(string.lower(obj.Name), CONFIG.KEYWORDS) then
                count += 1
            end
        end
    end
    for _, name in ipairs({"SetCDP","CDPEvent","CDPFunction"}) do
        if Utils.deepFind(ReplicatedStorage, name) then
            Log.add("📡 Achado: " .. name)
        end
    end
    Log.add("🎯 Suspeitos: " .. count)
    return count
end

-- ============================================================
--  NUKE (toggle)
-- ============================================================
local nukeActive = false
local nukeConn   = nil

local function stopNuke(ui)
    nukeActive = false
    if nukeConn then nukeConn:Disconnect(); nukeConn = nil end
    ui.killBtn.Text             = "☢️ ATIVAR NUKE"
    ui.killBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
    ui.status.Text              = "Status: Nuke desativado"
    ui.status.TextColor3        = Color3.fromRGB(150, 255, 150)
    Log.add("🛑 Nuke desativado")
end

local function startNuke(ui)
    if nukeActive then stopNuke(ui); return end

    nukeActive = true
    ui.killBtn.Text             = "🛑 PARAR NUKE"
    ui.killBtn.BackgroundColor3 = Color3.fromRGB(120, 30, 30)
    ui.status.Text              = "Status: Aguardando " .. CONFIG.START_DELAY .. "s..."
    ui.status.TextColor3        = Color3.fromRGB(255, 200, 60)
    Log.add("⏳ Esperando " .. CONFIG.START_DELAY .. "s (anti-kick)...")

    -- Intercepta CDPs criados em tempo real (só local, sem rede)
    nukeConn = game.DescendantAdded:Connect(function(obj)
        if not nukeActive then return end
        local name = string.lower(obj.Name)
        if Utils.matchesAny(name, CONFIG.KEYWORDS) then
            pcall(function()
                if obj:IsA("NumberValue") or obj:IsA("IntValue") then
                    obj.Value = 0
                elseif obj:IsA("BoolValue") then
                    obj.Value = false
                elseif obj:IsA("GuiObject") and #obj:GetChildren() == 0 then
                    obj:Destroy()
                end
            end)
        end
    end)

    task.spawn(function()
        -- Delay inicial — deixa o servidor estabilizar
        task.wait(CONFIG.START_DELAY)
        if not nukeActive then return end

        Log.add("☢️ Nuke iniciando!")
        ui.status.Text       = "Status: NUKE ATIVO ☢️"
        ui.status.TextColor3 = Color3.fromRGB(255, 60, 60)

        while nukeActive do
            -- Purge local primeiro (seguro, sem rede)
            local z, h, n = purgeObjects()

            -- Disparo de remotes (com delays internos entre cada um)
            fireAllRemotes()

            local total = z + h + n
            if total > 0 then
                Log.add(string.format("✅ Zerados:%d Ocultos:%d Del:%d", z, h, n))
            end

            -- Espera randomizada — evita padrão de timing detectável
            local waitTime = CONFIG.LOOP_MIN + math.random() * (CONFIG.LOOP_MAX - CONFIG.LOOP_MIN)
            Log.add(string.format("💤 Próximo: %.0fs", waitTime))
            ui.status.Text = string.format("Status: Nuke ativo — %.0fs ☢️", waitTime)
            task.wait(waitTime)
        end
    end)
end

-- ============================================================
--  PERSISTÊNCIA
-- ============================================================
local SCRIPT_URL = "https://raw.githubusercontent.com/PedrinhuuScripts/NUKE-CDP/refs/heads/main/README.md"

local function setupPersistence()
    local qfunc = (typeof(queue_on_teleport) == "function" and queue_on_teleport)
        or (syn    and syn.queue_on_teleport)
        or (fluxus and fluxus.queue_on_teleport)
        or (celery and celery.queue_on_teleport)

    if not qfunc then
        Log.add("⚠️ Sem queue_on_teleport")
        return false
    end

    local code = string.format([[
        getgenv().DhzAutoStart = true
        pcall(function()
            loadstring(game:HttpGet("%s"))()
        end)
    ]], SCRIPT_URL)

    return pcall(function() qfunc(code) end)
end

-- ============================================================
--  INICIALIZAÇÃO
-- ============================================================
local ui = buildUI()
Log.init(ui.lf)

local persisted = setupPersistence()

ui.scanBtn.MouseButton1Click:Connect(function()
    ui.scanBtn.BackgroundColor3 = Color3.fromRGB(80,180,255)
    task.wait(0.15)
    ui.scanBtn.BackgroundColor3 = Color3.fromRGB(50,150,255)
    local c = scan()
    ui.status.Text       = c > 0 and ("Status: "..c.." suspeitos") or "Status: Nenhum CDP"
    ui.status.TextColor3 = c > 0 and Color3.fromRGB(255,200,80) or Color3.fromRGB(180,180,180)
end)

ui.killBtn.MouseButton1Click:Connect(function()
    startNuke(ui)
end)

-- Auto-start após teleport
local isAuto = typeof(getgenv) == "function" and getgenv().DhzAutoStart == true
if isAuto then
    if typeof(getgenv) == "function" then getgenv().DhzAutoStart = false end
    Log.add("🔄 Voltando de teleport...")
    task.spawn(function()
        task.wait(5)
        startNuke(ui)
    end)
else
    Log.add("CDP Killer v5.1 carregado!")
    Log.add(persisted and "✅ Persistência ativa" or "⚠️ Sem persistência")
    Log.add("Clique ☢️ para ativar.")
    ui.status.Text = "Status: Pronto"
end
