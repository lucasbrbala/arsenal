--[[ Aimbot Arsenal GUI + Aimbot/ESP ]]--

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

-- Configurações do Aimbot
_G.AimbotEnabled = false
_G.TeamCheck = true
_G.AimPart = "Head"
_G.Sensitivity = 0.1
_G.FOVRadius = 50
_G.FREE_FOR_ALL = false
local discordLink = "https://discord.gg/AANkazXgKq"

-- FOV Circle
local fovCircle = Drawing.new("Circle")
fovCircle.Thickness = 1
fovCircle.NumSides = 64
fovCircle.Radius = _G.FOVRadius
fovCircle.Filled = false
fovCircle.Visible = false
fovCircle.Color = Color3.fromRGB(255, 255, 255)
fovCircle.Transparency = 0.7

-- GUI
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
ScreenGui.Name = "AimbotArsenalGUI"

local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 250, 0, 300)
Frame.Position = UDim2.new(0.02, 0, 0.2, 0)
Frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Frame.BorderSizePixel = 0
Frame.Visible = true

local Title = Instance.new("TextLabel", Frame)
Title.Size = UDim2.new(1, 0, 0, 40)
Title.BackgroundTransparency = 1
Title.Text = "Aimbot Arsenal"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 24

-- Botão padrão
local function createButton(text, posY, callback)
	local btn = Instance.new("TextButton", Frame)
	btn.Size = UDim2.new(1, -20, 0, 30)
	btn.Position = UDim2.new(0, 10, 0, posY)
	btn.Text = text
	btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
	btn.TextColor3 = Color3.new(1, 1, 1)
	btn.Font = Enum.Font.SourceSans
	btn.TextSize = 18
	btn.MouseButton1Click:Connect(callback)
	return btn
end

-- Botões do menu
createButton("FOV +", 50, function()
	_G.FOVRadius = _G.FOVRadius + 10
end)

createButton("FOV -", 90, function()
	_G.FOVRadius = math.max(10, _G.FOVRadius - 10)
end)

createButton("Toggle Team Check", 130, function()
	_G.TeamCheck = not _G.TeamCheck
end)

createButton("Discord", 170, function()
	setclipboard(discordLink)
end)

local Credit = Instance.new("TextLabel", Frame)
Credit.Size = UDim2.new(1, 0, 0, 30)
Credit.Position = UDim2.new(0, 0, 1, -30)
Credit.BackgroundTransparency = 1
Credit.Text = "Created By ttk yt"
Credit.TextColor3 = Color3.new(1, 1, 1)
Credit.Font = Enum.Font.SourceSansItalic
Credit.TextSize = 16

-- Mostrar/ocultar menu com tecla P
local showMenu = true
UserInputService.InputBegan:Connect(function(input)
	if input.KeyCode == Enum.KeyCode.P then
		showMenu = not showMenu
		Frame.Visible = showMenu
	end
end)

-- ESP
local function createESP(player)
	if player == LocalPlayer then return end
	if not player.Character then return end

	local box = Drawing.new("Square")
	box.Thickness = 2
	box.Filled = false
	box.Color = Color3.fromRGB(255, 0, 0)
	box.Transparency = 1

	local connection
	connection = RunService.RenderStepped:Connect(function()
		if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then
			box.Visible = false
			return
		end

		local rootPart = player.Character.HumanoidRootPart
		local rootPos, onScreen = Camera:WorldToViewportPoint(rootPart.Position)

		if onScreen then
			local head = player.Character:FindFirstChild("Head")
			local headPos = head and Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 1, 0)) or rootPos
			local legsPos = Camera:WorldToViewportPoint(rootPart.Position - Vector3.new(0, 3, 0))

			local height = math.abs(headPos.Y - legsPos.Y)
			local width = height * 0.5

			box.Size = Vector2.new(width, height)
			box.Position = Vector2.new(rootPos.X - width / 2, rootPos.Y - height / 2)
			box.Visible = true

			if _G.FREE_FOR_ALL or player.TeamColor ~= LocalPlayer.TeamColor then
				box.Color = Color3.fromRGB(255, 0, 0)
			else
				box.Color = Color3.fromRGB(0, 170, 255)
			end
		else
			box.Visible = false
		end
	end)

	player.CharacterAdded:Connect(function()
		box.Visible = false
	end)

	player.AncestryChanged:Connect(function()
		box:Remove()
		connection:Disconnect()
	end)
end

for _, player in pairs(Players:GetPlayers()) do
	createESP(player)
end

Players.PlayerAdded:Connect(function(player)
	player.CharacterAdded:Connect(function()
		createESP(player)
	end)
end)

-- Aimbot
local function getClosestPlayerToCursor()
	local closestPlayer = nil
	local shortestDistance = _G.FOVRadius

	for _, player in pairs(Players:GetPlayers()) do
		if player == LocalPlayer then continue end
		if not player.Character or not player.Character:FindFirstChild(_G.AimPart) then continue end
		if player.Character.Humanoid.Health <= 0 then continue end
		if _G.TeamCheck and not _G.FREE_FOR_ALL and player.TeamColor == LocalPlayer.TeamColor then continue end

		local head = player.Character[_G.AimPart]
		local screenPos, onScreen = Camera:WorldToViewportPoint(head.Position)
		local mousePos = UserInputService:GetMouseLocation()
		local distance = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude

		if onScreen and distance < shortestDistance then
			closestPlayer = player
			shortestDistance = distance
		end
	end

	return closestPlayer
end

RunService.RenderStepped:Connect(function()
	local mousePos = UserInputService:GetMouseLocation()
	fovCircle.Position = mousePos
	fovCircle.Radius = _G.FOVRadius
	fovCircle.Visible = _G.AimbotEnabled

	if _G.AimbotEnabled then
		local target = getClosestPlayerToCursor()
		if target and target.Character and target.Character:FindFirstChild(_G.AimPart) then
			local headPos = target.Character[_G.AimPart].Position
			Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, headPos), _G.Sensitivity)
		end
	end
end)

-- Tecla E ativa/desativa o Aimbot
UserInputService.InputBegan:Connect(function(input)
	if input.KeyCode == Enum.KeyCode.E then
		_G.AimbotEnabled = not _G.AimbotEnabled
		print("Aimbot " .. (_G.AimbotEnabled and "ativado" or "desativado"))
	end
end)
