-- ⚡ Ultra Fast Multi-Clone Server Hopper ⚡
-- Hops every 0.8s
-- Each clone randomizes start & server choice
-- Avoids all clones picking the same server
-- Only joins servers with 6-7 players

local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")

local PlaceId = game.PlaceId
local Player = Players.LocalPlayer
local Cursor = nil

-- settings
local HopDelay = 0.8 -- ultra fast hopping
local MinPlayers = 6 -- minimum players required
local MaxPlayers = 7 -- maximum players allowed

-- random startup delay (desync clones)
task.wait(math.random(1, 10))

-- Teleport safely
local function SafeTeleport(serverId)
    local ok, err = pcall(function()
        TeleportService:TeleportToPlaceInstance(PlaceId, serverId, Player)
    end)
    if not ok then
        warn("Teleport failed: " .. tostring(err))
        return false
    end
    return true
end

-- Grab server list and pick best one
local function GetServers()
    local success, servers = pcall(function()
        local url = "https://games.roblox.com/v1/games/" .. PlaceId .. "/servers/Public?sortOrder=Asc&limit=100"
        if Cursor then
            url = url .. "&cursor=" .. Cursor
        end
        return HttpService:JSONDecode(game:HttpGet(url))
    end)

    if success and servers and servers.data then
        Cursor = servers.nextPageCursor
        if not Cursor then
            Cursor = nil -- reset for next cycle
        end
        return servers.data
    end
    return {}
end

-- Main hopper
local function Hop()
    local servers = GetServers()
    if #servers == 0 then return end

    -- Filter servers to only include those with 6-7 players
    local filteredServers = {}
    for _, server in ipairs(servers) do
        if server.playing >= MinPlayers and server.playing <= MaxPlayers and server.id ~= game.JobId then
            table.insert(filteredServers, server)
        end
    end
    
    if #filteredServers == 0 then
        warn("No servers found with " .. MinPlayers .. "-" .. MaxPlayers .. " players")
        return
    end

    -- Shuffle list so each clone picks differently
    for i = #filteredServers, 2, -1 do
        local j = math.random(i)
        filteredServers[i], filteredServers[j] = filteredServers[j], filteredServers[i]
    end

    -- Try servers
    for _, server in ipairs(filteredServers) do
        if SafeTeleport(server.id) then
            return
        end
    end
end

-- loop
task.spawn(function()
    while task.wait(HopDelay) do
        pcall(Hop)
    end
end)