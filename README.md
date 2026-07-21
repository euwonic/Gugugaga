-- ESP Otimizado + Highlight + Nome do Time
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local ESP = { 
    Enabled = true, 
    MaxDistance = 7500, 
    TeamCheck = true,
    HighlightEnabled = true 
}

local PlayerESP = {}

local function CreateDrawing(class)
    local d = Drawing.new(class)
    d.Visible = false
    return d
end

local function NewESP(plr)
    if plr == LocalPlayer then return end
    
    local highlight = Instance.new("Highlight")
    highlight.FillTransparency = 0.7
    highlight.OutlineTransparency = 0.3
    highlight.OutlineColor = Color3.fromRGB(255,255,255)
    highlight.Enabled = false
    highlight.Adornee = nil
    highlight.Parent = plr.Character or plr.CharacterAdded:Wait()
    
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
    
    -- Configurações visuais
    esp.Box.Thickness = 2
    esp.Box.Filled = false
    esp.Box.Transparency = 0.85
    
    esp.TeamName.Size = 14
    esp.TeamName.Center = true
    esp.TeamName.Outline = true
    esp.TeamName.Color = Color3.fromRGB(200, 200, 255)
    
    -- Health Bar, Tracer, etc. (mesmo de antes)
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
    
    -- 3 Slots de itens (mantido)
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
        
        -- Highlight no corpo inteiro
        if ESP.HighlightEnabled then
            esp.Highlight.Adornee = char
            esp.Highlight.FillColor = teamColor
            esp.Highlight.OutlineColor = teamColor
            esp.Highlight.Enabled = true
        else
            esp.Highlight.Enabled = false
        end
        
        -- Box
        local height = (Camera:WorldToViewportPoint(root.Position + Vector3.new(0,3,0)).Y - Camera:WorldToViewportPoint(root.Position - Vector3.new(0,3.5,0)).Y) * 1.6
        esp.Box.Size = Vector2.new(height * 1.8, height * 2.5)
        esp.Box.Position = Vector2.new(screenPos.X - esp.Box.Size.X/2, screenPos.Y - esp.Box.Size.Y/2)
        esp.Box.Color = teamColor
        esp.Box.Visible = true
        
        -- Name
        esp.Name.Text = plr.DisplayName
        esp.Name.Position = Vector2.new(screenPos.X, screenPos.Y - esp.Box.Size.Y/2 - 22)
        esp.Name.Color = teamColor
        esp.Name.Visible = true
        
        -- Team Name
        esp.TeamName.Text = "[" .. teamName .. "]"
        esp.TeamName.Position = Vector2.new(screenPos.X, screenPos.Y - esp.Box.Size.Y/2 - 38)
        esp.TeamName.Visible = true
        
        -- Health Bar, Tracer, Distance, Itens... (mantidos iguais)
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
        
        -- Slots de itens (mesma lógica anterior)
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
                local isEquipped = equipped == tool
                slot.BG.Transparency = isEquipped and 0.25 or 0.8
                slot.BG.Color = isEquipped and Color3.fromRGB(0, 200, 255) or Color3.fromRGB(30,30,30)
                slot.Image.Data = "rbxassetid://3926305904" -- fallback (pode melhorar depois)
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

-- Inicialização
for _, plr in ipairs(Players:GetPlayers()) do NewESP(plr) end
Players.PlayerAdded:Connect(NewESP)

-- Toggle
UserInputService.InputBegan:Connect(function(i, gp)
    if gp or i.KeyCode \~= Enum.KeyCode.Insert then return end
    ESP.Enabled = not ESP.Enabled
    print("ESP:", ESP.Enabled and "✅ ATIVADO" or "❌ DESATIVADO")
end)

print("🚀 ESP com Highlight + Nome do Time carregado!")
print("INSERT = Ligar/Desligar")
