getgenv().NomesDeAnimaisDeAlvo = {
    "Graipuss Medussi",
    "Trenostruzza Turbo 3000",
    "Odin Din Din Dun",
    "Os Tralaleritos",
    "Sammyni Spyderini",
    "Tio Samito",
    "Garama e Madundung",
    "Mateus",
    "La Vacca Saturno Saturnita",
    "A Grande Combinação"
}

Jogadores locais = jogo:GetService("Jogadores")
local LocalPlayer = Jogadores.LocalPlayer
local RunService = jogo:GetService("RunService")
Serviço de Teleporte local = jogo:GetService("Serviço de Teleporte")
local HttpService = jogo:GetService("HttpService")
UIS local = jogo:GetService("UserInputService")
Câmera local = workspace.CurrentCamera

alvos locais = getgenv().TargetPetNames

lúpulo local = 0
locais visitadosJobIds = {[game.JobId] = true}
parada localHopping = falso
teletransporte localFails = 0
maxTeleportRetries local = 3

Animais de estimação detectados localmente = {}
Outdoors de animais de estimação locais = {}

-- ========== SERVERHOP E PET ESP ==========
função serverHop(força)
    se parar de pular e não forçar, então retorne fim
    PlaceId local, JobId = game.PlaceId, game.JobId
    tentativa local, foundServer = 0, falso
    tentativas máximas locais = 15
    enquanto não for encontradoServidor e tentar < maxAttempts fazer
        tentativa += 1
        tarefa.espera(0,5)
        cursor local = nulo
        página localTries = 0
        enquanto pageTries < 5 faz
            pageTries += 1
            URL local = "https://games.roblox.com/v1/games/" .. PlaceId .. "/servers/Public?sortOrder=Asc&limit=100"
            se cursor então url ..= "&cursor=" .. cursor fim
            httpSuccess local, resposta = pcall(função()
                retornar HttpService:JSONDecode(jogo:HttpGet(url))
            fim)
            se httpSuccess e resposta e response.data então
                para _, servidor em ipairs(response.data) faça
                    se tonumber(server.playing ou 0) < tonumber(server.maxPlayers ou 1)
                        e server.id ~= JobId
                        e não visitouJobIds[server.id] então
                        visitedJobIds[server.id] = verdadeiro
                        lúpulo += 1
                        se lúpulo >= 50 então
                            visitedJobIds = {[JobId] = verdadeiro}
                            lúpulo = 0
                        fim
                        TeleportService:TeleportToPlaceInstance(PlaceId, server.id)
                        retornar
                    fim
                fim
                cursor = resposta.nextPageCursor
                se não for cursor então quebra fim
            outro
                tarefa.espera(0,2)
            fim
        fim
    fim
    TeleportService:Teleport(PlaceId)
fim

TeleportService.TeleportInitFailed:Connect(função(_, resultado)
    teletransporte falha += 1
    se teleportFails >= maxTeleportRetries então
        teleportFails = 0
        tarefa.espera(1)
        TeleportService:Teleport(jogo.PlaceId)
    outro
        tarefa.espera(0,5)
        serverHop(verdadeiro)
    fim
fim)

função local addPetBillboard(modelo)
    se não for modelo ou modelo:FindFirstChild("PetESP") então retorne fim
    Outdoor local = Instância.new("BillboardGui")
    Billboard.Nome = "PetESP"
    Billboard.Adornee = modelo
    Outdoor.Tamanho = UDim2.novo(0, 120, 0, 40)
    Billboard.StudsOffset = Vetor3.novo(0, 4, 0)
    Billboard.AlwaysOnTop = verdadeiro
    Billboard.Parent = modelo
    Rótulo local = Instance.new("TextLabel")
    Rótulo.Tamanho = UDim2.novo(1, 0, 1, 0)
    Rótulo.TransparênciaDeFundo = 1
    Rótulo.Texto = "ðŸ µ " .. modelo.Nome
    Label.TextColor3 = Color3.fromRGB(255, 0, 0)
    Label.TextStrokeTransparency = 0,4
    Rótulo.Fonte = Enum.Fonte.GothamBold
    Label.TextScaled = verdadeiro
    Rótulo.Parente = Outdoor
    petBillboards[modelo] = Outdoor
fim

