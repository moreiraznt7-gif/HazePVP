local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")

local espEnabled, chamsEnabled, rainbowChamsEnabled = false, false, false
local aimbotEnabled, silentAimEnabled = false, false
local hitboxEnabled = false

local espBoxes = {}

function CreateESPBox(player)
    if espBoxes[player] then return end
    local box = Drawing.new("Square")
    box.Visible = false
    box.Color = Color3.new(1, 0, 0)
    box.Thickness = 2
    box.Filled = false
    espBoxes[player] = box

    RunService.RenderStepped:Connect(function()
        if espEnabled and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player ~= LocalPlayer then
            local pos, onscreen = Camera:WorldToViewportPoint(player.Character.HumanoidRootPart.Position)
            if onscreen then
                box.Visible = true
                box.Size = Vector2.new(50, 100)
                box.Position = Vector2.new(pos.X - 25, pos.Y - 50)
            else
                box.Visible = false
            end
        else
            box.Visible = false
        end
    end)
end

function ApplyChams(player)
    if chamsEnabled and player.Character then
        for _, part in pairs(player.Character:GetChildren()) do
            if part:IsA("BasePart") then
                part.Color = Color3.new(0, 1, 0)
                part.Material = Enum.Material.Neon
            end
        end
    elseif rainbowChamsEnabled and player.Character then
        local t = tick() % 5
        local color = Color3.fromHSV(t/5, 1, 1)
        for _, part in pairs(player.Character:GetChildren()) do
            if part:IsA("BasePart") then
                part.Color = color
                part.Material = Enum.Material.Neon
            end
        end
    elseif player.Character then
        for _, part in pairs(player.Character:GetChildren()) do
            if part:IsA("BasePart") then
                part.Color = Color3.new(1, 1, 1)
                part.Material = Enum.Material.Plastic
            end
        end
    end
end

function Aimbot()
    if not aimbotEnabled then return end
    local closest = nil
    local shortest = math.huge
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local pos, onscreen = Camera:WorldToViewportPoint(player.Character.Head.Position)
            if onscreen then
                local dist = (Vector2.new(pos.X, pos.Y) - Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)).Magnitude
                if dist < shortest then
                    shortest = dist
                    closest = player
                end
            end
        end
    end
    if closest and closest.Character:FindFirstChild("Head") then
        Camera.CFrame = CFrame.new(Camera.CFrame.Position, closest.Character.Head.Position)
    end
end

function SilentAim(targetPartName)
    if not silentAimEnabled then return end
    local closest = nil
    local shortest = math.huge
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild(targetPartName) then
            local pos, onscreen = Camera:WorldToViewportPoint(player.Character[targetPartName].Position)
            if onscreen then
                local dist = (Vector2.new(pos.X, pos.Y) - Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)).Magnitude
                if dist < shortest then
                    shortest = dist
                    closest = player
                end
            end
        end
    end
    if closest and closest.Character:FindFirstChild(targetPartName) then
        -- O disparo será direcionado ao closest.Character[targetPartName] (implementação depende do jogo!)
    end
end

function CustomizeHitbox(player, size)
    if hitboxEnabled and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        player.Character.HumanoidRootPart.Size = size
        player.Character.HumanoidRootPart.Transparency = 0.5
    elseif player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        player.Character.HumanoidRootPart.Size = Vector3.new(2,2,1)
        player.Character.HumanoidRootPart.Transparency = 0
    end
end

for _, player in pairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        CreateESPBox(player)
    end
end

Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        CreateESPBox(player)
    end
end)

RunService.RenderStepped:Connect(function()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            ApplyChams(player)
            CustomizeHitbox(player, Vector3.new(10,10,10))
        end
    end
    Aimbot()
    SilentAim("Head")
end)

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = game:GetService("CoreGui")
ScreenGui.Name = "PainelCheatCat"

local Frame = Instance.new("Frame", ScreenGui)
Frame.Position = UDim2.new(0.05,0,0.05,0)
Frame.Size = UDim2.new(0, 240, 0, 330)
Frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
Frame.Active = true
Frame.Draggable = true

local Title = Instance.new("TextLabel", Frame)
Title.Text = "Painel Cheat"
Title.Size = UDim2.new(1, 0, 0, 30)
Title.BackgroundTransparency = 1
Title.TextColor3 = Color3.new(1,1,1)
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 20

function makeLabel(text, posY)
    local lbl = Instance.new("TextLabel", Frame)
    lbl.Text = text
    lbl.Position = UDim2.new(0, 10, 0, posY)
    lbl.Size = UDim2.new(0, 220, 0, 24)
    lbl.BackgroundTransparency = 1
    lbl.TextColor3 = Color3.new(1,1,1)
    lbl.Font = Enum.Font.SourceSansBold
    lbl.TextSize = 16
    return lbl
end

function makeButton(text, posY, callback)
    local btn = Instance.new("TextButton", Frame)
    btn.Text = text
    btn.Position = UDim2.new(0, 30, 0, posY)
    btn.Size = UDim2.new(0, 180, 0, 26)
    btn.BackgroundColor3 = Color3.fromRGB(80,80,80)
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Font = Enum.Font.SourceSans
    btn.TextSize = 16
    btn.MouseButton1Click:Connect(callback)
    return btn
end

makeLabel("ESP", 40)
makeButton("ESP Box [ON/OFF]", 70, function() espEnabled = not espEnabled end)
makeButton("Chams [ON/OFF]", 100, function() chamsEnabled = not chamsEnabled end)
makeButton("Rainbow Chams [ON/OFF]", 130, function() rainbowChamsEnabled = not rainbowChamsEnabled end)

makeLabel("AIMBOT", 170)
makeButton("Aimbot [ON/OFF]", 200, function() aimbotEnabled = not aimbotEnabled end)
makeButton("Silent Aim [ON/OFF]", 230, function() silentAimEnabled = not silentAimEnabled end)

makeLabel("HITBOX", 270)
makeButton("Custom Hitbox [ON/OFF]", 300, function() hitboxEnabled = not hitboxEnabled end)

local closeBtn = makeButton("Fechar Painel", 330, function()
    Frame.Visible = false
    openBtn.Visible = true
end)

local openBtn = Instance.new("TextButton", ScreenGui)
openBtn.Text = "Abrir Painel"
openBtn.Size = UDim2.new(0, 120, 0, 40)
openBtn.Position = UDim2.new(0, 10, 0, 10)
openBtn.BackgroundColor3 = Color3.fromRGB(80,80,80)
openBtn.TextColor3 = Color3.new(1,1,1)
openBtn.Font = Enum.Font.SourceSansBold
openBtn.TextSize = 20
openBtn.Visible = false

closeBtn.MouseButton1Click:Connect(function()
    Frame.Visible = false
    openBtn.Visible = true
end)

openBtn.MouseButton1Click:Connect(function()
    Frame.Visible = true
    openBtn.Visible = false
end)

Frame.Visible = true
openBtn.Visible = false
