-- // MAKIMA ANTI BAT RECODE (UPDATED)
-- // Theme: Red & White
-- // Now includes: Infinite Jump & Perfectly Sized Minimized feature

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")

local LocalPlayer = Players.LocalPlayer

local PredictionBreakerEnabled = false
local Connection = nil
local UILocked = false

-- // Infinite Jump Variables
local InfJumpEnabled = false
local jumpForce = 50
local clampFallSpeed = 80
local InfJumpHeartbeat = nil
local InfJumpRequest = nil

local function ToggleInfJump(enabled)
    InfJumpEnabled = enabled

    if enabled then
        -- Clamp fall speed
        if InfJumpHeartbeat then InfJumpHeartbeat:Disconnect() end
        InfJumpHeartbeat = RunService.Heartbeat:Connect(function()
            local char = LocalPlayer.Character
            if not char then return end
            local hrp = char:FindFirstChild("HumanoidRootPart")
            if hrp and hrp.Velocity.Y < -clampFallSpeed then
                hrp.Velocity = Vector3.new(hrp.Velocity.X, -clampFallSpeed, hrp.Velocity.Z)
            end
        end)

        -- Infinite jump on JumpRequest
        if InfJumpRequest then InfJumpRequest:Disconnect() end
        InfJumpRequest = UserInputService.JumpRequest:Connect(function()
            if not InfJumpEnabled then return end
            local char = LocalPlayer.Character
            if not char then return end
            local hrp = char:FindFirstChild("HumanoidRootPart")
            if hrp then
                hrp.Velocity = Vector3.new(hrp.Velocity.X, jumpForce, hrp.Velocity.Z)
            end
        end)
    else
        if InfJumpHeartbeat then
            InfJumpHeartbeat:Disconnect()
            InfJumpHeartbeat = nil
        end
        if InfJumpRequest then
            InfJumpRequest:Disconnect()
            InfJumpRequest = nil
        end
    end
end

local function TogglePredictionBreaker(enabled)
    PredictionBreakerEnabled = enabled

    if enabled then
        if Connection then
            Connection:Disconnect()
        end

        Connection = RunService.Heartbeat:Connect(function()
            local character = LocalPlayer.Character
            if character and character:FindFirstChild("HumanoidRootPart") then
                local rootPart = character.HumanoidRootPart
                local originalVelocity = rootPart.Velocity
                
                -- Set high velocity on X and Z, but keep the current Y velocity
                rootPart.Velocity = Vector3.new(1000, rootPart.Velocity.Y, 1000)
                
                RunService.RenderStepped:Wait()
                
                -- Restore original X and Z, but KEEP the updated Y velocity so Inf Jump works
                if rootPart and rootPart.Parent then
                    rootPart.Velocity = Vector3.new(originalVelocity.X, rootPart.Velocity.Y, originalVelocity.Z)
                end
            end
        end)
    else
        if Connection then
            Connection:Disconnect()
            Connection = nil
        end
    end
end

-- // Color Palette
local MakimaColors = {
    MainBG = Color3.fromRGB(255, 255, 255),
    AccentText = Color3.fromRGB(200, 20, 40),
    ContainerBG = Color3.fromRGB(255, 255, 255),
    Border = Color3.fromRGB(200, 20, 40),
    Knob = Color3.fromRGB(200, 20, 40),
    PillBG = Color3.fromRGB(255, 255, 255)
}

-- // Background image (Discord CDN link)
local BG_IMAGE_URL = "https://cdn.discordapp.com/attachments/1551104742770348124/1563292463643238400/bf1292d41901dda345f84f6b92b25e18.jpg?ex=6ade6a8f&is=6add190f&hm=4877e237cdaefbdfbd8112f3f62b7e36bc6a1a86c02ee2da858c540187162011&"
local BG_IMAGE_FILE = "makima_bg_v2.jpg"

-- // UI Setup
local Folder = Instance.new("Folder")
Folder.Name = "MakimaUI_Hidden"
Folder.Parent = CoreGui

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MakimaAntiBat"
ScreenGui.ResetOnSpawn = false
ScreenGui.IgnoreGuiInset = true
ScreenGui.DisplayOrder = 999
ScreenGui.Parent = Folder

