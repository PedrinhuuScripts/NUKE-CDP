-- ============================================================
--  Dhz Scripts VIP — CDP Killer v5.0
--  Estrutura organizada + técnicas do Nuke Edition
-- ============================================================

local Players           = game:GetService("Players")
local TweenService      = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local LocalPlayer = Players.LocalPlayer
local PlayerGui   = LocalPlayer:WaitForChild("PlayerGui")

-- ============================================================
--  CONFIG CENTRAL
-- ============================================================
local CONFIG = {
    -- Palavras-chave para detectar CDPs
    KEYWORDS = {
        "cdp", "capacita", "patente", "capacidade",
        "cooldown", "cool_down", "timer", "tempo",
        "countdown", "recharge", "recarga", "espera",
        "habilidade", "skill", "ability", "poder",
        "delay", "wait", "interval", "remaining",
        "eb", "delta", "limite", "limit",
        "locked", "lock", "ready",
    },

    -- Objetos que NUNCA devem ser tocados
    WHITELIST = {
        "coregui", "robloxgui", "chat", "playergui_root",
        "screengui", "loadingscreen", "healthbar",
        "backpack", "camera",
    },

    -- Argumentos testados para o SetCDP (o jogo pode aceitar variações)
    -- Cada entrada é disparada em sequência
    SETCDP_PAYLOADS = {
        { LocalPlayer.Name, 0     },
        { LocalPlayer.Name, false },
        { 0                       },
        { false                   },
        { LocalPlayer.Name, 0, false },
    },

    -- Intervalo do loop de ataque contínuo (segundos)
    LOOP_INTERVAL = 0.5,

    -- Máximo de entradas no log
    MAX_LOGS = 10,
}

-- ============================================================
--  UTILS
-- ============================================================
local Utils = {}

function Utils.matchesAny(str, list)
    local lower = string.lower(tostring(str))
    for _, kw in ipairs(list) do
        if string.find(lower, kw, 1, true) then
            return true, kw
        end
    end
    return false
end

function Utils.isWhitelisted(obj)
    local path = string.lower(obj:GetFullName())
    return Utils.matchesAny(path, CONFIG.WHITELIST)
end

function Utils.isCDPSuspect(obj)
    if Utils.isWhitelisted(obj) then return false end
    local score = 0
    local name  = string.lower(obj.Name)
    local class = obj.ClassName

    if Utils.matchesAny(name, CONFIG.KEYWORDS) then score += 2 end

    if class == "NumberValue" or class == "IntValue"
    or class == "BoolValue"   or class == "StringValue" then
        score += 1
    end

    if obj:IsA("TextLabel") or obj:IsA("TextButton") then
        local txt = string.lower(obj.Text or "")
        if Utils.matchesAny(txt, CONFIG.KEYWORDS) then score += 1 end
    end

    if (class == "NumberValue" or class == "IntValue") and obj.Value > 0 then
        score += 1
    end

    return score >= 2
end

-- Resolve um caminho dentro de um root com FindFirstChild recursivo
function Utils.deepFind(root, name)
    return root:FindFirstChild(name, true)
end

-- ============================================================
--  LOG
-- ============================================================
local Log = { _entries = {}, _frame = nil }

function Log.init(frame) Log._frame = frame end

function Log.add(text)
    table.insert(Log._entries, text)
    if #Log._entries > CONFIG.MAX_LOGS then
        table.remove(Log._entries, 1)
    end
    if not Log._frame then return end
    Log._frame:ClearAllChildren()
    for i, entry in ipairs(Log._entries) do
        local lbl = Instance.new("TextLabel")
        lbl.Parent               = Log._frame
        lbl.BackgroundTransparency = 1
        lbl.Size                 = UDim2.new(1, -8, 0, 17)
        lbl.Position             = UDim2.new(0, 4, 0, (i-1)*17)
        lbl.Font                 = Enum.Font.Code
        lbl.TextColor3           = Color3.fromRGB(200, 220, 255)
        lbl.Text                 = "› " .. entry
        lbl.TextXAlignment       = Enum.TextXAlignment.Left
        lbl.TextSize             = 11
    end
