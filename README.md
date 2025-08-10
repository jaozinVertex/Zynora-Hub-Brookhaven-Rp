# Zynora-Hub-Brookhaven-Rp
Zynora Hub Script Brookhaven Updated one of the best currently, new in Beta still
local redzlib = loadstring(game:HttpGet("https://raw.githubusercontent.com/tlredz/Library/refs/heads/main/V5/Source.lua"))()

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local TextChatService = game:GetService("TextChatService")
local LocalPlayer = Players.LocalPlayer
local camera = workspace.CurrentCamera

local Window = redzlib:MakeWindow({
    Title = "Zynora Hub - Brookhaven 🏡 RP Beta",
    SubTitle = "By: Jaozin",
    SaveFolder = "ZynoraHub"
})

Window:AddMinimizeButton({
    Button = { BackgroundTransparency = 0 },
    Corner = { CornerRadius = UDim.new(0, 6) }
})

-- Aba Créditos
local CreditsTab = Window:MakeTab({
    Title = "Créditos"
})

CreditsTab:AddButton({
    Name = "Criador: Jaozin",
    Callback = function()
        print("Criador: Jaozin")
    end
})

CreditsTab:AddDiscordInvite({
    Name = "Zynora Community",
    Description = "Join server",
    Invite = "https://discord.gg/vTmd9F7w"
})

-- Aba Troll
local TrollTab = Window:MakeTab({
    Title = "Troll"
})

local selectedPlayerName = nil
local isViewing = false
local flingActive = false

local function getPlayerNames()
    local names = {}
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            table.insert(names, player.Name)
        end
    end
    return names
end

local playerDropdown = TrollTab:AddDropdown({
    Name = "Selecionar Jogador",
    Default = nil,
    Options = getPlayerNames(),
    Callback = function(name)
        selectedPlayerName = name
    end
})

TrollTab:AddButton({
    Name = "Atualizar Playerlist.",
    Description = "Para carregar a Playerlist, atualize ela.",
    Callback = function()
        local tablePlayers = Players:GetPlayers()
        local newPlayers = {}
        if playerDropdown and #tablePlayers > 0 then
            for _, player in ipairs(tablePlayers) do
                if player.Name ~= LocalPlayer.Name then
                    table.insert(newPlayers, player.Name)
                end
            end
            playerDropdown:Set(newPlayers)
            print("Lista de jogadores atualizada: ", table.concat(newPlayers, ", "))
            if selectedPlayerName and not Players:FindFirstChild(selectedPlayerName) then
                selectedPlayerName = nil
                playerDropdown:SetValue("")
                print("Seleção resetada, jogador não está mais no servidor.")
            end
        else
            print("Erro: Dropdown não encontrado ou nenhum jogador disponível.")
        end
    end
})

local function startViewing()
    if not selectedPlayerName then return end
    local target = Players:FindFirstChild(selectedPlayerName)
    if not target or not target.Character then return end
    camera.CameraSubject = target.Character:FindFirstChildWhichIsA("Humanoid") 
        or target.Character:FindFirstChild("HumanoidRootPart") 
        or target.Character.PrimaryPart
end

local function stopViewing()
    if not LocalPlayer.Character then return end
    camera.CameraSubject = LocalPlayer.Character:FindFirstChildWhichIsA("Humanoid") 
        or LocalPlayer.Character:FindFirstChild("HumanoidRootPart") 
        or LocalPlayer.Character.PrimaryPart
end

TrollTab:AddToggle({
    Name = "Visualizar Jogador",
    Description = "Liga/Desliga a visualização do jogador selecionado",
    Default = false,
    Callback = function(state)
        isViewing = state
        if isViewing then
            startViewing()
        else
            stopViewing()
        end
    end
})

local function TeleportToPlayer()
    if not selectedPlayerName then
        warn("Nenhum jogador selecionado")
        return
    end
    local target = Players:FindFirstChild(selectedPlayerName)
    if not target or not target.Character or not target.Character:FindFirstChild("HumanoidRootPart") then
        warn("Jogador alvo inválido")
        return
    end
    local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if hrp then
        hrp.CFrame = target.Character.HumanoidRootPart.CFrame * CFrame.new(0, 3, 0)
    else
        warn("Seu personagem não está carregado")
    end
end

TrollTab:AddButton({
    Name = "Teleportar até Jogador",
    Description = "Teleporta até o jogador selecionado",
    Callback = TeleportToPlayer
})

local function stopFling()
    flingActive = false
end

