-- VERSÃO DEFINITIVA: ULTRA-RÁPIDA (SEM DELAY) E COMPATÍVEL COM TODOS OS EXECUTORES
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

-- Usar PlayerGui garante que vai executar sem restrições do injetor
local PastaGui = LocalPlayer:WaitForChild("PlayerGui")

-- Evita duplicar elementos caso execute novamente
if PastaGui:FindFirstChild("PainelEspMaster") then PastaGui.PainelEspMaster:Destroy() end
if PastaGui:FindFirstChild("BotaoAlternarPainel") then PastaGui.BotaoAlternarPainel:Destroy() end

-- 1. CRIANDO O BOTÃO DE ABRIR/FECHAR
local ScreenBotao = Instance.new("ScreenGui")
ScreenBotao.Name = "BotaoAlternarPainel"
ScreenBotao.ResetOnSpawn = false
ScreenBotao.Parent = PastaGui

local BotaoAlternar = Instance.new("TextButton")
BotaoAlternar.Size = UDim2.new(0, 110, 0, 35)
BotaoAlternar.Position = UDim2.new(0, 15, 0.5, -17)
BotaoAlternar.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
BotaoAlternar.TextColor3 = Color3.fromRGB(255, 255, 255)
BotaoAlternar.Text = "Abrir Painel"
BotaoAlternar.Font = Enum.Font.SourceSansBold
BotaoAlternar.TextSize = 14
BotaoAlternar.BorderSizePixel = 0
BotaoAlternar.Parent = ScreenBotao

local UICornerBotao = Instance.new("UICorner")
UICornerBotao.CornerRadius = UDim.new(0, 6)
UICornerBotao.Parent = BotaoAlternar

-- 2. CRIANDO O PAINEL COMPACTO
local ScreenPainel = Instance.new("ScreenGui")
ScreenPainel.Name = "PainelEspMaster"
ScreenPainel.ResetOnSpawn = false
ScreenPainel.Enabled = false -- Começa fechado
ScreenPainel.Parent = PastaGui

local PainelGrande = Instance.new("Frame")
PainelGrande.Size = UDim2.new(0, 380, 0, 240)
PainelGrande.Position = UDim2.new(0.5, -190, 0.5, -120)
PainelGrande.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
PainelGrande.BorderSizePixel = 0
PainelGrande.Active = true
PainelGrande.Draggable = true
PainelGrande.Parent = ScreenPainel

local UICornerPainel = Instance.new("UICorner")
UICornerPainel.CornerRadius = UDim.new(0, 6)
UICornerPainel.Parent = PainelGrande

-- Título
local Titulo = Instance.new("TextLabel")
Titulo.Size = UDim2.new(1, 0, 0, 30)
Titulo.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
Titulo.Text = "  PAINEL ESP & INVENTÁRIO"
Titulo.TextColor3 = Color3.fromRGB(255, 255, 255)
Titulo.TextXAlignment = Enum.TextXAlignment.Left
Titulo.Font = Enum.Font.SourceSansBold
Titulo.TextSize = 14
Titulo.Parent = PainelGrande

-- Lista de Jogadores
local ListaJogadores = Instance.new("ScrollingFrame")
ListaJogadores.Size = UDim2.new(0, 160, 1, -40)
ListaJogadores.Position = UDim2.new(0, 8, 0, 35)
ListaJogadores.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
ListaJogadores.BorderSizePixel = 0
ListaJogadores.ScrollBarThickness = 4
ListaJogadores.Parent = PainelGrande

local UIListLayout = Instance.new("UIListLayout")
UIListLayout.Padding = UDim.new(0, 3)
UIListLayout.Parent = ListaJogadores

-- Tela de Informações do Inventário
local TextoInventario = Instance.new("TextLabel")
TextoInventario.Size = UDim2.new(1, -184, 1, -40)
TextoInventario.Position = UDim2.new(0, 176, 0, 35)
TextoInventario.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
TextoInventario.BorderSizePixel = 0
TextoInventario.TextColor3 = Color3.fromRGB(255, 255, 255)
TextoInventario.TextSize = 12
TextoInventario.Font = Enum.Font.SourceSans
TextoInventario.TextXAlignment = Enum.TextXAlignment.Left
TextoInventario.TextYAlignment = Enum.TextYAlignment.Top
TextoInventario.Text = " Selecione um jogador."
TextoInventario.Parent = PainelGrande

-- Botão de Alternar Visibilidade
BotaoAlternar.MouseButton1Click:Connect(function()
	ScreenPainel.Enabled = not ScreenPainel.Enabled
	BotaoAlternar.Text = ScreenPainel.Enabled and "Fechar Painel" or "Abrir Painel"
end)

-- 3. LÓGICA DO ESP E ATUALIZAÇÃO INSTANTÂNEA
local jogadorSelecionado = nil
local espAtual = nil

local function limparESP()
	if espAtual then espAtual:Destroy(); espAtual = nil end
