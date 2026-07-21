-- ESP com GUI Arrastável para Mobile

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local ESP = { 
    Enabled = false, 
    MaxDistance = 7500, 
    TeamCheck = true,
    HighlightEnabled = true 
}

local PlayerESP = {}
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- Criar Frame da GUI
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 180, 0, 100)
MainFrame.Position = UDim2.new(0.5, -90, 0.1, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MainFrame.BorderSizePixel = 0
MainFrame.BackgroundTransparency = 0.3
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 12)
UICorner.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 30)
Title.BackgroundTransparency = 1
Title.Text = "ESP Menu"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 16
Title.Font = Enum.Font.GothamBold
Title.Parent = MainFrame

local ToggleButton = Instance.new("TextButton")
ToggleButton.Size = UDim2.new(0.9, 0, 0, 50)
ToggleButton.Position = UDim2.new(0.05, 0, 0.4, 0)
ToggleButton.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
ToggleButton.Text = "ESP: DESLIGADO"
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.TextSize = 16
ToggleButton.Font = Enum.Font.GothamBold
ToggleButton.Parent = MainFrame

local ButtonCorner = Instance.new("UICorner")
ButtonCorner.CornerRadius = UDim.new(0, 10)
ButtonCorner.Parent = ToggleButton

-- Função para tornar arrastável
local dragging = false
local dragInput
local dragStart
local startPos

local function updateInput(input)
    local delta = input.Position - dragStart
    local tweenInfo = TweenInfo.new(0.1)
    TweenService:Create(MainFrame, tweenInfo, {Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)}):Play()
end

MainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
    end
end)

MainFrame.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        updateInput(input)
    end
end)

-- Toggle do ESP
local function toggleESP()
    ESP.Enabled = not ESP.Enabled
    if ESP.Enabled then
        ToggleButton.Text = "ESP: LIGADO"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(0, 255, 100)
    else
        ToggleButton.Text = "ESP: DESLIGADO"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
    end
end

ToggleButton.MouseButton1Click:Connect(toggleESP)

-- Resto do código ESP (mesmo de antes, sem toggle de tecla)
local function CreateDrawing(class)
    local d = Drawing.new(class)
    d.Visible = false
    return d
end

