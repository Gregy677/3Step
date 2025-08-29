-- 🔗 API Endpoint - Your Replit URL
getgenv().websiteEndpoint = "https://8d93f3f5-a95f-4cc1-84d9-5d3dfb8650f5-00-3iq0togrerm7d.riker.replit.dev/api"

-- Settings
local settings = {
    autoJoin = false,
    threshold = 10000000, -- 10M default
    currentTab = "all"
}

-- Roblox Services
local Players = game:GetService("Players")
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")
local RunService = game:GetService("RunService")
local StarterGui = game:GetService("StarterGui")
local Player = Players.LocalPlayer

-- Variables for tracking updates
local lastDataHash = ""

-- ======================
-- 🖥️ Ken Hub Pet Finder GUI
-- ======================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "KenHubPetFinder"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = Player:WaitForChild("PlayerGui")

-- Toggle Button
local ToggleButton = Instance.new("TextButton")
ToggleButton.Name = "ToggleButton"
ToggleButton.Size = UDim2.new(0, 60, 0, 60)
ToggleButton.Position = UDim2.new(1, -70, 0.5, -30)
ToggleButton.AnchorPoint = Vector2.new(1, 0.5)
ToggleButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
ToggleButton.BorderColor3 = Color3.fromRGB(0, 162, 255)
ToggleButton.BorderSizePixel = 3
ToggleButton.Text = "🎯"
ToggleButton.TextScaled = true
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.Font = Enum.Font.SourceSansBold
ToggleButton.Active = true
ToggleButton.Draggable = true
ToggleButton.ZIndex = 10
ToggleButton.Parent = ScreenGui

local ToggleCorner = Instance.new("UICorner")
ToggleCorner.CornerRadius = UDim.new(0, 15)
ToggleCorner.Parent = ToggleButton

-- Main Frame
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 380, 0, 500)
MainFrame.Position = UDim2.new(0.5, -190, 0.5, -250)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
MainFrame.BorderSizePixel = 0
MainFrame.Visible = false
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 15)
MainCorner.Parent = MainFrame

-- Header Frame
local HeaderFrame = Instance.new("Frame")
HeaderFrame.Name = "HeaderFrame"
HeaderFrame.Size = UDim2.new(1, 0, 0, 80)
HeaderFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
HeaderFrame.BorderSizePixel = 0
HeaderFrame.Parent = MainFrame

local HeaderCorner = Instance.new("UICorner")
HeaderCorner.CornerRadius = UDim.new(0, 15)
HeaderCorner.Parent = HeaderFrame

-- Ken Hub Title
local TitleText = Instance.new("TextLabel")
TitleText.Size = UDim2.new(0.6, 0, 0.6, 0)
TitleText.Position = UDim2.new(0, 15, 0, 5)
TitleText.BackgroundTransparency = 1
TitleText.TextColor3 = Color3.fromRGB(0, 162, 255)
TitleText.Text = "Ken Hub Pet Finder"
TitleText.TextScaled = true
TitleText.TextXAlignment = Enum.TextXAlignment.Left
TitleText.Font = Enum.Font.SourceSansBold
TitleText.Parent = HeaderFrame

-- Subtitle
local SubtitleText = Instance.new("TextLabel")
SubtitleText.Size = UDim2.new(0.6, 0, 0.3, 0)
SubtitleText.Position = UDim2.new(0, 15, 0.6, 0)
SubtitleText.BackgroundTransparency = 1
SubtitleText.TextColor3 = Color3.fromRGB(150, 150, 150)
SubtitleText.Text = "Live Pet Scanner"
SubtitleText.TextScaled = true
SubtitleText.TextXAlignment = Enum.TextXAlignment.Left
SubtitleText.Font = Enum.Font.SourceSans
SubtitleText.Parent = HeaderFrame

-- Refresh Button
local RefreshButton = Instance.new("TextButton")
RefreshButton.Size = UDim2.new(0, 80, 0, 30)
RefreshButton.Position = UDim2.new(1, -90, 0, 10)
RefreshButton.BackgroundColor3 = Color3.fromRGB(0, 162, 255)
RefreshButton.BorderSizePixel = 0
RefreshButton.Text = "🔄 Refresh"
RefreshButton.TextColor3 = Color3.fromRGB(255, 255, 255)
RefreshButton.TextScaled = true
RefreshButton.Font = Enum.Font.SourceSansBold
RefreshButton.Parent = HeaderFrame

