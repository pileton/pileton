-- Enhanced Clicker UI v4 for Roblox Executors
-- Compatible with Delta, KRNL, Fluxus, etc.

-- Services
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local HttpService = game:GetService("HttpService")
local RunService = game:GetService("RunService")

-- Variables
local LocalPlayer = Players.LocalPlayer
local PlayerName = LocalPlayer.Name
local ClickCount = 0
local CurrentTheme = "Tokyo Purple"
local HasAutoClicker = false
local ClickMultiplier = 1
local AutoClickerInterval = nil
local IsMinimized = false
local PurchasedThemes = {["Tokyo Purple"] = true, ["Green Hill"] = true, ["Blue Ocean"] = true}
local NewThemes = {}
local LastAutoClickTime = 0
local IsUsernameHidden = false
local HasAutoUpgrade = false
local AutoUpgradeInterval = nil

-- UI Creation Function
local function CreateClickerUI()
    -- Remove existing UI if it exists
    if game:GetService("CoreGui"):FindFirstChild("ClickerUI") then
        game:GetService("CoreGui"):FindFirstChild("ClickerUI"):Destroy()
        
        -- Clear intervals if they exist
        if AutoClickerInterval then
            AutoClickerInterval:Disconnect()
            AutoClickerInterval = nil
        end
        
        if AutoUpgradeInterval then
            AutoUpgradeInterval:Disconnect()
            AutoUpgradeInterval = nil
        end
    end
    
    -- Create ScreenGui
    local ClickerUI = Instance.new("ScreenGui")
    ClickerUI.Name = "ClickerUI"
    ClickerUI.Parent = game:GetService("CoreGui")
    ClickerUI.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    
    -- Main Frame
    local MainFrame = Instance.new("Frame")
    MainFrame.Name = "MainFrame"
    MainFrame.Size = UDim2.new(0, 500, 0, 300)
    MainFrame.Position = UDim2.new(0.5, -250, 0.5, -150)
    MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    MainFrame.BorderSizePixel = 0
    MainFrame.Parent = ClickerUI
    
    -- Apply rounded corners
    local UICorner = Instance.new("UICorner")
    UICorner.CornerRadius = UDim.new(0, 12)
    UICorner.Parent = MainFrame
    
    -- Create gradient based on theme
    local UIGradient = Instance.new("UIGradient")
    UIGradient.Rotation = 45 -- Diagonal gradient for more modern look
    UIGradient.Parent = MainFrame
    
    -- Add glowing outline to main frame
    local MainOutline = Instance.new("UIStroke")
    MainOutline.Thickness = 3
    MainOutline.Transparency = 0
    MainOutline.Parent = MainFrame
    
    -- Apply theme
    ApplyTheme(UIGradient, MainOutline, CurrentTheme)
    
    -- Create a container for the welcome section
    local WelcomeContainer = Instance.new("Frame")
    WelcomeContainer.Name = "WelcomeContainer"
    WelcomeContainer.Size = UDim2.new(0, 300, 0, 50)
    WelcomeContainer.Position = UDim2.new(0, 20, 0, 10)
    WelcomeContainer.BackgroundTransparency = 1
    WelcomeContainer.Parent = MainFrame
    
    -- User Profile Picture
    local ProfilePicture = Instance.new("ImageLabel")
    ProfilePicture.Name = "ProfilePicture"
    ProfilePicture.Size = UDim2.new(0, 40, 0, 40)
    ProfilePicture.Position = UDim2.new(0, 0, 0, 0)
    ProfilePicture.BackgroundTransparency = 1
    
    -- Try to get user's profile picture
    local userId = LocalPlayer.UserId
    local success, result = pcall(function()
        return game:GetService("Players"):GetUserThumbnailAsync(userId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size420x420)
    end)
    
    if success then
        ProfilePicture.Image = result
    else
        -- Fallback image if profile picture can't be loaded
        ProfilePicture.Image = "rbxassetid://7962146544" -- Default Roblox head
    end
    
    -- Add rounded corners to profile picture
    local ProfileCorner = Instance.new("UICorner")
    ProfileCorner.CornerRadius = UDim.new(1, 0) -- Fully round
    ProfileCorner.Parent = ProfilePicture
    
    ProfilePicture.Parent = WelcomeContainer
    
    -- Welcome Text
    local WelcomeLabel = Instance.new("TextLabel")
    WelcomeLabel.Name = "WelcomeLabel"
    WelcomeLabel.Size = UDim2.new(0, 250, 0, 40)
    WelcomeLabel.Position = UDim2.new(0, 50, 0, 0)
    WelcomeLabel.BackgroundTransparency = 1
    WelcomeLabel.Font = Enum.Font.GothamBold
    WelcomeLabel.Text = "Welcome back, " .. (IsUsernameHidden and "User" or PlayerName) .. "!"
    WelcomeLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    WelcomeLabel.TextSize = 20
    WelcomeLabel.TextXAlignment = Enum.TextXAlignment.Left
    WelcomeLabel.Parent = WelcomeContainer
    
    -- Add glow effect to welcome text
    local TextGlow = Instance.new("UIStroke")
    TextGlow.Thickness = 1.5
    TextGlow.Color = Color3.fromRGB(180, 120, 255)
    TextGlow.Transparency = 0.3
    TextGlow.Parent = WelcomeLabel
    
    -- Minimize Button (-)
    local MinimizeButton = Instance.new("TextButton")
    MinimizeButton.Name = "MinimizeButton"
    MinimizeButton.Size = UDim2.new(0, 30, 0, 30)
    MinimizeButton.Position = UDim2.new(1, -80, 0, 10)
    MinimizeButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    MinimizeButton.Text = "-"
    MinimizeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    MinimizeButton.Font = Enum.Font.GothamBold
    MinimizeButton.TextSize = 20
    MinimizeButton.Parent = MainFrame
    
    -- Add glow effect to minimize button
    local MinimizeGlow = Instance.new("UIStroke")
    MinimizeGlow.Color = Color3.fromRGB(150, 150, 150)
    MinimizeGlow.Thickness = 2
    MinimizeGlow.Parent = MinimizeButton
    
    -- Add rounded corners to minimize button
    local MinimizeCorner = Instance.new("UICorner")
    MinimizeCorner.CornerRadius = UDim.new(0, 8)
    MinimizeCorner.Parent = MinimizeButton
    
    -- Close Button (X)
    local CloseButton = Instance.new("TextButton")
    CloseButton.Name = "CloseButton"
    CloseButton.Size = UDim2.new(0, 30, 0, 30)
    CloseButton.Position = UDim2.new(1, -40, 0, 10)
    CloseButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
    CloseButton.Text = "X"
    CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    CloseButton.Font = Enum.Font.GothamBold
    CloseButton.TextSize = 16
    CloseButton.Parent = MainFrame
    
    -- Add glow effect to close button
    local CloseGlow = Instance.new("UIStroke")
    CloseGlow.Color = Color3.fromRGB(255, 100, 100)
    CloseGlow.Thickness = 2
    CloseGlow.Parent = CloseButton
    
    -- Add rounded corners to close button
    local CloseCorner = Instance.new("UICorner")
    CloseCorner.CornerRadius = UDim.new(0, 8)
    CloseCorner.Parent = CloseButton
    
    -- Content Container (will be hidden when minimized)
    local ContentContainer = Instance.new("Frame")
    ContentContainer.Name = "ContentContainer"
    ContentContainer.Size = UDim2.new(1, 0, 1, -50)
    ContentContainer.Position = UDim2.new(0, 0, 0, 50)
    ContentContainer.BackgroundTransparency = 1
    ContentContainer.Parent = MainFrame
    
    -- Tab Buttons Container
    local TabsFrame = Instance.new("Frame")
    TabsFrame.Name = "TabsFrame"
    TabsFrame.Size = UDim2.new(0, 120, 0, 250)
    TabsFrame.Position = UDim2.new(1, -130, 0, 0)
    TabsFrame.BackgroundTransparency = 1
    TabsFrame.Parent = ContentContainer
    
    -- Create Tab Buttons
    local TabButtons = {}
    local TabNames = {"Clicker", "Shop", "Settings", "New!"}
    local TabIcons = {
        Clicker = "rbxassetid://7059346373", -- Mouse cursor icon
        Shop = "rbxassetid://7059225553", -- Shopping cart icon
        Settings = "rbxassetid://7059291516", -- Gear icon
        ["New!"] = "rbxassetid://7059340326" -- Star icon
    }
    
    for i, tabName in ipairs(TabNames) do
        local TabButton = Instance.new("TextButton")
        TabButton.Name = tabName .. "Tab"
        TabButton.Size = UDim2.new(0, 100, 0, 40)
        TabButton.Position = UDim2.new(0, 10, 0, (i-1) * 50)
        TabButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        TabButton.Text = ""
        TabButton.TextColor3 = Color3.fromRGB(255, 255, 255)
        TabButton.Font = Enum.Font.GothamSemibold
        TabButton.TextSize = 14
        TabButton.Parent = TabsFrame
        
        -- Add rounded corners to tab buttons
        local TabCorner = Instance.new("UICorner")
        TabCorner.CornerRadius = UDim.new(0, 8)
        TabCorner.Parent = TabButton
        
        -- Add outline to tab buttons
        local TabOutline = Instance.new("UIStroke")
        TabOutline.Thickness = 2
        TabOutline.Color = Color3.fromRGB(120, 120, 120)
        TabOutline.Transparency = 0.3
        TabOutline.Parent = TabButton
        
        -- Add icon to tab button
        local TabIcon = Instance.new("ImageLabel")
        TabIcon.Name = "Icon"
        TabIcon.Size = UDim2.new(0, 20, 0, 20)
        TabIcon.Position = UDim2.new(0, 10, 0.5, -10)
        TabIcon.BackgroundTransparency = 1
        TabIcon.Image = TabIcons[tabName]
        TabIcon.Parent = TabButton
        
        -- Add tab name
        local TabLabel = Instance.new("TextLabel")
        TabLabel.Name = "Label"
        TabLabel.Size = UDim2.new(0, 60, 0, 20)
        TabLabel.Position = UDim2.new(0, 35, 0.5, -10)
        TabLabel.BackgroundTransparency = 1
        TabLabel.Text = tabName
        TabLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        TabLabel.Font = Enum.Font.GothamSemibold
        TabLabel.TextSize = 14
        TabLabel.TextXAlignment = Enum.TextXAlignment.Left
        TabLabel.Parent = TabButton
        
        -- Add NEW! indicator to the New! tab
        if tabName == "New!" then
            local NewIndicator = Instance.new("TextLabel")
            NewIndicator.Name = "NewIndicator"
            NewIndicator.Size = UDim2.new(0, 30, 0, 16)
            NewIndicator.Position = UDim2.new(1, -30, 0, 0)
            NewIndicator.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
            NewIndicator.Text = "NEW!"
            NewIndicator.TextColor3 = Color3.fromRGB(255, 255, 255)
            NewIndicator.Font = Enum.Font.GothamBold
            NewIndicator.TextSize = 8
            NewIndicator.Parent = TabButton
            
            -- Add rounded corners to NEW! indicator
            local NewIndicatorCorner = Instance.new("UICorner")
            NewIndicatorCorner.CornerRadius = UDim.new(0, 4)
            NewIndicatorCorner.Parent = NewIndicator
        end
        
        table.insert(TabButtons, TabButton)
    end
    
    -- Content Frame (where tab content will be displayed)
    local ContentFrame = Instance.new("Frame")
    ContentFrame.Name = "ContentFrame"
    ContentFrame.Size = UDim2.new(0, 350, 0, 220)
    ContentFrame.Position = UDim2.new(0, 20, 0, 10)
    ContentFrame.BackgroundTransparency = 1
    ContentFrame.Parent = ContentContainer
    
    -- Create Content for Clicker Tab
    local ClickerContent = Instance.new("Frame")
    ClickerContent.Name = "ClickerContent"
    ClickerContent.Size = UDim2.new(1, 0, 1, 0)
    ClickerContent.BackgroundTransparency = 1
    ClickerContent.Visible = true
    ClickerContent.Parent = ContentFrame
    
    -- Clicks Counter
    local ClicksLabel = Instance.new("TextLabel")
    ClicksLabel.Name = "ClicksLabel"
    ClicksLabel.Size = UDim2.new(0, 200, 0, 30)
    ClicksLabel.Position = UDim2.new(0.5, -100, 0, 20)
    ClicksLabel.BackgroundTransparency = 1
    ClicksLabel.Font = Enum.Font.GothamBold
    ClicksLabel.Text = "Clicks: 0"
    ClicksLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    ClicksLabel.TextSize = 18
    ClicksLabel.Parent = ClickerContent
    
    -- Add glow to clicks label
    local ClicksGlow = Instance.new("UIStroke")
    ClicksGlow.Thickness = 1.5
    ClicksGlow.Color = Color3.fromRGB(180, 120, 255)
    ClicksGlow.Transparency = 0.3
    ClicksGlow.Parent = ClicksLabel
    
    -- Clicker Button (now rectangular)
    local ClickerButton = Instance.new("TextButton")
    ClickerButton.Name = "ClickerButton"
    ClickerButton.Size = UDim2.new(0, 150, 0, 80)
    ClickerButton.Position = UDim2.new(0.5, -75, 0.5, -30)
    ClickerButton.BackgroundColor3 = Color3.fromRGB(180, 120, 255)
    ClickerButton.Text = "CLICK"
    ClickerButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    ClickerButton.Font = Enum.Font.GothamBold
    ClickerButton.TextSize = 20
    ClickerButton.Parent = ClickerContent
    
    -- Make clicker button rounded (but not fully round)
    local ClickerCorner = Instance.new("UICorner")
    ClickerCorner.CornerRadius = UDim.new(0, 10)
    ClickerCorner.Parent = ClickerButton
    
    -- Add glow to clicker button
    local ClickerGlow = Instance.new("UIStroke")
    ClickerGlow.Thickness = 3
    ClickerGlow.Color = Color3.fromRGB(200, 150, 255)
    ClickerGlow.Transparency = 0
    ClickerGlow.Parent = ClickerButton
    
    -- Create Content for Shop Tab with ScrollingFrame
    local ShopContent = Instance.new("Frame")
    ShopContent.Name = "ShopContent"
    ShopContent.Size = UDim2.new(1, 0, 1, 0)
    ShopContent.BackgroundTransparency = 1
    ShopContent.Visible = false
    ShopContent.Parent = ContentFrame
    
    -- Shop ScrollingFrame
    local ShopScrollFrame = Instance.new("ScrollingFrame")
    ShopScrollFrame.Name = "ShopScrollFrame"
    ShopScrollFrame.Size = UDim2.new(1, 0, 1, 0)
    ShopScrollFrame.BackgroundTransparency = 1
    ShopScrollFrame.BorderSizePixel = 0
    ShopScrollFrame.ScrollBarThickness = 6
    ShopScrollFrame.ScrollBarImageColor3 = Color3.fromRGB(180, 120, 255)
    ShopScrollFrame.CanvasSize = UDim2.new(0, 0, 0, 550) -- Adjusted for new content
    ShopScrollFrame.Parent = ShopContent
    
    -- Clicker Upgrades Category
    local ClickerUpgradesLabel = Instance.new("TextLabel")
    ClickerUpgradesLabel.Name = "ClickerUpgradesLabel"
    ClickerUpgradesLabel.Size = UDim2.new(0, 300, 0, 30)
    ClickerUpgradesLabel.Position = UDim2.new(0.5, -150, 0, 10)
    ClickerUpgradesLabel.BackgroundTransparency = 1
    ClickerUpgradesLabel.Font = Enum.Font.GothamBold
    ClickerUpgradesLabel.Text = "Clicker Upgrades"
    ClickerUpgradesLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    ClickerUpgradesLabel.TextSize = 22
    ClickerUpgradesLabel.Parent = ShopScrollFrame
    
    -- Add glow to category label
    local CategoryGlow = Instance.new("UIStroke")
    CategoryGlow.Thickness = 1.5
    CategoryGlow.Color = Color3.fromRGB(180, 120, 255)
    CategoryGlow.Transparency = 0.3
    CategoryGlow.Parent = ClickerUpgradesLabel
    
    -- Auto Clicker Button
    local AutoClickerButton = Instance.new("TextButton")
    AutoClickerButton.Name = "AutoClickerButton"
    AutoClickerButton.Size = UDim2.new(0, 300, 0, 60)
    AutoClickerButton.Position = UDim2.new(0.5, -150, 0, 50)
    AutoClickerButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    AutoClickerButton.Text = ""
    AutoClickerButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    AutoClickerButton.Font = Enum.Font.GothamSemibold
    AutoClickerButton.TextSize = 16
    AutoClickerButton.Parent = ShopScrollFrame
    
    -- Add rounded corners to auto clicker button
    local AutoClickerCorner = Instance.new("UICorner")
    AutoClickerCorner.CornerRadius = UDim.new(0, 8)
    AutoClickerCorner.Parent = AutoClickerButton
    
    -- Add outline to auto clicker button
    local AutoClickerOutline = Instance.new("UIStroke")
    AutoClickerOutline.Thickness = 2
    AutoClickerOutline.Color = Color3.fromRGB(120, 120, 120)
    AutoClickerOutline.Transparency = 0.3
    AutoClickerOutline.Parent = AutoClickerButton
    
    -- Auto Clicker Title
    local AutoClickerTitle = Instance.new("TextLabel")
    AutoClickerTitle.Name = "Title"
    AutoClickerTitle.Size = UDim2.new(0, 200, 0, 25)
    AutoClickerTitle.Position = UDim2.new(0, 15, 0, 10)
    AutoClickerTitle.BackgroundTransparency = 1
    AutoClickerTitle.Font = Enum.Font.GothamBold
    AutoClickerTitle.Text = "Auto Clicker"
    AutoClickerTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    AutoClickerTitle.TextSize = 16
    AutoClickerTitle.TextXAlignment = Enum.TextXAlignment.Left
    AutoClickerTitle.Parent = AutoClickerButton
    
    -- Auto Clicker Description
    local AutoClickerDesc = Instance.new("TextLabel")
    AutoClickerDesc.Name = "Description"
    AutoClickerDesc.Size = UDim2.new(0, 200, 0, 20)
    AutoClickerDesc.Position = UDim2.new(0, 15, 0, 35)
    AutoClickerDesc.BackgroundTransparency = 1
    AutoClickerDesc.Font = Enum.Font.Gotham
    AutoClickerDesc.Text = "Clicks automatically every second"
    AutoClickerDesc.TextColor3 = Color3.fromRGB(200, 200, 200)
    AutoClickerDesc.TextSize = 12
    AutoClickerDesc.TextXAlignment = Enum.TextXAlignment.Left
    AutoClickerDesc.Parent = AutoClickerButton
    
    -- Auto Clicker Price
    local AutoClickerPrice = Instance.new("TextLabel")
    AutoClickerPrice.Name = "Price"
    AutoClickerPrice.Size = UDim2.new(0, 80, 0, 25)
    AutoClickerPrice.Position = UDim2.new(1, -90, 0.5, -12)
    AutoClickerPrice.BackgroundTransparency = 1
    AutoClickerPrice.Font = Enum.Font.GothamBold
    AutoClickerPrice.Text = "250 clicks"
    AutoClickerPrice.TextColor3 = Color3.fromRGB(255, 220, 100)
    AutoClickerPrice.TextSize = 14
    AutoClickerPrice.Parent = AutoClickerButton
    
    -- Click Multiplier Button
    local MultiplierButton = Instance.new("TextButton")
    MultiplierButton.Name = "MultiplierButton"
    MultiplierButton.Size = UDim2.new(0, 300, 0, 60)
    MultiplierButton.Position = UDim2.new(0.5, -150, 0, 120)
    MultiplierButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    MultiplierButton.Text = ""
    MultiplierButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    MultiplierButton.Font = Enum.Font.GothamSemibold
    MultiplierButton.TextSize = 16
    MultiplierButton.Parent = ShopScrollFrame
    
    -- Add rounded corners to multiplier button
    local MultiplierCorner = Instance.new("UICorner")
    MultiplierCorner.CornerRadius = UDim.new(0, 8)
    MultiplierCorner.Parent = MultiplierButton
    
    -- Add outline to multiplier button
    local MultiplierOutline = Instance.new("UIStroke")
    MultiplierOutline.Thickness = 2
    MultiplierOutline.Color = Color3.fromRGB(120, 120, 120)
    MultiplierOutline.Transparency = 0.3
    MultiplierOutline.Parent = MultiplierButton
    
    -- Multiplier Title
    local MultiplierTitle = Instance.new("TextLabel")
    MultiplierTitle.Name = "Title"
    MultiplierTitle.Size = UDim2.new(0, 200, 0, 25)
    MultiplierTitle.Position = UDim2.new(0, 15, 0, 10)
    MultiplierTitle.BackgroundTransparency = 1
    MultiplierTitle.Font = Enum.Font.GothamBold
    MultiplierTitle.Text = "+1 Click Multiplier"
    MultiplierTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    MultiplierTitle.TextSize = 16
    MultiplierTitle.TextXAlignment = Enum.TextXAlignment.Left
    MultiplierTitle.Parent = MultiplierButton
    
    -- Multiplier Description
    local MultiplierDesc = Instance.new("TextLabel")
    MultiplierDesc.Name = "Description"
    MultiplierDesc.Size = UDim2.new(0, 200, 0, 20)
    MultiplierDesc.Position = UDim2.new(0, 15, 0, 35)
    MultiplierDesc.BackgroundTransparency = 1
    MultiplierDesc.Font = Enum.Font.Gotham
    MultiplierDesc.Text = "Get +1 click per click (Current: x" .. ClickMultiplier .. ")"
    MultiplierDesc.TextColor3 = Color3.fromRGB(200, 200, 200)
    MultiplierDesc.TextSize = 12
    MultiplierDesc.TextXAlignment = Enum.TextXAlignment.Left
    MultiplierDesc.Parent = MultiplierButton
    
    -- Multiplier Price
    local MultiplierPrice = Instance.new("TextLabel")
    MultiplierPrice.Name = "Price"
    MultiplierPrice.Size = UDim2.new(0, 80, 0, 25)
    MultiplierPrice.Position = UDim2.new(1, -90, 0.5, -12)
    MultiplierPrice.BackgroundTransparency = 1
    MultiplierPrice.Font = Enum.Font.GothamBold
    MultiplierPrice.Text = "100 clicks"
    MultiplierPrice.TextColor3 = Color3.fromRGB(255, 220, 100)
    MultiplierPrice.TextSize = 14
    MultiplierPrice.Parent = MultiplierButton
    
    -- Auto Upgrade Category
    local AutoUpgradeLabel = Instance.new("TextLabel")
    AutoUpgradeLabel.Name = "AutoUpgradeLabel"
    AutoUpgradeLabel.Size = UDim2.new(0, 300, 0, 30)
    AutoUpgradeLabel.Position = UDim2.new(0.5, -150, 0, 190)
    AutoUpgradeLabel.BackgroundTransparency = 1
    AutoUpgradeLabel.Font = Enum.Font.GothamBold
    AutoUpgradeLabel.Text = "Auto Upgrade"
    AutoUpgradeLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    AutoUpgradeLabel.TextSize = 22
    AutoUpgradeLabel.Parent = ShopScrollFrame
    
    -- Add glow to auto upgrade label
    local AutoUpgradeGlow = Instance.new("UIStroke")
    AutoUpgradeGlow.Thickness = 1.5
    AutoUpgradeGlow.Color = Color3.fromRGB(180, 120, 255)
    AutoUpgradeGlow.Transparency = 0.3
    AutoUpgradeGlow.Parent = AutoUpgradeLabel
    
    -- Auto Upgrade Button
    local AutoUpgradeButton = Instance.new("TextButton")
    AutoUpgradeButton.Name = "AutoUpgradeButton"
    AutoUpgradeButton.Size = UDim2.new(0, 300, 0, 60)
    AutoUpgradeButton.Position = UDim2.new(0.5, -150, 0, 230)
    AutoUpgradeButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    AutoUpgradeButton.Text = ""
    AutoUpgradeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    AutoUpgradeButton.Font = Enum.Font.GothamSemibold
    AutoUpgradeButton.TextSize = 16
    AutoUpgradeButton.Parent = ShopScrollFrame
    
    -- Add NEW! indicator to Auto Upgrade Button
    local AutoUpgradeNew = Instance.new("TextLabel")
    AutoUpgradeNew.Name = "NewIndicator"
    AutoUpgradeNew.Size = UDim2.new(0, 40, 0, 20)
    AutoUpgradeNew.Position = UDim2.new(0, 10, 0, -10)
    AutoUpgradeNew.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
    AutoUpgradeNew.Text = "NEW!"
    AutoUpgradeNew.TextColor3 = Color3.fromRGB(255, 255, 255)
    AutoUpgradeNew.Font = Enum.Font.GothamBold
    AutoUpgradeNew.TextSize = 10
    AutoUpgradeNew.Parent = AutoUpgradeButton
    
    -- Add rounded corners to NEW! indicator
    local AutoUpgradeNewCorner = Instance.new("UICorner")
    AutoUpgradeNewCorner.CornerRadius = UDim.new(0, 4)
    AutoUpgradeNewCorner.Parent = AutoUpgradeNew
    
    -- Add rounded corners to auto upgrade button
    local AutoUpgradeCorner = Instance.new("UICorner")
    AutoUpgradeCorner.CornerRadius = UDim.new(0, 8)
    AutoUpgradeCorner.Parent = AutoUpgradeButton
    
    -- Add outline to auto upgrade button
    local AutoUpgradeOutline = Instance.new("UIStroke")
    AutoUpgradeOutline.Thickness = 2
    AutoUpgradeOutline.Color = Color3.fromRGB(120, 120, 120)
    AutoUpgradeOutline.Transparency = 0.3
    AutoUpgradeOutline.Parent = AutoUpgradeButton
    
    -- Auto Upgrade Title
    local AutoUpgradeTitle = Instance.new("TextLabel")
    AutoUpgradeTitle.Name = "Title"
    AutoUpgradeTitle.Size = UDim2.new(0, 200, 0, 25)
    AutoUpgradeTitle.Position = UDim2.new(0, 15, 0, 10)
    AutoUpgradeTitle.BackgroundTransparency = 1
    AutoUpgradeTitle.Font = Enum.Font.GothamBold
    AutoUpgradeTitle.Text = "Auto Upgrade 1+ Clicker"
    AutoUpgradeTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    AutoUpgradeTitle.TextSize = 16
    AutoUpgradeTitle.TextXAlignment = Enum.TextXAlignment.Left
    AutoUpgradeTitle.Parent = AutoUpgradeButton
    
    -- Auto Upgrade Description
    local AutoUpgradeDesc = Instance.new("TextLabel")
    AutoUpgradeDesc.Name = "Description"
    AutoUpgradeDesc.Size = UDim2.new(0, 200, 0, 20)
    AutoUpgradeDesc.Position = UDim2.new(0, 15, 0, 35)
    AutoUpgradeDesc.BackgroundTransparency = 1
    AutoUpgradeDesc.Font = Enum.Font.Gotham
    AutoUpgradeDesc.Text = "Automatically buys +1 multiplier upgrades"
    AutoUpgradeDesc.TextColor3 = Color3.fromRGB(200, 200, 200)
    AutoUpgradeDesc.TextSize = 12
    AutoUpgradeDesc.TextXAlignment = Enum.TextXAlignment.Left
    AutoUpgradeDesc.Parent = AutoUpgradeButton
    
    -- Auto Upgrade Price
    local AutoUpgradePrice = Instance.new("TextLabel")
    AutoUpgradePrice.Name = "Price"
    AutoUpgradePrice.Size = UDim2.new(0, 80, 0, 25)
    AutoUpgradePrice.Position = UDim2.new(1, -90, 0.5, -12)
    AutoUpgradePrice.BackgroundTransparency = 1
    AutoUpgradePrice.Font = Enum.Font.GothamBold
    AutoUpgradePrice.Text = "500 clicks"
    AutoUpgradePrice.TextColor3 = Color3.fromRGB(255, 220, 100)
    AutoUpgradePrice.TextSize = 14
    AutoUpgradePrice.Parent = AutoUpgradeButton
    
    -- Themes Category
    local ThemesLabel = Instance.new("TextLabel")
    ThemesLabel.Name = "ThemesLabel"
    ThemesLabel.Size = UDim2.new(0, 300, 0, 30)
    ThemesLabel.Position = UDim2.new(0.5, -150, 0, 300)
    ThemesLabel.BackgroundTransparency = 1
    ThemesLabel.Font = Enum.Font.GothamBold
    ThemesLabel.Text = "Themes"
    ThemesLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    ThemesLabel.TextSize = 22
    ThemesLabel.Parent = ShopScrollFrame
    
    -- Add glow to themes label
    local ThemesGlow = Instance.new("UIStroke")
    ThemesGlow.Thickness = 1.5
    ThemesGlow.Color = Color3.fromRGB(180, 120, 255)
    ThemesGlow.Transparency = 0.3
    ThemesGlow.Parent = ThemesLabel
    
    -- Yellow Lemon Theme Button
    local YellowLemonButton = Instance.new("TextButton")
    YellowLemonButton.Name = "YellowLemonButton"
    YellowLemonButton.Size = UDim2.new(0, 300, 0, 60)
    YellowLemonButton.Position = UDim2.new(0.5, -150, 0, 340)
    YellowLemonButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    YellowLemonButton.Text = ""
    YellowLemonButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    YellowLemonButton.Font = Enum.Font.GothamSemibold
    YellowLemonButton.TextSize = 16
    YellowLemonButton.Parent = ShopScrollFrame
    
    -- Add rounded corners to Yellow Lemon button
    local YellowLemonCorner = Instance.new("UICorner")
    YellowLemonCorner.CornerRadius = UDim.new(0, 8)
    YellowLemonCorner.Parent = YellowLemonButton
    
    -- Add outline to Yellow Lemon button
    local YellowLemonOutline = Instance.new("UIStroke")
    YellowLemonOutline.Thickness = 2
    YellowLemonOutline.Color = Color3.fromRGB(120, 120, 120)
    YellowLemonOutline.Transparency = 0.3
    YellowLemonOutline.Parent = YellowLemonButton
    
    -- Yellow Lemon Title
    local YellowLemonTitle = Instance.new("TextLabel")
    YellowLemonTitle.Name = "Title"
    YellowLemonTitle.Size = UDim2.new(0, 200, 0, 25)
    YellowLemonTitle.Position = UDim2.new(0, 15, 0, 10)
    YellowLemonTitle.BackgroundTransparency = 1
    YellowLemonTitle.Font = Enum.Font.GothamBold
    YellowLemonTitle.Text = "Yellow Lemon"
    YellowLemonTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    YellowLemonTitle.TextSize = 16
    YellowLemonTitle.TextXAlignment = Enum.TextXAlignment.Left
    YellowLemonTitle.Parent = YellowLemonButton
    
    -- Yellow Lemon Description
    local YellowLemonDesc = Instance.new("TextLabel")
    YellowLemonDesc.Name = "Description"
    YellowLemonDesc.Size = UDim2.new(0, 200, 0, 20)
    YellowLemonDesc.Position = UDim2.new(0, 15, 0, 35)
    YellowLemonDesc.BackgroundTransparency = 1
    YellowLemonDesc.Font = Enum.Font.Gotham
    YellowLemonDesc.Text = "Bright yellow and green theme"
    YellowLemonDesc.TextColor3 = Color3.fromRGB(200, 200, 200)
    YellowLemonDesc.TextSize = 12
    YellowLemonDesc.TextXAlignment = Enum.TextXAlignment.Left
    YellowLemonDesc.Parent = YellowLemonButton
    
    -- Yellow Lemon Price
    local YellowLemonPrice = Instance.new("TextLabel")
    YellowLemonPrice.Name = "Price"
    YellowLemonPrice.Size = UDim2.new(0, 80, 0, 25)
    YellowLemonPrice.Position = UDim2.new(1, -90, 0.5, -12)
    YellowLemonPrice.BackgroundTransparency = 1
    YellowLemonPrice.Font = Enum.Font.GothamBold
    YellowLemonPrice.Text = "500 clicks"
    YellowLemonPrice.TextColor3 = Color3.fromRGB(255, 220, 100)
    YellowLemonPrice.TextSize = 14
    YellowLemonPrice.Parent = YellowLemonButton
    
    -- Blood Red Theme Button
    local BloodRedButton = Instance.new("TextButton")
    BloodRedButton.Name = "BloodRedButton"
    BloodRedButton.Size = UDim2.new(0, 300, 0, 60)
    BloodRedButton.Position = UDim2.new(0.5, -150, 0, 410)
    BloodRedButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    BloodRedButton.Text = ""
    BloodRedButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    BloodRedButton.Font = Enum.Font.GothamSemibold
    BloodRedButton.TextSize = 16
    BloodRedButton.Parent = ShopScrollFrame
    
    -- Add rounded corners to Blood Red button
    local BloodRedCorner = Instance.new("UICorner")
    BloodRedCorner.CornerRadius = UDim.new(0, 8)
    BloodRedCorner.Parent = BloodRedButton
    
    -- Add outline to Blood Red button
    local BloodRedOutline = Instance.new("UIStroke")
    BloodRedOutline.Thickness = 2
    BloodRedOutline.Color = Color3.fromRGB(120, 120, 120)
    BloodRedOutline.Transparency = 0.3
    BloodRedOutline.Parent = BloodRedButton
    
    -- Blood Red Title
    local BloodRedTitle = Instance.new("TextLabel")
    BloodRedTitle.Name = "Title"
    BloodRedTitle.Size = UDim2.new(0, 200, 0, 25)
    BloodRedTitle.Position = UDim2.new(0, 15, 0, 10)
    BloodRedTitle.BackgroundTransparency = 1
    BloodRedTitle.Font = Enum.Font.GothamBold
    BloodRedTitle.Text = "Blood Red"
    BloodRedTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    BloodRedTitle.TextSize = 16
    BloodRedTitle.TextXAlignment = Enum.TextXAlignment.Left
    BloodRedTitle.Parent = BloodRedButton
    
    -- Blood Red Description
    local BloodRedDesc = Instance.new("TextLabel")
    BloodRedDesc.Name = "Description"
    BloodRedDesc.Size = UDim2.new(0, 200, 0, 20)
    BloodRedDesc.Position = UDim2.new(0, 15, 0, 35)
    BloodRedDesc.BackgroundTransparency = 1
    BloodRedDesc.Font = Enum.Font.Gotham
    BloodRedDesc.Text = "Dark and bright red gradient"
    BloodRedDesc.TextColor3 = Color3.fromRGB(200, 200, 200)
    BloodRedDesc.TextSize = 12
    BloodRedDesc.TextXAlignment = Enum.TextXAlignment.Left
    BloodRedDesc.Parent = BloodRedButton
    
    -- Blood Red Price
    local BloodRedPrice = Instance.new("TextLabel")
    BloodRedPrice.Name = "Price"
    BloodRedPrice.Size = UDim2.new(0, 80, 0, 25)
    BloodRedPrice.Position = UDim2.new(1, -90, 0.5, -12)
    BloodRedPrice.BackgroundTransparency = 1
    BloodRedPrice.Font = Enum.Font.GothamBold
    BloodRedPrice.Text = "750 clicks"
    BloodRedPrice.TextColor3 = Color3.fromRGB(255, 220, 100)
    BloodRedPrice.TextSize = 14
    BloodRedPrice.Parent = BloodRedButton
    
    -- Flashbang Theme Button
    local FlashbangButton = Instance.new("TextButton")
    FlashbangButton.Name = "FlashbangButton"
    FlashbangButton.Size = UDim2.new(0, 300, 0, 60)
    FlashbangButton.Position = UDim2.new(0.5, -150, 0, 480)
    FlashbangButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    FlashbangButton.Text = ""
    FlashbangButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    FlashbangButton.Font = Enum.Font.GothamSemibold
    FlashbangButton.TextSize = 16
    FlashbangButton.Parent = ShopScrollFrame
    
    -- Add rounded corners to Flashbang button
    local FlashbangCorner = Instance.new("UICorner")
    FlashbangCorner.CornerRadius = UDim.new(0, 8)
    FlashbangCorner.Parent = FlashbangButton
    
    -- Add outline to Flashbang button
    local FlashbangOutline = Instance.new("UIStroke")
    FlashbangOutline.Thickness = 2
    FlashbangOutline.Color = Color3.fromRGB(120, 120, 120)
    FlashbangOutline.Transparency = 0.3
    FlashbangOutline.Parent = FlashbangButton
    
    -- Flashbang Title
    local FlashbangTitle = Instance.new("TextLabel")
    FlashbangTitle.Name = "Title"
    FlashbangTitle.Size = UDim2.new(0, 200, 0, 25)
    FlashbangTitle.Position = UDim2.new(0, 15, 0, 10)
    FlashbangTitle.BackgroundTransparency = 1
    FlashbangTitle.Font = Enum.Font.GothamBold
    FlashbangTitle.Text = "Flashbang"
    FlashbangTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    FlashbangTitle.TextSize = 16
    FlashbangTitle.TextXAlignment = Enum.TextXAlignment.Left
    FlashbangTitle.Parent = FlashbangButton
    
    -- Flashbang Description
    local FlashbangDesc = Instance.new("TextLabel")
    FlashbangDesc.Name = "Description"
    FlashbangDesc.Size = UDim2.new(0, 200, 0, 20)
    FlashbangDesc.Position = UDim2.new(0, 15, 0, 35)
    FlashbangDesc.BackgroundTransparency = 1
    FlashbangDesc.Font = Enum.Font.Gotham
    FlashbangDesc.Text = "Pure white theme (very bright!)"
    FlashbangDesc.TextColor3 = Color3.fromRGB(200, 200, 200)
    FlashbangDesc.TextSize = 12
    FlashbangDesc.TextXAlignment = Enum.TextXAlignment.Left
    FlashbangDesc.Parent = FlashbangButton
    
    -- Flashbang Price
    local FlashbangPrice = Instance.new("TextLabel")
    FlashbangPrice.Name = "Price"
    FlashbangPrice.Size = UDim2.new(0, 80, 0, 25)
    FlashbangPrice.Position = UDim2.new(1, -90, 0.5, -12)
    FlashbangPrice.BackgroundTransparency = 1
    FlashbangPrice.Font = Enum.Font.GothamBold
    FlashbangPrice.Text = "1000 clicks"
    FlashbangPrice.TextColor3 = Color3.fromRGB(255, 220, 100)
    FlashbangPrice.TextSize = 14
    FlashbangPrice.Parent = FlashbangButton
    
    -- Coming Soon Text
    local ComingSoonLabel = Instance.new("TextLabel")
    ComingSoonLabel.Name = "ComingSoonLabel"
    ComingSoonLabel.Size = UDim2.new(0, 200, 0, 20)
    ComingSoonLabel.Position = UDim2.new(0.5, -100, 0, 550)
    ComingSoonLabel.BackgroundTransparency = 1
    ComingSoonLabel.Font = Enum.Font.Gotham
    ComingSoonLabel.Text = "more coming soon!"
    ComingSoonLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
    ComingSoonLabel.TextSize = 12
    ComingSoonLabel.Parent = ShopScrollFrame
    
    -- Create Content for Settings Tab
    local SettingsContent = Instance.new("Frame")
    SettingsContent.Name = "SettingsContent"
    SettingsContent.Size = UDim2.new(1, 0, 1, 0)
    SettingsContent.BackgroundTransparency = 1
    SettingsContent.Visible = false
    SettingsContent.Parent = ContentFrame
    
    -- Theme Label
    local ThemeSettingsLabel = Instance.new("TextLabel")
    ThemeSettingsLabel.Name = "ThemeSettingsLabel"
    ThemeSettingsLabel.Size = UDim2.new(0, 200, 0, 30)
    ThemeSettingsLabel.Position = UDim2.new(0, 20, 0, 20)
    ThemeSettingsLabel.BackgroundTransparency = 1
    ThemeSettingsLabel.Font = Enum.Font.GothamSemibold
    ThemeSettingsLabel.Text = "Select Theme:"
    ThemeSettingsLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    ThemeSettingsLabel.TextSize = 16
    ThemeSettingsLabel.TextXAlignment = Enum.TextXAlignment.Left
    ThemeSettingsLabel.Parent = SettingsContent
    
    -- Add glow to theme label
    local ThemeSettingsGlow = Instance.new("UIStroke")
    ThemeSettingsGlow.Thickness = 1.5
    ThemeSettingsGlow.Color = Color3.fromRGB(180, 120, 255)
    ThemeSettingsGlow.Transparency = 0.3
    ThemeSettingsGlow.Parent = ThemeSettingsLabel
    
    -- Theme Dropdown Button
    local ThemeButton = Instance.new("TextButton")
    ThemeButton.Name = "ThemeButton"
    ThemeButton.Size = UDim2.new(0, 200, 0, 40)
    ThemeButton.Position = UDim2.new(0, 20, 0, 60)
    ThemeButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    ThemeButton.Text = CurrentTheme
    ThemeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    ThemeButton.Font = Enum.Font.GothamSemibold
    ThemeButton.TextSize = 14
    ThemeButton.Parent = SettingsContent
    
    -- Add rounded corners to theme button
    local ThemeCorner = Instance.new("UICorner")
    ThemeCorner.CornerRadius = UDim.new(0, 8)
    ThemeCorner.Parent = ThemeButton
    
    -- Add outline to theme button
    local ThemeOutline = Instance.new("UIStroke")
    ThemeOutline.Thickness = 2
    ThemeOutline.Color = Color3.fromRGB(120, 120, 120)
    ThemeOutline.Transparency = 0.3
    ThemeOutline.Parent = ThemeButton
    
    -- Theme Dropdown ScrollingFrame
    local ThemeDropdown = Instance.new("ScrollingFrame")
    ThemeDropdown.Name = "ThemeDropdown"
    ThemeDropdown.Size = UDim2.new(0, 200, 0, 150) -- Fixed height with scrolling
    ThemeDropdown.Position = UDim2.new(0, 20, 0, 105)
    ThemeDropdown.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    ThemeDropdown.BorderSizePixel = 0
    ThemeDropdown.ScrollBarThickness = 4
    ThemeDropdown.ScrollBarImageColor3 = Color3.fromRGB(180, 120, 255)
    ThemeDropdown.Visible = false
    ThemeDropdown.ZIndex = 10
    ThemeDropdown.Parent = SettingsContent
    
    -- Add rounded corners to dropdown
    local DropdownCorner = Instance.new("UICorner")
    DropdownCorner.CornerRadius = UDim.new(0, 8)
    DropdownCorner.Parent = ThemeDropdown
    
    -- Add outline to dropdown
    local DropdownOutline = Instance.new("UIStroke")
    DropdownOutline.Thickness = 2
    DropdownOutline.Color = Color3.fromRGB(120, 120, 120)
    DropdownOutline.Transparency = 0.3
    DropdownOutline.Parent = ThemeDropdown
    
    -- Create theme options based on purchased themes
    local ThemeOptions = {"Tokyo Purple", "Green Hill", "Blue Ocean"}
    if PurchasedThemes["Yellow Lemon"] then
        table.insert(ThemeOptions, "Yellow Lemon")
    end
    if PurchasedThemes["Blood Red"] then
        table.insert(ThemeOptions, "Blood Red")
    end
    if PurchasedThemes["Flashbang"] then
        table.insert(ThemeOptions, "Flashbang")
    end
    
    -- Set canvas size based on number of themes
    ThemeDropdown.CanvasSize = UDim2.new(0, 0, 0, #ThemeOptions * 40)
    
    for i, theme in ipairs(ThemeOptions) do
        local ThemeOption = Instance.new("TextButton")
        ThemeOption.Name = theme .. "Option"
        ThemeOption.Size = UDim2.new(1, -20, 0, 30)
        ThemeOption.Position = UDim2.new(0, 10, 0, (i-1) * 40 + 5)
        ThemeOption.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
        ThemeOption.Text = theme
        ThemeOption.TextColor3 = Color3.fromRGB(255, 255, 255)
        ThemeOption.Font = Enum.Font.GothamSemibold
        ThemeOption.TextSize = 14
        ThemeOption.ZIndex = 11
        ThemeOption.Parent = ThemeDropdown
        
        -- Add NEW! label if it's a new theme
        if NewThemes[theme] then
            local NewLabel = Instance.new("TextLabel")
            NewLabel.Name = "NewLabel"
            NewLabel.Size = UDim2.new(0, 40, 0, 20)
            NewLabel.Position = UDim2.new(1, -50, 0.5, -10)
            NewLabel.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
            NewLabel.Text = "NEW!"
            NewLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
            NewLabel.Font = Enum.Font.GothamBold
            NewLabel.TextSize = 10
            NewLabel.ZIndex = 12
            NewLabel.Parent = ThemeOption
            
            -- Add rounded corners to NEW! label
            local NewLabelCorner = Instance.new("UICorner")
            NewLabelCorner.CornerRadius = UDim.new(0, 4)
            NewLabelCorner.Parent = NewLabel
        end
        
        -- Add rounded corners to options
        local OptionCorner = Instance.new("UICorner")
        OptionCorner.CornerRadius = UDim.new(0, 6)
        OptionCorner.Parent = ThemeOption
        
        -- Add outline to options
        local OptionOutline = Instance.new("UIStroke")
        OptionOutline.Thickness = 1
        OptionOutline.Color = Color3.fromRGB(120, 120, 120)
        OptionOutline.Transparency = 0.5
        OptionOutline.Parent = ThemeOption
        
        -- Theme option click event
        ThemeOption.MouseButton1Click:Connect(function()
            CurrentTheme = theme
            ThemeButton.Text = theme
            ThemeDropdown.Visible = false
            ApplyTheme(UIGradient, MainOutline, theme)
            
            -- Remove NEW! label if clicked
            if NewThemes[theme] then
                NewThemes[theme] = nil
                if ThemeOption:FindFirstChild("NewLabel") then
                    ThemeOption.NewLabel:Destroy()
                end
            end
        end)
    end
    
    -- Toggle dropdown visibility
    ThemeButton.MouseButton1Click:Connect(function()
        ThemeDropdown.Visible = not ThemeDropdown.Visible
    end)
    
    -- Hide Username Toggle Label
    local HideUserLabel = Instance.new("TextLabel")
    HideUserLabel.Name = "HideUserLabel"
    HideUserLabel.Size = UDim2.new(0, 200, 0, 30)
    HideUserLabel.Position = UDim2.new(0, 20, 0, 120)
    HideUserLabel.BackgroundTransparency = 1
    HideUserLabel.Font = Enum.Font.GothamSemibold
    HideUserLabel.Text = "Hide Username:"
    HideUserLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    HideUserLabel.TextSize = 16
    HideUserLabel.TextXAlignment = Enum.TextXAlignment.Left
    HideUserLabel.Parent = SettingsContent
    
    -- Add NEW! indicator to Hide User Label
    local HideUserNew = Instance.new("TextLabel")
    HideUserNew.Name = "NewIndicator"
    HideUserNew.Size = UDim2.new(0, 40, 0, 20)
    HideUserNew.Position = UDim2.new(0, 150, 0.5, -10)
    HideUserNew.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
    HideUserNew.Text = "NEW!"
    HideUserNew.TextColor3 = Color3.fromRGB(255, 255, 255)
    HideUserNew.Font = Enum.Font.GothamBold
    HideUserNew.TextSize = 10
    HideUserNew.Parent = HideUserLabel
    
    -- Add rounded corners to NEW! indicator
    local HideUserNewCorner = Instance.new("UICorner")
    HideUserNewCorner.CornerRadius = UDim.new(0, 4)
    HideUserNewCorner.Parent = HideUserNew
    
    -- Add glow to hide user label
    local HideUserGlow = Instance.new("UIStroke")
    HideUserGlow.Thickness = 1.5
    HideUserGlow.Color = Color3.fromRGB(180, 120, 255)
    HideUserGlow.Transparency = 0.3
    HideUserGlow.Parent = HideUserLabel
    
    -- Hide Username Toggle Button
    local HideUserToggle = Instance.new("Frame")
    HideUserToggle.Name = "HideUserToggle"
    HideUserToggle.Size = UDim2.new(0, 50, 0, 24)
    HideUserToggle.Position = UDim2.new(0, 20, 0, 160)
    HideUserToggle.BackgroundColor3 = IsUsernameHidden and Color3.fromRGB(100, 200, 100) or Color3.fromRGB(200, 100, 100)
    HideUserToggle.Parent = SettingsContent
    
    -- Add rounded corners to toggle
    local ToggleCorner = Instance.new("UICorner")
    ToggleCorner.CornerRadius = UDim.new(1, 0) -- Fully round
    ToggleCorner.Parent = HideUserToggle
    
    -- Toggle Knob
    local ToggleKnob = Instance.new("Frame")
    ToggleKnob.Name = "Knob"
    ToggleKnob.Size = UDim2.new(0, 20, 0, 20)
    ToggleKnob.Position = IsUsernameHidden and UDim2.new(1, -22, 0.5, -10) or UDim2.new(0, 2, 0.5, -10)
    ToggleKnob.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    ToggleKnob.Parent = HideUserToggle
    
    -- Add rounded corners to knob
    local KnobCorner = Instance.new("UICorner")
    KnobCorner.CornerRadius = UDim.new(1, 0) -- Fully round
    KnobCorner.Parent = ToggleKnob
    
    -- Toggle Status Text
    local ToggleStatus = Instance.new("TextLabel")
    ToggleStatus.Name = "Status"
    ToggleStatus.Size = UDim2.new(0, 100, 0, 24)
    ToggleStatus.Position = UDim2.new(0, 80, 0, 160)
    ToggleStatus.BackgroundTransparency = 1
    ToggleStatus.Font = Enum.Font.GothamSemibold
    ToggleStatus.Text = IsUsernameHidden and "ON" or "OFF"
    ToggleStatus.TextColor3 = IsUsernameHidden and Color3.fromRGB(100, 200, 100) or Color3.fromRGB(200, 100, 100)
    ToggleStatus.TextSize = 14
    ToggleStatus.TextXAlignment = Enum.TextXAlignment.Left
    ToggleStatus.Parent = SettingsContent
    
    -- Toggle click event
    HideUserToggle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            IsUsernameHidden = not IsUsernameHidden
            
            -- Update toggle appearance
            HideUserToggle.BackgroundColor3 = IsUsernameHidden and Color3.fromRGB(100, 200, 100) or Color3.fromRGB(200, 100, 100)
            ToggleStatus.Text = IsUsernameHidden and "ON" or "OFF"
            ToggleStatus.TextColor3 = IsUsernameHidden and Color3.fromRGB(100, 200, 100) or Color3.fromRGB(200, 100, 100)
            
            -- Animate knob
            local knobPosition = IsUsernameHidden and UDim2.new(1, -22, 0.5, -10) or UDim2.new(0, 2, 0.5, -10)
            local knobAnimation = TweenService:Create(
                ToggleKnob,
                TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
                {Position = knobPosition}
            )
            
            knobAnimation:Play()
            
            -- Update welcome text
            WelcomeLabel.Text = "Welcome back, " .. (IsUsernameHidden and "User" or PlayerName) .. "!"
        end
    end)
    
    -- Create Content for New! Tab
    local NewContent = Instance.new("Frame")
    NewContent.Name = "NewContent"
    NewContent.Size = UDim2.new(1, 0, 1, 0)
    NewContent.BackgroundTransparency = 1
    NewContent.Visible = false
    NewContent.Parent = ContentFrame
    
    -- New Features ScrollingFrame
    local NewScrollFrame = Instance.new("ScrollingFrame")
    NewScrollFrame.Name = "NewScrollFrame"
    NewScrollFrame.Size = UDim2.new(1, 0, 1, 0)
    NewScrollFrame.BackgroundTransparency = 1
    NewScrollFrame.BorderSizePixel = 0
    NewScrollFrame.ScrollBarThickness = 6
    NewScrollFrame.ScrollBarImageColor3 = Color3.fromRGB(180, 120, 255)
    NewScrollFrame.CanvasSize = UDim2.new(0, 0, 0, 400) -- Adjust based on content
    NewScrollFrame.Parent = NewContent
    
    -- New Features Title
    local NewFeaturesTitle = Instance.new("TextLabel")
    NewFeaturesTitle.Name = "NewFeaturesTitle"
    NewFeaturesTitle.Size = UDim2.new(0, 300, 0, 40)
    NewFeaturesTitle.Position = UDim2.new(0.5, -150, 0, 10)
    NewFeaturesTitle.BackgroundTransparency = 1
    NewFeaturesTitle.Font = Enum.Font.GothamBold
    NewFeaturesTitle.Text = "New Features!"
    NewFeaturesTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    NewFeaturesTitle.TextSize = 24
    NewFeaturesTitle.Parent = NewScrollFrame
    
    -- Add glow to new features title
    local NewFeaturesTitleGlow = Instance.new("UIStroke")
    NewFeaturesTitleGlow.Thickness = 1.5
    NewFeaturesTitleGlow.Color = Color3.fromRGB(255, 100, 100)
    NewFeaturesTitleGlow.Transparency = 0.3
    NewFeaturesTitleGlow.Parent = NewFeaturesTitle
    
    -- Feature 1: Hide Username
    local Feature1 = Instance.new("Frame")
    Feature1.Name = "Feature1"
    Feature1.Size = UDim2.new(0, 300, 0, 80)
    Feature1.Position = UDim2.new(0.5, -150, 0, 60)
    Feature1.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    Feature1.Parent = NewScrollFrame
    
    -- Add rounded corners to feature frame
    local Feature1Corner = Instance.new("UICorner")
    Feature1Corner.CornerRadius = UDim.new(0, 8)
    Feature1Corner.Parent = Feature1
    
    -- Feature 1 Title
    local Feature1Title = Instance.new("TextLabel")
    Feature1Title.Name = "Title"
    Feature1Title.Size = UDim2.new(0, 280, 0, 30)
    Feature1Title.Position = UDim2.new(0.5, -140, 0, 10)
    Feature1Title.BackgroundTransparency = 1
    Feature1Title.Font = Enum.Font.GothamBold
    Feature1Title.Text = "Hide Username"
    Feature1Title.TextColor3 = Color3.fromRGB(255, 255, 255)
    Feature1Title.TextSize = 18
    Feature1Title.TextXAlignment = Enum.TextXAlignment.Left
    Feature1Title.Parent = Feature1
    
    -- Feature 1 Description
    local Feature1Desc = Instance.new("TextLabel")
    Feature1Desc.Name = "Description"
    Feature1Desc.Size = UDim2.new(0, 280, 0, 40)
    Feature1Desc.Position = UDim2.new(0.5, -140, 0, 40)
    Feature1Desc.BackgroundTransparency = 1
    Feature1Desc.Font = Enum.Font.Gotham
    Feature1Desc.Text = "Toggle to hide your username in the welcome message for privacy."
    Feature1Desc.TextColor3 = Color3.fromRGB(200, 200, 200)
    Feature1Desc.TextSize = 12
    Feature1Desc.TextWrapped = true
    Feature1Desc.TextXAlignment = Enum.TextXAlignment.Left
    Feature1Desc.Parent = Feature1
    
    -- Feature 2: Auto Upgrade
    local Feature2 = Instance.new("Frame")
    Feature2.Name = "Feature2"
    Feature2.Size = UDim2.new(0, 300, 0, 80)
    Feature2.Position = UDim2.new(0.5, -150, 0, 150)
    Feature2.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    Feature2.Parent = NewScrollFrame
    
    -- Add rounded corners to feature frame
    local Feature2Corner = Instance.new("UICorner")
    Feature2Corner.CornerRadius = UDim.new(0, 8)
    Feature2Corner.Parent = Feature2
    
    -- Feature 2 Title
    local Feature2Title = Instance.new("TextLabel")
    Feature2Title.Name = "Title"
    Feature2Title.Size = UDim2.new(0, 280, 0, 30)
    Feature2Title.Position = UDim2.new(0.5, -140, 0, 10)
    Feature2Title.BackgroundTransparency = 1
    Feature2Title.Font = Enum.Font.GothamBold
    Feature2Title.Text = "Auto Upgrade"
    Feature2Title.TextColor3 = Color3.fromRGB(255, 255, 255)
    Feature2Title.TextSize = 18
    Feature2Title.TextXAlignment = Enum.TextXAlignment.Left
    Feature2Title.Parent = Feature2
    
    -- Feature 2 Description
    local Feature2Desc = Instance.new("TextLabel")
    Feature2Desc.Name = "Description"
    Feature2Desc.Size = UDim2.new(0, 280, 0, 40)
    Feature2Desc.Position = UDim2.new(0.5, -140, 0, 40)
    Feature2Desc.BackgroundTransparency = 1
    Feature2Desc.Font = Enum.Font.Gotham
    Feature2Desc.Text = "Automatically purchases +1 click multiplier upgrades when you have enough clicks."
    Feature2Desc.TextColor3 = Color3.fromRGB(200, 200, 200)
    Feature2Desc.TextSize = 12
    Feature2Desc.TextWrapped = true
    Feature2Desc.TextXAlignment = Enum.TextXAlignment.Left
    Feature2Desc.Parent = Feature2
    
    -- Feature 3: New Themes
    local Feature3 = Instance.new("Frame")
    Feature3.Name = "Feature3"
    Feature3.Size = UDim2.new(0, 300, 0, 80)
    Feature3.Position = UDim2.new(0.5, -150, 0, 240)
    Feature3.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    Feature3.Parent = NewScrollFrame
    
    -- Add rounded corners to feature frame
    local Feature3Corner = Instance.new("UICorner")
    Feature3Corner.CornerRadius = UDim.new(0, 8)
    Feature3Corner.Parent = Feature3
    
    -- Feature 3 Title
    local Feature3Title = Instance.new("TextLabel")
    Feature3Title.Name = "Title"
    Feature3Title.Size = UDim2.new(0, 280, 0, 30)
    Feature3Title.Position = UDim2.new(0.5, -140, 0, 10)
    Feature3Title.BackgroundTransparency = 1
    Feature3Title.Font = Enum.Font.GothamBold
    Feature3Title.Text = "New Themes"
    Feature3Title.TextColor3 = Color3.fromRGB(255, 255, 255)
    Feature3Title.TextSize = 18
    Feature3Title.TextXAlignment = Enum.TextXAlignment.Left
    Feature3Title.Parent = Feature3
    
    -- Feature 3 Description
    local Feature3Desc = Instance.new("TextLabel")
    Feature3Desc.Name = "Description"
    Feature3Desc.Size = UDim2.new(0, 280, 0, 40)
    Feature3Desc.Position = UDim2.new(0.5, -140, 0, 40)
    Feature3Desc.BackgroundTransparency = 1
    Feature3Desc.Font = Enum.Font.Gotham
    Feature3Desc.Text = "Purchase new themes like Yellow Lemon, Blood Red, and Flashbang to customize your UI."
    Feature3Desc.TextColor3 = Color3.fromRGB(200, 200, 200)
    Feature3Desc.TextSize = 12
    Feature3Desc.TextWrapped = true
    Feature3Desc.TextXAlignment = Enum.TextXAlignment.Left
    Feature3Desc.Parent = Feature3
    
    -- Feature 4: Scrollable Dropdown
    local Feature4 = Instance.new("Frame")
    Feature4.Name = "Feature4"
    Feature4.Size = UDim2.new(0, 300, 0, 80)
    Feature4.Position = UDim2.new(0.5, -150, 0, 330)
    Feature4.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    Feature4.Parent = NewScrollFrame
    
    -- Add rounded corners to feature frame
    local Feature4Corner = Instance.new("UICorner")
    Feature4Corner.CornerRadius = UDim.new(0, 8)
    Feature4Corner.Parent = Feature4
    
    -- Feature 4 Title
    local Feature4Title = Instance.new("TextLabel")
    Feature4Title.Name = "Title"
    Feature4Title.Size = UDim2.new(0, 280, 0, 30)
    Feature4Title.Position = UDim2.new(0.5, -140, 0, 10)
    Feature4Title.BackgroundTransparency = 1
    Feature4Title.Font = Enum.Font.GothamBold
    Feature4Title.Text = "Scrollable Dropdown"
    Feature4Title.TextColor3 = Color3.fromRGB(255, 255, 255)
    Feature4Title.TextSize = 18
    Feature4Title.TextXAlignment = Enum.TextXAlignment.Left
    Feature4Title.Parent = Feature4
    
    -- Feature 4 Description
    local Feature4Desc = Instance.new("TextLabel")
    Feature4Desc.Name = "Description"
    Feature4Desc.Size = UDim2.new(0, 280, 0, 40)
    Feature4Desc.Position = UDim2.new(0.5, -140, 0, 40)
    Feature4Desc.BackgroundTransparency = 1
    Feature4Desc.Font = Enum.Font.Gotham
    Feature4Desc.Text = "Theme dropdown is now scrollable, making it easier to navigate through many themes."
    Feature4Desc.TextColor3 = Color3.fromRGB(200, 200, 200)
    Feature4Desc.TextSize = 12
    Feature4Desc.TextWrapped = true
    Feature4Desc.TextXAlignment = Enum.TextXAlignment.Left
    Feature4Desc.Parent = Feature4
    
    -- Tab switching logic
    local TabContents = {ClickerContent, ShopContent, SettingsContent, NewContent}
    
    for i, button in ipairs(TabButtons) do
        button.MouseButton1Click:Connect(function()
            -- Hide all content frames
            for _, content in ipairs(TabContents) do
                content.Visible = false
            end
            
            -- Show selected content
            TabContents[i].Visible = true
            
            -- Update button appearance
            for j, btn in ipairs(TabButtons) do
                if j == i then
                    btn.BackgroundColor3 = Color3.fromRGB(100, 70, 180)
                else
                    btn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
                end
            end
            
            -- Remove NEW! indicator from New! tab when clicked
            if i == 4 and button:FindFirstChild("NewIndicator") then
                button.NewIndicator:Destroy()
            end
        end)
    end
    
    -- Set default selected tab
    TabButtons[1].BackgroundColor3 = Color3.fromRGB(100, 70, 180)
    
    -- Clicker button functionality
    ClickerButton.MouseButton1Click:Connect(function()
        ClickCount = ClickCount + (1 * ClickMultiplier)
        ClicksLabel.Text = "Clicks: " .. ClickCount
        
        -- Update shop item descriptions
        MultiplierDesc.Text = "Get +1 click per click (Current: x" .. ClickMultiplier .. ")"
        
        -- Animation for button click
        local clickAnimation = TweenService:Create(
            ClickerButton,
            TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
            {Size = UDim2.new(0, 140, 0, 70)}
        )
        
        local returnAnimation = TweenService:Create(
            ClickerButton,
            TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
            {Size = UDim2.new(0, 150, 0, 80)}
        )
        
        clickAnimation:Play()
        clickAnimation.Completed:Connect(function()
            returnAnimation:Play()
        end)
        
        -- Animation for click counter
        local labelAnimation = TweenService:Create(
            ClicksLabel,
            TweenInfo.new(0.2, Enum.EasingStyle.Bounce, Enum.EasingDirection.Out),
            {TextSize = 22}
        )
        
        local labelReturnAnimation = TweenService:Create(
            ClicksLabel,
            TweenInfo.new(0.2, Enum.EasingStyle.Bounce, Enum.EasingDirection.Out),
            {TextSize = 18}
        )
        
        labelAnimation:Play()
        labelAnimation.Completed:Connect(function()
            labelReturnAnimation:Play()
        end)
    end)
    
    -- Auto Clicker button functionality
    AutoClickerButton.MouseButton1Click:Connect(function()
        if ClickCount >= 250 and not HasAutoClicker then
            ClickCount = ClickCount - 250
            ClicksLabel.Text = "Clicks: " .. ClickCount
            HasAutoClicker = true
            
            -- Update button appearance
            AutoClickerButton.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
            AutoClickerTitle.TextColor3 = Color3.fromRGB(150, 150, 150)
            AutoClickerDesc.TextColor3 = Color3.fromRGB(120, 120, 120)
            AutoClickerPrice.Text = "Purchased"
            AutoClickerPrice.TextColor3 = Color3.fromRGB(100, 200, 100)
            
            -- Start auto clicking - properly timed to once per second
            LastAutoClickTime = tick()
            AutoClickerInterval = RunService.Heartbeat:Connect(function()
                local currentTime = tick()
                if currentTime - LastAutoClickTime >= 1 then  -- Check if 1 second has passed
                    ClickCount = ClickCount + (1 * ClickMultiplier)
                    ClicksLabel.Text = "Clicks: " .. ClickCount
                    LastAutoClickTime = currentTime
                end
            end)
        end
    end)
    
    -- Multiplier button functionality
    MultiplierButton.MouseButton1Click:Connect(function()
        if ClickCount >= 100 then
            ClickCount = ClickCount - 100
            ClicksLabel.Text = "Clicks: " .. ClickCount
            ClickMultiplier = ClickMultiplier + 1
            
            -- Update description
            MultiplierDesc.Text = "Get +1 click per click (Current: x" .. ClickMultiplier .. ")"
            
            -- Animation for multiplier purchase
            local purchaseAnimation = TweenService:Create(
                MultiplierButton,
                TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
                {BackgroundColor3 = Color3.fromRGB(100, 200, 100)}
            )
            
            local returnAnimation = TweenService:Create(
                MultiplierButton,
                TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
                {BackgroundColor3 = Color3.fromRGB(60, 60, 60)}
            )
            
            purchaseAnimation:Play()
            purchaseAnimation.Completed:Connect(function()
                returnAnimation:Play()
            end)
        end
    end)
    
    -- Auto Upgrade button functionality
    AutoUpgradeButton.MouseButton1Click:Connect(function()
        if ClickCount >= 500 and not HasAutoUpgrade then
            ClickCount = ClickCount - 500
            ClicksLabel.Text = "Clicks: " .. ClickCount
            HasAutoUpgrade = true
            
            -- Update button appearance
            AutoUpgradeButton.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
            AutoUpgradeTitle.TextColor3 = Color3.fromRGB(150, 150, 150)
            AutoUpgradeDesc.TextColor3 = Color3.fromRGB(120, 120, 120)
            AutoUpgradePrice.Text = "Purchased"
            AutoUpgradePrice.TextColor3 = Color3.fromRGB(100, 200, 100)
            
            -- Remove NEW! indicator
            if AutoUpgradeButton:FindFirstChild("NewIndicator") then
                AutoUpgradeButton.NewIndicator:Destroy()
            end
            
            -- Start auto upgrading
            AutoUpgradeInterval = RunService.Heartbeat:Connect(function()
                if ClickCount >= 100 then
                    ClickCount = ClickCount - 100
                    ClicksLabel.Text = "Clicks: " .. ClickCount
                    ClickMultiplier = ClickMultiplier + 1
                    
                    -- Update description
                    MultiplierDesc.Text = "Get +1 click per click (Current: x" .. ClickMultiplier .. ")"
                end
            end)
        end
    end)
    
    -- Yellow Lemon theme purchase
    YellowLemonButton.MouseButton1Click:Connect(function()
        if ClickCount >= 500 and not PurchasedThemes["Yellow Lemon"] then
            ClickCount = ClickCount - 500
            ClicksLabel.Text = "Clicks: " .. ClickCount
            PurchasedThemes["Yellow Lemon"] = true
            NewThemes["Yellow Lemon"] = true
            
            -- Update button appearance
            YellowLemonButton.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
            YellowLemonTitle.TextColor3 = Color3.fromRGB(150, 150, 150)
            YellowLemonDesc.TextColor3 = Color3.fromRGB(120, 120, 120)
            YellowLemonPrice.Text = "Purchased"
            YellowLemonPrice.TextColor3 = Color3.fromRGB(100, 200, 100)
            
            -- Recreate theme dropdown to include new theme
            RecreateThemeDropdown(SettingsContent, ThemeDropdown)
        end
    end)
    
    -- Blood Red theme purchase
    BloodRedButton.MouseButton1Click:Connect(function()
        if ClickCount >= 750 and not PurchasedThemes["Blood Red"] then
            ClickCount = ClickCount - 750
            ClicksLabel.Text = "Clicks: " .. ClickCount
            PurchasedThemes["Blood Red"] = true
            NewThemes["Blood Red"] = true
            
            -- Update button appearance
            BloodRedButton.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
            BloodRedTitle.TextColor3 = Color3.fromRGB(150, 150, 150)
            BloodRedDesc.TextColor3 = Color3.fromRGB(120, 120, 120)
            BloodRedPrice.Text = "Purchased"
            BloodRedPrice.TextColor3 = Color3.fromRGB(100, 200, 100)
            
            -- Recreate theme dropdown to include new theme
            RecreateThemeDropdown(SettingsContent, ThemeDropdown)
        end
    end)
    
    -- Flashbang theme purchase
    FlashbangButton.MouseButton1Click:Connect(function()
        if ClickCount >= 1000 and not PurchasedThemes["Flashbang"] then
            ClickCount = ClickCount - 1000
            ClicksLabel.Text = "Clicks: " .. ClickCount
            PurchasedThemes["Flashbang"] = true
            NewThemes["Flashbang"] = true
            
            -- Update button appearance
            FlashbangButton.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
            FlashbangTitle.TextColor3 = Color3.fromRGB(150, 150, 150)
            FlashbangDesc.TextColor3 = Color3.fromRGB(120, 120, 120)
            FlashbangPrice.Text = "Purchased"
            FlashbangPrice.TextColor3 = Color3.fromRGB(100, 200, 100)
            
            -- Recreate theme dropdown to include new theme
            RecreateThemeDropdown(SettingsContent, ThemeDropdown)
        end
    end)
    
    -- Minimize button functionality
    MinimizeButton.MouseButton1Click:Connect(function()
        IsMinimized = not IsMinimized
        
        if IsMinimized then
            ContentContainer.Visible = false
            MainFrame.Size = UDim2.new(0, 500, 0, 50)
        else
            ContentContainer.Visible = true
            MainFrame.Size = UDim2.new(0, 500, 0, 300)
        end
    end)
    
    -- Close button functionality
    CloseButton.MouseButton1Click:Connect(function()
        -- Clear intervals if they exist
        if AutoClickerInterval then
            AutoClickerInterval:Disconnect()
            AutoClickerInterval = nil
        end
        
        if AutoUpgradeInterval then
            AutoUpgradeInterval:Disconnect()
            AutoUpgradeInterval = nil
        end
        
        ClickerUI:Destroy()
    end)
    
    -- Make UI draggable
    local dragging
    local dragInput
    local dragStart
    local startPos
    
    local function update(input)
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
    
    MainFrame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = MainFrame.Position
            
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)
    
    MainFrame.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            dragInput = input
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            update(input)
        end
    end)
    
    return ClickerUI