end

-- Função super rápida para ler inventário
local function obterTextoInventario(player)
	if not player or not player.Parent then return " Nenhum selecionado." end
	local itens = {}
	local equipado = "Nenhum"
	
	if player.Character then
		for _, obj in ipairs(player.Character:GetChildren()) do
			if obj:IsA("Tool") then
				equipado = obj.Name
				table.insert(itens, obj.Name .. " [Eq]")
			end
		end
	end
	
	local backpack = player:FindFirstChild("Backpack")
	if backpack then
		for _, tool in ipairs(backpack:GetChildren()) do
			if tool:IsA("Tool") then table.insert(itens, tool.Name) end
		end
	end
	
	return string.format(" Jogador: %s\n Equipado: %s\n\n Itens:\n • %s", player.Name, equipado, #itens > 0 and table.concat(itens, "\n • ") or "Vazio")
end

local function criarESP(player)
	limparESP()
	if not player or player == LocalPlayer or not player.Character then return end
	
	-- RESPOSTA IMEDIATA: Busca a cabeça sem usar WaitForChild para cortar o delay do clique
	local head = player.Character:FindFirstChild("Head") or player.Character:FindFirstChild("HumanoidRootPart")
	if not head then return end
	
	local billboard = Instance.new("BillboardGui")
	billboard.Name = "ESP_Vermelho"
	billboard.Size = UDim2.new(0, 200, 0, 40)
	billboard.StudsOffset = Vector3.new(0, 2.5, 0)
	billboard.AlwaysOnTop = true
	
	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(1, 0, 1, 0)
	label.BackgroundTransparency = 1
	label.TextColor3 = Color3.fromRGB(255, 0, 0)
	label.TextSize = 14
	label.Font = Enum.Font.SourceSansBold
	label.TextStrokeTransparency = 0
	label.Parent = billboard
	
	billboard.Parent = head
	espAtual = billboard
	
	-- Mantém o ESP grudado se o personagem morrer
	local conexao
	conexao = RunService.RenderStepped:Connect(function()
		if not player.Parent or not player.Character or not billboard.Parent then
			conexao:Disconnect()
			return
		end
		
		local itemEquipado = "Nenhum"
		for _, obj in ipairs(player.Character:GetChildren()) do
			if obj:IsA("Tool") then
				itemEquipado = obj.Name
				break
			end
		end
		label.Text = string.format("%s (%s)", player.Name, itemEquipado)
		
		-- Atualiza o texto lateral dinamicamente
		if jogadorSelecionado == player and ScreenPainel.Enabled then
			TextoInventario.Text = obterTextoInventario(player)
		end
	end)
end

-- Lista de Interface
local function atualizarListaInterface()
	for _, child in ipairs(ListaJogadores:GetChildren()) do
		if child:IsA("TextButton") then child:Destroy() end
	end
	
	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= LocalPlayer then
			local Botao = Instance.new("TextButton")
			Botao.Size = UDim2.new(1, -6, 0, 26)
			Botao.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
			Botao.TextColor3 = Color3.fromRGB(255, 255, 255)
			Botao.Text = " " .. player.Name
			Botao.Font = Enum.Font.SourceSansBold
			Botao.TextSize = 13
			Botao.TextXAlignment = Enum.TextXAlignment.Left
			Botao.BorderSizePixel = 0
			Botao.Parent = ListaJogadores
			
			local UICornerB = Instance.new("UICorner")
			UICornerB.CornerRadius = UDim.new(0, 4)
			UICornerB.Parent = Botao
			
			-- CLIQUE INSTANTÂNEO
			Botao.MouseButton1Click:Connect(function()
				jogadorSelecionado = player
				TextoInventario.Text = obterTextoInventario(player) -- Carrega o inventário na hora
				criarESP(player) -- Ativa o ESP imediatamente
			end)
		end
	end
	ListaJogadores.CanvasSize = UDim2.new(0, 0, 0, UIListLayout.AbsoluteContentSize.Y)
end

-- Monitoramento de entrada/saída
Players.PlayerAdded:Connect(atualizarListaInterface)
Players.PlayerRemoving:Connect(function(player)
	if jogadorSelecionado == player then
		limparESP()
		jogadorSelecionado = nil
		TextoInventario.Text = " O jogador saiu."
	end
	atualizarListaInterface()
end)

-- Cria conexões de renascimento para atualizar o ESP instantaneamente se o alvo morrer
Players.PlayerAdded:Connect(function(player)
	player.CharacterAdded:Connect(function()
		if jogadorSelecionado == player then
			task.wait(0.1) -- micro delay apenas para o jogo carregar o corpo
			criarESP(player)
		end
	end)
end)

for _, p in ipairs(Players:GetPlayers()) do
	p.CharacterAdded:Connect(function()
		if jogadorSelecionado == p then
			task.wait(0.1)
			criarESP(p)
		end
	end)
end

atualizarListaInterface()