local RefreshCorner = Instance.new("UICorner")
RefreshCorner.CornerRadius = UDim.new(0, 8)
RefreshCorner.Parent = RefreshButton

-- Tab Frame
local TabFrame = Instance.new("Frame")
TabFrame.Size = UDim2.new(1, -20, 0, 35)
TabFrame.Position = UDim2.new(0, 10, 0, 85)
TabFrame.BackgroundTransparency = 1
TabFrame.Parent = MainFrame

-- All Tab
local AllTab = Instance.new("TextButton")
AllTab.Size = UDim2.new(0.25, -5, 1, 0)
AllTab.Position = UDim2.new(0, 0, 0, 0)
AllTab.BackgroundColor3 = Color3.fromRGB(0, 162, 255)
AllTab.Text = "All"
AllTab.TextColor3 = Color3.fromRGB(255, 255, 255)
AllTab.TextScaled = true
AllTab.Font = Enum.Font.SourceSansBold
AllTab.Parent = TabFrame

local AllTabCorner = Instance.new("UICorner")
AllTabCorner.CornerRadius = UDim.new(0, 8)
AllTabCorner.Parent = AllTab

-- 1M+ Tab
local OneMillionTab = Instance.new("TextButton")
OneMillionTab.Size = UDim2.new(0.25, -5, 1, 0)
OneMillionTab.Position = UDim2.new(0.25, 5, 0, 0)
OneMillionTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
OneMillionTab.Text = "1M+/s"
OneMillionTab.TextColor3 = Color3.fromRGB(200, 200, 200)
OneMillionTab.TextScaled = true
OneMillionTab.Font = Enum.Font.SourceSansBold
OneMillionTab.Parent = TabFrame

local OneMillionTabCorner = Instance.new("UICorner")
OneMillionTabCorner.CornerRadius = UDim.new(0, 8)
OneMillionTabCorner.Parent = OneMillionTab

-- 3M+ Tab
local ThreeMillionTab = Instance.new("TextButton")
ThreeMillionTab.Size = UDim2.new(0.25, -5, 1, 0)
ThreeMillionTab.Position = UDim2.new(0.5, 10, 0, 0)
ThreeMillionTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
ThreeMillionTab.Text = "3M+/s"
ThreeMillionTab.TextColor3 = Color3.fromRGB(200, 200, 200)
ThreeMillionTab.TextScaled = true
ThreeMillionTab.Font = Enum.Font.SourceSansBold
ThreeMillionTab.Parent = TabFrame

local ThreeMillionTabCorner = Instance.new("UICorner")
ThreeMillionTabCorner.CornerRadius = UDim.new(0, 8)
ThreeMillionTabCorner.Parent = ThreeMillionTab

-- Settings Tab
local SettingsTab = Instance.new("TextButton")
SettingsTab.Size = UDim2.new(0.25, -5, 1, 0)
SettingsTab.Position = UDim2.new(0.75, 15, 0, 0)
SettingsTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
SettingsTab.Text = "⚙️"
SettingsTab.TextColor3 = Color3.fromRGB(200, 200, 200)
SettingsTab.TextScaled = true
SettingsTab.Font = Enum.Font.SourceSansBold
SettingsTab.Parent = TabFrame

local SettingsTabCorner = Instance.new("UICorner")
SettingsTabCorner.CornerRadius = UDim.new(0, 8)
SettingsTabCorner.Parent = SettingsTab

-- Status Label
local StatusLabel = Instance.new("TextLabel")
StatusLabel.Size = UDim2.new(1, -20, 0, 25)
StatusLabel.Position = UDim2.new(0, 10, 0, 130)
StatusLabel.BackgroundTransparency = 1
StatusLabel.TextColor3 = Color3.fromRGB(0, 162, 255)
StatusLabel.Text = "🔄 Loading pet sightings..."
StatusLabel.TextScaled = true
StatusLabel.TextXAlignment = Enum.TextXAlignment.Left
StatusLabel.Font = Enum.Font.SourceSansBold
StatusLabel.Parent = MainFrame