-- // Frame
local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 240, 0, 270) -- Increased to fit new elements
Frame.Position = UDim2.new(0.5, -120, 0.5, -135)
Frame.BackgroundColor3 = MakimaColors.MainBG
Frame.BorderSizePixel = 0
Frame.ClipsDescendants = true
Frame.Active = true
Frame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 16)
UICorner.Parent = Frame

local UIStroke = Instance.new("UIStroke")
UIStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
UIStroke.Thickness = 1.5
UIStroke.Color = MakimaColors.Border
UIStroke.Parent = Frame

-- // Background Image
local BGImage = Instance.new("ImageLabel")
BGImage.Name = "BGImage"
BGImage.BackgroundTransparency = 1
BGImage.Size = UDim2.new(1, 0, 1, 0)
BGImage.ScaleType = Enum.ScaleType.Crop
BGImage.ImageTransparency = 0.25
BGImage.ZIndex = 0
BGImage.Parent = Frame

local BGCorner = Instance.new("UICorner")
BGCorner.CornerRadius = UDim.new(0, 16)
BGCorner.Parent = BGImage

-- Detecta se os bytes baixados sao mesmo uma imagem que o Roblox aceita (PNG ou JPEG)
local function isValidImage(data)
    if type(data) ~= "string" or #data < 100 then return false end
    local b1, b2, b3 = data:byte(1, 3)
    local isPNG = (b1 == 0x89 and b2 == 0x50 and b3 == 0x4E)
    local isJPG = (b1 == 0xFF and b2 == 0xD8 and b3 == 0xFF)
    return isPNG or isJPG
end

local function downloadImage(url)
    -- 1) request / http_request (funciona na maioria dos executores)
    local req = (syn and syn.request) or (http and http.request) or http_request or request
    if req then
        local ok, res = pcall(req, {
            Url = url,
            Method = "GET",
            Headers = { ["User-Agent"] = "Mozilla/5.0" }
        })
        if ok and res and res.Body then
            if isValidImage(res.Body) then return res.Body end
            warn("[Makima] request retornou algo que nao e imagem (status " .. tostring(res.StatusCode) .. "). Formato webp ou link expirado?")
        end
    end
    -- 2) game:HttpGet como alternativa
    local ok2, body = pcall(function() return game:HttpGet(url) end)
    if ok2 and isValidImage(body) then return body end
    return nil
end

task.spawn(function()
    local ok, err = pcall(function()
        if not (writefile and getcustomasset) then
            error("seu executor nao tem writefile/getcustomasset")
        end

        -- Se ja existe cache, so usa se for uma imagem valida (apaga lixo de tentativas antigas)
        if isfile and isfile(BG_IMAGE_FILE) then
            local cached = readfile(BG_IMAGE_FILE)
            if not isValidImage(cached) then
                if delfile then delfile(BG_IMAGE_FILE) end
            end
        end

        if not (isfile and isfile(BG_IMAGE_FILE)) then
            local data = downloadImage(BG_IMAGE_URL)
            if not data then error("falha ao baixar a imagem (link expirado, bloqueado ou formato nao suportado)") end
            writefile(BG_IMAGE_FILE, data)
        end

        BGImage.Image = getcustomasset(BG_IMAGE_FILE)
    end)
    if not ok then
        warn("[Makima] Imagem de fundo nao carregou: " .. tostring(err))
    end
end)

-- // Title Bar
local TitleBar = Instance.new("Frame")
TitleBar.BackgroundTransparency = 1
TitleBar.BorderSizePixel = 0
TitleBar.Position = UDim2.new(0, 0, 0, 0)
TitleBar.Size = UDim2.new(1, 0, 0, 45)
TitleBar.ZIndex = 2
TitleBar.Parent = Frame

local Separator = Instance.new("Frame")
Separator.BackgroundColor3 = MakimaColors.Border
Separator.BorderSizePixel = 0
Separator.Position = UDim2.new(0, 12, 0, 45)
Separator.Size = UDim2.new(1, -24, 0, 1)
Separator.ZIndex = 2
Separator.Parent = Frame

