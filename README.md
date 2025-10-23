-- HUD Editor Hub Mobile - completo
local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer

-- Espera PlayerGui
local playerGui = player:WaitForChild("PlayerGui")

-- Cria ScreenGui
local gui = Instance.new("ScreenGui")
gui.Name = "HUDHub"
gui.Parent = playerGui
gui.ResetOnSpawn = false

-- Função helper para criar botões HUD
local function makeButton(name, text, pos)
	local btn = Instance.new("TextButton")
	btn.Name = name
	btn.Size = UDim2.new(0.15,0,0.09,0)
	btn.Position = pos
	btn.AnchorPoint = Vector2.new(0.5,0.5)
	btn.Text = text
	btn.TextScaled = true
	btn.BackgroundColor3 = Color3.fromRGB(35,35,35)
	btn.TextColor3 = Color3.new(1,1,1)
	btn.Parent = gui
	return btn
end

-- Botões HUD
local hudButtons = {
	makeButton("PlayButton","PLAY",UDim2.new(0.5,0,0.85,0)),
	makeButton("MenuButton","MENU",UDim2.new(0.15,0,0.9,0))
}

-- Cria HUB principal (Frame flutuante)
local hub = Instance.new("Frame")
hub.Size = UDim2.new(0,250,0,300)
hub.Position = UDim2.new(0.5,-125,0.05,0)
hub.AnchorPoint = Vector2.new(0,0)
hub.BackgroundColor3 = Color3.fromRGB(25,25,25)
hub.BackgroundTransparency = 0.1
hub.BorderColor3 = Color3.fromRGB(200,200,200)
hub.Parent = gui

-- Barra superior para arrastar o HUB
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1,0,0,30)
titleBar.BackgroundColor3 = Color3.fromRGB(50,50,50)
titleBar.Parent = hub

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1,0,1,0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "HUD Editor Hub"
titleLabel.TextScaled = true
titleLabel.TextColor3 = Color3.new(1,1,1)
titleLabel.Parent = titleBar

-- Função helper para criar botões no HUB
local function makeHubButton(text, y)
	local b = Instance.new("TextButton")
	b.Size = UDim2.new(1,-20,0,28)
	b.Position = UDim2.new(0,10,0,y)
	b.BackgroundColor3 = Color3.fromRGB(45,45,45)
	b.TextColor3 = Color3.new(1,1,1)
	b.Text = text
	b.TextScaled = true
	b.Parent = hub
	return b
end

-- Botões do HUB
local editBtn = makeHubButton("Entrar Modo Edição",50)
local saveBtn = makeHubButton("Salvar Layout",90)
local resetBtn = makeHubButton("Resetar Layout",130)

-- Estado de edição
local editing = false
local selectedButton = nil
local draggingHUD = false
local resizingHUD = false
local dragOffset = Vector2.new()

-- Retângulo de seleção visual
local selectionRect = Instance.new("Frame")
selectionRect.BorderSizePixel = 2
selectionRect.BorderColor3 = Color3.fromRGB(255,255,0)
selectionRect.BackgroundTransparency = 1
selectionRect.ZIndex = 10
selectionRect.Visible = false
selectionRect.Parent = gui

local function updateSelection(target)
	if target then
		selectionRect.Visible = true
		selectionRect.Position = target.Position
		selectionRect.Size = target.Size
		selectionRect.AnchorPoint = target.AnchorPoint
	else
		selectionRect.Visible = false
	end
end

-- Alterna modo edição
editBtn.Activated:Connect(function()
	editing = not editing
	editBtn.Text = editing and "Sair do Modo" or "Entrar Modo Edição"
	if not editing then
		selectedButton = nil
		updateSelection(nil)
	end
end)

-- Arrastar ou redimensionar HUD Buttons
local function startInteraction(btn, input)
	if not editing then return end
	selectedButton = btn
	updateSelection(btn)
	local abs = btn.AbsolutePosition
	local size = btn.AbsoluteSize
	local pos = input.Position
	local br = abs + size
	local dx, dy = math.abs(pos.X - br.X), math.abs(pos.Y - br.Y)
	if dx < 30 and dy < 30 then
		resizingHUD = true
	else
		draggingHUD = true
		dragOffset = pos - (abs + size/2)
	end
	input.Changed:Connect(function()
		if input.UserInputState == Enum.UserInputState.Ended then
			draggingHUD = false
			resizingHUD = false
		end
	end)
end

for _,btn in pairs(hudButtons) do
	btn.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
			startInteraction(btn,input)
		end
	end)
end

-- Movimento e resize
UIS.InputChanged:Connect(function(input)
	if not editing or not selectedButton then return end
	if input.UserInputType ~= Enum.UserInputType.Touch and input.UserInputType ~= Enum.UserInputType.MouseMovement then return end
	local pos = input.Position
	local viewport = workspace.CurrentCamera.ViewportSize
	if draggingHUD then
		local center = pos - dragOffset
		selectedButton.Position = UDim2.new(
			math.clamp(center.X/viewport.X,0.05,0.95),0,
			math.clamp(center.Y/viewport.Y,0.05,0.95),0
		)
	elseif resizingHUD then
		local abs = selectedButton.AbsolutePosition
		local newSize = Vector2.new(pos.X - abs.X,pos.Y - abs.Y)
		selectedButton.Size = UDim2.new(
			math.clamp(newSize.X/viewport.X,0.05,0.4),0,
			math.clamp(newSize.Y/viewport.Y,0.05,0.3),0
		)
	end
	updateSelection(selectedButton)
end)

-- Atualiza retângulo
RunService.RenderStepped:Connect(function()
	if selectedButton and editing then
		updateSelection(selectedButton)
	end
end)

-- Reset layout
resetBtn.Activated:Connect(function()
	hudButtons[1].Position = UDim2.new(0.5,0,0.85,0)
	hudButtons[1].Size = UDim2.new(0.15,0,0.09,0)
	hudButtons[2].Position = UDim2.new(0.15,0,0.9,0)
	hudButtons[2].Size = UDim2.new(0.15,0,0.09,0)
end)

-- Salvar layout (aqui só printa, depois dá para salvar)
saveBtn.Activated:Connect(function()
	print("Layout salvo!")
	for _,btn in pairs(hudButtons) do
		print(btn.Name, btn.Position, btn.Size)
	end
end)

-- Funções originais dos botões
hudButtons[1].Activated:Connect(function()
	if editing then return end
	print("Ação PLAY")
end)
hudButtons[2].Activated:Connect(function()
	if editing then return end
	print("Ação MENU")
end)