end

-- Function to recreate theme dropdown
function RecreateThemeDropdown(parent, oldDropdown)
    -- Remove old dropdown
    if oldDropdown then
        oldDropdown:Destroy()
    end
    
    -- Create new dropdown as a ScrollingFrame
    local ThemeDropdown = Instance.new("ScrollingFrame")
    ThemeDropdown.Name = "ThemeDropdown"
    ThemeDropdown.Size = UDim2.new(0, 200, 0, 150) -- Fixed height with scrolling
    ThemeDropdown.Position = UDim2.new(0, 20, 0, 105)
    ThemeDropdown.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    ThemeDropdown.BorderSizePixel = 0
    ThemeDropdown.ScrollBarThickness = 4
    ThemeDropdown.ScrollBarImageColor3 = Color3.fromRGB(180, 120, 255)
    ThemeDropdown.Visible = false
    ThemeDropdown.ZIndex = 10
    ThemeDropdown.Parent = parent
    
    -- Add rounded corners to dropdown
    local DropdownCorner = Instance.new("UICorner")
    DropdownCorner.CornerRadius = UDim.new(0, 8)
    DropdownCorner.Parent = ThemeDropdown
    
    -- Add outline to dropdown
    local DropdownOutline = Instance.new("UIStroke")
    DropdownOutline.Thickness = 2
    DropdownOutline.Color = Color3.fromRGB(120, 120, 120)
    DropdownOutline.Transparency = 0.3
    DropdownOutline.Parent = ThemeDropdown
    
    -- Create theme options based on purchased themes
    local ThemeOptions = {}
    for theme, _ in pairs(PurchasedThemes) do
        table.insert(ThemeOptions, theme)
    end
    
    -- Set canvas size based on number of themes
    ThemeDropdown.CanvasSize = UDim2.new(0, 0, 0, #ThemeOptions * 40)
    
    for i, theme in ipairs(ThemeOptions) do
        local ThemeOption = Instance.new("TextButton")
        ThemeOption.Name = theme .. "Option"
        ThemeOption.Size = UDim2.new(1, -20, 0, 30)
        ThemeOption.Position = UDim2.new(0, 10, 0, (i-1) * 40 + 5)
        ThemeOption.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
        ThemeOption.Text = theme
        ThemeOption.TextColor3 = Color3.fromRGB(255, 255, 255)
        ThemeOption.Font = Enum.Font.GothamSemibold
        ThemeOption.TextSize = 14
        ThemeOption.ZIndex = 11
        ThemeOption.Parent = ThemeDropdown
        
        -- Add NEW! label if it's a new theme
        if NewThemes[theme] then
            local NewLabel = Instance.new("TextLabel")
            NewLabel.Name = "NewLabel"
            NewLabel.Size = UDim2.new(0, 40, 0, 20)
            NewLabel.Position = UDim2.new(1, -50, 0.5, -10)
            NewLabel.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
            NewLabel.Text = "NEW!"
            NewLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
            NewLabel.Font = Enum.Font.GothamBold
            NewLabel.TextSize = 10
            NewLabel.ZIndex = 12
            NewLabel.Parent = ThemeOption
            
            -- Add rounded corners to NEW! label
            local NewLabelCorner = Instance.new("UICorner")
            NewLabelCorner.CornerRadius = UDim.new(0, 4)
            NewLabelCorner.Parent = NewLabel
        end
        
        -- Add rounded corners to options
        local OptionCorner = Instance.new("UICorner")
        OptionCorner.CornerRadius = UDim.new(0, 6)
        OptionCorner.Parent = ThemeOption
        
        -- Add outline to options
        local OptionOutline = Instance.new("UIStroke")
        OptionOutline.Thickness = 1
        OptionOutline.Color = Color3.fromRGB(120, 120, 120)
        OptionOutline.Transparency = 0.5
        OptionOutline.Parent = ThemeOption
        
        -- Theme option click event
        ThemeOption.MouseButton1Click:Connect(function()
            CurrentTheme = theme
            parent.ThemeButton.Text = theme
            ThemeDropdown.Visible = false
            
            -- Apply the theme
            local mainFrame = parent.Parent.Parent.Parent
            local uiGradient = mainFrame:FindFirstChild("UIGradient")
            local mainOutline = mainFrame:FindFirstChild("UIStroke")
            
            if uiGradient and mainOutline then
                ApplyTheme(uiGradient, mainOutline, theme)
            end
            
            -- Remove NEW! label if clicked
            if NewThemes[theme] then
                NewThemes[theme] = nil
                if ThemeOption:FindFirstChild("NewLabel") then
                    ThemeOption.NewLabel:Destroy()
                end
            end
        end)
    end
    
    -- Toggle dropdown visibility
    parent.ThemeButton.MouseButton1Click:Connect(function()
        ThemeDropdown.Visible = not ThemeDropdown.Visible
    end)
    
    return ThemeDropdown
end

-- Function to apply theme
function ApplyTheme(gradient, outline, theme)
    if theme == "Tokyo Purple" then
        gradient.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(100, 50, 180)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(30, 10, 60))
        })
        outline.Color = Color3.fromRGB(180, 120, 255)
    elseif theme == "Green Hill" then
        gradient.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(30, 120, 60)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(50, 180, 255))
        })
        outline.Color = Color3.fromRGB(100, 255, 150)
    elseif theme == "Blue Ocean" then
        gradient.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(20, 80, 150)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(10, 30, 80))
        })
        outline.Color = Color3.fromRGB(80, 150, 255)
    elseif theme == "Yellow Lemon" then
        gradient.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 230, 50)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(150, 220, 80))
        })
        outline.Color = Color3.fromRGB(255, 240, 100)
    elseif theme == "Blood Red" then
        gradient.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(180, 30, 30)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(100, 10, 10))
        })
        outline.Color = Color3.fromRGB(255, 80, 80)
    elseif theme == "Flashbang" then
        gradient.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 255, 255)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(240, 240, 240))
        })
        outline.Color = Color3.fromRGB(220, 220, 220)
    end
end

-- Create and show the UI
local UI = CreateClickerUI()
