-- Aguarda o jogo carregar completamente
repeat wait() until game:IsLoaded()

-- Variáveis principais
local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart, humanoid

-- Configuração de voo
local flying = false
local speed = 100  -- Velocidade do voo
local heightLock = 0  -- Mantém a altura inicial ao voar

-- Criando objetos de voo
local bodyVelocity = Instance.new("BodyVelocity")
local bodyGyro = Instance.new("BodyGyro")

-- Criar botão de voo na tela
local ScreenGui = Instance.new("ScreenGui")
local FlyButton = Instance.new("TextButton")

ScreenGui.Parent = game.CoreGui
FlyButton.Parent = ScreenGui
FlyButton.Size = UDim2.new(0, 200, 0, 50)
FlyButton.Position = UDim2.new(0.4, 0, 0.85, 0)  -- Ajuste a posição como quiser
FlyButton.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
FlyButton.TextSize = 20
FlyButton.Text = "Ativar Voo"
FlyButton.TextColor3 = Color3.fromRGB(255, 255, 255)

-- Função para definir o personagem corretamente após renascer
local function setupCharacter()
    character = player.Character or player.CharacterAdded:Wait()
    humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoid = character:FindFirstChildOfClass("Humanoid")

    -- Evitar morte ao cair
    humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
    humanoid:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
    humanoid:SetStateEnabled(Enum.HumanoidStateType.Freefall, false)

    -- Se estiver voando quando morrer, ativa novamente após renascer
    if flying then
        toggleFly(true)
    end
end

-- Atualiza as variáveis quando o jogador renasce
player.CharacterAdded:Connect(setupCharacter)

-- Torna o jogador imortal durante o voo
local function setImmortal(state)
    if humanoid then
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Dead, not state)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, not state)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, not state)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Physics, not state)
        humanoid.Health = if state then math.huge else 100
    end
end

-- Função para ativar/desativar voo
function toggleFly(forceEnable)
    if forceEnable then
        flying = true
    else
        flying = not flying
    end

    if flying then
        -- Define altura fixa
        heightLock = humanoidRootPart.Position.Y

        -- Configurar voo
        bodyVelocity.Parent = humanoidRootPart
        bodyVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)

        bodyGyro.Parent = humanoidRootPart
        bodyGyro.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)

        -- Torna o jogador imortal
        setImmortal(true)

        -- Congela animações (evita braços e pernas se mexendo)
        humanoid:ChangeState(Enum.HumanoidStateType.Physics)

        FlyButton.Text = "Desativar Voo"
        FlyButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
    else
        -- Desativar voo
        bodyVelocity:Destroy()
        bodyGyro:Destroy()

        -- Volta ao normal
        setImmortal(false)
        humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)

        FlyButton.Text = "Ativar Voo"
        FlyButton.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
    end
end

-- Atualizar voo conforme o analógico e a câmera
game:GetService("RunService").RenderStepped:Connect(function()
    if flying then
        local moveDirection = humanoid.MoveDirection
        local cameraDirection = workspace.CurrentCamera.CFrame.LookVector

        -- Somente voa se o jogador mover o analógico
        if moveDirection.Magnitude > 0 then
            local flyDirection = (cameraDirection * 1.5 + moveDirection).unit
            bodyVelocity.Velocity = flyDirection * speed
            bodyGyro.CFrame = CFrame.new(humanoidRootPart.Position, humanoidRootPart.Position + flyDirection)
        else
            -- Mantém a altura fixa quando parado
            bodyVelocity.Velocity = Vector3.new(0, (heightLock - humanoidRootPart.Position.Y) * 2, 0)
        end
    end
end)

-- Quando o botão for pressionado, ativa ou desativa o voo
FlyButton.MouseButton1Click:Connect(toggleFly)

-- Mensagem para confirmar ativação do script
game.StarterGui:SetCore("SendNotification", {
    Title = "Script de Voo Ativado!";
    Text = "Use o analógico e a câmera para voar.";
    Duration = 5;
})

-- Configurar personagem inicial
setupCharacter()