-- Content Frame (for pets list)
local ContentFrame = Instance.new("ScrollingFrame")
ContentFrame.Size = UDim2.new(1, -20, 1, -170)
ContentFrame.Position = UDim2.new(0, 10, 0, 160)
ContentFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
ContentFrame.ScrollBarThickness = 8
ContentFrame.BackgroundTransparency = 1
ContentFrame.BorderSizePixel = 0
ContentFrame.ScrollBarImageColor3 = Color3.fromRGB(0, 162, 255)
ContentFrame.Parent = MainFrame

local UIList = Instance.new("UIListLayout")
UIList.Padding = UDim.new(0, 10)
UIList.SortOrder = Enum.SortOrder.LayoutOrder
UIList.Parent = ContentFrame

-- Settings Frame
local SettingsFrame = Instance.new("Frame")
SettingsFrame.Size = UDim2.new(1, -20, 1, -170)
SettingsFrame.Position = UDim2.new(0, 10, 0, 160)
SettingsFrame.BackgroundTransparency = 1
SettingsFrame.Visible = false
SettingsFrame.Parent = MainFrame

-- Auto Join Toggle
local AutoJoinLabel = Instance.new("TextLabel")
AutoJoinLabel.Size = UDim2.new(1, -80, 0, 30)
AutoJoinLabel.Position = UDim2.new(0, 0, 0, 10)
AutoJoinLabel.BackgroundTransparency = 1
AutoJoinLabel.Text = "Auto Join High Value Pets:"
AutoJoinLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
AutoJoinLabel.TextScaled = true
AutoJoinLabel.TextXAlignment = Enum.TextXAlignment.Left
AutoJoinLabel.Font = Enum.Font.SourceSansBold
AutoJoinLabel.Parent = SettingsFrame

local AutoJoinToggle = Instance.new("TextButton")
AutoJoinToggle.Size = UDim2.new(0, 60, 0, 30)
AutoJoinToggle.Position = UDim2.new(1, -60, 0, 10)
AutoJoinToggle.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
AutoJoinToggle.Text = "OFF"
AutoJoinToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
AutoJoinToggle.TextScaled = true
AutoJoinToggle.Font = Enum.Font.SourceSansBold
AutoJoinToggle.Parent = SettingsFrame

local AutoJoinCorner = Instance.new("UICorner")
AutoJoinCorner.CornerRadius = UDim.new(0, 8)
AutoJoinCorner.Parent = AutoJoinToggle

-- Threshold Label
local ThresholdLabel = Instance.new("TextLabel")
ThresholdLabel.Size = UDim2.new(1, -130, 0, 30)
ThresholdLabel.Position = UDim2.new(0, 0, 0, 60)
ThresholdLabel.BackgroundTransparency = 1
ThresholdLabel.Text = "Auto Join Threshold ($/s):"
ThresholdLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
ThresholdLabel.TextScaled = true
ThresholdLabel.TextXAlignment = Enum.TextXAlignment.Left
ThresholdLabel.Font = Enum.Font.SourceSansBold
ThresholdLabel.Parent = SettingsFrame

-- Threshold TextBox
local ThresholdBox = Instance.new("TextBox")
ThresholdBox.Size = UDim2.new(0, 120, 0, 30)
ThresholdBox.Position = UDim2.new(1, -120, 0, 60)
ThresholdBox.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
ThresholdBox.BorderSizePixel = 0
ThresholdBox.Text = "10000000"
ThresholdBox.TextColor3 = Color3.fromRGB(255, 255, 255)
ThresholdBox.TextScaled = true
ThresholdBox.Font = Enum.Font.SourceSans
ThresholdBox.PlaceholderText = "10000000"
ThresholdBox.Parent = SettingsFrame

local ThresholdCorner = Instance.new("UICorner")
ThresholdCorner.CornerRadius = UDim.new(0, 8)
ThresholdCorner.Parent = ThresholdBox

-- Extract money value from pet string
local function extractMoneyValue(petString)
    local moneyMatch = petString:match("%$([%d%.]+)([KMBTkmbt]?)/?s")
    if not moneyMatch then return 0 end
    
    local value = tonumber(moneyMatch)
    local suffix = petString:match("%$[%d%.]+([KMBTkmbt])/?s")
    
    if suffix then
        suffix = suffix:upper()
        if suffix == "K" then
            value = value * 1000
        elseif suffix == "M" then
            value = value * 1000000
        elseif suffix == "B" then
            value = value * 1000000000
        elseif suffix == "T" then
            value = value * 1000000000000
        end
    end
    
    return value or 0