local function startFling()
    if flingActive then return end -- Já rodando
    if not selectedPlayerName then
        warn("Nenhum jogador selecionado")
        return
    end

    local target = Players:FindFirstChild(selectedPlayerName)
    if not target or not target.Character or not target.Character:FindFirstChild("HumanoidRootPart") then
        warn("Jogador alvo inválido")
        return
    end

    local char = LocalPlayer.Character
    if not char then
        warn("Seu personagem não está carregado")
        return
    end

    local hum = char:FindFirstChildOfClass("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")
    if not hum or not root then
        warn("Componentes do seu personagem faltando")
        return
    end

    flingActive = true

    -- Limpa ferramentas antigas
    ReplicatedStorage:WaitForChild("RE"):WaitForChild("1Clea1rTool1s"):FireServer("ClearAllTools")
    task.wait(0.3)

    -- Pega o sofá
    local ok, err = pcall(function()
        ReplicatedStorage.RE:FindFirstChild("1Too1l"):InvokeServer("PickingTools", "Couch")
    end)
    if not ok then
        warn("Falha ao pegar o sofá:", err)
        flingActive = false
        return
    end
    task.wait(0.5)

    -- Move o sofá para o personagem
    local tool = LocalPlayer.Backpack:FindFirstChild("Couch")
    if tool then
        tool.Parent = char
    else
        warn("Sofá não encontrado na mochila")
        flingActive = false
        return
    end
    task.wait(0.2)

    -- Teleporta próximo ao alvo (3 studs acima)
    local targetPos = target.Character.HumanoidRootPart.Position + Vector3.new(0, 3, 0)
    root.CFrame = CFrame.new(targetPos)
    task.wait(0.2)

    -- Ativa o sofá para sentar automaticamente
    if tool and tool.Activate then
        pcall(function()
            tool:Activate()
        end)
    else
        warn("Não foi possível ativar o sofá automaticamente.")
    end
    task.wait(0.3)

    -- Aplica BodyVelocity para o fling
    local bv = Instance.new("BodyVelocity")
    bv.Name = "FlingForce"
    bv.Velocity = Vector3.new(9e8, 9e8, 9e8)
    bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    bv.Parent = root

    hum:SetStateEnabled(Enum.HumanoidStateType.Seated, false)
    hum.PlatformStand = false

    local angle = 0

    -- Loop para girar para frente (em X) e seguir o alvo
    task.spawn(function()
        while flingActive and target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") do
            angle += 50

            local targetRoot = target.Character.HumanoidRootPart
            -- Ajusta posição para acompanhar o alvo com previsão da velocidade
            local pos_x = targetRoot.Position.X + (targetRoot.Velocity.X / 1.5)
            local pos_y = targetRoot.Position.Y + (targetRoot.Velocity.Y / 1.5)
            local pos_z = targetRoot.Position.Z + (targetRoot.Velocity.Z / 1.5)

            -- Aplica CFrame com rotação no eixo X (girando para frente)
            root.CFrame = CFrame.new(pos_x, pos_y, pos_z) * CFrame.Angles(math.rad(angle), 0, 0)

            root.Velocity = Vector3.new(9e8, 9e8, 9e8)
            root.RotVelocity = Vector3.new(9e8, 9e8, 9e8)

            task.wait()
        end

        -- Limpa tudo quando acabar
        flingActive = false
        bv:Destroy()
        hum:SetStateEnabled(Enum.HumanoidStateType.Seated, true)
        hum.PlatformStand = false

        for _, p in pairs(char:GetDescendants()) do
            if p:IsA("BasePart") then
                p.Velocity = Vector3.zero
                p.RotVelocity = Vector3.zero
            end
        end

        hum:UnequipTools()
    end)
end

TrollTab:AddToggle({
    Name = "Couch Fling",
    Description = "Ativa/desativa o Fling Couch giratório automático",
    Default = false,
    Callback = function(state)
        if state then
            startFling()
        else
            stopFling()
        end
    end
})

-- Fling Ball Function
local function FlingBall(target)
    local players = game:GetService("Players")
    local player = players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoid = character:WaitForChild("Humanoid")
    local hrp = character:WaitForChild("HumanoidRootPart")
    local backpack = player:WaitForChild("Backpack")
    local ServerBalls = workspace.WorkspaceCom:WaitForChild("001_SoccerBalls")

    local function GetBall()
        if not backpack:FindFirstChild("SoccerBall") then
            game:GetService("ReplicatedStorage").RE:FindFirstChild("1Too1l"):InvokeServer("PickingTools", "SoccerBall")
        end
        repeat task.wait() until backpack:FindFirstChild("SoccerBall")
        backpack.SoccerBall.Parent = character
        repeat task.wait() until ServerBalls:FindFirstChild("Soccer" .. player.Name)
        character.SoccerBall.Parent = backpack
        return ServerBalls:FindFirstChild("Soccer" .. player.Name)
    end

    local Ball = ServerBalls:FindFirstChild("Soccer" .. player.Name) or GetBall()
    Ball.CanCollide = false
    Ball.Massless = true
    Ball.CustomPhysicalProperties = PhysicalProperties.new(0.0001, 0, 0)

    if target ~= player then
        local tchar = target.Character
        if tchar and tchar:FindFirstChild("HumanoidRootPart") and tchar:FindFirstChild("Humanoid") then
            local troot = tchar.HumanoidRootPart
            local thum = tchar.Humanoid

            if Ball:FindFirstChildWhichIsA("BodyVelocity") then
                Ball:FindFirstChildWhichIsA("BodyVelocity"):Destroy()
            end

            local bv = Instance.new("BodyVelocity")
            bv.Name = "FlingPower"
            bv.Velocity = Vector3.new(9e8, 9e8, 9e8)
            bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
            bv.P = 9e900
            bv.Parent = Ball

            workspace.CurrentCamera.CameraSubject = thum
            repeat
                if troot.Velocity.Magnitude > 0 then
                    local pos = troot.Position + (troot.Velocity / 1.5)
                    Ball.CFrame = CFrame.new(pos)
                    Ball.Orientation += Vector3.new(45, 60, 30)
                else
                    for _, v in pairs(tchar:GetChildren()) do
                        if v:IsA("BasePart") and v.CanCollide and not v.Anchored then
                            Ball.CFrame = v.CFrame
                            task.wait(1/6000)
                        end
                    end
                end
                task.wait(1/6000)
            until troot.Velocity.Magnitude > 1000 or thum.Health <= 0 or not tchar:IsDescendantOf(workspace) or target.Parent ~= players

            workspace.CurrentCamera.CameraSubject = humanoid
        end
    end