local TitleLabel = Instance.new("TextLabel")
TitleLabel.BackgroundTransparency = 1
TitleLabel.Position = UDim2.new(0, 16, 0, 0)
TitleLabel.Size = UDim2.new(1, -110, 1, 0)
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.Text = "makima | anti bat"
TitleLabel.TextColor3 = MakimaColors.AccentText
TitleLabel.TextSize = 13
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
TitleLabel.ZIndex = 3
TitleLabel.Parent = TitleBar

-- // Close Button
local CloseBtn = Instance.new("TextButton")
CloseBtn.BackgroundColor3 = MakimaColors.AccentText
CloseBtn.Position = UDim2.new(1, -42, 0.5, -14)
CloseBtn.Size = UDim2.new(0, 28, 0, 28)
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.Text = "X"
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.TextSize = 13
CloseBtn.AutoButtonColor = true
CloseBtn.ZIndex = 3
CloseBtn.Parent = TitleBar

local CloseCorner = Instance.new("UICorner")
CloseCorner.CornerRadius = UDim.new(0, 8)
CloseCorner.Parent = CloseBtn

local CloseStroke = Instance.new("UIStroke")
CloseStroke.Color = MakimaColors.Border
CloseStroke.Thickness = 1
CloseStroke.Parent = CloseBtn

CloseBtn.MouseButton1Click:Connect(function()
    TogglePredictionBreaker(false)
    ToggleInfJump(false)
    ScreenGui:Destroy()
end)

-- // Minimize Button
local MinimizeBtn = Instance.new("TextButton")
MinimizeBtn.BackgroundColor3 = MakimaColors.AccentText
MinimizeBtn.Position = UDim2.new(1, -76, 0.5, -14)
MinimizeBtn.Size = UDim2.new(0, 28, 0, 28)
MinimizeBtn.Font = Enum.Font.GothamBold
MinimizeBtn.Text = "▼"
MinimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
MinimizeBtn.TextSize = 13
MinimizeBtn.AutoButtonColor = true
MinimizeBtn.ZIndex = 3
MinimizeBtn.Parent = TitleBar

local MinimizeCorner = Instance.new("UICorner")
MinimizeCorner.CornerRadius = UDim.new(0, 8)
MinimizeCorner.Parent = MinimizeBtn

local MinimizeStroke = Instance.new("UIStroke")
MinimizeStroke.Color = MakimaColors.Border
MinimizeStroke.Thickness = 1
MinimizeStroke.Parent = MinimizeBtn

-- // Properly calculated heights so Inf Jump completely vanishes
local isMinimized = false
local expandedSize = UDim2.new(0, 240, 0, 270)
local collapsedSize = UDim2.new(0, 240, 0, 110) 