end

-- Auto join function
local function autoJoinCheck(entry)
    if not settings.autoJoin then return end
    
    for _, pet in ipairs(entry.pets) do
        local value = extractMoneyValue(pet)
        if value >= settings.threshold then
            -- Auto join this server
            if entry.joinScript and entry.joinScript ~= "" then
                local placeId, jobId = string.match(entry.joinScript, 'TeleportToPlaceInstance%((%d+),%s*["\']([^"\']+)["\']')
                if placeId and jobId then
                    print("🚀 Auto-joining server for: " .. pet)
                    StarterGui:SetCore("SendNotification", {
                        Title = "Ken Hub Auto Join";
                        Text = "Joining server for " .. pet;
                        Duration = 3;
                    })
                    task.wait(1)
                    TeleportService:TeleportToPlaceInstance(tonumber(placeId), jobId)
                    return true
                end
            end
        end
    end
    return false
end

-- Filter pets based on tab - FIXED VERSION
local function filterPetsByTab(entries, tab)
    if tab == "all" then return entries end
    
    local filtered = {}
    for _, entry in ipairs(entries) do
        local filteredPets = {}
        for _, pet in ipairs(entry.pets) do
            local value = extractMoneyValue(pet)
            -- Fixed the tab checking logic
            if (tab == "1m" and value >= 1000000) or (tab == "3m" and value >= 3000000) then
                table.insert(filteredPets, pet)
            end
        end
        -- Only include entries that have pets meeting the criteria
        if #filteredPets > 0 then
            local newEntry = {}
            for k, v in pairs(entry) do
                newEntry[k] = v
            end
            newEntry.pets = filteredPets
            table.insert(filtered, newEntry)
        end
    end
    return filtered
end

-- Tab switching function - FIXED VERSION
local function switchTab(newTab)
    if newTab == "settings" then
        ContentFrame.Visible = false
        SettingsFrame.Visible = true
        StatusLabel.Visible = false
        settings.currentTab = newTab
        
        -- Update tab colors
        AllTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        AllTab.TextColor3 = Color3.fromRGB(200, 200, 200)
        OneMillionTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        OneMillionTab.TextColor3 = Color3.fromRGB(200, 200, 200)
        ThreeMillionTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        ThreeMillionTab.TextColor3 = Color3.fromRGB(200, 200, 200)
        SettingsTab.BackgroundColor3 = Color3.fromRGB(0, 162, 255)
        SettingsTab.TextColor3 = Color3.fromRGB(255, 255, 255)
        return
    end
    
    ContentFrame.Visible = true
    SettingsFrame.Visible = false
    StatusLabel.Visible = true
    settings.currentTab = newTab
    
    -- Update tab appearances
    if newTab == "all" then
        AllTab.BackgroundColor3 = Color3.fromRGB(0, 162, 255)
        AllTab.TextColor3 = Color3.fromRGB(255, 255, 255)
        OneMillionTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        OneMillionTab.TextColor3 = Color3.fromRGB(200, 200, 200)
        ThreeMillionTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        ThreeMillionTab.TextColor3 = Color3.fromRGB(200, 200, 200)
        SettingsTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        SettingsTab.TextColor3 = Color3.fromRGB(200, 200, 200)
    elseif newTab == "1m" then
        AllTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        AllTab.TextColor3 = Color3.fromRGB(200, 200, 200)
        OneMillionTab.BackgroundColor3 = Color3.fromRGB(0, 162, 255)
        OneMillionTab.TextColor3 = Color3.fromRGB(255, 255, 255)
        ThreeMillionTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        ThreeMillionTab.TextColor3 = Color3.fromRGB(200, 200, 200)
        SettingsTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        SettingsTab.TextColor3 = Color3.fromRGB(200, 200, 200)
    elseif newTab == "3m" then
        AllTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        AllTab.TextColor3 = Color3.fromRGB(200, 200, 200)
        OneMillionTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        OneMillionTab.TextColor3 = Color3.fromRGB(200, 200, 200)
        ThreeMillionTab.BackgroundColor3 = Color3.fromRGB(0, 162, 255)
        ThreeMillionTab.TextColor3 = Color3.fromRGB(255, 255, 255)
        SettingsTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        SettingsTab.TextColor3 = Color3.fromRGB(200, 200, 200)
    end
    
    loadPetList()