função local checkForPets()
    local encontrado = {}
    para _, obj em pares(workspace:GetDescendants()) faça
        se obj:IsA("Modelo") e obj.Nome então
            para _, alvo em pares(targetPets) faça
                se string.lower(obj.Name):find(string.lower(target)) e não obj:FindFirstChild("PetESP") então
                    addPetBillboard(obj)
                    tabela.insert(encontrado, obj.Nome)
                    stopHopping = verdadeiro
                    quebrar
                fim
            fim
        fim
    fim
    retorno encontrado
fim

espaço de trabalho.DescendantAdded:Connect(função(obj)
    tarefa.espera(0,1)
    se obj:IsA("Modelo") e obj.Nome então
        para _, alvo em pares(targetPets) faça
            se string.lower(obj.Name):find(string.lower(target)) e não obj:FindFirstChild("PetESP") então
                se não for detectadoPets[obj.Name] então
                    detectedPets[obj.Name] = verdadeiro
                    addPetBillboard(obj)
                    stopHopping = verdadeiro
                fim
                quebrar
            fim
        fim
    fim
fim)
-- ========== FIM SERVERHOP E PET ESP ==========

pcall(função()
    local antigo = LocalPlayer.PlayerGui:FindFirstChild("DeltaMainGUI")
    se antigo então antigo:Destroy() fim
fim)

interface gráfica local = Instance.new("ScreenGui")
gui.Nome = "DeltaMainGUI"
gui.ResetOnSpawn = falso
gui.IgnoreGuiInset = verdadeiro
gui.Parent = LocalPlayer.PlayerGui

quadro local = Instance.new("Quadro")
frame.Size = UDim2.new(0, 220, 0, 272)
frame.Posição = UDim2.novo(0,5, -110, 0,61, 0)
frame.BackgroundColor3 = Cor3.fromRGB(35,35,35)
frame.Active = verdadeiro
frame.Draggable = verdadeiro
frame.Parent = gui

canto local = Instance.new("UICorner")
canto.CornerRadius = UDim.new(0, 18)
canto.Parent = quadro

-- Quadros de conteúdo por aba
quadros de guia locais = {}
para tab = 1, 3 faça
    tabFrame local = Instance.new("Quadro")
    tabFrame.Name = "TabFrame" .. guia
    tabFrame.Size = UDim2.new(1, 0, 1, -45)
    tabFrame.Position = UDim2.new(0, 0, 0, 30)
    tabFrame.BackgroundTransparency = 1
    tabFrame.Visible = (tab == 1)
    tabFrame.Parent = quadro
    tabFrames[tab] = tabFrame
fim

-- Título: FAST HUB em vermelho
Título superior local = Instância.new("TextLabel")
topTitle.Size = UDim2.new(1, 0, 0, 28)
topTitle.Position = UDim2.new(0, 0, 0, 0)
topTitle.BackgroundTransparency = 1
topTitle.Text = "CENTRAL RÁPIDA"
topTitle.Font = Enum.Font.GothamBlack
topTitle.TextSize = 24
topTitle.TextColor3 = Color3.fromRGB(255, 40, 40)
topTitle.Parent = quadro

-- Nome da aba "Speed"
speedTitle local = Instance.new("TextLabel")
speedTitle.Size = UDim2.new(1, 0, 0, 20)
speedTitle.Position = UDim2.new(0, 0, 0, 25)
speedTitle.BackgroundTransparency = 1
speedTitle.Text = "Velocidade"
speedTitle.Font = Enum.Font.GothamBold
speedTitle.TextSize = 17
speedTitle.TextColor3 = Color3.fromRGB(60, 220, 255)
speedTitle.Parent = tabFrames[1]

-- Botão SPEED (alternar)
speedBtn local = Instance.new("TextButton")
speedBtn.Size = UDim2.new(1, -32, 0, 40)
speedBtn.Posição = UDim2.novo(0, 16, 0, 50-30)
speedBtn.BackgroundColor3 = Cor3.fromRGB(60, 180, 60)
speedBtn.Text = "ATIVAR VELOCIDADE"
speedBtn.Font = Enum.Font.GothamMedium
speedBtn.TextSize = 19
speedBtn.TextColor3 = Color3.fromRGB(255.255.255)
speedBtn.AutoButtonColor = verdadeiro
speedBtn.Parent = tabFrames[1]

btnCorner local = Instance.new("UICorner")
btnCorner.CornerRadius = UDim.new(0, 12)
btnCorner.Parent = speedBtn

velocidade localActive = falso
conexão de velocidade local

velocidade base local = 16
velocidade desejada local = 48
multiplicador local = (velocidade desejada/velocidade base) - 1
microStep local = 0,23