end

-- ============================================================
--  UI
-- ============================================================
local function buildUI()
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name         = "DhzScripts_VIP"
    ScreenGui.Parent       = PlayerGui
    ScreenGui.ResetOnSpawn = false

    local MainFrame = Instance.new("Frame")
    MainFrame.Parent          = ScreenGui
    MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
    MainFrame.Position        = UDim2.new(0, 20, 0.5, -115)
    MainFrame.Size            = UDim2.new(0, 0, 0, 0)
    MainFrame.Active          = true
    MainFrame.Draggable       = true
    Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 16)

    local Stroke = Instance.new("UIStroke", MainFrame)
    Stroke.Color        = Color3.fromRGB(100, 200, 255)
    Stroke.Thickness    = 2
    Stroke.Transparency = 0.3

    local Gradient = Instance.new("UIGradient", MainFrame)
    Gradient.Color = ColorSequence.new{
        ColorSequenceKeypoint.new(0, Color3.fromRGB(35, 40, 55)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(15, 20, 35)),
    }
    Gradient.Rotation = 135

    local Title = Instance.new("TextLabel")
    Title.Parent               = MainFrame
    Title.BackgroundTransparency = 1
    Title.Position             = UDim2.new(0, 15, 0, 12)
    Title.Size                 = UDim2.new(1, -30, 0, 24)
    Title.Font                 = Enum.Font.GothamBold
    Title.Text                 = "Dhz Scripts VIP"
    Title.TextColor3           = Color3.fromRGB(255, 255, 255)
    Title.TextScaled           = true
    Title.TextStrokeTransparency = 0.7
    Title.TextStrokeColor3     = Color3.fromRGB(100, 200, 255)
    local TG = Instance.new("UIGradient", Title)
    TG.Color = ColorSequence.new{
        ColorSequenceKeypoint.new(0,   Color3.fromRGB(100, 255, 200)),
        ColorSequenceKeypoint.new(0.4, Color3.fromRGB(150, 100, 255)),
        ColorSequenceKeypoint.new(1,   Color3.fromRGB(255, 150, 100)),
    }
    TG.Rotation = 45

    local Subtitle = Instance.new("TextLabel")
    Subtitle.Parent               = MainFrame
    Subtitle.BackgroundTransparency = 1
    Subtitle.Position             = UDim2.new(0, 15, 0, 38)
    Subtitle.Size                 = UDim2.new(1, -30, 0, 17)
    Subtitle.Font                 = Enum.Font.GothamSemibold
    Subtitle.Text                 = "CDP Killer — Bypass Edition"
    Subtitle.TextColor3           = Color3.fromRGB(150, 220, 255)
    Subtitle.TextScaled           = true

    local Status = Instance.new("TextLabel")
    Status.Parent               = MainFrame
    Status.BackgroundTransparency = 1
    Status.Position             = UDim2.new(0, 15, 0, 58)
    Status.Size                 = UDim2.new(1, -30, 0, 20)
    Status.Font                 = Enum.Font.Gotham
    Status.Text                 = "Status: Pronto"
    Status.TextColor3           = Color3.fromRGB(150, 255, 150)
    Status.TextScaled           = true

    local LogFrame = Instance.new("ScrollingFrame")
    LogFrame.Parent               = MainFrame
    LogFrame.BackgroundColor3     = Color3.fromRGB(22, 22, 30)
    LogFrame.BorderSizePixel      = 0
    LogFrame.Position             = UDim2.new(0, 15, 0, 82)
    LogFrame.Size                 = UDim2.new(1, -30, 0, 62)
    LogFrame.ScrollBarThickness   = 4
    LogFrame.ScrollBarImageColor3 = Color3.fromRGB(100, 200, 255)
    LogFrame.CanvasSize           = UDim2.new(0,0,0,0)
    LogFrame.AutomaticCanvasSize  = Enum.AutomaticSize.Y
    Instance.new("UICorner", LogFrame).CornerRadius = UDim.new(0, 8)

    -- Botão SCAN
    local ScanBtn = Instance.new("TextButton")
    ScanBtn.Parent           = MainFrame
    ScanBtn.BackgroundColor3 = Color3.fromRGB(50, 150, 255)
    ScanBtn.Position         = UDim2.new(0, 15, 1, -55)
    ScanBtn.Size             = UDim2.new(0.46, 0, 0, 38)
    ScanBtn.Text             = "🔍 Procurar CDP"
    ScanBtn.TextColor3       = Color3.new(1,1,1)
    ScanBtn.Font             = Enum.Font.GothamBold
    ScanBtn.TextScaled       = true
    Instance.new("UICorner", ScanBtn).CornerRadius = UDim.new(0, 10)

    -- Botão NUKE (toggle)
    local KillBtn = Instance.new("TextButton")
    KillBtn.Parent           = MainFrame
    KillBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
    KillBtn.Position         = UDim2.new(0.52, 0, 1, -55)
    KillBtn.Size             = UDim2.new(0.47, -10, 0, 38)
    KillBtn.Text             = "☢️ ATIVAR NUKE"
    KillBtn.TextColor3       = Color3.new(1,1,1)
    KillBtn.Font             = Enum.Font.GothamBold
    KillBtn.TextScaled       = true
    Instance.new("UICorner", KillBtn).CornerRadius = UDim.new(0, 10)

    TweenService:Create(MainFrame, TweenInfo.new(0.6, Enum.EasingStyle.Back),
        { Size = UDim2.new(0, 290, 0, 230) }):Play()

    return { MainFrame=MainFrame, Status=Status, LogFrame=LogFrame,
             ScanBtn=ScanBtn, KillBtn=KillBtn }
