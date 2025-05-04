local P, G, W = game:GetService("Players"), game:GetService("RunService"), workspace.CurrentCamera
local L, Pg, M, U = P.LocalPlayer, P.LocalPlayer.PlayerGui, P.LocalPlayer:GetMouse(), game:GetService("UserInputService")
local S = {Aimbot = false, ESP = false, FOV = 100, AimPart = "Head", TeamCheck = true, GameMode = "Arsenal"}

-- UI
local SG = Instance.new("ScreenGui", Pg)
SG.Name = "VHub"
local F = Instance.new("Frame", SG)
F.Size = UDim2.new(0, 250, 0, 320)
F.Position = UDim2.new(0.05, 0, 0.1, 0)
F.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
F.BorderSizePixel = 0
F.Active = true
F.Draggable = true

local T = Instance.new("TextLabel", F)
T.Text = "Vision Hub - Arsenal"
T.Size = UDim2.new(1, 0, 0, 30)
T.BackgroundTransparency = 1
T.TextColor3 = Color3.new(1, 1, 1)
T.Font = Enum.Font.GothamBold
T.TextSize = 18

local function B(t, y, f)
    local b = Instance.new("TextButton", F)
    b.Size = UDim2.new(0.9, 0, 0, 30)
    b.Position = UDim2.new(0.05, 0, 0, y)
    b.Text = t
    b.TextColor3 = Color3.new(1, 1, 1)
    b.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    b.Font = Enum.Font.Gotham
    b.TextSize = 14
    b.MouseButton1Click:Connect(f)
    return b
end

B("Toggle Aimbot", 40, function()
    S.Aimbot = not S.Aimbot
end)

B("Toggle ESP", 80, function()
    S.ESP = not S.ESP
end)

local Mz = B("Minimizar", 270, function()
    F.Visible = false
end)

local Op = Instance.new("TextButton", SG)
Op.Size = UDim2.new(0, 100, 0, 30)
Op.Position = UDim2.new(0.05, 0, 0.05, 0)
Op.Text = "Abrir Hub"
Op.TextColor3 = Color3.new(1, 1, 1)
Op.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Op.Font = Enum.Font.Gotham
Op.TextSize = 14
Op.Visible = false
Op.MouseButton1Click:Connect(function()
    F.Visible = true
    Op.Visible = false
end)

Mz.MouseButton1Click:Connect(function()
    Op.Visible = true
end)

-- FOV Circle
local fovCircle = Drawing.new("Circle")
fovCircle.Visible = true
fovCircle.Radius = S.FOV
fovCircle.Thickness = 1.5
fovCircle.Color = Color3.fromRGB(255, 255, 0)
fovCircle.Filled = false

-- Get Closest Player
local function getClosest()
    local closest, dist = nil, S.FOV
    for _, plr in ipairs(P:GetPlayers()) do
        if plr ~= L and plr.Character and plr.Character:FindFirstChild(S.AimPart) then
            if not S.TeamCheck or plr.Team ~= L.Team then
                local pos, onScreen = W:WorldToViewportPoint(plr.Character[S.AimPart].Position)
                if onScreen then
                    local mag = (Vector2.new(pos.X, pos.Y) - Vector2.new(M.X, M.Y)).Magnitude
                    if mag < dist then
                        dist = mag
                        closest = plr
                    end
                end
            end
        end
    end
    return closest
end

-- Aimbot Functionality
RunService.RenderStepped:Connect(function()
    fovCircle.Position = Vector2.new(W.ViewportSize.X / 2, W.ViewportSize.Y / 2)
    if S.Aimbot and U:IsMouseButtonPressed(Enum.UserInputType.MouseButton1) then
        local target = getClosest()
        if target and target.Character and target.Character:FindFirstChild(S.AimPart) then
            local aimPos = target.Character[S.AimPart].Position
            -- Smoothly adjust the CFrame to "stick" to the head
            W.CFrame = CFrame.new(W.CFrame.Position, aimPos)
        end
    end
end)

-- Wallbang Shoot
local function ShootAt(target)
    if target and target.Character and target.Character:FindFirstChild(S.AimPart) then
        if S.GameMode == "Arsenal" then
            local gun = L.Backpack:FindFirstChild("Gun") or L.Character:FindFirstChild("Gun")
            if gun then
                local aimPos = target.Character[S.AimPart].Position
                local direction = (aimPos - W.CFrame.Position).Unit

                local bullet = Instance.new("Part")
                bullet.Size = Vector3.new(0.2, 0.2, 0.2)
                bullet.Shape = Enum.PartType.Ball
                bullet.Material = Enum.Material.Neon
                bullet.BrickColor = BrickColor.new("Bright yellow")
                bullet.Anchored = false
                bullet.CanCollide = false
                bullet.CFrame = W.CFrame
                bullet.Velocity = direction * 500
                bullet.Parent = workspace

                game.Debris:AddItem(bullet, 2)

                task.delay(0.1, function()
                    bullet.Position = aimPos
                end)
            end
        end
    end
end

U.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        local target = getClosest()
        ShootAt(target)
    end
end)

-- ESP Box with Team Check
local espBoxes = {}

local function CreateBox()
    local box = Drawing.new("Square")
    box.Thickness = 1.5
    box.Color = Color3.fromRGB(255, 0, 0)
    box.Filled = false
    box.Visible = false
    return box
end

RunService.RenderStepped:Connect(function()
    if not S.ESP then
        for _, b in pairs(espBoxes) do
            b.Visible = false
        end
        return
    end

    for _, plr in ipairs(P:GetPlayers()) do
        if plr ~= L and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            if not S.TeamCheck or plr.Team ~= L.Team then
                local hrp = plr.Character.HumanoidRootPart
                local pos, onScreen = W:WorldToViewportPoint(hrp.Position)

                if onScreen then
                    local sizeX = (W:WorldToViewportPoint(hrp.Position + Vector3.new(2, 3, 0)).X
                                  - W:WorldToViewportPoint(hrp.Position - Vector3.new(2, 3, 0)).X)

                    local box = espBoxes[plr] or CreateBox()
                    espBoxes[plr] = box

                    box.Size = Vector2.new(sizeX, sizeX * 1.5)
                    box.Position = Vector2.new(pos.X - box.Size.X / 2, pos.Y - box.Size.Y / 2)
                    box.Visible = true
                end
            end
        elseif espBoxes[plr] then
            espBoxes[plr].Visible = false
        end
    end
end)