função local setSpeed(estado)
    se speedActive == estado então retorne fim
    velocidadeAtiva = estado
    se speedActive então
        se speedConnection então pcall(function() speedConnection:Disconnect() fim) fim
        speedConnection = RunService.Heartbeat:Connect(função()
            char local = LocalPlayer.Character
            hum local = char e char:FindFirstChildOfClass("Humanoid")
            hrp local = char e char:FindFirstChild("HumanoidRootPart")
            se hum e hrp e hum.MoveDirection.Magnitude > 0 então
                direção local = hum.MoveDirection.Unit
                extra local = direção * microStep * multiplicador
                hrp.CFrame = hrp.CFrame + extra
            fim
        fim)
        speedBtn.Text = "DESATIVAR VELOCIDADE"
        speedBtn.BackgroundColor3 = Cor3.fromRGB(200, 65, 65)
    outro
        se speedConnection então pcall(function() speedConnection:Disconnect() fim) speedConnection = nulo fim
        speedBtn.Text = "ATIVAR VELOCIDADE"
        speedBtn.BackgroundColor3 = Cor3.fromRGB(60, 180, 60)
    fim
fim

speedBtn.MouseButton1Click:Conectar(função()
    setSpeed(não speedActive)
fim)

LocalPlayer.CharacterAdded:Connect(função()
    setSpeed(falso)
fim)

-- Botão TP (toggle:barreira <-> chão)
tpBtn local = Instância.new("TextButton")
tpBtn.Size = UDim2.new(1, -32, 0, 40)
tpBtn.Posição = UDim2.novo(0, 16, 0, 100-30)
tpBtn.BackgroundColor3 = Cor3.fromRGB(60, 180, 60)
tpBtn.Text = "TP (PARA BARREIRA)"
tpBtn.Font = Enum.Font.GothamMedium
tpBtn.TextSize = 17
tpBtn.TextColor3 = Color3.fromRGB(255.255.255)
tpBtn.AutoButtonColor = verdadeiro
tpBtn.Parent = tabFrames[1]

local tpCorner = Instância.new("UICorner")
tpCorner.CornerRadius = UDim.new(0, 12)
tpCorner.Parent = tpBtn

local tpNaBarreira = falso
altura localBarreira = 160
altura localChao = 3

tpBtn.MouseButton1Click:Conectar(função()
    char local = LocalPlayer.Character
    hrp local = char e char:FindFirstChild("HumanoidRootPart")
    se hrp então
        se não tpNaBarreira então
            hrp.CFrame = CFrame.new(hrp.Position.X, alturaBarreira, hrp.Position.Z)
            tpNaBarreira = verdadeiro
            tpBtn.Text = "TP (VOLTA CHÃO)"
            tpBtn.BackgroundColor3 = Cor3.fromRGB(200, 65, 65)
        outro
            hrp.CFrame = CFrame.new(hrp.Position.X, alturaChao, hrp.Position.Z)
            tpNaBarreira = falso
            tpBtn.Text = "TP (PARA BARREIRA)"
            tpBtn.BackgroundColor3 = Cor3.fromRGB(60, 180, 60)
        fim
    fim
fim)

LocalPlayer.CharacterAdded:Connect(função()
    tpNaBarreira = falso
    tpBtn.Text = "TP (PARA BARREIRA)"
    tpBtn.BackgroundColor3 = Cor3.fromRGB(60, 180, 60)
fim)

-- Botão SERVER HOP
servidor localHopBtn = Instance.new("TextButton")
serverHopBtn.Size = UDim2.new(1, -32, 0, 40)
serverHopBtn.Position = UDim2.new(0, 16, 0, 150-30)
serverHopBtn.BackgroundColor3 = Cor3.fromRGB(60, 180, 60)
serverHopBtn.Text = "ATIVAR HOP DO SERVIDOR"
serverHopBtn.Font = Enum.Font.GothamMedium
serverHopBtn.TextSize = 17
serverHopBtn.TextColor3 = Color3.fromRGB(255.255.255)
serverHopBtn.AutoButtonColor = verdadeiro
serverHopBtn.Parent = tabFrames[1]

shCorner local = Instance.new("UICorner")
shCorner.CornerRadius = UDim.new(0, 12)
shCorner.Parent = serverHopBtn

servidor localHopActive = falso

