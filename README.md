-- LocalScript dentro de StarterPlayerScripts
local player = game.Players.LocalPlayer
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

local savedPosition = nil
local speed = 100 -- velocidade fixa

-- Criar ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "KakaPremium"
screenGui.Parent = player:WaitForChild("PlayerGui")

-- Frame Principal (Arrastável)
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 180, 0, 220)
frame.Position = UDim2.new(0.05, 0, 0.2, 0)
frame.BackgroundColor3 = Color3.fromRGB(20,20,20)
frame.BackgroundTransparency = 0.2
frame.Active = true
frame.Draggable = false -- usaremos script custom
frame.Parent = screenGui

-- Título
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 40)
title.Text = "Kaka Premium"
title.Font = Enum.Font.GothamBold
title.TextSize = 18
title.TextColor3 = Color3.fromRGB(255,255,255)
title.BackgroundTransparency = 1
title.Parent = frame

-- Criar função para botões
local function createButton(name, text, order)
	local btn = Instance.new("TextButton")
	btn.Name = name
	btn.Size = UDim2.new(0.8, 0, 0, 50)
	btn.Position = UDim2.new(0.1, 0, 0, 40 + (order*60))
	btn.BackgroundColor3 = Color3.fromRGB(0,0,0)
	btn.TextColor3 = Color3.fromRGB(0,255,100)
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 18
	btn.Text = text
	btn.AutoButtonColor = true
	btn.Parent = frame
	btn.ClipsDescendants = true
	btn.ZIndex = 2

	-- deixar circular
	btn.SizeConstraint = Enum.SizeConstraint.RelativeYY
	btn.TextScaled = true
	btn.BackgroundTransparency = 0.1
	btn.BorderSizePixel = 0
	btn.UICorner = Instance.new("UICorner", btn)
	btn.UICorner.CornerRadius = UDim.new(1,0)

	return btn
end

-- Criar botões
local saveButton = createButton("SaveBtn", "Salvar Posição", 0)
local tpButton = createButton("TpBtn", "Teleportar", 1)
local guidedButton = createButton("GuidedBtn", "Teleguiado", 2)

-- Tela preta (Doing Steal)
local effectFrame = Instance.new("Frame")
effectFrame.Size = UDim2.new(1,0,1,0)
effectFrame.BackgroundColor3 = Color3.fromRGB(0,0,0)
effectFrame.BackgroundTransparency = 1
effectFrame.Visible = false
effectFrame.ZIndex = 10
effectFrame.Parent = screenGui

local effectText = Instance.new("TextLabel")
effectText.Size = UDim2.new(1,0,1,0)
effectText.Text = "DOING STEAL"
effectText.Font = Enum.Font.GothamBold
effectText.TextScaled = true
effectText.TextColor3 = Color3.fromRGB(255,0,0)
effectText.BackgroundTransparency = 1
effectText.Parent = effectFrame

-- Tela preta (Salvando posição)
local saveEffect = Instance.new("Frame")
saveEffect.Size = UDim2.new(1,0,1,0)
saveEffect.BackgroundColor3 = Color3.fromRGB(0,0,0)
saveEffect.BackgroundTransparency = 1
saveEffect.Visible = false
saveEffect.ZIndex = 10
saveEffect.Parent = screenGui

local saveText = Instance.new("TextLabel")
saveText.Size = UDim2.new(1,0,0.5,0)
saveText.Position = UDim2.new(0,0,0.25,0)
saveText.Text = "Salvando posição"
saveText.Font = Enum.Font.GothamBold
saveText.TextScaled = true
saveText.TextColor3 = Color3.fromRGB(255,255,255)
saveText.BackgroundTransparency = 1
saveText.Parent = saveEffect

local saveSub = Instance.new("TextLabel")
saveSub.Size = UDim2.new(1,0,0.2,0)
saveSub.Position = UDim2.new(0,0,0.7,0)
saveSub.Text = "Espere..."
saveSub.Font = Enum.Font.Gotham
saveSub.TextScaled = true
saveSub.TextColor3 = Color3.fromRGB(200,200,200)
saveSub.BackgroundTransparency = 1
saveSub.Parent = saveEffect

-- Função SALVAR POSIÇÃO
saveButton.Activated:Connect(function()
	local character = player.Character or player.CharacterAdded:Wait()
	local root = character:WaitForChild("HumanoidRootPart")

	-- Mostrar tela
	saveEffect.Visible = true
	TweenService:Create(saveEffect, TweenInfo.new(0.2), {BackgroundTransparency = 0}):Play()
	task.wait(1.5)

	savedPosition = root.CFrame

	-- Fechar tela
	TweenService:Create(saveEffect, TweenInfo.new(0.5), {BackgroundTransparency = 1}):Play()
	task.wait(0.6)
	saveEffect.Visible = false
end)

-- Função TELEPORTAR (com efeito Doing Steal)
tpButton.Activated:Connect(function()
	if savedPosition then
		local character = player.Character or player.CharacterAdded:Wait()
		local root = character:WaitForChild("HumanoidRootPart")

		effectFrame.Visible = true
		TweenService:Create(effectFrame, TweenInfo.new(0.2), {BackgroundTransparency = 0}):Play()
		task.wait(0.5)

		root.CFrame = savedPosition + Vector3.new(0,2,0)

		TweenService:Create(effectFrame, TweenInfo.new(0.5), {BackgroundTransparency = 1}):Play()
		task.wait(0.6)
		effectFrame.Visible = false
	end
end)

-- Função TELEGUIADO (sem efeito, automático)
guidedButton.Activated:Connect(function()
	if savedPosition then
		local character = player.Character or player.CharacterAdded:Wait()
		local root = character:WaitForChild("HumanoidRootPart")
		local distance = (savedPosition.Position - root.Position).Magnitude
		if distance > 10000 then
			guidedButton.Text = "Muito longe! (>10000m)"
			task.wait(1.5)
			guidedButton.Text = "Teleguiado"
			return
		end

		local duration = distance / speed
		local startCFrame = root.CFrame
		local goalCFrame = savedPosition + Vector3.new(0,5,0)

		guidedButton.Text = "Viajando..."
		local startTime = tick()
		while tick() - startTime < duration do
			local alpha = (tick() - startTime)/duration
			root.CFrame = startCFrame:Lerp(goalCFrame, alpha)
			task.wait(0.03)
		end
		root.CFrame = goalCFrame

		guidedButton.Text = "Chegou!"
		task.wait(1.5)
		guidedButton.Text = "Teleguiado"
	end
end)

-- ARRÁSTAVEL (PC + Mobile)
local dragging = false
local dragInput, dragStart, startPos

frame.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = frame.Position

		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end
		end)
	end
end)

frame.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
		dragInput = input
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if input == dragInput and dragging then
		local delta = input.Position - dragStart
		frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
	end
end)