MinimizeBtn.MouseButton1Click:Connect(function()
    isMinimized = not isMinimized
    MinimizeBtn.Text = isMinimized and "▲" or "▼"
    
    TweenService:Create(Frame, TweenInfo.new(0.3, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {
        Size = isMinimized and collapsedSize or expandedSize
    }):Play()
end)

-- // Content Frame
local ContentFrame = Instance.new("Frame")
ContentFrame.BackgroundTransparency = 1
ContentFrame.Position = UDim2.new(0, 0, 0, 46)
ContentFrame.Size = UDim2.new(1, 0, 1, -46)
ContentFrame.ZIndex = 2
ContentFrame.Parent = Frame

-- // ─── Anti Bat Toggle Row ───
local ToggleRow = Instance.new("Frame")
ToggleRow.Size = UDim2.new(1, -24, 0, 42)
ToggleRow.Position = UDim2.new(0, 12, 0, 12)
ToggleRow.BackgroundColor3 = MakimaColors.ContainerBG
ToggleRow.BackgroundTransparency = 0.15
ToggleRow.BorderSizePixel = 0
ToggleRow.ZIndex = 2
ToggleRow.Parent = ContentFrame

local ToggleRowCorner = Instance.new("UICorner")
ToggleRowCorner.CornerRadius = UDim.new(0, 12)
ToggleRowCorner.Parent = ToggleRow

local ToggleRowStroke = Instance.new("UIStroke")
ToggleRowStroke.Color = MakimaColors.Border
ToggleRowStroke.Thickness = 1
ToggleRowStroke.Parent = ToggleRow

local StatusLabel = Instance.new("TextLabel")
StatusLabel.BackgroundTransparency = 1
StatusLabel.TextXAlignment = Enum.TextXAlignment.Left
StatusLabel.Font = Enum.Font.GothamBold
StatusLabel.TextColor3 = MakimaColors.AccentText
StatusLabel.Position = UDim2.new(0, 12, 0, 0)
StatusLabel.Text = "anti bat : OFF"
StatusLabel.TextSize = 13
StatusLabel.Size = UDim2.new(1, -80, 1, 0)
StatusLabel.ZIndex = 3
StatusLabel.Parent = ToggleRow

local pillW, pillH, knobSz = 48, 24, 18

local pill = Instance.new("Frame")
pill.Size = UDim2.new(0, pillW, 0, pillH)
pill.Position = UDim2.new(1, -pillW - 10, 0.5, -pillH / 2)
pill.BackgroundColor3 = MakimaColors.PillBG
pill.BorderSizePixel = 0
pill.ZIndex = 3
pill.Parent = ToggleRow

local pillCorner = Instance.new("UICorner")
pillCorner.CornerRadius = UDim.new(1, 0)
pillCorner.Parent = pill

local pillStroke = Instance.new("UIStroke")
pillStroke.Color = MakimaColors.Border
pillStroke.Thickness = 1
pillStroke.Parent = pill

local knob = Instance.new("Frame")
knob.Size = UDim2.new(0, knobSz, 0, knobSz)
knob.Position = UDim2.new(0, 3, 0.5, -knobSz / 2)
knob.BackgroundColor3 = MakimaColors.Knob
knob.BorderSizePixel = 0
knob.ZIndex = 4
knob.Parent = pill

local knobCorner = Instance.new("UICorner")
knobCorner.CornerRadius = UDim.new(1, 0)
knobCorner.Parent = knob

local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Text = ""
ToggleBtn.BackgroundTransparency = 1
ToggleBtn.Size = UDim2.new(1, 0, 1, 0)
ToggleBtn.ZIndex = 5
ToggleBtn.Parent = pill

-- // ─── Infinite Jump Toggle Row ───
local InfJumpRow = Instance.new("Frame")
InfJumpRow.Size = UDim2.new(1, -24, 0, 42)
InfJumpRow.Position = UDim2.new(0, 12, 0, 66) -- Pushed down slightly so it gets completely clipped
InfJumpRow.BackgroundColor3 = MakimaColors.ContainerBG
InfJumpRow.BackgroundTransparency = 0.15
InfJumpRow.BorderSizePixel = 0
InfJumpRow.ZIndex = 2
InfJumpRow.Parent = ContentFrame

local InfJumpRowCorner = Instance.new("UICorner")
InfJumpRowCorner.CornerRadius = UDim.new(0, 12)
InfJumpRowCorner.Parent = InfJumpRow

local InfJumpRowStroke = Instance.new("UIStroke")
InfJumpRowStroke.Color = MakimaColors.Border
InfJumpRowStroke.Thickness = 1
InfJumpRowStroke.Parent = InfJumpRow

local InfJumpLabel = Instance.new("TextLabel")
InfJumpLabel.BackgroundTransparency = 1
InfJumpLabel.TextXAlignment = Enum.TextXAlignment.Left
InfJumpLabel.Font = Enum.Font.GothamBold
InfJumpLabel.TextColor3 = MakimaColors.AccentText
InfJumpLabel.Position = UDim2.new(0, 12, 0, 0)
InfJumpLabel.Text = "inf jump : OFF"
InfJumpLabel.TextSize = 13
InfJumpLabel.Size = UDim2.new(1, -80, 1, 0)
InfJumpLabel.ZIndex = 3
InfJumpLabel.Parent = InfJumpRow

local ijPill = Instance.new("Frame")
ijPill.Size = UDim2.new(0, pillW, 0, pillH)
ijPill.Position = UDim2.new(1, -pillW - 10, 0.5, -pillH / 2)
ijPill.BackgroundColor3 = MakimaColors.PillBG
ijPill.BorderSizePixel = 0
ijPill.ZIndex = 3
ijPill.Parent = InfJumpRow

local ijPillCorner = Instance.new("UICorner")
ijPillCorner.CornerRadius = UDim.new(1, 0)
ijPillCorner.Parent = ijPill

local ijPillStroke = Instance.new("UIStroke")
ijPillStroke.Color = MakimaColors.Border
ijPillStroke.Thickness = 1
ijPillStroke.Parent = ijPill

local ijKnob = Instance.new("Frame")
ijKnob.Size = UDim2.new(0, knobSz, 0, knobSz)
ijKnob.Position = UDim2.new(0, 3, 0.5, -knobSz / 2)
ijKnob.BackgroundColor3 = MakimaColors.Knob
ijKnob.BorderSizePixel = 0
ijKnob.ZIndex = 4
ijKnob.Parent = ijPill

local ijKnobCorner = Instance.new("UICorner")
ijKnobCorner.CornerRadius = UDim.new(1, 0)
ijKnobCorner.Parent = ijKnob

local InfJumpBtn = Instance.new("TextButton")
InfJumpBtn.Text = ""
InfJumpBtn.BackgroundTransparency = 1
InfJumpBtn.Size = UDim2.new(1, 0, 1, 0)
InfJumpBtn.ZIndex = 5
InfJumpBtn.Parent = ijPill

-- // PC Keybind System Config
local CurrentKey = Enum.KeyCode.G
local ListeningForKey = false

local KeybindBtn = Instance.new("TextButton")
KeybindBtn.Size = UDim2.new(1, -24, 0, 28)
KeybindBtn.Position = UDim2.new(0, 12, 0, 120) -- Pushed down
KeybindBtn.BackgroundColor3 = MakimaColors.MainBG
KeybindBtn.BackgroundTransparency = 0.15
KeybindBtn.Font = Enum.Font.GothamMedium
KeybindBtn.Text = "[ Keybind : " .. CurrentKey.Name .. " ]"
KeybindBtn.TextColor3 = MakimaColors.AccentText
KeybindBtn.TextSize = 11
KeybindBtn.BorderSizePixel = 0
KeybindBtn.ZIndex = 3
KeybindBtn.Parent = ContentFrame

local KeybindCorner = Instance.new("UICorner")
KeybindCorner.CornerRadius = UDim.new(0, 8)
KeybindCorner.Parent = KeybindBtn

local KeybindStroke = Instance.new("UIStroke")
KeybindStroke.Color = MakimaColors.Border
KeybindStroke.Thickness = 1
KeybindStroke.Parent = KeybindBtn

-- // Lock UI Feature Setup
local LockUIBtn = Instance.new("TextButton")
LockUIBtn.Size = UDim2.new(1, -24, 0, 28)
LockUIBtn.Position = UDim2.new(0, 12, 0, 156) -- Pushed down
LockUIBtn.BackgroundColor3 = MakimaColors.MainBG
LockUIBtn.BackgroundTransparency = 0.15
LockUIBtn.Font = Enum.Font.GothamMedium
LockUIBtn.Text = "[ Lock UI : OFF ]"
LockUIBtn.TextColor3 = MakimaColors.AccentText
LockUIBtn.TextSize = 11
LockUIBtn.BorderSizePixel = 0
LockUIBtn.ZIndex = 3
LockUIBtn.Parent = ContentFrame

local LockUICorner = Instance.new("UICorner")
LockUICorner.CornerRadius = UDim.new(0, 8)
LockUICorner.Parent = LockUIBtn

local LockUIStroke = Instance.new("UIStroke")
LockUIStroke.Color = MakimaColors.Border
LockUIStroke.Thickness = 1
LockUIStroke.Parent = LockUIBtn

-- // Discord Link Label
local DiscordLabel = Instance.new("TextLabel")
DiscordLabel.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
DiscordLabel.BackgroundTransparency = 0.15
DiscordLabel.Font = Enum.Font.GothamBold
DiscordLabel.TextColor3 = MakimaColors.AccentText
DiscordLabel.Position = UDim2.new(0, 12, 0, 196) -- Pushed down
DiscordLabel.Text = "discord.gg/gFKHpWMQPx"
DiscordLabel.TextSize = 12
DiscordLabel.Size = UDim2.new(1, -24, 0, 20)
DiscordLabel.ZIndex = 3
local DiscordCorner = Instance.new("UICorner")
DiscordCorner.CornerRadius = UDim.new(0, 8)
DiscordCorner.Parent = DiscordLabel
DiscordLabel.Parent = ContentFrame

-- // Update Anti Bat UI
local function updateUI(state)
    StatusLabel.Text = state and "anti bat : ON" or "anti bat : OFF"
    TweenService:Create(knob, TweenInfo.new(0.15), {
        Position = state and UDim2.new(1, -knobSz - 3, 0.5, -knobSz / 2) or UDim2.new(0, 3, 0.5, -knobSz / 2)
    }):Play()
end

-- // Update Inf Jump UI
local function updateInfJumpUI(state)
    InfJumpLabel.Text = state and "inf jump : ON" or "inf jump : OFF"
    TweenService:Create(ijKnob, TweenInfo.new(0.15), {
        Position = state and UDim2.new(1, -knobSz - 3, 0.5, -knobSz / 2) or UDim2.new(0, 3, 0.5, -knobSz / 2)
    }):Play()
end

local function setToggle(state)
    PredictionBreakerEnabled = state
    updateUI(state)
    TogglePredictionBreaker(state)
end

local function setInfJump(state)
    InfJumpEnabled = state
    updateInfJumpUI(state)
    ToggleInfJump(state)
end

ToggleBtn.MouseButton1Click:Connect(function()
    setToggle(not PredictionBreakerEnabled)
end)

InfJumpBtn.MouseButton1Click:Connect(function()
    setInfJump(not InfJumpEnabled)
end)

-- // Keybind Listeners
KeybindBtn.MouseButton1Click:Connect(function()
    ListeningForKey = true
    KeybindBtn.Text = "[ Press Any Key (Backspace to Clear) ]"
end)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end

    if ListeningForKey then
        if input.UserInputType == Enum.UserInputType.Keyboard then
            if input.KeyCode == Enum.KeyCode.Backspace or input.KeyCode == Enum.KeyCode.Escape then
                CurrentKey = nil
                KeybindBtn.Text = "[ Keybind : None ]"
            else
                CurrentKey = input.KeyCode
                KeybindBtn.Text = "[ Keybind : " .. CurrentKey.Name .. " ]"
            end
            ListeningForKey = false
        end
    else
        if CurrentKey and input.KeyCode == CurrentKey then
            setToggle(not PredictionBreakerEnabled)
        end
    end
end)