serverHopBtn.MouseButton1Click:Conectar(função()
    serverHopActive = não serverHopActive
    se serverHopActive então
        serverHopBtn.Text = "DESATIVAR HOP DO SERVIDOR"
        serverHopBtn.BackgroundColor3 = Cor3.fromRGB(200, 65, 65)
        stopHopping = falso
        serverHop(verdadeiro)
    outro
        serverHopBtn.Text = "ATIVAR HOP DO SERVIDOR"
        serverHopBtn.BackgroundColor3 = Cor3.fromRGB(60, 180, 60)
        stopHopping = verdadeiro
    fim
fim)

-- ========== ABA 2: JOGADOR ESP + NOME ESP ==========
espFrame local = tabFrames[2]

-- ESP Player (Destaque)
espBtn local = Instance.new("TextButton")
espBtn.Size = UDim2.new(1, -32, 0, 36)
espBtn.Posição = UDim2.novo(0, 16, 0, 24)
espBtn.BackgroundColor3 = Cor3.fromRGB(60, 180, 60)
espBtn.Text = "Jogador ESP"
espBtn.Font = Enum.Font.GothamMedium
espBtn.TextSize = 16
espBtn.TextColor3 = Color3.fromRGB(255.255.255)
espBtn.AutoButtonColor = verdadeiro
espBtn.Parent = espFrame

espBtnCorner local = Instance.new("UICorner")
espBtnCorner.CornerRadius = UDim.new(0, 12)
espBtnCorner.Parent = espBtn

espActive local = falso
playerESP_Highlights local = {}

função local removeAllESP()
    para o jogador, destaque em pares (playerESP_Highlights)
        se destacar e destacar.Pai então
            destaque:Destruir()
        fim
    fim
    playerESP_Highlights = {}
fim

função local enableESPPlayers()
    removeAllESP()
    para _, jogador em ipairs(Players:GetPlayers()) faça
        se jogador ~= LocalPlayer e jogador.Character e jogador.Character:FindFirstChild("HumanoidRootPart") então
            char local = jogador.Personagem
            destaque local = Instance.new("Destaque")
            destaque.Adornee = char
            destaque.FillColor = Color3.fromRGB(255,0,0)
            destaque.FillTransparency = 0,8
            destaque.OutlineColor = Color3.fromRGB(255,0,0)
            destaque.Transparência do contorno = 0,6
            destaque.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
            destaque.Parent = char
            playerESP_Highlights[jogador] = destaque
        fim
    fim
fim

espBtn.MouseButton1Click:Conectar(função()
    espActive = não espActive
    se espActive então
        espBtn.Text = "Desligar ESP Player"
        espBtn.BackgroundColor3 = Cor3.fromRGB(200, 65, 65)
        enableESPPlayers()
    outro
        espBtn.Text = "Jogador ESP"
        espBtn.BackgroundColor3 = Cor3.fromRGB(60, 180, 60)
        removeAllESP()
    fim
fim)

Jogadores.JogadorAdicionado:Conectar(função(jogador)
    se espActive então
        player.CharacterAdded:Connect(função()
            tarefa.espera(0,3)
            enableESPPlayers()
        fim)
    fim
fim)
Jogadores.PlayerRemoving:Connect(função(jogador)
    se playerESP_Highlights[jogador] então
        playerESP_Highlights[jogador]:Destruir()
        playerESP_Highlights[jogador] = nulo
    fim
fim)
LocalPlayer.CharacterAdded:Connect(função()
    se espActive então
        tarefa.espera(0,3)
        enableESPPlayers()
    fim
fim)

-- Nome ESP
espNameBtn local = Instance.new("TextButton")
espNameBtn.Size = UDim2.new(1, -32, 0, 36)
espNameBtn.Position = UDim2.new(0, 16, 0, 72)
espNameBtn.BackgroundColor3 = Cor3.fromRGB(60, 180, 60)
espNameBtn.Text = "Nome ESP"
espNameBtn.Font = Enum.Font.GothamMedium
espNameBtn.TextSize = 16
espNameBtn.TextColor3 = Color3.fromRGB(255.255.255)
espNameBtn.AutoButtonColor = verdadeiro
espNameBtn.Parent = espFrame

espNameBtnCorner local = Instance.new("UICorner")
espNameBtnCorner.CornerRadius = UDim.new(0, 12)
espNameBtnCorner.Parent = espNameBtn

espNameActive local = falso
playerBillboardNames local = {}

função local removeAllBillboardNames()
    para jogador, gui em pares (playerBillboardNames) faça
        se gui e gui.Parent então
            gui:Destruir()
        fim
    fim
    playerBillboardNames = {}