end

-- ============================================================
--  CORE — REMOTES
--  Tenta todos os payloads conhecidos no SetCDP
-- ============================================================
local function fireAllRemotes()
    -- ── SetCDP via Knit ReservedService ───────────────────────────
    pcall(function()
        local modulesRoot = ReplicatedStorage:FindFirstChild("Modules", true)
        if modulesRoot then
            local reserved = modulesRoot:FindFirstChild("ReservedService", true)
            if reserved then
                local rf = reserved:FindFirstChild("RF")
                if rf then
                    local setcdp = rf:FindFirstChild("SetCDP")
                    if setcdp then
                        for _, payload in ipairs(CONFIG.SETCDP_PAYLOADS) do
                            pcall(function()
                                setcdp:InvokeServer(table.unpack(payload))
                            end)
                        end
                        Log.add("📡 SetCDP disparado (" .. #CONFIG.SETCDP_PAYLOADS .. " payloads)")
                    end
                end
            end
        end
    end)

    -- ── CDPEvent (raiz do ReplicatedStorage) ──────────────────────
    pcall(function()
        local ev = Utils.deepFind(ReplicatedStorage, "CDPEvent")
        if ev and ev:IsA("RemoteEvent") then
            ev:FireServer(false, 0)
            ev:FireServer(0, LocalPlayer)
            Log.add("📡 CDPEvent disparado")
        end
    end)

    -- ── CDPFunction / RemoteFunction ──────────────────────────────
    pcall(function()
        local fn = Utils.deepFind(ReplicatedStorage, "CDPFunction")
        if fn and fn:IsA("RemoteFunction") then
            fn:InvokeServer(0)
            fn:InvokeServer(false)
            Log.add("📡 CDPFunction invocado")
        end
    end)

    -- ── Remotes.Events.CDP.CDPFunction ────────────────────────────
    pcall(function()
        local remotes = ReplicatedStorage:FindFirstChild("Remotes")
        if remotes then
            local events = remotes:FindFirstChild("Events")
            if events then
                local cdpFolder = events:FindFirstChild("CDP")
                if cdpFolder then
                    for _, obj in ipairs(cdpFolder:GetDescendants()) do
                        if obj:IsA("RemoteEvent") then
                            pcall(function() obj:FireServer(0, LocalPlayer) end)
                        elseif obj:IsA("RemoteFunction") then
                            pcall(function() obj:InvokeServer(0) end)
                        end
                    end
                    Log.add("📡 Remotes.Events.CDP disparado")
                end
            end
        end
    end)
end

-- ============================================================
--  CORE — PURGE (limpa objetos CDP da UI e valores)
-- ============================================================
local function purgeObjects()
    local zeroed  = 0
    local hidden  = 0
    local nuked   = 0

    local roots = { PlayerGui, LocalPlayer }
    if LocalPlayer.Character then
        table.insert(roots, LocalPlayer.Character)
    end

    for _, root in ipairs(roots) do
        for _, obj in ipairs(root:GetDescendants()) do
            if not obj or not obj.Parent then continue end
            if Utils.isWhitelisted(obj) then continue end

            local name  = string.lower(obj.Name)
            local class = obj.ClassName

            -- Valores numéricos/bool → zerar
            if class == "NumberValue" or class == "IntValue" then
                if Utils.matchesAny(name, CONFIG.KEYWORDS) then
                    pcall(function() obj.Value = 0; zeroed += 1 end)
                end
            elseif class == "BoolValue" then
                if Utils.matchesAny(name, CONFIG.KEYWORDS) then
                    pcall(function() obj.Value = false; zeroed += 1 end)
                end

            -- UI com keyword no nome → esconder ou destruir (leaf)
            elseif obj:IsA("GuiObject") then
                if Utils.matchesAny(name, CONFIG.KEYWORDS) then
                    if #obj:GetChildren() == 0 then
                        pcall(function() obj:Destroy(); nuked += 1 end)
                    else
                        pcall(function() obj.Visible = false; hidden += 1 end)
                    end
                end

            -- TextLabel com texto CDP → esconder
            elseif (obj:IsA("TextLabel") or obj:IsA("TextButton")) then
                local txt = string.lower(obj.Text or "")
                if Utils.matchesAny(txt, CONFIG.KEYWORDS) then
                    pcall(function() obj.Visible = false; hidden += 1 end)
                end
            end
        end
    end

    return zeroed, hidden, nuked
end

-- ============================================================
--  CORE — SCAN (apenas contagem, sem remover)
-- ============================================================
local function scan()
    Log.add("🔍 Varrendo...")
    local count   = 0
    local targets = { PlayerGui, LocalPlayer }
    if LocalPlayer.Character then
        table.insert(targets, LocalPlayer.Character)
    end

    for _, root in ipairs(targets) do
        for _, obj in ipairs(root:GetDescendants()) do
            if Utils.isCDPSuspect(obj) then count += 1 end
        end
    end

    -- Contar remotes conhecidos
    local remoteCount = 0
    for _, name in ipairs({"SetCDP","CDPEvent","CDPFunction"}) do
        if Utils.deepFind(ReplicatedStorage, name) then
            remoteCount += 1
            Log.add("📡 Encontrado: " .. name)
        end
    end

    Log.add("🎯 Objetos suspeitos: " .. count)
    Log.add("📡 Remotes CDP: " .. remoteCount)
    return count
end

-- ============================================================
--  MAIN — NUKE LOOP (toggle on/off)
-- ============================================================
local nukeActive     = false
local nukeConnection = nil   -- DescendantAdded connection

local function stopNuke(ui)
    nukeActive = false
    if nukeConnection then
        nukeConnection:Disconnect()
        nukeConnection = nil
    end
    ui.KillBtn.Text             = "☢️ ATIVAR NUKE"
    ui.KillBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
    ui.Status.Text              = "Status: Nuke desativado"
    ui.Status.TextColor3        = Color3.fromRGB(150, 255, 150)
    Log.add("🛑 Nuke desativado")
end

local function startNuke(ui)
    if nukeActive then stopNuke(ui) return end

    nukeActive = true
    ui.KillBtn.Text             = "🛑 PARAR NUKE"
    ui.KillBtn.BackgroundColor3 = Color3.fromRGB(120, 30, 30)
    ui.Status.Text              = "Status: NUKE ATIVO ☢️"
    ui.Status.TextColor3        = Color3.fromRGB(255, 60, 60)
    Log.add("☢️ Nuke ativado!")

    -- Intercepta qualquer objeto CDP criado em tempo real
    nukeConnection = game.DescendantAdded:Connect(function(obj)
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

    -- Loop de ataque contínuo
    task.spawn(function()
        while nukeActive do
            -- 1. Disparar remotes
            fireAllRemotes()

            -- 2. Limpar objetos existentes
            local z, h, n = purgeObjects()
            local total = z + h + n
            if total > 0 then
                Log.add(string.format("✅ Zerados:%d | Ocultos:%d | Apagados:%d", z, h, n))
                ui.Status.Text = string.format("Status: %d tratados ☢️", total)
            end

            task.wait(CONFIG.LOOP_INTERVAL)
        end
    end)
end

-- ============================================================
--  PERSISTÊNCIA — queue_on_teleport
--  Reinjeta o script automaticamente em cada servidor novo
-- ============================================================
local SCRIPT_URL = "https://raw.githubusercontent.com/PedrinhuuScripts/CDP-KILLED/refs/heads/main/Scripts.md"

local function setupPersistence()
    -- Compatível com Synapse X, Fluxus, Celery, Solara, Wave e outros
    local queueFunc = queue_on_teleport
        or (syn      and syn.queue_on_teleport)
        or (fluxus   and fluxus.queue_on_teleport)
        or (celery   and celery.queue_on_teleport)

    if not queueFunc then
        Log.add("⚠️ Executor sem queue_on_teleport")
        return false
    end

    -- Marca para auto-start e recarrega o script no próximo servidor
    local code = string.format([[
        getgenv().DhzAutoStart = true
        local ok, err = pcall(function()
            loadstring(game:HttpGet("%s"))()
        end)
        if not ok then warn("DhzScripts reload error: " .. tostring(err)) end
    ]], SCRIPT_URL)

    local success = pcall(function() queueFunc(code) end)
    return success
end

-- ============================================================
--  INICIALIZAÇÃO
-- ============================================================
local ui = buildUI()
Log.init(ui.LogFrame)

-- Configura persistência entre servidores
local persisted = setupPersistence()

-- Eventos dos botões
ui.ScanBtn.MouseButton1Click:Connect(function()
    ui.ScanBtn.BackgroundColor3 = Color3.fromRGB(80, 180, 255)
    task.wait(0.15)
    ui.ScanBtn.BackgroundColor3 = Color3.fromRGB(50, 150, 255)
    local count = scan()
    ui.Status.Text       = count > 0
        and ("Status: " .. count .. " suspeitos")
        or  "Status: Nenhum CDP visível"
    ui.Status.TextColor3 = count > 0
        and Color3.fromRGB(255, 200, 80)
        or  Color3.fromRGB(180, 180, 180)
end)

ui.KillBtn.MouseButton1Click:Connect(function()
    startNuke(ui)
end)

-- ── AUTO-START ───────────────────────────────────────────────
-- Roda imediatamente se veio de um teleport OU na primeira vez
local isAutoStart = getgenv and getgenv().DhzAutoStart == true

if isAutoStart then
    -- Limpando flag para não entrar em loop
    if getgenv then getgenv().DhzAutoStart = false end

    Log.add("🔄 Retornando de teleport...")
    Log.add("⏳ Aguardando mapa carregar...")

    -- Espera o mapa renderizar completamente antes de atacar
    task.spawn(function()
        task.wait(3)
        Log.add("☢️ Auto-nuke iniciando!")
        startNuke(ui)
    end)
else
    -- Primeira execução manual: loga e aguarda o clique
    Log.add("CDP Killer v5.0 carregado!")
    if persisted then
        Log.add("✅ Persistência ativa (teleport ok)")
    end
    Log.add("Clique ☢️ para ativar o nuke.")
    Log.add("Na próxima server hop, auto-inicia!")
end