-- // Lock UI Toggle Logic
LockUIBtn.MouseButton1Click:Connect(function()
    UILocked = not UILocked
    LockUIBtn.Text = UILocked and "[ Lock UI : ON ]" or "[ Lock UI : OFF ]"
    LockUIBtn.BackgroundColor3 = UILocked and Color3.fromRGB(255, 205, 210) or MakimaColors.MainBG
end)

-- // Smooth Dragging Handler
local guiDragging = false
local dragStart = nil
local startPos = nil

local function clampPosition(pos)
    local screenSize = workspace.CurrentCamera.ViewportSize
    local guiSize = Frame.AbsoluteSize
    local maxX = screenSize.X - guiSize.X
    local maxY = screenSize.Y - guiSize.Y
    local x = math.clamp(pos.X.Offset, 0, math.max(0, maxX))
    local y = math.clamp(pos.Y.Offset, 0, math.max(0, maxY))
    return UDim2.new(0, x, 0, y)
end

TitleBar.InputBegan:Connect(function(input)
    if UILocked then return end
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        guiDragging = true
        dragStart = input.Position
        startPos = Frame.Position
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if guiDragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        local newPos = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        Frame.Position = clampPosition(newPos)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        guiDragging = false
    end
end)

-- // Initialize
updateUI(false)
updateInfJumpUI(false)
print("[Makima] Settings configured safely!")