end

-- Parser function
local function parsePetData(rawText)
    local entries, currentEntry, inJoinScript = {}, {}, false
    
    for line in rawText:gmatch("[^\r\n]+") do
        if line:match("^%[.+%]$") then
            if next(currentEntry) ~= nil then 
                table.insert(entries, currentEntry) 
            end
            currentEntry = { 
                timestamp = line:match("%[(.-)%]"), 
                pets = {}, 
                joinScript = "", 
                players = "" 
            }
            inJoinScript = false
        elseif line:match("```lua") then
            inJoinScript = true
        elseif line:match("```") and inJoinScript then
            inJoinScript = false
        elseif inJoinScript then
            currentEntry.joinScript = line
        elseif line:match("Players: ") then
            currentEntry.players = line:match("Players: (.-)$")
        elseif line ~= "" and currentEntry.joinScript == "" and currentEntry.timestamp then
            table.insert(currentEntry.pets, line)
        end
    end
    
    if next(currentEntry) ~= nil then 
        table.insert(entries, currentEntry) 
    end
    
    -- Sort by timestamp (latest first)
    table.sort(entries, function(a, b)
        return (a.timestamp or "") > (b.timestamp or "")
    end)
    
    return entries
end

-- Entry builder function
local function createPetEntry(entry, index)
    local Entry = Instance.new("Frame")
    Entry.Name = "PetEntry" .. index
    Entry.Size = UDim2.new(1, 0, 0, 95)
    Entry.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    Entry.BorderSizePixel = 0
    Entry.LayoutOrder = index
    Entry.Parent = ContentFrame

    local EntryCorner = Instance.new("UICorner")
    EntryCorner.CornerRadius = UDim.new(0, 12)
    EntryCorner.Parent = Entry

    -- Pet Names Label
    local PetLabel = Instance.new("TextLabel")
    PetLabel.Size = UDim2.new(1, -20, 0, 40)
    PetLabel.Position = UDim2.new(0, 10, 0, 5)
    PetLabel.BackgroundTransparency = 1
    local petText = #entry.pets > 0 and table.concat(entry.pets, ", ") or "No pets found"
    PetLabel.Text = "🎯 " .. petText
    PetLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    PetLabel.TextScaled = true
    PetLabel.TextWrapped = true
    PetLabel.TextXAlignment = Enum.TextXAlignment.Left
    PetLabel.Font = Enum.Font.SourceSansBold
    PetLabel.Parent = Entry

    -- Timestamp and Player Labels
    local TimestampLabel = Instance.new("TextLabel")
    TimestampLabel.Size = UDim2.new(0.5, 0, 0, 20)
    TimestampLabel.Position = UDim2.new(0, 10, 0, 45)
    TimestampLabel.BackgroundTransparency = 1
    TimestampLabel.Text = "⏰ " .. (entry.timestamp or "Unknown time")
    TimestampLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
    TimestampLabel.TextScaled = true
    TimestampLabel.TextXAlignment = Enum.TextXAlignment.Left
    TimestampLabel.Font = Enum.Font.SourceSans
    TimestampLabel.Parent = Entry

    local PlayerLabel = Instance.new("TextLabel")
    PlayerLabel.Size = UDim2.new(0.3, 0, 0, 20)
    PlayerLabel.Position = UDim2.new(0.5, 0, 0, 45)
    PlayerLabel.BackgroundTransparency = 1
    PlayerLabel.Text = "👥 " .. (entry.players or "0/0")
    PlayerLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
    PlayerLabel.TextScaled = true
    PlayerLabel.TextXAlignment = Enum.TextXAlignment.Left
    PlayerLabel.Font = Enum.Font.SourceSans
    PlayerLabel.Parent = Entry

    -- Join Button
    local JoinButton = Instance.new("TextButton")
    JoinButton.Size = UDim2.new(0, 70, 0, 25)
    JoinButton.Position = UDim2.new(1, -80, 0, 65)
    JoinButton.Text = "JOIN"
    JoinButton.BackgroundColor3 = Color3.fromRGB(0, 162, 255)
    JoinButton.BorderSizePixel = 0
    JoinButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    JoinButton.TextScaled = true
    JoinButton.Font = Enum.Font.SourceSansBold
    JoinButton.Parent = Entry

    local JoinCorner = Instance.new("UICorner")
    JoinCorner.CornerRadius = UDim.new(0, 6)
    JoinCorner.Parent = JoinButton

    JoinButton.MouseButton1Click:Connect(function()
        if entry.joinScript and entry.joinScript ~= "" then
            local placeId, jobId = string.match(entry.joinScript, 'TeleportToPlaceInstance%((%d+),%s*["\']([^"\']+)["\']')
            if placeId and jobId then
                TeleportService:TeleportToPlaceInstance(tonumber(placeId), jobId)
                StarterGui:SetCore("SendNotification", {
                    Title = "Ken Hub";
                    Text = "Joining server...";
                    Duration = 2;
                })
            end
        end
    end)