end

TrollTab:AddButton({
    Name = "Fling Ball",
    Description = "Lança uma bola de futebol no jogador selecionado",
    Callback = function()
        local target = Players:FindFirstChild(selectedPlayerName)
        if target then
            FlingBall(target)
        else
            warn("Nenhum jogador selecionado.")
        end
    end
})

local args = {
    [1] = "PickingTools",
    [2] = "Assault"
}

local Tab = Window:MakeTab({"Avatar"})

Tab:AddSection({ Name = "Copiar Skin" })

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Remotes = ReplicatedStorage:WaitForChild("Remotes")

local Target = nil

local function GetPlayerNames()
    local PlayerNames = {}
    for _, player in ipairs(Players:GetPlayers()) do
        table.insert(PlayerNames, player.Name)
    end
    return PlayerNames
end

local Dropdown = Tab:AddDropdown({
    Name = "Selecionar Jogador",
    Options = GetPlayerNames(),
    Default = Target,
    Callback = function(Value)
        Target = Value
    end
})

local function UpdateDropdown()
    Dropdown:Refresh(GetPlayerNames(), true)
end

Players.PlayerAdded:Connect(UpdateDropdown)
Players.PlayerRemoving:Connect(UpdateDropdown)

Tab:AddButton({
    Name = "Copiar Avatar[BETA] Bug",
    Callback = function()
        if not Target then 
            warn("Nenhum jogador selecionado")
            return 
        end

        local LP = Players.LocalPlayer
        local LChar = LP.Character
        local TPlayer = Players:FindFirstChild(Target)

        if not (TPlayer and TPlayer.Character) then
            warn("Jogador alvo inválido ou sem personagem")
            return
        end

        local LHumanoid = LChar and LChar:FindFirstChildOfClass("Humanoid")
        local THumanoid = TPlayer.Character:FindFirstChildOfClass("Humanoid")

        if not (LHumanoid and THumanoid) then
            warn("Humanoid faltando no personagem local ou alvo")
            return
        end

    -- Copiar visual do jogador alvo
        local PDesc = THumanoid:GetAppliedDescription()

        -- Aplicar partes do corpo
        local bodyParts = {
            PDesc.Torso,
            PDesc.RightArm,
            PDesc.LeftArm,
            PDesc.RightLeg,
            PDesc.LeftLeg,
            PDesc.Head
        }

        Remotes.Wear:InvokeServer(bodyParts)
        task.wait(0.3)

        -- Aplicar roupas e face
        if tonumber(PDesc.Shirt) then
            Remotes.Wear:InvokeServer(tonumber(PDesc.Shirt))
            task.wait(0.2)
        end

        if tonumber(PDesc.Pants) then
            Remotes.Wear:InvokeServer(tonumber(PDesc.Pants))
            task.wait(0.2)
        end

        if tonumber(PDesc.Face) then
            Remotes.Wear:InvokeServer(tonumber(PDesc.Face))
            task.wait(0.2)
        end

        -- Aplicar acessórios
        for _, acc in ipairs(PDesc:GetAccessories(true)) do
            if acc.AssetId and tonumber(acc.AssetId) then
                Remotes.Wear:InvokeServer(tonumber(acc.AssetId))
                task.wait(0.2)
            end
        end

        print("Avatar copiado de " .. Target)
    end
})

Tab:AddButton({
    Name = "Mais opções em breve",
    Description = "Demora um pouco para atualizar pois o Jaozin tem que ficar mais nos estudos",
    Callback = function()
        print("Ainda em desenvolvimento... Aguarde!")
    end
})

game:GetService("ReplicatedStorage").RE:FindFirstChild("1Too1l"):InvokeServer(unpack(args))

print("Zynora Hub carregado com sucesso!")