fim

função local enableESPNames()
    removeAllBillboardNames()
    para _, jogador em ipairs(Players:GetPlayers()) faça
        se jogador ~= LocalPlayer e jogador.Character e jogador.Character:FindFirstChild("Cabeça") então
            cabeça local = jogador.Personagem.Cabeça
            bb local = Instância.new("BillboardGui")
            bb.Nome = "NomeESP"
            bb.Adornee = cabeça
            bb.Tamanho = UDim2.novo(0, 110, 0, 18)
            bb.StudsOffset = Vetor3.novo(0, 2.3, 0)
            bb.AlwaysOnTop = verdadeiro
            bb.Parent = cabeça

            txt local = Instância.new("TextLabel")
            txt.Tamanho = UDim2.new(1, 0, 1, 0)
            txt.TransparênciaDeFundo = 1
            txt.Texto = jogador.Nome
            txt.TextColor3 = Color3.fromRGB(255, 30, 30)
            txt.TextTransparency = 0.25 -- vermelho bem fraco
            txt.Font = Enum.Font.GothamSemibold
            txt.TextStrokeTransparency = 0,8
            txt.TextScaled = verdadeiro
            txt.Parent = bb

            playerBillboardNames[jogador] = bb
        fim
    fim
fim

espNameBtn.MouseButton1Click:Conectar(função()
    espNameActive = não espNameActive
    se espNameActive então
        espNameBtn.Text = "Desativar nome do ESP"
        espNameBtn.BackgroundColor3 = Cor3.fromRGB(200, 65, 65)
        enableESPNames()
    outro
        espNameBtn.Text = "Nome ESP"
        espNameBtn.BackgroundColor3 = Cor3.fromRGB(60, 180, 60)
        removeAllBillboardNames()
    fim
fim)

Jogadores.JogadorAdicionado:Conectar(função(jogador)
    se espNameActive então
        player.CharacterAdded:Connect(função()
            tarefa.espera(0,3)
            enableESPNames()
        fim)
    fim
fim)
Jogadores.PlayerRemoving:Connect(função(jogador)
    se playerBillboardNames[jogador] então
        playerBillboardNames[jogador]:Destruir()
        playerBillboardNames[jogador] = nulo
    fim
fim)
LocalPlayer.CharacterAdded:Connect(função()
    se espNameActive então
        tarefa.espera(0,3)
        enableESPNames()
    fim
fim)

-- ========== ABAS PEQUENAS REDONDAS LADO A LADO ==========
local abaNomes = {"MAIN","MISC","OUTROS"}
abas locais = {}
abaSelecionada local = 1
corBranco local = Color3.fromRGB(255,255,255)
corSelecionado local = Color3.fromRGB(222,222,222)

espaço total local = 220 -- largura do quadro
btnPadding local = 10
btnSize local = 32
totalBtnW local = tamanhoBtn * 3 + preenchimentoBtn * 2
margem localEsquerda = math.floor((espaço total - totalBtnW)/2)

para i=1,3 faça
    botão local = Instance.new("TextButton")
    btn.Nome = "AbaBtn"..i
    btn.Tamanho = UDim2.novo(0, tamanho do btn, 0, tamanho do btn)
    btn.Posição = UDim2.new(0, margemEsquerda + (i-1)*(tamanhobtn+preenchimentobtn), 1, -tamanhobtn-8)
    btn.BackgroundColor3 = (i == abaSelecionada) e corSelecionado ou corBranco
    btn.Texto = abaNomes[i]
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 10
    btn.TextColor3 = Color3.fromRGB(25,25,25)
    btn.AutoButtonColor = verdadeiro
    btn.Parent = quadro
    btnCorner local = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(1,0)
    btnCorner.Parent = btn
    abas[i] = btn

    btn.MouseButton1Click:Conectar(função()
        para j=1,3 faça
            tabFrames[j].Visível = falso
            abas[j].BackgroundColor3 = corBranco
        fim
        tabFrames[i].Visible = verdadeiro
        btn.BackgroundColor3 = corSelecionado
        abaSelecionada = i
    fim)
fim

tarefa.spawn(função()
    tarefa.esperar(3)
    animais de estimação locais = checkForPets()
    se #pets > 0 então
        para _, nome em ipairs(animais de estimação) faça
            detectedPets[nome] = verdadeiro
        fim
    outro
        se serverHopActive então
            serverHop(falso)
        fim
    fim
fim)
