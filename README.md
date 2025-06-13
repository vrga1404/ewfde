local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "BloxFruitsHub"
ScreenGui.Parent = game.CoreGui

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 400, 0, 500)
MainFrame.Position = UDim2.new(0.5, -200, 0.5, -250)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner", MainFrame)
UICorner.CornerRadius = UDim.new(0, 12)

local Title = Instance.new("TextLabel", MainFrame)
Title.Text = "Blox Fruits Ultimate Hub"
Title.Size = UDim2.new(1, 0, 0, 40)
Title.BackgroundTransparency = 1
Title.TextColor3 = Color3.new(1, 1, 1)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 26

local SectionY = 50
function AddButton(text, callback)
    local btn = Instance.new("TextButton", MainFrame)
    btn.Size = UDim2.new(1, -20, 0, 36)
    btn.Position = UDim2.new(0, 10, 0, SectionY)
    btn.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
    btn.BorderSizePixel = 0
    btn.Text = text
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 18
    btn.MouseButton1Click:Connect(callback)
    local uic = Instance.new("UICorner", btn)
    uic.CornerRadius = UDim.new(0, 8)
    SectionY = SectionY + 42
end

--# Funções principais do hub

-- [1] Autofarm
local autofarmActive = false
function Autofarm()
    autofarmActive = not autofarmActive
    if autofarmActive then
        spawn(function()
            while autofarmActive do
                -- Exemplo de autofarm: atacar NPCs próximos
                local player = game.Players.LocalPlayer
                local char = player.Character
                local mobs = workspace.Enemies:GetChildren()
                for _, mob in pairs(mobs) do
                    if mob:FindFirstChild("HumanoidRootPart") and mob.Humanoid.Health > 0 then
                        char.HumanoidRootPart.CFrame = mob.HumanoidRootPart.CFrame * CFrame.new(0,5,0)
                        -- Simular ataque
                        game:GetService("ReplicatedStorage").Remotes.Combat:FireServer(mob)
                        wait(0.5)
                    end
                end
                wait(1)
            end
        end)
    end
end

-- [2] Auto Quest
local autoQuestActive = false
function AutoQuest()
    autoQuestActive = not autoQuestActive
    if autoQuestActive then
        spawn(function()
            while autoQuestActive do
                -- Exemplo: aceitar e completar quest automaticamente
                local questGiver = workspace.NPCs:FindFirstChild("QuestGiver")
                if questGiver then
                    game:GetService("ReplicatedStorage").Remotes.AcceptQuest:FireServer(questGiver)
                end
                wait(10)
            end
        end)
    end
end

-- [3] Auto Stats
local autoStatsActive = false
function AutoStats()
    autoStatsActive = not autoStatsActive
    if autoStatsActive then
        spawn(function()
            while autoStatsActive do
                game:GetService("ReplicatedStorage").Remotes.AddStat:FireServer("Melee", 5)
                game:GetService("ReplicatedStorage").Remotes.AddStat:FireServer("Defense", 5)
                game:GetService("ReplicatedStorage").Remotes.AddStat:FireServer("Sword", 5)
                wait(2)
            end
        end)
    end
end

-- [4] Teleport
function TeleportMenu()
    local islands = {
        ["Starter Island"] = Vector3.new(100, 10, 200),
        ["Jungle"] = Vector3.new(-1000, 35, 400),
        ["Marine Fortress"] = Vector3.new(3600, 200, 500),
        -- Adicione outras ilhas...
    }
    for name, pos in pairs(islands) do
        AddButton("Teleporte: " .. name, function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(pos)
        end)
    end
end

-- [5] ESP (mostrar jogadores/frutas)
local espActive = false
function ESP()
    espActive = not espActive
    if espActive then
        spawn(function()
            while espActive do
                for _, plr in pairs(game.Players:GetPlayers()) do
                    if plr ~= game.Players.LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                        -- Cria uma BillboardGui
                        if not plr.Character.HumanoidRootPart:FindFirstChild("ESP") then
                            local bb = Instance.new("BillboardGui", plr.Character.HumanoidRootPart)
                            bb.Name = "ESP"
                            bb.Size = UDim2.new(0, 100, 0, 40)
                            bb.AlwaysOnTop = true
                            local lbl = Instance.new("TextLabel", bb)
                            lbl.Size = UDim2.new(1, 0, 1, 0)
                            lbl.BackgroundTransparency = 1
                            lbl.Text = plr.Name
                            lbl.TextColor3 = Color3.new(0,1,0)
                            lbl.Font = Enum.Font.GothamBold
                            lbl.TextSize = 18
                        end
                    end
                end
                -- Fruits (exemplo)
                for _, fruit in pairs(workspace:GetChildren()) do
                    if fruit.Name:find("Fruit") and fruit:IsA("Model") and not fruit:FindFirstChild("ESP") then
                        local bb = Instance.new("BillboardGui", fruit)
                        bb.Name = "ESP"
                        bb.Size = UDim2.new(0, 100, 0, 40)
                        bb.AlwaysOnTop = true
                        local lbl = Instance.new("TextLabel", bb)
                        lbl.Size = UDim2.new(1, 0, 1, 0)
                        lbl.BackgroundTransparency = 1
                        lbl.Text = "🍎 " .. fruit.Name
                        lbl.TextColor3 = Color3.new(1,1,0)
                        lbl.Font = Enum.Font.GothamBold
                        lbl.TextSize = 18
                    end
                end
                wait(2)
            end
        end)
    else
        -- Remove ESPs
        for _, plr in pairs(game.Players:GetPlayers()) do
            if plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                local esp = plr.Character.HumanoidRootPart:FindFirstChild("ESP")
                if esp then esp:Destroy() end
            end
        end
        for _, fruit in pairs(workspace:GetChildren()) do
            if fruit:FindFirstChild("ESP") then fruit.ESP:Destroy() end
        end
    end
end

-- [6] Auto Compra de Frutas
function AutoBuyFruit()
    game:GetService("ReplicatedStorage").Remotes.BuyFruit:FireServer("Random")
end

-- [7] Fechar GUI
AddButton("Fechar Hub", function() ScreenGui:Destroy() end)

-- Adicionando os botões principais
AddButton("Ativar/Desativar Autofarm", Autofarm)
AddButton("Ativar/Desativar Auto Quest", AutoQuest)
AddButton("Ativar/Desativar Auto Stats", AutoStats)
AddButton("Ativar/Desativar ESP (Players & Frutas)", ESP)
AddButton("Comprar Fruta (Aleatória)", AutoBuyFruit)
AddButton("Teleportes (Abrir Menu)", TeleportMenu)

-- Créditos
local credits = Instance.new("TextLabel", MainFrame)
credits.Text = "by Copilot | github.com/vrga1404"
credits.Size = UDim2.new(1, 0, 0, 20)
credits.Position = UDim2.new(0,0,1,-22)
credits.BackgroundTransparency = 1
credits.TextColor3 = Color3.new(0.8,0.8,0.8)
credits.Font = Enum.Font.Gotham
credits.TextSize = 14

