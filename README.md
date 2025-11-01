repeat task.wait() until workspace:FindFirstChild("Plots")
local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")

-- Only run in this specific Place ID
local allowedPlaceId = 109983668079237 -- <-- Replace with your game's place ID
if game.PlaceId ~= allowedPlaceId then return end

-- IMPORTANT: Replace these with your own secure Discord webhooks
-- DO NOT share these webhooks publicly as they can be abused
local UnderTen = "YOUR_WEBHOOK_URL_HERE" -- webhook for under 10m
local OverTen = "YOUR_WEBHOOK_URL_HERE"  -- webhook for over 10m

-- 🔗 Your Replit API endpoint
local replitApiEndpoint = "https://8d93f3f5-a95f-4cc1-84d9-5d3dfb8650f5-00-3iq0togrerm7d.riker.replit.dev/api"

local embedColor = 3447003
local placeId = game.PlaceId

local req = (syn and syn.request) or (http and http.request) or (http_request) or (fluxus and fluxus.request) or request

-- Build Chilli Hub join link
local function getChilliHubJoinLink(jobId)
    return string.format(
        "https://chillihub1.github.io/chillihub-joiner/?placeId=%s&gameInstanceId=%s",
        tostring(placeId),
        tostring(jobId)
    )
end

-- Send to Replit API for GUI updates
local function sendToReplitAPI(petsFound, jobId, playerCount)
    if #petsFound == 0 then return end
    
    local Player = Players.LocalPlayer
    
    local jsonData = HttpService:JSONEncode({
        ["pets"] = petsFound,
        ["jobId"] = jobId,
        ["placeId"] = game.PlaceId,
        ["timestamp"] = os.time(),
        ["isPrivate"] = false,
        ["playerCount"] = playerCount,
        ["maxPlayers"] = Players.MaxPlayers,
        ["username"] = Player.Name,
        ["userId"] = tostring(Player.UserId),
        ["displayName"] = Player.DisplayName
    })

    if req then
        local success, response = pcall(function()
            return req({
                Url = replitApiEndpoint,
                Method = "POST",
                Headers = {
                    ["Content-Type"] = "application/json",
                    ["Authorization"] = "Bearer roblox-pet-scanner"
                },
                Body = jsonData
            })
        end)
        
        if success and response then
            print("✅ Successfully sent " .. #petsFound .. " pets to Replit API")
        else
            warn("❌ Failed to send pets to Replit API")
        end
    else
        warn("❌ HTTP request function not available")
    end
end

-- Send Webhook (original function)
local function sendWebhook(url, petFound, moneyPerSec, tag, mutation, jobId, pingEveryone, playerCount)
    local data = {
        ["username"] = "Ken Hub",
        ["embeds"] = {{
            ["title"] = "Ken Hub",
            ["description"] = petFound,
            ["color"] = embedColor,
            ["fields"] = {
                {["name"] = "💸 Money per Sec", ["value"] = moneyPerSec, ["inline"] = true},
                {["name"] = "🏷️ Tag", ["value"] = tag, ["inline"] = true},
                {["name"] = "🧬 Mutation", ["value"] = mutation, ["inline"] = true},
                {["name"] = "👥 Player Count", ["value"] = tostring(playerCount) .. "/8", ["inline"] = true},
                {["name"] = "Join Link", ["value"] = "[Join Here]("..getChilliHubJoinLink(jobId)..")", ["inline"] = false},
                {["name"] = "🆔 Job ID (Mobile)", ["value"] = "``"..jobId.."``", ["inline"] = false},
                {["name"] = "🆔 Job ID (PC)", ["value"] = "```"..jobId.."```", ["inline"] = false},
                {["name"] = "Join Server",
                 ["value"] = string.format("```lua\ngame:GetService('TeleportService'):TeleportToPlaceInstance(%d, '%s', game.Players.LocalPlayer)\n```", placeId, jobId),
                 ["inline"] = false
                }
            }
        }}
    }
    
    if pingEveryone then
        data["content"] = "@everyone"
    end
    
    local jsonData = HttpService:JSONEncode(data)
    
    req({
        Url = url,
        Method = "POST",
        Headers = {["Content-Type"] = "application/json"},
        Body = jsonData
    })
end

-- Convert money text to number
local function convertTextToNumber(text)
    if not text:find("/s") then
        return 0
    end
    
    text = text:gsub("%$", ""):gsub("/s", ""):gsub("%s+", "")
    
    local multiplier = 1
    
    if text:upper():find("K$") then
        multiplier = 1e3
        text = text:gsub("[Kk]$", "")
    elseif text:upper():find("M$") then
        multiplier = 1e6
        text = text:gsub("[Mm]$", "")
    elseif text:upper():find("B$") then
        multiplier = 1e9
        text = text:gsub("[Bb]$", "")
    elseif text:upper():find("T$") then
        multiplier = 1e12
        text = text:gsub("[Tt]$", "")
    end
    
    local num = tonumber(text) or 0
    return num * multiplier
end

-- Count players
local function getPlayerCount()
    return #Players:GetPlayers()
end

-- Collect and send webhooks
local webhooksToSend = {}
local petsForReplit = {} -- New: collect pets for Replit API

for _, v in pairs(workspace:GetDescendants()) do
    if v:IsA("TextLabel") and v.Name == "Generation" then
        local originalText = v.Text
        local value = convertTextToNumber(originalText)
        local playerCount = getPlayerCount()
        
        -- Only send if player count is 6 or 7
        if playerCount >= 6 and playerCount <= 8 and value >= 1000000 then
            local success, result = pcall(function()
                local petFound = v.Parent.DisplayName.Text
                local moneyPerSec = v.Text
                local tag = v.Parent.Rarity.Text
                local jobId = game.JobId
                
                local mutation = "None"
                local mutationTag = v.Parent:FindFirstChild("Mutation")
                if mutationTag and mutationTag.Visible then
                    mutation = mutationTag.Text
                end
                
                return petFound, moneyPerSec, tag, mutation, jobId
            end)
            
            if success then
                local petFound, moneyPerSec, tag, mutation, jobId = result, v.Text, v.Parent.Rarity.Text, "None", game.JobId
                
                local mutationTag = v.Parent:FindFirstChild("Mutation")
                if mutationTag and mutationTag.Visible then
                    mutation = mutationTag.Text
                end
                
                -- Format pet info for Replit API
                local petInfo = string.format("%s (%s, %s)", petFound, moneyPerSec, tag)
                if mutation ~= "None" then
                    petInfo = petInfo .. " [" .. mutation .. "]"
                end
                table.insert(petsForReplit, petInfo)
                
                local webhookUrl, shouldPing
                if petFound:lower() == "lucky block" then
                    if tag:lower() == "secret" then
                        webhookUrl = OverTen
                        shouldPing = false
                    else
                        webhookUrl = UnderTen
                        shouldPing = false
                    end
                else
                    webhookUrl = value >= 10000000 and OverTen or UnderTen
                    shouldPing = value >= 10000000
                end
                
                table.insert(webhooksToSend, {
                    url = webhookUrl,
                    petFound = petFound,
                    moneyPerSec = moneyPerSec,
                    tag = tag,
                    mutation = mutation,
                    jobId = jobId,
                    pingEveryone = shouldPing,
                    playerCount = playerCount
                })
            else
                warn("Error accessing pet data:", result)
            end
        end
    end
end

-- Send to Replit API first (for GUI updates)
if #petsForReplit > 0 then
    sendToReplitAPI(petsForReplit, game.JobId, getPlayerCount())
end

-- Send original Discord webhooks
for _, webhookData in pairs(webhooksToSend) do
    sendWebhook(webhookData.url, webhookData.petFound, webhookData.moneyPerSec, webhookData.tag, webhookData.mutation, webhookData.jobId, webhookData.pingEveryone, webhookData.playerCount)
    task.wait(0.1)
end

if #webhooksToSend > 0 then
    print("Sent", #webhooksToSend, "webhooks for player count 6–7.")
    print("Also sent", #petsForReplit, "pets to Replit API for GUI updates.")
end