end

-- Load pet list function
local function loadPetList()
    if settings.currentTab == "settings" then return end
    
    local success, result = pcall(function()
        return game:HttpGet("https://8d93f3f5-a95f-4cc1-84d9-5d3dfb8650f5-00-3iq0togrerm7d.riker.replit.dev/Brainrot/pets_found.txt")
    end)

    if success and result then
        local dataHash = HttpService:JSONEncode(result):len()
        if dataHash == lastDataHash then return end
        lastDataHash = dataHash
        
        -- Clear existing entries
        for _, child in ipairs(ContentFrame:GetChildren()) do
            if child:IsA("Frame") then child:Destroy() end
        end

        local petData = parsePetData(result)
        
        -- Check for auto-join before filtering
        for _, entry in ipairs(petData) do
            if autoJoinCheck(entry) then
                return -- Stop processing if we're auto-joining
            end
        end
        
        local filteredData = filterPetsByTab(petData, settings.currentTab)
        
        if #filteredData > 0 then
            StatusLabel.Text = "✅ Found " .. #filteredData .. " sightings"
            for i = 1, math.min(15, #filteredData) do
                createPetEntry(filteredData[i], i)
            end
        else
            if settings.currentTab == "all" then
                StatusLabel.Text = "📭 No pets found"
            elseif settings.currentTab == "1m" then
                StatusLabel.Text = "📭 No pets over 1M/s found"
            elseif settings.currentTab == "3m" then
                StatusLabel.Text = "📭 No pets over 3M/s found"
            end
        end
        
        ContentFrame.CanvasSize = UDim2.new(0, 0, 0, UIList.AbsoluteContentSize.Y + 20)
    else
        StatusLabel.Text = "❌ Failed to load data"
    end
end

-- Event connections
RefreshButton.MouseButton1Click:Connect(loadPetList)
ToggleButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

-- Tab connections
AllTab.MouseButton1Click:Connect(function() switchTab("all") end)
OneMillionTab.MouseButton1Click:Connect(function() switchTab("1m") end)
ThreeMillionTab.MouseButton1Click:Connect(function() switchTab("3m") end)
SettingsTab.MouseButton1Click:Connect(function() switchTab("settings") end)

-- Settings connections
AutoJoinToggle.MouseButton1Click:Connect(function()
    settings.autoJoin = not settings.autoJoin
    if settings.autoJoin then
        AutoJoinToggle.BackgroundColor3 = Color3.fromRGB(0, 162, 255)
        AutoJoinToggle.Text = "ON"
    else
        AutoJoinToggle.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
        AutoJoinToggle.Text = "OFF"
    end
end)

ThresholdBox.FocusLost:Connect(function()
    local value = tonumber(ThresholdBox.Text)
    if value and value > 0 then
        settings.threshold = value
    else
        ThresholdBox.Text = tostring(settings.threshold)
    end
end)

-- Initial load
task.wait(2)
loadPetList()

-- Auto-refresh every 2 seconds
task.spawn(function()
    while true do
        task.wait(2)
        if MainFrame.Visible then
            loadPetList()
        end
    end
end)

print("🎯 Ken Hub Pet Finder loaded successfully!")