local function NewESP(plr)
    if plr == LocalPlayer then return end
    -- (código do ESP completo igual ao anterior)
    local highlight = Instance.new("Highlight")
    highlight.FillTransparency = 0.65
    highlight.OutlineTransparency = 0.25
    highlight.Enabled = false
    highlight.Parent = workspace
    
    local esp = {
        Box = CreateDrawing("Square"),
        Name = CreateDrawing("Text"),
        TeamName = CreateDrawing("Text"),
        HealthBarBG = CreateDrawing("Square"),
        HealthBar = CreateDrawing("Square"),
        Tracer = CreateDrawing("Line"),
        Distance = CreateDrawing("Text"),
        ToolSlots = {},
        Highlight = highlight,
        Connections = {}
    }
    
    -- ... (o resto do código NewESP e Update é igual ao anterior)
    esp.Box.Thickness = 2
    esp.Box.Filled = false
    esp.Box.Transparency = 0.85
    
    esp.TeamName.Size = 14
    esp.TeamName.Center = true
    esp.TeamName.Outline = true
    esp.TeamName.Color = Color3.fromRGB(180, 220, 255)
    
    esp.HealthBarBG.Color = Color3.fromRGB(30,30,30)
    esp.HealthBarBG.Filled = true
    esp.HealthBar.Filled = true
    
    esp.Tracer.Thickness = 1.5
    esp.Tracer.Transparency = 0.7
    
    for _, t in pairs({esp.Name, esp.Distance}) do
        t.Size = 15
        t.Center = true
        t.Outline = true
    end
    
    for i = 1, 3 do
        local slot = { BG = CreateDrawing("Square"), Image = CreateDrawing("Image") }
        slot.BG.Size = Vector2.new(28, 28)
        slot.BG.Filled = true
        slot.Image.Size = Vector2.new(24, 24)
        esp.ToolSlots[i] = slot
    end
    
    local lastUpdate = 0
    local function Update()
        if not ESP.Enabled then return end
        if tick() - lastUpdate < 0.04 then return end
        lastUpdate = tick()
        
        local char = plr.Character
        if not char or not char:FindFirstChild("HumanoidRootPart") then
            esp.Highlight.Enabled = false
            for _, v in pairs(esp) do
                if typeof(v) == "Instance" and v.Visible \~= nil then v.Visible = false end
            end
            return
        end
        
        local root = char.HumanoidRootPart
        local humanoid = char:FindFirstChild("Humanoid")
        local myRoot = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        
        local distance = myRoot and (root.Position - myRoot.Position).Magnitude or 99999
        if distance > ESP.MaxDistance then
            esp.Highlight.Enabled = false
            for _, v in pairs(esp) do if typeof(v) == "Instance" and v.Visible \~= nil then v.Visible = false end end
            return
        end
        
        local screenPos, onScreen = Camera:WorldToViewportPoint(root.Position)
        if not onScreen then return end
        
        local teamColor = (ESP.TeamCheck and plr.Team and plr.Team.TeamColor.Color) or Color3.fromRGB(255, 60, 60)
        local teamName = plr.Team and plr.Team.Name or "Sem Time"
        
        if ESP.HighlightEnabled and char then
            esp.Highlight.Adornee = char
            esp.Highlight.FillColor = teamColor
            esp.Highlight.OutlineColor = teamColor
            esp.Highlight.Enabled = true
        end
        
        local height = (Camera:WorldToViewportPoint(root.Position + Vector3.new(0,3,0)).Y - Camera:WorldToViewportPoint(root.Position - Vector3.new(0,3.5,0)).Y) * 1.6
        
        esp.Box.Size = Vector2.new(height * 1.8, height * 2.5)
        esp.Box.Position = Vector2.new(screenPos.X - esp.Box.Size.X/2, screenPos.Y - esp.Box.Size.Y/2)
        esp.Box.Color = teamColor
        esp.Box.Visible = true
        
        esp.Name.Text = plr.DisplayName
        esp.Name.Position = Vector2.new(screenPos.X, screenPos.Y - esp.Box.Size.Y/2 - 22)
        esp.Name.Color = teamColor
        esp.Name.Visible = true
        
        esp.TeamName.Text = "[" .. teamName .. "]"
        esp.TeamName.Position = Vector2.new(screenPos.X, screenPos.Y - esp.Box.Size.Y/2 - 38)
        esp.TeamName.Visible = true
        
        local hpRatio = humanoid.Health / humanoid.MaxHealth
        local barW = esp.Box.Size.X * 0.9
        esp.HealthBarBG.Size = Vector2.new(barW, 6)
        esp.HealthBarBG.Position = Vector2.new(screenPos.X - barW/2, screenPos.Y - esp.Box.Size.Y/2 - 8)
        esp.HealthBarBG.Visible = true
        
        esp.HealthBar.Size = Vector2.new(barW * hpRatio, 6)
        esp.HealthBar.Position = esp.HealthBarBG.Position
        esp.HealthBar.Color = Color3.fromRGB(255*(1-hpRatio), 255*hpRatio, 0)
        esp.HealthBar.Visible = true
        
        esp.Tracer.From = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y)
        esp.Tracer.To = Vector2.new(screenPos.X, screenPos.Y)
        esp.Tracer.Color = teamColor
        esp.Tracer.Visible = true
        
        esp.Distance.Text = math.floor(distance) .. "m"
        esp.Distance.Position = Vector2.new(screenPos.X, screenPos.Y + esp.Box.Size.Y/2 + 10)
        esp.Distance.Visible = true
        
        local tools = {}
        local equipped = char:FindFirstChildOfClass("Tool")
        if equipped then table.insert(tools, equipped) end
        for _, t in ipairs(plr.Backpack:GetChildren()) do
            if t:IsA("Tool") and #tools < 3 then table.insert(tools, t) end
        end
        
        for i = 1, 3 do
            local slot = esp.ToolSlots[i]
            local tool = tools[i]
            if tool then
                slot.BG.Visible = true
                slot.Image.Visible = true
                local isEquipped = (equipped == tool)
                slot.BG.Transparency = isEquipped and 0.25 or 0.8
                slot.BG.Color = isEquipped and Color3.fromRGB(0, 200, 255) or Color3.fromRGB(30,30,30)
                slot.Image.Data = "rbxassetid://3926305904"
                slot.Image.Position = slot.BG.Position + Vector2.new(2,2)
            else
                slot.BG.Visible = false
                slot.Image.Visible = false
            end
        end
        
        local baseY = screenPos.Y + esp.Box.Size.Y/2 + 45
        for i, slot in ipairs(esp.ToolSlots) do
            slot.BG.Position = Vector2.new(screenPos.X - 50 + (i-1)*38, baseY)
        end
    end
    
    table.insert(esp.Connections, RunService.RenderStepped:Connect(Update))
    PlayerESP[plr] = esp
end

for _, plr in ipairs(Players:GetPlayers()) do NewESP(plr) end
Players.PlayerAdded:Connect(NewESP)

print("🚀 ESP com GUI Mobile carregado! Toque no botão para ligar/desligar.")
