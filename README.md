-- Dhz Scripts | CDP KILLER v4.5 - NUKE EDITION
-- Cores: Preto, Amarelo, Branco e Cinza
-- Créditos: PEDRINHUU SCRIPTS E DHZZ SCRIPTS

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")
local StarterGui = game:GetService("StarterGui")

-- Função para Notificação Oficial do Roblox
local function notifyRoblox()
    StarterGui:SetCore("SendNotification", {
        Title = "☢️ PEDRINHUU & DHZZ";
        Text = "Cdp Retirada, caso não funcionar reinicie o roblox, aguarde um tempo e faça o metodo novamente.\n\nÉ O PEDRINHUU!!";
        Duration = 10;
    })
end

-- UI Setup
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "NukeCDPVip"
ScreenGui.Parent = PlayerGui
ScreenGui.ResetOnSpawn = false

local MainFrame = Instance.new("Frame")
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0) -- Fundo Preto
MainFrame.Position = UDim2.new(0, 20, 0.5, -100)
MainFrame.Size = UDim2.new(0, 280, 0, 215) -- Aumentado levemente para caber os créditos
MainFrame.Active = true
MainFrame.Draggable = true

local Corner = Instance.new("UICorner", MainFrame)
local Stroke = Instance.new("UIStroke", MainFrame)
Stroke.Color = Color3.fromRGB(255, 255, 0) -- Borda Amarela
Stroke.Thickness = 2

local Title = Instance.new("TextLabel")
Title.Parent = MainFrame
Title.BackgroundTransparency = 1
Title.Position = UDim2.new(0, 0, 0, 10)
Title.Size = UDim2.new(1, 0, 0, 30)
Title.Font = Enum.Font.GothamBold
Title.Text = "NUKE CDP ☢️"
Title.TextColor3 = Color3.fromRGB(255, 255, 0) -- Titulo Amarelo
Title.TextSize = 22

local Status = Instance.new("TextLabel")
Status.Parent = MainFrame
Status.BackgroundTransparency = 1
Status.Position = UDim2.new(0, 15, 0, 50)
Status.Size = UDim2.new(1, -30, 0, 30)
Status.Font = Enum.Font.GothamSemibold
Status.Text = "Nenhuma ação..." 
Status.TextColor3 = Color3.fromRGB(255, 255, 255)
Status.TextScaled = true

local LogFrame = Instance.new("ScrollingFrame")
LogFrame.Parent = MainFrame
LogFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
LogFrame.Position = UDim2.new(0, 15, 0, 90)
LogFrame.Size = UDim2.new(1, -30, 0, 45)
LogFrame.ScrollBarThickness = 2
LogFrame.CanvasSize = UDim2.new(0, 0, 2, 0)
local LogCorner = Instance.new("UICorner", LogFrame)

-- Botão Procurar (Cinza)
local ScanBtn = Instance.new("TextButton")
ScanBtn.Parent = MainFrame
ScanBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 80) 
ScanBtn.Position = UDim2.new(0, 15, 1, -65)
ScanBtn.Size = UDim2.new(0.45, 0, 0, 35)
ScanBtn.Text = "Procurar Lixos"
ScanBtn.TextColor3 = Color3.new(1,1,1)
ScanBtn.Font = Enum.Font.GothamBold
local ScanCorner = Instance.new("UICorner", ScanBtn)

-- Botão Nukar (Amarelo)
local KillBtn = Instance.new("TextButton")
KillBtn.Parent = MainFrame
KillBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 0)
KillBtn.Position = UDim2.new(0.53, 0, 1, -65)
KillBtn.Size = UDim2.new(0.45, 0, 0, 35)
KillBtn.Text = "NUKAR CDP ☢️"
KillBtn.TextColor3 = Color3.fromRGB(0, 0, 0) 
KillBtn.Font = Enum.Font.GothamBold
local KillCorner = Instance.new("UICorner", KillBtn)

-- LABEL DE CRÉDITOS (NOVA)
local CreditsLabel = Instance.new("TextLabel")
CreditsLabel.Parent = MainFrame
CreditsLabel.BackgroundTransparency = 1
CreditsLabel.Position = UDim2.new(0, 0, 1, -22)
CreditsLabel.Size = UDim2.new(1, 0, 0, 20)
CreditsLabel.Font = Enum.Font.GothamBold
CreditsLabel.Text = "By: PEDRINHUU SCRIPTS E DHZZ SCRIPTS"
CreditsLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
CreditsLabel.TextSize = 10
CreditsLabel.TextTransparency = 0.4 -- Efeito sutil

-- LÓGICA AGRESSIVA
local killerActive = false

local function scanCDP()
    local count = 0
    for _, obj in pairs(game:GetDescendants()) do
        pcall(function()
            if string.find(string.lower(obj.Name), "cdp") or string.find(string.lower(obj.Name), "capacita") then
                count = count + 1
            end
        end)
    end
    Status.Text = count .. " Lixos Encontrados"
    Status.TextColor3 = Color3.fromRGB(255, 0, 0) 
end

local function startNuke()
    if killerActive then return end
    killerActive = true
    
    Status.Text = "Nukando CDP..."
    Status.TextColor3 = Color3.fromRGB(255, 255, 0) 
    notifyRoblox()

    task.spawn(function()
        while killerActive do
            local cleaned = false
            for _, obj in pairs(game:GetDescendants()) do
                pcall(function()
                    local n = string.lower(obj.Name)
                    if string.find(n, "cdp") or string.find(n, "capacita") or string.find(n, "timer") then
                        obj:Destroy()
                        cleaned = true
                    end
                end)
            end

            pcall(function()
                local rf = ReplicatedStorage:FindFirstChild("RF", true)
                if rf and rf:FindFirstChild("SetCDP") then
                    rf.SetCDP:InvokeServer(LocalPlayer.Name, 0)
                end
            end)

            if cleaned then
                Status.Text = "Bypass Ativo"
                Status.TextColor3 = Color3.fromRGB(0, 255, 0) 
            end
            task.wait(0.5)
        end
    end)
end

ScanBtn.MouseButton1Click:Connect(scanCDP)
KillBtn.MouseButton1Click:Connect(startNuke)

-- Persistência Server Hop
local SCRIPT_URL = "https://raw.githubusercontent.com/PedrinhuuScripts/CDP-KILLED/refs/heads/main/Scripts.md"
local queueonteleport = queue_on_teleport or (syn and syn.queue_on_teleport)

if queueonteleport then
    queueonteleport([[
        getgenv().DhzAutoStartKiller = true
        loadstring(game:HttpGet("]]..SCRIPT_URL..[["))()
    ]])
end

if getgenv().DhzAutoStartKiller then
    task.wait(2)
    startNuke()
end
